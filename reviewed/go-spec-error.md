# 错误和运行时异常

## error

    type error interface {
      Error() string
    }

这是一个接口，nil表示没有错误，具体分析可看标准库源码分析

## 运行时异常

spec上介绍的很多违规情况都会触发运行时异常，其真面目是：

    package runtime

    type Error interface {
      error
      // and perhaps other methods
    }

也是一个错误
