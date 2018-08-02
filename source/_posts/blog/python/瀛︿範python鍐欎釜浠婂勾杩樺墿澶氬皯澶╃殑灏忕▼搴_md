---
author: QwerSe
head:
date: 2018-02-10
title: 学习python写个今年还剩多少天的小程序
tags: python



category: coding
status: publish
summary: 今年还剩多少天
---



```
import time

def PoR():
	'''分出是闰年还是平年'''
	if (nian % 4) == 0:
		return 366
	else:
		return 365

def sum(yue):
	'''计算出到底是还剩多少天'''
	sumNian = PoR()
	a = 0
	
	if sumNian == 365:
		'''计算平年'''

		for i in ping:
			a = a+i

		pgo = ping[:yue]

		p = 0

		for i in pgo:
			p = p + i

		p = a - p 


		return  p - ri

		

	else:
		'''计算闰年'''
		b = 0

		for i in run:
			b = b + i

		o = 0


		rgo = run[:yue]

		for i in rgo:
			o = o + i

		o = b - o
		return o - ri
		

ping = [0,31,28,31,30,31,30,31,31,30,31,30,31]
run = [0,31,29,31,30,31,30,31,31,30,31,30,31]

newtime = time.strftime('%Y%m%d')
print ('----')
nian =int(newtime[:4])
yue =int(newtime[4:6])
ri =int(newtime[6:8])


bingo = sum(yue)

print ('今年还剩%s天'%bingo)

```

结果

```
----
今年还剩324天
```


