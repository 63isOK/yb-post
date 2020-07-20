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

下面先介绍终止语句，再详细介每一种语句。

## 结束语句

Go的术语是"结束语句"，也可以理解为终止语句，同一个块中，遇到这类语句，
语句下面的剩下语句就不会执行了。

在下列情况中，执行到某一点表示遇到了结束语句：

1. return goto 语句
2. 调用内置函数panic()
3. 在块中({}中)，遇到了结束语句,eg:if中的return
4. if语句中，如果有else分支，那这两个分支都是结束语句(走一个另一个就不走)
5. for语句中，没有break，没有循环条件(无限循环)，for后面的语句就不会走了,这个也可以理解为结束语句
6. switch语句中，没有break，如果有default，那每个分支都可以看成一个结束语句，走一个分支，其他分支就不会走了。包含fallthrough 语句也是同理
7. select语句中，没有break，每个分支都可以以看成一个结束语句
8. 标签语句，标记了结束语句

除了以上几种情况,其他的语句都不能看成是结束语句.

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

C++中有goto + 标签, 但没有break/continue + 标签,
Go中的break/continue也能跟标签,常用于跳出多重嵌套

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

send语句就是在channel中发送一个值.
前提有三个:信道/方向/值和信道元素类型的匹配.

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

就是++ --,在Go中不是操作符(C++中是操作符),是语句.

操作数增加或减去1 (这个1是无类型的)

操作数是可寻址的，也意味着可作为操作数出现在赋值语句
或者是map的索引表达式.

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

针对 a op= b, op就是算术操作符,和等号结合后作为单个操作符存在,
效果等同于a = a op b, 右边的a只计算一次.这里还有一些限制:
ab都只能是单值,a不能是空白标识符. a,b *= 1,2这种写法是不行的.

多值赋值操作有两种形式:

- 右值是返回多值的函数调用/channel操作/map操作/类型断言
  - x,y = f()
- 常规形式,左值右值的操作数个数一致
  - 限制,不管是左值还是右值,操作数都是单值

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
- 无类型布尔常量赋值给一个接口变量或空白标识符，这个常量会先隐式转换成bool类型

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

如果没有switch表达式，就默认是常量 true,
也就是说用true和各个case来做比较.

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

switch表达式，如果是申明和初始化一个未显式指明类型的临时变量
那么可以理解为将这个变量和各个case表达式进行==测试。

在case和default中，最后的可能会是一个fallthrough语句，
这个fallthrough语句的意思是从本分支的结尾处跳到下一个分支的开头，继续执行。
如果没有下一个分支,就结束switch语句.
此时是不用关系下个分支的比较结果。一般fallthrough出现在分支的最后面。

switch前面也能插入简单语句。

当然switch中也不应该存在两个case值一样的情况,这是一个逻辑错误,
当前编译器禁止出现一样的整数/浮点/字符串.

### type switch

这个类型switch是一个新东西，用于比较类型，而不是比较值。

新东西，写法很固定：

    switch x.(type) {
    // cases
    }

x是动态类型(接口类型)，case里是具体类型，且要实现x的接口类型。
case也能是接口类型,不过属于比较特殊的那种.

    TypeSwitchStmt  = "switch" [ SimpleStmt ";" ] TypeSwitchGuard
                      "{" { TypeCaseClause } "}" .
    TypeSwitchGuard = [ identifier ":=" ] PrimaryExpr "." "(" "type" ")" .
    TypeCaseClause  = TypeSwitchCase ":" StatementList .
    TypeSwitchCase  = "case" TypeList | "default" .
    TypeList        = Type { "," Type } .

从ebnf中可以看出，类型switch可能还包含一个短变量声明。
如果有这个声明，那短变量的作用域就覆盖所有的分支。
其效果类似于,在每个分支都声明了这个变量.

case分支，是可以出现nil的，匹配的情况是：接口变量是nil。最多只有一个nil。
如果是表达式switch,就不能再case中出现nil.

fallthrough语句不能出现在类型switch中。类型switch是可以转成if-else的。
对应的类型比较,就需要用到类型断言的方式.

## for语句

for语句是循环语句，for语句有3种形式：单条件迭代;for格式;range格式。

    ForStmt = "for" [ Condition | ForClause | RangeClause ] Block .
    Condition = Expression .

### 单条件的for语句

    for a < b {
      a *= 2
    }

最简单的格式，不满足条件就一直循环下去。
每次迭代都会检查条件。条件是可以省略的，等同于true，此时就是无限循环

### for格式的for语句

这是最能体现for精髓的语句,也是其他语言广泛使用的for格式.

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

如果条件判断也省略了,这条件判断为true.

### range格式的for语句

for range语句，只适用于迭代 array/slice/string/map/channel

    RangeClause = [ ExpressionList "=" | IdentifierList ":=" ] "range"
                  Expression .

从ebnf中可以看出，=和:=都能用

range后面的表达式称为range表达式(范围表达式)，
可以是数组/切片/字符串/map，或是数组指针，
带接收操作符的channel变量(要有权限：方向要对)。

