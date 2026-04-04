---
title: 解决 SSH 登录很慢问题
key: 2021-08-15
tags: ssh sshd
---


用ssh连其他linux机器，会等待10-30秒才有提示输入密码。严重影响工作效率。登录很慢，登录上去后速度正常，这种情况主要有两种可能的原因：

<!--more-->

## 1. DNS反向解析的问题

OpenSSH 在用户登录的时候会验证 IP，它根据用户的 IP 使用反向 DNS 找到主机名，再使用 DNS 找到 IP 地址，最后匹配一下登录的 IP 是否合法。如果客户机的 IP 没有域名，或者 DNS 服务器很慢或不通，那么登录就会很花时间。

解决办法：

在目标服务器上修改 sshd 服务器端配置，设置 `UseDNS` 为 no 并重启 sshd

```sh
vi /etc/ssh/sshd_config

#UseDNS yes
UseDNS no

```

当然也可以通过提供 DNS 正确反向解析的方法解决，有如下两种思路

（1） 在 server 上 /etc/hosts 文件中把常用的 ip 和 hostname 加入，然后在 /etc/nsswitch.conf 看看程序是否先查询 hosts 文件（一般缺省是这样）。

修改 server 上的 hosts 文件，将目标机器的 IP 和域名加上去。或者让本机的 DNS 服务器能解析目标地址。

```sh
vi /etc/hosts

192.168.12.16  ourdev
```

其格式是 `目标机器IP 目标机器名称` 这种方法促效。没有延迟就连上了。不过如果给每台都加一个域名解析，挺辛苦的。但在 windows 下用 putty 或 secure-crt 时可以采用这种方法。

（2）起一台dns服务器（可以是本机），加入反向解析，把这个 dns 服务器加入到 /etc/resolv.conf 中。

## 2. 关闭ssh的gssapi认证

用 `ssh -v user@server` 可以看到登录时有如下信息：

debug1: Next authentication method: gssapi-with-mic

debug1: Unspecified GSS failure. Minor code may provide more information

> 注：`ssh -vvv user@server` 可以看到更细的 debug 信息

解决办法：

在客户端上修改 ssh 客户端配置 (**注意不是 sshd_conf**）并重启 sshd

```sh
vi /etc/ssh/ssh_config，
GSSAPIAuthentication no  

```

可以使用

```sh
ssh -A -o StrictHostKeyChecking=no -o GSSAPIAuthentication=no -p 32200 username@server_ip
```

GSSAPI ( Generic Security Services Application Programming Interface) 是一套类似 Kerberos 5 的通用网络安全系统接口。该接口是对各种不同的客户端服务器安全机制的封装，以消除安全接口的不同，降低编程难度。但该接口在目标机器无域名解析时会有问题

使用 strace 查看后发现，ssh 在验证完 key 之后，进行 authentication gssapi-with-mic，此时先去连接 DNS 服务器，在这之后会进行其他操作。

原文

- [https://www.cnblogs.com/leoyang63/articles/12061136.html](https://www.cnblogs.com/leoyang63/articles/12061136.html)

EOF
