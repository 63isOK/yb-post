# 使用kurento需要理解的几个点

1. kurento应用和web应用有什么相同点和不同点
2. kurento应用的结构分了几层,每一层做了哪些事(每层各有什么能力)
3. 进行媒体传输时,可以分成几个阶段,每个阶段具体发生了哪些事
4. sdp处理过程是怎样的,谁发起,谁处理的
5. ice候选匹配的状态变化是怎样的
6. 如果要实现一个推流,多个看,有哪几种实现方案
7. kurento中,支持的数据源有哪些种类
8. 为什么需要一个applicate server
9. kurento的垃圾回收机制启用的时长是多少,在哪儿修改
10. kurento和coturn是怎么互动的
11. kurento client api有几种, api是谁来调用的
12. 实现kurento c++ client api的思路是什么样的
13. 使用kurento,能解决什么场合下的问题,也是是应用场景的问题
