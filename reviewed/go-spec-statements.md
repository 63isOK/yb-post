# 语句

上一节详细介绍了表达式，语句就是由表达式组成

语句控制了程序的执行。

    Statement =
      Declaration | LabeledStmt | SimpleStmt |
      GoStmt | ReturnStmt | BreakStmt | ContinueStmt | GotoStmt |
      FallthroughStmt | Block | IfStmt | SwitchStmt | SelectStmt | ForStmt |
      DeferStmt .

    SimpleStmt = EmptyStmt | ExpressionStmt | SendStmt | IncDecStmt |
                 Assignment | ShortVarDecl .

ebnf分析：
Go中的语句定义有很多类(15类)，其中简单语句还细分6种。

下面先介绍语句的结束，再详细介每一种语句。

## 结束语句

Go的术语是"结束语句"，也可以理解为终止语句，同一个块中，遇到这类语句，
语句下面的剩下语句就不会执行了。

在下列情况中，执行到某一点表示遇到了结束语句：

1. return goto 语句
2. 调用内置函数panic()
3. 在块中({}中)，遇到了结束语句
4. if语句中，如果有else分支，那这两个分支都是结束语句(走一个另一个就不走)
5. for语句中，没有break，没有循环条件(无限循环)，for后面的语句就不会走了
6. switch语句中，没有break，如果有default，那每个分支都以一个结束语句结束，
走一个分支，其他分支就不会走了。特例是fallthrough 语句
7. select语句中，没有break，每个分支都以一个结束语句结束
8. 标签语句，标记了结束语句

## 空语句

空语句表示什么都不会执行

    EmptyStmt = .

简单语句的一种

## 标签语句

标签语句是用于 goto/break/continue语句中

    LabeledStmt = Label ":" Statement .
    Label       = identifier .

虽然Go提供了，但大多数场景都不会用到这个

另外标签语句的标签虽然是标识符，但和变量标识符并不冲突

## 表达式语句

除了指定的内置函数，函数调用/方法调用/接收操作都可以在一个语句的上下文中出现，
出现的时候可能会用()包裹

    ExpressionStmt = Expression .

下列内置函数是不能出现在语句的上下文中的：

    append cap complex imag len make new real
    unsafe.Alignof unsafe.Offsetof unsafe.Sizeof
    复数的3个内置函数
    make new
    cap和len
    unsafe的几个函数

看看下面的例子：

    h(x+y)
    f.Close()
    <-ch
    (<-ch)
    len("foo")  // illegal if len is the built-in function

下面看看为啥cap len不能出现在语句的上下文中

    i := []int{1,2,3}
    len(i)
    append(i, 4)
    // 这就叫语句的上下文，
    // len和append以表达式语句出现在块中，那就违反了规则
    // 编译器会报错，说你计算了一个值，但没有使用，就认为是错的。这也合理

    i = append(i, 5) 
    // 这个为啥不出错
    // 因为这个不是表达式语句 ，而是赋值语句, 没毛病

表达式语句，也是简单语句中的一种

## send语句

send语句执行的是接收操作。就是在channel中发送一个值

    SendStmt = Channel "<-" Expression .
    Channel  = Expression .

send语句是利用接收操作符来完成的，操作符两段的channel和值表达式都在send之前计算。
把接收操作的过程称为"通讯"(一端收一端发)。

通讯会阻塞，直到可以进行发送。怎么判断何时可以进行发送：

- 非缓冲信道，接收者准备好了，才可以开始发送
- 缓冲信道，只要信道的缓冲还没有满，才可以发送

向一个已关闭的信道执行发送，会报运行时异常。
向一个nil信道执行发送，会永久阻塞。和接收一样。

这个send语句，只是给信道发送的语句，接收应该会放在表达式或赋值语句里。
发送语句，也是一个简单语句。

## IncDec语句

就是++ --

操作数增加或减去1 (这个1是无类型的)

和赋值语句类似，操作数是要可寻址的，或者是map的索引表达式

    IncDecStmt = Expression ( "++" | "--" ) .

从ebnf上看，++ -- 都在后面

## 赋值语句

    Assignment = ExpressionList assign_op ExpressionList .

    assign_op = [ add_op | mul_op ] "=" .

从ebnf上看，= 等号两边都是表达式列表

这里的赋值操作符是有扩展的，加法/乘积操作符都可以与等号结合。

赋值语句的左操作数是要可寻址，或这是map的索引表达式，或者空白标识符(只能用于=)。
操作数可以使用(),可选。

    x = 1
    *p = f()
    a[i] = 23
    (k) = <-ch  // same as: k = <-ch

赋值操作，分两个阶段：左操作数中的索引表达式/间接指针和右操作数的表达式都按
正常顺序计算;赋值列表按从左到右进行赋值。

    a, b = b, a  // exchange a and b

    x := []int{1, 2, 3}
    i := 0
    i, x[i] = 1, 2  // set i = 1, x[0] = 2

    i = 0
    x[i], i = 2, 1  // set x[0] = 2, i = 1

    x[0], x[0] = 1, 2  // set x[0] = 1, then x[0] = 2 (so x[0] == 2 at end)

    x[1], x[3] = 4, 5  // set x[1] = 4, then panic setting x[3] = 5.

    type Point struct { x, y int }
    var p *Point
    x[2], p.x = 6, 7  // set x[2] = 6, then panic setting p.x = 7

    i = 2
    x = []int{3, 5, 7}
    for i, x[i] = range x {  // set i, x[2] = 0, x[0]
      break
    }
    // after this loop, i == 0 and x == []int{3, 5, 3}

赋值表达式的一些规则：

- 所有类型的值都可以赋值给空白标识符
- 无类型常量赋值给一个接口变量或空白标识符，这个常量会先隐式转换成指定类型
- 无类型布尔常量赋值给一个接口便来嗯或空白标识符，这个常量会先隐式转换成bool类型
