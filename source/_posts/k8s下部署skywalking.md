---
title: k8s下部署skywalking
date: 2022-05-01 21:14:00
updated: 2022-05-01 21:14:00
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

使用helm repo安装

```shell script
export REPO=skywalking
helm repo add ${REPO} https://apache.jfrog.io/artifactory/skywalking-helm                                
helm install "${SKYWALKING_RELEASE_NAME}" ${REPO}/skywalking -n "${SKYWALKING_RELEASE_NAMESPACE}" \
  --set oap.image.tag=8.8.1 \
  --set oap.storageType=elasticsearch \
  --set ui.image.tag=8.8.1 \
  --set elasticsearch.imageTag=6.8.6
```



[skywalking-kubernetes](https://github.com/apache/skywalking-kubernetes)
