# 值

## 数值转换为啥没有隐式转换

显示，减少疑惑

## 常量是如何玩的

常量就是一个固定值，至于什么类型，就看关联什么类型

无类型常量就是常用的常量

## map为啥是内置类型

用的多

## map的key为啥不能是slice

key是需要支持比较操作的，slice没有比较。

## 为啥map/slice/channel是引用，而array是值传递

早期map和channel在语法上就是指针，排除了值传递的可能

而slice和array是一个东西的不同表现，在设计中有一个折中的考虑
