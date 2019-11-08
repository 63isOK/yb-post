# 包和测试

## 一个项目可包含多个包吗

可以，不同的包用不同的目录即可，
编译的时候就像编译一个包。

## 如何写单元测试

1. \_test.go
2. import "testing"
3. func TestXXX(t \*testing.T){}
4. go test

## testing中的辅助函数在哪

testing包适合编写单元测试，针对feature的功能测试就需要自己写assert断言，
这些断言函数，就属于辅助函数，输出错误信息的函数一般也会被封装成辅助函数。

虽然有时候写辅助函数有点重复和无趣，但是在表格驱动的测试例子，是非常有用的。

- 单元测试用testing，ok
- bdd测试用GoConvey，这个库google的核心产品都在用

## 为啥标准库中没有xxx

标准库的目的是为了支持运行时/连接os/提供大多数Go程序需要的关键功能(io/网络)

godoc.org可查找很多库
