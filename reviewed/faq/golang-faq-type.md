# 类型

## 是面向对象语言吗

是也不是。Go的类型和方法可以支持面向对象编程，但没有继承

## 如何动态分发方法

接口

## 为啥没有继承

更轻量的接口来表示类型之间的关系，接口

非入侵式的无需修改原始类型

## 为啥len是函数而不是方法

因为用的多

## 为啥不支持方法和操作符的重载

方法，如果不需要类型匹配，那分派时就会很简单。

操作符重载也是类似，为了简单，一切都可以丢掉

## 为啥不需要声明实现

Go类型系统已经搞定了，不需要使用者来关系这个

## 如何保证类型满足某个接口

让编译器来检查

    type T struct{}
    var _ I = T{}       // Verify that T implements I.
    var _ I = (*T)(nil) // Verify that *T implements I.

一般不需要用到

## 下面几个实现是否满足特定的接口

    type Equaler interface {
        Equal(Equaler) bool
    }

    type T int
    func (t T) Equal(u T) bool { return t == u } // does not satisfy Equaler

    type T2 int
    func (t T2) Equal(u Equaler) bool { return t == u.(T2) }  // satisfies Equaler

    type Opener interface {
       Open() Reader
    }

    func (t T3) Open() *os.File

只有T2满足，就是方法和接口中的方法要一致。

## []T 转[]interface{}

不能直接转，内存中的表现就不一样，只能先建一个[]interface{}再赋值

## T1 T2底层类型一样，[]T1是否可以转[]T2

T2(T1) ok

([]T2)([]T1) 不行

## 如何理解接口的nil

接口，底层实现是(t,v),t表示实现接口的具体类型，v表示具体类型的值，
eg： (int,3)

nil接口表示t和v都没有设置(nil,未设置)

    func returnsError() error {
      var p *MyError = nil    // 此时可以看成接口是(*MyError, nil), 并不是nil
      if bad() {
        p = ErrBad
      }
      return p // Will always return a non-nil error.  // 此时预期是返回nil

      // return nil 这个就是正确的
    }

这点属于语言的副作用了，不理解清楚，就会感到困惑

所以说，如果要返回一个接口类型，最好直接返回接口类型对应的值，
返回实现，就会在某些场景下出问题，例如上面的例子。

另外一点，要返回一个接口，实际返回的是接口实现类型，那一定不是nil。

## 为啥没有c中的联合 union

内存模型不支持

## 为啥没有多类型

就是复数类型
