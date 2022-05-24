# Nginx从入门到实践

## Nginx基本概念

### 什么是Nginx

```tex
Nginx ("engine x")是一个高性能的HTTP和反向代理服务器，特点是占有内存少，并发能
力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好
Nginx专为性能优化而开发，性能是其最重要的考量，实现上非常注重效率，能经受高负载
的考验，有报告表明能支持高达50000个并发连接数
```

### 正向代理

​	在客户端（浏览器）配置代理服务器，通过代理服务器进行互联网访问

### 反向代理

​	反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，再返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址。

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Nginx/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86.png)

### 负载均衡

​	由于单个服务器解决不了，所以我们增加服务器的数量，然后将请求分发到各个服务器上,将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上,将负载分发到不同的服务器，也就是我们所说的负载均衡

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Nginx/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1.png)

### 动静分离

​	为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Nginx/%E5%8A%A8%E9%9D%99%E5%88%86%E7%A6%BB.png)

## Nginx安装

操作平台 CentOS7

### 安装相关依赖

安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装

```clike
yum install gcc-c++
```

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库

```clike
yum install -y pcre pcre-devel
```

zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库

```clike
yum install -y zlib zlib-devel
```

OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库

```clike
yum install -y openssl openssl-devel
```

### 安装Nginx

#### 下载

> a. 直接下载`.tar.gz`安装包，地址：https://nginx.org/en/download.html
>
> b. **使用`wget`命令下载（推荐）**。确保系统已经安装了wget，如果没有安装，执行 yum install wget 安装。

```clike
wget -c https://nginx.org/download/nginx-1.19.0.tar.gz
```

#### 解压

```clike
tar -zxvf nginx-1.19.0.tar.gz
```

#### 配置

```clike
./configure
make && make install
```

#### 启动

```clike
ps aux|grep nginx #查看是否开启
 
./nginx -v #查看版本
 
cd /usr/local/nginx/sbin/nginx
./nginx #启动
./nginx -s stop #停止
./nginx -s quit #停止
./nginx -s reload #重载
```

## Nginx配置文件

Nginx 配置文件由三部分组成

```bash
...              #全局块

events {         #events块
   ...
}

http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```

- **第一部分 全局块**
  主要设置一些影响 nginx 服务器整体运行的配置指令。
  比如： worker_processes 1; ， worker_processes 值越大，可以支持的并发处理量就越多。

- **第二部分 events块**
  events 块涉及的指令主要影响Nginx服务器与用户的网络连接。
  比如： worker_connections 1024; ，支持的最大连接数。

- **第三部分 http块**
  http 块又包括 http 全局块和 server 块，是服务器配置中最频繁的部分，包括配置代理、缓存、日志定义等绝大多数功能。

- - **server块**：配置虚拟主机的相关参数。
  - **location块**：配置请求路由，以及各种页面的处理情况。
  
  ![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Nginx/%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E7%BB%93%E6%9E%84.jpg)

示例配置文件

```shell
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}
```

## 安装Tomcat

### ①下载安装

