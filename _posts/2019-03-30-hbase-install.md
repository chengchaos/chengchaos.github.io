---
title: HBase 安装
key: 20190330
tags: bigdata hbase
---

我的 HBase 的学习笔记。:lizard:

<!--more-->


## Hbase 的适用场景

- 瞬间写入量很多，
- 需要长久保存

HBase 不适用

- 有 join ，多级索引，表关系复杂的数据模型。


## CAP 定理

一致性/可用性/分区容错性

- 一致性：所有节点在同一时间具有相同的数据
- 可用性：保证每个请求不管成功或者失败都有些响应，但不保证获取的数据为正确的数据
- 分区容错性：系统中

## ACID 

数据库事务正确执行的基本要素

- 原子性
- 一致性
- 隔离性
- 持久性

## 伪分布式的 Hadoop

以 2.7.x 为例

### 下载

添加环境变量

```bash

$ vim ~/.bashrc

export HADOOP_HOME=/works/apps/hadoop
export HADOOP_PREFIX=$HADOOP_HOME
export HADOOP_MAPREAD_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_INSTALL=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR
export PATH=$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$PATH

$ source ~/.bashrc
```

修改 `hadoop-env.sh` 配置

```bash
$ cd $HADOOP_HOME
$ vim etc/hadoop/hadoop-env.sh
export HADOOP_NAMENODE_OPTS=" -Xms1G -Xmx1G -XX:+UseParallelGC"
export HADOOP_DATANODE_OPTS=" -Xms1G -Xmx1G"
export HADOOP_LOG_DIR=/works/datas/logs/hadoop
$ mkdir -p /works/datas/logs/hadoop
$ mkdir -p /works/datas/hdfs/{name,data}
```

配置 `hdfs-site.xml` 文件

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///works/datas/hdfs/name</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///works/datas/hdfs/data</value>
    </property>
</configuration>
```

配置 `core-site.xml` 文件

```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:///works/datas/hdfs/</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://mycentos7:8020</value>
        <commit>早期的端口是 9000，新版本的端口是 8020. Why ?</commit>
    </property>
<!--
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://0.0.0.0:9000</value>
    </property>
-->
</configuration>
```

```bash
### 格式化 namenode
$ hdfs namenode -foromat
### 启动单机模式
$ $HADOOP_PREFIX/sbin/start-dfs.sh
```

可以通过 web 浏览器访问主机的 50070 端口查看

http://192.168.1.4:50070/

http://192.168.56.123:9870/dfshealth.html#tab-overview

## Zookeeper

```bash
$ tar -zxvf apache-zookeeper-3.5.5.tar.gz -C /works/apps/
$ cd /works/apps/
$ ln -s apache-zookeeper-3.5.5 zookeeper
$ vim ~/.bashrc

export ZOOKEEPER_HOME=/works/apps/zookeeper


$ source ~/.bashrc
$ cd $ZOOKEEPER_HOME
$ cd conf/
$ cp zoo_sample.cfg zoo.cfg
$ vim zoo.cfg

dataDir=/works/datas/zk

$ mkdir -p /works/datas/zk
$ echo 1 > /works/datas/zk/myid

$ cd ..
$ vim bin/zkEnv.sh

ZOO_LOG_DIR=/works/datas/logs/zk

$ mkdir -p /works/datas/logs/zk
$ bin/zkServer.sh start

ZooKeeper JMX enabled by default
Using config: /works/apps/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

$ bin/zkCli.sh
```

## 伪分布式的 HBase

以 1.2.x 为例

首先编辑环境变量

```bash
export HBASE_HOME=/works/apps/hbase
export PATH=$HBASE_HOME/bin:$PATH

$ cd /works/Downloads
$ tar -zxvf hbase-2.2.3-bin.tar.gz -C /works/apps
$ cd /works/apps
$ ln -s hbase-2.2.3 hbase
```

配置 `hbase-env.sh` 文件

添加 JAVA_HOME 的配置

```bash
# export JAVA_HOME=/usr/java/jdk1.6.0/
export JAVA_HOME='/usr/java/default'

# 使用 HBase 自带的 zookeeper
# 默认为 true
# export HBASE_MANAGES_ZK=true

```

配置 `hbase-site.xml` 文件, 添加上 `hbase.rootdir` 和 `hbase.zookeeper.property.dataDir` 的配置.

```xml
<configuration>

    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/works/apps/zookeeper/</value>
    </property>

    <!-- 此处必须为true，不然hbase仍用自带的zk，-->
    <!-- 若启动了外部的zookeeper，会导致冲突，hbase启动不起来 --> 
    <property> 
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>


    <!-- value 为 /hbase 意思是使用本地磁盘 -->
    <!-- value 为 hdfs://hostname:8020/hbase 是用的 hdfs -->
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://c7h23:8020/hbase</value>
    </property>


    <!-- ZK位置（HBase使用外部ZK，hbase-env.sh中属性HBASE_MANAGES_ZK要设置为false）-->
    <!-- 必须ZK数量必须为奇数，多个可用逗号分隔 --> 
    <property> 
        <name>hbase.zookeeper.quorum</name> 
        <value>c7h23</value> 
    </property> 


    <!-- ZooKeeper 会话超时。Hbase 把这个值传递给 zk 集群，向它推荐一个会话的最大超时时间 --> 
    <property> 
        <name>zookeeper.session.timeout</name> 
        <value>120000</value> 
    </property> 

    <!-- 当 regionserver 遇到 ZooKeeper session expired ， regionserver 将选择 restart 而不是 abort --> 
    <property> 
        <name>hbase.regionserver.restart.on.zk.expire</name> 
        <value>true</value> 
    </property>


</configuration>
```

### Troubleshooting (故常排除)

```sh

hdfs-site.xml:
dfs.permissions.supergroup 

# groupadd supergroup
# groupmems -g supergroup -a hbase
```


复制 Hadoop 的 hdfs-site.xml 和 core-site.xml 到 HBase 的 conf 目录中


可以通过 web 浏览器访问主机的 16010 端口查看

http://192.168.1.4:16010/

是 60010 么？


## Java 对 HBase 操作

| Java 类 | 对应数据模型 |
| ---- | ---- |
| HBaseConfiguration | HBase 配置类 |
| HBaseAdmin | HBase 管理 Admin 类 |
| Table | HBase 表操作类 |
| Put | HBase 添加操作数据模型 |
| Get | HBase 单个查询操作数据模型 |
| Scan | HBase Scan 检索操作数据模型 |
| Result | HBase 查询的结果类型 |
| ResultScanner | HBase 检索结果模型 |


```xml

		<!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-client -->
		<dependency>
			<groupId>org.apache.hbase</groupId>
			<artifactId>hbase-client</artifactId>
			<version>1.2.0</version>
		</dependency>

```




<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
=