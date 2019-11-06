# 命令工具文档

前面介绍了go命令，她主要用于处理对Go源码的构建和处理。

gofmt和godoc单独出来的原因是她们的使用实在是太频繁了。

下面介绍的都是通过go tool 调用的。

## go命令

前面介绍了的，处理源码并调用其他命令的入口

## cgo

Go和c相互调用，暂时不需要了解

## cover

这是一个分析覆盖率的工具，
go test -coverprofile=cover.out 可生成profile文件

也可以直接使用 go test -cover

也可以生成profile之后，再生成一个网页

## fix

    go tool fix [-r name,...] [path ...]

如果Go发布新版本，可以用这个命令将以前老的api替换成新的

## vet

用go vet 可以检查一下可以的结构或是闭包效应等

## gofmt

一般都是自动格式化代码，并不需要过多关心

## godoc

godoc -http=172.17.0.2:8000

本地开一个web服务，用来显示各个包中的信息.
如果每个包写的注释否符合，就很完美
