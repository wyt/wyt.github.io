---
title: Windows下noinstallZIP方式配置MySQL5.7
date: 2017-11-04 21:57:00
categories:
- 数据库
- MySQL
tags:
- Windows
- MySQL
- 数据库
---


参考: https://dev.mysql.com/doc/refman/5.7/en/windows-install-archive.html


### 下载MySQL

https://dev.mysql.com/downloads/mysql/ 选择 Windows (x86, 64-bit), ZIP Archive

> MySQL Community 5.7 Server requires the Microsoft Visual C++ 2013 Redistributable Package to run on Windows platforms.

MySQL 5.7版本需要安装Microsoft Visual C++ 2013 Redistributable Package

下载安装: https://www.microsoft.com/en-us/download/details.aspx?id=40784

<!--more-->

### 解压安装

#### 1. 将下载的MySQL压缩包解压
如`D:\dev_app\DB\mysql-5.7.20-winx64`目录下，

将`D:\dev_app\DB\mysql-5.7.20-winx64\bin` 加入到系统环境变量中

同时创建数据存储目录如：`D:\dev_app\DB\mysql-5.7.20-winx64\data`

#### 2. 创建配置mysql.ini文件

> As of MySQL 5.7.18, my-default.ini is no longer included in or installed by distribution packages.

MySQL 5.7.18之后，my-default.ini不在包含在分发包中，我们在`D:\dev_app\DB\mysql-5.7.20-winx64`目录下自己创建一个my.ini

内容如下:

```bash
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html
# *** DO NOT EDIT THIS FILE. It's a template which will be copied to the
# *** default location during install, and will be replaced if you
# *** upgrade to a newer version of MySQL.

[mysqld]

# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M

# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin

# These are commonly set, remove the # and set as required.
basedir=D:\dev_app\DB\mysql-5.7.20-winx64
datadir=D:\\dev_app\\B\\mysql-5.7.20-winx64\\data
# port = .....
# server_id = .....


# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M 

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
```

### 初始化数据目录

管理员权限启动CMD, 使用mysqld初始化目录:
```bash
C:\Users\Administrator>mysqld --defaults-file=D:\dev_app\DB\mysql-5.7.20-winx64\my.ini --initialize
```
data目录下会生成一个.err扩展名的文件，打开它，里面会记录生成的MySQL临时密码：
```bash
2017-11-03T11:53:18.008877Z 1 [Note] A temporary password is generated for root@localhost: xd#pZirf&2Gk
```

执行目录初始化主要完成以下功能：

> * 检查数据目录是否存在
> * 创建mysql数据库及系统表等
> * 初始化系统表空间等
> * 服务器创建'root'@'localhost' 超级用户帐户和其他保留帐户
> * 其他

### 启动MySQL
```bash
# 可以不带console参数，日志写到文件中，不在屏幕上输出
C:\Users\Administrator>mysqld --console
```
启动成功
```bash
2017-11-03T12:06:51.583410Z 0 [Note] .\bin\mysqld.exe: ready for connections.
Version: '5.7.20'  socket: ''  port: 3306  MySQL Community Server (GPL)
2017-11-03T12:06:51.583410Z 0 [Note] Executing 'SELECT * FROM INFORMATION_SCHEMA
.TABLES;' to get a list of tables using the deprecated partition engine. You may
 use the startup option '--disable-partition-engine-check' to skip this check.
2017-11-03T12:06:51.584410Z 0 [Note] Beginning of list of non-natively partition
ed tables
2017-11-03T12:06:51.602411Z 0 [Note] End of list of non-natively partitioned tab
les
```
### 连接到MySQL
```bash
C:\Users\Administrator>mysql -u root -p
Enter password: ************
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.20

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
#### 分配一个新的root密码
```bash
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.00 sec)
```
### 将MySQL作为windows服务

确保mysql服务停止

```bash
C:\Users\Administrator>mysqladmin -u root -p shutdown
Enter password: ******

# another termianl
...
2017-11-04T13:45:23.331094Z 0 [Note] InnoDB: Buffer pool(s) dump completed at 171104 21:45:23
2017-11-04T13:45:24.331199Z 0 [Note] InnoDB: Shutdown completed; log sequence number 2565405
2017-11-04T13:45:24.347199Z 0 [Note] InnoDB: Removed temporary tablespace data file: "ibtmp1"
2017-11-04T13:45:24.347199Z 0 [Note] Shutting down plugin 'MEMORY'
2017-11-04T13:45:24.347199Z 0 [Note] Shutting down plugin 'CSV'
2017-11-04T13:45:24.347199Z 0 [Note] Shutting down plugin 'sha256_password'
2017-11-04T13:45:24.347199Z 0 [Note] Shutting down plugin 'mysql_native_password'
2017-11-04T13:45:24.347199Z 0 [Note] Shutting down plugin 'binlog'
2017-11-04T13:45:24.347199Z 0 [Note] mysqld: Shutdown complete
```

将MySQL Server作为Windows Service自动启动

```bash
C:\Users\Administrator>mysqld --install
Service successfully installed.
```

移除MySQL服务
```bash
C:\Users\Administrator>mysqld --remove
```