# 内置函数

这些内置函数都是预先声明的。
从调用方式上看,和普通函数没什么区别.
有些内置函数,第一个参数是可接受类型,而不是值.

这些内置函数并没有标准的Go类型，她们只能出现在调用表达式中。
不能作为函数值。怎么理解?

    // 这么用可以,因为是调用表达式
    close(xxx)
    // 这么用不可以,因为内置函数不能作为函数值
    a := close

## 关闭信道

关闭channle,是指不能再发,收是可以的.

    func close(c chan<- Type)

- 不再向信道c发送数据了
- 如果c是一个只接收信道，会报错
- 如果对一个已关闭的信道执行关闭或发送操作，会报运行时异常
- 关闭一个nil信道，会包运行时异常
- 在信道执行了close()，里面的元素被接收完之后，再次接收会返回零值，不阻塞的
- 利用多值接收操作，可判断一个信道是否已关闭

接收操作，只要能从信道内接收到值，就不会认为信道是关闭的。
直到接受到一个零值.当然也可以通过多返回值来判断信道是否关闭.

    func test7() {
      c := make(chan int, 10)
      c <- 1
      c <- 1
      c <- 1
      c <- 1
      close(c)

      i, ok := 0, true
      for ok == true {
        i, ok = <-c
        fmt.Println(i, ok)
      }

      fmt.Println("============")
    }
    Output:
    1 true
    1 true
    1 true
    1 true
    0 false
    ============

## 长度和容量

长度len()：

- string，字符串中的字节数
- 数组/数组指针，数组长度
- 切片，切片长度(元素个数)
- map, key的个数
- 信道，缓冲信道中元素个数

容量cap()：

- 数组/数组指针，数组长度
- 切片， 切片容量(要计算底层数组超过切片的部分)
- 信道，缓冲信道的容量

cap并没有对map进行描述,也没有对string进行描述,
所以对于map/string,cap()是非法的.

len(string)是常量。
len(数组/数组指针)，cap(数组/数组指针)都是常量。
如果参数中不包含函数调用(或带接收者的方法调用),len/cap返回常量,
此时参数是不会被计算的;否则参数先计算,且len/cap的返回值不是常量.

    const (
      c1 = imag(2i)                    // imag(2i) = 2.0 is a constant
      c2 = len([10]float64{2})         // [10]float64{2} contains no function calls
      c3 = len([10]float64{c1})        // [10]float64{c1} contains no function calls
      c4 = len([10]float64{imag(2i)})  
                      // imag(2i) is a constant and no function call is issued
      c5 = len([10]float64{imag(z)})
                      // invalid: imag(z) is a (non-constant) function call
    )
    var z complex128

## 申请

new(), 运行时声明一个变量，返回指针，按零值初始化

这个特别适合申请结构体.
不适用于slice/map/channel

## slice/map/channel的申请

make，申请之后，还会初始化为零值,
make只服务于slice/map/channel,类型后面可以有可选参数,
返回值是T,不是\*T.

Call| Type T   |  Result
---|---|---
make(T, n)      | slice    |  slice,长度和容量都是n
make(T, n, m)   | slice    |  slice,长度n,容量m
make(T)         | map      |  map
make(T, n)      | map      |  map,申请了大约n个元素的内存
make(T)         | channel  |  channel,无缓冲
make(T, n)      | channel  |  缓冲channel,最大元素个数n

m/n非负,且m要大于n,不然就是运行时异常.
对于map的设置长度,spec虽然规定了,具体还是看实现咋做的.

## 切片的追加和拷贝

追加和拷贝时slice的基本操作.

    append(s S, x ...T) S  // T is the element type of S

追加是支持可变参。
append是可以接受第二个参数是切片,
只要第二个参数后面带上三个点.
这里面有个特殊情况:第一个参数是`[]byte`,第二个参数是string...

append()可能会导致新的slice产生

    copy(dst, src []T) int

拷贝就是将源切片的元素复制到目的切片，返回复制元素的个数。

复制元素的个数是min(len(src), len(dst)),

和append一样,copy也有一个特殊情况:  
dst是`[]byte`,src却是一个string.

    // 将string的byte拷贝到字符切片
    copy(dst []byte, src string) int

## map元素的删除

delete为map服务.

    delete(map,key)

删除`map[key]`，如果map中不存在指定的key或map为nil，则delete不执行任何操作。

## 复数

有3个内置函数服务于复数.

- complex(), 构造复数
- real/imag, 获取复数的实部/虚部

复数的实部虚部是浮点型,而且两个的浮点类型要是一致的,
不能一个是float32,另一个是float64.  
这样复数的类型就有complex64/complex128,分别对应32/64的浮点参数.
如果其中一个是无类型常量,那么就要转换成另一个参数对应的类型.
如果两个参数都是无类型常量,或者虚部为0,
那么组合成的复数就是无类型复数.

real/imag返回的类型和复数具体类型对应.

## 异常处理

panic()/recover() 用于协助报告和处理运行时异常/程序自定义错误

    func panic(interface{})
    func recover() interface{}

在函数f中，显示调用panic或运行时异常，都会导致函数f结束执行，
之后就是执行延时函数。然后报告给上层调用者(也会调用其中的defer函数)，
依次走到协程的最顶层，之后程序结束，报告错误，这个过程被称为panicking。

    panic(42)
    panic("unreachable")
    panic(Error("cannot parse"))

recover函数允许程序接管协程的panicking行为。

A函数有个延时函数B(B里调用了recover函数)，执行A函数的协程中出现异常，
只要执行走到了B函数,B会调用recover来接管panicking行为。
B函数的返回值会以值得方式传递给调用panic的调用者.
如果B是正常返回,没有出现新的panic,则panicking序列会停止.
B中recover的返回值是interface{},这个返回值就是panic的调用参数。
如果recover返回值是正常的(没有新建一个panic)，那panicking序列会停止。
此时从异常出现到A的剩下部分产生的状态都会被丢弃，
说白了就是recover之后,调用完B之前的derfer函数,这是正常执行。
延时函数B后面要执行的其他延时函数会执行，之后返回A的调用者。

所以所有一截是丢弃掉了的,就是异常到A的剩下代码段.

以下几种情况，recover的返回值是nil：

- panic的参数是nil
- 协程没有panicking
- recover并不是通过延时函数直接调用的

    func g(a int) (ret int) {
      ret = a + 1

      panic("hi")
    }

    func protect(f func(int) int) int {
      defer func() {
        fmt.Println("defer")
        if x := recover(); x != nil {
          fmt.Printf("%v\n", x)
        }
      }()

      fmt.Println("go")
      return f(3)
    }

    func test8() {
      ret := protect(g)

      fmt.Println(ret, "=========")
    }
    Output:
    go
    defer
    hi
    0 =========

    // 这是将函数g可能出现的异常捕获掉

## 自举 bootstrapping

为了完整性,有些内置函数很有用,但并不一定保证不会被丢弃.

print/println,这两个函数在自举时用到了,没有返回值.

如果我们要打印,还是使用fmt库好.
