---
author: QwerSe
head: 
date: 2016-09-25
title:  mysql基础操作
tags: linux

category: GitBlog
status: publish
summary: SQL语言主要用于存取数据、查询数据、更新数据和管理关系数据库系统，SQL语言由IBM开发。
---



### 一、初识SQL语言
SQL（Structured Query Language 即结构化查询语言）
SQL语言主要用于存取数据、查询数据、更新数据和管理关系数据库系统，SQL语言由IBM开发。SQL语言分为3种类型：
DDL语句	  数据库定义语言： 	数据库、表、视图、索引、存储过程，例如CREATE DROP ALTER
DML语句	  数据库操纵语言： 	插入数据INSERT、删除数据DELETE、更新数据UPDATE、查询数据SELECT
DCL语句	  数据库控制语言： 	例如控制用户的访问权限GRANT、REVOKE

### 二、系统数据库

	information_schema：	虚拟库，主要存储了系统中的一些数据库对象的信息，例如用户表信息、列信息、权限信息、字符信息等
	performance_schema：	主要存储数据库服务器的性能参数
	mysql：						授权库，主要存储系统用户的权限信息
	test：							MySQL数据库系统自动创建的测试数据库

创建需要的业务主库...

### 三、忘记MySQL密码
 vim /etc/my.cnf						
    
     [mysqld]
     skip-grant-table
root#service mysqld restart
root#mysql
	
	mysql> select user,password,host from mysql.user;
	+------+-------------------------------------------+-----------+
	| user | password                                  | host      |
	+------+-------------------------------------------+-----------+
	| root | *F861720E101148897B0F5239DB926E756B1C28B3 | localhost |
	| root |                                           | sxl.com   |
	| root |                                           | 127.0.0.1 |
	| root |                                           | ::1       |
	|      |                                           | localhost |
	|      |                                           | sxl.com   |
	+------+-------------------------------------------+-----------+
	6 rows in set (0.00 sec)

	mysql> update mysql.user set password=password("456") where user="root" and host="localhost";
	mysql> flush privileges;
	mysql> \q

root#vim /etc/my.cnf						 
   
	   [mysqld]
	   #skip-grant-table
root#service mysqld restart


#### 创建业务数据库
语法

	CREATE DATABASE 数据库名;
#### 数据库命名规则：

	区分大小写
	唯一性
	不能使用关键字如 create select
	不能单独使用数字

#### 查看数据库

	SHOW DATABASES;

#### 选择数据库

	SELECT database();返回当前数据库的名字
	USE 数据库名

#### 删除数据库

	DROP DATABASE 数据库名;


#### 创建表(字段名称 类型 值)

	mysql> show tables; --查看当前所有表
	mysql> desc emp; --查看表属性
	mysql>insert into emp value(1,'haha',5000,'oracle'); --表内添加数据
	mysql> insert into emp values (2,'up02',6000,'linux');
	mysql> select * from emp; --查看表内数据
	[root@localhost python]# mysql -uroot -puplooking
	mysql>  create table dept (deptno int,dname varchar(20),locate varchar(20))engine=innodb;
	mysql> desc dept;
	+--------+-------------+------+-----+---------+-------+
	| Field  | Type        | Null | Key | Default | Extra |
	+--------+-------------+------+-----+---------+-------+
	| deptno | int(11)     | YES  |     | NULL    |       |
	| dname  | varchar(20) | YES  |     | NULL    |       |
	| locate | varchar(20) | YES  |     | NULL    |       |
	+--------+-------------+------+-----+---------+-------+
	3 rows in set (0.00 sec)

	--指定引擎，该引擎支持事务
#### 插入数据库 


	mysql> insert into dept values(1,'oracle','1F') ;
	  
	mysql> select * from dept;
	+--------+--------+--------+
	| deptno | dname  | locate |
	+--------+--------+--------+
	|      1 | oracle | 1F     |
	+--------+--------+--------+
	1 row in set (0.00 sec)


	/usr/local/mysql/bin/mysqld  --verbose --help | less
	   
	Default options are read from the following files in the given order:
	/etc/mysql/my.cnf /etc/my.cnf ~/.my.cnf 


