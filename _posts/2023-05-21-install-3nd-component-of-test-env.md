---
title: 安装开发环境的依赖组建
key: 2023-03-08
tags: linux docker mongodb rabbitmq
---

> Suggest search： linux docker mongodb rabbitmq

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


## 0x04 参考


EOF


