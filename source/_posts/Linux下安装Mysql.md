---
title: CentOS 7安装Mysql5.7 (yum方式)
date: 2022/06/18 18:40:00
categories:
- 数据库
- MySQL
tags:
- Linux
- MySQL
- 数据库
---

## 禁用linux防火墙 & seLinux

[linux下关闭【防火墙 & seLinux】](./linux下关闭防火墙%20&%20selinux.md) 

## 安装Mysql

### 系统信息

``` bash
[root@pseudo-cluster ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
[root@pseudo-cluster ~]#
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

### 删除mariadb

如果系统安装了mariadb，需要移除

```bash
[root@localhost ~]# yum list installed |grep mariadb
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
wget https://repo.mysql.com//mysql80-community-release-el7-6.noarch.rpm
yum localinstall yum localinstall mysql80-community-release-el7-6.noarch.rpm 
yum repolist enabled | grep "mysql.*-community.*"
yum repolist all | grep mysql
yum repolist enabled | grep mysql
yum-config-manager --disable mysql80-community
yum repolist enabled | grep mysql
yum repolist all | grep mysql
yum-config-manager --enable mysql57-community
yum repolist all | grep mysql
yum repolist enabled | grep mysql
yum install mysql-community-server
yum --enablerepo=mysql57-community clean metadata
yum install mysql-community-server
yum -y install mysql-community-server
service mysqld start
grep 'temporary password' /var/log/mysqld.log
vim /var/log/mysqld.log 
grep 'temporary password' /var/log/mysqld.log
grep 'password' /var/log/mysqld.log
service mysqld restart
grep 'password' /var/log/mysqld.log
mysql -uroot -p
ps -elf | grep mysqld
cat /var/run/mysqld/mysql.pid
cat /var/run/mysqld/mysqld.pid
cd /var/lib/mysql
mysql -uroot -p
service mysqld status
mysql -uroot
mysql -uroot -p
yum list installed | grep mysql
```

### 查看安装后的Mysql信息
``` bash
[wangyt@pseudo-cluster ~]$ yum list installed | grep  mysql
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
mysql-community-client.x86_64        5.7.38-1.el7             @mysql57-community
mysql-community-common.x86_64        5.7.38-1.el7             @mysql57-community
mysql-community-libs.x86_64          5.7.38-1.el7             @mysql57-community
mysql-community-server.x86_64        5.7.38-1.el7             @mysql57-community
mysql80-community-release.noarch     el7-6                    @/mysql80-community-release-el7-6.noarch
```

## 授权

```shell script
mysql -u root -p

mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;

GRANT ALL PRIVILEGES ON *.* to root@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
flush privileges;
```


## 参考

[2.5.1 Installing MySQL on Linux Using the MySQL Yum Repository](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html)
