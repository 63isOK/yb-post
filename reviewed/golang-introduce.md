# go 第一天

## go 简介
  
- 目标是提高产能
- 易表达 简洁 高效
- 并发机制更利于写多核 物联网
- 新的类型系统构造模块化系统更加灵活
- 编译速度快，支持gc(垃圾回收)和运行时反射
- 特点：快 静态类型 编译语言，类似c++
- 使用起来，感觉和动态类型 解释性语言很像

## 安装

ubuntu下直接用apt install golang 安装，
使用 go version 测试是否安装成功

GOPATH环境变量，指明工作目录

## 编译

在目录下执行　go build 生成的文件名和目录名一样
go install 将二进制丢到工作目录下的bin目录
go clean -i 删除bin目录下的二进制

编译安装之后，发现如下错误:  
go install: no install location for directory /home/yb/code outside GOPATH

解决方法是先指定工作区目录，再指定二进制目录

```shell
    export GOPATH=$HOME/code
    export GOBIN=$GOPATH/bin
    export PATH=$PATH:$GOBIN     // 这个是为了方便运行二进制
```

## 代码书写注意点

接下来主要介绍如何创建一个go包

### 代码组织

- 代码放在一个单独的工作区
- 一个工作区包含很多版本控制仓库
- 每个仓库包含一个或多个包 package
- 每个包在一个单独的目录下 包含一个或多个go源码文件
- 每个包的目录，决定了他的import目录

#### 工作区

工作区就是一个目录，包含两个子目录，src和bin

go工具构建和安装二进制，在bin目录  
src目录包含多个版本控制仓库

#### GOPATH环境变量

前面也提到过了设置GOPATH，就是设置工作区  
go env GOPATH 可以打印GOPATH

#### import 路径

导入路径是一个字符串，用于标识包，唯一的  
包的import路径对应着工作区下的目录或远程仓库的目录

标准库中的包，她们的import路径是fmt net/http，
我们自己开发的包，需要选择一个基础路径，
不能和标准库或其他或者库冲突

如果源码在仓库中，仓库的跟目录就是基础路径，
eg：github.com/63isOk 就是基础路径，
对应的，工作区结构是 mkdir -p $GOPATH/src/github.com/63isOK

#### 再来一次hello world

程序第一件事需要考虑package path，github.com/63isOK/hello，
之后在工作区创建响应的目录 mkdir -p $GOPATH/github.com/63isOK,

编译安装，go install github.com/63isOK/hello 任何地方执行都行，
效果等同于先cd到hello目录，再执行go install

也可以安装github上绑定账号，初始化工作区里的github仓库

#### 第一个library

go build $GOPATH/github.com/63isOK/golearn/stringutil  
编译并没有输出文件，而是将保存到本地构建缓存中了

使用 import 列表中添加 github.com/63isOK/golearn/stringutil

如果导入了包，源码中却没有使用，会有相关的提示

#### 包名 package name

go源码第一句话就是 package xxx   xxx就是包名,
同一个package的包名是一样的，
包名就是import路径中的最后一部分

可执行的包名只能是main

包名是可以重复的，但是import路劲是唯一的

#### test

go中包含一个轻量级的测试框架，testing包和go test命令

源文件是hello.go,测试文件对应是hello_test.go,
hello中有个Max函数，测试中是TestMax(t \*testing.T),

如果测试通过，提示ok，不然就会将测试不通过的显示出来

#### remote package

上面讨论的都是源码在本地src下，go get命令可以拉去远程包，
`go get = git(版本库下载) + go install`，
如果本地已经有仓库了，那就不会去下载，直接使用本地仓库，
对于import来说，本地和远程并没有什么区别，都是：
import "github.com/63isOK/golearn/stringutil".

这个特性，会让自己写的package更容易让别人使用
