# GoConvey

这是一个bdd框架，集成了大量的功能，可在控制台或h5显示测试结果。
这个项目被chrome所使用。口号是：在编辑器中写测试用例，在浏览器看结果。

## 使用很简单

    go get github.com/smartystreets/goconvey
    goconvey -port=8000 -host=172.17.0.2 -launchBrowser=false

然后就在浏览器中打开即可，因为我使用的是docker，所以设置了端口和ip

之后修改代码，测试会自动将结果显示在网页上，也有通知，真的非常方便。

如果代码有变更，测试结果会立马显示在浏览器上，如果测试不通过，马上有反馈。

## 讨论

- CI是集成测试，有几个必要条件：版本管理/build工具/测试代码/CI工具
- 其中测试代码的编写，一般基于两种框架：TDD/BDD
- BDD框架有很多种，各种编程语言的都有
- go的BDD框架也有很多种，chrome中集成的是GoConvey
- GoConvey实现了BDD框架，利用这个框架可以写BDD测试代码
- 不过GoConvey能做的更多，对于个人来说，简化了很多要执行的测试命令
