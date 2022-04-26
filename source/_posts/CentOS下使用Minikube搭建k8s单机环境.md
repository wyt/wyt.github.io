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
* [CentOS 7更新到最新版本](https://yongtao.wang/2022/02/18/CentOS%207%E6%9B%B4%E6%96%B0%E5%88%B0%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC/)
* [Docker install](https://yongtao.wang/2018/10/25/Docker%20install/)
* [安装kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
* [安装Minikube](https://minikube.sigs.k8s.io/docs/start/?spm=a2c6h.12873639.0.0.ab202043NBm9C5)

注意docker需要使用非root账户运行，minikube也需要非root账户。

<!--more-->

#### 清理环境

之前安装过的话，需要清理下环境，新安装的不需要执行。

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

##### 查看集群信息
```shell script
[wangyt@pseudo-cluster ~]$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
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

#### 查看Pod
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

#### Pod(容器组)

我们以tomcat为例，使用k8s来部署服务。

Pod是Kubernetes中的容器组，一个Pod可以包含一个或多个容器（Docker）。我们可以用一个YAML文件来描述如何建立一个Pod。

`tomcat-pod.yaml`：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tomcat
  labels:
    app: mytomcat
spec:
  containers:
  - name: tomcat
    image: tomcat:8
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 8080
```

使用`kubectl`部署Pod：

```shell script
[wangyt@pseudo-cluster tmp]$ kubectl create -f tomcat-pod.yaml
pod/tomcat created
```

查看Pod：

```shell script
[wangyt@pseudo-cluster tmp]$ kubectl get pods -A
NAMESPACE              NAME                                        READY   STATUS              RESTARTS   AGE
default                tomcat                                      0/1     ContainerCreating   0          9s
```

查看Pod详情：

```shell script
[wangyt@pseudo-cluster ~]$ kubectl describe pod/tomcat
Name:         tomcat
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Wed, 13 Apr 2022 04:01:32 +0800
Labels:       app=mytomcat
Annotations:  <none>
Status:       Running
IP:           172.17.0.5
IPs:
  IP:  172.17.0.5
Containers:
  tomcat:
    Container ID:   docker://3cbeae4ab93d5ee0c803391c3435e905efa7dc9ccb305b8ac299c87cfe7e3ccc
    Image:          tomcat:8
    Image ID:       docker-pullable://tomcat@sha256:8d7231354284be82ddc4567b209f4fea3a6f42f4f9626005e793953fc0a2d066
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 13 Apr 2022 04:01:33 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        500m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zp66r (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-zp66r:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

进入Pod容器：

```shell script
[wangyt@pseudo-cluster ~]$ kubectl exec -it tomcat -- /bin/bash
root@tomcat:/usr/local/tomcat# ps -elf | grep java
4 S root           1       0  0  80   0 - 845824 futex_ 20:01 ?       00:00:28 /usr/local/openjdk-11/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
0 S root         122     112  0  80   0 -  1264 pipe_w 21:21 pts/0    00:00:00 grep java
root@tomcat:/usr/local/tomcat#
```

#### Replica Set(副本集)

Replica Set控制Pod副本的运行数量。

`tomcat-rs.yaml`：

```yaml
apiVersion: apps/v1
kind: ReplicaSet # 类型为ReplicaSet
metadata:
  name: tomcat-rs
  labels:
    app: mytomcat
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mytomcat # 匹配Pod的元数据，将此副本集应用到该Pod
  template:
    metadata:
      labels:
        app: mytomcat
    spec:
      containers:
        - name: tomcat
          image: tomcat:8
```

部署ReplicaSet：

```shell script
[wangyt@pseudo-cluster basic]$ kubectl create -f tomcat-rs.yaml 
replicaset.apps/tomcat-rs created
```

查看rs：

```shell script
[wangyt@pseudo-cluster basic]$ kubectl get rs
NAME        DESIRED   CURRENT   READY   AGE
tomcat-rs   3         3         3       7s
```

副本集详情：

```shell script
[wangyt@pseudo-cluster basic]$ kubectl describe replicasets/tomcat-rs
Name:         tomcat-rs
Namespace:    default
Selector:     app=mytomcat
Labels:       app=mytomcat
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=mytomcat
  Containers:
   tomcat:
    Image:        tomcat:8
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  59m   replicaset-controller  Created pod: tomcat-rs-s9d98
  Normal  SuccessfulCreate  59m   replicaset-controller  Created pod: tomcat-rs-96cp7
```

查看Pods：

```shell script
[wangyt@pseudo-cluster basic]$ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
tomcat            1/1     Running   0          140m
tomcat-rs-96cp7   1/1     Running   0          19s
tomcat-rs-s9d98   1/1     Running   0          19s
```

#### Service(服务)

Service将一组Pod组织在一起，然后可以通过Service名来访问，还可以实现Pod的负载均衡。

tomcat-service.yaml：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
  labels:
    app: mytomcat
spec:
  type: NodePort
  ports:
    - port: 30000
      nodePort: 30000
      targetPort: 8080
  selector:
    app: mytomcat
```

Service有以下三种类型：

* `NodePort`：通过Node的静态端口暴露服务。
* `LoadBalancer`：使用云平台提供商的Load Balancer(例如阿里云会提供一个公网IP地址)。
* `ClusterIP`：只能在k8s内部可以访问的服务。

部署Service：

```shell script
[wangyt@pseudo-cluster basic]$ kubectl create -f tomcat-service.yaml
service/tomcat-service created
```

查看Service：

```shell script
[wangyt@pseudo-cluster basic]$ kubectl get services                
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)           AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP           3h6m
tomcat-service   NodePort    10.111.23.38   <none>        30000:30000/TCP   12s
```

查看Service详情：
```shell script
[wangyt@pseudo-cluster basic]$ kubectl describe services/tomcat-service
Name:                     tomcat-service
Namespace:                default
Labels:                   app=mytomcat
Annotations:              <none>
Selector:                 app=mytomcat
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.111.23.38
IPs:                      10.111.23.38
Port:                     <unset>  30000/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30000/TCP
Endpoints:                172.17.0.5:8080,172.17.0.6:8080,172.17.0.7:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
[wangyt@pseudo-cluster basic]$ 
```

访问服务：

这里使用NodePort访问服务，可以通过下面的命令获取Node的IP地址：

`minikube ip`

```shell script
[wangyt@pseudo-cluster basic]$ minikube ip
192.168.49.2
[wangyt@pseudo-cluster basic]$ curl 192.168.49.2:30000
<!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> Status Report</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/8.5.78</h3></body></html>[wangyt@pseudo-cluster basic]$ 
[wangyt@pseudo-cluster basic]$
```
