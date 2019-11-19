# 友好api的functional option模式

发表时间：2014/10/17

半年前，Go的三巨头之一发表了利用闭包来优雅处理接口多参数的问题。

前一篇介绍了如何用闭包来优雅地处理函数的option问题。
达到了两个目的：第一是无需太多api来实现;第二是对于option的扩展要简单。

其中用到(自引用)函数值来返回一个闭包而实现。

这篇文章，进一步利用这种套路来解决更多问题，并将这种讨论称为functional option模式

## 说明

这是14年的一次演讲稿

## 故事的开头

下面假设几个小的场景来推动故事的发展。

我们接到一个分布式社交软件的开发，关键组件可能是这么实现的。

    package abc

    import "net"

    type Server struct {
      listener net.Listener
    }

    func NewServer(addr string) (*Server, error) {
      l, err := net.Listen("tcp", addr)
        if err != nil {
          return nil, err
        }

        srv := Server{listener:l}
        go srv.run()

        return &srv, nil
    }

    func (s *Server) Addr net.Addr
    func (s *Server) Shutdown()

逻辑简单，也易用。一个协程处理一个将到来的请求。

## 发布第一个测试版后，新的需求来了

意味着问题：

- 比较慢的客户端会用尽我的资源
- 支持tls吗
- 能限制客户端的数量吗
- 有人使用dos攻击，我能限制单个ip的连接数吗

移动客户端的响应一般都会慢一点，或者干脆停止响应。
此时我们需要支持主动断开慢客户端。

在有安全指标的环境中，bug跟踪要满足这些安全连接的需求。

还需要报告短时间内，是谁用了我们的资源，还需要限制并发数。

接下来还要限制僵尸网络并发请求的速率。

当上面的都做到了，我们的接口可能改为下面这样了。

    func NewServer(add string,
                   clientTimeout time.Duration,
                   maxconns, maxconcurrent int,
                   cert *tls.Cert)

这个函数的签名并不是很完美，因为每次随需求变更都是不向后兼容的修改。
这中签名不优雅，且不向后兼容，这副作用就有点大了。

在api设计的理念中，这样的扩展方式是坏味道。麻烦，脆弱，且不易被发现。

包的使用者(新使用者)，通过这样的签名，无法知道哪些参数是可选的，哪些是必选的。

另外并不是所有使用者都对所有的参数有兴趣，万一对最大连接数不感兴趣，
那么是设置为0吗，合理吗，会不会设置为0之后，就没人能连上了。

虽然这个例子有点极端，但也说明了这类api写法的脆弱。

## 换另一把刀来试试

    func NewServer(add string) (*Server, error)
    func NewTLSServer(add string, cert *tls.Cert) (*Server, error)
    func NewServerWithTimeout(add string, timeout time.Duration) (*Server, error)
    func NewTLSServerWithTimeout(add string, 
                                 cert *tls.Cert
                                 timeout time.Duration) (*Server, error)

通过提供多个构造api来解决问题

虽然能解决一部分问题，可是，这些组合完全不够。万一调用者的需求在这些api之外呢。

## 要不要将api和配置结合起来

    type Config struct {
      Timeout time.Duration
      Cert *tls.Cert

      // 甚至 最大连接数，最大并发数
    }

    func NewServer(addr string, config Config) (*Server, error)

常见的解决方案，也很有效。

随着时间的增长，Config的结构也会扩展。而对外暴露的api是无需变动的。

配合良好的文档，威力强大。

成也萧何，败也萧何，配置中的默认值(或零值)让不感兴趣的字段不用操心的了，
如果调用者直到零值的意思，那不关心端口的人，是设置一个0端口呢，
还是使用默认端口8080呢。

大多数情况，api的编写者期望调用者按默认行为使用。
有时即使调用者不关心所有的可选参数，依然是要传第二个参数的。

可能还需要通过测试例子来看空值表示什么。

为什么调用者要构造一个空值，难道仅仅是为了保持api签名的一致。

    srv, _ := NewServer("localhost", nil)   // 使用默认值
    config := Config{Port:8000}
    srv2, _ := NewServer("localhost", &config)
    config.Port = 9000                      // 此时会发生什么事

