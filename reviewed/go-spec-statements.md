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

## if语句

if语句是用条件变量来控制分支的语句，条件变量的结果是一个布尔值。
如果条件变量是true，if分支执行;条件变量是false，else分支执行。

    IfStmt = "if" [ SimpleStmt ";" ] Expression Block
             [ "else" ( IfStmt | Block ) ] .

if语句中可以插入一个简单语句，执行完之后，再执行if语句。

从ebnf中还可以看出 else分支中还可以嵌套if语句，或是直接是块

## switch语句

switch 提供了多个分支，用表达式或指定类型去和每个case做比较

    SwitchStmt = ExprSwitchStmt | TypeSwitchStmt .

从ebnf中可以看出switch有两种，一种是常用的switch语句，一种是switch type组合。
一种是比较表达式和case的值，一种是比较类型。

### 表达式 switch

switch中的表达式和case的表达式(不一定要是常量)，计算顺序都是从左到右，从上到下。
第一个匹配的case，会触发相应块的执行，其他case都会被跳过。
如果没有case匹配，就会执行default分支。

如果没有switch表达式，就默认是常量 true

    ExprSwitchStmt = "switch" [ SimpleStmt ";" ] [ Expression ]
                     "{" { ExprCaseClause } "}" .
    ExprCaseClause = ExprSwitchCase ":" StatementList .
    ExprSwitchCase = "case" ExpressionList | "default" .

从ebnf上看，和c/c++的switch没什么区别(唯一区别是case后可跟多个表达式)

如果switch表达式是无类型常量，会优先隐式转换成默认类型，
如果是无类型布尔值，会优先隐式转换成bool类型。
nil不能充当switch表达式。

如果case表达式是无类型的，会优先隐式转换成switch表达式的类型。
switch表达式和case表达式，她们的值要可以进行有效的比较的。

switch表达式，如果是申明和初始化，可以理解为一个未显式指明类型的临时变量。

在case和default中，非最后的分支，可能会是一个fallthrough语句，
这个fallthrough语句的意思是从本分支的结尾处跳到下一个分支的开头，继续执行。
此时是不用关系下个分支的比较结果。一般fallthrough出现在分支的最后面。

switch前面也能插入简单语句。

### type switch

这个类型switch是一个新东西，用于比较类型，而不是比较值。

新东西，写法很固定：

    switch x.(type) {
    // cases
    }

x是动态类型(接口类型)，case里是具体类型，且要实现x的接口类型。

    TypeSwitchStmt  = "switch" [ SimpleStmt ";" ] TypeSwitchGuard
                      "{" { TypeCaseClause } "}" .
    TypeSwitchGuard = [ identifier ":=" ] PrimaryExpr "." "(" "type" ")" .
    TypeCaseClause  = TypeSwitchCase ":" StatementList .
    TypeSwitchCase  = "case" TypeList | "default" .
    TypeList        = Type { "," Type } .

从ebnf中可以看出，类型switch可能还包含一个短变量声明。
如果有这个声明，那短变量的作用域就覆盖所有的分支。
如果case后只有一个类型，短变量就是这个类型;否则，短变量的类型就从主要表达式而来。

case分支，是可以出现nil的，匹配的情况是：接口变量是nil。最多只有一个nil。

fallthrough语句不能出现在类型switch中。类型switch是可以转成if-else的。

## for语句

for语句是循环语句，一个for由3部分组成：迭代因子(用于条件判断);for分支;range分支。

    ForStmt = "for" [ Condition | ForClause | RangeClause ] Block .
    Condition = Expression .

### 单条件的for语句

    for a < b {
      a *= 2
    }

最简单的格式，只要条件不为false，就一直循环下去。
每次迭代都会检查条件。条件是可以省略的，等同于true，此时就是无限循环

### 普通for分支的for语句

    for i := 0; i < 10; i++ {
      f(i)
    }

会有一个初始化(前置语句，可以是赋值语句);一个后置语句(可以是自增自减)。
前置的初始化语句可能是短变量赋值语句。后置语句不能是短变量赋值语句。

    ForClause = [ InitStmt ] ";" [ Condition ] ";" [ PostStmt ] .
    InitStmt = SimpleStmt .
    PostStmt = SimpleStmt .

for分支的情况，上面3个语句都可以省略，或省略部分，但分号不能省。
除非退化成单条件for语句，这样分号就可以省略了。

### range分支的for语句

for range语句，只适用于迭代 array/slice/string/map/channel

    RangeClause = [ ExpressionList "=" | IdentifierList ":=" ] "range"
                  Expression .

从ebnf中可以看出，=和:=都能用

range后面的表达式称为range表达式，可以是数组/切片/字符串/map，
或是数组指针，带接收操作符的channel变量(要有权限：方向要对)。

和赋值一样，左操作数要能被寻址，或是map的索引表达式。
如果range表达式是channel，每次最多迭代一个元素。
如果迭代变量是空白标识符，就和没有标识符类似。

range表达式在迭代开始时计算一次，后面都不计算了。只有一个例外：
如果只有一个迭代变量，len(x)是个常数，则range表达式就不计算了。

如果左操作数中包含函数调用，那每次迭代都会计算一次

for range 的规则：

- 数组/数组指针/切片
  - 第一个返回值是索引，第二个是值
- 字符串string
  - 第一个返回值是索引，第二个是rune
  - 如果是用for分支，就是按byte遍历

    a := "天天蓝abc不是兰"
    for x, y := range a {
      fmt.Println(x, string(y))
    }
    Output:
    0 天
    3 天
    6 蓝
    9 a
    10 b
    11 c
    12 不
    15 是
    18 兰

- map
  - 迭代的顺序不是固定(本来就不是顺序的，Go还特意加了随机处理)
  - 第一个返回值是key，第二个是value
