# 别害怕Go的一等公民 function

发布时间：2016/11/13

两年之后，由对functional option模式有了新的理解

grpc项目大量引入functional option模式，当然是改进版本，
下面就是介绍这个。说白了就是为小白进行科普。

## 重新定义functional option模式

    type Config struct{ ... }

    func WithReticulatedSplines(c *Config) { ... }

    type Terrain struct {
      config Config
    }

    func NewTerrain(options ...func(*Config)) *Terrain {
      var t Terrain
      for _, option := range options {
        option(&t.config)
      }
      return &t
    }

    func main() {
      t := NewTerrain(WithReticulatedSplines)
      // [ simulation intensifies ]
    }

可以看到，和之前的模式变化并不大，只是将配置单独抽出来，放在Config中。
而且option函数并没有处理参数。这点正是别扭的部分。

     // WithCities adds n cities to the Terrain model
    func WithCities(n int) func(*Config) { ... }

    func main() {
      t := NewTerrain(WithCities(9))
      // ...
    }

这就是带参数后的写法，此时有个问题：option函数的签名不能统一

    func WithRed(a, b, c int) func(*Config) { ... }

既然option的签名不能统一，那option函数的入参就不能像普通函数一样传递。
解决方法是传递参数，将结果传递给构造函数，而不是将方法值丢给构造函数，
在构造函数去做计算。

## 函数是Go的一等公民

要理解上面最后一段话，先认识几个概念。

本质上，对函数计算完会返回一个结果。

普通类型的计算，返回的也是内置类型，是个普通函数

    package math

    func Min(a, b float64) float64

再看一个返回其他类型的，接口指针

    package bytes

    func NewReader(b []byte) *Reader

当然函数值也能作为函数的返回值

    func WithCities(n int) func(*Config)

这个返回的是一个函数值，函数值的签名是：入参是一个Config指针。
通过函数名(包括匿名函数)，我们可以将一个函数值当成常规值来用，
这就说明了函数在Go中是一等公民。

## 一等公民的特权

现在说的是一等公民能做什么，不是讲非一等公民不能做什么，
也不是讲一等公民能做的所有事。

函数值可以进行赋值，和内置类型int/string一样，也可以为他们定义类型别名，
甚至一等公民是可以实现接口的，为一等公民附加个方法也是平常事。

## 作为一等公民的函数，在functional option模式中，如何贡献力量

之前的模式中，option函数的签名是一个函数，我们在构造时要新加一个可选参数，
就会创建一个具体的函数，这个函数符合option函数的签名。

那可以进一步尝试，先将option函数签名改为接口。函数是一等公民，
当然也可以有方法，这些方法只要实现了option接口就可以了。

    type Option interface {
      Apply(*Config)
    }

构造基本不变

    func NewTerrain(options ...Option) *Terrain {
      var config Config
      for _, option := range options {
              option.Apply(&config)
      }
      // ...
    }

只要构造函数的入参实现了Option接口，就可以将入参存在接口变量，
通过接口来调用指定的方法。

到此，使用option函数签名和使用接口，作用都是一样，
一个是用函数签名来匹配，一个是通过接口来匹配。
到目前为止，除了升级匹配规则，其他都没变。
构造从以前传函数切片到现在的传接口切片。

下面看看，使用接口之后，具体的可选参数应该如何处理

    // 定一个类型，实现Option接口，最后返回接口值
    // 返回的接口值作为构造函数的入参，在构造函数中调用方法
    type splines struct{}

    func (s *splines) Apply(c *Config) { ... }

    func WithReticulatedSplines() Option {
      return new(splines)
    }

如果可选参数还带了附加信息，可以如下编写

    type cities struct {
      cities int
    }

    func (c *cities) Apply(c *Config) { ... }

    func WithCities(n int) Option {
      return &cities{
        cities: n,
      }
    }

实际情况中，机会每个可选参数都会附加一些业务信息，即使是开关信息。
就像上面例子中的n。不过这个参数是通过组合字面量再丢给接口值的。
对应的，我们在调用构造函数时，也需要做一些处理。

    func main() {
      t := NewTerrain(WithReticulatedSplines(), WithCities(9))
      // ...
    }

有个人叫Tomás Senart，她的演讲中提到了，单方法的接口和函数签名，
基本上没啥区别，她们都可以完成相同的事，用的也是相同的方法，
唯一不同的是，她们使用不同的匹配方式(一个通过签名，一个通过语言机制)。

上面的例子，通过签名和通过接口实现的functional option模式，就是一个典型的例子。
不过，总有不同，通过一等公民函数来实现，代码量少了一些。
通过接口，就必须有一个类型，通过这个类型来实现接口。

## 函数就不能做的更多吗

