# 错误和运行时异常

## error

    type error interface {
      Error() string
    }

这是一个接口，nil表示没有错误，具体分析可看标准库源码分析.
这个接口表示的是错误状态.

## 运行时异常

spec上介绍的很多违规情况都会触发运行时异常.
也是一个错误.

部分error会引发异常(这个异常和手动调用内置函数panic的效果一样),
运行时异常,等同于将runtime.Error作为panic的参数去调用.

    package runtime

    type Error interface {
        error

        // RuntimeError is a no-op function but
        // serves to distinguish types that are run time
        // errors from ordinary errors: a type is a
        // run time error if it has a RuntimeError method.
        RuntimeError()
    }

runtime.Error内嵌了一个接口error.

从这上面就可以看出spec中说的错误error和运行时异常特指的错误runtime.Error,
是不同的,也有一定的联系.

错误和异常,使用的场景和具体的意义也是不一样的.
