---
title: CentOS7安装Redis
date: 2021-08-31 10:32:21
categories: 
  - [开发工具,Redis]
  - [CentOS7]
tags:
  - Linux
  - CentOS7
  - Redis
description: CentOS7安装Redis环境
copyright_author: Monster
keywords: Redis
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/redis.jpg

---

## 下载

官网下载最新版本https://redis.io/，上传至服务器

## 解压

使用命令解压

```shell
tar xzvf redis-xxx.tar.gz
```

## 编译

使用cd命令进入到解压后的文件夹内

```shell
# 使用make命令解压到指定路径
make install PREFIX=/disk/redis
```

## 配置

移动配置文件

```shell
# 在解压后的文件夹内找到redis.conf，移动到/disk/redis/bin目录下
cp /redis.conf /disk/redis/bin
#  redis-cli,redis-server拷贝到/usr/bin下，让redis-cli指令可以在任意目录下直接使用
cp /disk/redis/bin/redis-server /usr/local/bin/
cp /disk/redis/bin/redis-cli /usr/local/bin/

```

## 常用命令

```shell
# 启动
/disk/redis/bin/redis-server  /disk/redis/bin/redis.conf $

# 停止
pkill redis

# 查看后台进程
ps -ef |grep redis

# 查看端口是否监听
netstat -lntp | grep 6379
```

## 设置redis密码

```shell
# 进入终端
redis-cli
# 查看密码
config get requirepass 
# 下面是未设置密码的返回
# requirepass 
# ''
# 设置密码，再次访问用 auth 密码
config set requirepass 此处填写密码
```

## 外网可访问

>- 配置防火墙（过程省略）
>- redis-cli连接到redis后，通过 config get daemonize和config get protected-mode 是不是都为no，如果不是，就用config set 配置名 属性 改为no。
>- 注释配置文件中的127.0.0.1

## 参考

https://www.cnblogs.com/hunanzp/p/12304622.html

https://blog.csdn.net/weixin_42912498/article/details/102961338

https://www.huaweicloud.com/articles/eb7d3974351700e14e9ef15ecdfe02e9.html

https://blog.csdn.net/qq_32575047/article/details/106473140