# 函数和方法

## 为啥值接收者和指针接收者的方法集是不同的

T 值接收者  
\*T 指针接收者

spec规定了，T的方法级只包含T的，\*T的方法集包含了T和\*T的

简单的讲，类型T有两类方法集：

1. 接收者是值，这类方法集不会修改对象
2. 接收者是指针，这类方法集会修改对象

如果对象是值，那么只能调用1中的方法集，调用2就会修改值的副本，又映射不到原对象，
就好像做了无用功，所以Go将方法级分类了，对象是值，就只能调用1的方法集。

如果对象是指针，调用1中的方法集，貌似也是可行的，因为传指针不一定必须修改原对象，
也是合理的，调用2中的方法集来修改原对象，没毛病。

## 协程中的闭包发生了什么事

    func main() {
        done := make(chan bool)

        values := []string{"a", "b", "c"}
        for _, v := range values {
            go func() {
                fmt.Println(v)
                done <- true
            }()
        }

        // wait for all goroutines to complete before exiting
        for _ = range values {
            <-done
        }
    }

典型的闭包效应,打印的结果只有小概率出现abc3个

    for _, v := range values {
        go func(u string) {
            fmt.Println(u)
            done <- true
        }(v)
    }

改进版，消除闭包效应，直接传进函数

    for _, v := range values {
        v := v // create a new 'v'.
        go func() {
            fmt.Println(v)
            done <- true
        }()
    }

推荐的写法，Go是支持的
