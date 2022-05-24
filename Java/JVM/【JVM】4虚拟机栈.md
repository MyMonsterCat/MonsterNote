---
title: 虚拟机栈
date: 2021-05-01 12:13:42
description: 由于跨平台性的设计，Java的指令都是根据栈来设计的
categories:
  - [Java,JVM]
tags:
  - JVM
keywords: JVM
copyright_author: Monster
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/7.jpg
---

## 虚拟机栈

​	由于跨平台性的设计，==Java的指令都是根据栈来设计的==。不同平台CPU架构不同，所以不能设计为基于寄存器的。 优点是跨平台，指令集小，编译器容易实现，缺点是性能下降，实现同样的功能需要更多的指令。

> ​	有不少Java开发人员一提到Java内存结构，就会非常粗粒度地将JVM中的内存区理解为仅有Java堆（heap）和Java栈（stack）？为什么？

首先==栈是运行时的单位，而堆是存储的单位==

- 栈解决程序的运行问题，即程序如何执行，或者说如何处理数据。
- 堆解决的是数据存储的问题，即数据怎么放，放哪里

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/%E6%A0%88%E5%92%8C%E5%A0%86.png)

> 简单来说，==栈管运行，堆管存储，栈是运行时的单位，而堆是存储的单位==

##  Java虚拟机栈是什么

​	Java虚拟机栈（Java Virtual Machine Stack），早期也叫Java栈。==每个线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧（Stack Frame），对应着一次次的Java方法调用，是线程私有的。==

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/%E6%A0%88%E5%B8%A7.jpg)

### 生命周期

​	生命周期和线程一致，也就是线程结束了，该虚拟机栈也销毁了

### 作用

​	主管Java程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。

> 局部变量，它是相比于成员变量来说的（或属性）

### 栈的特点

栈是一种快速有效的分配存储方式，==访问速度仅次于程序计数器==。JVM直接对Java栈的操作只有两个：

- 每个方法执行，伴随着**进栈**（入栈、压栈）
- 执行结束后的**出栈**工作

## 开发中遇到哪些异常？

​	==对于栈来说不存在垃圾回收问题，但是存在OOM和StackOverflowError==

​	Java 虚拟机规范允许**Java栈的大小是动态的或者是固定不变**的。

​	如果采用固定大小的Java虚拟机栈，那每一个线程的Java虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过Java虚拟机栈允许的最大容量，Java虚拟机将会抛出一个StackoverflowError 异常。

​	如果Java虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那Java虚拟机将会抛出一个 outofMemoryError 异常。

```java
public class StackErrorTest {
    private static int count = 1;
    public static void main(String[] args) {
        System.out.println(count++);
        main(args);
    }
}
```

当栈深度达到9803的时候，就出现栈内存空间不足

### 设置栈内存大小

我们可以使用参数` -Xss`选项来设置线程的最大栈空间，==栈的大小直接决定了函数调用的最大可达深度==

```java
-Xss1m
-Xss1k
```

##  栈的存储单位

​	==每个线程都有自己的栈==，栈中的数据都是以==栈帧（Stack Frame）==的格式存在。

​	==在这个线程上正在执行的每个方法都各自对应一个栈帧（Stack Frame）==。

​	栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息。

##  栈中存储什么？

​	每个线程都有自己的栈，栈中的数据都是以栈帧（Stack Frame）的格式存在。在这个线程上正在执行的每个方法都各自对应一个栈帧（Stack Frame）。栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息。

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/%E6%A0%88%E5%B8%A7%E5%AF%B9%E5%BA%94.png)



来看下面一个例子

```java
public class StackFrameTest {
    public static void main(String[] args) {
        method01();
    }

    private static int method01() {
        System.out.println("方法1的开始");
        int i = method02();
        System.out.println("方法1的结束");
        return i;
    }

    private static int method02() {
        System.out.println("方法2的开始");
        int i = method03();;
        System.out.println("方法2的结束");
        return i;
    }
    private static int method03() {
        System.out.println("方法3的开始");
        int i = 30;
        System.out.println("方法3的结束");
        return i;
    }
}
```

输出的结果为

```java
方法1的开始
方法2的开始
方法3的开始
方法3的结束
方法2的结束
方法1的结束
```

满足栈先进后出的概念

##  栈运行原理

​	不同线程中所包含的栈帧是不允许存在相互引用的，即==不可能在一个栈帧之中引用另外一个线程的栈帧==。

​	如果当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着，虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧。

​	Java方法有两种返回函数的方式，一种是正常的函数返回，使用return指令；另外一种是抛出异常。不管使用哪种方式，都会导致栈帧被弹出。

##  栈帧的内部结构

每个栈帧中存储着：

- 局部变量表（Local Variables）
- 操作数栈（operand Stack）（或表达式栈）
- 动态链接（DynamicLinking）（或指向运行时常量池的方法引用）
- 方法返回地址（Return Address）（或方法正常退出或者异常退出的定义）
- 一些附加信息

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/%E6%A0%88%E7%9A%84%E5%86%85%E9%83%A8%E7%BB%93%E6%9E%84.png)















