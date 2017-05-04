---
title: CentOS 使用技巧
date: 2016-03-17 13:06
categories: CentOS
tags: [hexo]
description: CentOS 使用技巧
---
  
## CentOS中由一般用户切换为root用户
1. 打开终端，提示符为“$”，表明该用户为普通用户，此时，直接输su，回车，输入root密码，回车，就可以切换到root用户下，此时的提示符变为“#”。
  `注意，输入密码时终端是不显示的，而且每次切换为root用户都要经过这个过程。`
2. 切换回普通用户，只要输入 “su 用户名”就OK了。

## 开启端口
``` bash
/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
/sbin/iptables -I INPUT -p tcp --dport 22 -j ACCEPT
/sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
```
然后保存：
``` bash
 /etc/rc.d/init.d/iptables save
```

查看打开的端口：
``` bash
/etc/init.d/iptables status
``` 

补充说明：

## 关闭防火墙
``` bash
/etc/init.d/iptables stop
service iptables stop # 停止服务
``` 
## 查看防火墙信息
``` bash
/etc/init.d/iptables status
``` 
## 开放端口:8080
``` bash
/sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
``` 
## 重启防火墙以便改动生效:(或者直接重启系统)
``` bash
/etc/init.d/iptables restart
``` 
## 将更改进行保存
``` bash
/etc/rc.d/init.d/iptables save
``` 
另外直接在/etc/sysconfig/iptables中增加一行：
``` bash
-A RH-Firewall-1-INPUT -m state –state NEW -m tcp -p tcp –dport 8080 -j ACCEPT
``` 
## 永久关闭防火墙
``` bash
chkconfig –level 35 iptables off #此方法源自网络，未实验，安全考虑拒绝使用此方法
``` 
## 一、配置防火墙，开启80端口、3306端口
CentOS 7.0默认使用的是firewall作为防火墙，这里改为iptables防火墙。
1. 关闭firewall：
``` bash
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
``` 
2. 安装iptables防火墙
``` bash
yum install iptables-services #安装
vi /etc/sysconfig/iptables #编辑防火墙配置文件
# Firewall configuration written by system-config-firewall
# Manual customization of this file is not recommended.
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
:wq! #保存退出
systemctl restart iptables.service #最后重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动
``` 