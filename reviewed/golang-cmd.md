# go命令行介绍

    go help

    Go is a tool for managing Go source code.

    Usage:

      go <command> [arguments]

    The commands are:

      bug         start a bug report
      build       compile packages and dependencies
      clean       remove object files and cached files
      doc         show documentation for package or symbol
      env         print Go environment information
      fix         update packages to use new APIs
      fmt         gofmt (reformat) package sources
      generate    generate Go files by processing source
      get         add dependencies to current module and install them
      install     compile and install packages and dependencies
      list        list packages or modules
      mod         module maintenance
      run         compile and run Go program
      test        test packages
      tool        run specified go tool
      version     print Go version
      vet         report likely mistakes in packages

    Use "go help <command>" for more information about a command.

    Additional help topics:

      buildmode   build modes
      c           calling between Go and C
      cache       build and test caching
      environment environment variables
      filetype    file types
      go.mod      the go.mod file
      gopath      GOPATH environment variable
      gopath-get  legacy GOPATH go get
      goproxy     module proxy protocol
      importpath  import path syntax
      modules     modules, module versions, and more
      module-get  module-aware go get
      module-auth module authentication using go.sum
      module-private module configuration for non-public modules
      packages    package lists and patterns
      testflag    testing flags
      testfunc    testing functions

    Use "go help <topic>" for more information about that topic.

## 反馈一个bug

    go bug

## 编译一个包，及其依赖

    go build [-o output] [-i] [build flags] [packages]

- 依据import导入路径，找到依赖，将依赖和包编译出来，此时不会安装。
- 如果要编译的源文件在一个单独的目录下，这个所有目录的都被会当成一个包的文件
- 编译会忽略"\_test.go"结尾的文件
- 编译后的可执行文件名，可能是第一个源文件名，可能是目录名，后缀依平台而定
- 如果编译多个包或是一个非main包，不会产生可执行，仅仅作为检查包是否可以编译
- -o 选项可指定输出可执行的名字，当然包含的目录要存在
- -i 选项表示编译的同时，还安装

至于build flags， build/clean/get/install/list/run/test都用得上：

- -a 强制重新构建不是最新的包
- -n 打印命令，但不执行
- -p n 一个命令执行多个程序，n表示程序的数量。eg：通知执行test和build，并发。
- -race 启用竞争检测，只在64位系统上支持
- -msan 启用和内存清理的交互，64位系统，c编译器是clang/llvm时才支持
- -v 打印编译包的名字
- -work 打印临时目录名，退出时不自动删除
- -x 打印命令
- -asmflags '[pattern=]arg list'，go tool中传递的asm调用参数
- -buildmode mode，使用的构建模式
- -gccgoflags '[pattern=]arg list'
- -gcflags '[pattern=]arg list'
- -installsuffix suffix
- -ldflags '[pattern=]arg list'
- -linkshared
- -mod mode
- -pkgdir dir 指定包安装和加载的地址
- -tags tag,list
- -trimpath
- -toolexec 'cmd args'

看了下，大部分情况下都会用到上面的参数

## 删除一个包，包括缓存

    go clean [clean flags] [build flags] [packages]

## 显示一个包或符号的文档

    go doc [-u] [-c] [package|[package.]symbol[.methodOrField]]

这个和godoc还是有点不一样的，godoc会用h5来显示，go doc直接打印出结果

## 显示go的环境变量信息

    go env [-json] [-u] [-w] [var ...]

## 使用新api更新包

    go fix [packages]

## 格式化源码 gofmt

    go fmt [-n] [-x] [packages]

## 通过源码生成go文件

    go generate [-run regexp] [-n] [-v] [-x] [build flags] [file.go... | packages]

## 给当前module添加一个依赖，并安装

    go get [-d] [-t] [-u] [-v] [-insecure] [build flags] [packages]

这个命令使用的非常频繁。

这个命令做得到第一件事添加依赖。

每个包都有一个版本号，类似 v1.1.1的tag，
如果没有tag，就找预发布版本，类似 v0.0.1-pre1,
如果这些都没有，就找最近一次提交。

如果依赖指明了具体的版本，就找指定版本，如果没有就取当前版本。

可以通过@来指定依赖的版本，eg：'go get golang.org/x/text@v0.3.0'。

如果有源码管理工具，可能就有提交hash/分支标记/其他标记，
也可以使用'go get golang.org/x/text@master'，分支名不能重叠，
eg：@v2指v2开头的版本，而不是v2分支。

指定一个版本@v1.1.1,目前依赖有两个版本v1.1.0和v1.1.3，
最后会选择v1.1.3，这里考虑到了向前兼容。

- -t 表示会运行包的测试文件
- -u 会更新依赖的版本
- -u=patch，表示使用release优先

这个命令做的第二件事就是下载依赖，构建/安装

## 编译并安装包和依赖

    go install [-i] [build flags] [packages]

可执行会丢在GOBIN下，就是GOPATH/bin。

不开启go module，会安装到GOBIN，开启go module，最后包会被构建，
被缓存，但不是安装。

- -i表示安装

## 显示包或module

    go list [-f format] [-json] [-m] [list flags] [build flags] [packages]

这个命令非常有用，可显示指定包的依赖关系，
也可安装包分组来分析，目前有builtin和std两种。

## module 维护

    go mod <command> [arguments]

go mod已经渗透到go命令的方方面面了。

commands包括：

- download    下载module到本地，进行缓存
- edit        编辑go.mod
- graph       显示模块的依赖图
- init        在当前目录初始化一个新的module
- tidy        添加遗失的模块信息，删除无用的模块信息
- vendor      制作第三方依赖的副本
- verify      校验依赖，是否完整正确
- why         解释为啥需要包或module,这个会解释哪个包用到了指定模块

## 编译并运行go程序

    go run [build flags] [-exec xprog] package [arguments...]

不安装，运行完之后，会自动将编译的文件删除掉。

## 对包进行测试

    go test [build/test flags] [packages] [build/test flags & test binary flags]

go test会先查找\_test.go结尾的文件，这些文件里可能包含了测试函数，
基准测试函数，或是example函数。

\_test.go文件会被单独编译成一个包，并链接，并和可执行二进制一起进行测试。

go tool会自动忽略一个叫testdata的目录，并从这个目录提取数据作为辅助数据，
给其他测试用。

go test 会先调用 go vet来查看是否有依赖确实的问题，如果有，就不会执行后面的了。
当然，这个go vet测试是可以关闭的。

测试结果和总结会输出到标准输出。

go test有两种运行方式：

- 本地目录方式
  - 这种方式，运行命令中不带包信息，eg： go test -v .
  - 不启用缓存
- 包列表方式
  - go test命令中带了明确的包信息
  - eg: go test ./...
  - eg: go test math

只有在包列表模式下，才会启用缓存机制，如果使用了缓存，总结处会显示cached，
而不是花了多长时间。缓存匹配的条件是测试同一个可执行，flag都一样。

## 运行指定工具

    go tool [-n] command [args...]

## 打印go版本

    go version [-m] [-v] [file ...]

## 报告包中可能存在的错误

    go vet [-n] [-x] [-vettool prog] [build flags] [vet flags] [packages]

## 构建模式

## go和c的互调

- cgo工具
- swig程序

## 构建和测试的缓存

GOCACHE，GODEBUG

## 环境变量

## 可识别的文件类型

- .go go源码
- .c .h c源码
- .cc .cpp .cxx .hh .hp .hxx c++yuanma 
- .m objective-c源码
- .s .S 汇编源码
- .swig .swigcxx swig定义文件
- .syso 系统对象文件
