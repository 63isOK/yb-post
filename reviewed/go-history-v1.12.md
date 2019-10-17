# go v1.12变更说明

[官方地址](https://golang.google.cn/doc/go1.12)

## v1.12介绍

发布时间：2019.02,距v1.11发布有6个月时间，
本次更新主要集中在toolchain/runtime/liraries.
依然兼容go v1.

## 对语言的更改

本次更新并未对go spec有任何更改

## 移植

竞争检测现在支持linux/arm64了.

gov1.12是FreeBSD 10.x的最后一个支持版本，
gov1.13会支持FreeBSD 11.2+/12.0+,
12.0+需要开启COMPAT_FREEBSD11选项(默认已开启)。

cgo现已支持linux/ppc64。

## 工具

### go tool vet 不再支持

    go vet 现在被重写了，用于代码分析的基础工具

### tour

go tour不再包含主要的二进制发行版本，
现在要在本地使用tour，需要手动安装

    go get -u golang.org/x/tour
    tour

### build 缓存

现在要构建缓存，需要先消除$GOPATH/pkg.
如果环境变量关闭(GOCACHE=off),会导致go命令写缓存失败

### 只有二进制包

v1.12是最后一个支持binary-only package的发布版本

### cgo

v1.12会将EGLDisplay(c类型)转换成uintptr(go类型)

### modules

当GO111MODULE设置为on时，go命令就可以在模块目录之外执行，
这样不需要import指定基于当前目录的相对目录，也不需要显式更改go.mod文件。

现在go命令下载额外的模块时，并并发安全的，GOPATH/pkg/mod在文件系统会被锁住。

go.mod文件中的go指令，表明了模块的版本，默认使用的go版本是v1.12.
go.mod文件中也可以指定比v1.12更高的版本，只有当编译失败时会提示版本不匹配。
如果某些模块指定了v1.11到v1.11.3/v1.11.4，用v1.12编译可能出错，
此时，可以将go.mod的go版本修改为指定版本.

### 编译工具链

编译器的实时变量有所提升。

### godoc 和 go doc

v1.12中，godoc不再是命令行接口，而仅仅是一个web服务，go doc作为命令行接口。
v1.12是最后一个还包含godoc的发行版本，v1.13中，需要通过go get获取godoc。

go doc支持-all 参数，这个参数是打印文档中所有暴露的api，和以前的godoc一样。

go doc支持-src 参数，这个参数是显示目标的源码

### trace 跟踪

跟踪工具现在支持绘制曲线，在分析gc或程序延时时，非常有用

### 汇编

在arm64中，R18寄存器改名为R18\_平台

## 运行时

v1.12在大部分堆仍在使用时，显著地提升了gc的性能。
这样减少了gc之后立马分配的延迟。

gc会更加积极地释放内存给操作系统，大块的申请并不会重用以前的堆空间

运行时的定时器和deadline更快了，cpu越多，可伸缩性越好。
实际上管理网络连接的deadline的性能有所提升。

linux上，MADV_FREE会释放不会用到的内存。

cpu.extension=off支持GODEBUG环境变量，windows还未支持。

v1.12解决了大堆分配过多的bug，所以内存分析会的准确性会提高

## 核心库

### tls 1.3

v1.12支持tls 1.3，crypto/tls包，遵循rfc8446,在环境变量GODEBUG添加tsl13=1可开启。

### 库的小改动

- bufio
  - 调用Peek后，UnreadRune/UnreadByte函数的返回新增了一个error
- bytes
  - 新增ReplaceAll函数，替换后，返回一个[]byte
  - 指向0值的Reader和NewReader(nil)是一个意思
- crpty/rand
  - Reader.Read如果阻塞1分钟，会在stderr打印一个警告
- crypto/tls
- database/sql
  - 在Rows和Row.Scan中提供了一个查询游标
- expvar
  - 新增一个方法Delete，用于删除Map的kv对
- fmt
  - map可按kv顺序打印，目的是为了简化测试
- go/doc
  - 有一个新的Mode bit
- go/token
  - File类型新增一个LineStart字段
- image
  - RegisterFormat并发安全了
- image/png
- io
  - 新接口StringWriter，封装了WriteString函数
- math
- math/bits
- net
  - Dialer.DualStack被弃用了
- net/http
  - http重定向到https，http服务会返回一个400 bad request
- net/url
  - url包含控制字符(null/tab/newline)时，
Parse/ParseRequestURI/URL.Parse会返回一个错误
- net/http/httputil
  - ReverseProxy会自动代理websocket请求
- os
  - 新增ProcessState.ExitCode方法
- path/filepath
- reflect
  - 新类型Maplter，是一个map的迭代
- regexp
- runtime/debug
  - 新类型BuildInfo
- string
  - 新增ReplaceAll
- syscall
- syscall/js
- testing
  - 基准测试功能扩展
- text/template
  - 执行模版时，长上下文的值不会被截断
- time
- unsafe
