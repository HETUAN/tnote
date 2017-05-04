---
title: 在Ubuntu上使用 nohup 命令后台运行程序
date: 2017-02-28 17:05:01
categories: nohup
tags: [nohup]
description: nohup
---
# 在Ubuntu上使用 nohup 命令后台运行程序

## 情景：
使用.net core 发布了一个控制台程序（程序是每隔12个小时运行一次，从链家地图看房中获取数据并保存到数据库），想让程序在ubuntu 14上后台运行。

## 启动命令：

nohup [命令内容] &
命令内容可以是直接运行命令
``` bash
dotnet /home/ubunut/dotnet/Release/Bruce.Core.Test.LianJiaConsole.dll
```
如果文件太多可以将文件放到脚本中
例如创建lianjia.sh
将上面的命令写入脚本
那么命令内容： ./lianjia.sh
nohup ./lianjia.h &

> 在结尾加上"&"来将命令同时放入后台运行

## 查看运行的后台进程
jobs   只能查看当前窗口的后台进程，如果关闭了后台执行脚本的窗口，该命令失效（后台进程不会失效），这个时候就只能用到下面的命令查看
ps -ef    可以查看主机所有运行的进程   ps -ef|grep 过滤条件

## fg %n将后台脚本调到前台执行
fg %n  将当前后台运行程序调到前台执行n是线程ID

## 终止后台运行的删除脚本
删除命令：kill -9  进程号
