---
title: ubuntu 中安装 Docker
key: 2019-08-21
tags: docker ubuntu
---





As The Titile



<!--more-->

## 操作系统

 安装 Docker Engine - Community， 需要 64-bit 版本的 Ubuntu：

- Disco 19.04
- Cosmic 18.10
- Bionic 18.04（LTS）
- Xenial 16.04（LTS）



## 删除旧版本

如果已经安装过了就先删除他们：



```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```



**支持的存储设备**



Ubunut 上的 Docker Engine - Community  支持 `overlay2`, `aufs` , `btrfs`  存储设备



## 安装



有三种方式：

- 设置 Docker 的 repositories
- 使用 DEB 包
- 脚本



**使用 repository 安装**



1：更新 apt 包索引：

```bash
$ sudo apt-get update
```



3: 安装包使 apt 可以使用 https 

```bash
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```



3: 添加 docker 的官方 GPG 密钥：



```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
$ sudo apt-key fingerprint 0EBFCD88
    
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```



 4: 设置仓库



```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```



## 安装



```bash
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```



**指定容器目录**



编辑 /lib/systemd/system/docker.service 文件，怎加 `--graph /works/docker ` 

```bash
ExecStart=/usr/bin/dockerd --graph /works/docker -H fd:// --containerd=/run/containerd/containerd.sock
```

**增加用户到docker 组**



```bash
$ sudo usermod -a -G docker chengchao
```









参考连接：



- [https://docs.docker.com/install/](https://docs.docker.com/install/)

<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
