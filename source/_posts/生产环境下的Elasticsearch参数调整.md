---
title: 生产环境下的Elasticsearch参数调整
date: 2019-6-28 09:37:00
updated: 2019-6-28 09:37:05
comments: true
categories:
- elasticsearch
tags:
- elasticsearch
---

开启`config/elasticsearch.yml`中`network.host`配置后，ES启动时会切换为生产模式，开发模式下的警告会升级为异常，导致ES节点无法启动。

#### 禁止swapping

设置Elasticsearch可锁定的、不被交换出的内存不受限制。编辑`config/elasticsearch.yml`
``` bash
# 设置为true
bootstrap.memory_lock: true
```

编辑`/etc/security/limits.conf`文件，追加下述配置：
``` bash
# elasticsearch用户锁定内存(不会被swap)不受限制
elasticsearch  -  memlock unlimited
```
<!--more-->

#### 增大文件描述符

编辑`/etc/security/limits.conf`文件，追加下述配置：
``` bash
# elasticsearch用户最多打开文件描述符的数量
elasticsearch  -  nofile  65536
```

#### 保证足够的线程数

编辑`/etc/security/limits.conf`文件，追加下述配置：
``` bash
# 设置elasticsearch用户最多创建线程数
elasticsearch  -  nproc 4096
```

#### 保证足够的虚拟内存

编辑`/etc/sysctl.conf`文件，追加下述配置：
``` bash
vm.max_map_count=262144
```

#### 使设置生效

``` bash
# 加载系统配置参数
[root@elasticstack-server filebeat-6.6.2-nginx]# sysctl -p
[root@elasticstack-server filebeat-6.6.2-nginx]# sysctl vm.max_map_count
vm.max_map_count = 262144

# 切换到elasticsearch用户使ulimit设置生效
[root@elasticstack-server filebeat-6.6.2-nginx]# su elasticsearch
[elasticsearch@elasticstack-server elasticsearch-6.6.2]$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31140
max locked memory       (kbytes, -l) unlimited
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65536
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

# 启动Elasticsearch
[elasticsearch@elasticstack-server elasticsearch-6.6.2]$ ./bin/elasticsearch -d -p pid
```

#### 检查设置

Elasticsearch启动成功后，检查设置项：
``` bash
# 设置成功
[root@elasticstack-server ~]# curl -X GET "localhost:9200/_nodes?filter_path=**.mlockall"
{"nodes":{"yW_SUaDnS6WED2QPm5y4TA":{"process":{"mlockall":true}}}}

[root@elasticstack-server ~]# curl -X GET "localhost:9200/_nodes/stats/process?filter_path=**.max_file_descriptors"
{"nodes":{"yW_SUaDnS6WED2QPm5y4TA":{"process":{"max_file_descriptors":65536}}}}
```

https://www.elastic.co/guide/en/elasticsearch/reference/6.2/system-config.html