[tomcat官网]([https://tomcat.apache.org](https://tomcat.apache.org/))

```shell
tar -zxvf apache-tomcat-10.0.5.tar.gz 
```

> **bin:** 存放 Tomcat 的 启动、停止 等相关命令
> **lib:** 存放 Tomcat 运行时所需要的 jar 包
> **conf:** Tomcat 配置文件目录
> **logs:** Tomcat 运行日志目录
> **webapps:** 存放运行在 Tomcat 服务器内的应用程序（JavaWeb 应用部署目录）
> **work:** 存放应用程序运行时动态生成的 java 代码和动态编译的 class 文件
> **temp:** 存放 Tomcat 运行时产生的临时文件

### ②相关命令

```shell
./startup.sh #启动
netstat -npl | grep :8080 #查看端口状态（Tomcat 默认监听 8080 端口）
./shutdown.sh #停止
```

注：详细过程可参考[linux下安装tomcat](https://blog.csdn.net/xietansheng/article/details/84405208)

## 实现反向代理

### ①启动多个tomcat，分别监听不同的端口

详细过程可以参考[开启多个tomcat](https://amos-x.com/index.php/amos/archives/centos7-tomcat-run/)

### ②配置nginx

找到nginx配置文件，进行反向代理配置。

```bash
server {
        listen       9001;   
        server_name  0.0.0.0   #监听地址
   
        location  ~ /edu/ {       
           root html;  #/html目录
           proxy_pass http://127.0.0.1:8081;  #请求转向
           index  index.html index.htm;      #设置默认页       
        } 
        location  ~ /vod/ {       
           root html;  #/html目录
           proxy_pass http://127.0.0.1:8081;  #请求转向
           index  index.html index.htm;      #设置默认页       
        } 
    }
```

开放对外访问的端口号9001

重启nginx服务器，使配置文件生效

### ③测试

访问http://127.0.0.1:9001/edu/ 直接跳转到127.0.0.1:8081
访问http://127.0.0.1:9001/vod/ 直接跳转到127.0.0.1:8082

> Nginx-location相关指令
>
> ​	=：用于不含正则表达式的uri前，要求请求字符串与uri严格匹配，如果匹配成功，
> ​	就停止继续向下搜索并立即处理该请求
> ​	~：用于表示uri包含正则表达式，并且区分大小写
> ​	~\*：用于表示uri包含正则表达式，并且不区分大小写
> ​	^~：用于不含正则表达式的uri前，要求Nginx服务器找到标识uri和请求字符串匹配度最高的location后，立即使用此location处理请求，而不再使用location块中的正则uri和请求字符串做匹配
> ​	注意: 如果uri包含正则表达式，则必须要有~或者~*标识。

```shell
# 基本语法
location [ = | ~ | ~* | ^~] uri {}
```

## 实现负载均衡

**1、实现效果**

> (1) 浏览器地址栏输入地址http://192.168.xxx.xxx/edu/index.html, 负载均衡效果，平均到8080
> 和8081端口中，

### 负载分配策略

在linux下有Nginx、LVS、 Haproxy 等等服务可以提供负载均衡服务，而且Nginx提供了以下几种分配方式(策略)

- **1、轮询(默认)**

  每个请求按时间顺序逐一分配到不 同的后端服务器，如果后端服务器down掉，能自动剔除

- **2、weight**
  weight代表权重默认为1,权重越高被分配的客户端越多。
  指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。例如: 。

- **3、ip hash**

  每个请求按访问ip的hash结果分配, 这样每个访客固定访问一个后端服务器,可以解诀session的问题。例如:

```shell
upstream server pool{
  ip_ hash
  server 192.168.5.21:80
  server 192.168.5.22:80
}
```

- **4、fair (第三方)**
  按后端服务器的响应时间来分配请求，响应时间短的优先分配

```shell
upstream server_pool 
	server 192.168.5.21:80;
	server 192.168.5.22:80;
	fair;
}
```

## 实现动静分离

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Nginx/%E5%8A%A8%E9%9D%99%E5%88%86%E7%A6%BB.jpg)



> ​	通过location指定不同的后缀名实现不同的请求转发。通过expires参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。具体Expires定义: 是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。(如果经常更新的文件，不建议使用Expires来缓存)，如果设置3d, 表示在这3天之内访问这个URL, 发送一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304,如果有修改，则直接从服务器重新下载，返回状态码200。

**2、准备工作**

> (1) 在liunx系统中准备静态资源，用于进行访问
>
> /data/image 图片文件夹
>
> /data/www html文件夹

**3、具体配置**

> (1) 在nginx配置文件中进行配置

```clike
location /www/{
	root /data/;
	index index.html index.htm;
}
location /image/{
	root /data/;
	autoindex on;
}
```

4、实际测试

```
http://localhost/www/index.html
http://1ocalhost/image/1.jpg
```

## 实现高可用集群

**1、什么是nginx高可用**

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Nginx/%E9%AB%98%E5%8F%AF%E7%94%A8.jpg)

**2、配置高可用的准备工作**

> (1) 需要两台服务器192.168.17.129 和192.168.17.131
> (2) 在两台服务器安装nginx.
> (3) 在两合服务器安装keepalived.

**3、在两台服务器安装keepalived**
使用yum命令进行安装

```shell
$ yum install keepalived
$ rpm -q -a keepalived    #查看是否已经安装上
```

默认安装路径: /etc/keepalived

安装之后，在etc里面生成目录keepalived, 有配置文件keepalived.conf

**4、完成高可用配置(主从配置)**

（1）修改keepalived的配置文件`keepalived.conf`为：

```shell
global_defs {
	notification_email {
	  acassen@firewall.loc
	  failover@firewall.loc
	  sysadmin@firewall.loc
	}
	notification_email_from Alexandre.Cassen@firewall.loc
	smtp_ server 192.168.17.129
	smtp_connect_timeout 30
	router_id LVS_DEVEL	# LVS_DEVEL这字段在/etc/hosts文件中看；通过它访问到主机
}

vrrp_script chk_http_ port {
	script "/usr/local/src/nginx_check.sh"
	interval 2   # (检测脚本执行的间隔)2s
	weight 2  #权重，如果这个脚本检测为真，服务器权重+2
}

vrrp_instance VI_1 {
	state BACKUP   # 备份服务器上将MASTER 改为BACKUP
	interface ens33 //网卡名称
	virtual_router_id 51 # 主、备机的virtual_router_id必须相同
	priority 100   #主、备机取不同的优先级，主机值较大，备份机值较小
	advert_int 1	#每隔1s发送一次心跳
	authentication {	# 校验方式， 类型是密码，密码1111
        auth type PASS
        auth pass 1111
    }
	virtual_ipaddress { # 虛拟ip
		192.168.17.50 // VRRP H虛拟ip地址
	}
}
```

（2）在路径/usr/local/src/ 下新建检测脚本 nginx_check.sh

```shell
#! /bin/bash
A=`ps -C nginx -no-header | wc - 1`
if [ $A -eq 0];then
	/usr/local/nginx/sbin/nginx
	sleep 2
	if [`ps -C nginx --no-header| wc -1` -eq 0 ];then
		killall keepalived
	fi
fi
```

(3) 把两台服务器上nginx和keepalived启动

```shell
$ systemctl start keepalived.service		#keepalived启动
$ ps -ef I grep keepalived		#查看keepalived是否启动
$ systemctl stop keepalived.service  #keepalived停止
```

**5、最终测试**

(1) 在浏览器地址栏输入虚拟ip地址192.168.17.50

(2) 把主服务器(192.168.17.129) nginx和keealived停止，再输入192.168.17.50.

## Nginx原理解析

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Nginx/worker.jpg)

​		Nginx 启动之后，在 Linux 系统中有两个进程，一个为 master，一个为 worker。master 作为管理员不参与任何工作，只负责给多个 worker 分配不同的任务（worker 一般有多个）。

```bash
ps -ef |grep nginx 
root     20473     1  0  2019 ?        00:00:00 nginx: master process /usr/sbin/nginx 
nginx     4628 20473  0 Jan06 ?        00:00:00 nginx: worker process 
nginx     4629 20473  0 Jan06 ?        00:00:00 nginx: worker process
```

### worker 是如何工作的？

​		客户端发送一个请求首先要经过 master，管理员收到请求后会将请求通知给 worker，多个 worker 以**争抢**的机制来抢夺任务，得到任务的 worker 会将请求经由 tomcat 等做请求转发、反向代理、访问数据库等（nginx 本身是不直接支持 java 的）。

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Nginx/%E6%9C%BA%E5%88%B6.jpg)

