---
title: Lambda 表达式中的变量必须是 final ?
date: 2021-10-08 18:39:21
description: Lambda 表达式中的变量必须是 final ?
categories:
  - [其他]
tags:
  - Java
keywords: SpringClond
copyright_author: Monster
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/12.jpg

---

​	偶尔，我们需要在 Lambda 表达式中修改变量的值，但如果直接尝试修改的话，编译器不会视而不见听而不闻，它会警告我们说：“variable used in lambda expression should be final or effectively final”。

![image-20211008184358759](https://gitee.com/lc_monster/my-image/raw/master/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/6/image-20211008184358759.png)

这个问题发生的原因是因为 Java 规范中是这样规定的：

> Any local variable, formal parameter, or exception parameter used but not declared in a lambda expression
> must either be declared final or be effectively final (§4.12.4),
> or a compile-time error occurs where the use is attempted.

大致的意思就是说，Lambda 表达式中要用到的，但又未在 Lambda 表达式中声明的变量，必须声明为 final 或者是 effectively final，否则就会出现编译错误。

关于 final 和 effectively final 的区别，可能有些小伙伴不太清楚，这里多说两句。

```java
final int a;
a = 1;
// a = 2;
// 由于 a 是 final 的，所以不能被重新赋值

int b;
b = 1;
// b 此后再未更改
// b 就是 effectively final

int c;
c = 1;
// c 先被赋值为 1，随后又被重新赋值为 2
c = 2;
// c 就不是 effectively final

```


​	明白了 final 和 effectively final 的区别后，我们了解到，如果把 limit 定义为 final，那就无法在 Lambda 表达式中修改变量的值。那有什么好的解决办法呢？既能让编译器不发出警告，又能修改变量的值。

​	思前想后，试来试去，我终于找到了 3 个可行的解决方案：

- 把 limit 变量声明为 static。


- 把 limit 变量声明为 AtomicInteger。


- 使用数组。

下面我们来详细地一一介绍下。

#### 01、把 limit 变量声明为 static

要想把 limit 变量声明为 static，就必须将 limit 变量放在 main() 方法外部，因为 main() 方法本身是 static 的。完整的代码示例如下所示。

```java
public class ModifyVariable2StaticInsideLambda {
    static int limit = 10;
    public static void main(String[] args) {
        Runnable r = () -> {
            limit = 5;
            for (int i = 0; i < limit; i++) {
                System.out.println(i);
            }
        };
        new Thread(r).start();
    }
}
```

#### 02、把 limit 变量声明为 AtomicInteger

​	AtomicInteger 可以确保 int 值的修改是原子性的，可以使用 set() 方法设置一个新的 int 值，get() 方法获取当前的 int 值。

```java
public class ModifyVariable2AtomicInsideLambda {
    public static void main(String[] args) {
        final AtomicInteger limit = new AtomicInteger(10);
        Runnable r = () -> {
            limit.set(5);
            for (int i = 0; i < limit.get(); i++) {
                System.out.println(i);
            }
        };
        new Thread(r).start();
    }
}

```

#### 03、使用数组

使用数组的方式略带一些欺骗的性质，在声明数组的时候设置为 final，但更改 int 的值时却修改的是数组的一个元素。

```java
public class ModifyVariable2ArrayInsideLambda {
    public static void main(String[] args) {
        final int [] limits = {10};
        Runnable r = () -> {
            limits[0] = 5;
            for (int i = 0; i < limits[0]; i++) {
                System.out.println(i);
            }
        };
        new Thread(r).start();
    }
}
```


原文链接：https://blog.csdn.net/qing_gee/article/details/104438986