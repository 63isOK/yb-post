# 表达式

一个表达式是指对一个值的计算(利用运算符或函数来计算)

## 操作数

表达式中的基本值叫做操作数。
操作数可能是一个文字，可能是非空标识是常量/变量/函数/带括号表达式,
而且还可能是其他包的,eg: mylib.Abc

空标识符在赋值语句的左边时，可以认为是一个操作数

    Operand     = Literal | OperandName | "(" Expression ")" .
    Literal     = BasicLit | CompositeLit | FunctionLit .
    BasicLit    = int_lit | float_lit | imaginary_lit | rune_lit | string_lit .
    OperandName = identifier | QualifiedIdent.

下面就是要了解，ebnf中定义的操作数有哪些

分析：看这个分析之前，先看完下面3节的内容(直到主要表达式)

1. 表达式中是有操作符和操作数，有哪些可以作为表达式就是这节要表明的内容
2. 如何使用操作数来组成表达式，如何使用表达式..., 这些跳到"主要表达式"章节

- 操作数的ebnf中，将操作数主要分成了3类：
  - 字面量
    - 基本字面量
      - int
      - float
      - 复数
      - rune
      - string
    - 复合字面量
      - 就是接下来讲解的，类型{...}
    - 函数字面量
      - 这个比较简单，可以赋值给函数变量或直接调用
  - 操作数名
    - 要么是标识符，要么是其他包的标识符(包名.标识符名的调用方式)
  - (表达式)
    - 这个是嵌套写法，只是用()来取值，这里不讨论
- 所以所操作数的种类就这些，接下来就是利用操作符将这些操作数组合成表达式
- 看了下面3节的内容的，直接跳到主要表达式即可

## 包级标识符

包级标识符是有包名前缀的标识符。包名和标识符都是非空的

    QualifiedIdent = PackageName "." identifier .

要在其他包中访问这个包级标识符，那么这个标识符必须导出。

也就是说标识符必须在包块中声明，并导出。

## 复合字面量

符合字面量是用来构建结构体/数组/切片/map的值或在她们每次计算是创建一个新值。

她包含了类型，类型后面跟{},{}里放的是元素列表，每个元素前面可能还有相关的key。

    CompositeLit  = LiteralType LiteralValue .
    LiteralType   = StructType | ArrayType | "[" "..." "]" ElementType |
                    SliceType | MapType | TypeName .
    LiteralValue  = "{" [ ElementList [ "," ] ] "}" .
    ElementList   = KeyedElement { "," KeyedElement } .
    KeyedElement  = [ Key ":" ] Element .
    Key           = FieldName | Expression | LiteralValue .
    FieldName     = identifier .
    Element       = Expression | LiteralValue .

从上面的ebnf上看，复合字面量的底层结构只支持struct/array/slice/map.
上面的key，会解释成"结构体字面量中的字段名"/"数组切片的索引"/"map的key"。
对于map复合字面量，所有的元素都必须有一个key。一个key对应多个元素是错误的。

    i := [10]int{0:10,5:50,9:100}
    m := map[string]string{"abc":"123","bb":"aa"}
    s := struct{a,b int}{"a":10,"b":100}

对于结构体，还有以下规则：

- key必须是一个结构体中已声明的字段名
- 如果符合字面量中不包含key，那么元素列表需要和结构体字段声明顺序保持一致
- key要么有，要么都没有
- 并不是所有的字段都需要指定，可用key指定一部分，剩下的会被初始化成零值
- 给其他包中结构体的非导出字段用key指定值，是错误的

对于数组和切片，还有以下规则：

- 每个元素都可以用key来标识数组中的索引
- 如果有key作为索引，那么key就是非负整数，int类型
- 如果某个元素没有key，那自动用上一个元素的key + 1
- 如果第一个元素没有key，自动认为其索引是0

使用上还是和结构体的规则有不一样的，结构体的key要么全没有要么全有。

对复合字面量取址，会创建一个唯一的变量，并用符合字面量来初始化这个变量，
返回的就是这个变量的地址。

切片/map类型的零值和这些类型初始化并是空的(没有元素)是不一样的。
slice/map的零值是nil，slice/map初始化但没有元素，她不是nil。
所以，从一个空的slice/map中取地址，和new申请一个slice/map获得的地址是不一样的，
因为一个初始化了，一个虽然分配了内存地址，但没有初始化并赋零值(值为nil，长度0)。

    p1 := &[]int{}    // p1 points to an initialized,
                      // empty slice with value []int{} and length 0
    p2 := new([]int)  // p2 points to an uninitialized slice
                      // with value nil and length 0

数组复合字面量的长度，就是符合字面量中指定的长度，
有可能长度是10,但符合字面量只指定了3个，那么剩下7个都会被初始化为对应的零值。
在数组复合字面量中，如果key超过了索引范围，会报错

数组复合字面量中的...表示数组的长度是最大索引 + 1,
这种写法是一种便利.

    buffer := [10]string{}             // len(buffer) == 10
    intSet := [6]int{1, 2, 3, 5}       // len(intSet) == 6
    days := [...]string{"Sat", "Sun"}  // len(days) == 2

