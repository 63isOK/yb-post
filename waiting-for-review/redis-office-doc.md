# 常用使用场景

## 会话存储

  好处：相比关系db，访问速度快；  
  超时机制也比较适用于会话过期设置

## 分析

  eg：计数、统计、分析

## 排行榜

  有序集合，可以方便进行排序，
速度比关系db快

## 队列

  利用list实现任务队列

## 最新N个记录

  关系db可能用limit 10来实现，redis也能

## 缓存

  redis做缓存，关系db做持久化，  
这就有以下概念：命中，击穿，雪崩