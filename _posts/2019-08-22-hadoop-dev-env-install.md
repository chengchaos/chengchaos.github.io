---
title: 安装开发环境的 Hadoop 
key: 2019-08-22
tags: hadoop spark
---

As the title says.

<!--more-->

## 下载

**地址：**

http://archive.cloudera.com/cdh5/cdh/5/

**准备工作：**

- JDK
- hostname
- ip hostname 映射
- ssh 免密码登录
- maven 添加镜像：

### maven 添加镜像

```xml

    <mirrors>
        <mirror>
            <id>alimaven</id>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>

```

### 安装开发环境的 Hadoop

1) 解压缩，修改配置文件：

1.1) hadoop-env.sh

```bash
export JAVA_HOME=/usr/local/jvm/jdk
```

1.2) core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://t420i:8020</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/works/datas/hadoop/tmp</value>
    </property>
</configuration>

```

1.3) hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>

```

2.) 执行命令命令行执行格式化 HDFS

```bash
$ $HADOOP_HOME/bin/hdfs namenode -format
19/08/16 20:29:02 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   user = chengchao
STARTUP_MSG:   host = t420i/192.168.88.240
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.6.0-cdh5.16.2
STARTUP_MSG:   classpath = /works/local/hadoop-2.6.0-cdh5.16.2/etc/hadoop:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/paranamer-2.3.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/apacheds-i18n-2.0.0-M15.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jersey-json-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/api-util-1.0.0-M20.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jasper-runtime-5.5.23.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/curator-client-2.7.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/guava-11.0.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/stax-api-1.0-2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/mockito-all-1.8.5.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jets3t-0.9.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-digester-1.8.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/httpclient-4.2.5.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/gson-2.2.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jersey-core-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/slf4j-api-1.7.5.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jetty-util-6.1.26.cloudera.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jsch-0.1.42.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/activation-1.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/api-asn1-api-1.0.0-M20.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/hadoop-annotations-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jsp-api-2.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/logredactor-1.0.3.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-httpclient-3.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/curator-recipes-2.7.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jackson-jaxrs-1.8.10.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-el-1.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/netty-3.10.5.Final.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jsr305-3.0.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/xz-1.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-compress-1.4.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/httpcore-4.2.5.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-lang-2.6.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-math3-3.1.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jaxb-impl-2.2.3-1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jackson-xc-1.8.10.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/xmlenc-0.52.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/avro-1.7.6-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-collections-3.2.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jaxb-api-2.2.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-configuration-1.6.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/snappy-java-1.0.4.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jasper-compiler-5.5.23.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-beanutils-core-1.8.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/hamcrest-core-1.3.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/asm-3.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/servlet-api-2.5.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jersey-server-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-cli-1.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-beanutils-1.9.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jetty-6.1.26.cloudera.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/jettison-1.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/protobuf-java-2.5.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/htrace-core4-4.0.1-incubating.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/hadoop-auth-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-net-3.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/curator-framework-2.7.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/log4j-1.2.17.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-codec-1.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/zookeeper-3.4.5-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-io-2.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/junit-4.11.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/java-xmlbuilder-0.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/lib/commons-logging-1.1.3.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/hadoop-nfs-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/hadoop-common-2.6.0-cdh5.16.2-tests.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/common/hadoop-common-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/jackson-mapper-asl-1.8.10-cloudera.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/jasper-runtime-5.5.23.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/guava-11.0.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/jackson-core-asl-1.8.10.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/jersey-core-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/leveldbjni-all-1.8.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/jetty-util-6.1.26.cloudera.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/jsp-api-2.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/commons-el-1.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/netty-3.10.5.Final.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/jsr305-3.0.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/commons-lang-2.6.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/xercesImpl-2.9.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/xmlenc-0.52.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/commons-daemon-1.0.13.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/asm-3.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/servlet-api-2.5.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/jersey-server-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/commons-cli-1.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/jetty-6.1.26.cloudera.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/xml-apis-1.3.04.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/protobuf-java-2.5.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/htrace-core4-4.0.1-incubating.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/log4j-1.2.17.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/commons-codec-1.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/commons-io-2.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/lib/commons-logging-1.1.3.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/hadoop-hdfs-nfs-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/hadoop-hdfs-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/hdfs/hadoop-hdfs-2.6.0-cdh5.16.2-tests.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/javax.inject-1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jackson-mapper-asl-1.8.10-cloudera.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jersey-json-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/guice-3.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jline-2.11.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/guava-11.0.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/stax-api-1.0-2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jackson-core-asl-1.8.10.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jersey-core-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/leveldbjni-all-1.8.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jetty-util-6.1.26.cloudera.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/activation-1.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jersey-guice-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jackson-jaxrs-1.8.10.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jsr305-3.0.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/xz-1.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/commons-compress-1.4.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/commons-lang-2.6.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jaxb-impl-2.2.3-1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jackson-xc-1.8.10.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/commons-collections-3.2.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jaxb-api-2.2.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/aopalliance-1.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/asm-3.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jersey-client-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/servlet-api-2.5.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jersey-server-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/commons-cli-1.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jetty-6.1.26.cloudera.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/jettison-1.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/protobuf-java-2.5.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/log4j-1.2.17.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/commons-codec-1.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/zookeeper-3.4.5-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/commons-io-2.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/guice-servlet-3.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/lib/commons-logging-1.1.3.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-server-web-proxy-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-client-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-server-tests-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-applications-distributedshell-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-common-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-applications-unmanaged-am-launcher-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-server-applicationhistoryservice-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-api-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-server-resourcemanager-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-server-nodemanager-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-registry-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/yarn/hadoop-yarn-server-common-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/javax.inject-1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/jackson-mapper-asl-1.8.10-cloudera.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/paranamer-2.3.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/guice-3.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/jackson-core-asl-1.8.10.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/jersey-core-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/leveldbjni-all-1.8.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/hadoop-annotations-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/jersey-guice-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/netty-3.10.5.Final.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/xz-1.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/commons-compress-1.4.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/avro-1.7.6-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/aopalliance-1.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/snappy-java-1.0.4.1.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/hamcrest-core-1.3.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/asm-3.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/jersey-server-1.9.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/protobuf-java-2.5.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/log4j-1.2.17.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/commons-io-2.4.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/guice-servlet-3.0.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/lib/junit-4.11.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.16.2-tests.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-client-common-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-plugins-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-client-nativetask-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-client-shuffle-2.6.0-cdh5.16.2.jar:/works/local/hadoop-2.6.0-cdh5.16.2/share/hadoop/mapreduce/hadoop-mapreduce-client-app-2.6.0-cdh5.16.2.jar:/works/apps/hadoop/contrib/capacity-scheduler/*.jar
STARTUP_MSG:   build = http://github.com/cloudera/hadoop -r 4f94d60caa4cbb9af0709a2fd96dc3861af9cf20; compiled by 'jenkins' on 2019-06-03T10:42Z
STARTUP_MSG:   java = 1.8.0_191
************************************************************/
19/08/16 20:29:02 INFO namenode.NameNode: registered UNIX signal handlers for [TERM, HUP, INT]
19/08/16 20:29:02 INFO namenode.NameNode: createNameNode [-format]
19/08/16 20:29:02 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Formatting using clusterid: CID-d78a2299-b270-444c-bb5f-c30166562475
19/08/16 20:29:03 INFO namenode.FSEditLog: Edit logging is async:true
19/08/16 20:29:03 INFO namenode.FSNamesystem: No KeyProvider found.
19/08/16 20:29:03 INFO namenode.FSNamesystem: fsLock is fair: true
19/08/16 20:29:03 INFO blockmanagement.DatanodeManager: dfs.block.invalidate.limit=1000
19/08/16 20:29:03 INFO blockmanagement.DatanodeManager: dfs.namenode.datanode.registration.ip-hostname-check=true
19/08/16 20:29:03 INFO blockmanagement.BlockManager: dfs.namenode.startup.delay.block.deletion.sec is set to 000:00:00:00.000
19/08/16 20:29:03 INFO blockmanagement.BlockManager: The block deletion will start around 2019 八月 16 20:29:03
19/08/16 20:29:03 INFO util.GSet: Computing capacity for map BlocksMap
19/08/16 20:29:03 INFO util.GSet: VM type       = 64-bit
19/08/16 20:29:03 INFO util.GSet: 2.0% max memory 889 MB = 17.8 MB
19/08/16 20:29:03 INFO util.GSet: capacity      = 2^21 = 2097152 entries
19/08/16 20:29:03 INFO blockmanagement.BlockManager: dfs.block.access.token.enable=false
19/08/16 20:29:03 WARN conf.Configuration: No unit for dfs.heartbeat.interval(3) assuming SECONDS
19/08/16 20:29:03 INFO blockmanagement.BlockManager: defaultReplication         = 1
19/08/16 20:29:03 INFO blockmanagement.BlockManager: maxReplication             = 512
19/08/16 20:29:03 INFO blockmanagement.BlockManager: minReplication             = 1
19/08/16 20:29:03 INFO blockmanagement.BlockManager: maxReplicationStreams      = 2
19/08/16 20:29:03 INFO blockmanagement.BlockManager: replicationRecheckInterval = 3000
19/08/16 20:29:03 INFO blockmanagement.BlockManager: encryptDataTransfer        = false
19/08/16 20:29:03 INFO blockmanagement.BlockManager: maxNumBlocksToLog          = 1000
19/08/16 20:29:03 INFO namenode.FSNamesystem: fsOwner             = chengchao (auth:SIMPLE)
19/08/16 20:29:03 INFO namenode.FSNamesystem: supergroup          = supergroup
19/08/16 20:29:03 INFO namenode.FSNamesystem: isPermissionEnabled = true
19/08/16 20:29:03 INFO namenode.FSNamesystem: HA Enabled: false
19/08/16 20:29:03 INFO namenode.FSNamesystem: Append Enabled: true
19/08/16 20:29:03 INFO util.GSet: Computing capacity for map INodeMap
19/08/16 20:29:03 INFO util.GSet: VM type       = 64-bit
19/08/16 20:29:03 INFO util.GSet: 1.0% max memory 889 MB = 8.9 MB
19/08/16 20:29:03 INFO util.GSet: capacity      = 2^20 = 1048576 entries
19/08/16 20:29:03 INFO namenode.FSDirectory: POSIX ACL inheritance enabled? false
19/08/16 20:29:03 INFO namenode.NameNode: Caching file names occuring more than 10 times
19/08/16 20:29:03 INFO snapshot.SnapshotManager: Loaded config captureOpenFiles: false, skipCaptureAccessTimeOnlyChange: false, snapshotDiffAllowSnapRootDescendant: true
19/08/16 20:29:03 INFO util.GSet: Computing capacity for map cachedBlocks
19/08/16 20:29:03 INFO util.GSet: VM type       = 64-bit
19/08/16 20:29:03 INFO util.GSet: 0.25% max memory 889 MB = 2.2 MB
19/08/16 20:29:03 INFO util.GSet: capacity      = 2^18 = 262144 entries
19/08/16 20:29:03 INFO namenode.FSNamesystem: dfs.namenode.safemode.threshold-pct = 0.9990000128746033
19/08/16 20:29:03 INFO namenode.FSNamesystem: dfs.namenode.safemode.min.datanodes = 0
19/08/16 20:29:03 INFO namenode.FSNamesystem: dfs.namenode.safemode.extension     = 30000
19/08/16 20:29:03 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.window.num.buckets = 10
19/08/16 20:29:03 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.num.users = 10
19/08/16 20:29:03 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.windows.minutes = 1,5,25
19/08/16 20:29:03 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
19/08/16 20:29:03 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
19/08/16 20:29:03 INFO util.GSet: Computing capacity for map NameNodeRetryCache
19/08/16 20:29:03 INFO util.GSet: VM type       = 64-bit
19/08/16 20:29:03 INFO util.GSet: 0.029999999329447746% max memory 889 MB = 273.1 KB
19/08/16 20:29:03 INFO util.GSet: capacity      = 2^15 = 32768 entries
19/08/16 20:29:03 INFO namenode.FSNamesystem: ACLs enabled? false
19/08/16 20:29:03 INFO namenode.FSNamesystem: XAttrs enabled? true
19/08/16 20:29:03 INFO namenode.FSNamesystem: Maximum size of an xattr: 16384
19/08/16 20:29:03 INFO namenode.FSImage: Allocated new BlockPoolId: BP-1439425910-192.168.88.240-1565958543756
19/08/16 20:29:03 INFO common.Storage: Storage directory /works/datas/hadoop/tmp/dfs/name has been successfully formatted.
19/08/16 20:29:03 INFO namenode.FSImageFormatProtobuf: Saving image file /works/datas/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
19/08/16 20:29:04 INFO namenode.FSImageFormatProtobuf: Image file /works/datas/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 326 bytes saved in 0 seconds .
19/08/16 20:29:04 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
19/08/16 20:29:04 INFO util.ExitUtil: Exiting with status 0
19/08/16 20:29:04 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at t420i/192.168.88.240
************************************************************/

```

