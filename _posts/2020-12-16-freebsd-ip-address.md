---
title: FreeBSD 配置 IP 地址
key: 2020-12-16
tags: FreeBSD IP
---

如题

<!--more-->

FreeBSD 使用 `ifconfig -a` 来查看网卡名称。

## 手动指定

```bash
ifconfig em1 inet 192.168.56.102/24 up
route add default 192.168.56.1
```

立即生效，但重启后失效。

## 配置文件

```bash
vim /etc/rc.conf

ifconfig_em1="inet 192.168.56.102 netmask 255.255.255.0"
defaultrouter="192.168.56.1"
hostname="myfreebsd"
ifconfig_wi0="inet 192.168.1.13 netmask 255.255.255.0"
```

修改 /etc/rc.conf 配置文件之后可以通过以下几个命令使其生效

```bash
sh /etc/tc
/etc/netstart
reboot
```

## DHCP

```bash
vim /etc/rc.conf
ifconfig_em1="DHCP"
```

释放：

```bash
# Release the current lease and exit the client.
dhclient -r
```

指定网卡接口通过 DHCP 获取 IP 地址：

```bash
# Starts the dhclient process for interface em1
dhclient em1
```

指定某个 interface 在 DHCP 获取不到时，使用固定 IP

```bash
vim /etc/dhclient.conf

alias {
    interface "em1";
    fixed-address 192.168.40.1;
    option subnet-mask 255.255.255.0;
}
```

## DNS

```bash
vim /etc/resolv.conf

nameserver 202.98.0.68
nameserver 8.8.8.8

```

## 路由

查看路由：

```bash
netstat -rn
```

添加路由：

```bash
route add default 192.168.8.1
```

修改路由：

```bash
route delete default 192.168.8.1
route add default 192.168.0.1
```

转自：

- [https://blog.csdn.net/weixin_34204057/article/details/94494105](https://blog.csdn.net/weixin_34204057/article/details/94494105)

EOF

---

Power by TeXt.
