---
title: Helm入门
date: 2022-05-02 13:23:00
updated: 2022-05-02 13:23:00
comments: true
categories:
- helm
tags:
- helm
- k8s
---

Helm是k8s的包管理器，我们可以使用Helm来安装部署应用；一个Helm包被称作Chart，存储和发布Chart的仓库叫做Repository。

Helm 3 不再使用服务端。

#### 安装Helm

选择合适的安装包

```bash
[wangyt@pseudo-cluster tmp]$ wget https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz
--2022-05-02 21:37:09--  https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz
Resolving get.helm.sh (get.helm.sh)... 152.199.39.108, 2606:2800:247:1cb7:261b:1f9c:2074:3c
Connecting to get.helm.sh (get.helm.sh)|152.199.39.108|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13633605 (13M) [application/x-tar]
Saving to: ‘helm-v3.8.2-linux-amd64.tar.gz’

100%[==================================================================================================================================================>] 13,633,605  10.5MB/s   in 1.2s   

2022-05-02 21:37:11 (10.5 MB/s) - ‘helm-v3.8.2-linux-amd64.tar.gz’ saved [13633605/13633605]

[wangyt@pseudo-cluster tmp]$ 
[wangyt@pseudo-cluster tmp]$ tar -zxvf helm-v3.8.2-linux-amd64.tar.gz 
linux-amd64/
linux-amd64/helm
linux-amd64/LICENSE
linux-amd64/README.md
[wangyt@pseudo-cluster tmp]$ ls -lh
drwxr-xr-x 2 wangyt wangyt   50 Apr 14 01:56 linux-amd64
[wangyt@pseudo-cluster tmp]$ 
[wangyt@pseudo-cluster tmp]$ sudo mv linux-amd64/helm /usr/local/bin/helm
[wangyt@pseudo-cluster tmp]$ helm help
The Kubernetes package manager
```

#### 基本命令

```shell script
## 添加仓库
[wangyt@localhost ~]$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

## 列出仓库
[wangyt@localhost ~]$ helm repo list
NAME            URL                                                
skywalking      https://apache.jfrog.io/artifactory/skywalking-helm
bitnami         https://charts.bitnami.com/bitnami                 

## 搜索仓库
[wangyt@localhost ~]$ helm search repo bitnami
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/bitnami-common                          0.0.9           0.0.9           DEPRECATED Chart with custom templates used in ...
bitnami/airflow                                 12.2.1          2.2.5           Apache Airflow is a tool to express and execute...
bitnami/apache                                  9.0.17          2.4.53          Apache HTTP Server is an open-source HTTP serve...
bitnami/argo-cd                                 3.1.16          2.3.3           Argo CD is a continuous delivery tool for Kuber...
bitnami/argo-workflows                          1.1.12          3.3.4           Argo Workflows is meant to orchestrate Kubernet...
...
```

#### 安装Chart示例

```shell script
[wangyt@localhost ~]$ helm repo update  # 确定我们可以拿到最新的charts列表
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "skywalking" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈


```

[安装Helm](https://helm.sh/zh/docs/intro/install/)
[快速入门指南](https://helm.sh/zh/docs/intro/quickstart/)
