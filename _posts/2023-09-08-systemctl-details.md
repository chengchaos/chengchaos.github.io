---
title: Linux systemctl 服务启动脚本
key: 2023-09-08
tags: Linux Systemd systemctl
---

[!Systemd](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016030701.gif)

> 本文主要记录了怎样编写 systemctl 启动脚本，没有别的意思。

Search suggest: linux Systemd systemctl

<!--more-->
## 0x01 由来

历史上，Linux 的启动一直采用init进程。

下面的命令用来启动服务。

```bash
$ sudo /etc/init.d/apache2 start
# 或者
$ service apache2 start
```

这种方法有两个缺点。

一是启动时间长。init 进程是串行启动，只有前一个进程启动完，才会启动下一个进程。

二是启动脚本复杂。init 进程只是执行启动脚本，不管其他事情。脚本需要自己处理各种情况，这往往使得脚本变得很长。

## 0x01 Systemd 简介

Centos7 开机第一个程序从 init 完全换成了 systemd 这种启动方式，同 centos 5/6 已经是实质差别。systemd 是靠管理 unit 的方式来控制开机服务，开机级别等功能。

在 `/usr/lib/systemd/system` 目录下包含了各种 unit 文件，有 service 后缀的服务 unit，有 target 后缀的开机级别 unit 等，这里介绍关于 service 后缀的文件。因为 systemd 在开机要想执行自启动，都是通过这些 `*.service`  的 unit 控制的，服务又分为系统服务（system）和用户服务（user）。

- 系统服务：开机不登陆就能运行的程序（常用于开机自启）。
- 用户服务：需要登陆以后才能运行的程序。

对于那些支持 Systemd 的软件，安装的时候，会自动在 `/usr/lib/systemd/system` 目录添加一个配置文件。

   
```base
systemctl enable httpd

```

上面的命令相当于在 `/etc/systemd/system` 目录添加一个符号链接，指向 `/usr/lib/systemd/system/httpd.service` 文件。

这是因为开机时，Systemd 只执行 /etc/systemd/system 目录里面的配置文件。这也意味着，如果把修改后的配置文件放在该目录，就可以达到覆盖原始配置的效果。

一个例子

```bash
systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor pre>
     Active: active (running) since Thu 2023-08-31 13:30:47 CST; 1 week 1 day a>
   Main PID: 27917 (nginx)
      Tasks: 2 (limit: 4915)
     CGroup: /system.slice/nginx.service
             ├─27917 nginx: master process /usr/sbin/nginx -g daemon off;
             └─27918 nginx: worker process

```

说明

- Loaded行：配置文件的位置，是否设为开机启动
- Active行：表示正在运行
- Main PID行：主进程ID
- Status行：由应用本身（这里是 httpd ）提供的软件当前状态
- CGroup块：应用的所有子进程
- 日志块：应用的日志

## 配置文件详解

### [Unit] 区块：启动顺序与依赖关系

- Description 字段：给出当前服务的简单描述。
- Documentation 字段：给出文档位置。
- After 字段：如果 network.target 或 sshd-keygen.service 需要启动，那么sshd.service应该在它们之后启动。
- Before 字段：定义 sshd.service 应该在哪些服务之前启动。

> 注： After 和 Before 字段只涉及启动顺序，不涉及依赖关系。

### [Service] 区块：启动行为

启动命令

- ExecStart字段：定义启动进程时执行的命令
- ExecReload字段：重启服务时执行的命令
- ExecStop字段：停止服务时执行的命令
- ExecStartPre字段：启动服务之前执行的命令
- ExecStartPost字段：启动服务之后执行的命令
- ExecStopPost字段：停止服务之后执行的命令

> 注：所有的启动设置之前，都可以加上一个连词号（`-`），表示"抑制错误"，即发生错误的时候，不影响其他命令的执行。比如 `EnvironmentFile=-/etc/sysconfig/sshd`（注意等号后面的那个连词号），就表示即使 `/etc/sysconfig/sshd` 文件不存在，也不会抛出错误。

> 注：[Service]中的启动、重启、停止命令全部要求使用绝对路径！

### 启动类型

Type 字段定义启动类型。它可以设置的值如下：

- simple（默认值）：ExecStart字段启动的进程为主进程
- forking：ExecStart字段将以fork()方式启动，此时父进程将会退出，子进程将成为主进程（后台运行）
- oneshot：类似于simple，但只执行一次，Systemd 会等它执行完，才启动其他服务
- dbus：类似于simple，但会等待 D-Bus 信号后启动
- notify：类似于simple，启动结束后会发出通知信号，然后 Systemd 再启动其他服务
- idle：类似于simple，但是要等到其他任务都执行完，才会启动该服务。一种使用场合是为让该服务的输出，不与其他服务的输出相混合

### 重启行为

Service区块有一些字段，定义了重启行为：

- KillMode字段：定义 Systemd 如何停止 sshd 服务：
- control-group（默认值）：当前控制组里面的所有子进程，都会被杀掉
- process：只杀主进程
- mixed：主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号
- none：没有进程会被杀掉，只是执行服务的 stop 命令。

Restart字段：定义了 sshd 退出后，Systemd 的重启方式

上面的例子中，Restart 设为 on-failure，表示任何意外的失败，就将重启 sshd。如果 sshd 正常停止（比如执行systemctl stop命令），它就不会重启。

