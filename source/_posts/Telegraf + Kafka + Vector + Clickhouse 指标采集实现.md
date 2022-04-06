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

### 简介

* [telegraf](https://docs.influxdata.com/telegraf/v1.22/)：指标收集工具，可收集中间件暴露的性能指标，本文中我们以收集redis指标为例。
* [kafka](https://kafka.apache.org/quickstart)：telegraf收集上来的指标上报到kafka，用于提高系统吞吐量。
* [vector](https://vector.dev/docs/)：高性能传输管道，支持多种源和目标，本示例中用于消费kafka消息，转换后，写入到ClickHouse中。
* [clickhouse](https://clickhouse.com/docs/zh/): 列式存储OLAP数据库，适用于分析型场景。
* [superset](https://superset.apache.org/docs/intro)：轻量级BI工具，用于快速接入各种数据源实现数据展现。

<!--more-->


