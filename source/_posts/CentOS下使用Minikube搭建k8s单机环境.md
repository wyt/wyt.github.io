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

#### Deployment(部署)

Deployment可以管理ReplicaSet和Pod的部署，具备更新和回滚的能力。

`tomcat-deployment.yaml`：

```shell script
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
  labels:
    app: mytomcat
spec:
  replicas: 4
  selector:
    matchLabels:
      app: mytomcat
  template:
    metadata:
      labels:
        app: mytomcat
    spec:
      containers:
        - name: tomcat
          image: tomcat:8
```

部署Deployment

```shell script
[wangyt@pseudo-cluster basic]$ kubectl create -f tomcat-deployment.yaml
deployment.apps/tomcat-deployment created
[wangyt@pseudo-cluster basic]$ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
tomcat            1/1     Running   0          4h37m
tomcat-rs-7sj7h   1/1     Running   0          11s
tomcat-rs-96cp7   1/1     Running   0          137m
tomcat-rs-s9d98   1/1     Running   0          137m
```

查看Deployment：

```shell script
kubectl get deployments
kubectl get pods
```

查看Deployment详情：

```shell script
[wangyt@pseudo-cluster basic]$ kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
tomcat-deployment   4/4     4            4           99s
[wangyt@pseudo-cluster basic]$ kubectl describe deployments/tomcat-deployment
Name:                   tomcat-deployment
Namespace:              default
CreationTimestamp:      Wed, 13 Apr 2022 08:39:17 +0800
Labels:                 app=mytomcat
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=mytomcat
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
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
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   tomcat-rs (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m25s  deployment-controller  Scaled up replica set tomcat-rs to 4
```

更新部署，升级到tomcat9版本。

```shell script
[wangyt@pseudo-cluster basic]$ kubectl set image deployment/tomcat-deployment tomcat=tomcat:9
deployment.apps/tomcat-deployment image updated
[wangyt@pseudo-cluster basic]$ 
[wangyt@pseudo-cluster basic]$ 
[wangyt@pseudo-cluster basic]$ 
[wangyt@pseudo-cluster basic]$ kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
tomcat                              1/1     Running             0          5h14m
tomcat-deployment-77895cc49-4jr79   0/1     ContainerCreating   0          12s
tomcat-deployment-77895cc49-qcrqf   0/1     ContainerCreating   0          12s
tomcat-rs-96cp7                     1/1     Running             0          174m
tomcat-rs-s9d98                     1/1     Running             0          174m
[wangyt@pseudo-cluster basic]$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
tomcat-deployment-77895cc49-4jr79   1/1     Running   0          9m59s
tomcat-deployment-77895cc49-qcrqf   1/1     Running   0          9m59s
tomcat-deployment-77895cc49-r7pp5   1/1     Running   0          5m7s
tomcat-deployment-77895cc49-rgsxw   1/1     Running   0          5m12s
```

查看版本是否更新：

可以看到版本变成了`Tomcat/9.0.62`

```bash
[wangyt@pseudo-cluster basic]$ curl `minikube ip`:30000
<!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> Status Report</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/9.0.62</h3></body></html>[wangyt@pseudo-cluster basic]$ 
[wangyt@pseudo-cluster basic]$
```

更新Pod数量：

```shell script
[wangyt@pseudo-cluster tmp]$ kubectl scale deployment tomcat-deployment --replicas 2
deployment.apps/tomcat-deployment scaled
[wangyt@pseudo-cluster tmp]$ kubectl get pods      
NAME                                READY   STATUS    RESTARTS   AGE
tomcat-deployment-77895cc49-4jr79   1/1     Running   0          15h
tomcat-deployment-77895cc49-rgsxw   1/1     Running   0          15h
```

回滚：

查看历史版本：

```shell script
[wangyt@pseudo-cluster tmp]$ kubectl rollout history deployments/tomcat-deployment 
deployment.apps/tomcat-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

```shell script
[wangyt@pseudo-cluster tmp]$ kubectl get deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment    1/1     1            1           19m
tomcat-deployment   2/2     2            2           16h
## 回滚到版本1
[wangyt@pseudo-cluster tmp]$ kubectl rollout undo deployment/tomcat-deployment --to-revision=1
deployment.apps/tomcat-deployment rolled back
## 查看回滚状态
[wangyt@pseudo-cluster tmp]$ kubectl rollout status deployment/tomcat-deployment
deployment "tomcat-deployment" successfully rolled out
[wangyt@pseudo-cluster tmp]$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-66cc6765b4-zzdmf   1/1     Running   0          21m
tomcat-rs-4rpwj                     1/1     Running   0          47s
tomcat-rs-tkjj8                     1/1     Running   0          49s
[wangyt@pseudo-cluster tmp]$ curl `minikube ip`:30000
<!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> Status Report</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/8.5.78</h3></body></html>
[wangyt@pseudo-cluster tmp]$ 
```

可以看到版本已变更为：Tomcat/8.5.78

#### Ingress(访问权)

访问权用来定义外部访问集群内部服务的规则，类似于Gateway。

开启nginx ingress controller，网络失败多试几次，不行开启代理。也可以使用`minikube ssh`登录，再使用`docker pull`手动拉取镜像。

```shell script
[wangyt@pseudo-cluster spring-boot-book-source-code]$ minikube addons enable ingress
  - Using image k8s.gcr.io/ingress-nginx/controller:v1.1.0
  - Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
  - Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
