---
title: CentOS7安装LibreOffice
date: 2021-02-01 09:25:00
categories:
  - [CentOS7]
tags:
  - Linux
  - CentOS7
  - LibreOffice
description: CentOS7安装LibreOffice！
copyright_author: Monster
keywords: LibreOffice
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/2.png
---

# CentOS7安装LibreOffice

## 下载安装包

①网站下载

**官网**
https://zh-cn.libreoffice.org/

**科大镜像**
[http://mirrors.ustc.edu.cn/td...](http://mirrors.ustc.edu.cn/tdf/libreoffice/stable/6.0.4/rpm/x86_64/LibreOffice_6.0.4_Linux_x86-64_rpm.tar.gz)

②wget下载

```shell
wget -P /tmp/office http://mirrors.ustc.edu.cn/tdf/libreoffice/stable/6.0.4/rpm/x86_64/LibreOffice_6.0.4_Linux_x86-64_rpm.tar.gz

wget -P /tmp/office http://mirrors.ustc.edu.cn/tdf/libreoffice/stable/6.0.4/rpm/x86_64/LibreOffice_6.0.4_Linux_x86-64_rpm_langpack_zh-CN.tar.gz
```

## 解压缩

```powershell
tar zxvf /tmp/office/LibreOffice_7.1.3.2_Linux_x86-64_rpm
```

## 进入到RPMS文件夹

```shell
yum install *.rpm
```

## 安装libcairo.so.2依赖库

```shell
yum install ibus
```

## 默认安装路径

```shell
/opt/libreoffice7.1
```

## 安装字体库

```shell
#新建文件夹
mkdir /usr/share/fonts/chinese
```

把Windows系统的字体C:\Windows\Fonts复制进去

```shell
# 修改字体权限
chmod -R 644 /usr/share/fonts/chinese
#刷新系统
sudo fc-cache -fv
```

参考文章 | https://segmentfault.com/a/1190000015129654?utm_source=channel-hottest

参考文章 | https://www.jianshu.com/p/555d118aaf8a