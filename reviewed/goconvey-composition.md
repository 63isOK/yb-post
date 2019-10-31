# GoConvey测试的编写

## 重要函数的分析

    func Convey(items ...interface{})

Convey用于定义一个spec的作用域。

## 使用goconvey的步骤

    import(
      "testing"
      . "github.com/smartystreets/goconvey/convey"
    )

在测试用例中，使用Convey来定义scope/context/behavior/ideas，
理解：Convey函数可表示很多东西：作用域/上下文/行为/想法。
只有顶层Convey需要\*testing.T对象参数，其他嵌套的不需要。