出现在赋值语句中，左操作数要能被寻址，或是map的索引表达式,
表达式左边可理解为迭代变量,每次迭代过程中的变量.
如果range表达式是channel，最多一个迭代变量;其他的可能有多个变量.
迭代变量中可以使用空白标识符.

range表达式在迭代开始时计算一次。只有一个例外：
如果只有一个迭代变量，len(x)是个常数，则range表达式就不计算了,
因为常量在编译期就确定了,运行时不会再次计算.

如果左操作数中包含函数调用，那每次迭代都会计算一次

for range 的规则：

- 数组/数组指针/切片
  - 返回值是索引,如果只有一个迭代变量,那范围是0到len(x)-1
  - 对于nil的slice,迭代的数量是0
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

go语句是开始一个函数调用的执行，
做为一个独立的并发线程/协程来控制，
她们使用相同的地址空间。

    GoStmt = "go" Expression .

ebnf中，go后面跟的表达式必须是函数调用或方法调用，不能用()包裹。
和表达式语句的限制一样，此处，部分内置函数是不能用的(不能做无用功)。
具体哪些函数受限了，可以看"表达式语句"一节

协程goroutine中函数值和参数的计算和普通函数的计算是一样的，
不同的是：程序不会等协程调用完了再执行，而是并发执行。
程序结束,协程结束.

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

- select的所有case中，每个分支的接收操作都只计算一次
  - 计算顺序按源码顺序
  - 计算的结果是一个收发的集合
- 如果一个或多个通讯可以进行执行了，select会随机选择一个进行执行
  - 如果没有通讯准备好，那就会执行default分支(如果有一个default分支的话)
  - 如果没有default分支，又没有通讯准备好，select会阻塞，直到某个通讯准备好。
- 如果选中的不是default分支，其他被选择的分支会执行相应的通讯操作
- 如果选择的分支是一个接收语句带赋值的
  - 那表达式的左操作数会计算，并用接收操作的返回值来赋值
- select选择中的分支的语句块会被执行

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

- return 显示带返回值列表
  - 此时每个表达式都是单值
  - 且可赋值给对应的返回值类型。

    func simpleF() int {
      return 2
    }

    func complexF1() (re float64, im float64) {
      return -7.0, -4.0
    }

- return 返回一个函数A调用
  - 这个函数A有返回值列表，列表中最少有一个返回值
  - 这样函数A返回的值会先存在临时变量中，临时变量的类型和函数F的返回值类型一致

    func complexF2() (re float64, im float64) {
      return complexF1()
    }

- return 的表达式列表为空
  - 此时函数F的返回值列表都是有名字的

这种情况是返回值有名字,直接一个return就行.

    func complexF3() (re float64, im float64) {
      re = 7.0
      im = 4.0
      return
    }

    func (devnull) Write(p []byte) (n int, _ error) {
      n = len(p)
      return
    }

返回值的实际处理是先赋值为零值,再用return来设置具体的值,
当然这个设置是在延时函数之前处理的.

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

## goto语句

goto语句会将执行的控制权交给同一个函数中的其他语句

    GotoStmt = "goto" Label .

goto的后面不能有变量声明，必须是先确定所有人的作用域，再使用goto

      goto L  // BAD, 跳过了v变量的声明，会导致错误
      v := 3
    L:

goto语句和标签声明不在同一个块，会报错

    if n%2 == 1 {
      goto L1  // 错误，goto语句和标签语句不在同一个块
    }
    for n > 0 {
      f()
      n--
    L1:
      f()
      n--
    }

## fallthrough语句

    FallthroughStmt = "fallthrough" .

指明当前分支结束后直接执行下一个分支的块代码，不管下个分支是否匹配。

只能用在switch分支中，不能出现在最后一个分支

## defer语句

延时语句，调用一个函数，这个函数的调用时机推迟到函数返回时。
不管是正常的return返回，还是异常退出返回时。

    DeferStmt = "defer" Expression .

这个表达式必须是函数或方法调用，不能用()包裹(和go语句类似)。
内置函数的限制和go语句类似(也和表达式语句类似)。

每次defer函数执行时，她的函数值和参数和普通函数一样进行计算，
只是没有正真调用执行，而是在外部函数返回时执行。

如果有多个延时函数，执行顺序按filo的方式(栈式)

更加准确的描述延时函数调用时机：函数返回时，设置完返回值后，
在将控制权交个上一层的调用着之前，执行延时函数。

如果延时函数为nil，会在执行延时语句时报异常，而不是在执行延时函数时报。

延时函数如果是函数字面量(延时语句的表达式不是一个函数标识符)，
那在函数体里是可以修改父函数作用域里的值的

    lock(l)
    defer unlock(l)  // unlocking happens before surrounding function returns

    // prints 3 2 1 0 before surrounding function returns
    for i := 0; i <= 3; i++ {
      defer fmt.Print(i)
    }

    // f returns 42
    func f() (result int) {
      defer func() {
        // result is accessed after it was set to 6 by the return statement
        result *= 7
      }()
      return 6
    }
