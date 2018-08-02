---
author: QwerSe
head:
date: 2018-03-31
title: 今日头条分析ajax进行post爬取
tags: python



category: coding
status: publish
summary: 小小爬虫
---
###启用了双线程，更加给力了。。。
```
import requests,re
import json
from urllib.parse import urlencode
from requests.exceptions import RequestException
from bs4 import  BeautifulSoup
from multiprocessing import Process

def index_html(sum,keys):

    '''得到索引页'''
    url_keys ={
        'offset':sum,
        'format':'json',
        'keyword':keys,
        'autoload':'true',
        'count':'20',
        'cur_tab':'3',
        'from':'gallery'
    }
    header = {
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:55.0) Gecko/20100101 Firefox/55.0"
    }
    url = 'https://www.toutiao.com/search_content/?' + urlencode(url_keys)
    try:

        wangye_index = requests.get(url,headers=header)
        if wangye_index.status_code == 200:
            return wangye_index.text
        return None
    except RequestException:
        print ('索引页出错')
        return None


def ajax_json(ajax):
    '''从索引页里面解析出，详情页的网址'''
    data = json.loads(ajax)
    if data and 'data' in data.keys():
        for i in data.get('data'):
            yield i.get('article_url')


def info_html(url):
    '''获取详情页图像网址'''
    header = {
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:55.0) Gecko/20100101 Firefox/55.0"
    }
    try:
        wangzhi = requests.get(url,headers=header)
        if wangzhi.status_code == 200:
            return wangzhi.text
        return None
    except RequestException:
        print ('页面出错')
        return None

def image_url(tupian):
    '''获取图片网址'''
    title_test = BeautifulSoup(tupian,'lxml')
    title = title_test.select('title')[0].get_text()
    jiexi = re.compile('gallery:\sJSON.parse\((.*?)\),', re.S)
    jieguo_1 = re.search(jiexi, tupian)

    jieguo = (jieguo_1.group(1))

    a = jieguo.replace('\\', '')

    tt = re.compile('"(http.*?)"', re.S)

    tt_url = re.findall(tt, a)
    cishu = 0
    for i in tt_url:
        cishu += 1
        if cishu % 4 == 0:

            yield [title,i]


def file_xie(t):

    '''把图片写入文件'''


    global sum_c
    sum_c +=1
    a = t[0]
    c = requests.get(t[1])
    with open(a[:6]+str(sum_c)+'.jpg','wb') as f:
        f.write(c.content)
        f.close()
    print ("写入完毕！")









def main(i):
    wangye_ajax = index_html(i,'街拍')
    url = ajax_json(wangye_ajax)
    for i in url:
        tail_html = info_html(i)
        for x in image_url(tail_html):
            file_xie(x)

sum_c = 0
if __name__ == '__main__':
	'''开启多线程'''
	
	p = Process(target=main, args=(0,))
	b = Process(target=main, args=(20,))
	p.start()
	b.start()

```