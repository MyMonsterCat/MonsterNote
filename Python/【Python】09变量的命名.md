---
title: Python基础-09变量的命名
date: 2021-06-17 09:45:49
description: 变量的命名
categories:
  - [Python,Python基础]
tags:
  - Python
keywords: Python
copyright_author: Monster
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/Python.jpg
---

# 变量的命名

## 目标

* 标识符和关键字
* 变量的命名规则

## 0.1 标识符和关键字

### 1.1 标识符

> 标示符就是程序员定义的 **变量名**、**函数名**
> 
> **名字** 需要有 **见名知义** 的效果，见下图：

![001_中国山东找蓝翔](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Python/009/001_中国山东找蓝翔.jpg)

* 标示符可以由 **字母**、**下划线** 和 **数字** 组成
* **不能以数字开头**
* **不能与关键字重名**

思考：下面的标示符哪些是正确的，哪些不正确为什么？

```
fromNo12
from#12
my_Boolean
my-Boolean
Obj2
2ndObj
myInt
My_tExt
_test
test!32
haha(da)tt
jack_rose
jack&rose
GUI
G.U.I
```

### 1.2 关键字

* **关键字** 就是在 `Python` 内部已经使用的标识符
* **关键字** 具有特殊的功能和含义
* 开发者 **不允许定义和关键字相同的名字的标示符**

通过以下命令可以查看 `Python` 中的关键字

```python
In [1]: import keyword
In [2]: print(keyword.kwlist)
```

> 提示：**关键字的学习及使用**，会在后面的课程中不断介绍
> 
> * `import` **关键字** 可以导入一个 **“工具包”**
> 
> * 在 `Python` 中不同的工具包，提供有不同的工具

## 02. 变量的命名规则

> **命名规则** 可以被视为一种 **惯例**，并无绝对与强制
> 目的是为了 **增加代码的识别和可读性**

**注意** `Python` 中的 **标识符** 是 **区分大小写的**

![002_标识符区分大小写](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Python/009/002_标识符区分大小写.jpg)

1. 在定义变量时，为了保证代码格式，`=` 的左右应该各保留一个空格
2. 在 `Python` 中，如果 **变量名** 需要由 **二个** 或 **多个单词** 组成时，可以按照以下方式命名
    1. 每个单词都使用小写字母
    2. 单词与单词之间使用 **`_`下划线** 连接
    * 例如：`first_name`、`last_name`、`qq_number`、`qq_password`

### 驼峰命名法

* 当 **变量名** 是由二个或多个单词组成时，还可以利用驼峰命名法来命名
* **小驼峰式命名法**
    * 第一个单词以小写字母开始，后续单词的首字母大写
    * 例如：`firstName`、`lastName`
* **大驼峰式命名法**
    * 每一个单词的首字母都采用大写字母
    * 例如：`FirstName`、`LastName`、`CamelCase` 

![003_驼峰命名法](https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Python/009/003_驼峰命名法.jpg)

