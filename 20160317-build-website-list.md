---
title: HEXO+Github 搭建博客
date: 2016-03-17 09:06
categories: Hexo创建博客
tags: [hexo]
description: 通过Hexo创建该博客的目录
---
## 配置环境
### 安装Node（必须）
作用：用来生成静态页面的
到Node.js官网下载相应平台的最新版本，一路安装即可。
安装Git（必须）
作用：把本地的hexo内容提交到github上去.
安装Xcode就自带有Git，我就不多说了。
申请GitHub（必须）
作用：是用来做博客的远程创库、域名、服务器之类的，怎么与本地hexo建立连接等下讲。
github账号我也不再啰嗦了,没有的话直接申请就行了，跟一般的注册账号差不多，SSH Keys，看你自己了，可以不配制，不配置的话以后每次对自己的博客有改动提交的时候就要手动输入账号密码，配置了就不需要了，怎么配置我就不多说了，网上有很多教程。

## 正式安装HEXO
### Node和Git都安装好后，可执行如下命令安装hexo：

``` bash
npm install -g hexo
``` 
* npm源的问题，可以尝试更换taobao的npm源* 
``` bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install hexo-cli -g
``` 
* 这样就可以了* 

### 初始化
然后，执行init命令初始化hexo到你指定的目录，我是直接cd到目标目录执行hexo init的。
命令：
``` bash
hexo init
``` 

好啦，至此，全部安装工作已经完成！
# 生成静态页面
cd 到你的init目录，执行如下命令，生成静态页面至hexo\\public\\目录。
``` bash
hexo generate （hexo g  也可以）
``` 
