---
title: 安装开发环境的依赖组建
key: 2023-03-08
tags: linux docker mongodb rabbitmq postgres
---

> Suggest search： linux docker mongodb rabbitmq postgres

Spring gateway 配置路由主要有两种方式，一种是用 yaml 配置文件，一种是写代码里，这两种方式都是不支持动态配置的。

<!--more-->

Ubuntu Server 安装 Docker

```bash
sudo apt install docker.io
sudo ufw status
```

## 0x01 Docker 安装 MongoDB

```bash
docker run -d --name mymoneo \
  --restart always \
  --memory 512m \
  --net host \
  -v /works/docker/data/mongodb/data:/data/db \
  -e TZ='Asia/Shanghai' \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=1qaz@WSX \
  --privileged=true \
  mongo:latest
```

## 0x02 Docker 安装 RabbitMQ

```bash
docker run -d \
  --name myrabbitmq \
  --restart always \
  --memory 512m \
  -p 5672:5672 -p 15672:15672 \
  rabbitmq:3-management
```

- 地址： http://192.168.1.42:15672
- 账号： guest
- 口令： guest

## 0x03 Docker 安装 PostgreSQL

```bash
# 拉取镜像
docker pull postgres:15
# 创建本地卷
docker volume create pgdata
docker inspect pgdata
[
    {
        "CreatedAt": "2023-05-22T08:06:49Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/pgdata/_data",
        "Name": "pgdata",
        "Options": {},
        "Scope": "local"
    }
]
# 运行
docker run -d \
  --name mypgsql \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=ab123456 \
  -v pgdata:/var/lib/postgresql/data \
  --restart=always \
  postgres:15

docker exec -it mypgsql bash
```

## pg_demp

关于pg_dump:

- pg_dump:  将一个PostgreSQL数据库抽出到一个脚本文件或者其它归档文件中。
- pg_dump 是一个用于备份 PostgreSQL 数据库的实用工具，即使当前数据库正在使用，也能够生成一致性的备份，且不会阻塞其他用户访问数据库(包括读、写)。
- PostgreSQL 提供的一个工具 pg_dump,逻辑导出数据，生成 sql 文件或其他格式文件。
- pg_dump 是一个客户端工具，可以远程或本地导出逻辑数据，恢复数据至导出时间点。
- pg_dump 只能备份一个数据库
- pg_dump 一次只转储一个数据库，并不会转储有关角色或表空间的信息 (因为那些是群集范围而不是每个数据库)。。

关于 pg_dumpall：

- 如果要备份 Cluster 中数据库共有的全局对象，例如角色和表空间，需要使用 pg_dumpall。
- 备份文件以文本或存档文件格式输出。
- Script dumps是一个普通文本文件，包含将数据库重构到保存时的状态所需的 SQL 命令。
- 要从这样的脚本恢复，需要将其提供给 psql。脚本文件甚至可以用来在其他机器或者其他架构上重构数据库;进行一些必要的修改，甚至可以在其他数据库上使用。
- pg_dumpall 在给定的群集中备份每个数据库, 并保留群集范围内的数据, 如角色和表空间定义。

关于pg_restore：

- PostgreSQL提供的一个工具pg_restore用来导入数据

> 原文链接：https://blog.csdn.net/justlpf/article/details/91789787

使用 pg_dump 导出整个数据库

```bash
pg_dump -h localhost \
  -U username 数据库名（缺失时同用户名)>/data/dum.sql

## 数据库名和 > 之间不能有空格。

psql -U 用户名 数据库名（缺失时同用户名) < /data/dum.sql
pg_restore -h host -p port -U username -W -d db -v "/data/dum.bakcup"

xz dum.sql ## 压缩
xzcat /data/dum.sql.xz | psql -h host -U user dbname
```

## 0x04 参考


EOF


