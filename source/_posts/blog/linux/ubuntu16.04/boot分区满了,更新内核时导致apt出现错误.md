---
author: qwerse
head: 
date: 2017-08-16
title: ubuntu16.04/boot分区满了,更新内核时导致apt出现错误
tags: linux
images: 
category: linux
status: publish
summary: 更新内核时由于需要安装到/boot分区，但是/boot分区是满的，无法更新软件。导致apt的安装程序收阻，这也是一个bug吧。完全无法恢复，只能扩大/boot分区，才行。
---


问题场景：

​	更新内核，出现问题，提示文件写入不了/boot分区。仔细用 df -h 一看，/boot分区占用100%了。

​		这个时候apt也用不了了，什么软件也安装不了，提示依赖问题。



问题总结：

​		更新内核时由于需要安装到/boot分区，但是/boot分区是满的，无法更新软件。导致apt的安装程序收阻，这也是一个bug吧。完全无法恢复，只能扩大/boot分区，才行。



解决方法：
#### 扩大/boot分区

​		假设有如下几个分区：

- /dev/sda1      /boot      500M
- /dev/sda2     /                400G
- /dev/dm-0    swap         4G
- /dev/sda3                        2G   #新的分区，用来替换现行的/boot分区

#####格式化新分区为ext4,并设置为主分区：

​	在图形界面使用gparted这个软件，sudo apt-get install gparted。来进行硬盘操作。

具体的使用方法我就不多说了，这不是主题，既然会玩linux，这就不是问题。

##### 挂在新的分区到/boot，替换旧的/boot分区

这个有个必须的操作，需要把旧分区的内容全部复制到新分区上

​	

​	挂在新硬盘到/mnt分区：sudo mount /dev/sda3 /mnt 

​	把旧/boot分区的内容复制到新分区上：sudo rsync -av /boot/ /mnt/

​	卸载原/boot分区：sudo umount /boot

​	卸载新分区所在的mnt文件夹：sudo umount /mnt

​	挂在新分区到/boot下：sudo mount /dev/sda3 /boot

##### 把更改信息添加到/etc/fstab文件，这是必须的，不然开不了机

​	查看新分区的UUID信息：sudo lbkid	 #查看/dev/sda3分区的UUID

​	如下例子：

​			UUID=883fa013-48ba-473e-b8bc-5c4d910872ff	/boot	ext4	defaults	02

​			把新分区的UUID写到到文件里，覆盖旧分区的UUID。



###重启开机，运行sudo apt-get install -f  就行了

​	











​				