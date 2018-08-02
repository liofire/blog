---
author: QwerSe
head: 
date: 2016-09-29
title:  MySql主从备份
tags: linux

category: mysql
status: publish
summary: MySQL支持单向、异步复制，复制过程中一个服务器充当主服务器，而一个或多个其它服务器充当从服务器。MySQL复制基于主服务器在二进制日志中跟踪所有对数据库的更改(更新、删除等等)
---

### 一.MySQL复制概述
MySQL支持单向、异步复制，复制过程中一个服务器充当主服务器，而一个或多个其它服务器充当从服务器。MySQL复制基于主服务器在二进制日志中跟踪所有对数据库的更改(更新、删除等等)。因此，要进行复制，必须在主服务器上启用二进制日志。每个从服务器从主服务器接收主服务器已经记录到其二进制日志的保存的更新。当一个从服务器连接主服务器时，它通知主服务器从服务器在日志中读取的最后一次成功更新的位置。从服务器接收从那时起发生的任何更新，并在本机上执行相同的更新。然后封锁并等待主服务器通知新的更新。从服务器执行备份不会干扰主服务器，在备份过程中主服务器可以继续处理更新。
 
### 二.复制实现细节
   MySQL使用3个线程来执行复制功能(其中1个在主服务器上，另两个在从服务器上）。当发出START SLAVE时，从服务器创建一个I/O线程，以连接主服务器并让它发送记录在其二进制日志中的语句。主服务器创建一个线程将二进制日志中的内容发送到从服务器。该线程可以即为主服务器上SHOW PROCESSLIST的输出中的Binlog Dump线程。从服务器I/O线程读取主服务器Binlog Dump线程发送的内容并将该数据拷贝到从服务器数据目录中的本地文件中，即中继日志。第3个线程是SQL线程，由从服务器创建，用于读取中继日志并执行日志中包含的更新。在从服务器上，读取和执行更新语句被分成两个独立的任务。当从服务器启动时，其I/O线程可以很快地从主服务器索取所有二进制日志内容，即使SQL线程执行更新的远远滞后。
  
####1.复制线程状态
通过show slave status/G和show master status可以查看复制线程状态。常见的线程状态有：
#####（1）主服务器Binlog Dump线程
·         
        Has sent all binlog to slave; waiting for binlog to be updated
线程已经从二进制日志读取所有主要的更新并已经发送到了从服务器。线程现在正空闲，等待由主服务器上新的更新导致的出现在二进制日志中的新事件。
 
#####（2）从服务器I/O线程状态
·         Waiting for master to send event
线程已经连接上主服务器，正等待二进制日志事件到达。如果主服务器正空闲，会持续较长的时间。如果等待持续slave_read_timeout秒，则发生超时。此时，线程认为连接被中断并企图重新连接。
#####（3）从服务器SQL线程状态
·         Reading event from the relay log
线程已经从中继日志读取一个事件，可以对事件进行处理了。
·         Has read all relay log; waiting for the slave I/O thread to update it
线程已经处理了中继日志文件中的所有事件，现在正等待I/O线程将新事件写入中继日志。
 
#### 2.复制过程中使用的传递和状态文件
默认情况，中继日志使用host_name-relay-bin.nnnnnn形式的文件名，其中host_name是从服务器主机名，nnnnnn是序列号。中继日志与二进制日志的格式相同，并且可以用mysqlbinlog读取。
从服务器在data目录中另外创建两个小文件。这些状态文件默认名为主master.info和relay-log.info。状态文件保存在硬盘上，从服务器关闭时不会丢失。下次从服务器启动时，读取这些文件以确定它已经从主服务器读取了多少二进制日志，以及处理自己的中继日志的程度。
 
如果要备份从服务器的数据，还应备份这两个小文件以及中继日志文件。它们用来在恢复从服务器的数据后继续进行复制。如果丢失了中继日志但仍然有relay-log.info文件，可以通过检查该文件来确定SQL线程已经执行的主服务器中二进制日志的程度。然后可以用Master_Log_File和Master_LOG_POS选项执行CHANGE MASTER TO来告诉从服务器重新从该点读取二进制日志。
### 三.mysql建立主从服务器配置方法
####1.确保主从服务器的版本兼容。从服务器至少与主服务器版本相同或更高。
#### 2.确保主服务器上my.cnf文件的[mysqld]部分包括一个log-bin选项。该部分还应有一个server-id=Master_id选项，其中master_id必须为1到232–1之间的一个正整数值。如：

	[mysqld]
	log-bin=mysql-bin
	server-id=1
####3.启动主服务器
注意：主服务器需要指定对哪些数据库记录二进制日志，这通过在启动主服务器时，加上
--binlog-do-db= db_name选项来实现。如果要记录多个数据库，要分别为每个数据库指定该选项。目前主服务器上的启动脚本请使用/data/mysql/bin/startmysql。另外，主机开机后自动启动mysqld的脚本也已经修改。
#### 4.在主服务器上为从服务器建立一个连接帐户，该帐户必须授予REPLICAITON SLAVE权限。

	mysql> GRANT REPLICATION SLAVE ON *.*
    -> TO '帐号'@'从服务器IP' IDENTIFIED BY '密码';
#### 5.备份数据库。

	shell> mysqldump --master-data  --opt database_name > backup-file.sql
#### 6.查看主服务器的状态

	mysql> show master status/G;
记录File 和 Position 项的值
注：由于没有锁定主服务器，这里记录的主服务器二进制日志position值可能会大于做mysqldump时的值，这将导致从服务器丢失在此期间的更新。如果可以保证在此期间主服务器不会出现创建新表的更新，那么丢失的影响不大；否则，将导致从服务器复制线程失败，这时必须在做mysqldump时锁定主服务器。
#### 7.在从服务器的my.cnf文件中添加下面的行：
        
		[mysqld]
        server-id=slave_id
slave_id值必须为2到232–1之间的一个正整数值。ID值唯一的标识了复制群集中的主从服务器，因此它们必须各不相同。
#### 8.使用--skip-slave-start选项启动从服务器启动从服务器，并导入备份数据库文件。

	shell> mysqladmin create target_db_name
	shell> mysql target_db_name < backup-file.sql
####9.在从服务器上执行下面的语句，以系统的实际值替换选项值：
        mysql> CHANGE MASTER TO
            ->     MASTER_HOST='master_host_name',
            ->     MASTER_USER='replication_user_name',
            ->     MASTER_PASSWORD='replication_password',
            ->     MASTER_LOG_FILE='recorded_log_file_name',
            ->     MASTER_LOG_POS=recorded_log_position;
其中recorded_log_file_name和recorded_log_position分别为步骤6所记录的File和Position值。
#### 10.启动从服务器线程

    mysql> START SLAVE；
#### 11.执行上述程序后，从服务器应连接主服务器，并补充自从快照以来发生的任何更新。如果没有正确更新，请检查复制线程状态以及data目录下的.err文件获取信息。
 
### 四.主从复制如何提高可靠性
目前采用的主从单向复制，从服务器只是实时的保存了主服务器的一个副本。当主服务器发生故障时，可以切换到从服务器继续做查询，但不能更新。
如果采用双向复制，即两台mysql服务器即作为主服务器，又作为从服务器。那么两者都可以执行更新操作并能实现负载均衡，当一方出现故障时，另一方不受影响。但是，除非能保证任何更新操作顺序都是安全的，否则双向复制会导致失败。
为了更好的提高可靠性和可用性，需要当主服务器不可用时，能够在从服务器上更新，当主服务器恢复后，能够将从服务器上的更新同步到主服务器上。目前设想的方法是当主服务器不可用时，令从服务器成为Master。原来的主服务器重新启动后，设定为Slave，并从新的Master上同步更新。待同步一致后，再重新各自切换为原来的角色。具体的操作方法有待实验验证。
 
PS:主服务器产生的二进制日志会占据大量的磁盘空间，应定期删除过期的bin-log.目前通过crontab每天运行/data/mysql/data/log_auto_del.sh来删除3天前的二进制日志文件。
博客来自：http://blog.csdn.net/libraworm/article/details/1703365