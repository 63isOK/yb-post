# 内置函数

这些内置函数都是预先声明的。

这些内置函数并没有标准的Go类型，她们只能出现在调用表达式中。
不能作为函数值。

## 关闭发送信道

    func close(c chan<- Type)

- 不再向信道c发送数据了
- 如果c是一个只接收信道，会报错
- 如果对一个已关闭的信道执行关闭或发送操作，会报运行时异常
- 关闭一个nil信道，会包运行时异常
- 在信道执行了close()，里面的元素被接收完之后，再次接收会返回零值，不阻塞的
- 利用多值接收操作，可判断一个信道是否已关闭

接收操作，只要能从信道内接收到值，就不会认为信道是关闭的。

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

长度：

- string，字符串中的字节数
- 数组/数组指针，数组长度
- 切片，切片长度(元素个数)
- map, key的个数
- 信道，缓冲信道中元素个数

容量：

- 数组/数组指针，数组长度
- 切片， 切片容量(要计算底层数组超过切片的部分)
- 信道，缓冲信道的容量

len(string)是常量。
len(数组/数组指针)，cap(数组/数组指针)都是常量。
len和cap的参数不能包含接收操作，不能包含非常量的函数调用。

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

不适用于slice/map/channel

## slice/map/channel的申请

make，申请之后，还会初始化为零值

## 切片的追加和拷贝

    append(s S, x ...T) S  // T is the element type of S

追加是支持可变参。

    copy(dst, src []T) int

拷贝就是将源切片的元素复制到目的切片，返回复制元素的个数。

复制元素的个数不能超过src/dst的最小容量

    copy(dst []byte, src string) int

拷贝还支持字符串

## map元素的删除

    delete(map,key)

删除map中的map[key]，如果map中不存在指定的key或map为nil，则delete不执行任何操作。

## 复数

`因为暂时不会用到复数，所有不分析`

## 异常处理

panic()/recover() 用于协助报告和处理运行时异常/程序自定义错误

    func panic(interface{})
    func recover() interface{}

在函数f中，显示调用panic或运行时异常，都会导致函数f结束执行，
之后就是执行延时函数。然后报告给上层调用者，依次走到协程的最顶层，
之后程序结束，报告错误，这个过程被称为panicking。

    panic(42)
    panic("unreachable")
    panic(Error("cannot parse"))

recover函数允许程序接管协程的panicking行为。

A函数有个延时函数B，A出现异常，B调用了recover来接管panicking行为。
B中recover的返回值是interface{},这个返回值会传给panic的调用者A。
如果返回值是正常的(没有新建一个panic)，那panicking序列会停止。
此时从A到异常出现之间产生的状态都会被丢弃，然后重新执行。
延时函数B后面要执行的其他延时函数会执行，之后返回A的调用者。

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

不关心