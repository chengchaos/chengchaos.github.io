---
title: Harbor 安装
key: 2021-08-22
tags: harbor
---

以 Docker 为代表的容器技术的出现，改变了传统的交付方式。通过把业务及其依赖的环境打包进 Docker 镜像，解决了开发环境和生产环境的差异问题，提升了业务交付的效率。如何高效地管理和分发 Docker 镜像？是众多企业需要考虑的问题。

Harbor 是一个用于存储和分发 Docker 镜像的企级 Registry 服务器，可以用来构建企业内部的 Docker 镜像仓库。

它在 Docker 的开源项目 Distribution 的基础上，添加了一些企业需要的功能特性，如镜像同步复制、漏洞扫描和权限管理等。

<!--more-->

详细参考 [https://cloud.tencent.com/developer/article/1404719](https://cloud.tencent.com/developer/article/1404719)

## Harbor 是什么

Harbor 是 VMware 公司开源的企业级DockerRegistry 项目，其目标是帮助用户迅速搭建一个企业级的 Dockerregistry 服务。

它以 Docker 公司开源的 registry 为基础，提供了管理 UI，基于角色的访问控制(Role Based Access Control)，AD/LDAP集成、以及审计日志(Auditlogging) 等企业用户需求的功能，同时还原生支持中文。

## Harbor 的安装

Harbor 被部署为多个 Docker 容器，因此可以部署在任何支持 Docker 的 Linux 服务器上，且需要 Docker 和 Docker Compose 才能安装。

所需软件：

docker docker-compose openssl

所需端口：

443 https
4443 https
80 http

### 安装步骤

- 1， 下载安装程序
- 2， 配置 harbor.yml 文件
- 3， 使用适当的选娘运行 install.sh 脚本安装和启动 Harbor。

### 1，下载安装程序

安装 cocker-compose

[https://chengchaos.github.io/2021/08/10/docker-compose-install.html](https://chengchaos.github.io/2021/08/10/docker-compose-install.html)


去 github 上下载 tgz 的压缩包， 地址是：

[https://github.com/goharbor/harbor/tags](https://github.com/goharbor/harbor/tags)

创建一个安装目录，例如：

```bash
sudo -i
mkdir -p /opt/harbor
cd /opt/harbor
wget -c https://github.com/goharbor/harbor/releases/download/v2.3.1/harbor-offline-installer-v2.3.1.tgz

```


## Docker镜像远程同步复制功能

如下图所示，Harbor 提供了基于策略的镜像同步复制功能。以项目为单位，通过配置复制同步策略，可以实现在多个 Harbor 实例间进行镜像同步复制。

Harbor 的镜像同步功能支持错误或失败重传，支持镜像增量同步复制。

![图](https://ask.qcloudimg.com/http-save/yehe-1376019/eh79ecej9x.png?imageView2/2/w/1620)

（图片来源于 —— <<采用Harbor开源企业级Registry实现高效安全的镜像运维>> 张海宁）

该功能目前只支持镜像的同步复制，其它信息，如用户、复制规则等，还不支持同步复制。

## 高可用方案

Harbor 官方提供的安装包，默认是单机部署的，各组件都只有一个容器，存在单点故障的风险。在要求不高的场合下，可以使用；但在生产环境中，一般是不能直接使用的。

Harbor 有很多种高可用负载均衡方案，结合公司目前的情况，使用基于镜像同步复制的高可用方案，方案框架图如下。

图中的 Harbor 包含了除 DB（Mysql镜像）外的 Harbor 其它组件。Keepalived 和 HAprox y用来实现流量入口的高可用和负载均衡。

两个 Harbor 实例（除DB外）组件的高可用，两个实例之间需要配置镜像同步复制规则，以实现镜像的备份。两个实例需要共享DB，但同一时刻内只会有一个实例会访问DB。DB由DBA提供高可用方案。

使用此方案有个明显的问题。两个 Harbor 实例都需要配置到目标实例的镜像同步复制规则，但由于两个实例属于共享一个数据库，所以就会出现把镜像同步给自己的问题，从而导致失败，且一直会重试。一个解决方法是在镜像同步复制的代码中判断一下目标实例是否为自己，如果是，则直接返回成功即可。

## 镜像自动化删除

默认情况下，Harbor  将镜像存储在本地磁盘，随着镜像越来越多，可能会导致磁盘空间不够。为了提供磁盘的利用率，需要把不用或者多余的镜像删除。Harbor 提供了删除镜像的功能，但需要手动去处理。

可以采用 cron job 定期去运行镜像删除脚本来实现镜像自动化删除。脚本通过调用 Harbor 的 RESTful API，来获取要删除镜像的名称和 tag。删除镜像的过程如下：

1) 获取所有 project 并解析 project_id 字段，得到 project 的个数；

2) 获取每个 project 的 repo 名称；

3) 根据 repo 名称获取每个 repo 下的 tag 并统计个数；

4) 若 tag 数大于规定的个数，进行排序，删除最早的 tag（并不会删除镜像）；

5) 删除镜像，命令如下：

```bash
docker-compose stop；

docker run -it --name gc --rm --volumes-from registry vmware/registry:2.6.1-photon garbage-collect /etc/registry/config.yml；

docker-compose start；
```

EOF
