---
author: QwerSe
head: 
date: 2017-11-30
title:  lamp构架下的网站环境搭建
tags: Linux
images: 
category: Linux
status: publish
summary: 查资料发现是CentOS 7 版本将MySQL数据库软件从默认的程序列表中移除，用mariadb代替了。
---

## apache2.4编译安装
httpd依赖于apr, apr-util

apr全称为apache portable runtime，能实现httpd跨平台运行

    httpd-2.4 依賴于1.4＋及以上版本的apr

        apr-1.5.0.tar.bz2
        apr-util-1.5.3.tar.bz2
        httpd-2.4.9.tar.bz2
    
        pcre-devel包
        openssl-devel     


    2、编译安装 
    # yum install gcc
    # yum install pcre-devel
    
    # tar xf apr-1.5.0.tar.bz2
    # cd apr-1.5.0
    # ./configure --prefix=/usr/local/apr   (--prefix指定apr安装的目录)
    # make
    # make  install
    
    # tar xf apr-util-1.5.3.tar.bz2
    # cd apr-util-1.5.3
    # ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
    # make && make install ###该项被漏掉，补充@20160714


    # tar xf httpd-2.4.9.tar.bz2
        以下为几个主要的配置项
        --sysconfdir=/etc/httpd24  指定配置文件路径
        --enable-so  启动模块动态装卸载
        --enable-ssl 编译ssl模块
        --enable-cgi 支持cgi机制（能够让静态web服务器能够解析动态请求的一个协议）
        --enable-rewrite  支持url重写     --Author : Leshami
        --with-zlib  支持数据包压缩       --Blog   : http://blog.csdn.net/leshami
        --with-pcre  支持正则表达式
        --with-apr=/usr/local/apr  指明依赖的apr所在目录
        --with-apr-util=/usr/local/apr-util/  指明依赖的apr-util所在的目录
        --enable-modules=most      启用的模块
        --enable-mpms-shared=all   以共享方式编译的模块
        --with-mpm=prefork         指明httpd的工作方式为prefork
    
    # cd httpd-2.4.9
    # ./configure                           \
        --with-apr=/usr/local/apr           \
        --with-apr-util=/usr/local/apr-util \
        --prefix=/usr/local/apache \
        --sysconfdir=/etc/httpd24  \
        --enable-so                \
        --enable-ssl               \
        --enable-cgi               \
        --enable-rewrite           \
        --with-zlib                \
        --with-pcre                \
        --with-mpm=prefork         \
        --enable-modules=most      \
        --enable-mpms-shared=all   
    
    # make 
    # make install

```
apache的启动：/usr/local/apache/bin/apachectl start(stop,restart)
制作systemctl的启动脚本，网上找一找就行。
一定要记得iptables -F 和setenforce 0.不然别人访问不了你的。
```

## 安装mysql

mysql直接使用的yum安装，编译太慢烦了。用docker更加简单。

一般网上给出的资料都是

```
#yum install mysql
#yum install mysql-server
#yum install mysql-devel
```

安装mysql和mysql-devel都成功，但是安装mysql-server失败，如下：



```
[root@yl-web yl]# yum install mysql-server
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.sina.cn
 * extras: mirrors.sina.cn
 * updates: mirrors.sina.cn
No package mysql-server available.
Error: Nothing to do
```



查资料发现是CentOS 7 版本将MySQL数据库软件从默认的程序列表中移除，用mariadb代替了。

有两种解决办法：

1、方法一：安装mariadb

MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可。开发这个分支的原因之一是：甲骨文公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险。MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。

安装mariadb，大小59 M。

```
[root@yl-web yl]# yum install mariadb-server mariadb 
```

mariadb数据库的相关命令是：

systemctl start mariadb  #启动MariaDB

systemctl stop mariadb  #停止MariaDB

systemctl restart mariadb  #重启MariaDB

systemctl enable mariadb  #设置开机启动

所以先启动数据库

```
[root@yl-web yl]# systemctl start mariadb
```

然后就可以正常使用mysql了



```
[root@yl-web yl]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 5.5.41-MariaDB MariaDB Server

Copyright (c) 2000, 2014, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> 
```



安装mariadb后显示的也是 MariaDB [(none)]> ，可能看起来有点不习惯。下面是第二种方法。

## 2、方法二：官网下载安装mysql-server

```
# wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
# rpm -ivh mysql-community-release-el7-5.noarch.rpm
# yum install mysql-community-server
```

安装成功后重启mysql服务。

```
# service mysqld restart
```

有时候直接启动会报错记得要关闭selinux    ```setenforce 0```

如果想变动mysql的数据库位置可以修改/etc/my.cnf文件

碰到数据库起不来的情况一般都是文件权限的问题数据库要用到的文件和文件夹一律```chown mysql:mysql filename ```





初次安装mysql，root账户没有密码。



```
[root@yl-web yl]# mysql -u root 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.01 sec)

mysql> 
```

设置密码

```
mysql> set password for 'root'@'localhost' =password('password');
Query OK, 0 rows affected (0.00 sec)

mysql> 
```

不需要重启数据库即可生效。

把在所有数据库的所有表的所有权限赋值给位于所有IP地址的root用户。

`mysql> grant all privileges on *.* to root@'%'identified by 'password';`

如果是新用户而不是root，则要先新建用户

`mysql>create user 'username'@'%' identified by 'password';`

此时就可以进行远程连接了。

## 安装php7.0

php源码安装比较好，因为这个解释器。编译效率好。

首先上个代码，来个直观印象

```
./configure --prefix=/usr/local/php7 \
 --with-curl \
 --with-freetype-dir \
 --with-gd \
 --with-gettext \
 --with-iconv-dir \
 --with-kerberos \
 --with-libdir=lib64 \
 --with-libxml-dir \
 --enable-fpm
 --with-openssl \
 --with-pcre-regex \
 --with-pdo-sqlite \
 --with-pear \
 --with-png-dir \
 --with-xmlrpc \
 --with-xsl \
 --with-zlib \
 --with-apxs2=/usr/local/apache/bin/apxs \
 --enable-fpm \
 --enable-bcmath \
 --enable-libxml \
 --enable-inline-optimization \
 --enable-gd-native-ttf \
 --enable-mbregex \
 --enable-mbstring \
 --enable-opcache \
 --enable-pcntl \
 --enable-shmop \
 --enable-soap \
 --enable-sockets \
 --enable-sysvsem \
 --enable-xml \
 --enable-zip \
 --enable-mysqlnd \
 --with-mysqli=mysqlnd \
 --with-pdo-mysql=mysqlnd
 --with-mysql-sock=/var/lib/mysql/mysql.sock
```

php下载www.php.net。下载7.0的版本就行了，然后在安装编译需要的程序。

```
yum -y install libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel MySQL pcre-develcurl-devel libxslt-devel
```

下载后解压进入执行上面的代码，就能够编译执行.make完执行make test检测一下，没有问题就make install。

安装完成，复制php配置文件到安装目录

```cp php.ini-development /usr/local/php/lib/php.ini```



配置http.conf

```
配置http.conf
//加载解析php模块
LoadModule php7_module modules/libphp7.so

//解析后缀名.php的文件
<FilesMatch "\.ph(p[2-6]?|tml)$">
    SetHandler application/x-httpd-php
</FilesMatch>

//默认页面
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```





