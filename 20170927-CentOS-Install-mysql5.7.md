---
title: CentOS上安装mysql5.7
date: 2016-03-17 13:06
categories: Hexo创建博客
tags: [hexo]
description: CentOS上安装mysql5.7
---

## 1. 更新yum源到最新
```bash
shell> yum update
```

## 2. 下载mysql的yum仓库文件
```bash
shell> wget http://dev.mysql.com/get/mysql57-community-release-el6-7.noarch.rpm 
```

## 安装mysql yum repo 
```bash
shell> rpm -Uvh mysql57-community-release-el6-7.noarch.rpm
# 或： yum localinstall mysql57-community-release-el6-7.noarch.rpm

```
# 查看
```bash
shell> vim /etc/yum.repos.d/mysql-community.repo # 可以看到里面关于mysql的内容，确保mysql57的enable是打开的状态（1）
```

3. 安装mysql服务

```bash
shell> yum install mysql-community-server
```

 完成后启动服务
```bash
shell> service mysqld start


[root@BitautoCar02 hepw]# service mysqld start
Initializing MySQL database:                               [  OK  ]
Starting mysqld:                                           [  OK  ]

```

# 启动后，查看安装后自动生成的密码
```bash
shell> grep "password" /var/log/mysqld.log

## 输出
[root@BitautoCar02 hepw]# grep "password" /var/log/mysqld.log
2017-09-27T09:18:24.350884Z 1 [Note] A temporary password is generated for root@localhost: N-x1Zrrky_4e
2017-09-27T09:18:30.456198Z 3 [Note] Access denied for user 'UNKNOWN_MYSQL_USER'@'localhost' (using password: NO)

## 第一行“root@localhost:”后面的字符就是密码了，我们需要用它来设置我们自己的root登录密码
shell> mysql_secure_installation

## 输出
Securing the MySQL server deployment.
Enter password for user root: [请在这里输出自动生成的密码，如：s8)9B9)!#iWJ]
## 注：密码必须满足1)长度大于8；2)包含至少1个数字，1个大写字母，1个小写字母，1个非字母数字的符号；3)长度不得超过32位。
## 完成后还需要设置一些初始化的配置，请按照提示来做

# 以上步骤结束后，可以使用root用户登录mysql试试了

# mysql的配置文件所在目录
shell> find / -name my.cnf
## 输出
/etc/my.cnf

4. 更多配置请到my.cnf中修改
```