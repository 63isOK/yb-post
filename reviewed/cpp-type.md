# 自定义类型

  由基础类型组合的复杂类型都属于本文中的自定义类型，
还包括了const修饰的基础类型、指针/引用/数组修饰的基础类型，
都称为自定义类型。

  内置类型，种类丰富，故意设计为低级别操作，
好处是直接映射硬件提供的功能，所以高效。
所以内置类型一般不会提供高级特性。
程序员通过下面这种方式来提供高级特性：
`内置类型做为参数，和抽象机制一起`

  c++的抽象机制，主要目的是让程序员设计和实现自定义类型，
这些类型提供合适的操作，使用者可以简单优雅地使用这些类型。
由使用c++抽象机制构建的类型，称为自定义类型。
这些自定义类型被称为`类`或是`枚举`。
自定类型可包含内置类型或自定义类型。

    这个cpp语法系列更多着眼于自定义类型的`设计``实现``使用`

  推荐优先使用自定义类型，好处是：
使用简单；
不易出错；
比直接使用内置类型实现相同功能更加高效。

## struct

  一种数据结构，常用于按需组织元素

  struct没有初始化，就是无用的，所以初始化是必须的。

    // 定义
    sturct Vector
    {
        int size;
        double* element; // 只有初始化之后才有使用价值
    };

    // 初始化
    // 参数Vector&中的&表示：传的是一个non-const 引用
    // 潜在意思是这个参数是可以被修改的
    void vector_init(Vector& v, int s)
    {
        v.size = s;
        v.element = new double[s];
        // new 从free store申请内存
        // 这个free store也被称为动态存储区和堆heap
        // 释放有delete操作实现
    }

    // 使用
    double read_and_sum(int s)
    {
        Vector v;
        vector_init(v, s);

        for (int i = 0; i != s; i++)
            std::cin >> v.element[i];

        double sum = 0;
        for (int i = 0; i != s; i++)
            sum += v.element[i];

        return sum;
    }

  标准库也提供了vector的实现，比上面的实现进行了很多优化。

## class

  class,有数据，也有基于这些数据的操作
  
  todo p59
  2.3 class
