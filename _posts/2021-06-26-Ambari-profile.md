---
title: Ambari 简介
key: 2021-06-26
tags: Ambari
---

Apache Ambari 是一种基于 Web 的工具，支持 Apache Hadoop 集群的创建、管理和监控。跟 Hadoop 等开源软件一样，也是 Apache Software Foundation 中的一个项目，并且是顶级项目。Ambari 已支持大多数 Hadoop 组件，包括HDFS、MapReduce、Hive、Pig、 Hbase、Zookeeper、Sqoop 和 Hcatalog 等；除此之外，Ambari 还支持 Spark、Storm 等计算框架及资源调度平台 YARN。简单来说，Ambari 就是为了让 Hadoop 以及相关的大数据软件更容易使用的一个工具。

<!--more-->

Apache Ambari 从集群节点和服务收集大量信息，并把它们表现为容易使用的，集中化的接口：Ambari Web.

Ambari Web 显示诸如服务特定的摘要、图表以及警报信息。可通过 Ambari Web 对 Hadoop 集群进行创建、管理、监视、添加主机、更新服务配置等；也可以利用 Ambari Web 执行集群管理任务，例如启用 Kerberos 安全以及执行 Stack 升级。任何用户都可以查看 Ambari Web 特性。拥有 administrator-level 角色的用户可以访问比 operator-level 或 view-only 的用户能访问的更多选项。例如，Ambari administrator 可以管理集群安全，一个 operator 用户可以监控集群，而 view-only 用户只能访问系统管理员授予他的必要的权限。

## 体系结构

Ambari 自身也是一个分布式架构的软件，主要由两部分组成：Ambari Server 和 Ambari Agent。简单来说，用户通过 Ambari Server 通知 Ambari Agent 安装对应的软件；Agent 会定时地发送各个机器每个软件模块的状态给 Ambari Server，最终这些状态信息会呈现在 Ambari 的 GUI，方便用户了解到集群的各种状态，并进行相应的维护。

Ambari Server 从整个集群上收集信息。每个主机上都有 Ambari Agent, Ambari Server 通过 Ambari Agent 控制每部主机。

## 安装

[http://doc.ailinux.net/docs/data/data-1ardi0l817b5q](http://doc.ailinux.net/docs/data/data-1ardi0l817b5q)

### 安装准备

- SSH 的无密码登录；
    Ambari 的 Server 会 SSH 到 Agent 的机器，拷贝并执行一些命令。因此我们需要配置 Ambari Server 到 Agent 的 SSH 无密码登录。在这个例子里，zwshen37 可以 SSH 无密码登录 zwshen38 和 zwshen39。
- 确保 Yum 可以正常工作；
    通过公共库（public repository），安装 Hadoop 这些软件，背后其实就是应用 Yum 在安装公共库里面的 rpm 包。所以这里需要您的机器都能访问 Internet。
- 确保 home 目录的写权限。
    Ambari 会创建一些 OS 用户。
- 确保机器的 Python 版本大于或等于 2.6.（Redhat6.6，默认就是 2.6 的）。

### 安装过程（CentOS）

首先需要获取 Ambari 的公共库文件（public repository）。登录到 Linux 主机并执行下面的命令（也可以自己手工下载）：

```sh
wget http://public-repo-1.hortonworks.com/ambari/centos6/2.x/updates/2.0.1/ambari.repo
```

将下载的 ambari.repo 文件拷贝到 Linux 的系统目录/etc/yum.repos.d/。拷贝完后，我们需要获取该公共库的所有的源文件列表。依次执行以下命令。

```bash
yum clean all
yum list|grep ambari
yum install ambari-server
```

待安装完成后，便需要对 Ambari Server 做一个简单的配置。执行下面的命令。

```sh
amari-server setup
```

在这个交互式的设置中，采用默认配置即可。Ambari 会使用 Postgres 数据库，默认会安装并使用 Oracle 的 JDK。默认设置了 Ambari GUI 的登录用户为 admin/admin。并且指定 Ambari Server 的运行用户为 root。

简单的 setup 配置完成后。就可以启动 Ambari 了。运行下面的命令。

```sh
ambari-server start
```

当成功启动 Ambari Server 之后，便可以从浏览器登录，默认的端口为 8080。登录密码为 admin/admin。

### 安装过程（SUSE）

```sh
cd  /etc/zypp/repos.d/  
wget http://public-repo-1.hortonworks.com/ambari/suse11/1.x/updates/1.4.4.23/ambari.repo

3、zypper install ambari-server  安装ambari-server

4、ambari-server setup  采用默认配置，如果之前安装过其他版本的ambari-server，rm -rf /var/lib/pgsql/data 删除旧版本的postgresql数据库或者cp/var/lib/pgsql/data/ /var/lib/pgsql/data.old备份，以免不兼容造成ambari-server无法正常启动

5、ambari-server start 启动，如果能够正常启动，使用ps -ef | grep ambari应能看到ambari-server的进程，如果ambari-server没起来，那么vi /var/log/ambari-server/ambari-server.log查看日志寻找出错点。

6、登陆localhost:8080进行配置。

在配置时，如果出现错误，cat /var/log/ambari-server/ambari-server.log 查看输出日志。

——如果是出现了WARN HeartBeatHandler:325 - Received registration request from host with non matching os type, hostname=linux-kc7s.site, serverOsType=sles11, agentOstype=opensuse11 错误的话，修改/etc/issue 以及/etc/SuSE-release中的版本信息，如/etc/issue中内容是：

Welcome to openSUSE 11.4 "Celadon" - Kernel \r (\l).

ambari会从中识别到版本信息为opensuse 11，但是ambari支持的胃sles11或者suse11，所以把/etc/issue中内容修改为：

Welcome to SUSE11.4 "Celadon" - Kernel \r (\l).

把/etc/SuSE-release中的openSUSE 11.4 (x86_64)修改为SUSE11.4 (x86_64)

 ，然后重新启动ambari-server进行配置。


```

### 部署一个 Hadoop 2.x 集群



## 操作指南

[https://docs.hortonworks.com/HDPDocuments/Ambari-2.6.1.5/bk_ambari-operations/content/ch_Overview_hdp-ambari-user-guide.html](https://docs.hortonworks.com/HDPDocuments/Ambari-2.6.1.5/bk_ambari-operations/content/ch_Overview_hdp-ambari-user-guide.html)

## HDP版本兼容性

```sh
CentOS 7
Python 2.7.x
JDK1.8+
Mariadb 5.5
Ambari2.6.1.5 
HDP 2.6.4
```

EOF

---

Power by TeXt.
