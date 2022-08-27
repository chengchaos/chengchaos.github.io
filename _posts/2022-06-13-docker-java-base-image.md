---
title: 自定义一个 Java 环境的 Docker 镜像 （Alpine + OpenJDK）
key: 2022-06-13
tags: docker,images,java,openjdk,alpine
---

先做一个运行 Java 的 Docker 基础镜像。

<!--more-->

## 先手动测试



```sh
docker rm alpine-java
docker run -itd --name alpine-java alpine
docker exec -it alpine-java  sh

/ # 
/ # cat /etc/*rele*
3.16.0
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.16.0
PRETTY_NAME="Alpine Linux v3.16"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"

/ # sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
/ # apk update
/ # apk add openjdk8
/ # java -version
openjdk version "1.8.0_322"
OpenJDK Runtime Environment (IcedTea 3.22.0) (Alpine 8.322.06-r0)
OpenJDK 64-Bit Server VM (build 25.322-b06, mixed mode)
/ # apk add curl
(1/3) Installing nghttp2-libs (1.47.0-r0)
(2/3) Installing libcurl (7.83.1-r1)
(3/3) Installing curl (7.83.1-r1)
Executing busybox-1.35.0-r13.trigger
OK: 127 MiB in 63 packages
/ # curl --version
curl 7.83.1 (x86_64-alpine-linux-musl) libcurl/7.83.1 OpenSSL/1.1.1o zlib/1.2.12 brotli/1.0.9 nghttp2/1.47.0
Release-Date: 2022-05-11
Protocols: dict file ftp ftps gopher gophers http https imap imaps mqtt pop3 pop3s rtsp smb smbs smtp smtps telnet tftp 
Features: alt-svc AsynchDNS brotli HSTS HTTP2 HTTPS-proxy IPv6 Largefile libz NTLM NTLM_WB SSL TLS-SRP UnixSockets
/ # apk add vim
/ # apk add netcat-openbsd
/ # apk add tzdata
/ # echo Asia/Shanghai > /etc/timezone 
/ # rm -rf /tmp/* /var/cache/apk/*
/ # exit

docker commit alpine-java chengchao/alpine-java
docker push chengchao/alpine-java
```



## 再创建 Dockerfile



```dockerfile
FROM alpine
MAINTAINER Cheng,Chao Chao.Cheng@partner.bmw-brilliance.cn
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
    apk update && \
    apk add openjdk8 busybox tzdata curl bash && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo Asia/Shanghai > /etc/timezone && \
    apk del tzdata && \
    rm -rf /tmp/* /var/check/apk/*
```



build ：

 ```bash
 docker build . -t alpine-openjdk8:v2
 ```




## 参考(照抄)

- [https://blog.csdn.net/java1856905/article/details/89379780](https://blog.csdn.net/java1856905/article/details/89379780)
- [https://blog.csdn.net/john1337/article/details/113846075](https://blog.csdn.net/john1337/article/details/113846075)

---

If you like TeXt, don't forget to give me a star. :star2:

[![Star This Project](https://img.shields.io/github/stars/kitian616/jekyll-TeXt-theme.svg?label=Stars&style=social)](https://github.com/kitian616/jekyll-TeXt-theme/)