Restart字段可以设置的值如下。


- no（默认值）：退出后不会重启
- on-success：只有正常退出时（退出状态码为0），才会重启
- on-failure：非正常退出时（退出状态码非0），包括被信号终止和超时，才会重启
- on-abnormal：只有被信号终止和超时，才会重启
- on-abort：只有在收到没有捕捉到的信号终止时，才会重启
- on-watchdog：超时退出，才会重启
- always：不管是什么退出原因，总是重启

> 注：对于守护进程，推荐设为on-failure。对于那些允许发生错误退出的服务，可以设为on-abnormal。


RestartSec 字段：表示 Systemd 重启服务之前，需要等待的秒数。

### [Install] 区块

Install区块，定义如何安装这个配置文件，即怎样做到开机启动。

- WantedBy 字段：表示该服务所在的 Target。 Target的含义是服务组，表示一组服务。

WantedBy=multi-user.target 指的是：sshd 所在的 Target 是multi-user.target。

这个设置非常重要，因为执行 `systemctl enable sshd.service` 命令时，sshd.service 的一个符号链接，就会放在 /etc/systemd/system 目录下面的  multi-user.target.wants 子目录之中。

Systemd 有默认的启动 Target。

```bash
systemctl get-default
### 输出 multi-user.target
```

上面的结果表示，默认的启动 Target 是 `multi-user.target`。在这个组里的所有服务，都将开机启动。这就是为什么`systemctl enable` 命令能设置开机启动的原因。

使用 Target 的时候，`systemctl list-dependencies` 命令和 `systemctl isolate` 命令也很有用。

```bash
#查看 multi-user.target 包含的所有服务
systemctl list-dependencies multi-user.target

#切换到另一个 target
#shutdown.target 就是关机状态
systemctl isolate shutdown.target
```


一般来说，常用的 Target 有两个：

- multi-user.target：表示多用户命令行状态；
- graphical.target：表示图形用户状态，它依赖于multi-user.target。



| 配置 | 说明 | 依赖 |
| ---- | ---- | ---- |
| Unit | | 
| After | 表示服务需要在***服务启动之后执行 |  无依赖 |
| Before | 表示服务需要在***服务启动之前执行 | 无依赖 |
| Wants | 弱依赖关系 | |
| Requires | 强依赖关系 | ***停止之后本服务也必须停止 |


| Service | | |
| ---- | ---- | ---- |
| EnvironmentFile | 环境参数文件 | EnvironmentFile=/etc/sysconfig/sshd，以key=value的形式保存,，以$key形式读取 |
| ExecStart | 启动进程时执行的命令 | |
| ExecReload | 重启服务时执行的命令 | |
| ExecStop | 停止服务时执行的命令 | |
| ExecStartPre | 启动服务之前执行的命令 | |
| ExecStartPost | 启动服务之后执行的命令 | |
| ExecStopPost | 停止服务之后执行的命令 | |


| Type | 说明 |
| ---- | ---- |
| simple（默认值）| ExecStart字段启动的进程为主进程 |
| forking | ExecStart字段将以fork()方式启动，此时父进程将会退出，子进程将成为主进程 |
| oneshot | 类似于simple，但只执行一次，Systemd 会等它执行完，才启动其他服务 |
| dbus | 类似于simple，但会等待 D-Bus 信号后启动 |
| notify | 类似于simple，启动结束后会发出通知信号，然后 Systemd 再启动其他服务 |
| idle | 类似于simple，但是要等到其他任务都执行完，才会启动该服务。一种使用场合是为让该服务的输出，不与其他服务的输出相混合 |

| KillMode | 说明 |
| ---- | ---- |
| control-group（默认值）| 当前控制组里面的所有子进程，都会被杀掉 |
| process | 只杀主进程 |
| mixed | 主进程将收到 `SIGTERM` 信号，子进程收到 `SIGKILL` 信号 |
| none | 没有进程会被杀掉，只是执行服务的 stop 命令。|

| Restart | 说明 |
| ---- | ---- |
| no（默认值）| 退出后不会重启 |
| on-success | 只有正常退出时（退出状态码为0），才会重启 |
| on-failure | 正常退出时（退出状态码非0），包括被信号终止和超时，才会重启 |
| on-abnormal | 只有被信号终止和超时，才会重启 |
| on-abort | 只有在收到没有捕捉到的信号终止时，才会重启 |
| on-watchdog | 超时退出，才会重启 |
| always | 不管是什么退出原因，总是重启 |

示例

```bash
[Unit]
Description=MonitorJSCloud
After=network.service

[Service]
Type=simple
ExecStart=/usr/bin/python/root/test/ping_test.py
Restart=always
RestartSec=10
PrivateTmp=true

[Install]
WantedBy=multi-user.target 
```


## 常用命令

修改配置文件以后，需要重新加载配置文件，然后重新启动相关服务。

   
```bash
### 重新加载配置文件
$ systemctl daemon-reload

### 查看配置文件
$ systemctl cat sshd.service

### 查看服务状态
$ systemctl status sshd.service

```


## Appendix

以下是参考链接。

- [编写使用systemctl启动服务脚本](https://www.cnblogs.com/liufarui/p/10960610.html)
- [CentOS7使用systemctl添加自定义服务](https://www.jianshu.com/p/79059b06a121)
- [Systemd 入门教程：命令篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
- [Systemd 入门教程：实战篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)
EOF