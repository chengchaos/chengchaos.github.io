---
title: Redis-高可用和集群
key: 20190223
tags: redis HA redis-cluster
---

我们一起学习配置和管理 Redis 的 Sentinel 和 Redis Cluster。

使用 Redis 的主从复制和持久化可以解决数据冗余备份的问题。但是如果没有人工干预，当主实例宕机，整个 Redis 服务将无法恢复。自 2.6 版本以后，Redis 原生支持的 Sentinel 是使用最广泛的高可用架构。利用 Sentinel 我们可以轻松地构建具备容错能力的 Redis 服务。

另外，由于 Redis 中所存储的数据增长速度都很快，一个存储了大量数据（通常在16G以上）的 Redis 实例的处理能力和内存容量可能会变成应用的瓶颈。随着 Redis 中数据集大小的增长，在进行持久化或主从复制时，也会越来越多地出现如延迟等问题，对于这种情况，水平扩展或者通过向 Redis 服务中增加更多节点来实现伸缩就成了一种迫切需要。从 3.0 版本开始支持的 Redis Cluster 是针对这类问题的解决方案，可以将数据通过分区的方式分布到多个 Redis 主实例中。


<!--more-->

## 配置 Sentinel

顾名思义，Sentinel（哨兵）充当了 Redis 主从实例的守卫着。

因为单个哨兵本身也可能失效，所以一个哨兵显然不足以保证高可用，对主实例进行故障迁移的决策是基于仲裁系统的，所以至少需要三个哨兵进程才能构成一个健壮的分布式系统来持续地监控 Redis 主实例的状态。

如果有多个哨兵进程监测到主实例下线，其中的一个哨兵进程会被选举出来负责推选一个从实例替代原有的主实例。如果配置恰当，上述的整个过程都是自动化的。

### 准备工作

1: 首先准备一主二从三个实例

| 角色 | IP | 端口 |
| ---- | ---- | ---- |
| Master | 192.168.0.31 | 6379 |
| Slave1 | 192.168.0.32 | 6379 |
| Slave2 | 192.168.0.33 | 6379 |
| Sentinel1 | 192.168.0.31 | 26379 |
| Sentinel2 | 192.168.0.32 | 26379 |
| Sentinel3 | 192.168.0.33 | 26379 |


2: 需要在 Redis 配置文件中正确地设置绑定的 IP 地址：

```sh
bind 127.0.0.1 192.168.0.31
```

3: 配置 Sentinel 

1：在每台主机上准备一个配置文件 `sentinel.conf`

```sh
port 26379
dir /tmp
sentinel monitor mymaster 192.168.0.31 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000

```

2：在三台主机上启动 Sentinel 进程

```sh

$ bin/redis-server conf/sentinel.conf --sentinel
```






<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
