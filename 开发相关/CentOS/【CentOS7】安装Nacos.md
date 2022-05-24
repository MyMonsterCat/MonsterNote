---
title: CentOS7安装Nacos
date: 2021-08-31 10:23:00
categories:
  - [Java,SpringCloud]
  - [CentOS7]
tags:
  - Linux
  - CentOS7
  - Nacos
description: CentOS7安装Nacos
copyright_author: Monster
keywords: Nacos
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/nacos.jpg
---

## 前提：需要java环境

## 官网下载最新版本并上传至服务器

https://github.com/alibaba/nacos/releases、

## 解压

```shell
tar -zxvf nacos-server-xxx.tar.gz 
```

## 修改配置文件的数据库连接

​	修改为自己实际的数据，注意`db.num=1，db.url.0=，db.user.0，db.password.0`

```shell
### If use MySQL as datasource:
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=nacos
db.password.0=nacos
```

## 数据库导入脚本

​	在数据库中刷入/nacos/conf目录下的nacos-mysql.sql数据库脚本，如果需要其他配置或者了解使用方式可以访问官网，官网地址：[https://nacos.io/zh-cn/docs/quick-start.html](https://links.jianshu.com/go?to=https%3A%2F%2Fnacos.io%2Fzh-cn%2Fdocs%2Fquick-start.html)。

## 执行

```shell
# 进入到bin目录下直接执行
sh startup.sh -m standalone
```

如果执行遇到问题`No DataSource set`

​	请参考文章进行解决：https://zhuanlan.zhihu.com/p/335362918

## 访问

​	服务启动之后，可以访问[http://ip:8848/nacos](https://links.jianshu.com/go?to=http%3A%2F%2Fip%3A8848%2Fnacos)访问管理后台，默认用户名密码：nacos/nacos



​	更多详细步骤请参考 |  https://cdmana.com/2021/03/20210330203555303b.html