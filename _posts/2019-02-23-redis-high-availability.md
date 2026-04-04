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
bind 192.168.0.31
```

3: 配置 Sentinel 

1：在每台主机上准备一个配置文件 `sentinel.conf`

```sh
bind 192.168.88.185
port 26379
daemonize yes
logfile "sentinel.log"
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


## 配置 Redis Cluster

### 手动配置

**1）配置文件**

```sh

$ cat conf/c6380.conf

include /home/chengchao/apps/redis/conf/redis.conf

daemonize yes
pidfile "/home/chengchao/apps/redis/redis-c6380.pid"
port 6380
bind c01
logfile "/home/chengchao/apps/redis/logs/redis-c6380.log"
dbfilename "dump-c6380.rdb"
dir "/home/chengchao/apps/redis/data"

cluster-enabled yes
cluster-config-file nodes-c6380.conf
cluster-node-timeout 10000

```

- `cluster-enabled yes` 启用集群功能


> **注意**
>
> 当 Redis 集群运行时，每个节点会打开两个 TCP 套接字。第一个套接字是用于客户端连接的标准 Redis 通信协议；第二个套接字的端口号是第一个端口号加上 10000，被用作实例间信息交换的通信端口。值 10000 是硬编码的。因此，监听大于 55536 的端口是不能启动集群的。

```sh
[chengchao@c01 redis]$ sudo firewall-cmd --permanent --zone=public --add-port=6380/tcp
success
[chengchao@c01 redis]$ sudo firewall-cmd --permanent --zone=public --add-port=6381/tcp
success
[chengchao@c01 redis]$ sudo firewall-cmd --permanent --zone=public --add-port=16380/tcp
success
[chengchao@c01 redis]$ sudo firewall-cmd --permanent --zone=public --add-port=16381/tcp
success
[chengchao@c01 redis]$ sudo firewall-cmd --reload
success
```


**2）其他主机**


复制文件到其他主机，并编辑为各自主机的 ip 或者主机名，创建目录（或者清楚数据）。


```sh
[chengchao@c01 conf]$ scp c638* c02:~/apps/redis/conf/
c6380.conf                                                            100%   58KB  17.1MB/s   00:00    
c6381.conf                                                            100%   58KB  19.5MB/s   00:00    
[chengchao@c01 conf]$ scp c638* c03:~/apps/redis/conf/
c6380.conf                                                            100%   58KB  16.4MB/s   00:00    
c6381.conf                                                            100%   58KB  20.5MB/s   00:00    
[chengchao@c01 conf]$ cd ..
[chengchao@c01 redis]$ pwd
/home/chengchao/apps/redis
[chengchao@c01 redis]$ mkdir -p {logs,data}
```

**3）启动各服务器上的 Redis**


```sh
[chengchao@c01 redis]$ bin/redis-server conf/c6380.conf
[chengchao@c01 redis]$ bin/redis-server conf/c6381.conf  
[chengchao@c01 redis]$ ps -ef | grep redis
chengch+  5731     1  0 20:32 ?        00:00:00 bin/redis-server c01:6380 [cluster]
chengch+  5736     1  0 20:33 ?        00:00:00 bin/redis-server c01:6381 [cluster]
chengch+  5755  5523  0 20:41 pts/1    00:00:00 grep --color=auto redis
[chengchao@c01 redis]$ echo '切换到另外两台主机上执行'
[chengchao@c02 redis]$ bin/redis-server conf/c6380.conf
[chengchao@c02 redis]$ bin/redis-server conf/c6381.conf 
[chengchao@c02 redis]$ ps -ef | grep redis
chengch+  5602     1  0 20:39 ?        00:00:00 bin/redis-server c02:6380 [cluster]
chengch+  5607     1  0 20:39 ?        00:00:00 bin/redis-server c02:6381 [cluster]
chengch+  5612  5442  0 20:40 pts/0    00:00:00 grep --color=auto redis
[chengchao@c03 redis]$ bin/redis-server conf/c6380.conf
[chengchao@c03 redis]$ bin/redis-server conf/c6381.conf 
[chengchao@c03 redis]$ ps -ef | grep redis
chengch+  5576     1  0 20:39 ?        00:00:00 bin/redis-server c03:6380 [cluster]
chengch+  5581     1  0 20:39 ?        00:00:00 bin/redis-server c03:6381 [cluster]
chengch+  5587  5447  0 20:40 pts/1    00:00:00 grep --color=auto redis
```

Redis 实例启动后，将会在 data 目录下生成节点配置文件。文件的内容如下：


```sh
[chengchao@c03 redis]$ ls data
nodes-c6380.conf  nodes-c6381.conf
[chengchao@c03 redis]$ cat data/nodes-c6380.conf 
d36a466e602979e614db25e48a8c62e7ae128b9a :0@0 myself,master - 0 0 0 connected
vars currentEpoch 0 lastVoteEpoch 0

```

看一下 Redis 信息

```sh
[chengchao@c03 redis]$ bin/redis-cli -h c01 -p 6380 info cluster
# Cluster
cluster_enabled:1

```

