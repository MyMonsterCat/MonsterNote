---
title: 类的加载过程
date: 2021-04-24 16:15:16
description: 按照Java虚拟机规范，从Class文件到加载到内存中的类，到类卸载出内存位置，它的整个生命周期包括七个阶段
categories:
  - [Java,JVM]
tags:
  - JVM
keywords: JVM
copyright_author: Monster
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/8.jpg
---

#  二、类的加载过程

## 2.1 完整的类加载过程

​	在了解类的加载系统之前，需要在宏观上对类的加载过程有一定的了解，这样才能在学习加载系统的清楚的明白各部件的作用。

​	按照Java虚拟机规范，从Class文件到加载到内存中的类，到类卸载出内存位置，它的整个生命周期包括如下七个阶段：==加载 -> 链接（验证、准备、解析） -> 初始化 -> 使用 -> 卸载==。其中链接分为验证、准备、解析。

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/%E5%AE%8C%E6%95%B4%E8%BF%87%E7%A8%8B.png)

​	比如下面一段简单的代码

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("我已经被加载了");
    }
}
```

它的加载过程是怎样的呢？

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B.png)



按照过程我们详细阐述。

## 2.2 加载阶段

### 1. 加载的理解

​	所谓加载，就是==将字节码文件加载到内存中，并且在内存中构建出Java类的原型==--**类模板对象**

- ​	类模板对象（instanceKlass），其实就是**Java类在JVM内存中的一个快照**

- ​	Java的对象并没有映射成C++的原生对象，而是==使用了OOP-KLASS模型来表示Java对象==

- ​	JVM将从字节码文件中解析出的常量池、 类字段、类方法等信息存储到模板中，这样JVM在运行期便能通过类模板而获取Java类中的任意信息，能够对Java类的成员变量进行遍历，也能进行Java方法的调用
- ​    反射的机制即基于这一基础。如果JVM没有将Java类的声明信息存储起 来，则JVM在运行期也无法反射

### 2. 加载的三个步骤

<details>
    <summary>  <b>加载阶段流程</b>  </summary>         
    <pre>
    	1.通过一个类的<font color = red>全限定类名</font>获取定义此类的二进制字节流
		2.将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
		3.在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入
	</pre> 
</details>

<details>
    <summary>  加载class文件的方式（对应流程中第一条）  </summary>         
    <pre>
    	1.从本地系统中直接加载
    	2.通过网络获取，典型场景：Web Applet
    	3.从zip压缩包中读取，成为日后jar、war格式的基础
    	4.运行时计算生成，使用最多的是：动态代理技术
    	5.由其他文件生成，典型场景：JSP应用从专有数据库中提取.class文件，比较少见
    	6.从加密文件中获取，典型的防Class文件被反编译的保护措施
	</pre> 
</details>

### 3. 类模型与Class实例的位置

​	**类模型的位置**：加载的类由JVM创建相应的类结构，类结构会存储在==方法区==（JDK 1.8之前：永久代；JDK1.8之后：==元空间==）

​	**Class实例的位置**：JVM将.class文件加载到方法区后，会在==堆内存==中创建一个java.lang.Class类的对象实例，用来封装类位于方法区内的数据结构。该Class对象是在加载类的过程中创建的，每个类都对应一个Class对象，==Class类的构造方法是私有的，只有JVM能够创建==。java.lang.Class实例是访问类型元数据的接口，也是实现反射的关键数据和入口。通过Class类提供的接口，可以获得目标类所关联的.class文件具体的数据结构：方法、字段

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/%E7%B1%BB%E6%A8%A1%E5%9E%8B%E4%B8%8EClass%E4%BD%8D%E7%BD%AE.png)

## 2.3 链接阶段

### 1. 验证

​	**目的**在于确保Class文件的字节流中包含信息符合当前的虚拟机要求，保证被加载类的正确性，不会危害虚拟机自身安全。

​	**主要包括四种验证**：==文件格式验证，语义验证，字节码验证，符号引用验证==

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/%E5%9B%9B%E7%A7%8D%E9%AA%8C%E8%AF%81.png)

#### 1.1 文件格式验证

​	在深入了解文件格式验证以前，先来做个小实验，我们使用字节码查看工具（**Binary Viewer**）随便查看一个编译后的字节码文件

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/Binary%20Viewer%E6%9F%A5%E7%9C%8B.png)

​	多打开几个字节码文件，我们发现，这些文件竟然都是==以0xCAFEBABE开头==，这就是我们是所说的**被JVM识别的魔数**，也就是说，==如果出现了不合法的字节码文件，那么将会导致JVM识别魔数失败，也就会导致验证不通过==

​	另外，我们也可以通过安装IDEA的插件（**jclasslib BytecodeViewer**），来查看我们的Class文件

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/jclasslib%20BytecodeViewer.png)

​	安装完成后，我们编译完一个class文件后，点击view即可显示我们安装的插件来查看字节码方法了

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/jc%E6%9F%A5%E7%9C%8B%E5%AD%97%E8%8A%82%E7%A0%81.png)



除了校验魔数外，文件格式验证还包括：

- 主次版本号（Minor Version、Major Version）

- 常量池的常量是否有不被支持的常量类型

- 指向常量的各种索引值中是否有指向不存在的常量或不符合类型的常量



#### 1.2 语义验证

所谓语义验证，也就是==检查是不是符合java语法==，一般来说包括：

- 类是否有父类，除了Object类之外，所有的类都应该有父类
- 类的父类是否继承了不允许被继承的类（被final修饰的类）
- 如果这个类不是抽象类，是否实现了其父类或接口中要求实现的所有方法
- 类的字段，方法是否与父类的产生矛盾。例如方法参数都一样，返回值不同



#### 1.3 字节码验证

字节码验证一般包括以下内容

- 通过数据流分析和控制流分析，确定程序语义是合法的，符合逻辑的

- 对类的方法体，进行校验分析，保证在运行时不会做出危害虚拟机的行为

- ==保证任意时刻操作数栈的数据类型与指令代码序列都能配合工作==，不会出现类似于在操作数栈放了一个int类型的数据，使用时却按照long类型加载到本地变量表中的情况

- 保障任何跳转指令都不会跳转到方法体之外的字节码指令上



#### 1.4 符号引用

- 通过字符串描述的全限定名是否能找到对应的类

- 符号引用中的类、字段、方法的可访问性是否可被当前类访问



### 2. 准备

​	为类变量分配内存并且设置该类变量的默认初始值，即==零值==。

​	![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E9%9B%B6%E5%80%BC.png)

先来看一个例子：


```java
public class HelloApp {
    private static int a = 1;  // 准备阶段为0，在下个阶段，也就是初始化的时候才是1
    public static void main(String[] args) {
        System.out.println(a);
    }
}
```

> 上面的变量a在准备阶段会赋初始值，但不是1，而是0。
>
> 这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中。

**总结：**

- 赋默认初始值时，不包含用final修饰的static字段，因为**final在编译阶段就会分配初始值了**，准备阶段会显示初始化

- **基本数据类型**：非final修饰的变量，在准备环节进行默认初始化赋值。final修饰以后，在准备环节直接进行显式赋值

- 如果使用字面量的方式定义一个字符串的常量的话，也是在准备环节直接进行显式赋值，如下图所示

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/%E8%B5%8B%E5%88%9D%E5%A7%8B%E5%80%BC.png)



### 3. 解析

​	**定义**：将常量池内的符号引用转换为直接引用的过程，也即将类、接口、方法、字段的符号引用转换为直接引用

​	**符号引用就是一组符号来描述所引用的目标**。符号引用的字面量形式明确定义在《java虚拟机规范》的class文件格式中，直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

​	解析动作主要针对**类或接口、字段、类方法、接口方法、方法类型**等。对应常量池中的CONSTANT Class info、CONSTANT Fieldref info、CONSTANT Methodref info等

​	事实上，在HotSpot VM中，加载、验证、准备和初始化会按照顺序有条不紊地执行，但是==解析操作往往会伴随着JVM在执行完初始化之后再执行==



## 2.4 初始化阶段

​	初始化阶段是类加载的最后一个阶段。如果前面的步骤都没有问题，那么表示类可以顺利装载到系统中。此时类才会执行Java字节码（到了初始化阶段，才真正开始执行类中定义的Java代码）, 简单来说，==初始化阶段是执行类构造器方法<clinit>()的过程，该方法只会被执行一次，且虚拟机的实现需要保证多线程情况下被正确地同步枷锁==

先来看三个例子：

```java
// 示例1
public class ClassInitTest {
    private static int num = 1;
    static {
        num = 2;
        number = 20;
        System.out.println(num);
        System.out.println(number);  //报错，非法的前向引用
    }

