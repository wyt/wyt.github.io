---
title: Linux安装Mysql
date: 2017-08-21 23:52:00
categories:
- 数据库
- MySQL
tags:
- Linux
- MySQL
- 数据库
---

## 安装Mysql

### 系统信息

``` bash
[root@localhost ~]# uname -a      
Linux localhost.localdomain 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@localhost ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m
```

### 查看当前系统是否已经安装Mysql

``` bash
[root@localhost ~]# yum list installed | grep mysql
mysql.x86_64            5.1.73-8.el6_8  @base   
```

### 删除自带Mysql

``` bash
[root@localhost ~]# yum -y remove mysql.x86_64
```

### 查看yum库上mysql版本

``` bash
[root@localhost ~]# yum list | grep mysql
mysql.x86_64                               5.1.73-8.el6_8                @base  
mysql-devel.x86_64                         5.1.73-8.el6_8                @base  
mysql-libs.x86_64                          5.1.73-8.el6_8                @base  
mysql-server.x86_64                        5.1.73-8.el6_8                @base  
...
```

<!--more-->

### yum安装Mysql

``` bash
[root@localhost ~]# yum -y install mysql-server mysql mysql-devel
```

### 查看安装后的Mysql信息
``` bash
[root@localhost ~]# rpm -qi mysql-server
Name        : mysql-server                 Relocations: (not relocatable)
Version     : 5.1.73                            Vendor: CentOS
Release     : 8.el6_8                       Build Date: Fri 27 Jan 2017 06:25:43 AM CST
Install Date: Mon 03 Jul 2017 12:53:49 AM CST      Build Host: c1bm.rdu2.centos.org
Group       : Applications/Databases        Source RPM: mysql-5.1.73-8.el6_8.src.rpm
Size        : 25884131                         License: GPLv2 with exceptions
Signature   : RSA/SHA1, Fri 27 Jan 2017 06:35:28 AM CST, Key ID 0946fca2c105b9de
Packager    : CentOS BuildSystem <http://bugs.centos.org>
URL         : http://www.mysql.com
Summary     : The MySQL server and related files
Description :
MySQL is a multi-user, multi-threaded SQL database server. MySQL is a
client/server implementation consisting of a server daemon (mysqld)
and many different client programs and libraries. This package contains
the MySQL server and some accompanying files and directories.
```

## 禁用linux防火墙 & seLinux

[linux下关闭【防火墙 & seLinux】](./linux下关闭防火墙%20&%20selinux.md) 


### 其他设置
####
设置Mysql服务开机启动
``` bash
[root@localhost ~]# cat /etc/rc.local    
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local
service mysqld start
```
#### 查看MySQL脚本信息

``` bash
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.1.73    |
+-----------+
1 row in set

mysql> SHOW VARIABLES LIKE "%version%";
+-------------------------+---------------------+
| Variable_name           | Value               |
+-------------------------+---------------------+
| protocol_version        | 10                  |
| version                 | 5.1.73              |
| version_comment         | Source distribution |
| version_compile_machine | x86_64              |
| version_compile_os      | redhat-linux-gnu    |
+-------------------------+---------------------+
5 rows in set

mysql> 
```