3.) 启动 HDFS

```bash
$HADOOP_HOME/sbin/start-dfs.sh
19/08/16 20:36:30 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [t420i]
t420i: starting namenode, logging to /works/local/hadoop-2.6.0-cdh5.16.2/logs/hadoop-chengchao-namenode-t420i.out
t420i: starting datanode, logging to /works/local/hadoop-2.6.0-cdh5.16.2/logs/hadoop-chengchao-datanode-t420i.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /works/local/hadoop-2.6.0-cdh5.16.2/logs/hadoop-chengchao-secondarynamenode-t420i.out
19/08/16 20:36:48 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
chengchao@chaosubuntu:/works/apps/hadoop/etc/hadoop$ jps
1589 Jps
918 NameNode
1467 SecondaryNameNode
1199 DataNode

```

### YARN

2.) 编辑 site 文件

2.1) **mapred-site.xml**

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>

```

2.2) **yarn-site.xml**

```xml
<configuration>

<!-- Site specific YARN configuration properties -->

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>

```

2.3) 启动 YARN

```bash
$HADOOP_HOME/sbin/start-yarn.sh
```

### Hadoop 的 Web 界面

hdfs：http://t420i:50070/

yarn：http://t420i:8088/cluster

## Spark 编译

1.) **下载 src，解压**

前置要求：

1.1）：maven 3.5； jdk1.8

2.2）：export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=512m"

**mvn 编译命令：**

```bash