* Verifying ingress addon...
* The 'ingress' addon is enabled
[wangyt@pseudo-cluster spring-boot-book-source-code]$ minikube addons list

|-----------------------------|----------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE  |    STATUS    |           MAINTAINER           |
|-----------------------------|----------|--------------|--------------------------------|
| ambassador                  | minikube | disabled     | third-party (ambassador)       |
| auto-pause                  | minikube | disabled     | google                         |
| csi-hostpath-driver         | minikube | disabled     | kubernetes                     |
| dashboard                   | minikube | enabled ✅   | kubernetes                     |
| default-storageclass        | minikube | enabled ✅   | kubernetes                     |
| efk                         | minikube | disabled     | third-party (elastic)          |
| freshpod                    | minikube | disabled     | google                         |
| gcp-auth                    | minikube | disabled     | google                         |
| gvisor                      | minikube | disabled     | google                         |
| helm-tiller                 | minikube | disabled     | third-party (helm)             |
| ingress                     | minikube | enabled ✅   | unknown (third-party)          |
| ingress-dns                 | minikube | disabled     | google                         |
| istio                       | minikube | disabled     | third-party (istio)            |
| istio-provisioner           | minikube | disabled     | third-party (istio)            |
| kubevirt                    | minikube | disabled     | third-party (kubevirt)         |
| logviewer                   | minikube | disabled     | unknown (third-party)          |
| metallb                     | minikube | disabled     | third-party (metallb)          |
| metrics-server              | minikube | disabled     | kubernetes                     |
| nvidia-driver-installer     | minikube | disabled     | google                         |
| nvidia-gpu-device-plugin    | minikube | disabled     | third-party (nvidia)           |
| olm                         | minikube | disabled     | third-party (operator          |
|                             |          |              | framework)                     |
| pod-security-policy         | minikube | disabled     | unknown (third-party)          |
| portainer                   | minikube | disabled     | portainer.io                   |
| registry                    | minikube | disabled     | google                         |
| registry-aliases            | minikube | disabled     | unknown (third-party)          |
| registry-creds              | minikube | disabled     | third-party (upmc enterprises) |
| storage-provisioner         | minikube | enabled ✅   | google                         |
| storage-provisioner-gluster | minikube | disabled     | unknown (third-party)          |
| volumesnapshots             | minikube | disabled     | kubernetes                     |
|-----------------------------|----------|--------------|--------------------------------|
```

`ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /tomcat(.*)
            backend:
              serviceName: tomcat-service
              servicePort: 30000
```


#### 数据存储


#### 示例说明

示例中tomcat貌似无论怎么访问都是404，原因如下：

```shell script
root@tomcat-rs-4rpwj:/usr/local/tomcat# ls -lh
total 128K
-rw-r--r-- 1 root root  20K Mar 31 16:05 BUILDING.txt
-rw-r--r-- 1 root root 6.1K Mar 31 16:05 CONTRIBUTING.md
-rw-r--r-- 1 root root  56K Mar 31 16:05 LICENSE
-rw-r--r-- 1 root root 1.7K Mar 31 16:05 NOTICE
-rw-r--r-- 1 root root 3.3K Mar 31 16:05 README.md
-rw-r--r-- 1 root root 7.0K Mar 31 16:05 RELEASE-NOTES
-rw-r--r-- 1 root root  17K Mar 31 16:05 RUNNING.txt
drwxr-xr-x 2 root root 4.0K Apr  1 19:59 bin
drwxr-xr-x 1 root root   22 Apr 13 17:00 conf
drwxr-xr-x 2 root root 4.0K Apr  1 19:59 lib
drwxrwxrwx 1 root root  220 Apr 14 02:34 logs
drwxr-xr-x 2 root root  159 Apr  1 19:59 native-jni-lib
drwxrwxrwx 2 root root   30 Apr  1 19:59 temp
drwxr-xr-x 2 root root    6 Apr  1 19:59 webapps
drwxr-xr-x 7 root root   81 Mar 31 16:05 webapps.dist
drwxrwxrwx 2 root root    6 Mar 31 16:05 work
root@tomcat-rs-4rpwj:/usr/local/tomcat# 
root@tomcat-rs-4rpwj:/usr/local/tomcat# 
root@tomcat-rs-4rpwj:/usr/local/tomcat# cd webapps
root@tomcat-rs-4rpwj:/usr/local/tomcat/webapps# ls -lh
total 0
root@tomcat-rs-4rpwj:/usr/local/tomcat/webapps# cd ../webapps.dist/
root@tomcat-rs-4rpwj:/usr/local/tomcat/webapps.dist# ls -lh
total 4.0K
drwxr-xr-x  3 root root  223 Apr  1 19:59 ROOT
drwxr-xr-x 15 root root 4.0K Apr  1 19:59 docs
drwxr-xr-x  7 root root   99 Apr  1 19:59 examples
drwxr-xr-x  6 root root   79 Apr  1 19:59 host-manager
drwxr-xr-x  6 root root  114 Apr  1 19:59 manager
root@tomcat-rs-4rpwj:/usr/local/tomcat/webapps.dist# exit
exit
docker@minikube:~$ exit
logout
[wangyt@pseudo-cluster basic]$ 
```



[Ingress](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/)
