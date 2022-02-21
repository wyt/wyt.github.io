---
title: Docker install
date: 2018-10-25 13:29:00
updated: 2022-2-21 11:24:00
comments: true
categories:
- docker
tags:
- docker
---

### Uninstall old versions

``` bash
[root@docker_007 ~]# yum remove docker \
  docker-client \
  docker-client-latest \
  docker-common \
  docker-latest \
  docker-latest-logrotate \
  docker-logrotate \
  docker-selinux \
  docker-engine-selinux \
  docker-engine
```

<!--more-->

### Install Docker CE

#### Install required packages

``` bash
[root@docker_007 ~]# yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

#### Use the following command to set up the stable repository

``` bash
# 注意设置阿里云repo
[root@docker_007 ~]# yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@docker_007 ~]# yum makecache fast
```

#### Install Docker CE

``` bash
[root@docker_007 ~]# yum install docker-ce
[root@docker_007 ~]# docker -v
Docker version 18.06.1-ce, build e68fc7a
```

##### 安装指定版本

``` bash
[root@dlink-72 docker]# yum list docker-ce --showduplicates | sort -r
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
 * updates: mirrors.huaweicloud.com
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
Installed Packages
 * extras: mirror.jdcloud.com
docker-ce.x86_64            3:18.09.0-3.el7                    docker-ce-stable 
docker-ce.x86_64            18.06.1.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.06.1.ce-3.el7                   @docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.12.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.09.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.09.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.2.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.3.ce-1.el7                   docker-ce-stable 
docker-ce.x86_64            17.03.2.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable 
 * base: mirrors.tuna.tsinghua.edu.cn
Available Packages
[root@dlink-72 docker]# yum install docker-ce-18.06.1.ce-3.el7
```

#### 镜像加速器

使用阿里云Docker镜像加速器，进入阿里云控制台查看配置。https://cr.console.aliyun.com/cn-hangzhou/mirrors

``` bash
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxxx.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

#### Start docker

``` bash
# 启动docker服务，并保持开机自启动
[root@docker_007 ~]# systemctl start docker
[root@docker_007 ~]# systemctl enable docker
```

#### 以非root用户启动运行docker(可选)

docker安装完之后，可以使用非root用户启动运行

```shell script
[wangyt@docker_007 ~]# sudo groupadd docker
[wangyt@docker_007 ~]# sudo usermod -aG docker $USER
[wangyt@docker_007 ~]# sudo groupadd docker
[wangyt@docker_007 ~]# newgrp docker 
```

执行完上述命令(虚拟机下可能需要重启)，重启docker：

```shell script
[wangyt@docker_007 ~]# sudo systemctl restart docker
[wangyt@docker_007 ~]# sudo systemctl enable docker
```

#### 远程访问

参考[How do I enable the remote API for dockerd](https://success.docker.com/article/how-do-i-enable-the-remote-api-for-dockerd)

注意关闭防火墙

``` bash
# 查看防火墙状态
firewall-cmd --state
# 停止firewall
systemctl stop firewalld.service
# 禁止firewall开机启动
systemctl disable firewalld.service
```

#### 查看docker服务日志

``` bash
[root@docker_007 ~]# journalctl -ru docker.service
#可以指定日期范围
[root@docker_007 ~]# journalctl -ru docker.service --since="2019-08-25 16:20:00" --until="2019-08-25 16:30:00"
```

#### 其他组件

docker compose
https://docs.docker.com/compose/install/

docker machine
https://docs.docker.com/machine/install-machine/

[Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
[Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/)
