# BST 二叉搜索树

binary search tree

## wiki introduce

在计算机领域，bst有时叫有序二叉树/存储二叉树，
是一种特殊的容器(存储item的数据结构)，提供了快速查找，新增/删除item，
也可以动态修改抽象数据类型，也可以基于索引表利用key来查找一个item

bst的key是按顺序存储的，所以搜索和其他操作都可以利用二叉搜索原则。
当在树中搜索一个key或新增一个key，会从root遍历到叶子节点，
会和节点中存储的key进行比较，基于比较结果，决定在左子树或右子树继续进行搜索。

一般情况，每比较一次，就会跳过一半的子树，每次操作的时间复杂度是O(n),
空间复杂度是O(log n), 这比未排序的数组效率(线性的)高不少，
比hash表的效率要低

## BST的定义

bst首先是带根节点的二叉树，内部节点都会存储key(也可能会有附加的value)，
且有两个子树。节点里值的规则是：
节点key要大于等于左子树的key，并小于等于右子树的key;
叶子节点不包含key，所以也就无法和其他叶子节点进行区分。

通常，node记录的信息是一个记录，而不是单个数据元素。
出于排序目的，比较是基于key来比较，而不是基于记录来比较。
二叉搜索树bst相对其他数据结构来说，优势在于存储算法和搜索算法。
有序遍历效率高，且易于编码设计。

bst是一个基础的数据结构，通常用于构造抽象数据结构，eg：set，multisets，关联数组。

- bst中的搜索和新增，会利用节点的key进行比较
- bst的形状依赖增删的顺序，可以被退化(退化成链表)
- 随机进行大量增删后，高度的平方会趋进于key数量
- 为了阻止退化，会导致时间复杂度趋于O(n)

## 顺序关系

顺序关系决定了bst节点的位置。
节点是依据key来进行比较排序的，如果两个节点的key是一样的，顺序关系就无法决定了，
这只能由程序来决定是否运行key重复。

## 操作

bst主要操作是增删查。

### 查 search

bst的查是通过key取查，一般的编程实现有递归和迭代

一般做法如下：

1. 检查root节点是否为空，为空 = 未找到; 执行2
2. 比较key，相等 = 找到，小于当前节点的key，执行3, 大于执行4
3. 在左子树中查找，如果无左子树 = 未找到
4. 在右子树中查找，如果无右子树 = 未找到

使用递归和迭代的方式都可以很方便写出代码

```go

type BST struct {
  root *Node
}

type Node struct {
  key, value interface{}
  left, right *BST
}

// 递归
func (bst *BST) search(k interface{}) *Node {
  return recursive(bst, k)
}

func recursive(bst *BST, k interface{}) *Node {
  if bst == nil || bst.root == nil {
    return nil
  }

  if bst.root.key == k { // 伪代码，实际使用中还需要进一步判断
    return bst.root
  } else if bst.root.key > k {
    if bst.root.left == nil {
      return nil
    }

    return recursive(bst.root.left.root) // 插入的时候会保证root非空
  } else {
    if bst.root.right == nil {
      return nil
    }

    return recursive(bst.root.right.root) // 插入的时候会保证root非空
  }
}

// 迭代

fun (bst *BST) search(k interface{}) *Node {
  cur := bst

  for cur != nil && cur.root != nil {
    if cur.root.key == k {
      return cur.root
    }

    if cur.root.key > k {
      cur = cur.root.left
    } else {
      cur = cur.root.right
    }
  }

  return nil
}
```

### 新增

新增之前，也需要依据key来查找要加入的位置，找到后再插入。
实现方式也可基于递归或迭代。

## 删除

删除会触发树的调整

删除规则：

- 删除的节点没有子节点，直接删除即可
- 删除的节点只有一个节点，字节点的位置移到删除节点即可
- 删除的节点A有2个字节点
  - 先找右子树的最左节点B
  - B节点如果有右子节点，就找出来，叫C
  - B移到A的位置，C移到B的位置

## 遍历

递归遍历是最可取的方式，迭代遍历太复杂

## 验证是否是bst

验证的方式：

如果是父节点的左子节点，第一判断是否比父节点的key小，
其次要和父节点的右子树所有节点比较

父节点的右子节点也是类似的。

## 应用

### 排序

二叉搜索树可以用于实现简单的排序算法，和堆排序类似。
二叉搜索树bst最差的情况是没有左节点，全是右节点(此时退化成链表了)，
时间复杂度是O(n的二次方)。也有很多方法来解决这种问题，
较常见的就是self-balancing binary search tree(自平衡二叉搜索树)。
自平衡二叉搜索树的时间复杂度是O(n * log n),她是比较排序算法中，
渐近最优的一种，实际过程中，比适用于静态列表排序的渐近最优算法(eg:堆排序)
差很多(不管是时间还是空间)，主要是受节点申请的影响。从另一面讲，
增量式排序中，自平衡二叉树是最优秀的，因为任何时候都是排序了的。

### 优先队列

利用二叉树搜索和任意删除特性，就比较适合实现优先队列。

在这点上，二叉堆也实现了差不多的功能，花了差不多的资源(时间空间)，
但bst可以同时做find-min/find-max/delete-min/delete-max，
而二叉堆只能实现一种，所以bst实现的优先队列也被称为双端优先队列。

## bst的种类

二叉搜索树的种类很多:

- avl(平衡树)和red-black(红黑树)都是自平衡二叉搜索树的形式
- splay tree(伸展树/分裂树)会自动将最常用的节点移到根节点附近
- treap(树堆)每个节点都有一个随机的优先级，父节点的优先级高于子节点
- tango树是基于快搜进行优化了的树
- t树是基于bst优化，减少了存储空间消耗的树,主要用于内存数据库。

退化树是最差的情况，也不是平衡的。

### 性能比较

2004年的一份资料表示：treap(树堆)的平均性能是最好的，红黑树也很接近。

### bst的优化

bst最主要还是在于搜索，可以基于搜索进行一定的优化。
[具体信息](http://wikipedia.moesalih.com/Binary_search_tree#Optimal_binary_search_trees)
