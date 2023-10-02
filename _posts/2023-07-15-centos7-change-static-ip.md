---
title: CentOS 7 change static IP
key: 2023-07-15
tags: CentOS7 static IP
---

> Suggest search： CentOS7 static IP



<!--more-->

## 0x01 查看网卡信息

```bash
ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether ba:ca:47:fc:ba:0d brd ff:ff:ff:ff:ff:ff

```

## 0x02 编辑配置文件

编辑网卡名称对应的配置文件， 这里是 ens

```bash
# vi /etc/sysconfig/network-scripts/ifcfg-你的网卡名字
vi /etc/sysconfig/network-scripts/ifcfg-ens18

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
#BOOTPROTO=dhcp
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens18
UUID=c11beef8-c312-4e1f-8b1e-7309df200a15
DEVICE=ens18
#ONBOOT=no
ONBOOT=yes
IPADDR=192.168.1.253
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
```

参数说明：

```bash
BOOTPROTO="static" # 使用静态IP地址，默认为dhcp IPADDR="19.37.33.66" # 设置的静态IP地址
NETMASK="255.255.255.0" # 子网掩码 
GATEWAY="19.37.33.1" # 网关地址 
DNS1="192.168.241.2" # DNS服务器（此设置没有用到，所以我的里面没有添加）

ONBOOT=yes  #设置网卡启动方式为 开机启动 并且可以通过系统服务管理器 systemctl 控制网卡
```

## 0x03 编辑 /etc/sysconfig/network

```bash


# Created by anaconda
NETWORKING=yes
GATEWAY=19.37.33.1


```

## 0x04 重启

EOF


