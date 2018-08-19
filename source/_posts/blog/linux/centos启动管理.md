---
author: QwerSe
head: 
date: 2017-9-19
title: centos启动管理
tags: linux
images: http://pingodata.qiniudn.com/cube2.jpg
category: linux
status: publish
summary: 当密码忘记的时候可以这么做
---



当密码忘记的时候可以这么做

1. 在开机界面的启动条按"e"编辑 在内核条上再按"e"编辑

   在quit后面添加一个 "1"，和quit要有一个空格。完成后返回按'b'，就能启动了

   就能进入单用户模式，改密码了。

2. 按e进不去编辑界面，要输入p，提示输入密码才给编辑，这是grub被锁的原因，这个时候只能用光盘模式修复了！

   这就像windows的pe模式一样的可以改grub的密码。

   进入光盘的shell。输入以下命令：

   `chroot mnt/sysimge`

再输入passwd root

就能改密码

这是centos6左右的改密码，7有所不同