---
author: QwerSe
head: 
date: 2017-01-03
title:  fedora安装和必备软件
tags: linux

category: web
status: publish
summary: 学习linux一直用的是centos，用其他的linux版本很不熟悉...
---

学习linux一直用的是centos，用其他的linux版本很不熟悉，一开始用的是manjaro，pacman管理不是很熟悉
之后用了deepin，没想到安装一半出错了，我都不知道是怎么回事，把错误反馈了社区，之后决定使用fedora，更centos差不多
我就用着了，不换了。
　　当然，刚开始有很多东西需要配置，才能更好的使用。
### 1必须要配置好软件源，才能方便行事
	
	
	#### 这是个网页，创建一个repo，把网页里面的内容复制进去
		https://github.com/FZUG/repo/blob/master/repos/FZUG.repo
	#### 这个是个可以安装上面源没有的软件
		rpm -ivh http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-stable.noarch.rpm


###　2开始安装常用软件


#### 看清楚name的行，sudo dnf install $name　就能安装你想要安装的软件了
#####　在安装软件时可能会少依赖，慢慢补就行了。	 
## Name 
```	
sogoupinyin 	 	搜狗拼音输入法
fcitx-rime 		中州韵输入法
fcitx-sunpinyin 	sunpinyin 输入法
fcitx-qt5 		fcitx qt5 输入模块
Multimedia
kwplayer 	 	酷我音乐
doubanfm-qt 	 	豆瓣FM
musicbox 	 	网易云音乐 CLI
deepin-music-player 	 	深度音乐
moonplayer 		Moonplayer播放器
simplescreenrecorder 	 	屏幕录像
obs-studio 	 	屏幕录像/流媒体直播
tragtor 	 	FFmpeg音视频转码
Notes / Dict
wiznote 		为知笔记 stable
wiznote-beta 	 	为知笔记 beta
youdao-dict 	 	有道词典 for linux
Browser / Proxy
opera-developer  	Opera developer
opera-beta 	 	Opera beta
opera-stable 	 	Opera stable
freshplayerplugin 	Firefox PPAPI flash 兼容插件
Downloads
bcloud 	　　　　　 	百度云客户端
xware-desktop 	 	迅雷客户端
pointdownload 		点载, 支持迅雷/BT/ed2k下载
System tools
ccal 	　　　　　	命令行下的农历日历
grub4dos 		grub 引导管理器
screenfetch 		系统信息收集
vim 7.4.764 		编辑器之神，增加剪贴板/lua/py3支持
dkms-mt7601u 		MT7601U USB Wifi 驱动
System libraries
opencc 		 	简繁转换库，修正依赖
sunpinyin 	 	基于 SLM 模型的输入法引擎，更新字典
python-html2text 	fedora 	HTML -> ASCII
python3-xlib 		Python3 X 库
python3-keybinder 	null 	Python3 键绑定库
python-mutagen 		处理音频元数据
python3-cairo 		Python3 cairo 绑定，修复 bug
pygobject3 		Python GObject 封装，修复 bug
deepin-ui 		Deepin UI toolkit
deepin-utils 		Deepin utils 库
deepin-gsettings 	gsettings python 绑定
pyjavascriptcore 	fedora 	pyjs
```


	
