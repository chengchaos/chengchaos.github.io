---
title: CentOS 8 网络配置
key: 2020-08-17
tags: network centos8 
---

CentOS 8 已经发布了，下载了一个体验一下，新安装好的 CentOS 8 默认网卡是没有启动的，安装好后需要先配置网络。在 `/etc/sysconfig/network-scripts` 目录下存放着网卡的配置文件，文件名称是 `ifcfg-网卡名称` 。

<!--more-->

## 修改配置文件

```bash
$ sudo -i
# vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

如果需要配置动态 IP，则按照以下修改方法修改

```
# 网卡配置文件按默认配置
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=e4987998-a4ce-4cef-96f5-a3106a97f5bf
DEVICE=enp0s3
ONBOOT=no  #如果使用dhcp分配ip的话，只需要将这里no改为yes，然后重启网络服务就行

```

如果需要配置静态 IP，则按照以下修改方法修改

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static   #将dhcp修改为stati表示使用静态ip
DEFROUTE=yes
IPADDR=192.168.128.129   #设置IP地址
NETMASK=255.255.255.0    #设置子网掩码
GATEWAY=192.168.128.1    #设置网关
DNS1=114.114.114.114     #设置dns
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=e4987998-a4ce-4cef-96f5-a3106a97f5bf
DEVICE=enp0s3
ONBOOT=yes  #将no改为yes
```

## 重启网络服务

使用 `nmcli c reload` 命令重启网络服务，nmcli 命令的参数如下所示：


```
$ nmcli  -h
Usage: nmcli [OPTIONS] OBJECT { COMMAND | help }

OPTIONS
  -o[verview]                                    overview mode (hide default values)
  -t[erse]                                       terse output
  -p[retty]                                      pretty output
  -m[ode] tabular|multiline                      output mode
  -c[olors] auto|yes|no                          whether to use colors in output
  -f[ields] <field1,field2,...>|all|common       specify fields to output
  -g[et-values] <field1,field2,...>|all|common   shortcut for -m tabular -t -f
  -e[scape] yes|no                               escape columns separators in values
  -a[sk]                                         ask for missing parameters
  -s[how-secrets]                                allow displaying passwords
  -w[ait] <seconds>                              set timeout waiting for finishing operations
  -v[ersion]                                     show program version
  -h[elp]                                        print this help

OBJECT
  g[eneral]       NetworkManager's general status and operations
  n[etworking]    overall networking control
  r[adio]         NetworkManager radio switches
  c[onnection]    NetworkManager's connections  # 网络管理一般使用 nmcli c
  d[evice]        devices managed by NetworkManager
  a[gent]         NetworkManager secret agent or polkit agent
  m[onitor]       monitor NetworkManager changes

```

网络管理一般使用 `nmclli c` ，用法如下：

```bash
$ nmcli c -h
Usage: nmcli connection { COMMAND | help }

COMMAND := { show | up | down | add | modify | clone | edit | delete | monitor | reload | load | import | export }

  show [--active] [--order <order spec>]
  show [--active] [id | uuid | path | apath] <ID> ...

  up [[id | uuid | path] <ID>] [ifname <ifname>] [ap <BSSID>] [passwd-file <file with passwords>]

  down [id | uuid | path | apath] <ID> ...

  add COMMON_OPTIONS TYPE_SPECIFIC_OPTIONS SLAVE_OPTIONS IP_OPTIONS [-- ([+|-]<setting>.<property> <value>)+]

  modify [--temporary] [id | uuid | path] <ID> ([+|-]<setting>.<property> <value>)+

  clone [--temporary] [id | uuid | path ] <ID> <new name>

  edit [id | uuid | path] <ID>
  edit [type <new_con_type>] [con-name <new_con_name>]

  delete [id | uuid | path] <ID>

  monitor [id | uuid | path] <ID> ...

  reload

  load <filename> [ <filename>... ]

  import [--temporary] type <type> file <file to import>

  export [id | uuid | path] <ID> [<output file>]
```

## 使用 nmcli 管理网络

查看网卡信息

```bash
# nmcli connection

NAME UUID TYPE DEVICE
ens33 a92fa07b-9b68-4d2b-a2e7-e55146099b1b ethernet ens33
ens36 418da202-9a8c-b73c-e8a1-397e00f3c6b2 ethernet ens36

# nmcli con xxx
```

显示具体的网络接口信息

```bash
# nmcli connection show xxx
```


显示所有活动连接

```bash
# nmcli connection show --active 
```

删除一个网卡连接

```bash
# nmcli connection delete xxx
```


给xxx添加一个IP（IPADDR）

```bash
# nmcli connection modify xxx ipv4.addresses 192.168.0.58
```


给xxx添加一个子网掩码（NETMASK）

```bash
# nmcli connection modify xxx ipv4.addresses 192.168.0.58/24
```


IP获取方式设置成手动（BOOTPROTO=static/none）

```bash
# nmcli connection modify xxx ipv4.method manual
```


添加一个ipv4

```bash
# nmcli connection modify xxx +ipv4.addresses 192.168.0.59/24
```


删除一个ipv4

```bash
# nmcli connection modify xxx -ipv4.addresses 192.168.0.59/24
```


添加DNS

```bash
# nmcli connection modify xxx ipv4.dns 114.114.114.114
```


删除DNS

```bash
# nmcli connection modify xxx -ipv4.dns 114.114.114.114
```


添加一个网关（GATEWAY）

```bash
# nmcli connection modify xxx ipv4.gateway 192.168.0.2
```


可一块写入：

```bash
# nmcli connection modify xxx ipv4.dns 114.114.114.114 ipv4.gateway 192.168.0.2
```


添加DNS

```bash
# nmcli connection modify xxx ipv4.dns 114.114.114.114
```


删除DNS

```bash
# nmcli connection modify xxx -ipv4.dns 114.114.114.114
```

添加一个网关（GATEWAY）

```bash
# nmcli connection modify xxx ipv4.gateway 192.168.0.2
```

可一块写入：

```bash
# nmcli connection modify xxx ipv4.dns 114.114.114.114 ipv4.gateway 192.168.0.2
```


使用nmcli重新回载网络配置

```bash
# nmcli c reload
```


如果之前没有xxx的connection，则上一步reload后就已经自动生效了

```bash
# nmcli c up xxx
```


参考：

- https://www.cnblogs.com/ay-a/p/11828607.html
- https://www.cnblogs.com/RXDXB/p/11660184.html


EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





