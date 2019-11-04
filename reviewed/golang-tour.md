# tour go之旅

分三块：

* 基本语法和数据结构
* 方法和接口
* 并发原语

~~安装是 go get golang.org/x/tour,要翻墙，或者直接访问
<https://tour.golang.org/welcome/1>。

不翻墙的走： go get -u github.com/Go-zh/tour, 之后执行tour~~

学习tour最新的方法是

```shell
go get golang.org/x/tour

tour -openbrowser=false -http="172.17.0.2:8000"

// 访问 172.17.0.2:8000
```

## 介绍

go playground 是golang.org提供的一个web服务，
这个服务接受一个go程序，返回输出结果，中间的编译链接运行都是在沙箱中，

## basic

---

包 变量 函数

---

### package

前面也说了，package，每个go程序都是由package组成，
main包是程序运行的开始点，import，会导入其他的包，
包名和import路径最后一段是一样的，
一个package下的所有源码，第一句都是package 包名

### import

导入多个package如下:

```go
import "fmt"
import "math"
```

"因式分解"的导入：

```go
import (
  "fmt"
  "math"
)
```

官方推荐使用这种方式

### export

和import相反，import是导入package，export是导出，
是暴露属性和方法，首字母大写就是导出，小写不导出。

### function

go中的function和method是同一个东西，都是函数。
区别在于func属于作用域，method依附于某一数据结构。
在作用域中，func和数据结构是同一等级的。
函数可以有任意个参数，参数类型在参数名后面.

格式 func 函数名(参数名 参数类型, ...) 返回类型 {}

`ps: 不像c++里面，需要先声明再调用，go里面测试并无此限制`

多个参数的参数类型一致，可以简写为 参数1, 参数2 参数类型

`ps：这倒是和c++类似： int a, b, c;`, go是 a, b, c int

### result

返回值可以有多个

```go
    func swap(x, y string) (string, string){
        return y, x
    }

    func main() {
        a, b := swap("abc", "123")
        fmt.Println(a, b)
    }
```

返回值也能取名字，有名字的返回值就相当于函数中定义的变量,
只有一个return，叫裸return，用在返回值有名字的情况

裸return在'长函数'中，会影响可读性

### 变量

var声明变量，有package级别的，也有function级别的

`ps：对应c++的局部变量和全局变量`

变量的初始化：

```go
    var i, j int = 1, 2
    var a, b, c = true, "123", 50

    // 因式分解
    var (
        a bool = false
        b int =2
    )
```

初始化中，类型可以被忽略不写

在函数中，下面两句是一个意思：

```go
    func abc(){
        var a, b int = 1, 2
        a, b := 1, 2
    }
```

:= 叫短赋值语句，用来代替显式var声明， 也称为 短语句
难怪静态语言，可以使用动态语言的写法，go为了抓住用户还是下了很多心思，

function之外，所有的语句都需要有关键字(eg:var func 等)，
所以function之外，:=是不能有的

`ps:短赋值缺省了var和后面的类型，都是通过后面的值来推导出变量类型，
只能在function中使用，用起来很方便，只是需要注意一点：短赋值也是变量的声明，
如果之前已经声明了此变量，遇到短赋值，错误就不能很快的定位出来`

### 基础类型

* bool
* string
* int
* int8
* int16
* int32
* int64
* uint
* uint8
* uint16
* uint32
* uint64
* uintptr
* byte, uint8的别名
* rune, int32的别名，表示一个unicode code point
* float32
* float64
* complex64
* complex128

包含了布尔、字符串、整形、无符号整形、浮点型

32位系统中 int uint uintptr是32位，64位系统是64位

变量声明时不明确初始化的值，都会初始化为0,

* 数值类型 0
* 布尔类型 false
* 字符串类型 ""

### 类型转换

T(v) 将v转换成类型T

和c系列语言不一样，go中不存在隐式类型转换,都是显式转换

var和:= 语句中如果没有指明变量类型，那就从右边的语句中推到

### 常量

常量用const修饰，不能使用:=语句,因为常量都是在文件作用域下，
:=只能在func作用域下使用。

数值常量是一个高精度的值

---

控制语句 循环 分支 延时

---

### 循环

go只有一种循环 for

```go
    for i := 0; i < 10; i++ {
        fmt.Println(i, "\n")
    }
```

`ps: 和c++类似， 不同点在于{}不能省略，没有()`

