# 声明和作用域

声明就是将一个非空标识绑定到 常量/类型/函数/标签/包,
程序中的每一个标识都必须声明,同一个代码块中,相同标识不能声明两次,
在文件块和包块中，也不能声明两个相同的标识

空白标识在使用上和其他标识符一样,只是没有绑定,所以不叫声明，
在包块中,标识符可以放在init()中进行声明

ebnf:

    Declaration   = ConstDecl | TypeDecl | VarDecl .
    TopLevelDecl  = Declaration | FunctionDecl | MethodDecl .

标识符在声明后，会在源码文件中的可见范围就是她的作用域。
Go在词汇上将作用域称为块，就是上篇文章提到的块。

- 宇宙块，包含了预定义标识符(内置类型/关键字等...)
- 包块，包括了常量/类型/变量/函数
- 文件块，包括了包中的一个文件
- 函数的参数和返回值，方法的接收者，作用域都是函数体
- 函数内的常量和变量，她们的作用域是从声明到最近一个块
- 函数内的类型标识符，她们的作用域是从声明到最近一个块

package句子不是声明，包名不属于任何一类作用域，她的作用仅仅是：
将属于同一个包的文件用一个包名进行区分，这个包名可以被其他包导入。

## label scopes 标签作用域

Go中有标签的概念

    Loop：
      代码

标签是给 break/continue/goto语句使用的，定义一个标签，但不使用，
是非法的。

标签不是标识符，也不属于任何作用域，所以标签和标识符是没有什么冲突的。

标签的作用域是标签声明的函数体里，`不包含嵌套函数`

## blank 空白标识符

就是下划线,称为空白标识符,
表示匿名占位符,用在声明/操作数/赋值上

## 预定义标识符

预定义标识符的作用域是宇宙块，隐式声明的。

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
- 标识符定义在包块,或者是field的名称,或是方法名

不符合上面两条的标识符,均不可导出

## 标识符的唯一性

- 什么叫唯一,一群个体,其他一个和其他任意一个都不同,叫唯一.
- 什么叫标识符唯一,相同作用域拼写不同叫唯一;拼写相同,但在不同package作用域且没有导出,也叫唯一
- 其他情况，就是有冲突的可能性

## 常量声明

常量声明就是给一系列标识符绑定值(要么是常量值，要么是常量表达式)

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

在"因式分解"的常量声明中(就是带括号的形式)，除了第一个声明的常量，
后面的常量可以省略掉表达式部分，这时，省略掉表达式的常量声明，
会一直往前找，直到找到第一个带表达式的声明项，和她共用一个类型。
类型是可以往前追溯，值就可是使用iota常量生成器来计算，
在一些简单的场景(轻量级)，iota是可以满足需求的。

    const (
        One = iota+1
        Two
        Three
        _
        Five
    )

## iota

- 常量生成器,在常量定义中使用
- iota是一个预定义的常量,上面也有提到
- iota是一个连续的无类型int常量
- 一条语句中使用使用,iota自增1
- 在const语句中,iota从0开始,多个const语句是不连续的
- 常量声明列表，iota表示`索引值`，一个声明中，iota的值是不变的
- const + iota 可代替其他语言中的枚举
- 对于想跳过的值，可以用空白标识符

## 类型声明

类型声明就是将标识符(类型名)绑定到一个新的类型中。

类型声明支持两种声明：

- 别名声明
- 类型定义

    TypeDecl = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
    TypeSpec = AliasDecl | TypeDef .

### 别名声明

给已有类型重新绑定一个标识符

    AliasDecl = identifier "=" Type .

在这个标识符的作用域内，这个别名都等同于这个类型

### 类型定义

这里就是新创建一个类型

    TypeDef = identifier Type

从ebnf上看，类型别名和类型定义唯一的区分就是是否使用=，
别名有=，新类型的定义没有

    type ii int // 新类型
    type iii = int // 类型别名,iii的类型还是指int

差别:

- 写法上有差异,带等号就是类型别名,不带就是声明新类型
- 用法上有差异,int可直接赋值给iii,因为iii就是int,但int不能直接赋值给ii,因为类型不同