在functional option问题中，函数作为一等公民，在功能上和接口实现的都一样，
仅仅在代码量上的微弱优势并不能让她脱颖而出(大多数人都会基于语言机制来实现)。
下面继续探讨一下在函数这条路上的探索。

封装。

调用一个函数或方法，我们要做的就是传数据进去，函数的任务就是解析数据，
做某些执行。

函数值允许我们将这种行为作为一个参数进行传递，之后再执行，
在传递函数值时，不涉及数据解析。
实际上，我们可以将函数值作为一等公民，任意传递，甚至在不同的上下文进行执行。

    type Calculator struct {
      acc float64
    }

    const (
      OP_ADD = 1 << iota
      OP_SUB
      OP_MUL
    )

    func (c *Calculator) Do(op int, v float64) float64 {
      switch op {
      case OP_ADD:
        c.acc += v
      case OP_SUB:
        c.acc -= v
      case OP_MUL:
        c.acc *= v
      default:
        panic("unhandled operation")
      }
      return c.acc
    }

这是一个用Do封装的算术计算函数，好处是简明，如果遇到扩展，Do和调用方都要改，
其次如果是开方或求幂，那么Do的参数就不够用了。

随着算法计算方法的增多，Do会膨胀的很厉害，我也可以换一种思路

    type Calculator struct {
      acc float64
    }

    type opfunc func(float64, float64) float64

    func (c *Calculator) Do(op opfunc, v float64) float64 {
      c.acc = op(c.acc, v)
      return c.acc
    }

将要操作的过程作为参数传进来，这就是参数值。

    func Add(a, b float64) float64 { return a + b }
    func Sub(a, b float64) float64 { return a - b }
    func Mul(a, b float64) float64 { return a * b }

    func main() {
      var c Calculator
      fmt.Println(c.Do(Add, 5))       // 5
      fmt.Println(c.Do(Sub, 3))       // 2
      fmt.Println(c.Do(Mul, 8))       // 16
    }

通过同样的函数签名来满足Do的参数。

扩展第一步，支持平方

    func Sqrt(n, _ float64) float64 {
      return math.Sqrt(n)
    }

    // 调用
    c.Do(Sqrt, 0) // operand ignored
    // 不好的就是：非得要一个我们不关心的参数0

扩展第二步，将参数和操作封装在一起，让Add返回一个函数值

    func Add(n float64) func(float64) float64 {
      return func(acc float64) float64 {
        return acc + n
      }
    }

    func (c *Calculator) Do(op func(float64) float64) float64 {
      c.acc = op(c.acc)

      return c.acc
    }

    c.Do(Add(10))

    // 好处是保证了Do参数中函数签名的一致
    // 这种思想，是通过返回函数值(再一层)来将不匹配的封装在一起
    // 那上层的就是一致的

到此，我们才说，Do不依赖Add函数本身，而是依赖Add的执行结果(Add(10)),
我们唯一需要保证的是Add的执行结果，
也就是内层函数值的签名和Do参数的签名是一致的，至于Add是3参数，还是300参数，
并无影响。

通过这种方式，具体函数的参数个数和类型，并不会影响我们了，
因为我们通过函数值，将不一致的全部封装在一起了。

    func Sqrt() func(float64) float64 {
      return func(n float64) float64 {
        return math.Sqrt(n)
      }
    }

    func main() {
      var c Calculator
      c.Do(Add(2))
      c.Do(Sqrt())   // 1.41421356237
    }
    // 就像这样
    // Do依然维护这不同的操作过程
    // 至于操作过程涉及的参数，和个数，并不再影响Do了
    // 因为Do不依赖操作本身，而依赖操作的结果
    // 操作的结果就是另一个函数值
    // 只要Do参数的签名和最内层的函数值签名保持一致，就ok了

这种新的方式，避免了语法上的尴尬。

还有一点，因为上面Sqrt的签名和math.sqrt的签名一致，所以还能这样：

    func main() {
      var c Calculator
      c.Do(Add(2))      // 2
      c.Do(math.Sqrt)   // 1.41421356237
      c.Do(math.Cos)    // 0.99969539804
    }
    // 直接传同样签名的函数值，也是ok的

我们从硬编码模型，慢慢转到了函数模型，一步步到最后，
计算函数Do和具体的计算函数，再也没有任何限制了。

## 再聊聊其他话题

我们来做一个聊天服务，从小处一步步开始。

新来一个连接

    type Mux struct {
      mu    sync.Mutex
      conns map[net.Addr]net.Conn
    }

    func (m *Mux) Add(conn net.Conn) {
      m.mu.Lock()
      defer m.mu.Unlock()
      m.conns[conn.RemoteAddr()] = conn
    }

断开一个连接

    func (m *Mux) Remove(addr net.Addr) {
      m.mu.Lock()
      defer m.mu.Unlock()
      delete(m.conns, addr)
    }