for中的初始化和循环因子的更新都是可以省略的，和c++类似，
在此基础上;也是可以省略的

```go
    var sum = 0;
    i := 1
    for ; i < 100; {
        sum += i
    }

    for i < 100 {
        sum += i
    }
```

后一种中，;省略后，类似c++中的while

无限循环

```go
    for {
    }
```

### 判断

if语句

`ps: go中的if可以和短语句结合起来，
c++中的if和赋值也可以连用，不过c++中用的少`

<https://tour.golang.org/flowcontrol/8>
<https://tour.go-zh.org/flowcontrol/8>

例子中展示了计算平方根，使用了循环和函数

### 分支

switch语句

`ps：和c++一样，都是为了简化过长的if else语句,  
不一样的是go语言中在每个case中会自动添加break，  
所以就不会出现多个case共用一个执行体`

case中的条件，并不局限于`整形`

case的条件可以是表达式，函数等，switch的执行顺序是从上到下，
命中之后会跳过后面所有的case，和c++类似

switch后面的因子可以省略，等同于 switch true {},这时反而不如用if else

### defer

关键字defer 表示修饰代码会在函数结束时执行

defer修改的代码，参数取值是实时取，但是执行确在函数返回时

如果一个函数中有多个defer块，执行顺序是按栈式处理，先出现的后执行

---

更多类型介绍

---

### pointers

指针，看介绍和c++的指针类似

```go
  var p *int
  i := 1
  p = &i

  a := *p
```

指针的声明 取址 解引用 都和c++类似，
go中的指针没有算术运算

### struct

```go
  type abc struct{
    a int
    b int
  }

  v := abc{1, 2}
  v.a = 100

  v1 := abc{1}  // a:1 b:0
  v2 := abc{b: 10}  // a:0 b:10
  v3 := abc{} // a:0 b:0

  // point
  p := &v
  p.b = 200
```

取值也跟c++类似, 指针找元素跟c++有差异，都是. 不是c++的 ->

理解一下为啥和c++不一样：p是指针，正常取元素应该是(\*p).b,
但这样写比较麻烦，go直接简化成p.b，省掉解引用\*

`规则多了写的就严谨，但也限制了想写三行诗的人，google在这方面真是不遗余力`

结构体的字面量就表示一个新的变量(申请了内存的)，
可以显式指定元素的值，未显式指定的会有默认值

### array

数组，和c++类似，都是装同一类型的元素(不同的类型有struct)

数组变量的是有数组元素类型和元素个数组成，换句话说就是个数不能改

```go
  var a [10]int
  b := [6]int{}
```

### slices

百度翻译是`片`，文档说数组是定长，slices是变长

可直接称为切片

```go
  a := [6]int{1, 2, 3, 4, 5, 6}
  var s []int = a[1:3]

  x := [1]bool{true} // array
  y := []bool{true}  // slice字面量，会先创建一个数组，再去进行引用
```

slices的声明是 []int  不带长度就是slices，带长度就是数组，
是数组的一小片，取法是 数组名[低边界 : 高边界]

写法虽然是这样，其实是一个半开区间\[1, 3), s的值是{2,3},
角标都是从0开始, 边界可以省略，就是最小-最大

slices更像是数组的引用，修改slices，也会修改数组的值,中文叫法是切片

slice有两个属性：长度和容量，
长度是半开区间中元素的个数，
容量是第一个元素对应到数组中，到数组结尾时中间的元素个数，
go提供了两个函数来去slices的长度和容量 len(s) cap(s)

go中的空叫nil，类似c++中的NULL

nil slices是什么：
切片没有和数组关联，且长度和容量都是0

go有个内置函数叫make，可以用于创建一个slice，
make(sliece类型，长度，容量)，最后一个参数可以省略

多维slice，多维数组的一种，变长，想想都复杂

```go
  a := [][]string{
    []string{"a", "b"}
    []string{"1", "1"}
    []string{"2", "2"}
  }

  a[0][0] = "c"
```

不管是数组还是切片，使用方法类似于c++

go中还有内置函数用来在切片中追加：
func append(s []T, vs ...T) []T, 第一个参数是切片，
后面参数是追加的值，返回新的切片，
这个新的切片不一定还指向老的数组(如果追加的超过容量，会新申请一个)，
新申请的容量，是有一个策略的，不是追加多少申请多少, 一般是当前容量乘2。

for循环还有一种形式，适用于slice和map：

