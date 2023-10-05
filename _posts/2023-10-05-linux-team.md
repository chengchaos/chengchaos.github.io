---
title: Linux 网卡 team 配置
key: 2023-10-05
tags: Linux network
---

> 本文主要记录了怎样设置 Linux 的 team 。

Search suggest: linux nmcli network team

<!--more-->
## 0x01 介绍

功能：

- 网络组将多个网卡聚合在一起，从而实现冗错和提高吞吐量
- 网络组不同于旧版中 bonding 技术，提供更好的性能和扩展性
- 网络组由内核驱动和 teamd 守护进程实现.

特性：

- 启动网络组接口不会自动启动网络组中的 port 接口
- 启动网络组接口中的 port 接口总会自动启动网络组接口
- 禁用网络组接口会自动禁用网络组中的 port 接口
- 没有 port 接口的网络组接口可以启动静态 IP 连接
- 启用 DHCP 连接时，没有 port 接口的网络组会等待 port 接口的加入

### runner 方式：

#### 1.roundrobin 【mode 0】轮转策略 (balance-rr)

特点：

- 从头到尾顺序的在每一个 slave 接口上面发送数据包，轮询方式往每条链路发送报文，基于 per packet 方式发送。服务上 ping 一个相同地址：1.1.1.1 双网卡的两个网卡都有流量发出。负载到两条链路上，说明是基于 per packet 方式 ，进行轮询发送。
- 提供负载均衡和容错的能力，当有链路出问题，会把流量切换到正常的链路上。
- **交换机端需要配置聚合口**

#### 2.activebackup【mode 1】活动-备份（主备）策略

特点：

- 一个端口处于主状态 ，一个处于从状态，所有流量都在主链路上处理，从链路不会有任何流量。当主端口 down 掉时，从端口接手主状态。
- **不需要交换机端支持**

#### 3.loadbalance【mode 2】限定流量

特点：

- 该模式将限定流量，以保证到达特定对端的流量总是从同一个接口上发出。既然目的地是通过 MAC 地址来决定的，因此该模式在“本地”网络配置下可以工作得很好。
- 如果所有流量是通过单个路由器（比如 “网关”型网络配置，只有一个网关时，源和目标 mac 都固定了，那么这个算法算出的线路就一直是同一条，那么这种模式就没有多少意义了。），那该模式就不是最好的选择。
- 和 balance-rr 一样，交换机端口需要能配置为“port channel”。这模式是通过源和目标 mac 做 hash 因子来做 xor 算法来选路的。
- **交换机端需要配置聚合口**

#### 4.broadcast【mode 3】广播策略

这种模式一个报文会复制两份往 bond 下的两个接口分别发送出去,当有对端交换机失效，我们感觉不到任何 downtime,但此法过于浪费资源;不过这种模式有很好的容错机制。

此模式适用于金融行业，因为他们需要高可靠性的网络，不允许出现任何问题

适用于拓扑，两个接口分别接入两台交换机，并且属于不同的 vlan，当一边的网络出现故障不会影响服务器另一边接入的网络正常工作。而且故障过程是0丢包

#### 5.lacp (implements the 802.3ad Link Aggregation ControlProtocol)【mode 4】

802.3ad 模式是 IEEE 标准，因此所有实现了 802.3ad 的对端都可以很好的互操作。802.3ad  协议包括聚合的自动配置，因此只需要很少的对交换机的手动配置（要指出的是，只有某些设备才能使用 802.3ad）。802.3ad 标准也要求帧按顺序（一定程度上）传递，因此通常单个连接不会看到包的乱序。

缺点：

标准要求所有设备在聚合操作时，要在同样的速率和双工模式，而且，和除了 balance-rr 模式外的其它 bonding 负载均衡模式一样，任何连接都不能使用多于一个接口的带宽。 此外，linux bonding 的 802.3ad 实现通过对端来分发流量（通过 MAC 地址的 XOR 值），因此在“网关”型配置下，所有外出（Outgoing）流量将使用同一个设备。进入（Incoming）的流量也可能在同一个设备上终止，这依赖于对端 802.3ad 实现里的均衡策略。在“本地”型配置下，路两将通过 bond里的设备进行分发。

应用拓扑同 mode 0,和 mode 2一样，不过这种模式除了配置 port channel 之外还要在 port channel 聚合口下开启 LACP 功能，成功协商后，两端可以正常通信。否则不能使用。

## 0x02 创建 team

语法： `nmcli con add type team con-name CNAME ifname INAME [config JSON]`

- CNAME 连接名， 
- INAME 接口名 
- JSON 指定runner方式
- 格式： `{"runner": {"name": "METHOD"}}`

METHOD 可以是 `broadcast`, `roundrobin`, `activebackup`, `loadbalance`, `lacp`。

准备环境：centos7 系统 2块网卡：相同仅主机工作环境(以下为 ens33\ens37 网卡名称)

```bash
nmcli connection add type team con-name team0 ifname team0 config
 {"runner":{"name":"activebackup"}} ipv4.method manual ipv4.addresses 192.168.32.100/24 connection.autoconnect yes
```

## 0x03 创建 port

语法：`nmcli connection add type team-slave con-name CNAME ifname INAME masterTEAM`

- CNAME 连接名 
- INAME 网络接口名(team0-ens33、team0-ens37) 
- TEAM 网络组接口名(team0)

> 注意：连接名若不指定，默认为 `team-slave-IFACE`

```bash
nmcli connection add con-name team0-ens33 type team-slave ifname ens33 master team0
# 把 ens33 网卡创建为 team0 网络组的子接口
nmcli connection add con-name team0-ens37 type team-slave ifname ens37 master team0
# 把 ens37 网卡创建为 team0 网络组的子接口
```

