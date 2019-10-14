# grpc中的概念和术语

## 服务定义

  利用pb定义一个service，在上篇文章中已经提过，
下面介绍4中不同的服务：

```proto
  rpc a(param) returns(response){}
  rpc b(param) returns(stream response){}
  rpc c(stream param) returns(response){}
  rpc d(stream param) returns(stream response){}
```

  a：一次请求一次响应，就像普通的函数调用  
  b：一次请求多次响应，响应全部发到client后，调用结束  
  c：多次请求一次响应，请求全部发到server后，开始处理  
  d：双向流式请求和响应，里面发送接受次序可以自定义

## api surface

  一般grpc的使用者是这么使用的：  
proto文件中定义一个service，在client调用定义的api，
在server实现api。

  在服务端是这样的：  
实现services，运行一个服务来处理客户端的调用请求。
grpc库的基础设施负责对调用请求解码、执行srevice方法、
将响应进行编码。

  在客户端是这样的：  
客户端有一个逻辑模块叫stub(属于客户端的一部分，
可由多种语言来实现)，stub实现了一个相同的方法(
pb文件中定义的方法)，在客户端应用程序中调用方法，
其实会调用stub的这个方法，stub中再调用服务端的方法。

  同步和异步  
同步调用会阻塞，直到得到响应，但网络本来是异步的，
所以在很多场景下，异步也是非常适用的。

  grpc提供了同步和异步的调用。

## rpc的生命周期

  上面说了4种rpc定义的方式，下面也按这个来讲

### 一元rpc

  上面的rpc a：  
  client发起一个调用call，
server会收到call时会收到一下信息：
客户端的元数据，调用的方法名，和超时  
  server要么立马回发一个元数据，
要么等client把请求发过来  
  一旦server收到请求后，会创建并填充响应对象，
如果调用执行成功，会返回状态信息
(状态码和可选的状态信息)，和可选的`尾部元数据`  
  如果状态是OK，client会获取响应，调用结束

  这是一个完整的闭环

### 服务端流式rpc

  上面的rpc b：  
  和一元rpc调用类似，只是等所有的响应信息
全部发送完之后，才发送状态信息和`尾部元数据`

### 客户端流式rpc

  上面的rpc c：  
  一般server会接受所有的请求信息再响应，
也有接受部分请求数据就返回的。

### 双向流式rpc

  上面的rpc d：  
  这个主要是用来自定义的，请求和响应的顺序任意

### 超时

  client可以指定rpc调用的超时

### rpc终结

  因为client和server都是独立的，判断成功的标准也
不一致，就容易出现rpc终结

### rpc取消

  client和server都可以取消rpc，任何时候  
  取消不会对操作进行回滚

### 元数据

  prc调用相关的一些信息，kv键值对列表，
key是string，value是string或二进制，一般用于认证信息

### channels

  一个通道标识client到server的一个连接，
创建client stub时使用这个连接

  client可以指定channel参数，来修改grpc的默认行为，
例如：消息是否压缩。  
  channel是有状态的：connected和idle