切片符合字面量，是基于底层的数组符合字面量的，因此，
切片的长度和容量是最大索引 + 1，下面两句定义一个切片复合字面都是一个意思：

    []T{x1, x2, … xn} 复合字面量定义
    tmp := [n]T{x1, x2, … xn}
    tmp[0 : n]  先定义数组复合字面量，再定义切片

在复合字面量中，有一种特殊情况，eg：array/slice/map类型的value，map的key，
都可能是复合字面量，那里面复合字面的类型是可以省略的
(前提是里面复合字面量的类型和外面字面量元素类型是一致的，废话，不一致就会报错)，
所以，基本上复合字面量里面嵌入其他符合字面量，里面符合字面量的类型是可以省的。

    [...]Point{{1.5, -3.5}, {0, 0}}
    // same as [...]Point{Point{1.5, -3.5}, Point{0, 0}}

    [][]int{{1, 2, 3}, {4, 5}}
    // same as [][]int{[]int{1, 2, 3}, []int{4, 5}}

    [][]Point{{{0, 1}, {1, 2}}}
    // same as [][]Point{[]Point{Point{0, 1}, Point{1, 2}}}

    map[string]Point{"orig": {0, 0}}
    // same as map[string]Point{"orig": Point{0, 0}}

    map[Point]string{{0, 0}: "orig"}
    // same as map[Point]string{Point{0, 0}: "orig"}
    基础类型(内置类型)是不需要再显示写类型的，因为没必要

和上面的规则同理，如果value或key是复合字面量的取址，那&T也会被省掉。

    type PPoint *Point
    [2]*Point{{1.5, -3.5}, {}}
    // same as [2]*Point{&Point{1.5, -3.5}, &Point{}}

    [2]PPoint{{1.5, -3.5}, {}}
    // same as [2]PPoint{PPoint(&Point{1.5, -3.5}), PPoint(&Point{})}

语法规则的冲突：复合字面量可以出现在表达式中，if/for/switch都可以带表达式，
冲突点在于复合字面量是带{},而if/for/switch中会将第一个{}解析为语句块。
解决问题的方法是在if/for/switch中，将符合字面量用()包裹。

    if x == (T{a,b,c}[i]) { … }
    if (x == T{a,b,c}[i]) { … }

## 函数字面量

函数字面量表示的一个匿名函数

    FunctionLit = "func" Signature FunctionBody .

函数字面量是可以直接赋值给函数变量的

    f := func(x, y int) int { return x + y }

或者直接调用

    func(ch chan int) { ch <- ACK }(replyChan)

函数字面量是闭包：函数字面量会引用周围变量。

## 主要的表达式

看这节之前，先回头看看操作数中最后的那部分。

下面主要分析，如何将操作数和操作符组合起来，合成表达式。
操作数相关的定义看上面3节，操作符，就是一元/二元操作符。

没有3元操作符(c++中的唯一一个3元操作符是?:)的原因子啊faq中也提到了：
没必要为一个问题提供多种实现方式。

    PrimaryExpr =
      Operand |
      Conversion |
      MethodExpr |
      PrimaryExpr Selector |
      PrimaryExpr Index |
      PrimaryExpr Slice |
      PrimaryExpr TypeAssertion |
      PrimaryExpr Arguments .

    Selector       = "." identifier .
    Index          = "[" Expression "]" .
    Slice          = "[" [ Expression ] ":" [ Expression ] "]" |
                     "[" [ Expression ] ":" Expression ":" Expression "]" .
    TypeAssertion  = "." "(" Type ")" .
    Arguments      = "(" [ ( ExpressionList | Type [ "," ExpressionList ] )
                     [ "..." ] [ "," ] ] ")" .

和操作数一样，都是一个ebnf丢出来，再用几节的内容来解释。先看下一节，再回头总结。

看完了可变参的解析，现在回头来看这个主要的表达式。

分析：

- Go中主要的表达式有操作数/方法表达式/选择器表达式/索引表达式/切片表达式/类型断言表达式/参数表达式/转换(这个后面再提)
- 这上面的表达式主要是Go定义的，表达式中除了这块，还有跟操作符相关的表达式

## Selectors 选择器

上面也看到了ebnf的定义： Selector       = "." identifier .

我们用x来表示主表达式，用x.f来表示选择器表达式，x不是包名。
f就是我们所称为的选择器，可以是字段/方法，不能是空白标识符。
选择器表达式(x.f)的类型就是f的类型。

如果x是包名，那f就不是选择器表达式了，而是包级的标识符。

选择器f，可能是一个类型T的字段或方法，也可能是T中嵌套字段的字段或方法。
f的深度指找到f，所经历嵌套的层数。T中的字段和方法，她们的深度是0,
之后嵌套一次，深度加1.

选择器还有以下规则：

