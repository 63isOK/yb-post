# go v1.13更变

## 变更介绍

v1.13发布于2019.09,距上一个版本的发布有6个月，
大多数变更集中在toolchain/runtime/libraries.

这个版本的go命令会下载google校验过的模块。总之module功能更加强大了。

## 语言变更

- 支持更多的字面量前缀

## 移植

v1.13是Native Client上运行的最后一个发行版本

## 工具

### module

GO111Module自动开启。

新环境变量GOPRIVATE指定非公共模块路径。
