# 自引用函数和option的设计

发表于2014/01/24

## 场景(原始需求)

写一个包，对外暴露的接口中，可选参数越来越多，所以想找到这么一种方式：
不需要太多的api;其次参数随时可以做扩展。

达到第一点有很多方式，达到第二点就不是很容易。

## 过程

最显而易见的是option 结构体，这个结构体有很多方法，可变参构造。
然而这种方法效果并不太令人满意。

经过1年多的时间，找到这么一种方法，自引用函数。

下面介绍如何提炼出这种方法。

先定义一个option类型

    type option func(*Foo)
    // 定义一个函数类型
    // 这类函数的目的是设置状态

其次定义一个Foo的方法Option

    func (f *Foo) Option (opts ...option) {
      for _, opt := range opts {
        opt(f)
      }
    }
    // Foo是指针接收者，是为了修改Foo的状态
    // 修改Foo状态的正是参数opts(就是一系列函数列表)
    // 函数列表中函数的参数，正好是接收者参数

之后，通过闭包来修改Foo对象的状态。eg：现在函数有个int的参数(age)

    func SetAge(age int) option {
      return func(f *Foo) {
        f.age = age
      }
    }
    // 利用闭包来修改状态
    // 还有一点，利用返回另一个函数值来使用闭包

最后，我们的代码可能是这样的

    foo.Option(
      SetAge(30),
      SetName("zz")) 
    // 此时，入参是变变参，数量无限制

这种解决方案(函数值 + 闭包)可应对大部分场景，但是威力可以继续升级。

如果能用option机制来设置临时值就好了。

改造方法是将option改为接口，

    type option func(*Foo) interface{}

    // 下面的具体修改状态的函数还提供了其他功能：返回上一次的状态
    func SetAge(age int) option {
      return func(f *Foo) interface{} {
        previous := f.age
        f.age = age
        return previous
      }
    }

    // 这个例子并不是很合适，只是说明了新增功能改如何扩展
    func (f *Foo) Option(opts ...option) (previous interface{}) {
      for _, opt := range opts {
        previous = opt(f)
      }

      return // 这里返回的是后后一个修改状态对应的前状态，并不是所有参数的前状态
    }

重新定义option 这个函数值

    type option func(*Foo) option
    // 这叫自引用函数，参数或返回值是另一个option时，叫自引用函数
    // 再抽象一点，option也是符合interface{}的
    type option func(*Foo) interface{}
    // 这个就和上面例子中新增功能：返回前状态。她们使用的option是一样的

    func (f *Foo) Option (opts ...options) (previous option) {
      for _, opt := range opts {
        previous = opt(f)
      }

      return
    }

    // 此时闭包要返回option，而不是interface{}
    func SetAge(age int) option {
      return func(f *Foo) option {
        previous := f.age
        f.age = age
        return SetAge(previous) // 返回的是函数值，而不是执行这个函数，没有()
      }
    }
    // 此时这个闭包返回的不是前状态了，而是另一个闭包

此时，常规用法是这样的

    prev := foo.Option(SetAge(30))
    xxx
    foo.Option(prev)
    // 这几句话是，修改一个状态，做一些事，将之前的状态复原

可以适用的场景是：

    func DoSomethingVerbosely(foo *Foo, verbosity int) {
        // Could combine the next two lines,
        // with some loss of readability.
        prev := foo.Option(pkg.Verbosity(verbosity))
        defer foo.Option(prev)
        // ... do some stuff with foo under high verbosity.
    }
    // 在某些场景修改一些状态，在函数返回后，将状态复原
    // 而前面提到的对接口参数的扩展，也是支持的

这种写法只是看起来花哨，现在返回的是闭包，我们无法取到前状态，
我们所做的仅仅是将一些信息用闭包的隐藏了。

对于可变参的扩展，只需要加几行代码即可。
这样的写法比一个接口，十个入参要好很多，至少，优雅了很多。
对于包的使用者来说，易用性大大提高了。

主要是通过闭包来达到的。