1. T不是指针或接口，T或\*T类型的值x，x.f表示最小深度的字段或方法,如果同一深度，有两个f，那选择器表达式就是非法的
2. T是接口，x是T的值，x.f表示动态值x的实际类型的f方法，如果T的方法集中没有叫f的，那选择器语句就是非法的
3. 一个例外(这个是Go语言中的一个简写)，x是指针，(\*x.f)是正常的，此时f是一个字段，而不是一个方法，`此时x.f是(\*x.f)的简写`
4. 其他任何情况，x.f都是非法的
5. x是接口类型，值是nil，调用x.f会引发一个运行时异常

    type T0 struct {
      x int
    }

    func (*T0) M0()

    type T1 struct {
      y int
    }

    func (T1) M1()

    type T2 struct {
      z int
      T1
      *T0
    }

    func (*T2) M2()

    type Q *T2

    // 下面是假设值
    var t T2     // with t.T0 != nil
    var p *T2    // with p != nil and (*p).T0 != nil
    var q Q = p

    t.z          // t.z
    t.y          // t.T1.y
    t.x          // (*t.T0).x

    p.z          // (*p).z
    p.y          // (*p).T1.y
    p.x          // (*(*p).T0).x

    q.x          // (*(*q).T0).x        (*q).x is a valid field selector

    p.M0()       // ((*p).T0).M0()      M0 expects *T0 receiver
    p.M1()       // ((*p).T1).M1()      M1 expects T1 receiver
    p.M2()       // p.M2()              M2 expects *T2 receiver
    t.M2()       // (&t).M2()           M2 expects *T2 receiver, see section on Calls
    q.M0()       // (*q).M0 is valid but not a field selector  选择器是一个方法
    // 关于指针.字段  是可以简写的的

分析： T2结构体中 T0和T1都是作为嵌入字段，带星号和不带星号的区别是：
方法集不一样，带星号继承了所有方法集，不带信号就没有指针接收者的方法集。
同时，除了方法集还有值的区别，一个是值，一个是指针。引用的时候，是可以直接用的。

此处出现了spec中关于指针和值的第一个简写：
`x是指针，x.f 选择表达式中，当f是字段不是方式时，(\*x.f)可以简写为x.f`

## 方法表达式

`接下来两节读起来比较晦涩，可以跳到方法表达式和方法值一节，
看完之后，再回头来单独看这两节。`

T是类型，M是T的方法集，T.M就是一个函数，和普通函数没什么区别，
只需将方法的接收者参数作为第一个入参即可。

    MethodExpr    = ReceiverType "." MethodName .
    ReceiverType  = Type .

有以下类型：

    type T struct {
      a int
    }
    func (tv  T) Mv(a int) int         { return 0 }  // value receiver
    func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

    var t T

T.Mv表达式，会生成一个如下函数：
func(tv T, a int) int，所以下面几种调用都是一致的：

    t.Mv(7)
    T.Mv(t, 7)
    (T).Mv(t, 7)
    f1 := T.Mv; f1(t, 7)
    f2 := (T).Mv; f2(t, 7)

(\*T).Mp表达式，同样会生成一个如下的函数：
func(tp \*T, f float32) float32

对于接收者是值的方法，会派生出一个指针接收者的方法，
(\*T).Mv表达式，同样会生成一个如下函数：
func(tv \*T, a int) int

这类函数，是间接通过接收者创建一个值并传递给底层方法，
在这个底层方法中，并不会修改原始接收者，即使她把地址传进来了。

一个指针接收者的方法，派生出的值接收函数，是非法的，
因为指针接收者的方法不在值接收者的方法集中。

从方法衍生的函数，使用的也是函数调用语法。接收者作为第一个入参。
所以f := T.Mv 之后，f的调用是 f(t,7) 而不是t.f(7)。这种情况，
f是从方法转换过来的函数，而不是方法。

接口类型的方法生成出函数，也是合法的，此时函数的第一个入参是接口类型的接收者。

## 方法值

在表达式x中，如果有个静态类型T，她的方法集叫M，
x.M叫方法值。

方法值x.M就是一个函数值，这个函数调用的参数值和x.M的方法调用参数一致。

这个T可以是接口类型，也可以不是

用上一节的例子，再分析一下：

t.Mv 会生成一个函数值的类型： func(int) int

    下面两种方式也是一样的
    t.Mv(7)
    f := t.Mv; f(7)

tp.Mp 会生成一个函数值的类型： func(float32) float32

## 方法表达式和方法值

T是类型，t是T的实例，M是T的方法，出现这种情况的因为看问题的方式有些不同。

t.M(arg) 都可以用新的函数来表示:

- M1 = T.M,  M1(t, arg),好处是M1和类型T无关,每次调用都作为参数传入t
- M2 = t.M， M2(arg)，好处是每次调用无需在关心调用者，t已经和M2绑定了

前一种被称为是`方法表达式`，后一种被称为`方法值`,这两种写法继续深入都有些细分。

先分析方法表达式:

- 这些搞法就是Go扩展的，用ebnf定义的
- 利用选择器语法，T.M会返回一个M1函数

下面继续分析一下这个M1函数，尤其是M方法的接收者是值和指针的时候。

`值接收者有值的方法级，指针接收者有指针的方法集 + 值的方法级。`

