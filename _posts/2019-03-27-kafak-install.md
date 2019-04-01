---
title: kafka 的安装
key: 20190327
tags: kafka
---

记录一下 Kafka 的安装

<!--more-->

## 操作系统

我们使用 CentOS 7

## Java

使用 JDK1.8+

## Zookeeper

Kafka 使用 Zookeeper 保存集群的元数据信息和消费者信息。Kafka 自带了一个 Zookeeper。


### Zookeeper 单机版

编辑 zoo.cfg

```bash
$ cd /works/apps/kafka/
$ cd config
$ cat > zoo.cfg << EOF
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
EOF
$ cd ..
$ bin/zookeeper-server-start.sh -daemon config/zoo.cfg 
```

测试 Zookeeper

```bash
$ telnet localhost 2181
Trying ::1...
Connected to localhost.
Escape character is '^]'.
srvr
Zookeeper version: 3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: standalone
Node count: 4
Connection closed by foreign host.
```
### Zookeeper 群组（Ensemble）

[待补充]


## Kafka Broker

解压，然后启动：

```bash
$ bin/kafka-server-start.sh 
USAGE: bin/kafka-server-start.sh [-daemon] server.properties [--override property=value]*
$ bin/kafka-server-start.sh config/server.properties 
```

### 测试 Kafka

**创建一个测试主题：**

```bash
$ bin/kafka-topics.sh --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 \
  --partitions 1 \
  --topic test
Created topic "test".

```

**在测试主题上发布消息：**

```bash
$ bin/kafka-console-producer.sh --broker-list localhost:9092 \
  --topic test
```

**从测试主题上读取消息：**

```bash
$ bin/kafka-console-consumer.sh --zookeeper localhost:2181 \
  --topic test --from-beginning
```


## Broker 配置

Kafka 有很多配置选项，不过大多数调优选项可以使用默认的配置。

### 常规配置

有一些选项，在单机安装时可以直接使用默认值，不过在部署到集群环境中则要做一些修改。

**1. `broker.id`**

每个 borker 都需要有一个标识符，使用 `broker.id` 来表示。默认值是 0，也可以被设置成其他任意整数。

这个值在整个 Kafka 集群里必须是唯一的。

**2. `port`**

默认 Kafka 监听 9092。

**2. `zookeeper.connect`**

通过 `zookeeper.connect` 属性指定 Kafka 用于保存 broker 元数据的 Zookeeper 地址。

值是逗号分隔的主机列表，例如：

```
zookeeper.connect=127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002
zookeeper.connect=localhost:2181/chengchao
```

使用 chroot 了的设置后，链接到kafka 的 zookeeper 也要指明 chroot 名称。

```bash
$ bin/kafka-topics.sh --create \
  --zookeeper localhost:2181/chengchao \
  --replication-factor 1 \
  --partitions 1 \
  --topic test 
Created topic "test".
$ 
$ bin/kafka-console-consumer.sh --zookeeper localhost:2181/chengchao\
  --topic test --from-beginning
```


**4. `log.dirs`**

Kafka 吧所有消息都保存在磁盘上，存放这些日志片段的目录是通过 `log.dirs` 指定的。

是一组逗号分隔的文件系统。

**5. `num.recovery.threads.per.data.dir`**

..


**6. `auto.create.topics.enable`**

默认情况下，一下情况会自动创建主题：

- 当一个生产者开始写入消息时
- 当一个消费者开始读取消息时
- 任意一个客户端向主题发送元数据请求时


### Topic 的默认配置


**1. `num.partitions`**

此参数指定了新创建的主题将包含多少个分区。（默认值 1）

如果启用了自动创建主题的功能（默认 true），主题的分区个数就是此参数指定的值。

我们可以增加主题分区的个数，但是不能减少分区的个数。因此，要让一个主题的分区少于设定的值，需要手动船舰主题。

Kafka 集群通过分区对主题进行横向扩展，所以当有新的 broker 加入集群时，可以通过分区个数来实现集群的负载均衡。

选定分区数量时，需要考虑：

- 主题需要达到多大的吞吐量？
- 单个分区读取数据的最大吞吐量是多少？
……

**2. `log.retention.ms`**

数据可以被保留多久。默认使用 `log.retention.hours` 参数配置时间，默认（168小时/7天）。

还有连个参数： `log.retention.ms`, `log.retention.minutes`。

如果指定了不止一个参数，Kafka 会优先使用具有最小值的那个。

**3. `log.retention.bytes`**

另一种设置数据是否过期的设置，其作用在每一个分区上，假设有一个包含 8 个分区的主题，并且 `log.retention.bytes` 设置为 1G，那么这个主题最多可以保留 8G 的数据。所以当主题的分区个数增加时，整个主题可以保留的数据也随之增加。

