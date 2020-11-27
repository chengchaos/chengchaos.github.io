---
title: ChannelOption 用到的 socket 的标准参数
key: 2020-11-27
tags: java netty
---

如下

<!--more-->

- ChannelOption.SO_BACKLOG, 1024

    BACKLOG用于构造服务端套接字 ServerSocket 对象，标识当服务器请求处理线程全满时，用于临时存放已完成三次握手的请求的队列的最大长度。如果未设置或所设置的值小于 1，Java 将使用默认值 50。

- ChannelOption.SO_KEEPALIVE, true

    是否启用心跳保活机制。在双方 TCP 套接字建立连接后（即都进入 ESTABLISHED 状态）并且在两个小时左右上层没有任何数据传输的情况下，这套机制才会被激活。

- ChannelOption.TCP_NODELAY, true

    在 TCP/IP 协议中，无论发送多少数据，总是要在数据前面加上协议头，同时，对方接收到数据，也需要发送 ACK 表示确认。为了尽可能的利用网络带宽，TCP 总是希望尽可能的发送足够大的数据。这里就涉及到一个名为 Nagle 的算法，该算法的目的就是为了尽可能发送大块数据，避免网络中充斥着许多小数据块。

    TCP_NODELAY 就是用于启用或关于 Nagle 算法。如果要求高实时性，有数据发送时就马上发送，就将该选项设置为 true 关闭 Nagle 算法；如果要减少发送次数减少网络交互，就设置为 false 等累积一定大小后再发送。默认为 false。

= ChannelOption.SO_REUSEADDR, true

    SO_REUSEADDR 允许启动一个监听服务器并捆绑其众所周知端口，即使以前建立的将此端口用做他们的本地端口的连接仍存在。这通常是重启监听服务器时出现，若不设置此选项，则bind时将出错。

    SO_REUSEADDR 允许在同一端口上启动同一服务器的多个实例，只要每个实例捆绑一个不同的本地IP地址即可。对于 TCP，我们根本不可能启动捆绑相同IP地址和相同端口号的多个服务器。

    SO_REUSEADDR 允许单个进程捆绑同一端口到多个套接口上，只要每个捆绑指定不同的本地IP地址即可。这一般不用于 TCP 服务器。

    SO_REUSEADDR 允许完全重复的捆绑：当一个 IP 地址和端口绑定到某个套接口上时，还允许此IP地址和端口捆绑到另一个套接口上。一般来说，这个特性仅在支持多播的系统上才有，而且只对 UDP 套接口而言（TCP不支持多播）

- ChannelOption.SO_RCVBUF  AND  ChannelOption.SO_SNDBUF

    定义接收或者传输的系统缓冲区buf的大小，

转自：

- [ChannelOption用到的socket的标准参数](https://www.cnblogs.com/xiaoyongsz/p/6133266.html)

EOF

---

Power by TeXt.