从上面的规则可以看出，值接收者的方法集是不包含指针方法集的。
所以只有3种情况：

- 值接收者 - 值方法集
- 指针接收者 - 指针方法集
- 指针接收者 - 值方法集

下面就看看在这3种情况下，M1函数的转换方式

    // T test sturct
    type T struct {
      a int
    }

    // Mv value receiver
    func (t T) Mv(a int) int {
      fmt.Printf("Mv, %d\n", a)
      return 0
    }

    // Mp pointer receiver
    func (t *T) Mp(f float32) float32 {
      fmt.Printf("Mp, %f\n", f)
      return 1
    }

    func test() {
      var t T
      fvv := T.Mv       // M1: func(t T, a int) int
      fpp := (*T).Mp    // M1: func(t *T, f float32) float32
      fpv := (*T).Mv    // M1: func(t *T, a int) int
                        // 可以看出这3种M1的转换都比较简单
                        // 而且这种转换都是Go的语法糖，是Go提供的

      fvv(t, 1)
      fpp(&t, 1.2)
      fpv(&t, 2)

      fmt.Println("=========")

      fv := t.Mv        // M2: func(int) int
      fp := (&t).Mp     // M2: func(float32) float32
                        // 方法值的方式，将实例t和方法绑定在一起
                        // 所以在调用时只是简化了接收者，其他的并没有变

      fv(3)
      fp(2.2)
      fmt.Println("=========")
    }

    Output:
    Mv, 1
    Mp, 1.200000
    Mv, 2
    =========
    Mv, 3
    Mp, 2.200000
    =========

方法表达式理解之后，是比较简单的，更复杂的是引入接口等。

关于方法值，还有以下几点：

    func makeT() T {
      return T{}
    }

    fv2 := makeT().Mv
    fp2 := makeT().Mp   // cannot take the address of makeT()
    fv2(3)
    fp2(3.2)
    fmt.Println("=========")

    解决方法就是用函数放回一个可访问的地址来调用Mp

    func makepT() *T {
      return &T{}
    }

    至于函数返回值不能取地址的问题，可以看后面的内存模型的规则。

方法值已经将调用者和调用方法绑定了，所以Go会自动检测实际调用者：

x是一个值，x.Mp会自动转换成(&x).Mp, `这是方法调用的自动检测`  
x是一个指针，x.Mv会自动转换成(\*x).Mv, `这是选择器表达式的自动检测`

方法值的套用，同样适用于T是接口的情况。

## 索引表达式

索引表达式的形式如下：

    a[x]

索引表达式是主要表达式的一种。

索引一般用于数组/数组指针/切片/string/map，`a[x]`中的x要么是索引要么是map的key。

索引有以下规则

- 非map类型
  - x是int类型，或是无类型常量
  - 常量索引必须是非负的，可以用int来表示
  - 如果常量索引是无类型的，那么会自动认为是int类型
  - [0-len(a))半开区间是x的范围，否则就是超出了范围
- array
  - 常量索引必须在范围内
  - 如果x超出范围，会在运行时报异常
  - `a[x]`表示索引x处的元素，元素类型就是数组指定的类型
- 数组指针

      a[x] 是(*a)[x]的简写
      这个简写符合 `选择器语法的简写`，指针变量，可以将前面的星号简化掉

- 切片
  - x超出返回，会报运行时异常
  - `a[x]`表示索引x处的元素，元素类型就是切片指定的类型
- string
  - 如果字符串是一个常量，那么索引也是一个在范围内的常量
  - x超出返回，会报运行时异常
  - a[x]表示索引x处的元素,一个byte值，元素类型就是byte
  - a[x] 是不能赋值的
- map
  - x的类型必须可赋值给map的key类型
  - 如果map中包含key为x的元素，a[x]的类型就是map的value类型
  - 如果map中不包含key为x的元素，或map为空，map为nil，a[x]会返回零值
- 其他情况下，a[x]是非法语句

在map中，索引表达式可用于赋值或初始化，map[int]int:

    m[1] = 1
    v, ok := m[2]

如果map未初始化(为nil)，此时赋值会导致运行时异常

## 切片表达式

切片表达式可用于string/array/pointer to array/slice，
从而构建一个新的子字符串或子切片。

切片表达式有两种形式：简单的[a:b];带容量的[a:b:c]

### 简单切片表达式

    a[low:high]

这种写法会构造出一个新的子串或切片，游标指明的是一个半开区间[low:high),
生成的新对象长度是high - low。

为了书写方便，游标都是可以省略的：

    a[2:]  // same as a[2 : len(a)]
    a[:3]  // same as a[0 : 3]
    a[:]   // same as a[0 : len(a)]

    根据"选择器表达式规则"，对于指针数组，a[1:3]是(*a)[1:3]的简写

对于数组和字符串，游标范围是0-len(),
对于切片，游标范围是0-cap(),而不是0-len(),因为切片是可变长度

除了无类型字符串，其他使用切片表达式生成的新对象，
其类型和切片表示式的操作数类型是一致的;
对无类型字符串进行切片，得到的是字符串类型。

