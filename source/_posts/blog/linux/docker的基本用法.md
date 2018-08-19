---
author: QwerSe
head: 
date: 2016-12-8
title:  docker的基本用法
tags: linux

category: linux 
status: publish
summary: docker这么火，肯定要有所了解的，不然就out了！
---
就如同mysql改查一样只需要了解就行，以后需要在深入，就行了。

构建镜像

	docker search nginx \\以nginx的docker镜像为例
	docker pull   nginx \\获取镜像
	docker images       \\查看本机有哪些docker镜像
在亲自获取过程中由于国情，被 q 了，只有到国内去下载了

	1，阿里云docker加速器
	2，时速云docker市场
都能获取到镜像。



#### 1，从一个镜像，启动一个容器
	
	docker run -it --name one-nginx nginx /bin/bash
	\\run 运行的意思，-it 开启一个伪终端，不关闭，--name 起一个自定义名字,/bin/bash 启动的程序
		docker run -idt --name one-nginx nginx /bin/bash
		\\-d 把程序后端运行
#### 2，查看容器运行情况

	docker ps  \\查看进程
	docker ps -a \\查看所有进程
	docker ps -l \\查看最后一个进程
#### 3，删除容器
	
	docker rm 容器id
	
#### 4，删除镜像
	
	docker rmi 镜像id
 
