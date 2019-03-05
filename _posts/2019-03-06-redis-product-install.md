---
title: Redis 生产环境下的部署
key: 20190305
tags: Redis
---

如果将 Redis 服务部署到生产环境，则需要考虑更多的因素，这里我们重点讨论如何将 Redis 部署到 Linux 系统中。

本文是我学习《Redis 4.x Cookbook》的读书笔记。

<!--more-->


## 在 Linux 上部署 Redis

运行 Redis 最常见的操作系统是 Linux。在启动 Redis 实例之前，通常需要将一些 Linux 内核和操作系统级别的参数设置为前挡的值，以便在生产环境中发挥最高性能。

### 步骤

**1.设置与内存相关的内核参数**

使用如下命令设置内核参数：

```bash
~$ sudo sysctl -w vm.overcommit_memory=1
~$ sudo sysctl -w vm.swappiness=0
# echo 1 > /proc/sys/vm/overcommit_memory
```



使用如下命令来持久化保存这些参数：

```bash
# echo "vm.overcommit_memory=1" >> /etc/sysctl.conf
# echo "vm.swappiness=0" >> /etc/sysctl.conf
# sysctl -p

```

使用如下命令检查这些参数是否被设置：

```bash
~$ sudo sysctl vm.overcommit_memory vm.swappiness
vm.overcommit_memory = 1
vm.swappiness = 0
```

#### 说明

**什么是 OverCommit**

Linux 对大部分申请内存的请求都回复 "yes"，以便能跑更多更大的程序。因为申请内存后，并不会马上使用内存。这种技术叫做 Overcommit。


`mv.overcommit_memory` 文件指定了内核针对内存分配的策略，其志可以是： `0, 1, 2`.

- `0` （默认）：表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的内存，则允许内存申请；否则，内存申请失败，并把错误返回给应用进程。`0` 是启发式的 overcommitting handle, 会尽量减少 swap 的使用，root 可以分配比一般用户略多的内存。
- `1` ：表示内核允许分配所有的物理内存，而不管当前的内存状态如何，允许超过 `CommitLimit`，直到内存用完为止。**在数据库服务器上不建议设置为1，从而尽量避免使用 swap**。
- `2` ：表示不允许超过 `CommitLimit` 的值。

执行以下命令可以卡到 `commitLimit` 和 `Committed_As` 的参数设置：

```bash
~ $ grep -i commit /proc/meminfo
CommitLimit:     7175228 kB
Committed_AS:   20506432 kB
```

`CommitLimit` 是一个内存分配上限，
`Committed_As` 是已经分配的内存大小。

```
CommitLimit = 物理内存 * overcommit_ratio(默认50，即 50%) + swap
```

`vm.overcommit_ratio` ： 默认值为：50 （即50%）

这个参数值只有在vm.overcommit_memory=2的情况下，这个参数才会生效。


