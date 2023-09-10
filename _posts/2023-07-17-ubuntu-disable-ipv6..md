---
title: Ubuntu disabie IPv6
key: 2023-07-17
tags: Ubuntu disabie IPv6
---

> Search suggest： disabie IPv6 Ubuntu



<!--more-->

## 0x01 临时方式

```bash
    sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
    sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
    sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1

```

## 0x02 永久方式

编辑 `/etc/sysctl.conf` 增加如下内容

```bash
    net.ipv6.conf.all.disable_ipv6=1
    net.ipv6.conf.default.disable_ipv6=1
    net.ipv6.conf.lo.disable_ipv6=1
```

应用：

```bash
    sudo sysctl -p
```

## 0x03 编辑 /etc/sysconfig/network

```bash


# Created by anaconda
NETWORKING=yes
GATEWAY=19.37.33.1


```

## 0x04 参(zhao)考(chao)

- <https://linux.cn/article-12689-1.html>

EOF