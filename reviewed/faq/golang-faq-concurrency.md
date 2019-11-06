# 并发

## 哪些操作是原子的，互斥锁？

Go内存模型中详细讲原子操作。

sync和sync/atomi包提供了底层的同步和原子原语

高层次的操作(eg:并发服务之间的协调)，需要编码来解决，此时可以使用协程和通道。

Do not communicate by sharing memory. Instead, share memory by communicating.

通过通信来共享内存，而不是共享内存来通信

## 为啥程序不花费更多的cpu个数来跑得更快呢

Go语言提供的是并发，某些场景下并发=并行，但，不是所有场景都是。

本质上顺序执行的，是无法通过更多的cpu来解决的。

## 如何控制使用cpu的个数

GOMAXPROCS 环境变量

## 为啥协程没有ID来标识

匿名工作流，不需要名字