## 0x04 启用 team0 的两个 port

```bash
nmcli connection up team0-ens33
nmcli connection up team0-ens37
```

## 0x05 删除 team

语法： 

- 1、 `nmcli connection down team0`
- 2、 `teamdctl team0 state` (查看当前网络组team主网卡 和热备网卡哪个在工作)
- 3、 `nmcli connection delete team0-ens33 / nmcli connection delete team0-ens37`
- 4、 `nmcli connection show` (查看网络组 删除后就看不到了)

## 0x06 也可以：全手动配置 team 双网卡

```bash
vi /etc/sysconfig/network-scripts/team

DEVICE=“team”
DEVICETYPE=“Team”
ONBOOT=“yes”
BOOTPROTO=static
NETMASK=255.255.255.0
IPADDR=ip
GATEWAY=网关
TEAM_CONFIG='{"runner": {"name": "activebackup"}}'


vim /etc/sysconfig/network-scripts/ifcfg-em1 # 编辑文件 ifcfg-em1

DEVICE="em1“
DEVICETYPE=“TeamPort”
ONBOOT=“yes”
TEAM_MASTER=“team”

vim /etc/sysconfig/network-scripts/ifcfg-em2 # 编辑文件 ifcfg-em2

DEVICE=“em2”
DEVICETYPE=“TeamPort”
ONBOOT=“yes”
TEAM_MASTER=“team”
```

## 0x07 实战

```bash
# nmcli connection add type team \
  con-name team0 \
  ifname team0 \
  config '{"runner":{"name":"activebackup"}}' \
  ipv4.method manual \
  ipv4.addresses 192.168.0.100/24 \
  connection.autoconnect yes
Connection 'team0' (b674d86e-324c-4b6f-8ce5-64c50339eb6f) successfully added.

# nmcli connection add con-name team0-eth0 \
  type team-slave \
  ifname eth0 \
  master team0
Connection 'team0-eth0' (0d6b9a6c-2405-4353-8e1e-d5cd74028db0) successfully added.
# nmcli connection add con-name team0-ens19 \
  type team-slave \
  ifname ens19 \
  master team0
Connection 'team0-ens19' (46a30945-a765-4c71-9ec3-8214ff4f81b1) successfully added.
# nmcli connection up team0-eth0
```

添加 DNS 

```
# nmcli help
Usage: nmcli [OPTIONS] OBJECT { COMMAND | help }

OPTIONS
  -a, --ask                                ask for missing parameters
  -c, --colors auto|yes|no                 whether to use colors in output
  -e, --escape yes|no                      escape columns separators in values
  -f, --fields <field,...>|all|common      specify fields to output
  -g, --get-values <field,...>|all|common  shortcut for -m tabular -t -f
  -h, --help                               print this help
  -m, --mode tabular|multiline             output mode
  -o, --overview                           overview mode
  -p, --pretty                             pretty output
  -s, --show-secrets                       allow displaying passwords
  -t, --terse                              terse output
  -v, --version                            show program version
  -w, --wait <seconds>                     set timeout waiting for finishing operations

OBJECT
  g[eneral]       NetworkManager's general status and operations
  n[etworking]    overall networking control
  r[adio]         NetworkManager radio switches
  c[onnection]    NetworkManager's connections
  d[evice]        devices managed by NetworkManager
  a[gent]         NetworkManager secret agent or polkit agent
  m[onitor]       monitor NetworkManager changes

# nmcli c help
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

# nmcli c modify help
Usage: nmcli connection modify { ARGUMENTS | help }

ARGUMENTS := [id | uuid | path] <ID> ([+|-]<setting>.<property> <value>)+

Modify one or more properties of the connection profile.
The profile is identified by its name, UUID or D-Bus path. For multi-valued
properties you can use optional '+' or '-' prefix to the property name.
The '+' sign allows appending items instead of overwriting the whole value.
The '-' sign allows removing selected items instead of the whole value.

Examples:
nmcli con mod home-wifi wifi.ssid rakosnicek
nmcli con mod em1-1 ipv4.method manual ipv4.addr "192.168.1.2/24, 10.10.1.5/8"
nmcli con mod em1-1 +ipv4.dns 8.8.4.4
nmcli con mod em1-1 -ipv4.dns 1
nmcli con mod em1-1 -ipv6.addr "abbe::cafe/56"
nmcli con mod bond0 +bond.options mii=500
nmcli con mod bond0 -bond.options downdelay

# nmcli c show
NAME         UUID                                  TYPE      DEVICE
team0        b674d86e-324c-4b6f-8ce5-64c50339eb6f  team      team0
team0-ens19  46a30945-a765-4c71-9ec3-8214ff4f81b1  ethernet  ens19
team0-eth0   0d6b9a6c-2405-4353-8e1e-d5cd74028db0  ethernet  eth0
eth0         e684cb85-db50-4e25-b17d-995cf44e8f13  ethernet  --

# nmcli c modify ipv4.dns team0 "114.114.114.114 8.8.8.8"

# cat /etc/sysconfig/network-scripts/ifcfg-team0
TEAM_CONFIG="{\"runner\":{\"name\":\"activebackup\"}}"
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR=192.168.1.100
PREFIX=24
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=team0
UUID=b674d86e-324c-4b6f-8ce5-64c50339eb6f
DEVICE=team0
ONBOOT=yes
DEVICETYPE=Team
DNS1=192.168.1.1
DNS2=8.8.8.8

# cat /etc/resolv.conf 
# Generated by NetworkManager
nameserver 192.168.1.1
nameserver 114.114.114.114
```



## Appendix

以下是参考链接。

- [网卡 team配置](https://www.cnblogs.com/backups/p/linux_team.html)

_EOF_

