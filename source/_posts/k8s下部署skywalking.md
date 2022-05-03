---
title: k8s下部署skywalking
date: 2022-05-01 21:14:00
updated: 2022-05-03 23:02:00
comments: true
categories:
- skywalking
tags:
- helm
- k8s
- skywalking
---

#### 节点

初始化环境变量

```shell script
export SKYWALKING_RELEASE_NAME=skywalking  # change the release name according to your scenario
export SKYWALKING_RELEASE_NAMESPACE=default  # change the namespace to where you want to install SkyWalking
```

添加skywalking helm仓库

```shell script
export REPO=skywalking
helm repo add ${REPO} https://apache.jfrog.io/artifactory/skywalking-helm
```

<!--more-->

拉去chart包到本地

```shell script
[wangyt@pseudo-cluster ~]$ helm pull skywalking/skywalking
[wangyt@pseudo-cluster ~]$ tar -zxvf skywalking-4.2.0.tgz       ## 解压
[wangyt@pseudo-cluster ~]$ ls -lh
drwxrwxr-x  5 wangyt wangyt 241 May  3 16:30 skywalking
-rw-r--r--  1 wangyt wangyt 44K May  3 16:30 skywalking-4.2.0.tgz
```

修改`./skywalking/values-my-es.yaml`，替换为自己的es集群

```yaml
oap:
  image:
    tag: 8.8.1
  storageType: elasticsearch

ui:
  image:
    tag: 8.8.1

elasticsearch:
  enabled: false
  config:               # For users of an existing elasticsearch cluster,takes effect when `elasticsearch.enabled` is false
    host: 192.168.101.1
    port:
      http: 9200
```

执行安装

```shell script
helm install "${SKYWALKING_RELEASE_NAME}" ${REPO}/skywalking -n "${SKYWALKING_RELEASE_NAMESPACE}" -f ./skywalking/values-my-es.yaml
```

查看pod

```shell script
[wangyt@localhost ~]$ kubectl get po
NAME                              READY   STATUS      RESTARTS   AGE
skywalking-es-init-9ttt5          0/1     Completed   0          25m
skywalking-oap-5f4cc5c9fb-72sz4   1/1     Running     1          25m
skywalking-oap-5f4cc5c9fb-q2f5h   1/1     Running     1          25m
skywalking-ui-544bfb887b-cvvpf    1/1     Running     0          25m
[wangyt@localhost ~]$
```

查看服务

```shell script
[wangyt@localhost ~]$ kubectl get svc -o wide
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)               AGE     SELECTOR
kubernetes       ClusterIP   10.233.0.1      <none>        443/TCP               5h42m   <none>
skywalking-oap   ClusterIP   10.233.24.130   <none>        11800/TCP,12800/TCP   25m     app=skywalking,component=oap,release=skywalking
skywalking-ui    ClusterIP   10.233.55.146   <none>        80/TCP                25m     app=skywalking,component=ui,release=skywalking
```

```shell script
[wangyt@localhost ~]$ kubectl port-forward --address 0.0.0.0 svc/skywalking-ui 8080:80 --namespace default
```

然后就可以使用宿主机IP访问skywalking-ui了。

[skywalking-kubernetes](https://github.com/apache/skywalking-kubernetes)
