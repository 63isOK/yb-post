# 声明和作用域

- 声明就是将一个非空标识绑定到 常量/类型/函数/标签/包
- 程序中的每一个标识都必须声明
- 同一个代码块中,相同标识不能声明两次
- 空白标识,并不是实际意义上的声明,因为没有绑定过程
- 在package块中,只有一个init用于声明init函数

    Declaration   = ConstDecl | TypeDecl | VarDecl .
    TopLevelDecl  = Declaration | FunctionDecl | MethodDecl .

- 一般声明有 常量/类型/变量
- 顶级声明有 一般声明/函数/方法
- go中用块来表示作用域,包括上一节的 显示代码块/隐式代码块

go中关于作用域,一般有以下几个规则:

- 预定义标识的作用域是universe,作用域是全局的,任何地方都可以有的,eg:true nil make new
- 常量/变来嗯/类型/函数(不包括方法),如果定义在最顶层(函数体之外),就是package作用域
- 用import导入一个包,那么这个文件的作用域包含了导入包的文件作用域
- 方法的接收者/函数参数/函数返回值,他们的作用域都在函数体内
- 函数体内定义的常量/变量/类型,她们的作用域从定义开始到最近的块结束

和其他语言一样,内层作用域是可以重新定义一个同名的标识符,同样有覆盖作用,
另外包名不属于任何作用域,她的目的是将同一个包的文件分组

## label scopes 标签作用域

- 标签有标签的声明语句
- 配合break/continue/goto 使用
- 未使用的标签是非法的
- 标签不属于作用域,也不会于其他标识符冲突
- 标签的作用域和函数里的变量类似,都是从定义到最近的块结束

## blank 标识符

就是下划线,称为空白标识符,
一般用于匿名占位符

## 预定义标识符

预定义标识符的作用域是全局的

    Types:
        bool byte complex64 complex128 error float32 float64
        int int8 int16 int32 int64 rune string
        uint uint8 uint16 uint32 uint64 uintptr

    Constants:
        true false iota

    Zero value:
        nil

    Functions:
        append cap close complex copy delete imag len
        make new panic print println real recover

## 可导出的标识符

可导出,意味着可被其他包使用,满足下面两个条件即可导出:

- 首字母大写,这个大写是unicode大写字母
- 标识符定义在package作用域,或者是field的名称,或是方法名

不符合上面两条的标识符,均不可导出

## 标识符的唯一性

- 什么叫唯一,一群个体,其他一个和其他任意一个都不同,叫唯一.
- 什么叫标识符唯一,相同作用域拼写不同叫唯一;拼写相同,但在不同package作用域且没有导出,也叫唯一

## 常量声明

常量的声明可用列表来声明:

    ConstDecl      = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
    ConstSpec      = IdentifierList [ [ Type ] "=" ExpressionList ] .

    IdentifierList = identifier { "," identifier } .
    ExpressionList = Expression { "," Expression } .

看这写法,常用的应该有两种:

    const (
        a,b,c = 1,2,3
    )

    const (
        a = 1
        b = 2
        c = 3
    )

当然,常量也是可以指明类型的,如果不指明类型,就看常量表达式是否带类型,
如果表达式不带类型,那么常量还是不带类型,直到具体使用时再确定类型.

常量的声明还可以简化写法,常量表达式是可以省略的,省略调的常量如何取值,
他们的值和第一个常量的常量表达式一致,甚至可以用iota常量生成器混入常量表达式,
这样常量的的值可各不相同

    const (
        One = iota+1
        Two
        Three
    )

## iota

- 常量生成器,在常量定义中使用
- iota是一个预定义的常量,上面也有提到
- iota是一个连续的无类型int常量
- 一条语句中使用使用,iota自增1
- 在const语句中,iota从0开始,多个const语句是不连续的
- const + iota 可代替其他语言中的枚举

## 类型声明

    TypeDecl = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
    TypeSpec = AliasDecl | TypeDef .
    AliasDecl = identifier "=" Type .
    TypeDef = identifier Type .

支持两种类型声明方式:一种是声明已有类型的别名;一种是声明一种新的类型

    type ii int // 新类型
    type iii = int // 类型别名,iii的类型还是指int

差别:

- 写法上有差异,带等号就是类型别名,不带就是声明新类型
- 用法上有差异,int可直接赋值给iii,因为iii就是int,但int不能直接赋值给ii,因为类型不同

关于新类型的声明有一点需要了解:

    // 只继承数据,不继承方法集
    type A struct{
        ...
    }

    func (a *A) Lock() {}
    func (a *A) UnLock() {}

    type I interface {
        ...
    }

    type B A // 此时B是不会继承A的方法集的,只继承数据结构
    type B I // 如果I是接口,那么B会继承I的所有方法集

    // 全部继承(其实里面还分各种情况,具体后面遇到了再说)
    type C struct {
        A
    }

总结一下:

- type a b 写法,声明一个新类型
- b是结构体,就只继承数据,不继承方法集
- b是接口,继承方法集

## 变量声明

    VarDecl     = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
    VarSpec  = IdentifierList (Type ["=" ExpressionList ]|"=" ExpressionList ) .

- 变量声明时,如果指定了初始化表达式,就用表达式计算的值进行初始化,否则,使用零值
- 如果未指明类型,就用表达式推导
- 如果表达式是各无类型的常量表达式,那显示转换成默认类型 var i = 12 转换成int类型,var j = nil 就是非法的
- 如果一个变来嗯声明后,一直未用到,那么编译器会认为是非法的

## 短变量声明

- 更短更简洁的变量声明 :=
- 短变量声明会遇到重新赋值的情况,常见的就是err,
- 短变量声明多个变量时,如果至少有一个是新变量,就可以使用短变量声明方式
- 适合在函数内部使用, 也适合用在 if for switch 语句中

## 函数声明

    FunctionDecl = "func" FunctionName Signature [ FunctionBody ] .
    FunctionName = identifier .
    FunctionBody = Block .

- Signature就是返回值,如果有,那么函数体中要有明确的return语句
- 函数体有时可省略,表示 外部实现

## 方法声明

- 方法就是函数加上了一个接收者

    MethodDecl = "func" Receiver MethodName Signature [ FunctionBody ] .
    Receiver   = Parameters .

- 接收者是一个非可变参数
- 接收者可指定是值类型还是指针类型
- 接收者类型T不能是指针或接口类型,T必须在相同package中定义
- 如果方法里未用到接收者的值,则在定义方式时可忽略接收者变量,就和函数参数类似
