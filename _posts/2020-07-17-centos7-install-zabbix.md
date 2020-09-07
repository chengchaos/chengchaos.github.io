---
title: CentOS 7 安装 Zabbix
key: 2020-07-17
tags: centos7 zabbix
---



参考： 
- https://www.cnblogs.com/it-peng/p/11393837.html

## 介绍

略

<!--more-->

## 安装 Docker 


略

## 安装 Zabbix

zabbix使用docker容器安装的官网地址（版本4.2）

https://www.zabbix.com/documentation/4.2/manual/installation/containers


### 第一步：启动数据库

```bash
docker run --name mysql-server -t \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="zabbix_pwd" \
      -e MYSQL_ROOT_PASSWORD="root_pwd" \
      -d mysql:5.7 \
      --character-set-server=utf8 --collation-server=utf8_bin
```

### 第二步：启动 Zabbix server 实例

并将其关联到已创建的 MySQL server 实例。

```bash
$ docker run --name zabbix-server-mysql -t \
      -e DB_SERVER_HOST="mysql-server" \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="zabbix_pwd" \
      -e MYSQL_ROOT_PASSWORD="root_pwd" \
      --link mysql-server:mysql \
      -p 10051:10051 \
      -d zabbix/zabbix-server-mysql:latest	  
```


### 第三步：启动 Zabbix Web 界面

并将其关联到已创建的 MySQL server 和 Zabbix server 实例。

```bash
docker run --name zabbix-web-nginx-mysql -t \
      -e DB_SERVER_HOST="mysql-server" \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="zabbix_pwd" \
      -e MYSQL_ROOT_PASSWORD="root_pwd" \
      --link mysql-server:mysql \
      --link zabbix-server-mysql:zabbix-server \
      -p 9111:8080 \
      -d zabbix/zabbix-web-nginx-mysql:latest
```


默认的用户名： Admin ； 密码 zabbix ;


### 部署 Zabbix-agent 端
 
```bash
$ docker run --name zabbix-agent \
  -p 10050:10050 \
  -e ZBX_HOSTNAME="kw-kafka" \
  -e ZBX_SERVER_HOST="192.168.1.133" \
  -e ZBX_SERVER_PORT=10051 \
  -d zabbix/zabbix-agent

```

后来我发现用 Docker 都跑不起来，Agent 和 Server 都没法互相通信，然后我就放弃了，下一步看看 Prometheus 。


---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





