---
title: Redis-复制
key: 20190223
tags: redis replication
---

生产环境中的 Redis 单实例也会遇到注入系统崩溃、网络连接闪断或者突然断电等单点故障问题，Redis 也提供了一个复制机制，使得数据能够从一个 Redis 主服务器复制到一个或多个其他 Redis 服务器中。（Master/Slave 不过现在不允许这么叫了，会让有些人想起奴隶制）

<!--more-->

复制不仅仅提高了整个系统的容错能力，还可以用于对系统进行水平扩展。在一个读取多于写入的应用中，我们可以通过增加多个 Redis 只读从实例来减轻主实例的压力。

Redis 的复制机制是 RedisCluster 的基础，RedisCluster 在此基础上提供了高可用性。

## 配置 Redis 的复制机制

参与复制的 Redis 实例划分为主节点（master）和从节点（slave）。默认情况下，Redis 都是主节点。每个从节点只能有一个主节点，而主节点可以同时具有多个从节点。

复制的数据流是单向的，只能由主节点复制到从节点。

配置复制的方式有 3 中：

- 1） 在配置文件中加入 `slaveof {masterHost} {masterPort}`
- 2） 在 redis-server 启动命令后加入 `--slaveof {masterHost} {masterPort}`
- 3） 直接使用命令： `slaveof {masterHost} {masterPort}`


使用配置文件的方式测试一下：准备一份配置文件，可以将 redis.conf 复制一份并重命名为 redis-slave.conf 然后进行修改：

```sh
port 6380
pidfile /var/run/redis_6380.pid
dir ./slave
slaveof 127.0.0.1 6379
```

**启动主实例**

```sh
$ cd /work/apps/redis4
$ bin/redis-server conf/6379.conf
```

**启动从实例**

```sh
$ cd /work/apps/redis4
$ bin/redis-server conf/redis-slave.conf

```

分别用 redis-cli 连接到两个实例上面，执行 `info replication`:


6379:


```sh
chengchao@t420i:~$ cd /work/apps/redis4/
chengchao@t420i:redis4$ bin/redis-cli -p 6379
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=294,lag=1
master_replid:10432c0e133d08cfa64d65098a2ba6993aa07e26
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:294
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:294
127.0.0.1:6379> 

```

6380:

```sh
chengchao@t420i:~$ cd /work/apps/redis4/
chengchao@t420i:redis4$ bin/redis-cli -p6380
Unrecognized option or bad number of args for: '-p6380'
chengchao@t420i:redis4$ bin/redis-cli -p 6380
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:9
master_sync_in_progress:0
slave_repl_offset:294
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:10432c0e133d08cfa64d65098a2ba6993aa07e26
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:294
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:294
127.0.0.1:6380> 

```

此时在主实例中设置 key 和 value 会同步到从实例中。

从实例默认是只读的 （`slave-read-only=yse`），不可以设置 key 和 value。

关闭从实例，在主实例上设置一个新的 key 和 value，重新启动从实例，新的 key 和 value 也会被同步过来。


`slaveof` 命令不但可以建立复制，还可以在从节点执行 `slaveof no one` 来断开与主节点复制关系。从节点断开复制后不会抛弃原有数据，只是无法获取主节点上的数据变化。流程如下：

- 断开与主节点复制关系
- 从节点晋升为主节点

`slaveof` 命令还可以实现切主操作，即把当前从节点对主节点的复制切换到另一个主节点。执行 `slaveof {masterHost} {masterPort}` 即可。工作流程如下：

- 断开与旧主节点复制关系
- 与新主节点建立复制关系
- 删除从节点当前所有数据
- 对新节点进行复制操作




### 安全性

对于数据比较重要的节点，主节点会通过设置 `requriepass` 参数进行密码验证，这是所有客户端必须使用 `AUTH` 命令进行校验。从节点与主节点的复制连接是通过一个特殊标识的客户端来完成的，因此需要从节点的 `masterauth` 参数与主节点的密码保持一致，这样从节点才可以正确地连接到主节点并发起复制流程。



### 工作原理


配置文件 redis-slave.conf 的配置添加：`slaveof 127.0.0.1 6379` 表示此实例是 127.0.0.1:6379 的一个从实例。