参考 [vm内核参数优化设置](http://www.cnblogs.com/wjoyxt/p/3777042.html)

**`swappiness`**


`swappiness` 的值的大小对如何使用 swap 分区是有着很大的联系的。`swappiness=0` 的时候表示最大限度使用物理内存，然后才是 swap空间，`swappiness＝100` 的时候表示积极的使用 swap 分区，并且把内存上的数据及时的搬运到swap空间里面。

CentOS7 的基本默认设置为 30，具体如下：
    
```bash
~ $ cat /proc/sys/vm/swappiness
30
```


也就是说，你的内存在使用到 70% 的时候，就开始出现有交换分区的使用。大家知道，内存的速度会比磁盘快很多，这样子会加大系统 IO，同时造的成大量页的换进换出，严重影响系统的性能，所以我们在操作系统层面，要尽可能使用内存，对该参数进行调整。



**2.禁用透明大页(transparent_hugepage)**

使用如下命令：

```bash
~ # cat >> /etc/rc.local << EOF
echo naver > /sys/kernel/mm/transparent_hugepage/enabled
echo naver > /sys/kernel/mm/transparent_hugepage/defrag 

EOF
```

#### 说明

**1.什么是Transparent HugePages？**

`Transparent Huge Pages` 是 RHEL6 以后的新特性。

为了提升性能，Kernel 会将程序缓存在内存中，每页内存以2M为单位。

想要有效的使用 THP，kernel 要在内存中找到一系列连续的物理内存来满足需求，也可能会对齐。

为了达到这个效果，系统新加了一个 khugepaged 进程，这个进程会偶尔尝试把正在使用的较小页面换到 hugepage 中。这样就能使 hugepage 使用达到最大化。

**2.如何关闭THP**

尽管THP的本意是为提升性能，但某些数据库厂商还是建议直接关闭 THP(比如说 Oracle、MongoDB等)，否则可能导致性能下降，内存锁，甚至系统重启等问题。

比较流行的关闭方法有两种：

第一种：在 `/etc/rc.local` 中加入如下两行

```bash
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
```

参考： [Linux的Transparent Hugepage与关闭方法](https://blog.csdn.net/ffwar/article/details/77853498 )



**3.网络的优化**

```bash
~ $ sudo sysctl -w net.core.somaxconn=65535
~ $ sudo sysctl -w  net.ipv4.tcp_max_syn_backlog=65535

```

#### 说明

参考 [Hadoop集群必配参数：net.core.somaxconn](https://www.tuicool.com/articles/nIZJFn)

`net.core.somaxconn` 的作用

`net.core.somaxconn` 是 Linux 中的一个kernel参数，表示 socket 监听（listen）的 backlog 上限。

什么是 backlog 呢？ backlog 就是 socket 的监听队列，当一个请求（request）尚未被处理或建立时，他会进入 backlog。而 socket server 可以一次性处理 backlog中的所有请求，处理后的请求不再位于监听队列中。

当server处理请求较慢，以至于监听队列被填满后，新来的请求会被拒绝。

在Hadoop 1.0中，参数 ipc.server.listen.queue.size 控制了服务端socket的监听队列长度，即backlog长度，默认值是128。而Linux的参数 net.core.somaxconn 默认值同样为128。当服务端繁忙时，如NameNode或JobTracker，128是远远不够的。这样就需要增大backlog，例如我们的3000台集群就将 ipc.server.listen.queue.size 设成了32768，为了使得整个参数达到预期效果，同样需要将kernel参数 net.core.somaxconn 设成一个大于等于32768的值。


**4.打开文件数**

```
~ $ su - redis
~ $ ulimit -n 288000
```

**注意**：

我们必须将 nofile 设置为一个小于 /proc/sys/fs/file-max 的值，因此，在设置之前，我们需要使用 cat 命令检查 /proc/sys/fs/file-max 的值。

要持久化保存这个参数，可以将他们添加到 /etc/security/limits.conf 文件中：

```
redis soft nofile 288000
redis hard nofile 288000
```

**`vm.panic_on_oom`**


`vm.panic_on_oom` 
值为 0 表示开启（默认）
值为 1 时表示关闭此功能

等于 0 时，表示当内存耗尽时，内核会触发 `OOM killer` 杀掉最耗内存的进程。

当 OOM Killer 被启动时，通过观察进程自动计算得出各当前进程的得分  `/proc/<PID>/oom_score`,分值越高越容易被kill掉。

而且计算分值时主要参照 `/proc/<PID>/oom_adj` ,  `oom_adj` 取值范围从 `-17` 到`15`，当等于 `-17` 时表示在任何时候此进程都不会被 oom killer kill 掉（适用于mysql）。

- `/proc/[pid]/oom_adj` ，该 pid 进程被 oom killer 杀掉的权重，介于  [-17,15] 之间，越高的权重，意味着更可能被 oom killer 选中，`-17` 表示禁止被 kill 掉。

- `/proc/[pid]/oom_score`，当前该 pid 进程的被 kill 的分数，越高的分数意味着越可能被 kill，这个数值是根据 oom_adj 运算后的结果，是 oom_killer 的主要参考。

sysctl 下有2个可配置选项：

```bash
vm.panic_on_oom = 0              # 内存不够时内核是否直接panic   vm.oom_kill_allocating_task = 1  # oom-killer是否选择当前正在申请内存的进程进行kill
```


<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