> 以上两个设置任意一个达到，消息就会被删除。


**4. `log.segment.bytes'**

当消息到达 broker 时，它们被追加到分区的当前日志片段上。

当日志片段大小达到 `log.segment.bytes` 指定的上限时（默认 1G）。当前日志片段会被关闭，一个新的日志片段被打开。

如果一个日志片段被关闭，就等待过期。

如果消息量不大，会增加日志过期时间。

**5. `log.segment.ms`**

设置日志片段多长时间后关闭。和上一个参数类似，这个使用时间计量单位。哪个先到达都关闭日志片段。

默认 `log.segment.ms` 没有设定值，所以只根据大小来关闭日志片段。

**6. `message.max.bytes`**

限制单个消息的大小，默认值是 100000 (1M)，超过设定的大小会返回错误消息。

消费客户端设置的 `fetch.message.max.bytes` 必须与服务器端设置的消息大小进行协调。如果比 `message.max.bytes` 小，会无法读取比较大的消息，导致出现消费者被阻塞的情况。

配置集群的 `replica.fetch.max.bytes` 参数时，也遵循同样的原则。

## Kafka 集群

### 需要多少个 broker

一个 Kafka 集群需要多少个 broker 需要考虑：

1. 需要多少磁盘来报了数据，单个 broker 有多少空间可用。

例如：整个集群需要报了 10T 的数据，每个 broker 可以保存 2TB，那么至少需要 5 个 broker。如果启用了数据复制，那么至少还需要一倍的空间。（10 个 brokker）。

2. 集群处理请求的能力。

### broker 配置


要把一个 broker 加入到集群中，需要修改两个配置参数：

1. 所有 broker 都必须配置相同的 `zookeeper.connect`，该参数指定了用于保存元数据的 zookeeper 群组和路径。

2. 别个 broker 都必须为 `broker.id` 参数设置唯一的值。

### 操作系统

**1. 虚拟内存**


```bash
# 尽量不要发生内存交换
vm.swappiness=1
# 设置为小于 10 可以减少脏页的数量
vm.dirty_background_ratio=5
# 增加内核进程刷新到磁盘之前的脏页数量
# 设置为大于 20 （60 -80）
vm.dirty_ratio=50

```

运行期间进行检查脏页的数量：

```bash

# cat /proc/vmstat | egrep "dirty|writeback"
```

**3. 网络**

```bash
# socket 读写缓冲区
# 合理值在 131072（128kb）
net.core.wmem_default=131072
net.core.rmem_default=131072
# 读写缓冲区最大值
# 合理值在 2097152 (2MB)
net.core.wmem_max=2097152
net.core.rmem_max=2097152
# TCP socket 的读写缓冲区
# 最小 默认 最大
# 最大不能大于 net.core.wmem_max
# 和 net.core.rmem_max 的设定
# 下面的设置是 4k 64k 2MB
net.ipv4.tcp_wmem=4096 65536 2048000
net.ipv4.tcp_rmem=4096 65536 2048000
# 启用 TCP 时间窗扩展
net.ipv4.tcp_window_scaling=1
# 接受更多的并发连接(默认 1024）
net.ipv4.tcp_max_syn_backlog=10240
# 应对网络流量的爆发(默认 1000)
# 允许更多的数据包排队等待内核处理
net.core.netdev_max_backlog=10240

```

## 生产环境的注意事项



### 垃圾回收选项

G1 垃圾回收器选项：

- `MaxGCPauseMillis` :指定每次垃圾回收默认的停顿时间。默认 200ms。

- `InitiatingHeapOccupancyPercent` :G1 启动前可以使用的堆内存百分比，默认 45 。Kfaka 容易产生垃圾对象，可以把这个值设置的小一些，例如 35

一个例子：

```bash
# export JAVA_HOME=/usr/java/jdk1.8.0_51
# export KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 \
  -XX:+DisableExplicitGC -Djava.awt.headless=true"
# /usr/local/kafka/bin/kafka-server-start.sh -daemon \
  /usr/local/kafka/config/server.properties
