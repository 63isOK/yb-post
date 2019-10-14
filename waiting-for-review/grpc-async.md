# grpc中的异步调用

  太多场景都是异步调用的，特别是rpc这个真实调用，
本身走的是网络，很多程序都是异步的，所以下面看看异步

## CompletionQueue

  grpc使用CompletionQueue来实现异步操作

* 绑定CompletionQueue到一个rpc调用
* 在读写时带一个唯一的tag，类型是void*
* 调用CompletionQueue::Next来等操作完成

## client

  在client，创建完channel和stub之后：

* 创建一个rpc对象，并初始化
* 开始请求，设置tag
* 等待下一个响应，通过tag判断是不是本次调用

## server

* 将异步service绑定到server
* 提供一个唯一的tag，请求一次rpc调用

  异步，完成队列还有一些地方没连起来，先查资料，
等后面再补全理解。