$ mvn -Pyarn -Phadoop-2.4 -Dhadoop.version=2.4.0 -DskipTests clean package

$ ./dev/make-distribution.sh \
--name 2.6.0-cdh5.16.2 --tgz \
-Pyarn -Phadoop-2.6 \
-Phive -Phive-thriftserver \
-Pscala-2.11 \
-Dhadoop.version=2.6.0


```

### 环境变量

```bash

export JAVA_HOME='/usr/local/jvm/jdk'
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export M2_HOME='/usr/local/jvm/maven'
export SCALA_HOME='/usr/local/jvm/scala'

# setup hadoop env
export HADOOP_HOME='/works/apps/hadoop'
export HADOOP_PREFIX=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME
export HADOOP_INSTALL=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME

```

## Spark 部署

### (local 模式)

啥都不用改, 解压到指定位置. 即可.

> 把 spark 加到系统环境变量中.

```bash
$ spark-shell --master local[2]

21/09/26 15:50:59 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://c7h23:4040
Spark context available as 'sc' (master = local[2], app id = local-1632642665156).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.0.3
      /_/

Using Scala version 2.12.10 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_301)
Type in expressions to have them evaluated.
Type :help for more information.

```

### (standalone 模式)

Spark standalone 模式的架构和 Hadoop HDFS/YARN 类似, 即 1 个 Master 带多个 Worker.

首先调整配置文件

```bash
cd conf/
cp spark-env.sh.template spark-env.sh


