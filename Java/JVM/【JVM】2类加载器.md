---
title: 类加载器
date: 2021-04-27 16:15:16
description: 一般情况下，Java开发人员并不需要在程序中显式地使用类加载器，但是如果遇到相关问题，就需要了解内部原理才能够进行排查和解决
categories:
  - [Java,JVM]
tags:
  - JVM
keywords: JVM
copyright_author: Monster
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/5.jpg
---

我们已经知道，类的前三个加载过程是==加载--链接--初始化==，而执行这三个过程的就是类加载器子系统

## 学习类加载器有必要吗？

​	一般情况下，Java开发人员并不需要在程序中显式地使用类加载器，但是如果遇到相关问题，就需要了解内部原理才能够进行排查和解决

​	我们有时候开发中遇到java.lang.ClassNotFoundException异常或java.lang.NoClassDeFoundError异常，面对这类异常时，我们经常手足无措。所以只有了解了类加载器的加载机制，才能够在出现异常的时候快速地根据错误异常日志定位问题和解决问题



## 类加载器的作用

|                             作用                             |
| :----------------------------------------------------------: |
| 类加载器子系统负责从文件系统或者网络中加载Class文件，class文件在文件开头有特定的文件标识。 |
| Classloader只负责class文件的加载，至于是否可运行，则由执行引擎决定 |
| 加载的类信息存放于称为方法区的内存空间，除了类信息，方法区还会存放运行时常量池信息，还可能包括字符串字面量和数字常量 |

​	

未完待续...