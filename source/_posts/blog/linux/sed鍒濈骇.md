---
author: QwerSe
head: 
date: 2017-09-11
title:  sed的基本用法
tags: linux

category: linux 
status: publish
summary: 要记住一点''和""区别，''会把字符串里面的特殊字符给字面化，""会保留特殊的字符含义
---

###  sed匹配打印
主要是使用-p参数进行打印

![](http://ww1.sinaimg.cn/large/ab318c02gy1fjfg27xdtfj20l10fawfh.jpg)



我最喜欢的还是sed -n '1~2p' filename
从第1行开始，每各2个打印一次
seq 10 | sed '1~2p' :
1
3
5
7
9

### sed插入、修改、删除、替换
![](http://ww1.sinaimg.cn/large/ab318c02gy1fjfg5aejo7j20im0fkgmi.jpg)

### sed与shell的交互
要记住一点''和""区别，''会把字符串里面的特殊字符给字面化，""会保留特殊的字符含义，下面我举个例子：
```
#A='qwerse'
#echo 'this $A'
this $A
#echo "this $A"
this qwerse

```

![](http://ww1.sinaimg.cn/large/ab318c02gy1fjfgggn178j20ct06t74k.jpg)

一定要避免在脚本中出现此类错误！
这是常用的用法，更高级的还需要了解缓存区，想要深入去百度搜搜。



