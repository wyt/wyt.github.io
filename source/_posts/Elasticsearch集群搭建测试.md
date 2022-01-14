---
title: Elasticsearch集群搭建测试
date: 2019-8-21 14:06:30
updated: 2019-8-21 14:06:30
comments: true
categories:
- elasticsearch
tags:
- elasticsearch
---

单台机器搭建一个伪Elasticsearch集群，用于测试学习。

| 节点 | 主机 | HTTP端口 | TCP端口 |
| --- | --- | --- | --- |
| node-0 | 192.168.91.146 | 9200 | 9300 |
| node-1 | 192.168.91.146 | 9201 | 9301 |
| node-2 | 192.168.91.146 | 9202 | 9302 |

<!--more-->

#### 准备目录

下载elasticsearch安装包，按照如下目录组织

``` bash
[root@pseudo-cluster es-cluser-test]# pwd
/usr/local/es-cluser-test
[root@pseudo-cluster es-cluser-test]# ls -lh
total 8.0K
drwxr-xr-x. 9 elasticsearch elasticsearch 155 Aug 11 03:22 es-6.6.2-node0
drwxr-xr-x. 9 elasticsearch elasticsearch 165 Aug 11 03:22 es-6.6.2-node1
drwxr-xr-x. 9 elasticsearch elasticsearch 155 Aug 11 03:22 es-6.6.2-node2
-rwxr-xr-x. 1 elasticsearch elasticsearch 253 Aug  9 03:07 shutdown.sh
-rwxr-xr-x. 1 elasticsearch elasticsearch 264 Aug  9 03:13 startup.sh
[root@pseudo-cluster es-cluser-test]# 
```

#### 添加用户&组&赋权

``` bash
[root@pseudo-cluster es-cluser-test]# groupadd elasticsearch
[root@pseudo-cluster es-cluser-test]# useradd elasticsearch -g elasticsearch
[root@pseudo-cluster es-cluser-test]# chown -R elasticsearch.elasticsearch /usr/local/es-cluser-test
```

#### 修改配置文件

分别修改node0、node1、node2三个节点的`config/elasticsearch.yml`配置文件

``` bash
cluster.name: es-pseudo-cluster
node.name: node-0
bootstrap.memory_lock: true
network.host: 192.168.91.146
http.port: 9200
transport.tcp.port: 9300
discovery.zen.ping.unicast.hosts: ["192.168.91.146:9301", "192.168.91.146:9302"]
discovery.zen.minimum_master_nodes: 2
```

``` bash
cluster.name: es-pseudo-cluster
node.name: node-1
bootstrap.memory_lock: true
network.host: 192.168.91.146
http.port: 9201
transport.tcp.port: 9301
discovery.zen.ping.unicast.hosts: ["192.168.91.146:9300", "192.168.91.146:9302"]
discovery.zen.minimum_master_nodes: 2
```

``` bash
cluster.name: es-pseudo-cluster
node.name: node-2
bootstrap.memory_lock: true
network.host: 192.168.91.146
http.port: 9202
transport.tcp.port: 9302
discovery.zen.ping.unicast.hosts: ["192.168.91.146:9300", "192.168.91.146:9301"]
discovery.zen.minimum_master_nodes: 2
```

#### 启停脚本

##### 启动脚本

``` bash
#!/usr/bin/env bash

current_path=`pwd`
nodes=(es-6.6.2-node0 es-6.6.2-node1 es-6.6.2-node2)

for val in "${nodes[@]}"
do
    path=$current_path/$val
    echo $path
    cd $path
    ./bin/elasticsearch -d -p pid
done

ps -elf | grep -v 'grep' | grep Elasticsearch
```

##### 启动

