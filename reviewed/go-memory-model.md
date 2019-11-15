# Go 内存模型

2014/05/31 版本

## 介绍

内存模型，说的是如何保证在多个协程在写一个变量的条件下，一个协程可以正确的读。

## 忠告

多个协程并发修改同一个数据，程序需要将这种访问方式进行序列化(说白了就是顺序访问)

序列化访问，可以用channel操作来保护数据，或使用sync sync/atomic包来做原子操作。

别自作聪明，Go里实现的可能和其他语言是有区别的。

## 发生之前

单协程的情况，读写就按执行顺序来。编译器和处理器可能会重新排序执行过程，
不过这种排序都会依照spec，并不会修改程序的行为。

对于单协程，w/r表示对变量v的读写操作

满足下面的条件，就说写是对读可见的

- 读不是发生在写之前
- 写之后，读之前，并没有其他的写操作

某一个写是对读可见，就要保证这个读只能见到这一个写，即要满足以下条件：

- 写在读前
- 其他的写要么发生在这个写之前，要么在读之后

后面这组条件比前面那组条件更加严格。她要求没有其他写并发在我们讨论的读写过程中。

单协程情况下，是没有并发的，所以上面两组条件是相同的。
如果是多协程，就需要用同步事件来保证满足上面的条件。

零值初始化，在内存模型中就是一个写行为

变量的读写(这个值比一个机器字大)的行文，可以理解为多次读写机器字(顺序无所谓)

## 同步

### 初始化

在程序的初始化和执行一节，也提到了程序的初始化是在一个协程里。
但过程中是可以创建新的协程的(init()可能会新建协程)，这就会产生并发。

a包引入b包，b包的init()会先执行，比a包所有的初始化都要先。

main.main()是在所有的init()都完成之后，才会执行。

### 创建协程

go语句就能新建协程

    var a string

    func f() {
      print(a)
    }

    func hello() {
      a = "hello, world"
      go f()
    }
    // calling hello will print "hello, world" at some point in the future 
    // (perhaps after hello has returned).

## 协程的销毁

协程的函数执行完就会退出，无法保证协程会在xx事件之前退出。

    var a string

    func hello() {
      go func() { a = "hello" }()
      print(a)
    }
    // 无法保证hello()一定会打包hello

如果一个协程必须要另一个协程做完事再处理(类似写对读可见)，
那就需要使用到同步机制，要么是锁，要么是信道机制。

## 信道通讯

Go中协程同步的主要方法是信道。

第一：信道能保证，发送操作会发生在接收操作之前。

    var c = make(chan int, 10)
    var a string

    func f() {
      a = "hello, world"
      c <- 0
    }

    func main() {
      go f()
      <-c
      print(a)
    }

第二：关闭信道，接收，会返回一个零值，因为信道已经关闭了

第三：从一个非缓冲信道接收，要在发送之前。(先准备好接收，再发送，之后执行接收)

第四：从容量为c的信道中接收第k个元素，要发生在第c+k发送之前

这是将第三规则扩展到缓冲信道，允许在缓存信道中构建一个计数信号模型：
信道的元素数量，对应有效使用量，信道的容量，对应最大并发使用数，
发送一次，信号加1,接收一次信号减1,这是一种常见的并发数限制手段。

    var limit = make(chan int, 3)

    func main() {
      for _, w := range work {
        go func(w func()) {
          limit <- 1
          w()
          <-limit
        }(w)
      }
      select{}
    }
    // 好处是并发数是3
    // 但是for循环，导致会有很多协程在阻塞，等待执行

## 锁

利用sync.Mutex sync.RWMutex 等来进行原子同步

sync.Once将锁释放也封装了，本质上还是锁

## 不正确的同步

    var a, b int

    func f() {
      a = 1
      b = 2
    }

    func g() {
      print(b)
      print(a)
    }

    func main() {
      go f()
      g()
    }

上面没有同步。

    var a string
    var done bool

    func setup() {
      a = "hello, world"
      done = true
    }

    func doprint() {
      if !done {
        once.Do(setup)
      }
      print(a)
    }

    func twoprint() {
      go doprint()
      go doprint()
    }

也没有正确的使用锁

    var a string
    var done bool

    func setup() {
      a = "hello, world"
      done = true
    }

    func main() {
      go setup()
      for !done {
      }
      print(a)
    }

这个也没有理解同步的概念，可能会导致for空转很多次，性能上是不允许的。

    type T struct {
      msg string
    }

    var g *T

    func setup() {
      t := new(T)
      t.msg = "hello, world"
      g = t
    }

    func main() {
      go setup()
      for g == nil {
      }
      print(g.msg)
    }

跟上面的类似，setup执行完之前，for会空转

所以说，显示使用同步原语是必须的。

`基于通信而共享内存(使用信道机制)，而不是基于共享内存而通信(锁机制)`
