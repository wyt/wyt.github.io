---
title: CentOS 7更新到最新版本(7.9.2009)
date: 2022-2-18 14:34:20
tags:
- CentOS
- yum
---

### 使用阿里云镜像替换CentOS默认yum源
```shell script
[root@localhost ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
[root@localhost ~]# sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
[root@localhost ~]# yum makecache
```

<!--more-->

### 查看更新前版本
```shell script
[root@localhost ~]# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 
```

### 执行更新

重要数据需要做好备份。

```shell script
[root@localhost ~]# yum check-update
[root@localhost ~]# yum clean all
[root@localhost ~]# reboot
[root@localhost ~]# yum update -y
```

### 查看更新后版本
```shell script
[root@localhost ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
```

参考资料：
1.[CentOS 镜像](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.49fe1b11CNfeGD)
