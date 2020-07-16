# 类型

类型决定了值的范围，以及和这些值相关的操作和方法。
类型用类型名来引用，如果有类型名，就使用类型字面量：

    Type      = TypeName | TypeLit | "(" Type ")" .
    TypeName  = identifier | QualifiedIdent .
    TypeLit   = ArrayType | StructType | PointerType | FunctionType |
            InterfaceType | SliceType | MapType | ChannelType .

Go是有预定义一些类型名的，其他的就需要自定义类型(这个后面会详细说到)。

上面的TypeLit是组合类型，也就是说这些组合类型需要通过类型字面量来构造。

每个类型都有一个基础类型，可以理解为底层类型，underlying type，
如果类型是预定义的内置类型，那她们的基础类型就是她们自己。
否则基础类型就要看类型申明了。

    type (
      A1 = string
      A2 = A1
    )

    type (
      B1 string
      B2 B1
      B3 []B1
      B4 B3
    )

string, A1, A2, B1, and B2 的基础类型都是string  
`[]B1, B3, and B4 的基础类型都是[]B1`

## method sets

只要是类型，就可以有与之相关的方法集。
接口类型的方法集就是她们的接口。

非接口类型T的方法集 = 接收者是T的所有方法  
非接口类型\*T的方法集 = 接收者是T和\*T的所有方法(具体设计缘由看以前的文章)

上面这一规则，进一步适用到结构体中的字段。

除了T和\*T，其他类型都有一个空的方法集。

同一个方法集中的方法，都有一个非空且唯一的名字。当然，同名是肯定存在的，
按作用域覆盖的规则来决定名字具体指哪个方法。

## bool 类型

内置类型,值可以是ture/false(预定义的bool常量)。

## 数值 类型

表示整数/浮点数,具体的位数依赖于机器的体系结构(少数，还有一些是不依赖架构的)

下面就是不依赖系统架构的：

    uint8       the set of all unsigned  8-bit integers (0 to 255)
    uint16      the set of all unsigned 16-bit integers (0 to 65535)
    uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
    uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

    int8        the set of all signed  8-bit integers (-128 to 127)
    int16       the set of all signed 16-bit integers (-32768 to 32767)
    int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
    int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

    float32     the set of all IEEE-754 32-bit floating-point numbers
    float64     the set of all IEEE-754 64-bit floating-point numbers

    complex64   the set of all complex numbers with float32 real and imaginary parts
    complex128  the set of all complex numbers with float64 real and imaginary parts

    byte        alias for uint8
    rune        alias for int32

- 只有少数类型的位数会依赖机器: uint/int/uintptr
- 为了减少移植性问题,数值类型都设计成内置类型
- byte是uint8的别名,rune是int32的别名
- 不同数值类型的转换,需要显式表示

和系统架构(体系结构：64位系统还是32位系统)相关的数值类型：

- uint/int/uintptr 可能是32/64位

## string 类型

- 字符串
- 可能为空
- bytes的序列
- string的长度就是byte数
- 字符串是不可变的,一旦创建,字符串内容就无法改变
- 也是内置类型,类型名叫string
- string可以通过`[]`下标访问,`&str[5]是无效的`
- len(stirng)可查看string的长度，也即是字节数
- 如果是string常量，那在编译器，长度也是一个常量

string和byte是强关联的，长度可以理解为字节数。
和广义上的一个字(cjk)是很有区别的。
rune是和广义上的字相等的，rune和byte不是完全相等的。
特别是遇到中日韩字符时。

## array 类型

    ArrayType   = "[" ArrayLength "]" ElementType .
    ArrayLength = Expression .
    ElementType = Type .

- 数组
- 单一类型 固定数量 的序列
- 这里的单一类型就是元素类型，固定数量称为数组长度
- 长度也是类型的一部分
- 数量是数组类型的一部分,是一个非负整数
- len(array)也可以获取数组长度j
- 可通过`[]`下标引用具体某一个元素
- 数组是一维的,可组合成多维数组
  - `[2][3][4]int 等同于[2]([3]([4]int))`

## slice 类型

    SliceType = "[" "]" ElementType .

