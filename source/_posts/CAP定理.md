---
title: CAP定理
date: 2019-7-24 00:01:45
categories:
- 分布式
tags:
- CAP
---

#### 基本内容

* Consistency: all nodes see the same data at the same time. 即所有的节点在同一时刻读到同样的数据。

* Availability: a guarantee that every request receives a response about whether it was successful or failed. 请求无论成功还是失败，都能收到一个响应。

* Partition-Tolerance: the system continues to operate despite arbitrary message loss or failure of part of the system. 系统仍能运行尽管部分节点出问题或者丢失消息

#### CAP选择

分布式系统中不能同时满足C、A、P

* CA，加强一致性&可用性，放弃分区容忍性，例如传统单机数据库;

* AP，加强可用性&分区容忍性，放弃强一致性，例如大多数NoSQL系统;

* CP，加强一致性&分区容忍性，可用性比较差，例如Zookeeper; 
 
  如果ZooKeeper集群中出现了网络分割的故障（注：由于交换机故障导致交换机底下的子网间不能互访）；那么ZooKeeper会将它们都从自己管理范围中剔除出去，外界就不能访问到这些节点了，即便这些节点本身是“健康”的，可以正常提供服务的；所以导致到达这些节点的请求被丢失了。
  
#### BASE理论

在分布式系统中，一般选择加强可用性和分区容忍性而牺牲一致性。

* Basically Available：基本可用，允许分区失败；

* Soft state：软状态，接受一段时间的状态不同步；

* Eventually consistent：最终一致，保证最终的数据状态是一致的。

  在没有发生故障的前提下，数据达到一致状态的时间延迟，取决于网络延迟，系统负载和数据复制方案设计等因素。
  
More info: [系统架构设计理论与原则、负载均衡及高可用系统设计速记](https://www.cnblogs.com/jeffwongishandsome/p/talk-about-arch-design-and-load-balance-and-hign-availability-in-system-architecture.html)