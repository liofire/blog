---
author: qwerse
head: 
date: 2018-01-16
title: 局域网搭建dns
tags: dns
category: linux
status: publish
summary: 引用了csdn"lvshaorong的博客"，再次表示感谢。pdnsd是一款非常简单配置的局域网dns。
---

### 选择dns软件"pdnsd"

用过dnsmasq和bind。都不太符合自己的意思。尤其是bind简直是太笨重了。

#### pdnsd的优势

Ubuntu16.04默认使用dnsmasq作为dns解析服务，换句话说，一个Ubuntu 16.04LTS的电脑本身就是一个开放53端口的DNS服务器，听起来有点像买Ubuntu电脑送DNS服务器的感觉。但事实确实是这样的，因为Ubuntu默认开机启动dnsmasq服务，而且即使你把它stop了，Ubuntu还会定时重启，而且在启动dnsmasq服务的时候，Ubuntu系统会自动修改/etc/resolv.conf这个默认DNS服务指向文件为127.0.0.1，导致你试图修改这个文件改变你的DNS服务器地址后会莫名其妙被改回127.0.0.1。但是我之所以要用Pdnsd替换点dnsmasq的主要目的还是为了防止DNS污染。

首先Pdnsd有这样几个优势

1、可以将UDP协议的DNS请求转换为TCP协议进行发送，通过这种方式避免DNS污染

2、pdnsd可以选择性的拒绝一些常见的被污染的目标地址，直接通过境内的dns服务器即可获得正确结果

3、pdnsd可以做本地DNS缓存，而且可以自由设置过期时间，对于已缓存的DNS地址，局域网内其他电脑在DNS解析上的耗时可以缩减到1ms以内，大大提升了电脑的上网速度

#### Ubuntu 16.04LTS系统为例详细讲解安装过程

##### 1、  第一步：将dnsmasq的监听端口从53端口挪到其他端口

首先要科普一下，Ubuntu系统中的dnsmasq并不只有一个实例在运行，而是会开启多个，在我的电脑中因为开启了Ubuntu自带的Wifi分享功能，本机多了一个内网IP地址10.42.0.1，并且作为这个Wifi网络的DNS服务器，所以我现在电脑上运行着三个dnsmasq进程。
查看电脑53端口的占用情况，可以用一下命令
`sudo netstat -apn | grep 53`

我的计算机有线网卡的内网地址为192.168.1.100，并且分享了一个Wifi网络，在这个Wifi网络中的内网地址是10.42.0.1在Ubuntu默认情况下，所有网卡的53端口（包括ipv6地址）均会被dnsmasq所占，并且不止一个dnsmasq实例正在运行。



##### 注：之所以有127.0.0.1:53而没有192.168.1.100:53或者0.0.0.0:53是Ubuntu默认不向局域网开放自己的53端口，理由当然是为了安全，防止让你真的变成一台局域网DNS服务器，而有10.42.0.1:53的原因是本计算机分享了一个Wifi，在这个Wifi局域网中，这台电脑既当网关也当DNS服务器，该wifi中的所有电脑被DHCP分配到的DNS服务器均为10.42.0.1,所以对于这个内网，dnsmasq是向局域网开放自己的53端口的。

##### 注：运行在127.0.0.1的:53的dnsmasq进程和运行在10.42.0.1的dnsmasq是两个 不同的进程，使用不同的配置文件

在Ubuntu默认情况下，127.0.0.1和0.0.0.0的53端口一般由同一个dnsmasq进程所占，而其他网段比如10.42.0.1由另一个dnsmasq进程控制，由于10.42.0.1是笔记本共享出的wifi，所以这里的dnsmasq除了做dns转发以外，还有一个更重要的作用就是做DHCP服务器动态为连接热点的设备分配IP地址。
那么如何观察本机一共有多少dnsmasq进程，这些dnsmasq的配置文件又各是谁呢？可以用如下指令

`ps -fC dnsmasq|more`


现在我们要用pdnsd替换掉的就是监听在127.0.0.1:53和192.168.1.81:53和0.0.0.0:53这几个端口的dnsmasq服务，也就是pid为12625的这个dnsmasq，其他的dnsmasq因为并没有挤占端口，而且还要作为DHCP服务使用，所以我们不要停掉他们。那么如何将dnsmasq的监听端口挪走呢，只需要修改dnsmasq的配置文件并重启即可，虽然仅仅stop掉dnsmasq的服务也有用，但是Ubuntu隔一段时间又会自动重启并同时修改/etc/resolv.conf文件，所以最好的方式还是把dnsmasq的端口挪走一劳永逸。
首先我们停止dnsmasq的服务

`sudo service dnsmasq stop`

现在使用netstat -nl观察端口的监听情况是不是发现127.0.0.1:53和192.168.1.81:53和0.0.0.0:53这几个53端口都已经没有啦，而且并没有影响10.42.0.1:53上做DHCP服务的dnsmasq的运行。

##### 注：为什么127.0.0.1:53没有了而10.42.0.1:53还是由dnsmasq所占，原因如上面已经提到过的，

