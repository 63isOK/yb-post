# 类型

类型决定了一系列变量可以对齐存储的值做哪些操作

    Type      = TypeName | TypeLit | "(" Type ")" .
    TypeName  = identifier | QualifiedIdent .
    TypeLit   = ArrayType | StructType | PointerType | FunctionType |
            InterfaceType | SliceType | MapType | ChannelType .

## method sets

方法集合:

- 类型可以有个和类型相关的方法集合
- 如果这些方法集合满足了某个接口类型,那这些方法集合就可以被看成类型的接口
- 方法集合,需要定义接收者
- 方法集合中的方法名,要是唯一的

## bool 类型

内置类型

## 数值 类型

表示整数/浮点数,具体的位数依赖于机器的体系结构

- 只有少数类型的位数会依赖机器: uint/int/uintptr
- 为了减少移植性问题,数值类型都设计成内置类型
- byte是uint8的别名,rune是int32的别名
- 不同数值类型的转换,需要显示表示

## string 类型

- 字符串
- 可能为空
- bytes的序列
- string的长度就是byte数
- 字符串是不可变的,一旦创建,字符串内容就无法改变
- 也是内置类型
- string可以通过[]下标访问

## array 类型

- 数组
- 单一类型 固定数量 的序列

    ArrayType   = "[" ArrayLength "]" ElementType .
    ArrayLength = Expression .
    ElementType = Type .

- 数量是数组类型的一部分,是一个非负整数
- 可通过[]下标引用具体某一个元素
- 数组是一维的,可组合成多维数组

## slice 类型

- 切片类型
- 是对底层数组中连续一段的描述
- 同时提供了对底层数组访问一段序列的能力
- 未初始化切片是nil

    SliceType = "[" "]" ElementType .

- 多个slice可能会共享同一段底层数组;相比之下,多个数组是不会共享存储的
- 但slice的容量不够时,会自动扩展成当前容量的2倍,底层数组也换成一个新的了
- 初始化切片类型使用 make([]T, length, capacity)
- 切片也可以从数组上生成
- make([]int, 50, 100) 和 new([100]int) [0:50] 是一样的
- 多维切片也是可以组合的,底层也是多维数组
- 多维切片内部的切片需要单独初始化

## struct 类型

- 可由不同类型的元素组成,一个元素叫field,每个field都有名字和类型

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

- 一个filed没有显式指定,就称为这个嵌入field
- 这个嵌入的field可以是类型名T,也可以是\*T(此时T不能是接口类型)
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
- field声明时,可附加一个ie字符串字面量标识
  - 空的标识和不添加标识等价
  - 这个标识只在反射接口和类型标识中起作用,其他时候都忽略

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

- 函数类型中的参数名/返回值名,要么全部显示,要么全部忽略
- 最后一个参数类型前可带有... ,表示可变参数,后面可能有0个参数,也可能有多个参数

## 接口 类型

- 接口类型,内含方法集
- 接口类型的变量,可存储任何实现了接口的类型的任何实例
- 未初始化的接口变量是nil

ebnf:

    InterfaceType      = "interface" "{" { MethodSpec ";" } "}" .
    MethodSpec         = MethodName Signature | InterfaceTypeName .
    MethodName         = identifier .
    InterfaceTypeName  = TypeName .

- 接口里面也可以嵌入接口
- 接口里面不能嵌入接口本身,也不能形成嵌套嵌入

## map 类型

- 无序,一个类型多个元素的分组,里面存的是kv
- 未初始化map是nil

    MapType     = "map" "[" KeyType "]" ElementType .
    KeyType     = Type .

- key需要实现==和!=
- key不能是function/map/slice
- key也可以是接口类型,接口变量中的值要实现比较操作,不然会包异常
- m[a]=b 可直接使用这种方式来添加一个 map element

## channel type

- uninitialized channel is nil
- provides a mechanism for `concurrently executing functions` to
 communicate by `sending` and `receiving` values of a specified element type.

    ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .

- <- mean `direction`
- make(chan int, 100): buffered channel
- make(chan int); make(chan int, 0): unbuffered channel
