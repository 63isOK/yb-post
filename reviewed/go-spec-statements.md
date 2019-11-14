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