这两个端口的dnsmasq并不是一个dnsmasq进程，所使用配置文件和发挥的功能也不一样，而我们要修改两个网段最终的DNS服务程序，只需要将127.0.0.1:53这个dnsmasq换掉即可，不需要管10.42.0.1:53这个内网的dnsmasq进程
然后修改dnsmasq的配置文件

`sudo nano /etc/dnsmasq.conf`

在第十行将dnsmasq的监听端口从53挪到9053，这样我们就把系统默认的dnsmasq的端口挪到了9053

然后重启dnsmasq服务

`sudo service dnsmasq restart`

现在在使用netstat -nl查看，是不是发现原来127.0.0.1:53和192.168.1.81:53和0.0.0.0:53这几个53端口都变成9053啦，现在我们已经为pdnsd服务腾出了工作端口，下一步我们可以安装pdnsd了

注：于此同时，由于我们系统的dns服务器指向还是127.0.0.1,所以此时你的电脑已经上不了忘了，解决方法是修改/etc/resolv.con文件将127.0.0.1换成ISP提供的DNS服务器或者114.114.114.114或者阿里的223.5.5.5等等。

#### 2、安装Pdnsd服务


`sudo apk-get install pdnsd`

在安装时会跳出一个选择框，我们选manual
（注意如果此时提示你网络问题请到/etc/resolv.conf文件中把dns服务器地址从127.0.1.1改到114.114.114.114）
安装完pdnsd服务之后我们需要修改它的配置文件，主要目的是设置两组DNS服务器，第一组是国内的114.114.114.114,第二组是OpenDNS的5353端口，114.114.114.114的53端口即使使用TCP连接解析也会返回一些脏地址，但我们仍然要用他的原因它的ping通率要比OpenDns要高，主要作用是加速一些国内正常DNS的解析，防止访问国内网站时走OpenDNS导致解析变慢。所以下面配置文件中的114.114.114.114也可以换成你当地ISP分给你的DNS服务器，尤其是在114.114.114.114 ping不通的情况下。另外，下面配置文件中的reject列表也非常重要，只有当114.114.114.114返回脏地址在这个列表中时才会触发pdnsd再发送一个请求到OpenDNS，所以务必要将114.114.114.114所有的脏地址都收录到这个列表中。同时，如果你本地ping不通OpenDNS的两个地址时，有两种方式可以解决，第一种是使用Redsocks+代理将tcp协议的DNS查询请求加密发送到国外代理服务器，另一种是使用能ping通的国外其他DNS服务器，尽量避开53端口，也就是说尽量找国外支持53端口以外进行解析的DNS服务器。

`sudo nano /etc/pdnsd.conf`

将这个文件修改如下

```
// Read the pdnsd.conf(5) manpage for an explanation of the options.  
  
/* Note: this file is overriden by automatic config files when 
  /etc/default/pdnsd AUTO_MODE is set and that 
  /usr/share/pdnsd/pdnsd-$AUTO_MODE.conf exists 
*/  
  
global {  
    perm_cache=4096;  
    cache_dir="/var/cache/pdnsd";  
    run_as="pdnsd";  
    server_ip = 192.168.1.81;  # 将自己的有线网卡的局域网地址设为工作地址，这样可以方便局域网用户把你的电脑当成DNS服务器,这里要换成你的地址，但是好像0.0.0.0我试过不行  
    server_port=53;            # 使用DNS协议默认的53号地址  
    status_ctl = on;  
    query_method=tcp_only;   #非常重要，使用tcp协议请求DNS结果，防止被污染  
    neg_domain_pol = off;    
    paranoid = on;    
    par_queries = 1;    
    min_ttl = 6h;    
    max_ttl = 12h;    
    timeout = 10;    
}  
  
/* with status_ctl=on and resolvconf installed, this will work out from the box 
  this is the recommended setup for mobile machines */  
server {    
   label = "routine";  # 这个随便写    
   ip = 114.114.114.114;   # 这里为上级 dns 的 ip 地址，如果8.8.8.8能够ping通，可以用8.8.8.8    
   timeout = 5;    
   reject = 74.125.127.102,  #这里非常重要，这些ip是防火墙喜欢将DNS污染到的目标地址，也就是脏地址，你可以使用nslookup命令查询当地的脏地址并补充在这个列表中，如果114.114.114.114返回脏地址后pdnsd会自动重新发送DNS请求到下面的OpenDNS服务器，如果一个脏地址没有在列在这里，那么还是会收到污染的结果  
       74.125.155.102,    
       74.125.39.102,    
       74.125.39.113,    
       209.85.229.138,    
       128.121.126.139,    
       159.106.121.75,    
       169.132.13.103,    
       192.67.198.6,    
       202.106.1.2,    
       202.181.7.85,    
       203.161.230.171,    
       203.98.7.65,    
       207.12.88.98,    
       208.56.31.43,    
       209.145.54.50,    
       209.220.30.174,    
       209.36.73.33,    
       211.94.66.147,    
       213.169.251.35,    
       216.221.188.182,    
       216.234.179.13,    
       243.185.187.39,    
       37.61.54.158,    
       4.36.66.178,    
       46.82.174.68,    
       59.24.3.173,    
       64.33.88.161,    
       64.33.99.47,    
       64.66.163.251,    
       65.104.202.252,    
       65.160.219.113,    
       66.45.252.237,    
       69.55.52.253,    
       72.14.205.104,    
       72.14.205.99,    
       78.16.49.15,    
       8.7.198.45,    
       93.46.8.89,    
       37.61.54.158,    
       243.185.187.39,    
       190.93.247.4,    
       190.93.246.4,    
       190.93.245.4,    
       190.93.244.4,    
       65.49.2.178,    
       189.163.17.5,    
       23.89.5.60,    
       49.2.123.56,    
       54.76.135.1,    
       77.4.7.92,    
       118.5.49.6,    
       159.24.3.173,    
       188.5.4.96,    
       197.4.4.12,    
       220.250.64.24,    
       243.185.187.30,    
       249.129.46.48,    
       253.157.14.165;    
   reject_policy = fail;    
   exclude = ".google.com",    
       ".cn",      #排除国内DNS解析，如果正常翻，则可以在前面加#注释    
       ".baidu.com",   #排除国内DNS解析，如果正常翻，则可以在前面加#注释    
       ".qq.com",  #排除国内DNS解析，如果正常翻，则可以在前面加#注释    
       ".gstatic.com",    
       ".googleusercontent.com",    
       ".googlepages.com",    
       ".googlevideo.com",    
       ".googlecode.com",    
       ".googleapis.com",    
       ".googlesource.com",    
       ".googledrive.com",    
       ".ggpht.com",    
       ".youtube.com",    
       ".youtu.be",    
       ".ytimg.com",    
       ".twitter.com",    
       ".facebook.com",    
       ".fastly.net",    
       ".akamai.net",    
       ".akamaiedge.net",    
       ".akamaihd.net",    
       ".edgesuite.net",    
       ".edgekey.net";    
}  
  
server {    
   # Better setup dns server(DON'T USE PORT 53) on your own vps for faster proxying    
   label = "special";  # 这个随便写    
   ip = 208.67.222.222,208.67.220.220; # 这里为上级 dns 的 ip 地址    
   port = 5353;    
   proxy_only = on;    
   timeout = 5;    
}    

```

