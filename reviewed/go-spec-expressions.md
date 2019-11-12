# 表达式

一个表达式是指对一个值的计算(利用运算符或函数来计算)

## 操作数

表达式中的基本值叫做操作数。
操作数可能是一个文字，非空标识是常量/变量/函数/带参表达式

空标识符在赋值语句的左边时，可以认为是一个操作数

    Operand     = Literal | OperandName | "(" Expression ")" .
    Literal     = BasicLit | CompositeLit | FunctionLit .
    BasicLit    = int_lit | float_lit | imaginary_lit | rune_lit | string_lit .
    OperandName = identifier | QualifiedIdent.

下面就是要了解，ebnf中定义的操作数有哪些

## 合格的标识符

合格的标识符是有包名前传的标识符。包名和标识符都是非空的

    QualifiedIdent = PackageName "." identifier .

要在其他包中访问这个合格的标识符，那么这个标识符必须导出。

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

数组复合字面量中的...表示数组的长度是最大索引 + 1

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
