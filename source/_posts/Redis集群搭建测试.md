---
title: Redis集群搭建测试
date: 2019-8-1 11:28:49
updated: 2019-8-2 10:46:15
categories:
- 缓存
- Redis
tags:
- 缓存
- Redis
- Cluster
---

单台机器搭建一个伪Redis集群，用于测试学习。redis-3.2.13可以使用redis源码自行编译`make prifix=/tmp/redis`。

#### 下载redis-cluster-test项目

``` bash
[root@pseudo-cluster local]# pwd
/usr/local
[root@pseudo-cluster local]# git clone https://github.com/wyt/redis-cluster-test.git
[root@pseudo-cluster bin]# pwd
/usr/local/redis-cluster-test/redis-3.2.13/bin
[root@pseudo-cluster bin]# chmod +x ./*
[root@pseudo-cluster redis-cluster-test]# pwd
/usr/local/redis-cluster-test
[root@pseudo-cluster redis-cluster-test]# chmod +x shutdown.sh startup.sh
```

#### 安装ruby环境等
``` bash
# 安装ruby环境
[root@pseudo-cluster redis-cluster-test] yum install centos-release-scl-rh
[root@pseudo-cluster redis-cluster-test] yum install rh-ruby23 -y
# 当前session生效
[root@pseudo-cluster redis-cluster-test] scl enable rh-ruby23 bash
[root@pseudo-cluster redis-cluster-test]# ruby -v
ruby 2.3.8p459 (2018-10-18 revision 65136) [x86_64-linux]
[root@pseudo-cluster redis-cluster-test]# gem install redis
Fetching: redis-4.1.2.gem (100%)
Successfully installed redis-4.1.2
Parsing documentation for redis-4.1.2
Installing ri documentation for redis-4.1
```

<!--more-->

