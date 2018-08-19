---
author: QwerSe
head:
date: 2018-04-16
title: 爬神漫画py程序
tags: python
category: coding
status: publish
summary: 很久没爬了，用这学习一下，捡一下知识。
---
###流程图

有一点不好就是，不能直接转换成png或jpg。还得继续学习。。

![流程图](http://ww1.sinaimg.cn/large/ab318c02gy1fqe7j1qk4qj20hg0glq3l.jpg)

```
import requests 
from  bs4 import BeautifulSoup as shop
import re, time
import os
import pyvips

from selenium import webdriver #模拟浏览器，用于解析js

tong =[1]
driver = webdriver.PhantomJS(executable_path='C:\\pjs\\bin\\phantomjs.exe')
tou_url = 'http://www.shenmanhua.com'
head ={ "Accept":"text/html,application/xhtml+xml,application/xml;",
            "Accept-Encoding":"gzip",
            "Accept-Language":"zh-CN,zh;q=0.8",
            "Referer":"http://www.www.baidu.com/",
            "User-Agent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36"
            }
def get_html(url):
	'''网页下载器'''
    
    wangye = requests.get(url,headers=head)
    return wangye.text
def js(url):
     '''解析js'''
        
    driver.get(url)
    
    return driver.page_source
def one():
	'''解析所有漫画页面'''
    for i in range(1,275):
        head_url = 'http://www.shenmanhua.com/all_p{}.html'.format(str(i))
        head_wangye = get_html(head_url)
        yield head_wangye
def two(url):
	得到每部漫画的简介页
    wangye = get_html(url)
    c = shop(wangye,'lxml')
    d = c.title.string
    dirname = d.split(' ')[0]
    #建立文件夹
    File_Path = os.getcwd()+'\\'+dirname
    if not os.path.exists(File_Path):
        
        os.makedirs(File_Path)
    tong[0] = File_Path
    
    f = c.select('ul > li > a[title]')
    html_3 = three(f)
    sum_1 = 1
    
    for i in html_3:
        kong = []
  
        
        txt_html = get_html(i)
        if sum_1 == 1:
            
            js_html = js(i)
            txt_html = js_html
            sum_1 = 0        
        
        
        #h = i.get('title') #章节名
        kong.append(txt_html)
        
        #kong.append(h)
           
        yield kong
        
def three(f):
	'''得到每个章节的网址'''
    
    for i in f:
        g = i.get('href') #章节后缀0
        uurl = tou_url + g
        yield uurl
            

def mh_info(x):
    #得到mh_info对象
    sum_fire = 1
    fire = []
    for ii in x:
        
        zz = re.compile('mh_info=\{(.*?)\};',re.S)
        to = re.compile('totalimg:(\d+)',re.S)
        im = re.compile('imgpath:"(.*?)"',re.S)
        pa = re.compile('pagename:"(.*?)"',re.S)
        
        mh_p = (zz.search(ii[0]).group(1))
        totalimg = (to.search(mh_p).group(1))
        imgpath = (im.search(mh_p).group(1))
        pagename = (pa.search(mh_p).group(1))
        
        
        
        fire.append(totalimg)
        fire.append(pagename)
        if sum_fire == 1:
            fire.append(imgpath)
        
            
        
        yield fire
        fire = []
        sum_fire += 1              
            
                
                


           
def zhizhao_url(fire):
	'''依靠得到的mh_info算出其他章节。'''
    
    
    for i in fire :
        if len(i) == 3:
            z = 'http://mhpic.mh51.com/comic/{}'.format(i[2])
        filepath = tong[0] +'\\' + i[1]
        if not os.path.exists(filepath):
            
            os.makedirs(filepath)        
        for x in range(1,int(i[0])+1):
            url = z + '{}.jpg-smh.middle.webp'.format(str(x))
            
            
            
            
            '''下载'''
            html = requests.get(url)
            with open(filepath+'\\'+str(x)+'.webp','wb') as file:
                file.write(html.content)
                file.close()
            webp = filepath+'\\'+str(x)+'.webp'
            jpg = filepath+'\\'+str(x)+'.png'
            
            time.sleep(1)
            commandline = 'dwebp "%s" -o "%s"' % (webp, jpg)
            print (commandline)
            os.system(commandline)
                
                            
            print ('ok')

def main():
    wangye = one()
    for i in wangye:
        a = shop(i,'lxml')
        b =a.select('.sdiv')
        for i in b:
            
            houcuo = i.get('href')
            zhangjie_url = tou_url + houcuo
            x = two(zhangjie_url)
            f= mh_info(x)
            
            zhizhao_url(f)
            
main()

```
    
    
