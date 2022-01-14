---
title: Zookeeper集群搭建测试
date: 2019-8-26 15:04:00
updated: 2019-8-26 15:04:00
comments: true
categories:
- Zookeeper
tags:
- Zookeeper
---

单台机器搭建一个伪Zookeeper集群，用于测试学习。

| 节点 | 主机 | 客户端连接端口 |follower连接leader端口 | leader选举端口 |
| --- | --- | --- | --- | --- |
| server.1 | 192.168.91.146 | 2181 | 2887 | 3887 |
| server.2 | 192.168.91.146 | 2182 | 2888 | 3888 |
| server.3 | 192.168.91.146 | 2183 | 2889 | 3889 |

<!--more-->

#### 准备目录

``` bash
[root@pseudo-cluster zookeeper-cluster-test]# ls -lh
total 8.0K
-rwxr-xr-x. 1 root root 147 Aug 23 11:03 shutdown.sh
-rwxr-xr-x. 1 root root 265 Aug 23 10:53 startup.sh
drwxr-xr-x. 8 root root 166 Aug 22 16:58 zookeeper-3.5.5-server.1
drwxr-xr-x. 8 root root 166 Aug 22 17:26 zookeeper-3.5.5-server.2
drwxr-xr-x. 8 root root 166 Aug 22 17:26 zookeeper-3.5.5-server.3
```

#### 修改配置文件

分别修改server.1、server.2、server.3三个节点的`conf/zoo.cfg`配置文件

``` bash
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=5
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=2
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=__DATA_ZOO__/data
dataLogDir=__DATA_ZOO__/logs
# the port at which the clients will connect
clientPort=2181
# server.x=[hostname]:nnnnn[:nnnnn], etc : (No Java system property) servers making up the ZooKeeper ensemble. When the server starts up, it determines which server it is by looking for the file myid in the data directory. That file contains the server number, in ASCII, and it should match x in server.x in the left hand side of this setting. The list of servers that make up ZooKeeper servers that is used by the clients must match the list of ZooKeeper servers that each ZooKeeper server has. There are two port numbers nnnnn. The first followers use to connect to the leader, and the second is for leader election. If you want to test multiple servers on a single machine, then different ports can be used for each server.
server.1=192.168.91.146:2887:3887
server.2=192.168.91.146:2888:3888
server.3=192.168.91.146:2889:3889
```

``` bash
tickTime=2000
initLimit=5
syncLimit=2
dataDir=__DATA_ZOO__/data
dataLogDir=__DATA_ZOO__/logs
clientPort=2182
server.1=192.168.91.146:2887:3887
server.2=192.168.91.146:2888:3888
server.3=192.168.91.146:2889:3889
```

``` bash
tickTime=2000
initLimit=5
syncLimit=2
dataDir=__DATA_ZOO__/data
dataLogDir=__DATA_ZOO__/logs
clientPort=2183
server.1=192.168.91.146:2887:3887
server.2=192.168.91.146:2888:3888
server.3=192.168.91.146:2889:3889
```

#### 启停脚本

##### 启动脚本

``` bash
#!/usr/bin/env bash
current_path=`pwd`
nodes=(zookeeper-3.5.5-server.1 zookeeper-3.5.5-server.2 zookeeper-3.5.5-server.3)
for val in "${nodes[@]}"
do
    path=$current_path/$val
    echo $path
    cd $path
        ./bin/zkServer.sh start
done
jps -l | grep QuorumPeerMain
```

##### 启动

``` bash
[root@pseudo-cluster zookeeper-cluster-test]# pwd
/usr/local/zookeeper-cluster-test
[root@pseudo-cluster zookeeper-cluster-test]# ./startup.sh 
/usr/local/zookeeper-cluster-test/zookeeper-3.5.5-server.1
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-cluster-test/zookeeper-3.5.5-server.1/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
/usr/local/zookeeper-cluster-test/zookeeper-3.5.5-server.2
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-cluster-test/zookeeper-3.5.5-server.2/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
/usr/local/zookeeper-cluster-test/zookeeper-3.5.5-server.3
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-cluster-test/zookeeper-3.5.5-server.3/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
56626 org.apache.zookeeper.server.quorum.QuorumPeerMain
56739 org.apache.zookeeper.server.quorum.QuorumPeerMain
56679 org.apache.zookeeper.server.quorum.QuorumPeerMain
[root@pseudo-cluster zookeeper-cluster-test]#
```

##### 查看集群



##### 停止脚本

``` bash
#!/usr/bin/env bash

jps -l | grep QuorumPeerMain

kill -9 $(jps -l | grep org.apache.zookeeper.server.quorum.QuorumPeerMain | awk '{{print $1}}')
```

##### 停止

``` bash
[root@pseudo-cluster zookeeper-cluster-test]# ./shutdown.sh 
56626 org.apache.zookeeper.server.quorum.QuorumPeerMain
56739 org.apache.zookeeper.server.quorum.QuorumPeerMain
56679 org.apache.zookeeper.server.quorum.QuorumPeerMain
```