```go
  a = []int{1, 2, 3, 4, 5}
  for i, v := range a{
    fmt.Println("the %dth value is %d\n", i, v)
  }
```

这种类似c++中的 range for， 同样是遍历整个切片、map等，
go语言上的这种写法每次迭代返回两个值：索引和索引位置的值，
go语言为了这些规则束缚用户，又添加了两条路：
如果不想要索引的，用\_代替第一个参数，不想要值的，直接省略就行。

```go
  a := make([]int , 10)
  for i := range a {    // 只要索引不要值
  }
  for _, v := range a { // 只要值不要索引
  }
```

slice和for循环的练习：

```go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
  mat := make([][]uint8, dy)
  for i := range mat {
    mat[i] = make([]uint8, dx)
  }

  for x := range mat {
    for y := range mat[x] {
      mat[x][y] = uint8((x ^ y) % 255)
      // 这个颜色值可以用 (x+y)/2, x*y%255, x^y%255得到
    }
  }

return mat
}

func main() {
  pic.Show(Pic)
}
```

额外看一下mat的类型[][]uint8:
第一眼 是个切片，每个元素的类型是[]uint8,
在第一个for循环之前，mat里的值应该是空数组(不知道是不是nil)，个数是dy 256，
第一个for循环后，空数组都填充成0(dx 256)，后一个for进行赋值

### maps

key-value,
没有元素称为nil map， 特点是没有key，也不能添加key，
要先用make函数初始化一个，才能使用

```go
  var m map[string]int
  m = make(map[string]int)
  m["abc"] = 123

  var m1 map[string]int{
    "abc": 123
  }

  m1["abc"] = 1             // insert or update
  var i int = m1["abc"]     // retrieve
  delete(m1, "abc")         // delete key
  elem, ok = m1["abc"]      // exists test
```

`和c++相比，除了写法有所更改，其他的没什么不同`

map字面量需要带key, 如果顶层类型是类型名，可以省略，待后面跟进

测试map中是否存在key abc，ok是bool型，表示是否存在，
如果不存在，ok是false，elem指向第0个key的value；
如果存在，ok是true，elem指向指定key对应的value

### 函数变量

`c++中用函数指针来表示函数对象，go中的用法也差不多`

闭包，在函数体中引用了函数体外的变量，go函数可以是闭包，

```go
  func a() func(int) int{
    sum := 0
    return func (x int) int{
      sum += x
      return sum
    }
  }

  func main(){
    f := a()
    fmt.Println(f(1))
    fmt.Println(f(1))
    fmt.Println(f(1))
  }
```

a函数的返回值就是一个闭包

fibonacci:

```go
func fibonacci() func() int {
  a, b, c := 1, 1, 0

  return func() int {
    switch c {
    case 0:
      c = 1
      return 1
    case 1:
      c = 2
      return 1
    default:
      b += a
      a = b - a
      return b
    }
  }
}
```

闭包依赖父函数，所以父函数中的局部变量不会销毁，一直存在内存

## method and infterface

### 方法

go中没有类，但是可以定义基于类型的方法

`方法和函数，在go中有不同的意思，方法是一类特殊的函数`

函数带上一个接收者参数，就是方法，表示只在这个类型上的方法

```go
type Mytype struct{
}

func (mt Mytype) diy() int{
}

func diy2(mt Mytype) int{
}

func main(){
  a := Mytype{}
  a.diy()
  diy2(a)   // diy2是普通函数，效果和diy方法一致
}
```

方法看起来很强大，可以在各种类型中做扩展，限制也有：
只能对同一个包的类型添加方法，eg：不能直接对int扩展方法。
除非我们用type Myint int，来给内置的int取个别名，
就能在当前包中添加方法。

方法的参数，接收者，都有传值和传址的概念，和c++类似，
带指针的，就是传址，可以在函数里修改原始值；传值，是副本。

另外指针接收者比值接收者好的另一个理由是：
如果要变更接收者，指针接收者更容易。

如果是普通函数，参数是指针，那么调用时需要用到&变量，
方法着没有这些限制，如果方法的接收者是指针，调用时传变量(非指针)，
go会自动进行转换，所以只要使用方法，就可无脑了，不用关心这些坑。
另外传址比传值效率高，减少拷贝，万一值很大呢。

`从这点也可看出，go中如果吸收其他语言的精华，变动尽量小，
如果是新创的东西，尽量添加多条规则，放开限制，让使用者自由发挥`

