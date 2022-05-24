---
title: CentOS7部署Python服务
date: 2021-08-30 15:44:16
categories: CentOS7
tags:
  - Linux
  - CentOS7
  - Python
description: CentOS7部署Python服务
copyright_author: Monster
keywords: Python
cover: https://raw.githubusercontent.com/MyMonsterCat/md_image/main/%E5%9F%BA%E7%A1%80/Blog_Cover/15.jpg
---

# 安装python3环境

由于CentOS7上默认装有2.x版本的python，所以需要装python3，此处选择的版本为`python3.6.2`

```shell
# 查看自己的操作系统
cat /etc/redhat-release
# 查看python版本号
python
# 安装编译环境
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
# 手动下载python源码，也可以选择wget下载
wget https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tar.xz
# 解压
tar -xvJf  Python-3.6.2.tar.xz //解压
cd Python-3.6.2
# 编译到/usr/local/python3
./configure prefix=/usr/local/python3 
#安装
make && make install 
#建立软连接
ln -s /usr/local/python3/bin/python3 /usr/bin/python3 //创建Python3软链接
python3 -V //查看Python3的版本
```

make不成功：

https://www.huaweicloud.com/articles/4542b9f873fca76693ff717d230cb619.html

设置清华镜像（没有pip.conf文件就创建）

https://www.runoob.com/w3cnote/pip-cn-mirror.html

# 安装pip3

```shell
#安装setuptools
wget --no-check-certificate https://pypi.python.org/packages/source/s/setuptools/setuptools-19.6.tar.gz#md5=c607dd118eae682c44ed146367a17e26
# 解压
tar -zxvf setuptools-19.6.tar.gz
# 编译,安装
cd setuptools-19.6/
python3 setup.py build
python3 setup.py install
# 建立软链
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
# 查看pip3版本
pip3 -V
# 更新
pip3 install --upgrade pip
```

# 生成清单文件

通过pip3 freeze命令把依赖的包重定向到requirement.txt中

```shell
pip3 freeze > requirements.txt
```

将requirements.txt文件里的包全部安装

```shell
pip install –r requirements.txt
```

  项目所需如下

```shell
tensorflow==1.14.0
Keras==2.3.1
Flask==1.1.2
matplotlib==3.3.4
pandas==1.1.5
pymysql
sklearn
h5py==2.10.0
```

# 创建虚拟环境

切换到pip3所在的目录，执行命令

```shell
#安装虚拟环境
pip3 install virtualenv

#先按照这个教程
https://www.cnblogs.com/hszstudypy/p/11511228.html
 
# 创建python3.6版本的虚拟环境 venv
virtualenv venv --python=python3.6
 
# 切换到虚拟环境所在的目录
cd venv
 
# 启用虚拟环境
source ./bin/activate 【1、退出虚拟环境:deactivate 2、删除虚拟环境:rm -r venv】
 
# 安装依赖清单里的库
pip3 install -r requirements.txt
 
# 列出当前虚拟环境所安装的依赖库
pip3 list
```

安装虚拟环境可能取到的问题

https://www.codeleading.com/article/46581599676/

操作虚拟环境

https://zhuanlan.zhihu.com/p/60647332

# 启动服务

```shell
# 在后台启动xxx.py
python xxx.py &
```

**运行可能遇到的问题1：**

https://stackoverflow.com/questions/12806122/missing-python-bz2-module

**运行可能遇到的问题2：**

这步看情况采用：需要nohup来保证python程序能够在后台运行

```shell
nohup python test.py &  在后台运行test.py
jobs 查看后台运行的进程
fg %n 让后台进程n转到前台
bg %n 让暂停运行的后台进程n继续运行
kill %n 杀死job 
ctrl+z 使前台正在运行的进程转到后台
ctrl+c 终止前台进程

具体用法
nohup python -u test.py > test.log 2>&1 &
# 2 输出错误信息到提示符窗口
# 1 表示输出信息到提示符窗口, 1前面的&注意添加, 否则还会创建一个名为1的文件
# 最后会把日志文件输出到test.log文件

tail -f test.log # 实时输出
cat test.log # 全部输出
```

参考文章：

https://blog.csdn.net/qq_36441027/article/details/111182378

https://www.pythonf.cn/read/174790

https://blog.csdn.net/TLuffy/article/details/111577429?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-5.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-5.control

python实现接口服务：

https://blog.csdn.net/songlh1234/article/details/83381642





# nacos

https://www.jianshu.com/p/2e065c15d730

https://cdmana.com/2021/03/20210330203555303b.html

问题https://zhuanlan.zhihu.com/p/335362918

