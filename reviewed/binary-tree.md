# 二叉树

binary tree，简称BT。

## wiki introduce

[link](http://wikipedia.moesalih.com/Binary_tree)

在计算机学科中，bt是一种树数据结构(每个节点最多两个子树：左子树和右子树)。
这是一种递归定义的概念，每个非空bt都可以定义成一个三元组(l,s,r)。
其中l和r也是一个bt，也可能是空集合。s就是一个单例集合(数学中的一个单位，
表示一个确切的元素)。当然，`二叉树是可以为空树的`

在图论中，bt树是一种树状结构，bt也被称为分叉树。
分叉树这一术语是在一些老旧的编程书籍上才出现过，在现代计算机术语流行之前。
也可以将bt解释为`无向图`(而不是有向图);也有作者用带根节点的bt来表示树是带根的。
k-ary树，在图论中也是一种树，节点可以有多个子树。
k-2树是一种特殊的树，一个节点最多只有两个子树，也就是我们提到的bt。

在数学上中，bt(二叉树)被很多作者下了不同定义，有的是上面的二叉树定义，
有的只是将bt定义为：非叶子节点有两个子树，对排序并没有要求。

在计算机领域，bt被用于两种场景：

1. 通过和节点相关值或标签来访问节点。这种带标签的二叉树常用于实现
二叉树(binary search tree, bst)或二叉堆(binary heap, bh)，分别用于
搜索和存储。非根节点是以左右子树出现，即使只有一个子树，也是正常的。
如何将节点添加到二叉树，和二叉数概念并无关系。
2. 一个分叉树结构，常用于哈夫曼编码和分支图。
文档分为charpter章节/section小节/paragraph段落就是一种典型的k-ary树(不是bt)

## 二叉树bt的定义

### 递归定义

二叉树bt，可能只有一个子树，有些书上添加节点称为扩展二叉树，递归定义如下：

- 空结合是可以用于扩展二叉树的
- 如果用t1和t2来扩展二叉树，可以用t1.t2表示根节点的左子树是t1,右子树是t2,
此时，新扩展二叉树(添加一个新的节点)，如果有子树存在，
那就以一个叶子节点添加到子树中

理解： 可以有空树，新增节点时，如有子树，就添加子树中，如果子树还有下一级节点，
那新增的节点会添加到下一级的节点中，如此递归。
至于新增的排序规则是如何的，就不是二叉树定义的，那涉及到前序/中序/后序的概念。

### 图论中的概念

图论中，bt是带根的树，同时也是有序树。
有序树中是存在level的概念，也就是深度，到根节点的层数。那么子节点可以这么理解：
与当前节点相连，且level大一层的节点就是当前节点的子节点。
而排序，是用于区分左右子节点的，区分不了[只有左子树的节点和只有右子树的节点]。

左右子节点的区分，有时是必要的，有各种二叉树bt都有各自的定义：

- 每个节点都有左子树
- 每个节点都有右子树
- 每个节点都有左右子树，或者每个节点都没有子节点

## 二叉树的分类

关于树的术语，并没有统一的标准，每本书都有可能略有不同。

- 带根节点的二叉树 rooted binary tree
  - 有一个root节点，每个节点最多有两个子节点
- 满二叉树 full binary tree
  - 每个节点的子节点树要么是0要么是2
  - 递归定义如下：
    - 单个顶点
    - root节点有两个子树，每个子树都是满二叉树
- 完全二叉树 complete binary tree
  - 除了最后一层，其他层的节点位都被填满了
  - 最后一层就算有数据，也是从左到右填满
  - 如果层数(level)是h，节点数是[1,2的h次方)
  - 还有一种定义是：完美二叉树去掉了最右侧的几个叶子节点
  - 完全二叉树是可以用数组呈现出来的
- 完美二叉树 perfect binary tree
  - 内部节点(相对于叶子节点的枝干节点)有两个子节点
  - 所有的叶子节点有相同的level和depth(左右子树最大的层数)
  - 就是一个特殊的完全二叉树
  - 或者说叶子节点只出现在最后一层且内部节点都有2个子节点
  - 特别像常规的血统图：每个人都有两个亲生父母,如果按性别决定左右，那就是pbt
  - 完美二叉是完全二叉树的一个特例
- 无限完全二叉树 infinite complete binary tree
  - 每个节点都有两个子节点
  - 层level是无限的
  - eg： stern-Brocot树
- 平衡二叉树 balanced binary tree
  - 左右子树的高度相差不能超过1
  - 不同的平衡二叉树方案决定了不同叶子节点之间的距离
  - BBT和其他二叉树是完全不一样的
- 退化树/病理树 degenerate/pathological tree
  - 每个父节点只有一个关联的子节点(也就是说每个枝干节点最多有一个字节点)
  - 此时，树可以被理解为链表数据结构

## 二叉树的属性

下面用n表示节点数量，h表示层(level/height),只包含root的树，高度h是0,
用l表示叶子节点数

- 满二叉树 n最小是 2h+1, 最大是2的(h+1)次方-1
- 后面都是一些数学公式