关于新类型的声明有一点需要了解:

定义的新类型是有一个与之关联的方法集，
其中包括了"接口类型和符合类型元素"的方法集，不包括继承的方法集。

    // A Mutex is a data type with two methods, Lock and Unlock.
    type Mutex struct         { /* Mutex fields */ }
    func (m *Mutex) Lock()    { /* Lock implementation */ }
    func (m *Mutex) Unlock()  { /* Unlock implementation */ }

    // NewMutex has the same composition as Mutex but its method set is empty.
    type NewMutex Mutex

    // The method set of PtrMutex's underlying type *Mutex remains unchanged,
    // but the method set of PtrMutex is empty.
    type PtrMutex *Mutex

    // The method set of *PrintableMutex contains the methods
    // Lock and Unlock bound to its embedded field Mutex.
    type PrintableMutex struct {
      Mutex
    }

    // MyBlock is an interface type that has the same method set as Block.
    type MyBlock Block

这个官方例子分析：

- 回到最初始，定一个新类型的方法： type 标识符 类型
- 在之前的文章，类型的表示是struct {}， interface {}， func()()
- 下面只考虑和方法集有关的话题：

    type 标识符 struct {} 这是定义一个新的struct类型
    type 标识符 interface{} 这是定义一个新的接口类型
    上面两句是从spec中得出的结论，和Go基础教程上介绍的完全吻合

    如果是像上面这种，都是新创建一个类型，类型也是新定义的，
    这种情况下是不涉及方法集疑惑的。下面看看从已有类型来新建类型：

    type 标识符 结构体1
    type 标识符 接口2

    从上面的spec描述上看:
    1. 结构体1的方法集是不会被新类型继承，但结构体元素的方法集保持不变
    2. 接口2的方法集是会被新类型继承

## 变量声明

变量的声明就是创建多个变量，并和标识符绑定在一起，之后确定类型并初始化。

    VarDecl     = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
    VarSpec  = IdentifierList (Type ["=" ExpressionList ]|"=" ExpressionList ) .

如果给了表达式，就按"赋值"的规则初始化变量;否则变量初始化为零值

如果指定了类型，那每个变量都是这个类型;否则，变量的类型就从赋值中推导

如果值是无类型常量，那么常量会隐式转换成其对应的默认类型

nil不能用于初始化一个没有指明类型的变量

如果一个变量声明后,一直未用到,那么编译器会认为是非法的

## 短变量声明

    ShortVarDecl = IdentifierList ":=" ExpressionList .

短变量声明，可能会在同一作用域下，重新声明变量(特殊情况)，
重新声明并不会新建一个变量，而是直接将新值赋给老的变量。
短变量声明多个变量时,如果至少有一个是新变量,就可以使用短变量声明方式

## 函数声明

函数声明就是将标识符(函数名),绑定到一个函数

    FunctionDecl = "func" FunctionName Signature [ FunctionBody ] .
    FunctionName = identifier .
    FunctionBody = Block .

从上面看，函数类型和函数声明还是有些区别的，函数声明在func后加了一个函数名。
这点上，和结构体类型/接口类型的的声明还是有一点点不同

Signature，定义了参数(包括函数参数和返回值参数)。

如果有返回值，那函数体内就一定要有结束语句(具体后面会定义什么叫结束语句)

和c家族类似，函数是可以先声明，后实现的。

## 方法声明

方法声明就是函数加上了一个接收者

    MethodDecl = "func" Receiver MethodName Signature [ FunctionBody ] .
    Receiver   = Parameters .

接收者也是整个参数中的一部分，接收者是一个非可变参数.
接收者可指定是值类型还是指针类型,
接收者类型T不能是指针或接口类型,T必须在相同package中定义

因为接收者也是参数的一部分，所以命名也不能有冲突(保持唯一)。
如果方法里未用到接收者的值,则在定义方式时可忽略接收者变量,就和函数参数类似

对于接收者来说，方法要保持唯一，如果接收者是结构体，那么方法名和字段名要不冲突。

实际处理中，方法是可以转换为函数的，将接收者作为第一个参数即可。

反过来，`函数是不能转换为方法的`。