- 切片类型
- 是对底层数组中连续一段的描述
- 同时提供了对底层数组一段序列的访问能力
- 能访问的元素就是切片的长度
- 未初始化切片是nil
- len(slice)可以查看长度
- 支持下标访问
- 对于同一个元素，slice访问的索引可能会小于底层数组的索引
- slice可能并未覆盖到底层数组的后面一段，Go扩展了一个属性来描述：capacity
- 切片的容量 = slice当前长度 + 底层数组最后未覆盖的那一段
- 如果slice的长度达到了slice的容量，会新建一个底层数组
- cap(slice)可以获取容量
- 多个slice可能会共享同一段底层数组;一般情况下,多个数组是不会共享存储的
- 但slice的容量不够时,会自动扩展成当前容量的2倍,底层数组也换成一个新的了
- 初始化切片类型使用 `make([]T, length, capacity)`,容量是可选的
- 切片也可以从数组上生成
- `make([]int, 50, 100) 和 new([100]int) [0:50] 是一样的`
- 多维切片也是可以组合的,底层也是多维数组
- 多维切片内部的切片需要单独初始化

## struct 类型

- 可由不同类型的元素组成,一个元素叫field,每个field都有名字和类型
- 字段名可能是显式的，也可能是隐式的(嵌入的)
- 非空字段的名字必须是唯一的

ebnf:

    StructType    = "struct" "{" { FieldDecl ";" } "}" .  
    FieldDecl     = (IdentifierList Type | EmbeddedField) [ Tag ] .  
    EmbeddedField = [ "*" ] TypeName .  
    Tag           = string_lit .

    // An empty struct.  
    struct {}

    // A struct with 6 fields.
    struct {
      x, y int
      u float32
      _ float32  // padding
      A *[]int
      F func()
    }

从上面的ebnf看，嵌入式字段支持值或者指针，
另外，`struct的定义只有 struct{字段...}`,
那我们平常看到的type struct {} 是什么回事，
哦，对了，是利用type来定义别名。

- 一个filed没有显式指定字段名,就称为这个嵌入field
- 这个嵌入的field可以是类型名T,也可以是\*T(此时T不能是接口类型,或指针)
- filed名不能重复 struct{ T, \*T, \*p.T } 就是错误的

结构体里的嵌入字段

    // A struct with four embedded fields of types T1, *T2, P.T3 and *P.T4
    struct {
      T1        // field name is T1
      *T2       // field name is T2
      P.T3      // field name is T3
      *P.T4     // field name is T4
      x, y int  // field names are x and y
    }

- 嵌入field或方法,在struct是可以直接用的
- 嵌入field和普通field基本一样,差别是无法使用s.filed名方式来调用嵌入filed
- 嵌入field,指针和非指针的区别:
  - type s struct { A, \*B }
  - s的方法集包含   A作为接收者的方法集 + B/\*B作为接收者的方法集
  - \*s的方法集包含  A/\*A作为接收者的方法集 + B/\*B作为接收者的方法集
  - 一句话: 但嵌入filed T不是指针类型时, s的方法集中不包括\*T的方法集
  - 具体设计缘由看之前的文章
- field声明时,可附加一个字符串字面量标识,对应上面的tag
  - 空的标识和不添加标识等价
  - 这个标识只在反射接口和类型标识中起作用,其他时候都忽略
  - pb/json等其他映射也会用到这个标识

## 指针 类型

指针类型和其他语言的概念一样

    PointerType = "*" BaseType .
    BaseType    = Type .

## 函数 类型

- 相同参数类型,相同结果类型的所有函数集合,称为函数类型
- 未初始化值是nil

ebnf:

    FunctionType   = "func" Signature .
    Signature      = Parameters [ Result ] .
    Result         = Parameters | Type .
    Parameters     = "(" [ ParameterList [ "," ] ] ")" .
    ParameterList  = ParameterDecl { "," ParameterDecl } .
    ParameterDecl  = [ IdentifierList ] [ "..." ] Type .

从定义中可以看出，此处是一个函数类型的定义，不是函数的定义，
func () () 就是函数签名，具体某一个函数变量就会多个函数名和函数体。

- 函数类型中的参数名/返回值名,要么全部显示,要么全部忽略
  - 全部显示：参数名称和结果的名称都需要保证唯一
  - 全不显示：那每一项都需要用类型来表示。eg： func (int,int) string
