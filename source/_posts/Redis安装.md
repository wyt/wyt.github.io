---
title: Redis安装
date: 2017-10-16 22:27:00
categories:
- 缓存
- Redis
tags:
- 缓存
- Redis
---

## 安装Redis

### 下载安装

``` bash
[root@localhost src] wget http://download.redis.io/releases/redis-4.0.2.tar.gz
[root@localhost src] tar xzf redis-4.0.2.tar.gz
[root@localhost src] cd redis-4.0.2
[root@localhost src] make
```

<!--more-->

### 启动Redis服务

``` bash
[root@localhost src]# pwd
/usr/local/src/redis-4.0.1/src
[root@localhost src]# ./redis-server 
3163:C 12 Jul 04:23:15.614 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
3163:C 12 Jul 04:23:15.614 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=3163, just started
3163:C 12 Jul 04:23:15.614 # Warning: no config file specified, using the default config. In order to specify a config file use ./redis-server /path/to/redis.conf
3163:M 12 Jul 04:23:15.615 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 4.0.1 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 3163
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

3163:M 12 Jul 04:23:15.616 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
3163:M 12 Jul 04:23:15.616 # Server initialized
3163:M 12 Jul 04:23:15.616 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
3163:M 12 Jul 04:23:15.616 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
3163:M 12 Jul 04:23:15.616 * DB loaded from disk: 0.000 seconds
3163:M 12 Jul 04:23:15.616 * Ready to accept connections
```

### 通过内建的客户端和Redis交互

``` bash
[root@localhost src] ./redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

### 远程连接配置

``` bash
[root@localhost src]# pwd
/usr/local/src/redis-4.0.1/src
[root@localhost src]# vim ../redis.conf

# 设置密码为123456
requirepass 123456

[root@localhost src]# ./redis-server ../redis.conf 
```

#### 客户端远程连接

``` bash
PS E:\dev_app\Redis-x64-3.0.504> .\redis-cli.exe -h 192.168.80.129 -p 6379 -a "123456"
```


Ref: https://redis.io/download