``` bash
[root@pseudo-cluster es-cluser-test]# pwd
/usr/local/es-cluser-test
[root@pseudo-cluster es-cluser-test]# ls -lh
total 8.0K
drwxr-xr-x. 9 elasticsearch elasticsearch 155 Aug 14 03:52 es-6.6.2-node0
drwxr-xr-x. 9 elasticsearch elasticsearch 155 Aug 14 03:56 es-6.6.2-node1
drwxr-xr-x. 9 elasticsearch elasticsearch 155 Aug 14 03:56 es-6.6.2-node2
-rwxr-xr-x. 1 elasticsearch elasticsearch 253 Aug  9 03:07 shutdown.sh
-rwxr-xr-x. 1 elasticsearch elasticsearch 264 Aug 14 03:55 startup.sh
[root@pseudo-cluster es-cluser-test]# 
[root@pseudo-cluster es-cluser-test]# su elasticsearch
[elasticsearch@pseudo-cluster es-cluser-test]$ ./startup.sh 
/usr/local/es-cluser-test/es-6.6.2-node0
/usr/local/es-cluser-test/es-6.6.2-node1
/usr/local/es-cluser-test/es-6.6.2-node2
0 S elastic+  30162      1 99  80   0 - 776807 futex_ 03:57 pts/0   00:00:01 /usr/local/jdk1.8.0_221/bin/java -Xms512m -Xmx512m -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.io.tmpdir=/tmp/elasticsearch-3768772128440104570 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data -XX:ErrorFile=logs/hs_err_pid%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -Xloggc:logs/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=32 -XX:GCLogFileSize=64m -Des.path.home=/usr/local/es-cluser-test/es-6.6.2-node0 -Des.path.conf=/usr/local/es-cluser-test/es-6.6.2-node0/config -Des.distribution.flavor=default -Des.distribution.type=tar -cp /usr/local/es-cluser-test/es-6.6.2-node0/lib/* org.elasticsearch.bootstrap.Elasticsearch -d -p pid
0 S elastic+  30238      1  0  80   0 - 773344 futex_ 03:57 pts/0   00:00:00 /usr/local/jdk1.8.0_221/bin/java -Xms512m -Xmx512m -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.io.tmpdir=/tmp/elasticsearch-3776090600362042909 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data -XX:ErrorFile=logs/hs_err_pid%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -Xloggc:logs/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=32 -XX:GCLogFileSize=64m -Des.path.home=/usr/local/es-cluser-test/es-6.6.2-node1 -Des.path.conf=/usr/local/es-cluser-test/es-6.6.2-node1/config -Des.distribution.flavor=default -Des.distribution.type=tar -cp /usr/local/es-cluser-test/es-6.6.2-node1/lib/* org.elasticsearch.bootstrap.Elasticsearch -d -p pid
0 S elastic+  30314      1  0  80   0 - 25758 futex_ 03:57 pts/0    00:00:00 /usr/local/jdk1.8.0_221/bin/java -Xms512m -Xmx512m -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.io.tmpdir=/tmp/elasticsearch-4360777951552716525 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data -XX:ErrorFile=logs/hs_err_pid%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -Xloggc:logs/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=32 -XX:GCLogFileSize=64m -Des.path.home=/usr/local/es-cluser-test/es-6.6.2-node2 -Des.path.conf=/usr/local/es-cluser-test/es-6.6.2-node2/config -Des.distribution.flavor=default -Des.distribution.type=tar -cp /usr/local/es-cluser-test/es-6.6.2-node2/lib/* org.elasticsearch.bootstrap.Elasticsearch -d -p pid
[elasticsearch@pseudo-cluster es-cluser-test]$ 
```

##### 查看集群

通过Elasticsearch Head查看集群

![elasticsearch-pseudo-cluster](https://objects.yongtao.wang/images/20190821/elasticsearch-pseudo-cluster.png)

##### 停止脚本

``` bash
#!/usr/bin/env bash

current_path=`pwd`
nodes=(es-6.6.2-node0 es-6.6.2-node1 es-6.6.2-node2)

for val in "${nodes[@]}"
do
    path=$current_path/$val
    echo $path
    cd $path
    kill -9 `cat pid` 
done

ps -elf | grep -v 'grep' | grep Elasticsearch
```

##### 停止

``` bash
[elasticsearch@pseudo-cluster es-cluser-test]$ ./shutdown.sh 
/usr/local/es-cluser-test/es-6.6.2-node0
/usr/local/es-cluser-test/es-6.6.2-node1
/usr/local/es-cluser-test/es-6.6.2-node2
```