### interface

接口类型是一系列方法的签名

一个接口类型的变量，可以存储任何实现了这些方法的值

比较绕口，看下的例子

```go
  // 定义一个接口类型
  type diyer interface{
    diy() int
  }

  // 定义两种类型，及基于类型的方法
  type Myint int
  func (i Myint) diy() int{
  }

  type Myvec struct{
  }
  func (v Myvec) diy() int{
  }

  func main(){
    var d diyer       // 声明接口对象
    i := Myint(1)     // 声明两个类型的对象
    v := Myvec{}

    a = i             // 接口对象存储实现了diy方法的类型的值
    a.diy()

    a = v
    a.diy()

    p = &v
    a = p             // error: 会提示指针类型的没有实现diy
  }
```

到目前为止，只知道go中的interface是一个类型，和c++中的接口不是一个意思，
那就当成新东西来理解, 目前的招式是：

* 有几个类型，都实现了同样的方法
* 有个接口定义时，也指定了同样的方法名
* 接口变量可以存储那几个类型的变量
* 结果就是使用接口变量和直接使用类型变量是一个效果

因为没有进一步了解，所以有以下猜测：

* 看起来像是接口变量是从类型变量中抽象出来的，
因为我们可以不直接使用具体类型的变量，而直接使用接口变量
* 对外暴露时，定制接口，具体实现视情况而选择不同的类型 - 像不像c++的多态
* 如果使用接口变量对外暴露，类型的普通函数就像是私有成员函数，
方法就像公有的 - 像不像c++的封装
* 就不知道后面有没有类似c++继承的东西(就是go里的类型扩展)
* 在这里接口变量就像基类，具体实现接口的那些类型像是派生类，
go语言也说了，go中没有class的概念，作为补偿，有类型，
所有的方法都是附加在类型上的，在c++中成员变量和成员函数的级别是同级，
在go中只是换了个说法，让方法依附于类型(难道成员方法不是基于class这个类型的吗)，
只是接口暴露的是方法，c++中只要是公有的都暴露，还分各种场景，复杂很多，
再说c++中的类不是接口的意思。

`上面的猜测基于上面的知识自己脑补的，不一定是这样，而且拿接口和基类比较也不妥`

`go确实简化了很多，接口只是接口，具体实现视具体情况而定，多态的确实如我说猜`

```go
package main

import "fmt"

// 声明接口
type I interface {
  M()
}

// 两个类型
type T struct {
  S string
}

type P struct {
  i int
}

// 类型实现接口 - 即实现接口中的方法即可
func (t T) M() {
  fmt.Println(t.S)
}

func (p P) M() {
  fmt.Println(p.i)
}

// 通过传不同的参数来调用不用的实现
func main() {
  var i I = T{"hello"}
  i.M()

  var j I = P{123}
  j.M()
}
```

接口的实现是隐式的，不需要显式声明，不需要关键字，
好处是可以在任何包中都可以添加一份实现，方便扩展。
`go语言又一次强调了方便`

回过头看看接口变量，上例中的 i j，go是这么说的：i可以理解为一个元组：
(参数value，实现类型type)，i和j对应的是(hello, main.T) (123, main.P),
执行i.M() 实际调用的是main.T.M(), 结构体的属性是hello

接口变量容易遇到一个问题：空指针 nil，
一般常出现在方法定义在指针接受者上，而给接口变量赋值之前，
具体的类型变量指针为空，那需要在方法中添加\_`非空判断`\_

这只是元组里的参数为nil，类型已知，还有一种都为空的情况：
声明接口变量后，不赋值，直接使用，这个时候下一条指令的数据和代码都为空，
所以接口变量遇到nil很容易出现崩溃。

go中的接口变量，下面简称接口

上面两种分下类：

* 非nil接口，数据为nil，实现类型正常，要做非nil判断
* nil接口，数据和实现类型均为nil，使用会出现崩溃

下面还有一种情况：

* empty 接口，接口声明时没有方法，好处是任何类型都可以看成她的实现，
在使用上定义一个empty接口，任意类型的变量都可以赋值。
看到这地方，接口除了上面猜测的，
还可以使用empty接口作为参数或返回值来做点什么，- 像不像c++中的泛型

### 类型断言

写法： t := i.(T) 用于获取接口值(接口变量的元组中的参数)的类型(元组中的实现类型)

