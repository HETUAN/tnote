---
title: CentOS上安装Git
date: 2016-03-17 13:06
categories: Hexo创建博客
tags: [hexo]
description: CentOS上安装Git客户端
---

## 确保已安装了依赖的包
``` bash
yum install curl 
yum install curl-devel 
yum install zlib-devel 
yum install openssl-devel 
yum install perl 
yum install cpio 
yum install expat-devel 
yum install gettext-devel
```
这样也可以
``` bash
yum install curl curl-devel zlib-devel openssl-devel perl cpio expat-devel gettext-devel
```

## 下载最新的git包
``` bash
wget https://Github.com/Git/Git/archive/v2.3.0.tar.gz
``` 
## 解压
``` bash
tar xzvf v2.3.0.tar.gz
``` 
## 进入目录
``` bash
cd v2.3.0 -你的目录可能不是这个 
``` 
## 安装
``` bash
autoconf 
./configure 
make 
sudo make install
``` 
## 检查下安装的版本，大功告成
``` bash
git --version
``` 
### 上面是熟练的人装逼用的
### 入门者请使用下面的而方法
### 安装Git - 使用软件包管理器
在CentOS安装Git的最简单的方法是使用yum包管理。以下命令将帮助您安装的Git软件。
``` bash
sudo yum install Git
``` 