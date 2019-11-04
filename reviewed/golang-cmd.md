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