还有一种写法是 t, ok := i.(T)

* i 是接口
* T 是类型
* ok表示断言是否成功或失败，bool，断言条件：接口中的类型是否是T
* t 如果断言成功，t就取接口i中的值，如果失败就是默认值
* 如果用第一种写法，如果断言失败，就会出现异常

这和 range for 类似, 都是返回两个值

### type switch

```go
  switch v := i.(type){
  case int:
  case string:
  default:
    // unknown type
  }
```

使用switch和type关键字来组合成一个type switch，不会像上面会包异常,
switch和type是一个组合技能，i.(type)单独拆开不能使用

在default中，v的值和i一样

### Stringers

最常见的interface，在fmt包中

```go
  type Stringer interface{
    String() string
  }
```

只要实现方法String，即可利用这个来实现打印任务,
String方法定制了打印样式，fmt.Println(类型)则调用了接口

### error

go中另一个常见的接口是

```go
  type error interface{
    Error() string
  }
```

和Stringer类似 ，都是属于fmt包

Error接口是定制错误打印的格式，和上面的Stringer接口类似，

接口也是变量，也可以充当返回值，
作为返回值时，通过判nil来确定是否出现error

### Readers

在io包中有这么一个接口 io.Reader,

```go
  type Reader interface{
    Read(p []byte) (n int, err error)
  }
```

用于读取数据流的结尾，有很多实现：文件、网络、压缩、密码等

Read方法就是将数据丢到slice中，并返回字节数和错误值，
如果没有数据，错误值就是EOF  end of file

`需要注意的是：for range 适用于slice和map 如果用在array上，
边界需要自己控制，不然就无限跑了`

### images

```go
  type Image interface{
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
  }
```

一个接口中定义了3个方法，这个接口在image包中

其中color.Model color.Color是两个接口，位于image/color包

这个接口给我展示了两个信息：接口可以有多个方法，方法的返回值可以是接口

---

go在多核方面有优势，下面就是并发相关的知识

---

## concurrency

### goroutine

协程，更加轻量级的线程，由go运行时提供并管理

进程 线程都属于系统级(内核态)的玩意，goroutine 是用户态的东西，
减少了多线程切换消耗，下面具体了解一下

```go
  go f(a, b, c)
```

这样就启动了一个协程，abc的计算是在当前协程，f的执行在新的协程中

协程共享地址空间，资源的访问需要同步，sync包提供了一些原语

### channel

频道 通道，go中被翻译为信道

通过 "<-" 操作，可利用channel进行发送或接受值

```go
  ch <- v     // 发送v的值到信道ch
  v := <-ch   // 从信道ch接收值，并存到v中
```

和map和slice类似，channel在使用前要先创建：

```go
  ch1 := make(chan int)
  ch2 := make(chan int, 100)  // buffered channel

  v, ok := <-ch   // ok用于判断ch是否close
```

chan是关键字

默认地，通过channel进行收发,都会等待对方做好准备才会开始，
好处是goroutine在同步时，需要添加额外的锁和条件变量

### buffered channel

带缓冲的信道，只有缓冲满时，发送才会被阻塞；
缓冲空时，接收才会被阻塞，而普通的信道是，另一端未做好准备都会被阻塞

超出缓冲会导致所有协程休眠，死锁
fatal error: all goroutines are asleep - deadlock!

读空信道，也会造成死锁

发送者可以close一个信道，表示无数据进行发送，
接受者可以通过第二个返回值来判读channel是否close

```go
  for i := range ch{
  }   // range for 组合除了可以用在slice和map上，还可以用于channel
```

channel由发送者close，向一个已经close的channel发送数据，会报异常

### select

在一个协程中，可以等待多个通信操作,select典型就是一个io多路复用的例子。

```go
  select{
  case ch1 <- x:    // 向ch1中发送一个x
  case <- ch2:      // 从ch2中取出一个
  default:          // 没有可执行的case是，执行这个
  }
```

select中的case能执行的时候就执行case，如果都没到时机，select就会阻塞，
多个case都可以执行时，select会随机选一个执行

执行匿名函数

```go
  func main(){
    go func(){
    }()
  }
```

select中的defalut，如果不加延时，基本上就是无限循环

### sync.Mutex

互斥，有两个方法： Lock Unlock

Unlock 可以用defer修饰

<https://tour.golang.org/concurrency/1>
<https://tour.go-zh.org/concurrency/1>
