---
title: 题目：求自增变量的值
date: 2021-05-26 23:08:57
description: 自增变量
categories:
  - [Java,JVM]
tags:
  - JVM
keywords: SpringClond
copyright_author: Monster
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/9.jpg
---

请计算下面这段程序的输出

```java
 public static void main(String[] args) {
        int i = 1;
        i = i++;
        int j = i++;
        int k = i + ++i * i++;
        System.out.println("i=" + i);
        System.out.println("j=" + j);
        System.out.println("k=" + k);
    }
```

 结果为：i=4，j=1，k =11

 首先我们要清楚在JVM中的运行时数据区的结构

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E7%9A%84%E7%BB%93%E6%9E%84.png)

如图所示，不难理解，我们的主函数拥有一个栈帧，局部变量保存在局部变量表中，操作的变量值保存在操作数栈中

首先看这两行代码

```java
int i = 1;
i = i++;
```

画个图方便理解

**执行 i = i++; 先将i变量压入操作数栈**

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526203145976.png)

**然再对i变量进行自增（自增在局部变量表中）**，注意此时操作数栈中还是1

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526203219133.png)

**最后把计算结果赋值给 i , i 仍然是1**（虽然自增加1了但是又被重新赋值1了）

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526203145976.png)

接下来是

```java
int j = i++;
```

同样的道理，虽然i自增过了，但是是在局部变量表中自增的，最后又把操作数栈中的值**i=1**赋值给了j

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526203852009.png)

最后是

```java
int k = i + ++i * i++;
```

①首先把i的值压入操作数栈，此时操作数栈中的i=1

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526203952785.png)

**先执行++i**

②局部变量表中的i变量自增1，即局部变量表中的i=2

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526204001726.png)

③把局部变量表中的i压入操作数栈，**此时操作数栈中的i=1+2 =3**

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526204347748.png)

**再执行i++**

④把局部变量表中的i压入操作数栈，**此时操作数栈中有两个3**

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526204803958.png)

⑤局部变量表中的i变量

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526204937835.png)

⑥i自增后又被赋值为3，3和3出栈，求乘积

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526212913300.png)

⑦求和赋值为k

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526213050763.png)

![](https://gitee.com/lc_monster/my-image/raw/master/Topic/image-20210526213147003.png)

它所对应的字节码如下所示

```clike
         0: iconst_1
         1: istore_1
         2: iload_1
         3: iinc          1, 1
         6: istore_1
         7: iload_1
         8: iinc          1, 1
        11: istore_2
        12: iload_1
        13: iinc          1, 1
        16: iload_1
        17: iload_1
        18: iinc          1, 1
        21: imul
        22: iadd
        23: istore_3
        24: return
```





