---
title: python学习笔记（一）
catalog: true
date: 2018-11-19 23:05:19
subtitle: Python 基础学习
header-img: 7vLMRNQ.jpg
tags: Python
---

# python的安装

因为我是用的mac开发，所以我这里只说一下mac是如何安装的

方法一：从Python官网下载Python 3.7的安装程序（网速慢的同学请移步国内镜像），双击运行并安装；

`https://www.python.org/ftp/python/3.7.0/python-3.7.0-macosx10.9.pkg`

方法二：如果安装了Homebrew。

```shell
brew install python3
```

在这里我需要提一下python的包管理工具`pip`

如果有homebrew工具的话可以使用

```shell
brew install pip3
```

以安装requests为例，命令：

```shell
pip3 install requests
```

# python 基本语法

头部需要引入

```python
#!/usr/bin/env python
# coding:utf-8

# 输出hello-world

print('hello, world')
```

## python 数据类型

### 整数

Python可以处理任意大小的整数，当然包括负整数，在程序中的表示方法和数学上的写法一模一样，例如：1，100，-8080，0，等等。

计算机由于使用二进制，所以，有时候用十六进制表示整数比较方便，十六进制用0x前缀和0-9，a-f表示，例如：0xff00，0xa5b4c3d2，等等。

### 浮点数

浮点数也就是小数，之所以称为浮点数，是因为按照科学记数法表示时，一个浮点数的小数点位置是可变的，比如，1.23x109和12.3x108是完全相等的。浮点数可以用数学写法，如1.23，3.14，-9.01，等等。但是对于很大或很小的浮点数，就必须用科学计数法表示，把10用e替代，1.23x109就是1.23e9，或者12.3e8，0.000012可以写成1.2e-5，等等。

整数和浮点数在计算机内部存储的方式是不同的，整数运算永远是精确的（除法难道也是精确的？是的！），而浮点数运算则可能会有四舍五入的误差。

### 字符串

字符串是以单引号'或双引号"括起来的任意文本，比如'abc'，"xyz"等等。