- channel
  - 如果channel是nil，就会永久阻塞
  - 读会一直持续，直到channel被close

等range表达式计算完之后，后面就是赋值语句。

选择短变量声明或赋值语句，取决于变量是否在for外面声明。

## go语句

go语句是开始一个函数调用的执行，做为一个独立的并发线程来控制，
也叫goroutine，协程，她们使用相同的地址空间。

    GoStmt = "go" Expression .

ebnf中，go后面跟的表达式必须是函数调用或方法调用，不能用()包裹。
和表达式语句的限制一样，此处，部分内置函数是不能用的(不能做无用功)。
具体哪些函数受限了，可以看"表达式语句"一节

协程goroutine中函数值和参数的计算和普通函数的计算是一样的，
不同的是：程序不会等协程调用完了再执行，而是并发执行。

协程中的函数/方法执行完之后，协程结束，如果函数/方法还有返回值，
在执行完之后，会丢弃。

    go Server()
    go func(ch chan<- bool) { for { sleep(10); ch <- true }} (c)

## select语句

select语句，和其他语言一样，是多路复用的。
select语句会选择一个可能的操作来执行，这些操作包括(send/receive).

    SelectStmt = "select" "{" { CommClause } "}" .
    CommClause = CommCase ":" StatementList .
    CommCase   = "case" ( SendStmt | RecvStmt ) | "default" .
    RecvStmt   = [ ExpressionList "=" | IdentifierList ":=" ] RecvExpr .
    RecvExpr   = Expression .

从ebnf上看，select和switch很类似，只是没有switch中的简单语句和switch表达式，
且select中的case全是通讯操作。

接收语句中，可以获取一个或两个返回值，可以用短变量声明语句。
接收语句中，是需要存在接收操作符的，可以用()包裹。
最多一个default语句，这个default可以出现在任何地方,这点和switch一样。

select执行步骤：

1. select的所有case中，接收操作的channel操作数，send语句中的channel和右表达式，
在进入select语句时都是按源码中出现的次序，计算一次。
结果是channel的集合(接收)，或是值集合(要发送的)。不管select最后执行的是哪个分支，
这个计算的副作用都是存在的。接收语句中的赋值语句还没有执行
2. 如果一个或多个通讯可以进行执行了，select会随机选择一个进行执行。
如果没有通讯准备好，那就会执行default分支(如果有一个default分支的话)。
如果没有default分支，又没有通讯准备好，select会阻塞，直到某个通讯准备好。
3. 除了default分支，其他被选择的分支会执行相应的通讯操作
4. 如果select选择的分支是一个接收语句，里面包含短变量声明或赋值语句，
那表达式的左操作数会计算，并用接收操作的返回值来赋值
5. select选择中的分支的语句块会被执行

如果一个channel是nil，她是无发触发通讯操作的，
所以如果selelct只有一个nil 信道分支,且没有defalut分支，那select会永久阻塞。

    var a []int
    var c, c1, c2, c3, c4 chan int
    var i1, i2 int
    select {
    case i1 = <-c1:
      print("received ", i1, " from c1\n")
    case c2 <- i2:
      print("sent ", i2, " to c2\n")
    case i3, ok := (<-c3):  // same as: i3, ok := <-c3
      if ok {
        print("received ", i3, " from c3\n")
      } else {
        print("c3 is closed\n")
      }
    case a[f()] = <-c4:
      // same as:
      // case t := <-c4
      //   a[f()] = t
    default:
      print("no communication\n")
    }

    for {  // send random sequence of bits to c
      select {
      case c <- 0:  // note: no statement, no fallthrough, no folding of cases
      case c <- 1:
      }
    }

    select {}  // block forever

## return语句

在函数中return语句表示结束函数的执行，可能还包含返回值，
如果函数F中还包含延时函数，会在函数F返回调用者之前调用。

    ReturnStmt = "return" [ ExpressionList ] .

如果函数没有返回值列表，那return不能指明返回值

带返回值的函数，返回有3种情况：

- return 显示带返回值列表，此时每个表达式都是单值，且可赋值给对应的返回值类型。

    func simpleF() int {
      return 2
    }

    func complexF1() (re float64, im float64) {
      return -7.0, -4.0
    }

- return 返回一个函数A调用，这个函数A有返回值列表，列表中最少有一个返回值，
这样函数A返回的值会先存在临时变量中，临时变量的类型和函数F的返回值类型一致

    func complexF2() (re float64, im float64) {
      return complexF1()
    }

- return 空，此时函数F的返回值列表都是有名字的

    func complexF3() (re float64, im float64) {
      re = 7.0
      im = 4.0
      return
    }

    func (devnull) Write(p []byte) (n int, _ error) {
      n = len(p)
      return
    }

不同的实现可能不同,所以在开发时尽量避免以下情况：

    func f(n int) (res int, err error) {
      if _, err := f(n-1); err != nil {
        return  // invalid return statement: err is shadowed
      }
      return
    }

## break语句

在for/switch/select语句中结束执行

    BreakStmt = "break" [ Label ] .

可以和标签语句配合，指明下一步执行点

带标签，break语句必须是在封闭的for/switch/select语句中。
标签指明了跳出哪一层for/switch/select.

    OuterLoop:
      for i = 0; i < n; i++ {
        for j = 0; j < m; j++ {
          switch a[i][j] {
          case nil:
            state = Error
            break OuterLoop
          case item:
            state = Found
            break OuterLoop
          }
        }
      }

## continue语句

在for语句中，continue会跳到最近一次迭代去执行(直接跳到for循环的后置表达式)。

    ContinueStmt = "continue" [ Label ] .

也支持标签。标签指明了跳到哪层for去执行后置表达式。
