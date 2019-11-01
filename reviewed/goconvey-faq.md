# FAQ

## goconvey是什么

- go test工具的一个扩展
- 也是go中的一个BDD框架
- 也可以作为传统测试的一个web查看结果的手段

goconvey主要包含了两部分：

- BDD框架
- 一个服务 + web ui (用于显示测试结果)

这两个部分是独立的，可分开使用

## goconvey是否可以和go test一起使用

可以

## Convey()嵌套，这么怪异的方式的好处是什么

这种思想是基于树的，可以进行关联测试，实际上收益会很大的。

eg：测试 "登录 消费 查看结果"，和测试"登录 修改资料",
完全可以将登录关联

基于树形分支，具体可查看前面的文档来了解。

## web ui是什么

一个服务，可将测试的结果实时显示到h5界面上，全自动，代码更改即可看到结果

## 如何设置自动化

自动化看结果的途径有两种：

- web ui，这个服务已经设置了自动化
- 控制台，需要执行一个python的自动化脚本，所做的动作和web ui一致

## web ui和传统go test是否可以一起工作

是的

## 在断言中，如何输出调试信息

使用convey包中的Print/Printf/Println函数

## 如何让测试失败后，继续执行下一个测试

默认测试失败就停止，如题上的策略也是可以指定的：

    Convey("test A", t, FailureContinues, func(){

    })

添加参数FailureContinues即可

由于嵌套也会继承这个属性，所以可以在init()中指定

    func init() {
        SetDefaultFailureMode(FailureContinues)
    }

## 为啥没看到覆盖率报告

需要将测试放在$GOPATH下

## 是否商业可用

这个项目是开源项目，商业可用。
如果有能力，可以回馈这个项目。
