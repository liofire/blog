---
author: QwerSe
head:
date: 2018-04-23
title: 搭建jupyter-notebook服务器
tags: 服务
category: python
status: publish
summary: 搭建jupyter-notebook服务器（centos7）同时开机启动
---


### 安装notebook
- 安装pip缺少一些系统文件我嫌麻烦，使用了conda集成包。
官网直接下载下来的是sh文件执行就行，不要把conda加入系统变量会导致yum用不了。

##### 配置notebook

- 生成 /home/ipynb/.jupyter/jupyter_notebook_config.py 文件

`jupyter notebook --generate-config`

- 编辑配置文件

```
#注释为ssl，没什么大用
#c.NotebookApp.certfile = u'/home/ipython/full_chain.pem'
#c.NotebookApp.keyfile = u'/home/ipython/private.key'
c.NotebookApp.ip = '*'                  #任何ip可以连接
c.NotebookApp.open_browser = False      #启动不打开浏览器
#c.NotebookApp.password = 'sha1:26ab20ed1e8a:73a3b3aecc20aa27b9eba88be46dc7d247dc5b46' #密码这里先#掉（第4步会生成密码，然后填入sha1，解注释）
c.NotebookApp.port =6082    #开启的端口
c.NotebookApp.notebook_dir = '/home/ipython/test' #工作目录

```

- 设置密码

localhost:8888 进入notebook 新建一个ipynb
运行
`from notebook.auth import passwd`
`passwd()`

输入两遍密码得到sha1 返回第3步，把密码的#解掉，""中填入sha1


### 插件安装

- 增强jupyter的编辑能力(强烈推荐！)，使jupyter如IDE一般强大
https://github.com/jupyterlab/jupyterlab
我是使用的conda，用conda安装。githup也有其他安装方法，看看文档。
`conda install -c conda-forge jupyterlab`

### 开机自动运行

- 利用systemctl来进行开机启动

```
[Unit]
Description=Jupyter Notebook
After=network.target

[Service]
Type=simple
PIDFile=/home/ipython/run/jupyter.pid
ExecStart=/conda/bin/jupyter-lab
User=ipython
Group=ipython
WorkingDirectory=/home/ipython/test
Environment='XDG_RUNTIME_DIR=/home/ipython/run' #这个每个电脑都不一样，使用jupyter --path 查看RUNTIME的值就知道了
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

感谢：
. http://chnwentao.com/2017/11/%E5%B7%A5%E5%85%B7JupyterNotebook%E4%BD%BF%E7%94%A8%E5%B0%8F%E7%BB%93/
. https://bitmingw.com/2017/07/09/run-jupyter-notebook-server/