空值的解决方法是，Config传指针，这样nil表示空值。

这种方法的问题多多，甚至比上一中多构造的问题还多一点。

- nil和空值有区别吗
- 包的设计者和调用者能对Config达成统一的共识吗
- 如果一个新的调用请求中，Config少了几个字段，如何处理

好的api，不会强求调用创建一些虚拟值来满足某些罕见的用例。

作为一个Go的信徒，永远不要在公开的函数中，对一个nil有需求。
当我们需要传递配置信息时，她应该是自解释，且有表达力的。

我们也值得拥有更好的解决方案

## 可变参配置

    func NewServer(addr string, config ...Config) (*Server, error)

    func main() {
      srv, _ := NewServer("localhost")
        srv2, _ := NewServer("localhost", Config{
          Timeout: 100 * time.Second,
          MaxConns: 10
        })
    }

为了解决上面那个强制性问题(经常未使用，但还要配置)，将函数签名改为可变参。

这种方案，我们不需要关心nil和零值，可变参，不传，就表示我们不关心。

现在解决了两个大问题：

- 对默认行为大的调用要尽可能简洁
- 现在函数签名接收的是Config，不是指针，好处是没有nil，其次调用者不持有内部值

这种解决方案也不是完美的，从签名上看，预期提供的Config不止一个，
在函数实现时需要考虑多种组合情况，严重的是有突出的信息。
eg：两个Config都指定了Port，还不一样。

有没有一种方法，使用可变参签名，且提高配置参数的表达能力呢

## functional options 模式

    func NewServer(addr string, options ...func(*Server)) (*Server, error)

    func main(){
      srv, _ := NewServer("localhost")

        timeout := func(srv *Server){
          srv.timeout = 60 * time.Second
        }

        tls, _ := func(srv *Server) {
          config := loadTLSConfig()
          srv.listener = tls.NewListener(srv.listener, &config)
        }

        srv2, _ := NewServer("localhost", timeout, tls)
    }

这种方案是通过函数值来实现，对于api签名来说，调用者决定关心哪些可选项，
这种表达力比上面几种方案强多了。

这种方案的火花来至于Go三巨头之一的rob pike，详情可看上一篇。

这种方案和前面所有方案的区别，可从签名上看出，目前方案的参数是函数值，
通过函数值来处理前面说到的变量，这些函数值的参数都是Server本身。
前面所有的方案，可选参数都是存储配置值的。

可变参签名的好处是，让默认分支的行为更加紧凑。

可以看到，函数列表中的每个函数只修改一个字段

下面看看NewServer的实现

    func NewServer(addr string, options ...func(*Server)) (*Server, error) {
      l, err := net.Listen("tcp", addr)
        if err != nil {
          return nil, err
        }

        srv := Server{listener:l}

        for _, option := range options {
          option(srv)
        }

        return &srv, nil
    }

只有在函数值列表中出现了的，才会修改Server里的状态。

这种方式设计的api，有以下好处：

- 默认就是合理的
- 灵活的配置性
- 随时可扩展
- 表达力强，对调用者来说，无需文档
- 对新人(第一次使用接口的人)来说，设计是安全的，哪些是必选，哪些可选一目了然
- 对nil和空值没有需求

对于可选参数来说，这种模式是ok的，但威力还可以提升，后续的文章会补充

## 总结

下面的过程可以使用functional option模式

    下面是多个接口

    创建对象

    设置对象的属性1
    设置对象的属性2
    设置对象的属性3
    设置对象的属性4

可套用functional option模式

    创建对象(必选参数，函数值列表)

这样的好处是：

- api随时间可以扩展，签名保持不变(向后兼容的基础)，这是要保证的最基本规则
- 默认分支是最简单的，就是所有可选参数都没的情况
- 有意义的配置参数(对调用者来说，表现力强)
- 控制复杂值的初始化

本文中，介绍了很多种配置模式，有些在当今也非常常用，我们在每种模式下都应该问：

- 可以更简单一点吗
- 参数是比选的吗
- 签名让函数更加简单吗，更加安全吗
- api包括陷阱或误导吗
