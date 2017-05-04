---
title: CentOS 防火墙相关
date: 2016-03-21 21:50
categories: CentOS 
tags: [CentOS]
description: CentOS 防火墙相关使用技巧
---
#### 1. 安装iptables防火墙
``` bash
yum install iptables-services #安装
``` 
#### 2. 启动/重启防火墙 | 查看防火墙状态 
``` bash 
service iptables start 
service iptables restart
service iptables status
``` 
#### 3. 修改配置文件
``` bash
vi /etc/sysconfig/iptables

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
-A INPUT -j REJECT --reject-with icmp-host-prohibited  
-A FORWARD -j REJECT --reject-with icmp-host-prohibited  
COMMIT  
``` 
#### 4. 开启例外端口则，增加如下配置：
``` bash
vi /etc/sysconfig/iptables

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
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT  
-A INPUT -j REJECT --reject-with icmp-host-prohibited  
-A FORWARD -j REJECT --reject-with icmp-host-prohibited  
COMMIT  
``` 
#### 5.  关闭自动启动
``` bash
chkconfig iptables off
``` 
#### 6.  查看状态
``` bash
chkconfig --list iptables
netstat
``` 
#### 7. 打开/关闭端口
``` bash
/sbin/iptables -I INPUT -p tcp --dport 4000 -j ACCEPT   #写入修改
/sbin/iptables -I INPUT -p tcp --dport 80 -j DROP     #写入修改
/etc/init.d/iptables save   #保存修改
service iptables restart    #重启防火墙，修改生效
# 或者修改配置文件
``` 
 