对nil的slice进行切片，得到的也是nil slice;其他情况，新切片和操作数共享底层数组。
当然不是一直共享，发生容量变化，会导致重新申请数组的。

### 完整的切片表达式

    a[low:high:max]

这种写法不适用于字符串。

相对于简单的切片表达式a[1:2],新增的max表示容量信息，具体容量是(max-low)

其他特性和简单切片表达式的规则类似。

## 类型断言表达式

    x.(T)

x是接口类型，T是具体类型，这就是类型断言。

类型断言，判断x是不是nil，还判断了x里存储的值是不是T类型的。

具体分析一下，如果T不是接口类型，类型断言就是看x的动态类型和T是不是同一个类型。
这种情况下，如果T实现了x的接口，断言就是成功，其他情况，断言失败。
如果T是一个接口，就是判断动态类型x是否实现了接口T。

如果类型断言成立，表达式返回的值就是x的值，类型是T。类型断言是否成立，
可以通过第二个返回值来判断。如果类型断言不成立，会报一个运行时异常。

    var x interface{} = 7          // x has dynamic type int and value 7
    i := x.(int)                   // i has type int and value 7
                                   // int不是接口，就判断7和int是不是相同类型

    type I interface { m() }

    func f(y I) {
      s := y.(string)    // illegal:
                         // string does not implement I (missing method m)
                         // string没有实现I接口，所以会出错
      r := y.(io.Reader) // r has type io.Reader and the dynamic type of y
                         // must implement both I and io.Reader
                         // io.Reader是接口，如果y还实现了io.Reader,断言成立
      …
    }

个人理解：
这个类型断言表达式，含有两种意义：

- 接口变量中存的值是不是某个具体的类型
- 接口变量中存的值是不是还实现了某个其他接口

如果断言成立，表达式返回新的类型，值还是那个值。

类型断言表达式一般用于赋值或初始化

    v, ok = x.(T)           // 赋值
    v, ok := x.(T)          // 初始化，至少有一个变量是新申请的
    var v, ok = x.(T)       // 初始化
    var v, ok T1 = x.(T)    // T1表示什么意思？ 只能看后面的spec有没有解释

上面的ok，是一个无类型的bool值，断言成立就是true，失败就是false。
失败时，v就是T类型的零值。多返回值的表达式不会产生运行时异常

## 调用

这里的调用，是指函数调用表达式

  f(arg)

对于调用参数，除了一种特殊情况外，其他情况下，参数都必须是单值表达式，
且可赋值给参数类型的，这些参数会在函数调用执行之前先计算。
函数表达式的返回值就是函数的返回值

方法的调用和函数的调用基本类似，就是多了一个接收者参数。

在函数调用中，函数值和参数按一定的顺序计算，这个顺序后面会详细提到。
计算完之后，就会将参数作为入参传给函数，函数执行。函数的返回值以值返回。

调用一个nil 函数，会报一个运行时异常

特殊情况，一个函数或方法A的返回值(无论是数量还是顺序)，
正好是另一个函数或方法B的入参，那么B(A(A的参数))是合法的。
这就要求B的入参正好是A的返回值，且A至少有一个返回值。
如果B最后是一个可变参(...),依然是ok的。

如果x的类型的方法集中包含m，那么x.m()调用就是合法的。
如果x是可寻址的(可以通过&来找地址)，且m在\*(x的类型)的方法集中，
那么x.m()就是(&x).m的简写，这个是`调用简写`

到目前为止，学习了两种简写：

- `选择器表达式简写`，x是指针，(\*x).f可以简写为x.f
- `调用表达式简写`，m是\*T的方法集，t的类型是T，(&t).m可简写为t.m

## 可变参

函数或方法的最后一个参数可以是可变参，用...Type来表示

可变参表示实际过程中可以传递0个参数，或多个参数。
0个参数表示nil;否则将参数作为[]T进行传递。

    func Greeting(prefix string, who ...string)
    Greeting("nobody")  
                        // who 是nil
    Greeting("hello:", "Joe", "Anna", "Eileen")
                        // who是[]string{"Joe", "Anna", "Eileen"}

调用一个可变参函数或方法时，是这样的, 变量后跟...

    s := []string{"James", "Jasmine"}
    Greeting("goodbye:", s...)   // s...表示所有的可变参


    func variadic(a ...int) {
      if a == nil {
        fmt.Println("a == nil")
      } else {
        fmt.Printf("here: %v\n", a)
      }
    }

    func test2() {
      variadic()
      variadic(1)
      i := []int{1, 2, 3}
      variadic(i...)   // 在这个调用中，可变参[]T和i共享底层数组

      fmt.Println("=========")
    }
    Output:
    a == nil
    here: [1]
    here: [1 2 3]
    =========
    // 这个测试例子也印证了上面的话

到目前为止，我们已经看完了主要表达式的所有spec，跳到主要表达式

## 操作符

