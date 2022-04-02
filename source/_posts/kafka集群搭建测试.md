---
title: Kafka集群搭建测试
date: 2019-8-27 11:04:00
updated: 2019-8-27 11:04:00
comments: true
categories:
- Kafka
tags:
- Kafka
---

单台机器搭建一个伪Kafka集群，用于测试学习。使用kafka内嵌的zookeeper。
生产环境应该使用独立的kafka集群。

| broker.id | 配置文件 | 主机 | 客户端连接端口 |
| --- | --- | --- | --- |
| 3 | server.3 | 192.168.101.146 | 9092 |
| 5 | server.5 | 192.168.101.146 | 2182 |
| 7 | server.7 | 192.168.101.146 | 2183 |

<!--more-->

#### 准备目录

``` bash
[root@pseudo-cluster kafka-cluster-test]# pwd
/usr/local/kafka-cluster-test
[root@pseudo-cluster kafka-cluster-test]# ls -lh
total 76K
drwxr-xr-x. 3 wangyt wangyt 4.0K Aug 17  2021 bin
drwxr-xr-x. 2 wangyt wangyt 4.0K Mar 29 02:09 config
drwxr-xr-x. 6 wangyt wangyt   95 Aug 29  2019 k_data
drwxr-xr-x. 2 wangyt wangyt 4.0K Aug 12  2019 libs
-rw-r--r--. 1 wangyt wangyt  32K Jun 20  2019 LICENSE
drwxr-xr-x. 2 wangyt wangyt  12K Mar 29 01:32 logs
-rw-r--r--. 1 wangyt wangyt  337 Jun 20  2019 NOTICE
drwxr-xr-x. 2 wangyt wangyt   81 Aug 17  2021 pids
-rwxr-xr-x. 1 wangyt wangyt  248 Aug 12  2019 shutdown.sh
drwxr-xr-x. 2 wangyt wangyt   44 Aug 12  2019 site-docs
-rwxr-xr-x. 1 wangyt wangyt  394 Aug 12  2019 startup.sh
[root@pseudo-cluster kafka-cluster-test]# cd config/
[root@pseudo-cluster config]# ls -lh
total 84K
-rw-r--r--. 1 wangyt wangyt  906 Jun 20  2019 connect-console-sink.properties
-rw-r--r--. 1 wangyt wangyt  909 Jun 20  2019 connect-console-source.properties
-rw-r--r--. 1 wangyt wangyt 5.2K Jun 20  2019 connect-distributed.properties
-rw-r--r--. 1 wangyt wangyt  883 Jun 20  2019 connect-file-sink.properties
-rw-r--r--. 1 wangyt wangyt  881 Jun 20  2019 connect-file-source.properties
-rw-r--r--. 1 wangyt wangyt 1.6K Jun 20  2019 connect-log4j.properties
-rw-r--r--. 1 wangyt wangyt 2.3K Jun 20  2019 connect-standalone.properties
-rw-r--r--. 1 wangyt wangyt 1.2K Jun 20  2019 consumer.properties
-rw-r--r--. 1 wangyt wangyt 4.7K Jun 20  2019 log4j.properties
-rw-r--r--. 1 wangyt wangyt 1.9K Jun 20  2019 producer.properties
-rw-r--r--. 1 wangyt wangyt 6.8K Aug 29  2019 server-3.properties
-rw-r--r--. 1 wangyt wangyt 6.8K Aug 29  2019 server-5.properties
-rw-r--r--. 1 wangyt wangyt 6.8K Aug 29  2019 server-7.properties
-rw-r--r--. 1 wangyt wangyt 1.1K Jun 20  2019 tools-log4j.properties
-rw-r--r--. 1 wangyt wangyt 1.2K Jun 20  2019 trogdor.conf
-rw-r--r--. 1 wangyt wangyt 1.1K Aug 29  2019 zookeeper.properties
```

#### 修改配置文件

config目录下复制三份server.properties，分别修改server-3、server-5、server-7

``` bash
broker.id=3
listeners=PLAINTEXT://192.168.91.146:9092
log.dirs=k_data/kafka-logs-node3
zookeeper.connect=localhost:2181
```

``` bash
broker.id=5
listeners=PLAINTEXT://192.168.91.146:9093
log.dirs=k_data/kafka-logs-node5
zookeeper.connect=localhost:2181
```

``` bash
broker.id=7
listeners=PLAINTEXT://192.168.91.146:9094
log.dirs=k_data/kafka-logs-node7
zookeeper.connect=localhost:2181
```

zookeeper.properties配置

```bash
dataDir=k_data/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
```

#### 启停脚本

##### 启动脚本

startup.sh

``` bash
#!/usr/bin/env bash

pidExt='.pid'
propExt='.properties'
nodes=(server-3 server-5 server-7)

# start zookeeper
nohup ./bin/zookeeper-server-start.sh config/zookeeper.properties >/dev/null 2>&1 &
echo "$!" > ./pids/zoo$pidExt
sleep 5

# start kafka cluster
for val in "${nodes[@]}"
do
        nohup ./bin/kafka-server-start.sh config/$val$propExt >/dev/null 2>&1 &
        echo "$!" > ./pids/$val$pidExt
done
```

##### 停止脚本

``` bash
#!/usr/bin/env bash

function process_dir() {
        for file in `ls $1`
        do
                if [ -d $1"/"$file ]
                then
                        process_dir $1"/"$file
                else
                        #echo $1"/"$file `cat $1"/"$file`
                        kill -9 `cat $1"/"$file`
                fi
        done
}

process_dir ./pids
rm -rf ./pids/*
```
