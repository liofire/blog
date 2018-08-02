---
author: QwerSe
head:
date: 2018-07-20
title: 出现PAGE_FAULT_IN_NONPAGED_AREA错误
tags: windows

category: windows
status: publish
summary: 出现PAGE_FAULT_IN_NONPAGED_AREA 错误解决方式
---




![](http://ww1.sinaimg.cn/large/ab318c02gy1ftg6ekwd9gj20cw0h8q49.jpg)

连续4台电脑出现这个问题往上面找了，说是内存的原因，感觉不对。

怎么感觉都是系统的原因，同事老王说是windows打补丁的结果，-_-||。

最后还是老王 在pe下使用`chkdsk c: /f /r` 进行了修复，重启果然就好了。。

！！！注意c是系统盘