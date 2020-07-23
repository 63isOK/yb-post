# Uber Go Style Guide

原文[url](https://github.com/uber-go/guide/blob/master/style.md),
中文翻译[url](https://github.com/xxjwxc/uber_go_guide_cn)

## Table of Contents

- [Introduction](#introduction)
- [Guidelines](#guidelines)
  - [Pointers to Interfaces](#pointers-to-interfaces)
  - [Verify Interface Compliance](#verify-interface-compliance)
  - [Receivers and Interfaces](#receivers-and-interfaces)
  - [Zero-value Mutexes are Valid](#zero-value-mutexes-are-valid)
  - [Copy Slices and Maps at Boundaries](#copy-slices-and-maps-at-boundaries)
  - [Defer to Clean Up](#defer-to-clean-up)
  - [Channel Size is One or None](#channel-size-is-one-or-none)
  - [Start Enums at One](#start-enums-at-one)
  - [Use `"time"` to handle time](#use-time-to-handle-time)
  - [Error Types](#error-types)
  - [Error Wrapping](#error-wrapping)
  - [Handle Type Assertion Failures](#handle-type-assertion-failures)
  - [Don't Panic](#dont-panic)
  - [Use go.uber.org/atomic](#use-gouberorgatomic)
  - [Avoid Mutable Globals](#avoid-mutable-globals)
  - [Avoid Embedding Types in Public Structs](#avoid-embedding-types-in-public-structs)
  - [Avoid Using Built-In Names](#avoid-using-built-in-names)
  - [Avoid `init()`](#avoid-init)
- [Performance](#performance)
  - [Prefer strconv over fmt](#prefer-strconv-over-fmt)
  - [Avoid string-to-byte conversion](#avoid-string-to-byte-conversion)
  - [Prefer Specifying Container Capacity](#prefer-specifying-container-capacity)
    - [Specifying Map Capacity Hints](#specifying-map-capacity-hints)
    - [Specifying Slice Capacity](#specifying-slice-capacity)
- [Style](#style)
  - [Be Consistent](#be-consistent)
  - [Group Similar Declarations](#group-similar-declarations)
  - [Import Group Ordering](#import-group-ordering)
  - [Package Names](#package-names)
  - [Function Names](#function-names)
  - [Import Aliasing](#import-aliasing)
  - [Function Grouping and Ordering](#function-grouping-and-ordering)
  - [Reduce Nesting](#reduce-nesting)
  - [Unnecessary Else](#unnecessary-else)
  - [Top-level Variable Declarations](#top-level-variable-declarations)
  - [Prefix Unexported Globals with _](#prefix-unexported-globals-with-_)
  - [Embedding in Structs](#embedding-in-structs)
  - [Use Field Names to Initialize Structs](#use-field-names-to-initialize-structs)
  - [Local Variable Declarations](#local-variable-declarations)
  - [nil is a valid slice](#nil-is-a-valid-slice)
  - [Reduce Scope of Variables](#reduce-scope-of-variables)
  - [Avoid Naked Parameters](#avoid-naked-parameters)
  - [Use Raw String Literals to Avoid Escaping](#use-raw-string-literals-to-avoid-escaping)
  - [Initializing Struct References](#initializing-struct-references)
  - [Initializing Maps](#initializing-maps)
  - [Format Strings outside Printf](#format-strings-outside-printf)
  - [Naming Printf-style Functions](#naming-printf-style-functions)
- [Patterns](#patterns)
  - [Test Tables](#test-tables)
  - [Functional Options](#functional-options)

## Introduction

风格就是我们常说的编码规范。源码的格式化我们不需要操心，gofmt会处理。

本文档通过描述"做什么"和"不做什么"来管理写代码时的复杂度。
这些规则存在的目的是让代码易于管理。

大部分惯例来至"effective go" 和
[Go常见错误指导](https://github.com/golang/go/wiki/CodeReviewComments)

所有的代码在运行golint和go vet时，不应该报错。
所以推荐使用goimports/golint/go vet来检查错误,
而ide基本上都为我们调用了这些检查.

## Guidelines

### Pointers to Interfaces

用于不要使用接口指针。每次传递接口都使用传值，
因为里面的基础数据仍然可以是指针.

一个接口有两个部分：

1. 一个指针指向具体类型的信息，可以理解为类型
2. 数据指针。如果数据是一个指针，就直接存，如果数据是一个值，会存指向值的指针。

如果想通过接口方法来修改底层的值，那就需要使用指针. 什么指针?

    // 手动修改例子来调试
    type A interface {
      add()
      print()
      modify(n int)
    }

    type B struct {
      v int
    }

    func (b *B)add() {
      b.v++
    }

    func (b B)print() {
      fmt.Println(b.v)
    }

    func (b B)modify(n int) {
      b.v += n
      fmt.Println(b.v, n)
    }

    // main函数中执行
    var i A = &B{1}
    i.add()
    i.modify(5)
    i.print()

对上面的例子,先说几点:

- 如果B对象不是将指针赋给接口变量i,而是将B对象的值付给i
  - 首先,赋值是没问题的,因为接口变量底层也是指针
  - 其次,语法检测会报错,因为值接收者的方法集中没有add()方法
  - 再者,将add改为值方法,此时会发现,add/modify无法改变B对象的值
    - 因为是传值,所以改变是无法影响原有对象的
- 如果B对象是将指针付给接口变量i,是可以通过方法来改变原有对象的值

所以,这个部分讲的是:

- 使用接口变量时,`没必要使用接口指针变量`
- 给接口变量赋值时,可以是对象值,也可以是对象指针
  - 如果要修改对象,那必须是将对象指针赋值给接口变量

### Verify Interface Compliance

检查接口的合理性,是在编译期做的,主要包括:

- 实现指定接口的导出类型,会作为接口api的一部分进行检查
- 实现同一接口的类型(不管是导出还是非导出),都属于实现类型的集合
- 任何违反接口合理性检查的其他场景,都会终止编译,并通知用户

最后这条才是重点,大意是错误使用接口会在编译报错.
所以可以利用这个特点,可以将部分问题在编译期暴露出来.

    // 坏味道
    // 如果Handler没有完全实现http.Handler,会在运行时报错
    type Handler struct {
      // ...
    }
    func (h *Handler) ServeHTTP(
      w http.ResponseWriter,
      r *http.Request,
    ) {
      ...
    }

    // 好味道
    type Handler struct {
      // ...
    }

    // 加上这个,会触发编译期的检查
    // 这么做是保证提早发现问题(从运行时提早到编译期)
    // 得到的好处很多,但花费很少
    var _ http.Handler = (*Handler)(nil)

    func (h *Handler) ServeHTTP(
      w http.ResponseWriter,
      r *http.Request,
    ) {
      // ...
    }

### Receivers and Interfaces

指针也能调用值的方法集,更具体的是:
值接收者的方法集是指针接收者方法集的子集,反之不是.

    type S struct {
      data string
    }

    func (s S) Read() string {
      return s.data
    }

    func (s *S) Write(str string) {
      s.data = str
    }

    sVals := map[int]S{1: {"A"}}

    // You can only call Read using a value
    sVals[1].Read()

    // This will not compile:
    //  sVals[1].Write("test")

    sPtrs := map[int]*S{1: {"A"}}

    // You can call both Read and Write using a pointer
    sPtrs[1].Read()
    sPtrs[1].Write("test")

也可以用指针来满足一个接口，即使方式是一个值接收者

    type F interface {
      f()
    }

    type S1 struct{}

    func (s S1) f() {}

    type S2 struct{}

    func (s *S2) f() {}

    s1Val := S1{}
    s1Ptr := &S1{}      // 指针对象是可以使用值接收者的方法集
    s2Val := S2{}
    s2Ptr := &S2{}

    var i F
    i = s1Val
    i = s1Ptr
    i = s2Ptr

    // The following doesn't compile, since s2Val is a value,
    // and there is no value receiver for f.
    //   i = s2Val

这上面两个例子，原理都是一个：
一个值变量，只能使用值接收者的方法集;
一个指针变量，能使用值接收者的方法集 + 指针接收者的方法集。

而接口,底层就是指针,某个类型实现了接口,说的是这个类型的方法集合接口匹配,
更进一步,这个类型的值方法集或指针方法集合接口的方法集匹配.
有趣的事情就在这里出现了:

- 值方法集合满足接口(和接口的方法集匹配)
  - 不管给接口变量赋值的是值变量还是指针变量,都ok
  - 因为不管值变量和指针变量,他们的方法集都包含值方法集
- 指针方法集才满足接口
  - 这是只能将指针变量赋值给接口变量
  - 将值变量赋值给接口变量会报错(因为不满足接口)
  - 为什么是编译期报错? 上一节的接口合理性检查提到了检测规则

再看看为啥 i=s2Val会报错,因为s2的值方法集不满足接口定义,仅此而已.

### Zero-value Mutexes are Valid

sync.Mutex sync.RWMutex的零值是有意义的,所以永远不要用互斥量指针(因为没必要)。

如果有一个结构体指针，那互斥量也要是非指针字段。
理由和上条建议一样,互斥量的零值是有意义的,没必要弄指针.

互斥量是来保护某些字段的，暴露到包外是不明智的选择(),
结构体中的互斥量，如果互斥量是嵌入到结构体，那结构体不要暴露;
如果互斥量以字段的形式出现在结构体中，那只要互斥量字段不要暴露就行。

    // 对于不导出类型(私有类型)
    // 使用嵌入和字段都可以
    type smap struct {
      sync.Mutex // only for unexported types

      data map[string]string
    }

    func newSMap() *smap {
      return &smap{
        data: make(map[string]string),
      }
    }

    func (m *smap) Get(k string) string {
      m.Lock()
      defer m.Unlock()

      return m.data[k]
    }

    // 对于导出类型
    // 互斥量只能作为非导出字段,这样是为了安全
    type SMap struct {
      mu sync.Mutex

      data map[string]string
    }

    func NewSMap() *SMap {
      return &SMap{
        data: make(map[string]string),
      }
    }

    func (m *SMap) Get(k string) string {
      m.mu.Lock()
      defer m.mu.Unlock()

      return m.data[k]
    }

总的来说:

- 互斥量的零值是有意义的,避免使用互斥量指针
- 不要将互斥量的操作暴露到外面,这是为了安全着想

### Copy Slices and Maps at Boundaries

map/slice的边界拷贝:为了数据安全.

slice和map都是引用类型，她们都包含一个指针，指向实际数据，
所以拷贝数据时需要警惕,特别是边界。

#### Receiving Slices and Maps

如果将slice/map以参数的形式接收过来，自己存储的时候如果是存引用(不是值)，
那么后续操作中是可以修改这个slice/map的。

分析：slice/map作为参数传递，本身就是引用类型，如果我们有个变量存储了这个引用，
那么，在后续的操作中(可能不在这个函数中)，只要修改了这个变量，
就会修改对应的slice/map。如果不想后续操作会改变slice/map，可以在函数做拷贝

bad:

    func (d *Driver) SetTrips(trips []Trip) {
      d.trips = trips
    }

    trips := ...
    d1.SetTrips(trips)

    // Did you mean to modify d1.trips?
    trips[0] = ...   // 这个会改变，因为d.trips存的是引用

good:

    func (d *Driver) SetTrips(trips []Trip) {
      d.trips = make([]Trip, len(trips))
      copy(d.trips, trips)
    }

    trips := ...
    d1.SetTrips(trips)

    // We can now modify trips[0] without affecting d1.trips.
    trips[0] = ...
    // 新建变量来存储

总结:以slice/map为参数,而且这个引用还保存了,
就需要知道引用的这个值是可能被改变的;
如果想存引用但不想被改动,就新建一个slice/map.

#### Returning Slices and Maps

将slice/map作为函数返回值时，也需要注意，是否能让外部修改slice/map

bad:

    type Stats struct {
      mu sync.Mutex
      counters map[string]int
    }

    // Snapshot returns the current stats.
    func (s *Stats) Snapshot() map[string]int {
      s.mu.Lock()
      defer s.mu.Unlock()

      return s.counters
    }

    // snapshot is no longer protected by the mutex, so any
    // access to the snapshot is subject to data races.
    snapshot := stats.Snapshot()
    // 会修改原生slice/map

good:

    type Stats struct {
      mu sync.Mutex
      counters map[string]int
    }

    func (s *Stats) Snapshot() map[string]int {
      s.mu.Lock()
      defer s.mu.Unlock()

      result := make(map[string]int, len(s.counters))
      for k, v := range s.counters {
        result[k] = v
      }
      return result
    }

    // Snapshot is now a copy.
    snapshot := stats.Snapshot()
    // 修改的是一个拷贝，不影响原始数据

为了保护数据,可以在返回之前,新建一个slice/map.

map/slice的边界拷贝:为了数据安全.

因为map/slice是引用类型,
所以进出函数不做限制都有可能发生值改变的情况,
所以在函数边界添加拷贝,这样就可以起到保护数据的作用.

### Defer to Clean Up

延时函数，成对的操作(特别是资源的申请和释放)，特别适合使用defer

bad：

    p.lock()
    if p.count < 10 {
      p.unlock()
      return p.count
    }

    p.count++
    newcount := p.count
    p.unlock()
    // 要处理多个可能的返回点，容易出错

good：

    p.Lock()
    defer p.Unlock()

    if p.count < 10 {
      return p.count
    }

    p.count++
    return p.count

    // more readable
    // 使用延时函数，写一处就行
    // 由语言机制去简化操作
    // 可读性大大提高

defer是很方便，但不是万灵药，defer也有消耗，非常小,纳秒级。
只有函数的消耗在纳秒级，才无需使用defer(应该使用其他方式)。
就算看在defer的可读性上，也足够选择使用defer了。
当方法是一个大方法，且访问内存更加复杂的，就更应该使用defer。

### Channel Size is One or None

在实际使用中，channel的缓冲大小要么是1,要么是无缓冲的(无缓冲，表示缓冲大小为0)。
其他任何大小都必须严格审查。
大小具体如何决定，取决于在高负载或阻塞写下的写入，以及何时发生这种事。

bad:

    // Ought to be enough for anybody!
    c := make(chan int, 64)

good:

    // Size of one
    c := make(chan int, 1) // or
    // Unbuffered channel, size of zero
    c := make(chan int)

### Start Enums at One

使用枚举的标准方式是定义一个自定义类型，一个const组，并使用iota。
建议枚举值从1开始，或者非0值

bad：

    type Operation int

    const (
      Add Operation = iota
      Subtract
      Multiply
    )

    // Add=0, Subtract=1, Multiply=2

good：

    type Operation int

    const (
      Add Operation = iota + 1
      Subtract
      Multiply
    )

    // Add=1, Subtract=2, Multiply=3

当然并不是绝对的,有些我们已经习惯从0开始的,还是从0开始比较好.

### Use `"time"` to handle time

首先我们的观点中对时间有很多误解,
一天是不是正好24小时整,一年是不是固定的365天.

所以正确的姿势是使用time包来处理时间问题.

#### 用time.Time来表示时刻

时刻的比较使用

    func isActive(now, start, stop time.Time) bool {
      return (start.Before(now) || start.Equal(now)) && now.Before(stop)
    }

#### 用time.Duration表示时间段

能用time包提供的,就不要自己实现

    func poll(delay time.Duration) {
      for {
        // ...
        time.Sleep(delay)
      }
    }

    poll(10*time.Second)

应该基于意图来选择合适的处理,
加一天,就不要使用加24小时.

#### 使用time.Time/time.Duration来和外部系统交互

- flag包,通过time.ParseDuration支持time.Duration
- encoding/json包,通过UnmarshlJSON支持time.Time
- database/sql包, 支持DATATIME/TIMESTAMP转time.Time
- gopkg.in/yaml.v2包,也支持time.Time

所以说和外部系统交互时,time包也足够强大,

万一time搞不定,就将time.Duration转成int/float64.
并在字段名上体现单位.
将time.Time转成string.

    // {"intervalMillis": 2000}
    type Config struct {
      IntervalMillis int `json:"intervalMillis"`
    }

### Error Types

一般使用error的方式有以下几种：

1. errors.New 给error附加了一个string，可用于描述错误
2. fmt.Errorf 格式化error的string
3. 自定义，实现Error 接口
4. 使用github.com/pkg/errors的封装

如果一个函数返回一个error，要考虑以下几点：

- 如果是简单信息，无需额外信息，可使用errors.New
- 如果调用者要检测并处理这个错误，自定义类型最合适
- 如果仅仅是将下游函数的错误进行传递，那么检查"错误封装的节",就是下一节
- 其他情况，使用fmt.Errorf

当调用者需要检测这个错误，并处理时，如果使用errors.New：

bad：

    // package foo

    func Open() error {
      return errors.New("could not open")
    }

    // package bar

    func use() {
      if err := foo.Open(); err != nil {
        if err.Error() == "could not open" {
          // handle
        } else {
          panic("unknown error")
        }
      }
    }
    // 两处都需要维护一致的错误信息

good：

    // package foo

    var ErrCouldNotOpen = errors.New("could not open")

    func Open() error {
      return ErrCouldNotOpen
    }

    // package bar

    if err := foo.Open(); err != nil {
      if err == foo.ErrCouldNotOpen {
        // handle
      } else {
        panic("unknown error")
      }
    }
    // 将错误信息提取出来，一般在重构时会消除这个坏味道

调用者需要检测并处理错误，如果不仅仅局限于静态字符串，
那最好使用自定义错误类型。

bad：

    func open(file string) error {
      return fmt.Errorf("file %q not found", file)
    }

    func use() {
      if err := open(); err != nil {
        if strings.Contains(err.Error(), "not found") {
          // handle
        } else {
          panic("unknown error")
        }
      }
    }

good:

    type errNotFound struct {
      file string
    }

    func (e errNotFound) Error() string {
      return fmt.Sprintf("file %q not found", e.file)
    }

    func open(file string) error {
      return errNotFound{file: file}
    }

    func use() {
      if err := open(); err != nil {
        if _, ok := err.(errNotFound); ok {
          // handle
        } else {
          panic("unknown error")
        }
      }
    }

如果返回自定义错误类型的函数被导出了，
那这个自定义错误类型就成了公共api的一部分。
此时最好也暴露一个和错误类型匹配的检测函数。

    // package foo

    type errNotFound struct {
      file string
    }

    func (e errNotFound) Error() string {
      return fmt.Sprintf("file %q not found", e.file)
    }

    func IsNotFoundError(err error) bool {
      _, ok := err.(errNotFound)
      return ok
    }

    func Open(file string) error {
      return errNotFound{file: file}
    }

    // package bar

    if err := foo.Open("foo"); err != nil {
      if foo.IsNotFoundError(err) {
        // handle
      } else {
        panic("unknown error")
      }
    }

### Error Wrapping

调用失败时，现在主要有3种主流方式来传递错误。

1. 直接返回原始错误类型, 适合没有附加上下文，且想保持原始错误类型的场景
2. 有上下文，是用github.com/pkg/errors来封装，可以附加更多上下文
3. 使用fmt.Errof,适合调用者不关心错误的场景(不检测也不处理)

推荐使用的还是添加上下文，添加了没坏处，
万一用到了上下文中的信息，不就赚到了吗，
特别是扩展时，也方便。
这意味着多多使用pkg/errors.

当返回错误时，附加上下文，尽量不要附加一些"失败于:"类似的短语，目的是保持简洁。

bad：

    s, err := store.New()
    if err != nil {
        return fmt.Errorf(
            "failed to create new store: %s", err)
    }
    // 错误传递之后，就是下面的语句了
    // failed to x: failed to y: failed to create new store: the error

good:

    s, err := store.New()
    if err != nil {
        return fmt.Errorf(
            "new store: %s", err)
    }
    // 错误传递之后，就是下面的语句了
    // x: y: new store: the error

当然，如果错误是发到另一个系统，
错误信息最好是简洁的(前缀有个err或failed)

### Handle Type Assertion Failures

类型断言，如果只返回一个值，断言不成立时会报运行时异常。
因此第二个返回值ok，需要一直有。

bad：

    t := i.(string)

good：

    t, ok := i.(string)
    if !ok {
      // handle the error gracefully
    }

### Don't Panic

产品级的代码，在运行时要避免运行时异常。

运行时异常，是主要的源码级级联故障。
如果错误发生了，函数必须返回一个错误以允许调用者来决定如何处理

bad：

    func foo(bar string) {
      if len(bar) == 0 {
        panic("bar must not be empty")
      }
      // ...
    }

    func main() {
      if len(os.Args) != 2 {
        fmt.Println("USAGE: foo <bar>")
        os.Exit(1)
      }
      foo(os.Args[1])
    }

good:

    func foo(bar string) error {
      if len(bar) == 0 {
        return errors.New("bar must not be empty")
      }
      // ...
      return nil
    }

    func main() {
      if len(os.Args) != 2 {
        fmt.Println("USAGE: foo <bar>")
        os.Exit(1)
      }
      if err := foo(os.Args[1]); err != nil {
        panic(err)
      }
    }
    // 函数只返回错误，让调用者决定是panic还是做其他处理

`panic/recover不是错误处理策略`.

程序只有当遇到无法恢复的情况才应该panic(eg:nil的解引用操作)。
例外：程序初始化时，遇到错误信息，应该panic。

    var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
    // 解析不成功就应该panic

在测试中，使用t.Fatal t.FailNow (而不是panic)，会让测试用例标记为失败。

bad：

    // func TestFoo(t *testing.T)

    f, err := ioutil.TempFile("", "test")
    if err != nil {
      panic("failed to set up test")
    }
    // 失败是失败了，但没有将用例标记为失败

good:

    // func TestFoo(t *testing.T)

    f, err := ioutil.TempFile("", "test")
    if err != nil {
      t.Fatal("failed to set up test")
    }

### Use go.uber.org/atomic

使用sync/atomic处理原始类型的原子操作很方便，处理变量就容易出错，
推荐使用go.uber.org/atomic包，她隐藏了一些底层类型的细节，添加了bool的原子操作。

bad：

    type foo struct {
      running int32  // atomic
    }

    func (f* foo) start() {
      if atomic.SwapInt32(&f.running, 1) == 1 {
         // already running…
         return
      }
      // start the Foo
    }

    func (f *foo) isRunning() bool {
      return f.running == 1  // race!
    }

good:

    type foo struct {
      running atomic.Bool
    }

    func (f *foo) start() {
      if f.running.Swap(true) {
         // already running…
         return
      }
      // start the Foo
    }

    func (f *foo) isRunning() bool {
      return f.running.Load()
    }

具体遇到了再详细了解。

## Performance

这些只适用于热路径的优化。

什么叫热路径：
二八规则也适用于软件，80%的用户只会使用20%的特征，那这20%就是热路径 hot path，
对应的叫cool path。

微软对hot path的解释是：调用的最多，花费资源最多的调用。

### Prefer strconv over fmt

各种类型和string的转换，使用strconv，而不是fmt

bad：

    for i := 0; i < b.N; i++ {
      s := fmt.Sprint(rand.Int())
    }
    // BenchmarkFmtSprint-4    143 ns/op    2 allocs/op

good:

    for i := 0; i < b.N; i++ {
      s := strconv.Itoa(rand.Int())
    }
    // BenchmarkStrconv-4    64.2 ns/op    1 allocs/op

性能还是有差别的

### Avoid string-to-byte conversion

避免多次从固定字符串转换成切片，应该转换一次，并保存。
这是重构时的一个坏味道。

bad:

    for i := 0; i < b.N; i++ {
      w.Write([]byte("Hello world"))
    }
    // BenchmarkBad-4   50000000   22.2 ns/op

good:

    data := []byte("Hello world")
    for i := 0; i < b.N; i++ {
      w.Write(data)
    }
    // BenchmarkGood-4  500000000   3.25 ns/op

### Prefer Specifying Map Capacity Hints

在用make初始化map时，尽可能提供容量，用处是检查重新分配的次数

bad：

    m := make(map[string]os.FileInfo)

    files, _ := ioutil.ReadDir("./files")
    for _, f := range files {
        m[f.Name()] = f
    }
    // 文件较多时，会触发多次申请

good:

    files, _ := ioutil.ReadDir("./files")

    m := make(map[string]os.FileInfo, len(files))
    for _, f := range files {
        m[f.Name()] = f
    }
    // 更少的申请操作，意味更高的性能

## Style

### Be Consistent

风格要保持统一，好处多多。

### Group Similar Declarations

组声明方式保持一致

bad：

    import "a"
    import "b"

good:

    import (
      "a"
      "b"
    )

变量/常量/类型声明都是类似的：

bad：

    const a = 1
    const b = 2

    var a = 1
    var b = 2

    type Area float64
    type Volume float64

good:

    const (
      a = 1
      b = 2
    )

    var (
      a = 1
      b = 2
    )

    type (
      Area float64
      Volume float64
    )

同组的放一起，不同组不要放一起：

bad：

    type Operation int

    const (
      Add Operation = iota + 1
      Subtract
      Multiply
      ENV_VAR = "MY_ENV"
    )

good:

    type Operation int

    const (
      Add Operation = iota + 1
      Subtract
      Multiply
    )

    const ENV_VAR = "MY_ENV"

组也可以用在函数内部：

bad：

    func f() string {
      var red = color.New(0xff0000)
      var green = color.New(0x00ff00)
      var blue = color.New(0x0000ff)

      ...
    }

good:

    func f() string {
      var (
        red   = color.New(0xff0000)
        green = color.New(0x00ff00)
        blue  = color.New(0x0000ff)
      )

      ...
    }

### Import Group Ordering

导入的组应该分两个，标准包，非标准包

bad:

    import (
      "fmt"
      "os"
      "go.uber.org/atomic"
      "golang.org/x/sync/errgroup"
    )

good:

    import (
      "fmt"
      "os"

      "go.uber.org/atomic"
      "golang.org/x/sync/errgroup"
    )

### Package Names

遵循以下规则：

- 全小写，无下划线
- 大部分导入情况下，不需要起别名的名字
- 简短 + 简洁
- 不要复数。eg：使用net/url,而不是net/urls
- 不要 common/util/shared/lib, 这些名字不好，且包含的信息不够

### Function Names

Go的惯例是使用 MixedCaps，而不是下划线加小写。
例外：测试函数包含下划线，表示测试一组相关的用例。
eg：TestMyFunction_WhatIsBeingTested

### Import Aliasing

如果包的导入路径中最后元素和包名不一致，要使用导入别名

    import (
      "net/http"

      client "example.com/client-go"
      trace "example.com/trace/v2"
    )

其他情况，尽量不要使用导入别名，除非有冲突。

bad：

    import (
      "fmt"
      "os"

      nettrace "golang.net/x/trace"
    )

good:

    import (
      "fmt"
      "os"
      "runtime/trace"

      nettrace "golang.net/x/trace"
    )

还有一个例外是包名太长，eg:spf13的jww

### Function Grouping and Ordering

- 函数应该按调用顺序排序
- 一个文件中，应该按接收者来分组

暴露的函数应该在源文件中先出现，当然要在struct/const/var定义之后。

类似newXXX()/NewXXX()的应该出现在类型后面，方法前面。

函数应该按接收者分组，纯工具函数应该放在源码文件的最后。

bad:

    func (s *something) Cost() {
      return calcCost(s.weights)
    }

    type something struct{ ... }

    func calcCost(n []int) int {...}

    func (s *something) Stop() {...}

    func newSomething() *something {
        return &something{}
    }
    // 无序

good：

    type something struct{ ... }        // 类型第一

    func newSomething() *something {    // 构造第二
        return &something{}
    }

    func (s *something) Cost() {        // 方法第三，按接收者分组
      return calcCost(s.weights)
    }

    func (s *something) Stop() {...}

    func calcCost(n []int) int {...}    // 功能函数最后

### Reduce Nesting

- 尽量减少嵌套层数
- 错误分支/指定条件分支先处理
- 循环中的return/continue分支先处理

bad：

    for _, v := range data {
      if v.F1 == 1 {
        v = process(v)
        if err := v.Call(); err == nil {
          v.Send()
        } else {
          return err
        }
      } else {
        log.Printf("Invalid v: %v", v)
      }
    }

good:

    for _, v := range data {
      if v.F1 != 1 {
        log.Printf("Invalid v: %v", v)
        continue
      }

      v = process(v)
      if err := v.Call(); err != nil {
        return err
      }
      v.Send()
    }

### Unnecessary Else

else分支能省就省,适合只有两个分支的情况

bad:

    var a int
    if b {
      a = 100
    } else {
      a = 10
    }

good:

    a := 10
    if b {
      a = 100
    }

### Top-level Variable Declarations

对于顶层变量声明，使用var关键字。不要指定类型，除非预期类型和表达式类型不一致

bad：

    var _s string = F()

    func F() string { return "A" }

good:

    var _s = F()
    // Since F already states that it returns a string, we don't need to specify
    // the type again.

    func F() string { return "A" }

下面是预期类型和表达式类型不一致的情况：

    type myError struct{}

    func (myError) Error() string { return "error" }

    func F() myError { return myError{} }

    var _e error = F()
    // F returns an object of type myError but we want error.

### Prefix Unexported Globals with _

顶层变量和常量，如果不导出，那就加前缀"下划线"来表示。
例外：未导出的错误变量，前缀应该是err

原理：顶层变量和常量，她们的作用域是包，不特殊标识，会在其他文件容易导致错误。

bad：

    // foo.go

    const (
      defaultPort = 8080
      defaultUser = "user"
    )

    // bar.go

    func Bar() {
      defaultPort := 9090
      ...
      fmt.Println("Default port", defaultPort)

      // We will not see a compile error if the first line of
      // Bar() is deleted.
    }

good:

    // foo.go

    const (
      _defaultPort = 8080
      _defaultUser = "user"
    )

### Embedding in Structs

结构体中的嵌入类型，应该在字段列表的最上面，且应该有一个空行和常规字段分开。

bad：

    type Client struct {
      version int
      http.Client
    }

good:

    type Client struct {
      http.Client

      version int
    }

### Use Field Names to Initialize Structs

初始化一个结构体时，要指明字段名。go vet现在支持这个检查

bad：

    k := User{"John", "Doe", true}

good:

    k := User{
        FirstName: "John",
        LastName: "Doe",
        Admin: true,
    }

例外：table测试时，如果字段少于4个，字段名可省略

    tests := []struct{
      op Operation
      want string
    }{
      {Add, "add"},
      {Subtract, "subtract"},
    }

### Local Variable Declarations

局部变量声明时如果显示指定了值，最好使用短变量声明方式(:=)

    var s = "foo"     // bad
    s := "foo"        // good

有些时候，使用var会更加清晰

bad：

    func f(list []int) {
      filtered := []int{}
      for _, v := range list {
        if v > 10 {
          filtered = append(filtered, v)
        }
      }
    }

good：

    func f(list []int) {
      var filtered []int
      for _, v := range list {
        if v > 10 {
          filtered = append(filtered, v)
        }
      }
    }

### nil is a valid slice

nil slice是有效的。

nil的slice有以下意思：

第一,显示返回一个0长度的slice，使用nil代替

    // bad
    if x == "" {
      return []int{}
    }

    // good
    if x == "" {
      return nil
    }

第二, 检查一个slice是否空，使用 len(s) == 0, 不要检查nil

    // bad
    func isEmpty(s []string) bool {
      return s == nil
    }

    // good
    func isEmpty(s []string) bool {
      return len(s) == 0
    }

第三, 未初始化(make)的slice(也就是零值 slice)是可用的

bad：

    nums := []int{}
    // or, nums := make([]int)

    if add1 {
      nums = append(nums, 1)
    }

    if add2 {
      nums = append(nums, 2)
    }

good:

    var nums []int

    if add1 {
      nums = append(nums, 1)
    }

    if add2 {
      nums = append(nums, 2)
    }

### Reduce Scope of Variables

尽量减少变量的作用域，当然不能引起冲突

bad：

    err := ioutil.WriteFile(name, data, 0644)
    if err != nil {
     return err
    }

good：

    if err := ioutil.WriteFile(name, data, 0644); err != nil {
     return err
    }

如果在函数外面还要使用这个结果，那就不要减少变量的作用域

### Avoid Naked Parameters

避免使用裸参数，裸参数会导致代码阅读性大大降低

    // func printInfo(name string, isLocal, done bool)

    // bad
    printInfo("foo", true, true)

    // good
    printInfo("foo", true /* isLocal */, true /* done */)

使用注释提高裸参数的阅读性。最好的方式是将裸的bool做成自定义类型，
未来也可以扩展更多的状态(可以是非true/false)

    type Region int

    const (
      UnknownRegion Region = iota
      Local
    )

    type Status int

    const (
      StatusReady = iota + 1
      StatusDone
      // Maybe we will have a StatusInProgress in the future.
    )

    func printInfo(name string, region Region, status Status)

### Use Raw String Literals to Avoid Escaping

Go支持原始字符串字面量，这样可以避免转义。

    // bad
    wantError := "unknown name:\"test\""

    // good
    wantError := `unknown error:"test"`

如果有转义字符，我们就可以选择使用原始字面量

### Initializing Struct References

初始化结构体引用，使用 &T{} 而不是 new(T)

bad:

    sval := T{Name: "foo"}

    // inconsistent
    sptr := new(T)
    sptr.Name = "bar"
    // 风格不一致

good：

    sval := T{Name: "foo"}

    sptr := &T{Name: "bar"}
    // 一致的风格

### Initializing Maps

使用make创建一个空的map，然后编程来添加元素。
这样的好处是初始化和声明的独立的。

bad：

    var (
      // m1 is safe to read and write;
      // m2 will panic on writes.
      m1 = map[T1]T2{}
      m2 map[T1]T2
    )
    // 声明和初始化的方式很相似

good：

    var (
      // m1 is safe to read and write;
      // m2 will panic on writes.
      m1 = make(map[T1]T2)
      m2 map[T1]T2
    )
    // 声明和初始化在视觉上就很独立,易区分

初始化时尽可能提供容量信息。

使用map字面量来初始化一个map，是优先选择的。

bad：

    m := make(map[T1]T2, 3)
    m[k1] = v1
    m[k2] = v2
    m[k3] = v3

good:

    m := map[T1]T2{
      k1: v1,
      k2: v2,
      k3: v3,
    }

基本规则如下：

- 有固定元素集来初始化map时，使用map字面量的方式
- 否则使用make来初始化。尽可能指定容量信息

### Format Strings outside Printf

使用Print-家族函数时，如果格式化字符串是单独声明，那最好声明成const 无类型

    // bad
    msg := "unexpected values %v, %v\n"
    fmt.Printf(msg, 1, 2)

    // good
    const msg = "unexpected values %v, %v\n"
    fmt.Printf(msg, 1, 2)

go vet 静态分析也会检测这点

### Naming Printf-style Functions

基于go vet对printf家族的检查，使用go vet -printfuncs=wrapf,statusf

具体后面可以继续跟进

## Patterns

### Test Tables

当测试逻辑是相同时，为了避免重复写代码，使用表格驱动测试。

bad:

    // func TestSplitHostPort(t *testing.T)

    host, port, err := net.SplitHostPort("192.0.2.0:8000")
    require.NoError(t, err)
    assert.Equal(t, "192.0.2.0", host)
    assert.Equal(t, "8000", port)

    host, port, err = net.SplitHostPort("192.0.2.0:http")
    require.NoError(t, err)
    assert.Equal(t, "192.0.2.0", host)
    assert.Equal(t, "http", port)

    host, port, err = net.SplitHostPort(":8000")
    require.NoError(t, err)
    assert.Equal(t, "", host)
    assert.Equal(t, "8000", port)

    host, port, err = net.SplitHostPort("1:8")
    require.NoError(t, err)
    assert.Equal(t, "1", host)
    assert.Equal(t, "8", port)

good:

    // func TestSplitHostPort(t *testing.T)

    tests := []struct{
      give     string
      wantHost string
      wantPort string
    }{
      {
        give:     "192.0.2.0:8000",
        wantHost: "192.0.2.0",
        wantPort: "8000",
      },
      {
        give:     "192.0.2.0:http",
        wantHost: "192.0.2.0",
        wantPort: "http",
      },
      {
        give:     ":8000",
        wantHost: "",
        wantPort: "8000",
      },
      {
        give:     "1:8",
        wantHost: "1",
        wantPort: "8",
      },
    }

    for _, tt := range tests {
      t.Run(tt.give, func(t *testing.T) {
        host, port, err := net.SplitHostPort(tt.give)
        require.NoError(t, err)
        assert.Equal(t, tt.wantHost, host)
        assert.Equal(t, tt.wantPort, port)
      })
    }

### Functional Options

函数的可选项，这是一种模式，特别适合在公共api中。

bad：坏坏的味道：

    // package db

    func Connect(
      addr string,
      timeout time.Duration,
      caching bool,
    ) (*Connection, error) {
      // ...
    }

    // Timeout and caching must always be provided,
    // even if the user wants to use the default.

    db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
    db.Connect(addr, newTimeout, db.DefaultCaching)
    db.Connect(addr, db.DefaultTimeout, false /* caching */)
    db.Connect(addr, newTimeout, false /* caching */)

good: 优雅是何物：

    type options struct {
      timeout time.Duration
      caching bool
    }

    // Option overrides behavior of Connect.
    type Option interface {
      apply(*options)
    }

    type optionFunc func(*options)

    func (f optionFunc) apply(o *options) {
      f(o)
    }

    func WithTimeout(t time.Duration) Option {
      return optionFunc(func(o *options) {
        o.timeout = t
      })
    }

    func WithCaching(cache bool) Option {
      return optionFunc(func(o *options) {
        o.caching = cache
      })
    }

    // Connect creates a connection.
    func Connect(
      addr string,
      opts ...Option,
    ) (*Connection, error) {
      options := options{
        timeout: defaultTimeout,
        caching: defaultCaching,
      }

      for _, o := range opts {
        o.apply(&options)
      }

      // ...
    }

    // Options must be provided only if needed.

    db.Connect(addr)
    db.Connect(addr, db.WithTimeout(newTimeout))
    db.Connect(addr, db.WithCaching(false))
    db.Connect(
      addr,
      db.WithCaching(false),
      db.WithTimeout(newTimeout),
    )
