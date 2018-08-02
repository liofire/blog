---
author: QwerSe
head: 
date: 2017-09-12
title:  awk使用方法
tags: linux

category: Linux
status: publish
summary: awk最重要的功能就是对文本数据进行出来，他比其他文本处理工具更加强大也复杂
---
awk是个比较高级的程序，它本身也是一个解释性的编程语言。

awk最重要的功能就是对文本数据进行出来，他比其他文本处理工具更加强大也复杂。

基本常用的格式：

```
#awk -F: '{print $1}' /etc/passwd
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
operator
games
ftp
nobody
systemd-bus-proxy
systemd-network
dbus
sshd

```

上面是最常用的方法。

---

深入一点，awk自成一门编程语言肯定也会有if等逻辑语句

- 常用条件操作符

  ##### <    <=    ==     !=     >=    ~     !~

- 匹配

  ```
  awk -F: '{if (1~/root/)   print 0}' /etc/passwd

  awk -F: '$1~/root/' /etc/passwd

  上面两条结果都是：root❌0:0:root:/root:/bin/bash

  ```

  ​

  ​

  ​

  #####  这两条结果都相同

- 精确匹配

  `awk -F: '{if($3=="0") print $0}' /etc/passwd`

  要记住后面print $0所打印的只是if的结果也就是：

  ```
  root❌0:0:root:/root:/bin/bash

  把print 0 改成 print $3 打印出的结果是0
  ```

- 不匹配

  `awk -F: '$3 != "0"' /etc/passwd`

  结果

  ```
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
  sync:x:5:0:sync:/sbin:/bin/sync
  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
  halt:x:7:0:halt:/sbin:/sbin/halt
  mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
  operator:x:11:0:operator:/root:/sbin/nologin
  games:x:12:100:games:/usr/games:/sbin/nologin
  ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
  nobody:x:99:99:Nobody:/:/sbin/nologin
  systemd-bus-proxy:x:999:998:systemd Bus Proxy:/:/sbin/nologin
  systemd-network:x:998:997:systemd Network Management:/:/sbin/nologin
  dbus:x:81:81:System message bus:/:/sbin/nologin
  sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
  ```

  打印出了以:为分隔符，但是$3位置结果不是0的行。

- 比较

  `cat 2 |awk -F: '{if($3==$4) print $1,$3,$4}'`

  结果

  ```
  root 0 0
  bin 1 1
  daemon 2 2
  nobody 99 99
  dbus 81 81
  sshd 74 74

  ```

  当然也可以使用<和>

- 使用正规则表达式

  `cat 2 |awk -F: '$1 ~ /^root/'`

  结果

  `root:x:0:0:root:/root:/bin/bash`

- 逻辑运算符

  和 && 

  或 ||

  取反  !

  会shell的基本都知道这几个东西，主要是用来进行多个表达式判断用的

  `cat 2 |awk -F: '{if(1==1 && 2==3)print $0}'`

  结果会是0，这就是&&和的意思，两个判断式子都对才行

  ||两个判断是对其中一个也行

  ！取反 对是错，错是对

  ---

  AWK高级进阶

- awk内置字符串函数

  ```
  -gsub(r,s) 在整个$0中用s代替r #awk 'gsub(/root/,1234)' {print $0} /etc/passwd
  -gsub(r,s,t) 在t行中用s代替r  #awk  'gsub(/root/,"deng") {print $0}' /etc/passwd #！deng一定要是带引号。
  -index(s,t) 返回s中字符串t的第一位置 # awk  '{print index("qwerse","e")}'
  -length(s) 返回s长度 awk  '{print length($0)}' /etc/passwd
  -match(s,r) 测试s是否包含匹配r的字符串 #awk -F: 'BEGIN {print match("root1",root)}'
  -split(s,a,fs) 在fs上将s分成序列作为一个数组储存在a #awk  'BEGIN {print split("qwerasdrfdsfr",a,"r") }END{print a[2]}'  /dev/null
  -printf(fmt,exp) 返回fmt格式化后的exp cat 2 | awk -F: '{print $3=sprintf("%20d",$3)}'



  ```

  ​