# Options for the daemons used in the standalone deploy mode
# - SPARK_MASTER_HOST, to bind the master to a different IP address or hostname
# - SPARK_MASTER_PORT / SPARK_MASTER_WEBUI_PORT, to use non-default ports for the master
# - SPARK_MASTER_OPTS, to set config properties only for the master (e.g. "-Dx=y")
# - SPARK_WORKER_CORES, to set the number of cores to use on this machine
# - SPARK_WORKER_MEMORY, to set how much total memory workers have to give executors (e.g. 1000m, 2g)
# - SPARK_WORKER_PORT / SPARK_WORKER_WEBUI_PORT, to use non-default ports for the worker
# - SPARK_WORKER_DIR, to set the working directory of worker processes
# - SPARK_WORKER_OPTS, to set config properties only for the worker (e.g. "-Dx=y")
# - SPARK_DAEMON_MEMORY, to allocate to the master, worker and history server themselves (default: 1g).
# - SPARK_HISTORY_OPTS, to set config properties only for the history server (e.g. "-Dx=y")
# - SPARK_SHUFFLE_OPTS, to set config properties only for the external shuffle service (e.g. "-Dx=y")
# - SPARK_DAEMON_JAVA_OPTS, to set config properties for all daemons (e.g. "-Dx=y")
# - SPARK_DAEMON_CLASSPATH, to set the classpath for all daemons
# - SPARK_PUBLIC_DNS, to set the public dns name of the master or workers
SPARK_MASTER_HOST=192.168.56.123
SPARK_WORKER_CORES=2
SPARK_WORKER_MEMORY=2g
SPARK_WORKER_INSTANCES=1

sbin/start-all.sh
bin/spark-shell --master spark://192.168.56.123:7077
```

一个 WordCount 的示例:

```scala
val file = spark.sparkContext.textFile("file:///home/chengchao/data/wc.txt")
val wordCounts = file.flatMap(line => line.split(",")).
  map(word => (word, 1)).
  reduceByKey(_ + _)
wordCounts.collect


```

### 提交作业

```bash
bin/spark-submit \
  --class <main-class> \
  --master <master-url> \
  --deploy-mode <deploy-mode> \
  --conf <key=value> \
  ... # other options
  <application-jar> \
  [application-arguments]

 spark-submit --class cn.chengchaos.spark.sql.bba.BbaDemo1App1 \
   --master yarn \
   my-spark-demo-jar-with-dependencies.jar
 spark-submit --class cn.chengchaos.spark.sql.bba.BbaDemo1App1 \
   --master local[2] \
   my-spark-demo-jar-with-dependencies.jar
```

EOF
