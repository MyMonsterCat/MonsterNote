---
title: CentOS7安装Java环境
date: 2021-08-30 16:57:16
categories: 
  - [Java,Java基础]
  - [CentOS7]
tags:
  - Linux
  - CentOS7
  - Java
description: CentOS7安装Java
copyright_author: Monster
keywords: Java
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/16.jpg
---

## 使用yum安装

```shell
yum install java-1.8.0-openjdk-devel.x86_64
```

## 配置java路径

```shell
vi /etc/profile
```

## 复制以下三行到文件中,下面路径根据自己的安装路径来。

```

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.el6_9.x86_64

export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export PATH=$PATH:$JAVA_HOME/bin
```

## 查看是否安装成功

```shell
# 立即刷新
source /etc/profile
# 查看java版本
java -version
```