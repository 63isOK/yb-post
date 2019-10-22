# 缓存置换策略

## wiki introduce

在计算机中，缓存算法(cache algorithms)也被称为缓存置换算法，也叫缓存置换策略。
利用计算机程序或硬件维护的结构，缓存信息可以存储在计算机上，

缓存本身就是为了提高性能而存在的，缓存命中率越高，性能就越高。
所以将最常用的缓存在内存中，运行速度是最快的。一旦存放缓存的内存空间满了，
那算法就需要决定为新的缓存腾出哪个空间。这就是缓存置换算法。

## overview

总览

这里有个公式是讲内存引用的一个平均时间：
`T = m * Tm + Th + E`

- T 表示时间
- m 表示遗漏率(也叫非命中率) miss ratio = 1 - (hit ratio 命中率)
- Tm 表示访问未命中的内存用的平均时间，(这里也考虑到了多级缓存的情况)
- Th 延时，也叫引用缓存的时间
- E 表示各种次要影响，eg：多处理器的队列影响

缓存有两个主要影响点：延时和命中率。当然还有一些其他的次要影响点。
命中率(hit ratio)是在缓存中找到想要元素的概率，各种置换策略的目标就是提高命中率。
延时(latency)从请求到最后预期的缓存返回的时间。这个值是越小越好。
各种缓存置换策略就是比较这两个值。

命中率的测量依赖于基准程序，不同的程序，命中率会有波动。
如果是音视频程序，命中率基本为0。

以下几点也是需要考虑的：

- 花费不同的元素。元素保持的越久花费越多
- 占用不同大小的元素。一般会优先将size大的缓存先置换，这样可放多个小的缓存
- 带过期时间的元素。过期了就会被置换

部分算法也支持缓存一致性，这只适用于多个独立的缓存使用同一份数据。
eg：多个数据库服务更新同一个数据文件。

## 策略

### Bélády's algorithm

贝雷迪算法

最有效的算法应该是将未来最长时间不会用到的元素置换掉。
这最佳的结果，称为贝雷迪的最佳算法/简单置换策略/透视算法。
实际上，预测某个元素未来多长时间不会用到，是非常困难的，几乎没有实现。

因为对未来并没办法透视，所以贝雷迪算法无法实现。
下面的算法(策略)都是基于一定规则制定的。

### FIFO

first in first out，类似队列的方式。

这种算法的行为和fifo队列类似。
这时算法置换的总是最先添加的那个元素，并不考虑命中率。

### LIFO

last in first out, 类似栈的方式。

这种算法的行为正好和fifo相反。

### LRU

least recently used，最近最少使用。和贝雷迪算法的观点有点相反，
贝雷迪是未来最长时间没使用的被置换，lru是最近最少使用的被置换，
一个是基于未来(透视)，一个是基于过去(这个是明确的)。

lru算法要求跟踪缓存中的元素，如果要一直保证丢弃最近最少使用的元素，
这个代价是比较昂贵的。一般lru算法的实现中都会有个一个"年龄"来跟踪元素，
每次缓存被使用，年龄都会更新。

lru算法是一个缓存算法家族，其中包括heodore Johnson and Dennis Shasha的2Q算法，
Pat O'Neil, Betty O'Neil and Gerhard Weikum的LRU/K算法。

lru是基于访问缓存请求次数的，在实现中，可将这个次数作为一个自增的值，
赋给当前访问的元素，如果发生了置换，就置换值最小的元素。

### TLRU

time aware least recently used (tlru),带时间感知的最近最少使用算法，
也是LRU的一个变种, 适合用在网络缓存程序中，eg：icn/cdn和分布式网络。

icn: information-centric networking 信息中心网。  
cnd: content delivery networking 内容分发网。  

tlru中添加了一个新的术语：ttu(time to use)。ttu是一个时间戳，定义了元素的有效期。
tlru要确保"最不流行/最小生命周期"的元素在下次置换中被换掉。

### MRU

most recently used, 最近使用算法，和LRU正好相对，MRU会置换掉最近使用的元素。
第11界数据库顶级会议VLDB发布研究结果：在文件重复扫描中，那些迭代参数，
最近使用算法MRU是最合适的置换算法。第22界VLDB发布研究结果：在大数据集中，
随机访问和重复扫描，MRU缓存算法的命中率比其他算法高很多。
特别是某些元素存在的时间越长越容易被访问，这种情况非常适合MRU。

### PLRU

pseudo lru, 伪LRU算法。

对应cpu缓存，如果有较大关联(我理解为cpu的多级缓存)，那实现上的代价就比较大。
plru的命中率会更差一点，但是延时会更少(多级缓存，延时肯定小),这算是一种折中，
不过对于cpu缓存来说，plru确实更加适合。

为什么说是伪LRU算法，因为不是每级缓存都是LRU，考虑到cpu的特性，
每级缓存的i/o速度都不一样，所以说plru只适用于cpu缓存。

具体说明可参考[wiki](http://wikipedia.moesalih.com/Cache_replacement_policies#Pseudo-LRU_(PLRU))

### RR

random replacement,随机置换算法，和fifo/lifo类似，都不需要考虑命中率。

rr算法一般用于arm处理器中，特别是随机模拟算法。

### SLRU

segmented lru, 分段式lru。

slru缓存分两段，具体可查看wiki

### LFU

least-frequently used, 最小使用频率，和lru稍微有点不同。

ps： 会不会出现这种情况：其他元素使用非常高，后面置换会不会只置换最后一次置换对象。

### LFRU

leaast frequent recently used, 最近最少使用频率。应该是对lfu的一种补充。
可以说是lfu和lru的组合。比较适合icn/cdn和分布式网络。

lfru缓存分两段，分流行的(频率大的)和不流行的(频率小的)

### LFUDA

lfu with dynamic aging, 带动态老化的lfu

### LIRS

low inter-reference recency set, 最近最小引用集。
基于lru性能优化过的一种算法，考虑到了频次和时间。

### ARC

adaptive replacement cache,自适应置换缓存。lru和lfu的一种平衡算法。
对slru的一种改善。

### CAR

clock with adaptive replacement, 带时钟的ARC。

### MQ

multi queue，多队列。

### pannier

基于容器的缓存算法和复合对象。
