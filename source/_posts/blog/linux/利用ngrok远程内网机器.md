---
author: QwerSe
head: 
date: 2017-10-29
title:  利用ngrok远程内网机器
tags: linux

category: linux 
status: publish
summary: linux没有windows下有andesk和一些好用的远程桌面工具，只有ngrok这个比较好的工具
---





需要用ngrok把公司内网的服务器穿出来，好22连接管理。
linux没有windows下有andesk和一些好用的远程桌面工具，只有ngrok这个比较好的工具

####1，安装好go环境，ngrok是用go编写的。



1、下载go的软件包

```
wget http://qwerse.hk.ufileos.com/go1.4.2.linux-amd64.tar.gz
```

2、解压出来可以随便指定位置

```
tar -zxvf go1.4.2.linux-386.tar.gzmv go /usr/local/
```

3、go的命令需要做软连接到/usr/bin

```
ln -s /usr/local/go/bin/* /usr/bin/
```

#### 2，安装git安装git

1、安装git，我安装的是2.6版本，防止会出现另一个错误，安装git所需要的依赖包

```
yum -y install zlib-devel openssl-devel perl hg cpio expat-devel gettext-devel curl curl-devel perl-ExtUtils-MakeMaker hg wget gcc gcc-c++
```

2、下载git

```
文本模式复制代码
```

```
wget https://www.kernel.org/pub/software/scm/git/git-2.6.0.tar.gz
```

3、解压git

```
tar zxvf git-2.6.0.tar.gz
```

4、编译git

```
cd git-2.6.0./configure --prefix=/usr/local/gitmakemake install
```

5、创建git的软连接

```
ln -s /usr/local/git/bin/* /usr/bin/
```



#### 3,下载及编译ngrok

下载ngrok源码

```
cd ~/your_download_dir
git clone https://github.com/inconshreveable/ngrok.git ngrok
cd ngrok
```

生成证书

注意这里有个`NGROK_BASE_DOMAIN`；
假设最终需要提供的地址为`aevit.your-domain.com`，则`NGROK_BASE_DOMAIN`设置为`your-domain.com`；
假设最终需要提供的地址为`aevit.ngrok.your-domain.com`，则`NGROK_BASE_DOMAIN`设置为`ngrok.your-domain.com`；

下面以`ngrok.your-domain.com`为例：

```
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=ngrok.your-domain.com" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=ngrok.your-domain.com" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 
```

执行完以上命令，就会在`ngrok`目录下新生成6个文件

```
device.crt
device.csr
device.key
rootCA.key
rootCA.pem
rootCA.srl
```

`ngrok`通过`bindata`将ngrok源码目录下的`assets`目录（资源文件）打包到可执行文件(`ngrokd`和`ngrok`)中 去，`assets/client/tls` 和 `assets/server/tls` 下分别存放着用于`ngrok`和`ngrokd`的默认证书文件，我们需要将它们**替换**成我们自己生成的：(因此这一步**务必**放在编译可执行文件之前)

```
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp device.crt assets/server/tls/snakeoil.crt
cp device.key assets/server/tls/snakeoil.key
```

编译linux端版本

```
make clean

# 如果是32位系统，这里 GOARCH=386
GOOS=linux GOARCH=amd64 make release-server release-client
```

最后成功的话，会在当前目录生成一个`bin`文件夹，里面包含了`ngrokd`和`ngrok`文件；
其中，`bin/ngrokd`文件是服务端程序；
`bin/ngrok`文件是客户端程序（注意上面指定了`GOOS`为64位linux的，所以这个文件是不能在`mac`或`win`等其他平台跑的，下面将进行说明如何交叉编译）



#####服务端启动

```
/usr/local/ngrok/bin/ngrokd -domain="$NGROK_DOMAIN" -httpAddr=":80"
```

#####客户端使用

在`ngrok`程序的同级目录下，编写配置文件

```
vim ngrok.cfg
```

内容如下:

```
server_addr: "ngrok.your-domain.com:4443"
trust_host_root_certs: false
tunnels:
  example:
   subdomain: "example" #定义服务器分配域名前缀
   proto:
    http: 80 #映射端口，不加ip默认本机
    https: 80
  web:
   subdomain: "web" #定义服务器分配域名前缀
   proto:
    http: 192.168.1.100:80 #映射端口，可以通过加ip为内网任意一台映射
    https: 192.168.1.100:80
 
  ssh:
   remote_port: 50001 #服务器分配tcp转发端口，如果不填写此项则由服务器分配
   proto:
    tcp: 22 #映射本地的22端口
```

使用ngrok
```
./ngrok -config=./ngrok.cfg -subdomain=blog 80  #映射本机的80端口→blog.ngrok.your-domain.com
./ngrok -config ngrok.cfg start ssh  #开启ssh的隧道
```

