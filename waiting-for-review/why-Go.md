# Go的魅力

## devops工程师如何对待Go

到目前为止,大多数devops工具都是用Go写的.
docker,kubernetes,terraform(一个devops工具),
vault(敏感信息访问工具),prometheus,helm,loki,grafana,
都是用Go写的.

高效,易复用,简单,健壮都是Go的特点.

## trivago全面使用Go

trivago是一个酒店软件,他们遇到以下问题

### 竞争检测

并发高,请求的超时管理和共享资源的管理称为难题.

Go中的超时,取消,上下文机制和并发,正好能解决这些问题.

有时候出现竞争是因为某些函数不是可重入的,或者说这块没设计好,
加锁又会导致整个系统的性能下降.

Go官方提供了工具,添加构建参数 -race就可以做竞争检测了.

### 二进制文件是静态链接的

编译Go程序时,不需要解释器或虚拟机,
如果不适用cgo,那么发布就一个文件,很简单了.
如果再配合docker,只用一步就可以搞定所有流程,
甚至连docker的alpine或debian的slim版本都不用,
直接使用空镜像scratch更加简洁.这样最后的docker镜像就更小了.
