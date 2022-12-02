---
title: MAC 设置 DNS 地址
key: 2022-12-01
tags: MAC DNS CLI
---

Omitted ...

<!--more-->

## 0x01 列出网络设备

```bash
➜  ~ networksetup -listallnetworkservices
An asterisk (*) denotes that a network service is disabled.
AX88x72A
Wi-Fi
iPhone USB
Thunderbolt Bridge
```

## 0x02 设置 DNS

命令格式：  `networksetup -setdnsservers <网络设备> <DNS 服务器地址>`，命令执行后没有任何反馈说明设置成功了。 例如：

```bash
networksetup -setdnsservers Wi-Fi 192.168.1.1
```
## 0x03 查看 DNS

命令格式：  `networksetup -getdnsservers <网络设备> `。 例如：

```bash
➜  ~ networksetup -getdnsservers Wi-Fi
192.168.1.1
```
## 0x04 删除 DNS

命令格式：  `networksetup -setdnsservers <网络设备> empty`。 例如：

```bash
➜  ~ networksetup -setdnsservers Wi-Fi empty
192.168.1.1
```
## 0x05 IP 转发

```bash
➜  ~ sudo sysctl -a | grep forwarding
Password:
net.inet.ip.forwarding: 0
net.inet6.ip6.forwarding: 0
➜  ~ sudo sysctl -w net.inet.ip.forwarding=1
net.inet.ip.forwarding: 0 -> 1
```

### 端口转发

编辑 /etc/pf.conf 文件，在文件底部添加： 

> 在这句代码下一行添加：
> `rdr-anchor "com.apple/*"`

```bash
rdr pass on en0 inet proto tcp from any to any port 80 -> 127.0.0.1 port 8000
```
使配置生效：

```bash
sudo pfctl -ef /etc/pf.conf
```

> 后面还没有测试完成， 儿子睡了， 

参考：

- 苹果Mac怎么使用命令设置dns教程
 <https://jingyan.baidu.com/article/1612d500e03f4ee20e1eee02.html>
- <https://www.jianshu.com/p/6b13316eccab>
EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>


