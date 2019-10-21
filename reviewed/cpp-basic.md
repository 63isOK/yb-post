# 认知补充

## c++的两个实体

* 语言的核心特征 - 定义了内置类型、循环等
* stl - 容器、io操作

## stl是纯c++写的吗

    除了线程切换上下文用机器语言写的，其他都是c++

## main函数中可以不明显写return吗

    现在的规范都推荐写上 return 0；
    `不写，也没问题`，默认会返回成功，也就是0
    非0表示失败，这个返回值在linux中用的比较多，windows上不常用

## std::cout << "hello"; <<怎么读

    put to

## 关于函数的参数

    编译期需检查的值和类型转换 需要引起重视
    函数如果是声明而不是定义，那么编译器会忽略函数名
    定义时函数名才有意义

## 函数类型

    函数类型包含了返回值 + 参数顺序、参数类型
    如果是成员函数，还得 + 类名

    int a(int b); // int(int);
    int b::a(int aaa); // int b::(int);

## 代码维护

    第一步是写出容易理解的代码
    可理解是只将computure执行的任务拆分成我们能理解的一个个小块(这里是指function and class)
    更大的代码和任务，都是由这些小块组成的，函数，stl提供的算法等

    代码中的错误和代码量、代码复杂度直接相关
    使用更加短的function可以有效解决代码量和复杂度的问题，一个function只干一件事

## 重名函数

    也就是function overloading 重载，是泛型编程的基石
    重载函数，里面实现的语义应该是一样的
    eg： print(int)和print(std::string)都应该是打印操作，而不该一个是打印另一个是计算

## type and entity

    type关系到操作，不同type的数据、对象，他们所拥有的操作是不一样的
    什么是声明：
        声明就是用type定义一个对象，这个对象称为entity 实体
        声明就是告诉程序，这儿将会有一个实体。并指明这个实体的类型
    type：定义了一个值域和一组操作
    object：一块内存，里面有个某种类型的值
    value：可以用bit来解释，具体一个value有多少bit，就看她的type是什么
    variable：变量 = object 变量是object的名字

    c++提供的基础类型不多，每种基础类型的长度都不一定一样，具体占多少字节，由具体的实现决定
    以前很多厂商提供的int是16位，现在x86系统大部分提供的是32位，未来64位普及后，说不定就是64位
    所以说具体是多少，可以用sizeof(type)来查看
    数值有整形和浮点型

## initialization 初始化

    对象在使用之前需要先初始化，c++中有两种非常常用的初始化方式：
        = ， eg int a = 1; 这种方式继承于C，
        {}， eg int a{1};  这种方式是c++推荐使用的。
        为什么标准要搞一套新东西？
          = 继承于c，没有强制类型检查，int a = 3.2;会触发隐式转换，最后a是3
          {}，int a{3.2}; 会报错，因为会丢失精度
          这是为了兼容c而引入"缩小转换"+"隐式转换"，使用{}是推荐的

    常量需要初始化，变量只有在极少场景下可以不初始化
    一个对象，只有有了合适的值，才有意义，对机器来说，
        声明了不定义(也就是不初始化，没有分配内存，也就没有相应的值)
        对机器来说，就是没有太大意义，定义才有意义
    定义的过程会触发对象的初始化

    定义一个变量时，如果通过初始化能推导出类型，就不毕显示指定类型

    auto i = 12; // int
    auto b = true; // bool
    auto a = 'a'; // char

    使用auto时，初始化方式推荐使用 = ， 因为此时不会有隐式转换
        此时并不是说不适合用{}初始化方式，都是可以的
    没有特殊原因需要显示指明类型，推荐使用auto，特殊原因一般是：
        定义块太长，有个明显的类型方便阅读
        想要明确对象的范围或精度 eg double a{1};

    auto的好处：减少冗余；类型很长的时候用auto很方便
        更大的好处在与泛型编程时，类型实在是太长了

## 作用域和声明周期 scope and lifetime

    作用域： 
        local scope，局部作用域，出现在函数体内和lambda内，包括了函数参数
        class scope, 类作用域，类成员都在这个作用域里
        namespace scope, 命名空间作用域

    如果一个成员不在{}里，称为全局变量、全局函数，
        也可以认为在全局命令空间作用域

    上面说的都是有名字的对象，有名字，肯定会出现在某一作用域里
    还有一些对象是没有名字的：临时变量和new创建的对象

    对象在使用之前需要先构造(初始化)，在作用域结束时会析构(销毁)
    namespace scope里的对象的析构在程序结束时，
    class scope里的对象，析构在对象释放时
    new创建的对象，在delete时释放