`info replication` 命令的输出中，master_replid 是主实例启动时生成的随机字符串，用来表示主实例。master_repl_offset 是复制流中的一个偏移标记，会锁着主实例上数据时间的发生而增长。

当主从实例之间网络连接通畅并且建立了复制关系后，主实例会把其接收到的写入命令转发给从实例执行，以实现主从实例之间的数据同步。

在 Redis 的复制机制中，共有两种重新同步机制：部分重新同步和完全重新同步。

当一个 Redis 的从实例启动并连接到主实例时，从实例总会尝试通过发送 `master_replid;master_repl_offset` 请求进行部分重新同步。其中 `master_replid;master_repl_offset` 表示与主实例同步的最后一个快照。如果主实例接受部分重新同步的请求，那么它会从从实例停止时的最后一个偏移处开始增量地进行命令同步。否则，则需要进行完全重新同步。当从实例第一次连接到它的主实例时，总是需要进行完全重新同步。我们将在后续复制机制的调优案例中对有关主实例如何决定是否接受部分重新同步请求的细节进行讨论。在进行完全重新同步时，为了将所有的数据复制到从实例中，主实例需要将数据转储到一个 RDB 文件中，然后将这个文件发送给从实例。从实例接收到 RDB 文件后，会先将内存中的所有数据清除，再将 RDB 文件中的数据导入。主实例上的复制过程是完全异步的，因此并不会阻塞服务器处理客户端的请求。

### 更多细节

当一个从实例被提升为主实例时，其他的从实例必须与新主实例重新同步。在 4.0 之前，因为 master_replid 发生了改变，所以这个过程是一个完全的重新同步。在 4.0 之后，新主实例会记录旧主实例的 master_replid 和 offset，因此能够接受来自其他从实例的部分重新同步。

## 复制机制的调优


### 复制积压缓冲区

在 Redis 主实例失去与从实例的网络连接期间，主实例上的一段内存（实际是一个环形缓冲区）会跟踪最近所有的写入命令，这个缓冲区实际上是一个固定长度的列表。

在 Redis 中，我们将这个缓冲区称为 **`replication backlog`**。Redis 使用这个 backlog 缓冲区来决定是进行完全重新同步还是部分重新同步。更具体的说：在发出 slaveof 命令后，从实例使用最后一个 offset 和最后一个主实例 ID 向主实例发送一个部分重新同步请求。当主实例和从实例之间的连接建立后，主实例首先会检查请求中的 master_replid 是否与他自己的 master_replid 一致。然后主实例会检查请求中的 offset 能否从 backlog 缓冲区中获取。如果 offset 位于 backlog 的范围内，那么就从中获得连接断开期间的所有写入命令。否则，部分重新同步请求会被拒绝，此时完全重新同步将被启动。

默认情况下，backlog 缓冲区的大小是 1MB。


我们可以通过在峰值期间使用 `INFO` 命令计算 `master_repl_offset` 的变化量，估算 backlog 缓冲区的合适大小，我们也可以用这个公式来估算主从实例间的网络流量。

> t 是可能发生的最长断开连接的时间（秒）

```
t * (master_repl_offset2 - master_repl_offset1) / (t2 - t1)
```



一般来说，将这个值设置为比 RDB 快照还大是没有意义的。

有关 backlog 的另一个参数是 `repl-backlog-ttl`， 此参数的含义是：如果所有的从实例与主实例的连接全部断开，那么主实例等待多久释放 backlog 占用的内存。默认值是 3600 秒。

### 网络

从网络传输角度看，可以通过将参数 `repl-disable-tcp-nodelay` 设置为 `yes` 来减少带宽的使用。如果设为 yes，Redis 会尝试将价格小包合并成一个包发送，这在主从实例未知相距较远是有用，需要注意的是，这个选项可能造成约 40 毫秒的复制延迟。

### I/O 和内存


从 I/O 角度和内存角度看，可以通过使用付cup按复制直接将 RDB 的内容发送给从实例而无需在磁盘上创建 RDB 文件。这种机制可以在 RDB 快照过程中节省大量的 I/O 和内存。如果 Redis 所在的主机上的磁盘速度不快或内存够使用率很高，而网络带宽有足够，可以考虑打开这个选项，通过把 `repl-diskless-sync` 设置为 `yes` 来讲复制机制从默认的 `disk-backed` 切换为 `diskless` 。但是这个特性目前处于试验阶段。




<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
