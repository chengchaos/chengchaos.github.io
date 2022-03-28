---
title: ngrok how to
key: 2021-10-16
tags: linux centos nginx optimize
---

ngrok 是一个反向代理，通过在公共端点和本地运行的 Web 服务器之间建立一个安全的通道，实现内网主机的服务可以暴露给外网。ngrok 可捕获和分析所有通道上的流量，便于后期分析和重放，所以ngrok可以很方便地协助服务端程序测试。

<!--more-->

## 1. 特点

- 官方维护，一般较为稳定
- 跨平台，闭源
- 有流量记录和重发功能

## 2. 使用方法

1, 进入ngrok官网（[https://ngrok.com/](https://ngrok.com/），注册 ngrok 账号并下载 ngrok；

2, 根据官网给定的授权码，运行如下授权命令；

```sh
./ngrok authtoken 1hAotxhmORtzCYvUc3BsxDBPh1H_******************
./ngrok http 80即可将机器的80端口http服务暴露到公网，并且会提供一个公网域名。

```

还可以通过一些命令将内网的文件和其他TCP服务 暴露到公网中。



有授权的设置文件共享

ngrok http -auth="user:password" file:///Users/alan/share



无授权的设置文件共享

ngrok http "file:///C:\Users\alan\Public Folder"



将主机的3389的TCP端口暴露到公网

ngrok tcp 3389

更多使用方法参考：https://ngrok.com/docs

## 参考

- [常见内网穿透工具使用总结](https://mp.weixin.qq.com/s/ZLqL52TV7w7-3mzph4RJCQ)

EOF
