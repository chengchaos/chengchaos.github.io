---
title: Nacos 安装
key: 2021-09-01
tags: nacos
---

[什么是 Nacos](https://nacos.io/zh-cn/docs/what-is-nacos.html)

<!--more-->

## 选择版本

您可以在 Nacos 的 [release notes](https://github.com/alibaba/nacos/releases) 及 [博客](https://nacos.io/zh-cn/blog/index.html) 中找到每个版本支持的功能的介绍，当前推荐的稳定版本为2.0.3。

## 准备环境

Nacos 依赖 Java 环境来运行。如果您是从代码开始构建并运行 Nacos，还需要为此配置 Maven环境，请确保是在以下版本环境中安装使用:

- 64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
- 64 bit JDK 1.8+；
- Maven 3.2.x+；

## 单击模式部署

### 下载源码或者安装包

你可以通过源码和发行包两种方式来获取 Nacos。

#### 从 Github 上下载源码方式

```bash
git clone https://github.com/alibaba/nacos.git
cd nacos/
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U  
ls -al distribution/target/

// change the $version to your actual path
cd distribution/target/nacos-server-$version/nacos/bin
```

#### 下载编译后压缩包方式

您可以从 [最新稳定版本](https://github.com/alibaba/nacos/releases) 下载 nacos-server-$version.zip 包。

```bash
unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz
cd nacos/bin
```

### 启动服务器

#### Linux/Unix/Mac

启动命令(standalone代表着单机模式运行，非集群模式):

```bash
sh startup.sh -m standalone
```

如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行：

```bash
bash startup.sh -m standalone
```

#### Windows

启动命令(standalone代表着单机模式运行，非集群模式):

```bat
startup.cmd -m standalone
```

### 服务注册&发现和配置管理

#### 服务注册

```bash
curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'
```

#### 服务发现

```bash
curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=nacos.naming.serviceName'
```

#### 发布配置

```bash
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=HelloWorld"
```

#### 获取配置

```bash
curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"
```

### 关闭服务器

#### 1 Linux/Unix/Mac

```bash
sh shutdown.sh
```

#### 2 Windows

```bat
shutdown.cmd
```

或者双击shutdown.cmd运行文件。

## 集群模式

### 集群配置文件

在nacos的解压目录nacos/的conf目录下，有配置文件cluster.conf，请每行配置成ip:port。（请配置3个或3个以上节点）

```bash
# ip:port
200.8.9.16:8848
200.8.9.17:8848
200.8.9.18:8848
```

### 确定数据源

#### 使用内置数据源

无需进行任何配置

#### 使用外置数据源

生产使用建议至少主备模式，或者采用高可用数据库。

初始化 MySQL 数据库

[sql语句源文件](https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql)

application.properties 配置

[application.properties 配置文件](https://github.com/alibaba/nacos/blob/master/distribution/conf/application.properties)

### 启动服务器2

#### for Linux/Unix/Mac

##### Stand-alone mode

```bash
sh startup.sh -m standalone
```

##### 集群模式2

使用内置数据源

```bash
sh startup.sh -p embedded
```

使用外置数据源

```bash
sh startup.sh
```

### 服务注册&发现和配置管理 1

服务注册

```bash
curl -X PUT 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'
```

服务发现

```bash
curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=nacos.naming.serviceName'
```

发布配置

```bash
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=helloWorld"
```

获取配置

```bash
curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"
```

### 关闭

```bash
sh shutdown.sh
```

EOF