### master-workers 的机制的好处 

- 对于每个 worker 进程来说，独立的进程，不需要加锁，所以省掉了锁带来的开销，同时在编程以及问题查找时，也会方便很多。
- 采用独立的进程，可以让互相之间不会影响，一个进程退出后，其它进程还在工作，服务不会中断，master 进程则很快启动新的worker 进程。当然，worker 进程的异常退出，肯定是程序有 bug 了，异常退出，会导致当前 worker 上的所有请求失败，不过不会影响到所有请求，所以降低了风险。

### 设置多少个 worker 合适？

​		Nginx 和 redis 类似，都采用了 io 多路复用机制，每个 worker 都是一个独立的进程，每个进程里只有一个主线程，通过异步非阻塞的方式来处理请求，每个 worker 的线程可以把一个 cpu 的性能发挥到极致，因此，**worker 数和服务器的 cpu 数相等是最为适宜的**。设少了会浪费 cpu，设多了会造成 cpu 频繁切换上下文带来的损耗

```shell
# 设置worker数量
worker.processes 4 

# work绑定cpu(4work绑定4cpu)
worker_cpu_affinity 0001 0010 0100 1000

# work绑定cpu (4work绑定8cpu中的4个)
worker_cpu_affinity 0000001 00000010 00000100 00001000
```

### 连接数 worker_connection

​		这个值是表示每个 worker 进程所能建立连接的最大值，所以，一个 nginx 能建立的最大连接数，应该是 **worker_connections * worker_processes**。当然，这里说的是最大连接数，对于HTTP 请 求 本 地 资 源 来 说 ， 能 够 支 持 的 最 大 并 发 数 量 是 worker_connections * worker_processes，如果是支持 http1.1 的浏览器每次访问要占两个连接，所以普通的静态访问最大并发数是： worker_connections * worker_processes /2，而如果是 HTTP 作 为反向代理来说，最大并发数量应该是 worker_connections * worker_processes/4。因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。

### 两个问题

**发送请求，占用了woker的几个连接数?**
	2或者4个。

**nginx有一个master,有四个woker,每个woker支持最大的连接数1024,支持的最大并发数是多少?**
	普通的静态访问最大并发数是: worker connections * worker processes /2，
	而如果是HTTP作为反向代理来说，最大并发数量应该是worker connections * worker processes/4



知识来源  |   [尚硅谷Nginx](http://www.atguigu.com/download_detail.shtml?v=221)

知识整理  |   Monster