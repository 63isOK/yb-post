# 代码编写

## 文档如何记录

godoc，一个go程序，从源码中提取文档，并在web上展示。

Go官网golang就是用godoc实现的。

## 是否有个Go编码规范

没有

这个可以参考uber的规范

## 如何给标准库提补丁

先邮件讨论，再按规范提交

## go get为啥使用https

一般公司经常使用80/443端口(http/https)，其他端口都被屏蔽掉了，
eg：9418/22(git/ssh).

所以第一是为了可用，第二是为了安全

## 是否应该用go get来管理包版本

是的
