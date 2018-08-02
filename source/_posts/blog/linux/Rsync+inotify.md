---
author: QwerSe
head: 
date: 2016-09-13
title:  rsync+inotify(触发式同步)
tags: linux

category: GitBlog
status: publish
summary: rsync可以实现触发式的文件同步，但是通过crontab守护进程方式进行触发，同步的数据和实际数据会有差异。
而inotify可以监控文件系统的各种变化，当文件有任何变动时，就触发rsync同步，这样刚好解决了同步数据的实时性问题。
---





### 



#### 一、rsync+cron的优点与不足
 
与传统的cp、s 需求，例如定期的备份文件服务器数据到远端服务器，对本地磁盘定期做数据镜像等。 随着应用系统规模的不断扩大，对数据的安全性和可靠性也提出的更好的要求，rsync在高端业务系统中也逐渐暴露出了很多不足，首先，rsync同步数据时，需要扫描所有文件后进行比对，进行增量传输。如果文件数量达到了百万甚至千万量级，扫描所有文件将是非常耗时的。而且正在发生变化的往往是其中很少的一部分，这是非常低效的方式。其次，rsync不能实时的去监测、同步数据，虽然它可以通过linux守护进程的方式进行触发同步，但是两次触发动作一定会有时间差，这样就导致了服务端和客户端数据可能出现不一致，无法在应用故障时完全的恢复数据
    
#### 二、inotify介绍
     Inotify 是一种强大的、细粒度的、异步的文件系统事件监控机制，linux内核从2.6.13起，加入了Inotify支持，通过Inotify可以监控文件系统中添
     加、删除，修改、移动等各种细微事件，利用这个内核接口，第三方软件就可以监控文件系统下文件的各种变化情况，而inotify-tools就是这样的一
     个第三方软件。
    	rsync可以实现触发式的文件同步，但是通过crontab守护进程方式进行触发，同步的数据和实际数据会有差异，而inotify可以监控文件系统的各种
    变化，当文件有任何变动时，就触发rsync同步，这样刚好解决了同步数据的实时性问题。
    
#### 三、 安装inotify工具inotify-tools
##### 1. 确认内核支持

    [root@uplook ~]# ll /proc/sys/fs/inotify
    总计 0
    -rw-r--r-- 1 root root 0 10-01 08:07 max_queued_events
    -rw-r--r-- 1 root root 0 10-01 08:07 max_user_instances
    -rw-r--r-- 1 root root 0 10-01 08:07 max_user_watches
    
##### 2 安装rsync与inotify-tools

    http://inotify-tools.sourceforge.net
    inotify-tools是用来监控文件系统变化的工具，因此必须安装在内容发布节点（数据源）
    [root@uplook ~]# tar xf inotify-tools-3.14.tar.gz 
    [root@uplook ~]# cd inotify-tools-3.14
    [root@uplook inotify-tools-3.14]# ./configure && make && make install

inotify-tools指令:
  
   inotifywait		用于等待文件或文件集上的一个特定事件，它可以监控任何文件和目录
   inotifywatch		用于收集被监控的文件系统统计数据，包括每个inotify事件发生多少次等信息
    

####四、 inotifywait相关参数
Inotifywait是一个监控等待事件，可以配合shell脚本使用，常用的一些参数：

    -m， 即--monitor，表示始终保持事件监听状态
    -r，  	即--recursive，表示递归查询目录
    -q， 	即--quiet，表示打印出监控事件
    -e， 	即--event，通过此参数可以指定要监控的事件，常见的事件有modify、delete、create、attrib等
    




    [root@uplook ~]# inotifywait -mrq -e modify,delete,create,attrib /var/www/html/
    
    [root@uplook ~]# inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f' -e modify,delete,create,attrib /var/www/html/
    04/06/14 14:23 /var/www/html/file1
    04/06/14 14:23 /var/www/html/file1

    [root@uplook ~]# nohup inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f' -e modify,delete,create,attrib /var/www/html >> file_modify.txt　&
    
    [root@uplook ~]# inotifywait -mrq --format '%w%f' -e modify,delete,create,attrib /var/www/html/
    /var/www/html/file2
    /var/www/html/file2



#### 五、配置示例
拓扑图
			            192.168.5.11		发布服务器(数据源)
						/var/www/html


	web2 192.168.5.16   web3 192.168.5.2
	/var/www/html			 /var/www/html

发部服务器
 hosts

    [root@source ~]# cat /etc/hosts
    127.0.0.1   		localhost
    192.168.5.11   source
    192.168.5.16   web2
    192.168.5.2     web3
        
配置ssh公钥认证

    [root@source ~]# ssh-keygen
    [root@source ~]# ssh-copy-id -i web2
    [root@source ~]# ssh-copy-id -i web3
    [root@source ~]# ssh web2 date;ssh web3 date
    2015年 09月 29日 星期日 11:31:19 CST
    2015年 09月 29日 星期日 11:33:07 CST
    
 安装inotify-tools
[root@source ~]# yum -y install rsync httpd
[root@source ~]# tar xf inotify-tools-3.14.tar.gz 
[root@source ~]# cd inotify-tools-3.14
[root@source inotify-tools-3.14]# ./configure && make && make install

编写脚本
  [root@source ~]# cat /root/rsync_inotify.sh
           
    #!/bin/bash																									
    web2=192.168.5.16																						
    web3=192.168.5.2        																				
    src=/var/www/html/																						
    dst=/var/www/html/																						
    																													
    inotifywait -mrq  --format '%w%f' -e modify,delete,create,attrib $src \				
    　　| while read file																						
    do																													
    	    																						                    
                    rsync -avzHXA --delete $src root@$web2:$dst &>/dev/null	        
                    rsync -avzHXA --delete $src root@$web3:$dst &>/dev/null    	    
         		        																						        
    																													
    done																												
    
    [root@source ~]# chmod a+x /root/rsync_inotify.sh
    [root@source ~]# nohup /root/rsync_inotify.sh &
    
    
    web2, web3
    [root@web2 ~]# yum -y install rsync httpd
    [root@web3 ~]# yum -y install rsync httpd


同步测试



