#
```

### 数据中心布局

最好把集群的broker 安装在不同的机架上，至少不要让它们共享可能出现单点故障的基础设施，比如电源和网络。也就是说，部署服务器需要至少两个电源连接（两个不同的回路）和两个网络交换器（保证可以进行无缝的故障切换）。除了这些以外，最好还要把broker 安放在不同的机架上。因为随着时间的推移，机架也需要进行维护，而这会导致机器离线（比如移动机器或者重新连接电源）。


### 共享Zookeeper

Kafka 使用 Zookeeper 来保存 broker、主题和分区的元数据信息。

对于一个包含多个节点的 Zookeeper 群组来说，Kafka 集群的这些流量并不算多，那些写操作只是用于构造消费者群组或集群本身。实际上，在很多部署环境里，会让多个Kafka 集群共享一个Zookeeper 群组（每个集群使用一个chroot 路径）。

虽然多个 Kafka 集群可以共享一个 Zookeeper 群组， 但如果有可能的话， 不建议把 Zookeeper 共享给其他应用程序。Kafka 对 Zookeeper 的延迟和超时比较敏感， 与 Zookeeper 群组之间的一个通信异常就可能导致 Kafka 服务器出现无法预测的行为。这样很容易让多个 broker 同时离线，如果它们 Zookeeper 之间断开连接，也会导致分区离线。这也会给集群控制器带来压力，在服务器离线一段时间之后，当控制器尝试关闭一个服务器时，会表现出一些细小的错误。其他的应用程序因重度使用或进行不恰当的操作给 Zookeeper 群组带来压力，所以最好让它们使用自己的 Zookeeper 群组。


## kafka-topics.sh 

```bash
$ $ bin/kafka-topics.sh 
Create, delete, describe, or change a topic.
Option                                   Description                            
------                                   -----------                            
--alter                                  修改主题的分区数和（或）复制作业的配置.         
--config <String: name=value>            A topic configuration override for the 
                                           topic being created or altered.The   
                                           following is a list of valid         
                                           configurations:                      
                                                cleanup.policy                        
                                                compression.type                      
                                                delete.retention.ms                   
                                                file.delete.delay.ms                  
                                                flush.messages                        
                                                flush.ms                              
                                                follower.replication.throttled.       
                                           replicas                             
                                                index.interval.bytes                  
                                                leader.replication.throttled.replicas 
                                                max.message.bytes                     
                                                message.format.version                
                                                message.timestamp.difference.max.ms   
                                                message.timestamp.type                
                                                min.cleanable.dirty.ratio             
                                                min.compaction.lag.ms                 
                                                min.insync.replicas                   
                                                preallocate                           
                                                retention.bytes                       
                                                retention.ms                          
                                                segment.bytes                         
                                                segment.index.bytes                   
                                                segment.jitter.ms                     
                                                segment.ms                            
                                                unclean.leader.election.enable        
                                         See the Kafka documentation for full   
                                           details on the topic configs.        
--create                                 创建一个新 topic.                    
--delete                                 删除一个 topic                         
--delete-config <String: name>           A topic configuration override to be   
                                           removed for an existing topic (see   
                                           the list of configurations under the 
                                           --config option).                    
--describe                               列出给定主题的详细信息.     
--disable-rack-aware                     Disable rack aware replica assignment  
--force                                  抑制（Suppress）控制台提示               
--help                                   Print usage information.               
--if-exists                              if set when altering or deleting       
                                           topics, the action will only execute 
                                           if the topic exists                  
--if-not-exists                          if set when creating topics, the       
                                           action will only execute if the      
                                           topic does not already exist         
--list                                   List all available topics.             
--partitions <Integer: # of partitions>  The number of partitions for the topic 
                                           being created or altered (WARNING:   
                                           If partitions are increased for a    
                                           topic that has a key, the partition  
                                           logic or ordering of the messages    
                                           will be affected                     
--replica-assignment <String:            A list of manual partition-to-broker   
  broker_id_for_part1_replica1 :           assignments for the topic being      
  broker_id_for_part1_replica2 ,           created or altered.                  
  broker_id_for_part2_replica1 :                                                
  broker_id_for_part2_replica2 , ...>                                           
--replication-factor <Integer:           正在创建的主题中的每个分区的复制因子  .      
  replication factor>                     
--topic <String: topic>                  要创建、更改或描述的主题。除了 --create 选项之外，还可以接受正则表达式
--topics-with-overrides                  if set when describing topics, only    
                                           show topics that have overridden     
                                           configs                              
--unavailable-partitions                 if set when describing topics, only    
                                           show partitions whose leader is not  
                                           available                            
--under-replicated-partitions            if set when describing topics, only    
                                           show under replicated partitions     
--zookeeper <String: hosts>              REQUIRED: The connection string for    
                                           the zookeeper connection in the form 
                                           host:port. Multiple hosts can be     
                                           given to allow fail-over.  
```


**创建主题**

```bash
$ bin/kafka-topics.sh --create   \
  --zookeeper localhost:2181/chengchao   \
  --replication-factor 1   \
  --partitions 1   \
  --topic test2
Created topic "test2".

```

**列出主题**

```bash
$ bin/kafka-topics.sh --zookeeper localhost:2181/chengchao --list
```

<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