发送消息

    func (m *Mux) SendMsg(msg string) error {
      m.mu.Lock()
      defer m.mu.Unlock()
      for _, conn := range m.conns {
        err := io.WriteString(conn, msg)
        if err != nil {
          return err
        }
      }
      return nil
    }
    // 发送消息时，加锁，以减少数据竞争

这是否是我们平时用的写法？而且还使用了底层的原子操作，不是按Go推荐的来。

Don’t communicate by sharing memory, share memory by communicating.

按照这条规则，我们就使用协程和信道来重构

    type Mux struct {
      add     chan net.Conn
      remove  chan net.Addr
      msg     chan string
      conns   map[net.Addr]net.Conn
    }

    func (m *Mux) Add(conn net.Conn) {
      m.add <- conn
    }

    func (m *Mux) Remove(addr net.Addr) {
      m.remove <- addr
    }

    func (m *Mux) SendMsg(msg string) {
      m.msg <- msg
    }

    func (m *Mux) loop() {
      m.conns := make(map[net.Addr]net.Conn)
      for {
        select {
        case conn := <- m.add:
          m.conns[conns.RemoteAddr()] = conn
        case addr <- m.remove:
          delete(m.conns[addr])
        case msg := <- m.msg:
          for _, conn := range m.conns {
            io.WriteString(conn, msg)
          }
        }
      }
    }

这次没有通过锁来保证顺序访问，而是通过3个channel。
这次为啥不需要锁来保证对关键数据的访问顺序，因为关键数据map只在一个函数中变动，
loop()。

回头再看看通过信道实现的例子，loop中基本上硬编码，只做3件事：
连/断开/广播消息。如果现在新增一个功能，那我们需要在Mux新增一个channel，
在loop多加一个分支。这个套路和可选参数的硬编码模型是一模一样的。

一等公民函数是否能用在这呢(将硬编码模型转变成函数模型)

使用函数模型，第一步就是将连/断开/广播转换成行为进行传递。

    type Mux struct {
      ops chan func(map[net.Addr}net.Conn])
    }
    // 和之前构造函数的option函数列表类似，构造函数是通过切片来传递
    // 而这个场景因为要考虑同步，所以用的是channel
    // 不管是切片还是channel，里面的元素都是函数，函数参数是需要改变状态的对象
    // 之前是Config，现在是一个map，此处，map也能放在一个自定义类型中

    func (m *Mux) Add(conn net.Conn) {
      m.ops <- func(m map[net.Addr]net.Conn) {
        m[conn.RemoteAddr()] = conn
      }
    }

    func (m *Mux) Remove(addr net.Addr) {
      m.ops <- func(m map[net.Addr]net.Conn) {
        delete(m, addr)
      }
    }

    func (m *Mux) SendMsg(msg string) error {
      m.ops <- func(m map[net.Addr]net.Conn) {
        for _, conn := range m {
          io.WriteString(conn, msg)
        }
      }
      return nil
    }

    func (m *Mux) loop() {
      conns := make(map[net.Addr]net.Conn)
      for op := range m.ops {
        op(conns)
      }
    }

通过函数模型，将loop中复杂的逻辑拆分到各个匿名函数中。

相比硬编码模型，函数模型中发送消息时，少了一个error返回。

    func (m *Mux) SendMsg(msg string) error {
      ret := make(chan error, 1)
      m.ops <- func(m map[net.Addr]net.Conn) {
        for _, conn := range m.conns {
          err := io.WriteString(conn, msg)
          if err != nil {
            ret <- err      // 只要出错就报告

            return
          }
        }

        ret <- nil          // 无错就报告nil
      }

      return <- ret         // SendMsg的出口，支持报错
    }

新增一个功能，发送私聊消息，用函数模型，就这么优雅

    func (m *Mux) PrivateMsg(addr net.Addr, msg string) error {
      ret := make(chan net.conn, 1)
      m.ops <- func(m map[net.Addr]net.Conn) {
        ret <- m[addr]
      }   // 先找到指定地址的用户

      conn := <- ret
      if conn == nil {
        return errors.Errorf("client %v not registered", addr)
      }

      return io.WriteString(conn, msg)    // 私聊
    }
    // 这个版本需要并发处理

新增功能的过程，并没有修改loop函数，也不影响其他操作。

## 结论

- 一等公民函数的表现力很强大
- 一等公民函数并不是新东东，即使C语言也是支持的
- 和Go期铜的其他功能一样，一等公民函数的过度使用会导致复杂度的提升，
这点和channel过度使用一样，过度会导致复杂。要把握好什么是适度
- 每个Go的信仰者的工具箱里，都应该有一等公民函数
- 函数并不比接口难，只是使用时不那么合乎我们平常使用的方法
- 但花时间和精力掌握一等公民函数是值得的。就算仅仅是为了方便看懂其他人的代码
