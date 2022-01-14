---
title: Docker Swarm 一、 概述
date: 2018-10-26 19:36:00
updated: 2018-10-26 19:36:00
comments: true
categories:
- docker
tags:
- docker
- docker swarm
---

自docker engine 1.12引入swarm模式后，可由一个或多个docker引擎组成一个集群，称之为swarm，一个swarm由一个或多个节点组成。

### 关键概念

#### 节点

swarm节点分为两种类型：`manager`和`worker`，

Manager节点维护集群状态，调度服务，提供swarm模式下的API服务等，多个Manager节点只有一个leader执行编排任务；

Worker节点接收并执行从Manager节点分派的任务，默认情况下Manager节点同时也是Worker节点；

#### 服务

服务是对在管理节点和工作节点执行任务的定义，创建服务时可以指定具体使用的镜像和容器中执行的命令等。

服务有两种模式:`replicated` 和 `global`

#### 任务


#### 负载均衡




[Swarm mode overview](https://docs.docker.com/engine/swarm/)
[Swarm mode key concepts](https://docs.docker.com/engine/swarm/key-concepts/)