- 参数列表和结果列表都需要用(),除非结果只有一个没有名字的类型，就不需要()
  - 参数肯定是需要()
  - 结果有多个需要()
  - 结果有名字，需要()
  - 只有结果是1个，且没有名字时才不需要()
- 最后一个参数类型前可带有...前缀,表示可变参数,后面可能有0个参数,也可能有多个参数

## 接口 类型

指定了方法集的称为接口。

对于接口变量,她可以存储某一类值,
只要这个值对应类型的方法集是这个接口对应方法集的超集就行.
换句话说:这个值得类型实现了这个接口所有的方法集就行.
简称:这个值的类型实现了这个接口.

- 接口类型,内含方法集
- 接口类型的变量,可存储任何实现了接口的类型的任何实例
- 未初始化的接口变量是nil

ebnf:

    InterfaceType      = "interface" "{" { MethodSpec ";" } "}" .
    MethodSpec         = MethodName Signature | InterfaceTypeName .
    MethodName         = identifier .
    InterfaceTypeName  = TypeName .

- 接口的"方法集"中，每个方法的方法名都是唯一的，且非空
- 只要某个类型实现了接口，那接口变量就能存储这个类型的实例
- 一个类型可能实现了多个接口的，eg：所有类型都是实现了interface{}
- 接口里面也可以嵌入接口
- 接口里面不能嵌入接口本身,也不能形成嵌套嵌入

## map 类型

    MapType     = "map" "[" KeyType "]" ElementType .
    KeyType     = Type .

- 无序,一个类型多个元素的分组,里面存的是kv
- 未初始化map是nil
- key需要实现==和!=, 可完成比较操作
- key不能是function/map/slice,因为这些不能比较
- key也可以是接口类型,接口变量中的值要实现比较操作,不然会报运行时异常
- `m[a]=b` 可直接使用这种方式来添加一个 map element
- map中kv对的个数就是长度，len(map)可获取
- 删除直接用delete就行

需要注意的是：除了nil的map外，map的容量是跟着kv队的数量而增长的。
空map和nil是一样的，不一样的地方是nil map不能添加kv对。

    make(map[string]int)
    make(map[string]int, 100) // 100是大小，申请内存时会参考这个值，可省略

这里就可以看出make在不同场景下的区别了：

- slice： `make([]int, 10, 20)`
  - 10是长度，20是容量，容量不指定就默认和长度一致
  - 长度是指当前切片元素个数，容量指底层数组的长度
  - 用法随场景而定
- map： `make(map[int]string, 10)`
  - 10不是长度也不是容量
  - 10 是在申请内存时，告诉运行时这个map该是多大
  - 如果忽略，那最开始申请的就比较小
  - map的长度和容量是随着kv对的变化而变化的
  - 如果容量大于了10(第二个参数指定的)，就会发生重新申请，和切片类似
- channel： make(chan int, 100)
  - 100 指缓冲信道的buffer大小
  - 如果忽略，就是同步信道(就是缓冲大小是0)

## channel type

信道，是Go提出了一个机制，用于并发函数之间的通信

    ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .

- 未初始化的信道是nil
- <- 操作符表明信道方向，省略表示双向信道
- 信道也支持赋值和显示转换

    var c chan T          // can be used to send and receive values of type T
    var c chan<- float64  // can only be used to send float64s
    var c <-chan int      // can only be used to receive ints

<- 操作符是左结合优先：

    var c chan<- chan int    // same as chan<- (chan int)
    var c chan<- <-chan int  // same as chan<- (<-chan int)
    var c <-chan <-chan int  // same as <-chan (<-chan int)
    var c chan (<-chan int)

make(chan int, 10), 10表示容量，前面提到过的。

如果容量没有指定或为0,表示是同步信道，读写都准备好了，通信才算成功，
不然都得阻塞。

如果指定了容量，且非0,表示是缓冲信道，
只有空的时候读会阻塞，满的时候写会阻塞，其他都不会阻塞。

nil信道是不能用来进行通信的。

- 关闭信道用close()
- 从信道里读数据，可利用多返回值的方式来判断信道关闭前有没有接到值
- 信道是fifo队列模式
