---
author: QwerSe
head:
date: 2018-07-06
title: cobbler配置的安装和坑
tags: cobbler

category: linux
status: publish
summary: 能进安装界面，但是卡死不动，是ks文件没有配置好。
---



#####  先说一下坑

- url --url="http://127.0.0.1/cobbler/ks_mirror/Centos-7.4mini-x86_64/" 

  ks文件里一定要指定`url --url`,127.0.0.1一定要替换局域网里面的ip

- 能进安装界面，但是卡死不动，是ks文件没有配置好。怎么获取好的配置文件呢？最简单的方式就是用光盘安装一遍。进入root主目录有个ana开头的文件，把里面的内容复制下来就行了。我一般喜欢用图形安装方式，比较不容易出现问题。

#### 安装配置步骤

网上教程太多，重复的造轮子也没有什么意义

选择一篇我自己实践过的文章，写的非常的好。

[文章（点击下载）](https://www.jianguoyun.com/p/DROpJUgQqeD4BhighWE)

附一下
我的ks文件，是centos7的图形安装
```
#version=DEVEL
# System authorization information
auth --enableshadow --passalgo=sha512
# Use CDROM installation media
install
# Use graphical install
graphical
# Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=sda
# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8

# Network information
network  --bootproto=dhcp --device=eth0 --onboot=off --ipv6=auto --no-activate
network  --hostname=localhost.localdomain

# Root password
rootpw --iscrypted $6$KISrAVI4.I92wl5M$Q4vAA3tqkoXOq0otYYUBEwtHdb1SB9YttiEBzOkcHhALakx/1sUoh1y83bSB2zUFvtTd6Zgb9Xq4lxhJ.hE.y/
# System services
services --enabled="chronyd"
# System timezone
timezone America/New_York --isUtc
# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
autopart --type=lvm
# Partition clearing information
clearpart --none --initlabel

%packages
@minimal
@core
chrony
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
```













