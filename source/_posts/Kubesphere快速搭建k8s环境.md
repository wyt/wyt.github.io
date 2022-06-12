---
title: Kubesphere快速搭建k8s环境
date: 2022-05-04 10:20:00
updated: 2022-05-04 10:20:00
comments: true
categories:
- kubesphere
tags:
- k8s
- kubesphere
---

#### 准备linux机器

CentOS 7.x	2 核 CPU，4 GB 内存，40 GB 磁盘空间

#### 容器运行时

Docker	19.3.8 +

提前安装好

#### 依赖项要求

需要安装`socat` & `conntrack`

```shell script
yum install -y socat
yum install -y conntrack
```

#### 网络和 DNS 要求

* 请确保 /etc/resolv.conf 中的 DNS 地址可用，否则，可能会导致集群中的 DNS 出现问题

<!--more-->

#### 下载kubekey

```shell script
export KKZONE=cn
curl -sfL https://get-kk.kubesphere.io | VERSION=v2.0.0 sh -
chmod +x kk
```

#### 安装

安装等待过程可能会很慢

```shell script
 _   __      _          _   __           
| | / /     | |        | | / /           
| |/ / _   _| |__   ___| |/ /  ___ _   _ 
|    \| | | | '_ \ / _ \    \ / _ \ | | |
| |\  \ |_| | |_) |  __/ |\  \  __/ |_| |
\_| \_/\__,_|_.__/ \___\_| \_/\___|\__, |
                                    __/ |
                                   |___/

02:01:26 CST [NodePreCheckModule] A pre-check on nodes
02:01:28 CST success: [localhost.localdomain]
02:01:28 CST [ConfirmModule] Display confirmation form
+-----------------------+------+------+---------+----------+-------+-------+-----------+--------+----------+------------+-------------+------------------+--------------+
| name                  | sudo | curl | openssl | ebtables | socat | ipset | conntrack | chrony | docker   | nfs client | ceph client | glusterfs client | time         |
+-----------------------+------+------+---------+----------+-------+-------+-----------+--------+----------+------------+-------------+------------------+--------------+
| localhost.localdomain | y    | y    | y       | y        | y     | y     | y         |        | 20.10.14 |            |             |                  | CST 02:01:28 |
+-----------------------+------+------+---------+----------+-------+-------+-----------+--------+----------+------------+-------------+------------------+--------------+

## 检查完ok没问题输入yes确认安装

[wangyt@localhost ~]$ sudo ./kk create cluster --with-kubernetes v1.21.5 --with-kubesphere v3.2.1


...

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

...

#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://192.168.101.120:30880
Account: admin
Password: P@88w0rd

NOTES：
  1. After you log into the console, please check the
     monitoring status of service components in
     "Cluster Management". If any service is not
     ready, please wait patiently until all components 
     are up and running.
  2. Please change the default password after login.

#####################################################
https://kubesphere.io             2022-05-03 02:20:17
#####################################################
02:20:19 CST success: [localhost.localdomain]
02:20:19 CST Pipeline[CreateClusterPipeline] execute successful
Installation is complete.

Please check the result using the command:

        kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

安装完成后，如果`kubectl`命令找不到，需要执行下面命令：

```shell script
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

[在 Linux 上以 All-in-One 模式安装 KubeSphere](https://kubesphere.io/zh/docs/quick-start/all-in-one-on-linux/)
