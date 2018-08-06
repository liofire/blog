---
author: QwerSe
head: 
date: 2018-8-6
title: python爬取厨房故事并导出json数据
tags: spider

category: Python
status: publish
summary: 朋友叫我帮他爬取www.chufanggushi.com这个网站的数据要合成json，估计他是想利用这些资源建立个小网站。
---
[TOC]

朋友叫我帮他爬取www.chufanggushi.com这个网站的数据要合成json，估计他是想利用这些资源建立个小网站。

#### 分析网站

看了下，网站

![](http://ww1.sinaimg.cn/large/ab318c02gy1ftztfso1isj20wy0du7a7.jpg)

看到了api，当然是利用api来获取数据了。
把api的内容复制到本地，分析结构

![](http://ww1.sinaimg.cn/large/ab318c02gy1ftztg93bz7j20in0bv3zr.jpg)

"like_count"是热度
"title"是菜谱名等等...
#### 获取数据

分析完了，就是获取数据了，获取5个数据。用代码进行解释

```
import requests
import re
import json
from pymongo import MongoClient
import time


class JSONObject:
    '''使json数据能用属性的方式访问'''
    def __init__(self, d):
        self.__dict__=d

#创建mongodb的连接
client = MongoClient('mongodb://10.10.12.115:27017/')
db = client.pc
data = db.data



count = 0

#开始数据的循环爬取，只爬取前19页
for x in range(1,20):
    
    url = "https://www.chufanggushi.com/api/recipes/?page_size=24&page={}&type=recipe".format(str(x))


    head={ "accept":"application/json, text/plain, */*",
    "accept-language":"zh",
    "cookie":"_omappvp=e4wgfkyiwCHvob2yYtjt8tRPUmhji3bW1DC9McBdzp03f6vShqfTpuHVE4I2ih0V0dpzDfaZR7ZddaTcicvEj9pX6ZrXnGTX; _ga=GA1.2.894983333.1533443961; _gid=GA1.2.1955174767.1533443961; amplitude_id_132c72b21efcc3b4fd03ac4374be22f1chufanggushi.com=eyJkZXZpY2VJZCI6IjI3OWM0N2JjLWE4NzAtNGUzYS05YzA3LWQ5NzVhMGRiOGYzNVIiLCJ1c2VySWQiOm51bGwsIm9wdE91dCI6ZmFsc2UsInNlc3Npb25JZCI6MTUzMzQ0Mzk2MTAyNywibGFzdEV2ZW50VGltZSI6MTUzMzQ0NzcxMDI1MSwiZXZlbnRJZCI6MjcsImlkZW50aWZ5SWQiOjAsInNlcXVlbmNlTnVtYmVyIjoyN30=",
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.119 Safari/537.36',
    'Referer':'http://www.sse.com.cn/assortment/stock/list/share/'
    }
 
    api_txt= requests.get(url, headers=head).text

    api_json = json.loads(api_txt,object_hook=JSONObject)

    for i in range(24):
        '''获取json的数据'''
        cid =  count
        url = api_json.data[i].url #提取网址，下面方式相同
        like_count = api_json.data[i].like_count
        name = api_json.data[i].title
        img = api_json.data[i].cell_image.url
        min = (api_json.data[i].preparation_time + api_json.data[i].baking_time)/60
        
        data_data = {"name":name,"cid":cid,"img":img,"like":like_count,"min":min}
        data.insert(data_data)
        print (count)
        count += 1
        time.sleep(0.5)
```
#### 导出数据
 使用mongodb，导出json
 `./mongoexport -d pc -c data chufang.json`