#### 规划集群
``` bash
[root@pseudo-cluster redis-cluster-test]# pwd
/usr/local/redis-cluster-test
[root@pseudo-cluster redis-cluster-test]# ls -lh
total 16K
drwxr-xr-x. 2 root root   64 Aug  1 18:57 7000
drwxr-xr-x. 2 root root   64 Aug  1 18:57 7001
drwxr-xr-x. 2 root root   64 Aug  1 18:57 7002
drwxr-xr-x. 2 root root   64 Aug  1 18:57 7003
drwxr-xr-x. 2 root root   64 Aug  1 18:57 7004
drwxr-xr-x. 2 root root   64 Aug  1 18:57 7005
-rw-r--r--. 1 root root 7.1K Aug  1 18:56 README.md
drwxr-xr-x. 3 root root   35 Aug  1 18:54 redis-3.2.13
-rwxr-xr-x. 1 root root   75 Aug  1 18:54 shutdown.sh
-rwxr-xr-x. 1 root root  635 Aug  1 18:54 startup.sh
[root@pseudo-cluster redis-cluster-test]# ./startup.sh 
/usr/local/redis-cluster-test/7000
/usr/local/redis-cluster-test/7001
/usr/local/redis-cluster-test/7002
/usr/local/redis-cluster-test/7003
/usr/local/redis-cluster-test/7004
/usr/local/redis-cluster-test/7005
5 S root      14845      1  0  80   0 - 35257 ep_pol 18:59 ?        00:00:00 ../redis-3.2.13/bin/redis-server *:7000 [cluster]
5 S root      14849      1  0  80   0 - 35257 ep_pol 18:59 ?        00:00:00 ../redis-3.2.13/bin/redis-server *:7001 [cluster]
5 S root      14853      1  0  80   0 - 35257 ep_pol 18:59 ?        00:00:00 ../redis-3.2.13/bin/redis-server *:7002 [cluster]
5 S root      14857      1  0  80   0 - 35257 ep_pol 18:59 ?        00:00:00 ../redis-3.2.13/bin/redis-server *:7003 [cluster]
5 S root      14861      1  0  80   0 - 35257 ep_pol 18:59 ?        00:00:00 ../redis-3.2.13/bin/redis-server *:7004 [cluster]
5 S root      14865      1  0  80   0 - 35257 ep_pol 18:59 ?        00:00:00 ../redis-3.2.13/bin/redis-server *:7005 [cluster]
[root@pseudo-cluster redis-cluster-test]# ./redis-3.2.13/bin/redis-trib.rb create --replicas 1 192.168.91.146:7000 192.168.91.146:7001 192.168.91.146:7002 192.168.91.146:7003 192.168.91.146:7004 192.168.91.146:7005
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.91.146:7000
192.168.91.146:7001
192.168.91.146:7002
Adding replica 192.168.91.146:7003 to 192.168.91.146:7000
Adding replica 192.168.91.146:7004 to 192.168.91.146:7001
Adding replica 192.168.91.146:7005 to 192.168.91.146:7002
M: 50925d802a5f1be70521fafea196e8b112c122ca 192.168.91.146:7000
   slots:0-5460 (5461 slots) master
M: c4a38513a633aab450a3d491c30ec6b305b7abef 192.168.91.146:7001
   slots:5461-10922 (5462 slots) master
M: 963dd0cac7581f29464624cb0df8a678cdfc54cd 192.168.91.146:7002
   slots:10923-16383 (5461 slots) master
S: 6f3aa71e0d754d54738d746bd617d31adabd0d3f 192.168.91.146:7003
   replicates 50925d802a5f1be70521fafea196e8b112c122ca
S: 60cd05e9ed0350bd271fabacf00483bc9a34c2da 192.168.91.146:7004
   replicates c4a38513a633aab450a3d491c30ec6b305b7abef
S: 2755ff19dcb8d00967e66f7d19bf95603fa352dc 192.168.91.146:7005
   replicates 963dd0cac7581f29464624cb0df8a678cdfc54cd
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 192.168.91.146:7000)
M: 50925d802a5f1be70521fafea196e8b112c122ca 192.168.91.146:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 963dd0cac7581f29464624cb0df8a678cdfc54cd 192.168.91.146:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 2755ff19dcb8d00967e66f7d19bf95603fa352dc 192.168.91.146:7005
   slots: (0 slots) slave
   replicates 963dd0cac7581f29464624cb0df8a678cdfc54cd
M: c4a38513a633aab450a3d491c30ec6b305b7abef 192.168.91.146:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 6f3aa71e0d754d54738d746bd617d31adabd0d3f 192.168.91.146:7003
   slots: (0 slots) slave
   replicates 50925d802a5f1be70521fafea196e8b112c122ca
S: 60cd05e9ed0350bd271fabacf00483bc9a34c2da 192.168.91.146:7004
   slots: (0 slots) slave
   replicates c4a38513a633aab450a3d491c30ec6b305b7abef
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

#### 查看集群信息

``` bash
[root@pseudo-cluster redis-cluster-test]# ./redis-3.2.13/bin/redis-cli -c -p 7000
127.0.0.1:7000> CLUSTER NODES
963dd0cac7581f29464624cb0df8a678cdfc54cd 192.168.91.146:7002 master - 0 1564657233357 3 connected 10923-16383
2755ff19dcb8d00967e66f7d19bf95603fa352dc 192.168.91.146:7005 slave 963dd0cac7581f29464624cb0df8a678cdfc54cd 0 1564657233862 6 connected
50925d802a5f1be70521fafea196e8b112c122ca 192.168.91.146:7000 myself,master - 0 0 1 connected 0-5460
c4a38513a633aab450a3d491c30ec6b305b7abef 192.168.91.146:7001 master - 0 1564657234365 2 connected 5461-10922
6f3aa71e0d754d54738d746bd617d31adabd0d3f 192.168.91.146:7003 slave 50925d802a5f1be70521fafea196e8b112c122ca 0 1564657235373 4 connected
60cd05e9ed0350bd271fabacf00483bc9a34c2da 192.168.91.146:7004 slave c4a38513a633aab450a3d491c30ec6b305b7abef 0 1564657234868 5 connected
127.0.0.1:7000> CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_sent:207
cluster_stats_messages_received:207
```

#### 集群数据操作
``` bash
[root@pseudo-cluster redis-cluster-test]# ./redis-3.2.13/bin/redis-cli -c -p 7000
127.0.0.1:7000> set foo bar
-> Redirected to slot [12182] located at 192.168.91.146:7002
OK
192.168.91.146:7002> set hello world
-> Redirected to slot [866] located at 192.168.91.146:7000
OK
192.168.91.146:7000> set name sunwukong
-> Redirected to slot [5798] located at 192.168.91.146:7001
OK
192.168.91.146:7001> get foo
-> Redirected to slot [12182] located at 192.168.91.146:7002
"bar"
192.168.91.146:7002> get hello
-> Redirected to slot [866] located at 192.168.91.146:7000
"world"
192.168.91.146:7000> get name
-> Redirected to slot [5798] located at 192.168.91.146:7001
"sunwukong"
192.168.91.146:7001> 
# 从节点默认不能写入
[root@pseudo-cluster redis-cluster-test]# ./redis-3.2.13/bin/redis-cli -c -p 7004
127.0.0.1:7004> set key001 value001
-> Redirected to slot [12657] located at 192.168.91.146:7002
OK
192.168.91.146:7002> get key001
"value001"
192.168.91.146:7002> 
```

参考：
[redis-cluster-test](https://github.com/wyt/redis-cluster-test)，wyt
[cluster-tutorial](http://www.redis.cn/topics/cluster-tutorial.html)，redis.cn