**4）使 Redis 实例彼此发现**

```sh
[chengchao@c01 redis]$ bin/redis-cli -h c01 -p 6380 CLUSTER MEET c01 6380
(error) ERR Invalid node address specified: c01:6380
[chengchao@c01 redis]$ bin/redis-cli -h c01 -p 6380 CLUSTER MEET 192.168.88.185 6380
OK
[chengchao@c01 redis]$ bin/redis-cli -h c01 -p 6380 CLUSTER MEET 192.168.88.153 6380
OK
[chengchao@c01 redis]$ bin/redis-cli -h c01 -p 6380 CLUSTER MEET 192.168.88.238 6380
OK
[chengchao@c01 redis]$ bin/redis-cli -h c01 -p 6380 CLUSTER MEET 192.168.88.185 6381
OK
[chengchao@c01 redis]$ bin/redis-cli -h c01 -p 6380 CLUSTER MEET 192.168.88.153 6381
OK
[chengchao@c01 redis]$ bin/redis-cli -h c01 -p 6380 CLUSTER MEET 192.168.88.238 6381
OK

```

**5）进行数据槽（slot）的分配**


Redis 集群中，数据被按照以下的算法分布到 16384 个哈希槽中：

```
    HASH_SLOT = CRC16(key) mod 16384
```

在一台主机上使用 `redis-cli`，通过指定主机和端口号的方式进行：


```sh
$ for i in {0..5400}; do
    bin/redis-cli -h 192.168.88.185 -p 6380 CLUSTER ADDSLOTS $i;
done

...

$ for i in {5401..11000}; do
    bin/redis-cli -h 192.168.88.153 -p 6380 CLUSTER ADDSLOTS $i;
done

...

$ for i in {11001..16383}; do
    bin/redis-cli -h  192.168.88.238 -p 6380 CLUSTER ADDSLOTS $i;
done
```


**6）实现数据复制**

看一下集群信息：


```sh
[chengchao@c01 redis]$ bin/redis-cli -h c01 -p 6380 CLUSTER NODES
e2aec6b51860529d46afa35db654c6acc4adbfe7 192.168.88.153:6380@16380 master - 0 1551186610000 0 connected 5401-11000
d36a466e602979e614db25e48a8c62e7ae128b9a 192.168.88.238:6380@16380 master - 0 1551186610000 2 connected 11001-16383
50532c18ed3d2768ef63b2448242b6a23c778d73 192.168.88.238:6381@16381 master - 0 1551186609000 0 connected
4016c5369654963c7db2476d43fee67dff03c614 192.168.88.185:6381@16381 master - 0 1551186612000 3 connected
a693c0b2192440056b229d8c4e5ea70ec24c2a4d 192.168.88.153:6381@16381 master - 0 1551186612936 4 connected
3c9e8ffa68b16442edb1689ce7a4efe441b7a169 192.168.88.185:6380@16380 myself,master - 0 1551186611000 1 connected 0-5400
```

选择三个节点作为从节点

```sh

$ bin/redis-cli -h c01 -p 6381 CLUSTER REPLICATE e2aec6b51860529d46afa35db654c6acc4adbfe7
$ bin/redis-cli -h c02 -p 6381 CLUSTER REPLICATE d36a466e602979e614db25e48a8c62e7ae128b9a
$ bin/redis-cli -h c03 -p 6381 CLUSTER REPLICATE 3c9e8ffa68b16442edb1689ce7a4efe441b7a169
$ echo "再看一下集群信息 ..."
$ bin/redis-cli -h c01 -p 6380 CLUSTER NODES
e2aec6b51860529d46afa35db654c6acc4adbfe7 192.168.88.153:6380@16380 master - 0 1551187012000 0 connected 5401-11000
d36a466e602979e614db25e48a8c62e7ae128b9a 192.168.88.238:6380@16380 master - 0 1551187014000 2 connected 11001-16383
50532c18ed3d2768ef63b2448242b6a23c778d73 192.168.88.238:6381@16381 slave 3c9e8ffa68b16442edb1689ce7a4efe441b7a169 0 1551187012000 5 connected
4016c5369654963c7db2476d43fee67dff03c614 192.168.88.185:6381@16381 slave e2aec6b51860529d46afa35db654c6acc4adbfe7 0 1551187012416 3 connected
a693c0b2192440056b229d8c4e5ea70ec24c2a4d 192.168.88.153:6381@16381 slave d36a466e602979e614db25e48a8c62e7ae128b9a 0 1551187014421 4 connected
3c9e8ffa68b16442edb1689ce7a4efe441b7a169 192.168.88.185:6380@16380 myself,master - 0 1551187011000 1 connected 0-5400
```

**7）测试一下**


```sh
[chengchao@c01 redis]$ bin/redis-cli -c -h c01 -p 6380
c01:6380> dbsize
(integer) 0
c01:6380> set foo bar
-> Redirected to slot [12182] located at 192.168.88.238:6380
OK
192.168.88.238:6380> get foo
"bar"
192.168.88.238:6380> 

```


<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