    private static int number = 10;

    public static void main(String[] args) {
        System.out.println(ClassInitTest.num); // 2
        System.out.println(ClassInitTest.number); // 10
    }
}
```

> 上述代码，在类加载的过程中：
>
> ​	对于num，其为静态，准备阶段为0，初始化阶段赋值为1	
>
> ​	对于number，其为静态，准备阶段为0，初始化阶段赋值先赋值为20，随后赋值为10，在JVM看来，流程是没有问题，但是在IDEA编辑器看来，在number还未声明的时候就去调用它的toString()去输出，是不符合语法规范的，所以会报非法的前向引用这个错

```java
// 示例2
public class ClinitTest {
    static class Father {
        public static int A = 1;
        static {
            A = 2;
        }
    }

    static class Son extends Father {
        public static int b = A;
    }

    public static void main(String[] args) {
        System.out.println(Son.b);
    }
}
```

> 上述代码，在类加载的过程中：
>
> ​	由于main函数输出的是son.b，首先先去执行Son的初始化，但是Son继承了Father，因此我们先执行Father初始化，同时在执行Father初始化过程中，对于A，准备阶段零值为0，初始化阶段先赋值为1，然后进入静态代码块，赋值为2，最后被A=2传给Son的B，所以son.b = 2

来看下示例2的字节码

```basic
iconst_1
putstatic #2 <com/monster/jvm/ClinitTest$Father.A>
iconst_2
putstatic #2 <com/monster/jvm/ClinitTest$Father.A>
return
```



```java
// 示例3
public class DeadThreadTest {
    public static void main(String[] args) {
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t 线程t1开始");
            new DeadThread();
        }, "t1").start();

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t 线程t2开始");
            new DeadThread();
        }, "t2").start();
    }
}
class DeadThread {
    static {
        if (true) {
            System.out.println(Thread.currentThread().getName() + "\t 初始化当前类");
            while(true) {

            }
        }
    }
}
```

上面示例3的代码，输出结果为

```
线程t1开始
线程t2开始
线程t2 初始化当前类
```

> ​	从上面可以看出初始化后，只能够执行一次初始化，这也就是同步加锁的过程，**虚拟机必须保证一个类的无参构造方法在多线程下被同步加锁**

#### 初始化过程总结

​	初始化方法不需定义，是javac编译器**自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来**。

​	<clinit>()不同于类的构造器（关联：构造器是虚拟机视角下的<init>()）若该类具有父类，JVM会保证子类的<clinit>()执行前，父类<clinit>()已经执行完毕。**由父及子，父类先行**

​	也就是说，当我们代码中包含static变量的时候，就会有clinit方法，**如果没有类变量和静态代码块**，也不会有<clinit>()方法，构造器方法中指令按语句**在源文件中出现的顺序**执行

> ​	换一种说法，也就是Java编译器并不会为所有的类都产生<clinit>()方法。我们知道，如果没有类变量和静态代码块，就不会有<clinit>()方法，那么哪些情况下不会生成<clinit>()方法呢？

- 一个类中没有任何类变量以及静态代码块
- 一个类声明了类变量，但是没有显式赋值
- 一个类中包含static final修饰的基本数据类型的字段，这些类字段初始化语句采用编译时常量表达式

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/%E4%B8%8D%E4%BC%9A%E7%94%9F%E6%88%90clinit.png)

> 如图场景3所示，那么使用static+final修饰的字段的显式赋值的操作，到底是在那个阶段进行的赋值？

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/static%20final.png)



## 2.5 使用阶段

使用（Using）一般来说有4种方式

- 使用new关键字创建对象
- 使用反射的方式创建对象
- 调用clone()方法创建对象
- 使用反序列化方式得到对象



## 2.6 卸载阶段

### 1. 类、类的加载器、类的实例之间的引用关系

​	其实，在类加载器内部实现中，用一个Java集合（Vector）存放所加载类的引用，如下图所示

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/Vector.png)

​	一个Class对象**总是会引用它的类加载器**，调用Class对象的getClassLoader()方法就可以获得它的类加载器

​	由此可见某个类的Class实例与其加载的类加载器之间为双向关联关系

​	一个类的对象实例总是引用代表这个类的Class对象。在Object类中定义了getClass()方法，这个方法返回对象所属类的Class对象的引用

![](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/JVM/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD/%E5%BC%95%E7%94%A8%E5%85%B3%E7%B3%BB.png)



### 2. 类的生命周期

​	当一个类被加载、链接、初始化后，它的生命周期就开始了。当代表这个类的Class对象不再被引用，即**不可触及**时，Class对象的生命周期就结束了，这个类在方法区内的数据也会被卸载，从而结束这个类的生命周期。

​	如上图所示，loader1变量和obj变量间接引用Sample类的Class对象，而objClass变量则直接引用它，如果在运行中，将上图左侧三个引用类型变量==都置为null，此时Sample对象生命周期结束，MyClassLoader对象结束生命周期，代表Sample类的Class对象也结束生命周期，Sample类在方法区内的二进制数据也被卸载==，当再次需要使用时，会检查Sample类的Class对象是否存在，如果存在会直接使用，不再重新加载，如果不存在Sample类会被重新加载，在Java虚拟机的堆内存中生成一个新的代表Sample类的Class实例。



### 3. 类的卸载

​	被启动类加载器的类型在==整个运行期间是不可能被卸载==的（JVM和JSL规范）

​	被系统类加载器和扩展类加载器加载的类型在运行期不太可能被卸载，因为系统类加载器或者扩展类的实例基本上在整个运行期间总能直接或者间接访问到

​	被开发者自定义的类加载器加载的类型只有在很简单的上下文环境中才能被卸载，而且一般还要借助于调用虚拟机的垃圾收集功能才可以做到

​	因此，一个已经加载的类型被卸载的几率很小，几乎不会被卸载



### 4. 方法区的垃圾回收

方法区的垃圾收集主要是==常量池中废弃的常量和不再使用的类型==

只要常量池中的常量==没有被任何地方引用==，就可以被回收

> 判断一个类型能否被回收，需要同时满足满足三个条件
>
> - 该类的所有实例都已经被回收，Java堆内存中不存在该类以及派生子类的实例
>
> - 加载该类的类加载器已经被回收
>
> - 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法
>
>  满足以上三个条件，**仅仅是允许被回收，并不是和对象一样，没有引用了就必然被回收**

## 2.7 补充说明

​	加载、验证、准备、初始化、使用和卸载这六个阶段的顺序是**确定的**

​	**解析阶段不一定**，在某些情况下可以在初始化阶段之后再开始，为了支持Java语言的运行时绑定特性（也称为动态绑定或晚期绑定，其实就是多态），例如子类重写父类方法