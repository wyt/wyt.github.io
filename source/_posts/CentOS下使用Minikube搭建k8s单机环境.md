---
title: CentOS下使用Minikube搭建k8s单机环境
date: 2022-2-21 14:20:00
categories:
- k8s
- minikube
tags:
- k8s
- minikube
---

#### 准备工作

* [linux下关闭【防火墙 & seLinux】](https://yongtao.wang/2022/02/21/linux%E4%B8%8B%E5%85%B3%E9%97%AD%E9%98%B2%E7%81%AB%E5%A2%99%20&%20selinux/)
* [CentOS 7更新到最新版本](https://yongtao.wang/2022/02/18/CentOS%207%E6%9B%B4%E6%96%B0%E5%88%B0%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC(7.9.2009)/)
* [Docker install](https://yongtao.wang/2018/10/25/Docker%20install/)
* [安装kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
* [安装Minikube](https://minikube.sigs.k8s.io/docs/start/?spm=a2c6h.12873639.0.0.ab202043NBm9C5)

注意docker需要使用非root账户运行，minikube也需要非root账户。

<!--more-->

#### 清理之前遗留的环境(新安装不需要执行)
```shell script
[wangyt@pseudo-cluster ~]$ minikube stop
* Stopping node "minikube"  ...
* Powering off "minikube" via SSH ...
* 1 node stopped.
[wangyt@pseudo-cluster ~]$ minikube delete
* Deleting "minikube" in docker ...
* Deleting container "minikube" ...
* Removing /home/wangyt/.minikube/machines/minikube ...
* Removed all traces of the "minikube" cluster.
[wangyt@pseudo-cluster ~]$ rm -rf ~/.kube/
[wangyt@pseudo-cluster ~]$ rm -rf ~/.minikube/
```

#### 创建一个minikube虚拟机
```shell script
[wangyt@pseudo-cluster ~]$ minikube start
* minikube v1.25.1 on Centos 7.9.2009
* Automatically selected the docker driver
* Starting control plane node minikube in cluster minikube
* Pulling base image ...
* Downloading Kubernetes v1.23.1 preload ...
    > preloaded-images-k8s-v16-v1...: 504.42 MiB / 504.42 MiB  100.00% 3.69 MiB
* Creating docker container (CPUs=2, Memory=2200MB) ...
! This container is having trouble accessing https://k8s.gcr.io
* To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
* Preparing Kubernetes v1.23.1 on Docker 20.10.12 ...
  - kubelet.housekeeping-interval=5m
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

#### 开启dashboard
```shell script
[wangyt@pseudo-cluster ~]$ minikube dashboard --port=8899 --url=true
* Verifying dashboard health ...
* Launching proxy ...
* Verifying proxy health ...
http://127.0.0.1:8899/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/

# 代理出去
[wangyt@pseudo-cluster ~]$ kubectl proxy --port=8899 --address='192.168.101.146' --accept-hosts='^.*' &
```

访问：

http://192.168.101.146:8899/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/

#### 查看pod
```shell script
[wangyt@pseudo-cluster ~]$ kubectl get po -A
NAMESPACE              NAME                                        READY   STATUS    RESTARTS        AGE
kube-system            coredns-64897985d-xk7qf                     1/1     Running   0               4h53m
kube-system            etcd-minikube                               1/1     Running   0               4h53m
kube-system            kube-apiserver-minikube                     1/1     Running   0               4h53m
kube-system            kube-controller-manager-minikube            1/1     Running   0               4h53m
kube-system            kube-proxy-6m7sk                            1/1     Running   0               4h53m
kube-system            kube-scheduler-minikube                     1/1     Running   0               4h53m
kube-system            storage-provisioner                         1/1     Running   1 (4h53m ago)   4h53m
kubernetes-dashboard   dashboard-metrics-scraper-58549894f-krr2c   1/1     Running   0               4h39m
kubernetes-dashboard   kubernetes-dashboard-ccd587f44-84sgj        1/1     Running   0               4h39m
[wangyt@pseudo-cluster ~]$ kubectl get pods --all-namespaces
NAMESPACE              NAME                                        READY   STATUS    RESTARTS        AGE
kube-system            coredns-64897985d-xk7qf                     1/1     Running   0               4h54m
kube-system            etcd-minikube                               1/1     Running   0               4h54m
kube-system            kube-apiserver-minikube                     1/1     Running   0               4h54m
kube-system            kube-controller-manager-minikube            1/1     Running   0               4h54m
kube-system            kube-proxy-6m7sk                            1/1     Running   0               4h54m
kube-system            kube-scheduler-minikube                     1/1     Running   0               4h54m
kube-system            storage-provisioner                         1/1     Running   1 (4h53m ago)   4h54m
kubernetes-dashboard   dashboard-metrics-scraper-58549894f-krr2c   1/1     Running   0               4h40m
kubernetes-dashboard   kubernetes-dashboard-ccd587f44-84sgj        1/1     Running   0               4h40m
```