## 常量

    c++中支持两种概念上的常量：
    const：表示不会改变这个对象的值，一般用在接口上
        表示传给函数的指针或引用不会被修改
        编译器看到了会用const来保证不会修改值
        const修饰的值，可在运行时计算
    constexpr：新标准搞出来的东西，表示会在编译期计算值
        主要用在特定的常量，将数据放在只读内存，防止被修改
        好处是性能提升了，因为编译期就把值计算出来了

    一般的常量(不局限于c++)分两种：物理不变、逻辑不变
    c++采用的是物理不变，即内存中常量对象的值不变
    实现方式是通过编译器的const机制来实现，
    任何修改内存值的操作都是违规的

    const的重点是值不会变
        const int a = 1; 编译器确定常量值
        const int b = fun(); 运行器确定常量值

    constexpr是const的更进一步：编译器就确定常量值
        优化程度更高，c++11和c++14描述的规则不一样

    如果一个函数要在常量表达式中使用，那么这个函数要用constexpr修饰
        有这个constexpr，就是告诉编译器在编译期就计算

    constexpr函数可以当做非常量参数，此时结果就不是一个常量表达式
    constexpr函数的参数可以不是常量表达式，此时函数就和普通函数一样
        结果是一个非常量表达式
        好处是不必为了"constexpr函数"和"普通函数"定义两次

    constexpr的好处好像蛮多，是不是所有的函数都可以constexpr修饰
        不是，constexpr函数有要求：要简单，要没有副作用
        说白了：不能修改非local变量

    后面会说明需要常量表达式的场景
        主要是编译器计算对性能的提升巨大
        除此之外，也丰富了c++语言的特征库

## 指针 数组 引用

    最基本的数据容器是一段连续的内存，里面放同样类型的数据，
        也称为数组
        也是硬件所提供的方式

    char v[6];
    char* p = &v[5];
    char x = *p;

    [] 读array of
    char*的*读：pointer to
    *p,*在前面表示： contents of
    &在前面表示：address of
    for(int 1 = 0; i < 10; i++)怎么读：
        set i to zero;while i is not 10;increment i;

    上面的for loop是一种方式，新标准提供了一种更简单的方式：
        for (auto x : vec)
        称为 range for 语句

    上面的范围for语句怎么读：
        for every element of vec,from the first to the last ...

    for (auto& x : ver)
        &读reference to
        称为引用

    引用和指针类似
        只是指针对象前要加*才能访问对象
        引用使用时，对象一定要初始完成

    函数传参，使用const 引用，比较常用，也能节省开销

    在声明中， & * [] 称为声明操作符

## NULL 空指针

    空指针类型有两个值
        NULL,老代码中的，可以用0代替
        nullptr，新标准出的，为了消除int 0和空指针类型的混乱

    没有空引用的说法
        引用是需要有个有效对象的
        有些隐晦的技巧可以让对象无效，这样就违反了上面的规则
        尽量避免这种违规操作

## 测试 - 讲了一些基本的语句

    std::cout << "123";
    std::cin >> x;

    << 读 put to
    >> 读 get from

    if (auto x = fun(); 0 != x) {}
    x的作用域是if分支，和for语句中声明变量一样
    好处是限定x的作用域。
    因为if已经包含了0比较和nullptr比较
    所以可以省略成 if (auto x = func()){}
    这样性能略高一点

## 硬件映射

    当我们使用一些基础操作时，都是硬件提供的实现，
    像+-*/都是调用机器指令，通过机器指令去操作硬件
    所以最基本的操作都是由硬件来提供的

    指针，就是内存中的机器地址
    就是像类似的硬件映射，在底层的性能上让c、c++领先其他语言几十年
    c c++的计算机制是基于硬件的，而不是基于数学

    assign操作，也就是赋值操作，
        对基本类型来说，就是一个拷贝操作
    初始化initialization是不同于赋值操作的
        赋值的对象都是有值(从底层看，是分配了内存，内存里是有东西的)
        初始化的对象初始化之前是没有分配内存
    对绝大部分类型来说，对未初始化的变量进行读写，结果都是未知
        内置类型更明显

    int a = 1;
    int& b = a; // b初始化 不是赋值

    初始化和赋值的不同，更多体现在自定义类型中
    设计良好的自定义类型，最好提供= 和 == 操作

    函数的参数和返回值，语义上都是初始化

    数字分隔符 '
    π = 3.14159'26535'89793

## 建议

    这里的建议只是c++ core guide中的一部分
    全部的可以查看
[isocpp](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)

* Don’t panic! All will become clear in time
* 别只使用内置类型，多用标准库提供的
* 写好c++程序，并不要求知道所有c++的细节
* 着眼于编程技术，而不是语言特征
* 关于语言的讨论，以iso c++ 标准为主
* ‘package’操作，就行函数
* 一个函数只执行一个逻辑
* 函数尽量短
* 重载，对不同类型执行相同的逻辑
* 函数如果是在编译期计算，使用constexpr
* 明白语言对硬件的映射
* 大数字时使用数值分隔符来增加可读性
* 避免复杂表达式
* 避免失真转换(缩小转换 double -> int)
* 变量的作用域能小则小
* 避免魔鬼数字，使用符号常量
* 尽量用不可变数据(能用const的地方多用，提高性能)
* 每次声明只声明一个名字(int a,b,c,d;不可取)
* 通用变量和局部变量名字尽量短，其他的尽量长
* 避免看起来一样的名字
* 避免全大写的名字
* 优先使用 {}初始化方式
* 优先使用auto
* 避免未初始化变量
* 作用域尽量小
* 如果if语句里有声明，和0的比较可以隐藏
* 只有在位操作时使用unsigned
* 使用指针尽量简单
* 使用nullptr，而不是0和NULL
* 变量初始化时再声明
* 代码中可以说明的东西，不要靠注释来说明
* 注释中主要放意图
* 保持一致的缩放风格