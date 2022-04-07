---
title: Telegraf + Kafka + Vector + ClickHouse 指标采集实现
date: 2022-4-6 10:32:00
categories:
- 指标收集
tags:
- telegraf
- kafka
- vector
- clickhouse
---

本示例在windows环境下演示，绝大部分命令和linux通用。

### 简介

* [telegraf](https://docs.influxdata.com/telegraf/v1.22/)：指标收集工具，可收集中间件暴露的性能指标，本文中我们以收集redis指标为例。
* [kafka](https://kafka.apache.org/quickstart)：telegraf收集上来的指标上报到kafka，用于提高系统吞吐量。
* [vector](https://vector.dev/docs/)：高性能传输管道，支持多种源和目标，本示例中用于消费kafka消息，转换后，写入到ClickHouse中。
* [clickhouse](https://clickhouse.com/docs/zh/): 列式存储数据库，适用于OLAP分析型场景。
* [superset](https://superset.apache.org/docs/intro)：轻量级BI工具，用于快速接入各种数据源实现数据展现。

<!--more-->

### 启动redis用于演示

```bash
PS D:\dev_app\Redis-x64-3.0.504> pwd

Path
----
D:\dev_app\Redis-x64-3.0.504


PS D:\dev_app\Redis-x64-3.0.504> .\redis-server.exe
[10572] 06 Apr 17:10:45.476 # Warning: no config file specified, using the default config. In order to specify a config file use D:\dev_app\Redis-x64-3.0.504\redis-server.exe /path/to/redis.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.0.504 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 10572
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

[10572] 06 Apr 17:10:45.483 # Server started, Redis version 3.0.504
[10572] 06 Apr 17:10:45.484 * DB loaded from disk: 0.001 seconds
[10572] 06 Apr 17:10:45.484 * The server is now ready to accept connections on port 6379
```


### telegraf

telegraf采集redis指标，https://github.com/influxdata/telegraf/tree/master/plugins/inputs/redis

可以看到主要是通过执行redis的info命令实现的

#### 配置

```properties
[global_tags]

[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = "0s"
  hostname = ""
  omit_hostname = false

[[inputs.redis]]
  servers = ["tcp://localhost:6379"]
```

#### 测试

将采集的redis指标打印到控制台

```bash
PS D:\dev_app\collectors\telegraf-1.22.0> .\telegraf.exe --config .\telegraf.properties --test --input-filter redis
2022-04-06T09:58:49Z I! Starting Telegraf 1.22.0
2022-04-06T09:58:49Z I! Loaded inputs: redis
2022-04-06T09:58:49Z I! Loaded aggregators:
2022-04-06T09:58:49Z I! Loaded processors:
2022-04-06T09:58:49Z W! Outputs are not used in testing mode!
2022-04-06T09:58:49Z I! Tags enabled: host=wangyt16577
> redis_cmdstat,command=info,host=wangyt16577,port=6379,replication_role=master,server=localhost calls=3i,usec=248i,usec_per_call=82.67 1649239129000000000
> redis_keyspace,database=db0,host=wangyt16577,port=6379,replication_role=master,server=localhost avg_ttl=0i,expires=0i,keys=46i 1649239129000000000
> redis,host=wangyt16577,port=6379,replication_role=master,server=localhost aof_current_rewrite_time_sec=-1i,aof_enabled=0i,aof_last_bgrewrite_status="ok",aof_last_rewrite_time_sec=-1i,aof_last_write_status="ok",aof_rewrite_in_progress=0i,aof_rewrite_scheduled=0i,blocked_clients=0i,client_biggest_input_buf=0i,client_longest_output_list=0i,clients=1i,cluster_enabled=0i,connected_slaves=0i,evicted_keys=0i,expired_keys=0i,instantaneous_input_kbps=0,instantaneous_ops_per_sec=0i,instantaneous_output_kbps=0,keyspace_hitrate=0,keyspace_hits=0i,keyspace_misses=0i,latest_fork_usec=0i,loading=0i,lru_clock=5071960i,master_repl_offset=0i,mem_fragmentation_ratio=0.92,migrate_cached_sockets=0i,pubsub_channels=0i,pubsub_patterns=0i,rdb_bgsave_in_progress=0i,rdb_changes_since_last_save=0i,rdb_current_bgsave_time_sec=-1i,rdb_last_bgsave_status="ok",rdb_last_bgsave_time_sec=-1i,rdb_last_save_time=1649236245i,rdb_last_save_time_elapsed=2884i,redis_version="3.0.504",rejected_connections=0i,repl_backlog_active=0i,repl_backlog_first_byte_offset=0i,repl_backlog_histlen=0i,repl_backlog_size=1048576i,sync_full=0i,sync_partial_err=0i,sync_partial_ok=0i,total_commands_processed=3i,total_connections_received=4i,total_net_input_bytes=92i,total_net_output_bytes=5843i,uptime=2883i,used_cpu_sys=0.13,used_cpu_sys_children=0,used_cpu_user=0.08,used_cpu_user_children=0,used_memory=757248i,used_memory_lua=36864i,used_memory_peak=757248i,used_memory_rss=698688i 1649239129000000000
PS D:\dev_app\collectors\telegraf-1.22.0>
```

注意指标的格式：

`redis_cmdstat,command=info,host=myhost,port=6379,replication_role=master,server=localhost calls=3i,usec=248i,usec_per_call=82.67 1649239129000000000`

分为了四部分，对等的json格式为：

```json
{
	"fields": {
		"calls": 3,
		"usec": 248,
		"usec_per_call": 82.67
	},
	"name": "redis_cmdstat",
	"tags": {
		"command": "info",
		"host": "myhost",
		"port": "6379",
		"replication_role": "master",
		"server": "localhost"
	},
	"timestamp": 1649239129
}
```

### kafka

#### 启动

##### 启动kafka内置的zookeeper

```bash
PS D:\dev_app\kafka_2.13-3.1.0> ls


    目录: D:\dev_app\kafka_2.13-3.1.0


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/1/12     17:07                bin
d-----         2022/1/12     17:07                config
d-----         2022/1/12     17:07                libs
d-----         2022/1/12     17:07                licenses
d-----          2022/4/6     18:33                logs
d-----         2022/1/12     17:07                site-docs
d-----         2022/3/31     16:19                tmp
-a----         2022/1/12     17:01          14536 LICENSE
-a----         2022/1/12     17:01          28184 NOTICE


PS D:\dev_app\kafka_2.13-3.1.0> ./bin/windows/zookeeper-server-start.bat ./config/zookeeper.properties
```

##### 启动kafka

```bash
PS D:\dev_app\kafka_2.13-3.1.0> ./bin/windows/kafka-server-start.bat ./config/server.properties
```

##### 创建一个topic

```bash
## 创建一个topic
PS D:\dev_app\kafka_2.13-3.1.0> ./bin/windows/kafka-topics.bat --create --topic telegraf --bootstrap-server localhost:9092
Created topic telegraf.
```

```bash
## 列出所有topic
PS D:\dev_app\kafka_2.13-3.1.0> ./bin/windows/kafka-topics.bat --list --bootstrap-server localhost:9092
telegraf
```

```bash
## 查看topic详情
PS D:\dev_app\kafka_2.13-3.1.0> ./bin/windows/kafka-topics.bat --describe --topic telegraf --bootstrap-server localhost:9092
Topic: telegraf TopicId: 7rVMZCJXTSa2Gp8opDA_Uw PartitionCount: 1       ReplicationFactor: 1    Configs: segment.bytes=1073741824
        Topic: telegraf Partition: 0    Leader: 0       Replicas: 0     Isr: 0
PS D:\dev_app\kafka_2.13-3.1.0>
```

##### telegraf配置

```properties
[[outputs.kafka]]
  brokers = ["localhost:9092"]
  topic = "telegraf"
  data_format = "json"
  json_timestamp_units = "1ms"
```

### vector

在本示例中，我们使用vector消费kafka中的数据，输出给clickhouse。

#### 快速上手vector

先通过几个小示例感受下vector：

```bash
PS D:\dev_app\collectors\vector-0.20.0> pwd

Path
----
D:\dev_app\collectors\vector-0.20.0


PS D:\dev_app\collectors\vector-0.20.0> ls


    目录: D:\dev_app\collectors\vector-0.20.0


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/3/28     16:24                bin
d-----          2022/4/1     10:56                config
d-----         2022/3/28     16:24                data
-a----         2022/2/10     23:54          16811 LICENSE.txt
-a----          2022/2/9     18:57          17760 README.md.txt
```

config目录下增加一个配置文件myconfig.toml内容如下：
表示从标准输入设备接收，输出到控制台

```properties
[sources.in]
type = "stdin"

[sinks.out]
inputs = ["in"]
type = "console"
encoding.codec = "json"
```

执行测试：

```bash
PS D:\dev_app\collectors\vector-0.20.0> echo 'Hello world!' | ./bin/vector.exe --config ./config/myconfig.toml
2022-04-06T11:13:23.093723Z  INFO vector::app: Log level is enabled. level="vector=info,codec=info,vrl=info,file_source=info,tower_limit=trace,rdkafka=info,buffers=info"
2022-04-06T11:13:23.095105Z  INFO vector::app: Loading configs. paths=["config\\myconfig.toml"]
2022-04-06T11:13:23.100891Z  INFO vector::sources::stdin: Capturing STDIN.
2022-04-06T11:13:23.101194Z  INFO vector::topology::running: Running healthchecks.
2022-04-06T11:13:23.101366Z  INFO vector::topology::builder: Healthcheck: Passed.
2022-04-06T11:13:23.101421Z  INFO vector::topology::running: Starting source. key=in
2022-04-06T11:13:23.101652Z  INFO vector::topology::running: Starting sink. key=out
2022-04-06T11:13:23.101947Z  INFO vector: Vector has started. debug="false" version="0.20.0" arch="x86_64" build_id="2a706a3 2022-02-10"
2022-04-06T11:13:23.102098Z  INFO vector::app: API is disabled, enable by setting `api.enabled` to `true` and use commands like `vector top`.
2022-04-06T11:13:23.102138Z  INFO source{component_kind="source" component_id=in component_type=stdin component_name=in}: vector::sources::stdin: Finished sending.
2022-04-06T11:13:23.102229Z  INFO vector::shutdown: All sources have finished.
2022-04-06T11:13:23.102394Z  INFO vector: Vector has stopped.
{"host":"wangyt16577","message":"Hello world!\r","source_type":"stdin","timestamp":"2022-04-06T11:13:23.101995900Z"}
PS D:\dev_app\collectors\vector-0.20.0>
```

快速入门可参考：https://vector.dev/docs/setup/quickstart/

#### vector配置

clickhouse中要事先创建好相关表，见下节。

```bash
### 消费kafka ###
[sources.kafka_source_001]
type = "kafka"
bootstrap_servers = "localhost:9092"
group_id = "consumer-group-name"
key_field = "message_key"
topics = [ "telegraf" ]

### json字符串转成json对象 ###
[transforms.transforms_parse_json]
type = "remap"
inputs = [ "kafka_source_001" ]
source = '''
. = parse_json!(.message)
'''

### redis_cmdstat 类型数据处理 ###

## 过滤出redis_cmdstat类型数据
[transforms.redis_cmdstat]
type = "filter"
inputs = [ "transforms_parse_json" ]
condition = { type = "vrl", source = '.name == "redis_cmdstat"' }

## 将redis_cmdstat类型数据转换成一层json结构
[transforms.redis_cmdstat_remap]
type = "remap"
inputs = [ "redis_cmdstat" ]
source = '''
.fields.timestamp = .timestamp
.fields.name = .name
.fields.command = .tags.command
.fields.host = .tags.host
.fields.port = .tags.port
.fields.replication_role = .tags.replication_role
.fields.server = .tags.server
. = .fields
'''

## 转换好的数据输出到clickhouse中
[sinks.redis_cmdstat_out]
type = "clickhouse"
inputs = [ "redis_cmdstat_remap" ]
database = "mydb"
endpoint = "http://localhost:8123"
auth.strategy = "basic"
auth.password = "password"
auth.user = "default"
table = "t_redis_cmd_stat"
compression = "gzip"


### redis_keyspace ###

[transforms.redis_keyspace]
type = "filter"
inputs = [ "transforms_parse_json" ]
condition = { type = "vrl", source = '.name == "redis_keyspace"' }

[transforms.redis_keyspace_remap]
type = "remap"
inputs = [ "redis_keyspace" ]
source = '''
.fields.timestamp = .timestamp
.fields.name = .name
.fields.database = .tags.database
.fields.host = .tags.host
.fields.port = .tags.port
.fields.replication_role = .tags.replication_role
.fields.server = .tags.server
. = .fields
'''


[sinks.redis_keyspace_out]
type = "clickhouse"
inputs = [ "redis_cmdstat_remap" ]
database = "mydb"
endpoint = "http://localhost:8123"
auth.strategy = "basic"
auth.password = "password"
auth.user = "default"
table = "t_redis_cmd_stat"
compression = "gzip"

### redis ###

[transforms.redis]
type = "filter"
inputs = [ "transforms_parse_json" ]
condition = { type = "vrl", source = '.name == "redis"' }

[transforms.redis_remap]
type = "remap"
inputs = [ "redis" ]
source = '''
.fields.timestamp = .timestamp
.fields.name = .name
.fields.host = .tags.host
.fields.port = .tags.port
.fields.replication_role = .tags.replication_role
.fields.server = .tags.server
. = .fields
'''

[sinks.redis_out]
type = "clickhouse"
inputs = [ "redis_cmdstat_remap" ]
database = "mydb"
endpoint = "http://localhost:8123"
auth.strategy = "basic"
auth.password = "password"
auth.user = "default"
table = "t_redis_cmd_stat"
compression = "gzip"
```

### clickhouse

#### 创建表语句

t_redis_keyspace_stat

```sql
CREATE TABLE t_redis_keyspace_stat
(
    `timestamp` DateTime64(3, 'Asia/Shanghai'),
    `host` String,
    `server` String,
    `port` String,
    `replication_role` String,
    `name` String,
    `database` String,
    `avg_ttl` Float64,
    `expires` Float64,
    `keys` Float64
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp)
;
```

t_redis_cmd_stat

```sql
CREATE TABLE t_redis_cmd_stat
(
    `timestamp` DateTime64(3, 'Asia/Shanghai'),
    `host` String,
    `server` String,
    `port` String,
    `replication_role` String,
    `name` String,
    `command` String,
    `calls` Float64,
    `usec` Float64,
    `usec_per_call` Float64
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp)
;
```

t_redis_stat

```sql
CREATE TABLE t_redis_stat
(
    `timestamp` DateTime64(3, 'Asia/Shanghai'),
    `host` String,
    `port` String,
    `server` String,
    `replication_role` String,
    `name` String,
    `aof_current_rewrite_time_sec` Float64,
    `aof_enabled` Float64,
    `aof_last_bgrewrite_status` String,
    `aof_last_rewrite_time_sec` Float64,
    `aof_last_write_status` String,
    `aof_rewrite_in_progress` Float64,
    `aof_rewrite_scheduled` Float64,
    `blocked_clients` Float64,
    `client_biggest_input_buf` Float64,
    `client_longest_output_list` Float64,
    `clients` Float64,
    `cluster_enabled` Float64,
    `connected_slaves` Float64,
    `evicted_keys` Float64,
    `expired_keys` Float64,
    `instantaneous_input_kbps` Float64,
    `instantaneous_ops_per_sec` Float64,
    `instantaneous_output_kbps` Float64,
    `keyspace_hitrate` Float64,
    `keyspace_hits` Float64,
    `keyspace_misses` Float64,
    `latest_fork_usec` Float64,
    `loading` Float64,
    `lru_clock` Float64,
    `master_repl_offset` Float64,
    `mem_fragmentation_ratio` Float64,
    `migrate_cached_sockets` Float64,
    `pubsub_channels` Float64,
    `pubsub_patterns` Float64,
    `rdb_bgsave_in_progress` Float64,
    `rdb_changes_since_last_save` Float64,
    `rdb_current_bgsave_time_sec` Float64,
    `rdb_last_bgsave_status` String,
    `rdb_last_bgsave_time_sec` Float64,
    `rdb_last_save_time` Float64,
    `rdb_last_save_time_elapsed` Float64,
    `redis_version` String,
    `rejected_connections` Float64,
    `repl_backlog_active` Float64,
    `repl_backlog_first_byte_offset` Float64,
    `repl_backlog_histlen` Float64,
    `repl_backlog_size` Float64,
    `sync_full` Float64,
    `sync_partial_err` Float64,
    `sync_partial_ok` Float64,
    `total_commands_processed` Float64,
    `total_connections_received` Float64,
    `total_net_input_bytes` Float64,
    `total_net_output_bytes` Float64,
    `uptime` Float64,
    `used_cpu_sys` Float64,
    `used_cpu_sys_children` Float64,
    `used_cpu_user` Float64,
    `used_cpu_user_children` Float64,
    `used_memory` Float64,
    `used_memory_lua` Float64,
    `used_memory_peak` Float64,
    `used_memory_rss` Float64
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp)
;
```







