# container/list 双向链表

## package分析

list包实现的是双向链表,组成的数据结构有两个：Element 表示节点，List 表示链表,
list对外暴露的能力都是通过这两个数据结构来提供的。

## Element 分析

```text
 [fields]
+Value : interface{}
-list : *List
-next : *Element
-prev : *Element
 [methods]
+Next() : *Element
+Prev() : *Element
```

可以看出就是一个简单的链表节点：

- 节点值 Value
- 前后节点指针 next和prev

值得注意的是值是直接暴露的，而前后节点指针是通过方法去获取的，
并不是遵循了一致性，而是更加强调了实用性。其二是多加了一个链表属性，
好处是节点和链表的关系是聚合关系，而不是组合关系，
(组合和聚合都是整体和部分的关系，组合中的部分不能离开整体而独立存在，
聚合则是可以)。这个链表属性并未通过其他方式暴露，仅作为校验用。

```go
// Next returns the next list element or nil.
func (e *Element) Next() *Element {
    if p := e.next; e.list != nil && p != &e.list.root {
        return p
    }
    return nil
}

// Prev returns the previous list element or nil.
func (e *Element) Prev() *Element {
    if p := e.prev; e.list != nil && p != &e.list.root {
        return p
    }
    return nil
}
```

从上面可以看出:

1. 第一个节点的前指针，指向root，最后一个节点的后指针指向root
2. 链表的内部实现其实是一个ring环，root作为一个特殊节点，并不会存储实际的数据
3. 双向链表list的底层就是一个环链
4. 对外暴露中，取最后节点的下一节点/取起始节点的前一节点返回nil
5. 节点Element的list属性，仅仅是将节点和链表关联的一个属性

## List 分析

```go
// List represents a doubly linked list.
// The zero value for List is an empty list ready to use.
type List struct {
    root Element
    // sentinel list element, only &root, root.prev, and root.next are used
    len  int     // current list length excluding (this) sentinel element
}
```

List链表的属性都是不暴露的，root表示特殊锚点，len表示链表长度

List链表对外暴露的功能如下：

- 取第一个/最后一个节点
- 取链表长度
- 对链表的增删改(查是通过遍历的方式来查)
- 对外暴露了一个新函数New用来创建一个新的空链表

```go
// Front returns the first element of list l or nil if the list is empty.
func (l *List) Front() *Element {
    if l.len == 0 {
        return nil
    }
    return l.root.next
}

// Back returns the last element of list l or nil if the list is empty.
func (l *List) Back() *Element {
    if l.len == 0 {
        return nil
    }
    return l.root.prev
}

// Len returns the number of elements of list l.
// The complexity is O(1).
func (l *List) Len() int { return l.len }
```

取第一个和最后一个节点，利用root节点可方便实现，
长度也有对应的数据记录，实现也比较简单。

```go
// 链表增删改的一些辅助函数(不导出)

// insert inserts e after at, increments l.len, and returns e.
func (l *List) insert(e, at *Element) *Element {
    n := at.next
    at.next = e
    e.prev = at
    e.next = n
    n.prev = e
    e.list = l
    l.len++
    return e
}

// insertValue is a convenience wrapper for insert(&Element{Value: v}, at).
func (l *List) insertValue(v interface{}, at *Element) *Element {
    return l.insert(&Element{Value: v}, at)
}

// remove removes e from its list, decrements l.len, and returns e.
func (l *List) remove(e *Element) *Element {
    e.prev.next = e.next
    e.next.prev = e.prev
    e.next = nil // avoid memory leaks
    e.prev = nil // avoid memory leaks
    e.list = nil
    l.len--
    return e
}

// move moves e to next to at and returns e.
func (l *List) move(e, at *Element) *Element {
    if e == at {
        return e
    }
    e.prev.next = e.next
    e.next.prev = e.prev

    n := at.next
    at.next = e
    e.prev = at
    e.next = n
    n.prev = e

    return e
}

```

insert是在某个节点后插入一个新节点，insertValue是便捷的封装而已，
remove是删除一个节点，move是将一个节点移到另一个节点后面。

```go
func (l *List) InsertBefore(v interface{}, mark *Element) *Element {
    if mark.list != l {
        return nil
    }
    // see comment in List.Remove about initialization of l
    return l.insertValue(v, mark.prev)
}

func (l *List) InsertAfter(v interface{}, mark *Element) *Element {
    if mark.list != l {
        return nil
    }
    // see comment in List.Remove about initialization of l
    return l.insertValue(v, mark)
}

func (l *List) PushFront(v interface{}) *Element {
    l.lazyInit()
    return l.insertValue(v, &l.root)
}

func (l *List) PushBack(v interface{}) *Element {
    l.lazyInit()
    return l.insertValue(v, l.root.prev)
}

func (l *List) PushFrontList(other *List) {
    l.lazyInit()
    for i, e := other.Len(), other.Back(); i > 0; i, e = i-1, e.Prev() {
        l.insertValue(e.Value, &l.root)
    }
}

func (l *List) PushBackList(other *List) {
    l.lazyInit()
    for i, e := other.Len(), other.Front(); i > 0; i, e = i-1, e.Next() {
        l.insertValue(e.Value, l.root.prev)
    }
}
```

Push系列是在首尾添加节点，Insert系列是在链表中间添加节点。
Push还支持添加其他链表

```go
func (l *List) Remove(e *Element) interface{} {
    if e.list == l {
        // if e.list == l, l must have been initialized when e was inserted
        // in l or l == nil (e is a zero Element) and l.remove will crash
        l.remove(e)
    }
    return e.Value
}

func (l *List) Init() *List {
    l.root.next = &l.root
    l.root.prev = &l.root
    l.len = 0
    return l
}
```

Remove是删除一个节点，Init是删除整个链表的节点

```go
func (l *List) MoveToFront(e *Element) {
    if e.list != l || l.root.next == e {
        return
    }
    // see comment in List.Remove about initialization of l
    l.move(e, &l.root)
}

func (l *List) MoveToBack(e *Element) {
    if e.list != l || l.root.prev == e {
        return
    }
    // see comment in List.Remove about initialization of l
    l.move(e, l.root.prev)
}

func (l *List) MoveBefore(e, mark *Element) {
    if e.list != l || e == mark || mark.list != l {
        return
    }
    l.move(e, mark.prev)
}

func (l *List) MoveAfter(e, mark *Element) {
    if e.list != l || e == mark || mark.list != l {
        return
    }
    l.move(e, mark)
}
```

Move系列是节点移动

```go
func New() *List { return new(List).Init() }
```

New就是初始化一个新的链表

## 源码读完之后的分析

链表和节点是双向链表中的概念，也有对应的数据结构，
节点的值类型是interface{},所以同一链表是支持不同类型的值。

底层使用环链，在处理首尾节点相关的操作时，确实很方便，
todo：需要自己实现一个非环链的双向链表，来对比测试两者的性能。
可以做一定次数的增删改查的基准测试

链表的遍历应该是：

```go
for e := l.Front(); e != nil; e = e.Next() {
}
```

## 测试

- 空链的root节点的前后指针，要么是root(已初始化)，要么是nil(未初始化)

分析：

- 测试的时候，对底层环链和对外提供的功能的行为都进行了测试
- 写测试例子时，将测试逻辑放在函数里，
好处是可以在测试逻辑反复对暴露函数进行测试
- 对底层环链和对外暴露的功能封装在一个公用的函数里，让测试逻辑部分只专注于逻辑
- 覆盖所有测试边界
- 这种测试的写法是值得学习的

TODO:  
可以实现一个非环链的，进行基准测试