操作符就是将操作数组合成一个新的表达式。

    Expression = UnaryExpr | Expression binary_op Expression .
    UnaryExpr  = PrimaryExpr | unary_op UnaryExpr .

    binary_op  = "||" | "&&" | rel_op | add_op | mul_op .
    rel_op     = "==" | "!=" | "<" | "<=" | ">" | ">=" .
    add_op     = "+" | "-" | "|" | "^" .
    mul_op     = "*" | "/" | "%" | "<<" | ">>" | "&" | "&^" .

    unary_op   = "+" | "-" | "!" | "^" | "*" | "&" | "<-" .

从ebnf中可看出：

- 表达式分一元表达式和二元表达式
- 一元表达式分主要表达式，或 一元操作符 + 主要表达式 组成
- 二元表达式, 是二元操作符和表达式组合而成

现在已经看了主要表达式的spec，只剩下一元操作符和二元操作符没分析，
分析完之后，就可以看清表达式具体包含了哪些东西

从上面的ebnf中，还可以看出：

- 一元表达式包括：
  - +-表示征服
  - ! 非
  - ^ 取反
  - \* & 地址操作符
  - <- 通道方向
- 二元表达式
  - 逻辑与或
  - 逻辑比较  大于等于小于等
  - 加法操作 加减 或 异或
  - 乘积操作 乘除 取余 位移 按位与 按位置零

比较，何处不在。对于二元操作符，两个操作数要么类型是一致的，
要么一边是位移操作或无类型常量也行。常量表达式单独分析。

除了位移操作，如果一个操作数是无类型常量，另一个不是，
那么这个常量会隐式地转换成另一个操作数的类型。

位移操作的右操作数，要是一个int类型，或者是无类型常量，但可以转换成uint。
如果位移操作的左操作数是一个无类型的常量，会优先将表达式类型隐式转换成
"表达式外部，左边的操作数的类型"，eg：var i int32 = 1 << 10,
会优先将位移表达式隐式转换成int32,而不是int。

操作符的优先级：

- 一元操作符的优先级最高
- ++ -- 是语句，不是表达式，所以她们在Go中不是操作符，优先级比操作符低
- 二元操作符有5个优先级等级
  - 乘积操作符优先级最高
  - 之后是加法操作符/逻辑比较/逻辑与/逻辑或
- 同一级别的优先级，按从左到右的结合规则，先出现的先结合

## 算术操作

对于整数/浮点/复数类型，加减乘除都是适用的，+ 还适用于string，
按位逻辑操作和位移操作只适合int。

    +    sum                    integers, floats, complex values, strings
    -    difference             integers, floats, complex values
    *    product                integers, floats, complex values
    /    quotient               integers, floats, complex values
    %    remainder              integers

    &    bitwise AND            integers
    |    bitwise OR             integers
    ^    bitwise XOR            integers
    &^   bit clear (AND NOT)    integers

    <<   left shift             integer << unsigned integer
    >>   right shift            integer >> unsigned integer

除可能会发生截断

可能会发生整数溢出 overflow

整数溢出会导致报运行时异常

字符串可以使用 "+" "+="

    s := "hi" + string(c)
    s += " and good bye"

## 比较操作符

比较操作符是一个二元操作符，左右各加一个操作数，就成了表达式，
这个表达式的结果，会产生一个无类型的布尔值

比较的前提是第二个操作数可以赋值改第一个操作符，反之亦然

等于/不等于是无序比较操作符，大于小于等都是有序比较操作符

比较有以下规则：

- boolean值可以比较
- int可以比较
- 浮点可以比较
- 复数可以比较，只有实数和虚数都相等，复数才是相等的
- 字符串可以比较，按字节比较
- 指针可以比较，指向同一个变量或都为nil。指向不同的零值结果可能相等可能不相等
- chanel可以比较，信道值都指向同一个make出来的信道，或都为nil，叫相等
- 接口值可以比较，都为nil，或她们的动态类型或值都相同，叫相等
- 非接口类型的x，和接口类型可以比较，只要x的类型实现了接口，如果接口值的动态类型是具体类型，且值也是一样的，叫相等
- 结构体可以比较，只要每个字段都是可比较的。只要两个结构体的非空字段是相等的，那这两个结构体就是相等的
- 数组是可以比较的，只要她们的元素类型是可比较的，只要两个数组的元素是一样的，就是相等

两接口比较，如果值是无法比较的，那么会报运行时异常，除了直接比较两个接口，
还会出现在接口数组，或包含接口的结构体。

谈了这么多，slice/map/function是没法比较的，这3个唯一能比较的是nil。
除了这3个，还有指针/channel/接口都可以和nil做比较

## 逻辑操作符

逻辑 与 或 非

&& || !

优先级是 一元操作符 与  或

## 地址操作符

t T

取地址是&t，此时t必须是可寻址的：

- 要么是变量
- 要么是间接的指针
- 要么是slice的索引操作
- 要么是结构体的选择器表达式中的字段
- 要么是数组的索引操作

除了寻址，t也可以是用(复合字面量)，用括号包围的复合字面量。

