---
title: CentOS7开启iptables
date: 2021-04-16 09:25:00
author: Monster
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/3.png
description: CentOS7开启iptables
categories:
  - [CentOS7]
tags:
  - Linux
  - CentOS7
  - iptables
---

## 第一步：关闭firewalld

```clike
systemctl status firewalld #查看防火墙状态
systemctl stop firewalld  #停止防火墙
systemctl disable firewalld  #禁用防火墙
```

## 第二步：启用iptables

```clike
yum install iptables-services #安装iptables
vi /etc/sysconfig/iptables #编辑防火墙配置文件
```

```clike
*filter

:INPUT ACCEPT [0:0] 

:FORWARD ACCEPT [0:0] 

:OUTPUT ACCEPT [0:0] 

-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT 

-A INPUT -p icmp -j ACCEPT

-A INPUT -i lo -j ACCEPT

-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT

-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT 

-A INPUT -m state --state NEW -m tcp -p tcp --dport 9527 -j ACCEPT

-A INPUT -j REJECT --reject-with icmp-host-prohibited 

-A FORWARD -j REJECT --reject-with icmp-host-prohibited

COMMIT

:wq! #保存退出
```

```clike
systemctl restart iptables.service #重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动
```

```clike
systemctl enable iptables.service #打开
systemctl status iptables #查看状态
systemctl start iptables #开启
systemctl restart iptables.service #重启
```

