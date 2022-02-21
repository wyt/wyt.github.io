---
title: linux下关闭【防火墙 & seLinux】
date: 2022-02-21 11:26:00
categories:
- linux
---

### 禁用seLinux

#### 查看状态

```shell script
[root@localhost ~]#  sestatus
```

#### 暂时关闭

暂时关闭，重启失效

```shell script
[root@localhost ~]#  setenforce 0
```

<!--more-->

#### 永久关闭

设置`SELINUX=disabled`，需要重启。

``` bash
[root@localhost ~]# cat /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 
```

### 关闭防火墙

#### CentOS 7

#### 查看状态

```shell script
[root@localhost ~]# firewall-cmd --state
```
或者
```shell script
[root@localhost ~]# systemctl status firewalld
```

##### 暂时关闭

```shell script
[root@localhost ~]# systemctl stop firewalld.service
```

##### 永久关闭

```shell script
# 禁止firewall开机启动
[root@localhost ~]# systemctl disable firewalld.service
```

#### CentOS 6

##### 查看状态

```shell script
[root@localhost ~]# service iptables status
```

##### 暂时关闭

``` bash
[root@localhost ~]# service iptables stop
```
##### 永久关闭

``` bash
[root@localhost ~]# chkconfig iptables off
```