如果计算t会导致运行时异常，那么&t也会报运行时异常

    &x
    &a[f(2)]
    &Point{2, 3}
    *p
    *pf(x)

    var x *int = nil
    *x   // causes a run-time panic
    &*x  // causes a run-time panic

## 接收操作符

channel类型的操作数ch，接收操作<- ch的值就是ch信道里的值。
其中的<- 就是接收操作符。接收操作的前提是保证方向是允许的。

接收操作的类型信道元素的类型。这个表达式会阻塞，直到值可用。
从一个nil信道执行接收操作，会永远阻塞。
从一个已关闭的信道执行接收操作，会立马执行，再接收完所有的元素后，
会接收一个元素的的零值。

    v1 := <-ch
    v2 = <-ch
    f(<-ch)
    <-strobe  // wait until clock pulse and discard received value

接收表达式一般会用在赋值或初始化中

    x, ok = <-ch        // 赋值
    x, ok := <-ch       // 初始化
    var x, ok = <-ch    // 初始化
    var x, ok T = <-ch  // 这个T 是啥意思? 和类型断言的例子一样，后面分析

ok是无类型的布尔值，用于表明通信是否成功，
true表示已正确取到了一个对面发送的元素，
false表示信道已经被关闭，此时x会是元素的零值

## 类型转换

类型转换是说表达式的类型发生了转换。转换可以在源码中已字面量的方式出现，
也可以在表达式的上下文中隐式出现。

显示的类型转换格式如下：

    Conversion = Type "(" Expression [ "," ] ")" .

如果前面这个类型以 \* 或 <- 开头，或以func关键字开头且没有返回值，
这个类型必须用()包括，不然会出现二义性。

    *Point(p)        // same as *(Point(p))
    (*Point)(p)      // p is converted to *Point
    <-chan int(c)    // same as <-(chan int(c))
    (<-chan int)(c)  // c is converted to <-chan int
    func()(x)        // function signature func() x
    (func())(x)      // x is converted to func()
    (func() int)(x)  // x is converted to func() int
    func() int(x)    // x is converted to func() int (unambiguous)

常量x可以转换成类型T，前提是x可以用T的值表示。
例外:整数常量是可以转换成string的，这个转换规则后面会提到。

转换一个常量，会生成一个了带类型的常量

    uint(iota)               // iota value of type uint
    float32(2.718281828)     // 2.718281828 of type float32
    complex128(1)            // 1.0 + 0.0i of type complex128
    float32(0.49999999)      // 0.5 of type float32
    float64(-1e-1000)        // 0.0 of type float64
    string('x')              // "x" of type string
    string(0x266c)           // "♬" of type string
    MyString("foo" + "bar")  // "foobar" of type MyString
    string([]byte{'a'})      // not a constant: []byte{'a'} is not a constant
    (*int)(nil)              // not a constant: nil is not a constant,
                             // *int is not a boolean, numeric, or string type
    int(1.2)                 // illegal: 1.2 cannot be represented as an int
    string(65.0)             // illegal: 65.0 is not an integer constant

非常量x，转换成类型T，只要满足以下一条即可

- x可赋值给T的变量
- 忽略结构体的tag，x的类型和T的有相同的底层类型
- 忽略结构体的tag，x的类型和T都是指针类型，且不是自定义类型，底层类型是一样的
- x的类型和T都是整数类型或浮点类型
- x是整数，或[]byte,或rune，T是string类型
- x是字符串，T是[]byte或rune

转换的时候是不需要考虑结构体的tag的。

转换在运行时会有花费。
`所有的类型转换，只更改变类型，不改变值`

指针和整数相互转换的机制，在Go中是没有的，unsafe包提供了有限的转换能力。

数值之间的转换：

- 整数转换，有符号转无符号，会隐式扩展成无限精度?,会进行截断，不会溢出
- 浮点转整数，小数会被丢弃
- 整数或浮点转浮点，复数转另一个复数类型，结果会四舍五入。

字符串类型string的转换：

- 有符号整数或无符号整数转string，会转换成对应的utf-8的字符串，超出的转成"\uFFFD"

    string('a')       // "a"
    string(-1)        // "\ufffd" == "\xef\xbf\xbd"
    string(0xf8)      // "\u00f8" == "ø" == "\xc3\xb8"
    type MyString string
    MyString(0x65e5)  // "\u65e5" == "日" == "\xe6\x97\xa5"

