---
author: qwerse
head: 
date: 2017-07-13
title: windows下硬盘安装linux(全能版)
tags: windows
images: 
category: windows
status: publish
summary: 这个方法几乎能安装所有的操作系统，关键在于那10个G的硬盘，和启动文件的放置。
---









本文主要是对windows下硬盘安装linux进行知识梳理。
用到的环境：win7(64)+centos6.9


		

软件下载地址：http://qwerse.hk.ufileos.com/linuxyp安装.7z
### 安装所需软件

1.分区助手专业版(必需)：用来对硬盘分区，将磁盘的一部分格式化成Linux可以识别的ext3格式

2.extfs ：主要是在windows下可以读写ext3的分区

3.EasyBCD(必需)：用于添加和修改启动项

4.WinGrub(建议使用)：用于看分区编号

5.动态磁盘转换器(视情况)：Linux只能安装在基本磁盘，我是因为第一次安装错误导致磁盘变成了动态磁盘，所以需要用到这个，关于动态磁盘的概念请自行百度，一般现在的系统都是基本磁盘。

6.CentOS-6.9.iso : 镜像文件






CentOS的安装文件需要自行另外下载(各大高校的ftp上应该有)或者购买安装光碟。

下面主要介绍硬盘安装双系统的方法，光盘安装的步骤会少些，也一并介绍了：

### 在Windows7上面的准备工作

打开磁盘管理器：
Win徽标键+R键调出运行框，输入指令diskmgmt.msc回车调出磁盘管理器。在最后一个分区上面右键选择压缩卷，如果磁盘空间不够的话一定注意要备份该分区上的数据哦


！！！也可以选择删除卷，但这样做的话一定要将数据备份

！！！压缩多少视自己硬盘情况和想要Linux系统占用多大空间而定，硬盘大的话建议100G以上。这步完成后磁盘上会出现一部分未分配空间，如下图：

安装分区助手专业版:
![](http://photo.blog.sina.com.cn/showpic.html#blogid=86e874d30101e3d8&url=http://album.sina.com.cn/pic/002taUQrzy6IcO7hONC74)

		在磁盘未分配空间上右键->创建分区，创建一个10GB的分区用于存放CentOS 6.9的安装文件(光盘安装可以跳过这步)，

		这里的文件系统说明一下，如果是安装32位的CentOS的话，因为安装文件小于4GB，可以选择使用FAT32文件系统。如果是安装64位的CentOS，则只能使用EXT3的文件系统。在高级选项中记得选择创建为“逻辑分区”。

		确定后在主界面选择提交执行。

在剩下的未分配空间上创建一个Ext3格式的主分区，方法同上。
![](http://s4.sinaimg.cn/mw690/002taUQrzy6IcP7rDnd73&690)

		但是注意必须在高级选项中选择创建为“主分区”，如果“创建为”后面是不可以选择为“主分区”的，则说明主分区的数量已经到达上限，可以用下面方法将其中一个原本为“主分区”磁盘转换为“逻辑分区”。



安装extfs软件


安装完成后在分配的10GB磁盘空间上右键->Service Management，在弹出的页面选点Start，再点Apply。如下图，注意勾选项：
![](http://s12.sinaimg.cn/mw690/002taUQrzy6IcRGX6C77b&690)

然后在“我的电脑”中就可以看到ext3格式的10GB的空间了。

将CentOS-6.9-x86_64.iso也复制到10GB的存放分区中。
![](http://s14.sinaimg.cn/mw690/002taUQrzy6IcSimAsB2d&690)


安装WinGrub，在Tool->Partition List打开的页面中找到存放10GB的分区的编号：如：我这里是(hd0,5)，记下来这个编号，后面有用。
![](http://s16.sinaimg.cn/mw690/002taUQrzy6IcSOUhX12f&690)
![](http://s4.sinaimg.cn/mw690/002taUQrzy6IcSTcAsrc3&690)

安装EasyBCD，打开后找到Add New Entry，点NeoGrub标签。点Install然后点Configure。
![](http://s6.sinaimg.cn/mw690/002taUQrzy6IcThA5c915&690)
		在弹出的文档中加入下面内容，这里的(hd0,5)是上一步中找到的分区编号

		
		title Install CentOS

		kernel (hd0,5)/isolinux/vmlinuz

		initrd (hd0,5)/isolinux/initrd.img

		保存后重启。

### 开始安装


！！！
。。。。





这里就会提示要写磁盘了：
![](http://s6.sinaimg.cn/mw690/002taUQrzy6Id01Enrf05&690)

这里一定要更改设置，否则他会默认把启动文件放到Windows所在分区，这样会损坏Win7的启动文件，造成Win7无法启动！！！


建议选Desktop，这个安装最完整：


好了，完成：


3.启动进入Win7之后打开EasyBCD,在Add New Entry的NeoGrub标签下点Remove。然后如图添加一个CentOS 6.9的启动项，安装了CentOS系统的磁盘分区一定要选对：

Add Entry重启后就可以进入CentOS 6.9了。

4.原来放CentOS系统安装文件的那10GB的分区可以删除了还给Win7使用，这步之后磁盘号可能会发生变化，导致不能进入CentOS，这种情况下需要重复一下上面的第3步。

