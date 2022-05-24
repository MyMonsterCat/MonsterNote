---
title: 运行时数据区
date: 2021-04-29 16:15:16
description: Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域，这些区域有各自的用途
categories:
  - [Java,JVM]
tags:
  - JVM
keywords: JVM
copyright_author: Monster
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/6.jpg
---

## 运行时数据区是什么

​	Java虚拟机在执行Java程序的过程中会==把它所管理的内存划分为若干个不同的数据区域==，这些区域有各自的用途，以及创建和销毁的时间，有的区域随着虚拟机进程的启动而一直存在，有些区域则是依赖用户线程的启动和结束而建立和销毁。根据《Java虚拟机规范》的规定，Java虚拟机所管理的内存 将会包括以下几个运行时数据区域：

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E7%9A%84%E7%BB%93%E6%9E%84.png)

其中，**虚拟机栈、本地方法栈、程序计数器为线程私有，方法区、堆内存为线程共享**

## 程序计数器 

​	程序计数器(Program Counter Register)的命名源于CPU的寄存器，寄存器是用来存储指令相关的现场信息，JVM的PC寄存器是对物理PC寄存器的一种抽象模拟，它有以下几个特点：

- ==用于存储下一条即将要执行的字节码指令地址==
- 是JVM中运行速度最快的一块区域
- ==运行时数据区中唯一不会出现OOM的区域==
- 没有垃圾回收
- 程序计数器为线程私有，每个线程都有自己的程序计数器。每个线程有一个独立的程序计数器，线程之间互不影响，即线程私有且生命周期与线程的生命周期保持一致

### 演示PC寄存器

PC寄存器用来存储指向下一条指令的地址，也即将要执行的指令代码。由执行引擎读取下一条指令。

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/%E5%AF%84%E5%AD%98%E5%99%A8.png)

我们先来看下面一段代码

```java
public class PCRegisterTest {
    public static void main(String[] args) {
        int i = 10;
        int j = 20;
        int k = i + j;
    }
}
```

​	使用命令`javap -verbose xxx.class`将代码进行编译成字节码文件，我们再次查看 ，发现在字节码的左边有一个行号标识，它其实就是指令地址，用于指向当前执行到哪里

```java
0: bipush        10
2: istore_1
3: bipush        20
5: istore_2
6: iload_1
7: iload_2
8: iadd
9: istore_3
10: return
```

​	通过PC寄存器，我们就可以知道当前程序执行到哪一步了

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/%E6%93%8D%E4%BD%9C%E6%8C%87%E4%BB%A4%E8%BF%87%E7%A8%8B.png)

##  使用PC寄存器存储字节码指令地址有什么用呢？

​	因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行。

​	JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令。

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/Java%E5%88%87%E6%8D%A2%E7%BA%BF%E7%A8%8B.png)

##  PC寄存器为什么被设定为私有的？

​	我们都知道所谓的多线程在一个特定的时间段内只会执行其中某一个线程的方法，CPU会不停地做任务切换，这样必然导致经常中断或恢复，如何保证分毫无差呢？为了能够准确地记录各个线程正在执行的当前字节码指令地址，最好的办法自然是为每一个线程都分配一个PC寄存器，这样一来各个线程之间便可以进行独立计算，从而不会出现相互干扰的情况。

​	由于CPU时间片轮限制，众多线程在并发执行过程中，任何一个确定的时刻，一个处理器或者多核处理器中的一个内核，只会执行某个线程中的一条指令。

​	这样必然导致经常中断或恢复，如何保证分毫无差呢？每个线程在创建后，都会产生自己的程序计数器和栈帧，程序计数器在各个线程之间互不影响。

###  CPU时间片

​	CPU时间片即CPU分配给各个程序的时间，每个线程被分配一个时间段，称作它的时间片。

​	在宏观上：俄们可以同时打开多个应用程序，每个程序并行不悖，同时运行。

​	但在微观上：由于只有一个CPU，一次只能处理程序要求的一部分，如何处理公平，一种方法就是引入时间片，每个程序轮流执行