- `[]byte转string`，按顺序将元素排列，就是字符串

    string([]byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'})   // "hellø"
    string([]byte{})                                     // ""
    string([]byte(nil))                                  // ""

    type MyBytes []byte
    string(MyBytes{'h', 'e', 'l', 'l', '\xc3', '\xb8'})  // "hellø"

- []rune转string

    string([]rune{0x767d, 0x9d6c, 0x7fd4})   // "\u767d\u9d6c\u7fd4" == "白鵬翔"
    string([]rune{})                         // ""
    string([]rune(nil))                      // ""

    type MyRunes []rune
    string(MyRunes{0x767d, 0x9d6c, 0x7fd4})  // "\u767d\u9d6c\u7fd4" == "白鵬翔"

- string 转[]byte

    []byte("hellø")   // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
    []byte("")        // []byte{}

    MyBytes("hellø")  // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}

- string转 []rune

    []rune(MyString("白鵬翔"))  // []rune{0x767d, 0x9d6c, 0x7fd4}
    []rune("")                 // []rune{}

    MyRunes("白鵬翔")           // []rune{0x767d, 0x9d6c, 0x7fd4}

## 常量表达式

常量表达式是指操作数都是常量，这样在编译期就可以确定值。

好处当然是效率高。

一般无类型的bool/数值/字符串常量，常作为常量表达式的操作数。

常量的比较，会生成一个无类型的bool常量，
如果位移表达式的左操作数是无类型常量，那表达式的结果就是整数常量;
否则，结果就是常量，类型和左操作数的类型一致(范围还是一个整形eg：int32 int64等)

对无类型常量的其他操作(非位移操作)，结果都是一个无类型的常量。
如果二元操作符，遇到两个不同类型的无类型操作数，
结果是整形/浮点/rune/复数。eg：无类型复数除以无类型整数，
结果是无类型复数。

    const a = 2 + 3.0          // a == 5.0   (untyped floating-point constant)
    const b = 15 / 4           // b == 3     (untyped integer constant)
    const c = 15 / 4.0         // c == 3.75  (untyped floating-point constant)
    const Θ float64 = 3/2      // Θ == 1.0   (type float64, 3/2 is integer division)
    const Π float64 = 3/2.     // Π == 1.5   (type float64, 3/2. is float division)
    const d = 1 << 3.0         // d == 8     (untyped integer constant)
    const e = 1.0 << 3         // e == 8     (untyped integer constant)
    const f = int32(1) << 33   // illegal    (constant 8589934592 overflows int32)
    const g = float64(2) >> 1  // illegal
                               // (float64(2) is a typed floating-point constant)
                               // 因为位移操作需要的整形类型
    const h = "foo" > "bar"    // h == true  (untyped boolean constant)
    const j = true             // j == true  (untyped boolean constant)
    const k = 'w' + 1          // k == 'x'   (untyped rune constant)
    const l = "hi"             // l == "hi"  (untyped string constant)
    const m = string(k)        // m == "x"   (type string)
    const Σ = 1 - 0.707i       //            (untyped complex constant)
    const Δ = Σ + 2.0e-4       //            (untyped complex constant)
    const Φ = iota*1i - 1/1i   //            (untyped complex constant)

内置函数complex可通过无类型参数生成一个无类型的复数

常量表达式都是可以立马计算的，计算出的值或常量表达式的精度一般都很大，
至少比内置类型的大。

    const Huge = 1 << 100         // Huge == 1267650600228229401496703205376  
                                  // (untyped integer constant)
    const Four int8 = Huge >> 98  // Four == 4
                                  // (type int8)

除法的被除数就算是无类型常量或常量表达式，也要遵循"被除数不能是0"的规则

带类型的常量，值一定要在类型的值范围内。

位异或操作符 ^ 还有些规则：对于无类型常量，如果带符号，用1来处理;
不带符号的用-1来处理。

## 计算的顺序

在包级别，初始化的依赖关系，决定了变量声明中，单个初始化表达式的计算顺序。
除此之外，表达式/赋值/返回语句/函数调用/方法调用/通信中操作数的计算，
都是按文字从左到右依次计算。

    y[f()], ok = g(h(), i()+x[j()], <-c), k()
    这是一个赋值列表
    赋值的右边都是两个函数
    执行顺序依次是 f() h() i() j() <-c g() k() 最后赋值

还是有些顺序是未指定的，此时会导致有歧义：

    a := 1
    f := func() int { a++; return a }
    x := []int{a, f()}
    // x may be [1, 2] or [2, 2]:
    //evaluation order between a and f() is not specified

    m := map[int]int{a: 1, a: 2}  
    // m may be {2: 1} or {2: 2}:
    // evaluation order between the two map assignments is not specified

    n := map[int]int{a: f()}
    // n may be {2: 3} or {3: 3}:
    // evaluation order between the key and the value is not specified

在包级别，初始化的依赖关系决定了变量声明中表达式的计算顺序，和从左到右的规则，
但并不是所有表达式都是这个顺序

    var a, b, c = f() + v(), g(), sqr(u()) + v()

    func f() int        { return c }
    func g() int        { return a }
    func sqr(x int) int { return x*x }

    // functions u and v are independent of all other variables and functions
    // 如果u和v是其他包的，执行顺序就是 u() v() f() v() g()

()会改变计算顺序

## 总结

spec中零零散散说了很多，不过这节的主题是表达式，
在操作符一节还是用ebnf来说明了，表达式分一元表达式和二元表达式

一元表达式由一元操作符(可选)和主要表达式组合而成  
二元表达式由二元操作符和两个表达式组合而成

主要表达式可细分为：

- 操作数
- 类型转换
- 方法表达式
- 选择器表达式
- 索引表达式
- 切片表达式
- 类型断言表达式
- 参数表达式

每一级都可以继续细分,具体可以查看上面的spec