保存后重启pdnsd发现并没有像我们想的一样53端口出现监听，因为我们还需要将pdnsd以守护模式运行

`sudo nano /etc/default/pdnsd`

将START_DAEMON=no改为START_DAEMON=yes

现在可以重启pdnsd服务了

`sudo /etc/init.d/pdnsd restart`

重启之后再使用netstat -nl查看端口情况，是不是53端口已经正在使用了，或者使用sudo netstat -apn | grep 53看看是不是由pdnsd服务占用53端口



最后一步，就是将我们本机的DNS服务器地址指向pdnsd

`sudo nano /etc/resolv.conf`

将原来的127.0.1.1改成上面配置文件配置的地址，即如果是局域网地址，就填192.168.1.81，视你自己的网络情况而定，如果上面填的是0.0.0.0，那么此处就填127.0.0.1

然后再使用nslookup www.baidu.com
和nslookup www.facebook.com
看一下dns解析结果是否正确
如果baidu.com无法解析说明Pdnsd配置有问题，此时要检查53端口是否已被监听
如果baidu.com正常而facebook.com无法解析或者返回错误地址，首先我们可以查看并清空一下缓存

`sudo pdnsd-ctl dump`
`sudo pdnsd-ctl empty-cache`

如果还是返回污染后的地址，就把这个污染地址加到上面/etc/pdnsd.conf的脏域名列表中，就可以获得正确结果了
那么如何判断DNS解析后的域名是否正确呢，方法是nslookup www.twittter.com 和nslookup www.youtube.com，真的解析地址会返回多个解析结果，而污染后的结果只有一个
如下图中前两个是错误的解析结果，后两个是正确的解析结果，或者你还可以通过webdnstools.com这个网站去查询正确的解析结果与本地的结果进行比对


**----局域网设置内网域名-----**

a.最简单办法直接用pdnsd-ctl 插入一条 A 记录,举例：

`pdnsd-ctl add a 127.0.0.1 www.linuxbyte.org. 900 `（900 为TTL 值）

同样你可以用pdnsd-ctl add 插入其他的mx，ns，cname 记录等
b.第一方法适合临时做一些修改，要想真正的劫持一个域名可以在 pdnsd.conf 中加入 rr section。举例：

```
rr {
name = linuxbyte.org;
ns = localhost;
soa = localhost, root.localhost, 42, 86400, 900, 86400, 86400;
}
```
这个是劫持了ns 记录，把域名解析交给你指定的dns 服务器来做。
​
```
rr {
name = *.linuxbyte.org;
cname = www.linuxsky.org;
}
```
这里把所有linuxbyte.org 的二级域名都做了cname到了 www.linuxsky.org

```
rr {
name = www.linuxbyte.org;
a = 192.168.0.123
}```
直接插入一条A 记录。







```