---
title: Linux 网络
key: 2021-09-20
tags: linux network
---

## 常见的网络相关参数

<!--more-->

### net.ipv4.tcp_max_tw_buckets

对于 TCP 连接, 服务端和客户端通信完成后状态变为 TIMEW_WAIT, 加入某台服务器非常忙, 连接数特别多的话, 那么这个 TIME_WAIT 数量就会越来越多, 所以有一个最大值, 当超过这个最大值, 系统就会删除最早的连接.

可以使用 `sysctl -a | grep tw_buckets` 查看它的值, CentOS7 默认是 32768. 可以适当调低, 但是不要调到几十几百这样的数值, 因为这种状态的 TCP 连接也是有用的, 如果通用的客户端再次和服务端通信, 就不用再次建立新的连接了.

如果遇到大量的 TIME_WAIT 导致端口耗尽服务异常时，可以试试修改这个参数

### net.ipv4.tcp_tw_recycle = 1

启用 timewait 快速回收.

这个参数的作用是快速回收 timewait 状态的连接, 上面提到虽然系统会自动删除调 timewait 状态的连接, 但是如果吧这样的连接重新利用不是更好吗? 所以此参数设置为 1 就可以让 timewait 状态的连接快速回收, 它需要和下面的参数一起配合使用.

### net.ipv4.tcp_tw_reuse = 1

启用 timewait 重用.

将此参数设置为 1, 将 timewait 状态的连接重新用于新的 TCP 连接, 要结合上面的参数一起使用.

### net.ipv4.tcp_syncookies = 1

TCP 三次握手中，客户端向服务端发起 syn 请求，服务端收到后，也会向客户端发起 syn 请求同时连带ack 确认，假如客户端发送请求后直接断开和服务端的连接，不接收服务端发起的这个请求，服务端会重试多次。

这个重试的过程会持续一段时间，当这种状态的连接数量非常大时，服务器会消耗很大的资源，从而造成瘫痪，正常的连接进不来，这种恶意的半连接行为其实叫做 syn flood 攻击。

此参数设置为 1，是开启 SYN Cookies，开启后可以避免发生上述的 syn flood 攻击。开启该参数后，服务端接收客户端的 ack 后，再向客户端发送 ack+syn 之前会要求 client 在短时间内回应一个序号，如果客户端不能提供序号或者提供的序号不对则认为该客户端不合法，于是不会发 ack+syn 给客户端，更涉及不到重试。

### net.ipv4.tcp_max_syn_backlog

该参数定义系统能接受的最大半连接状态的 TCP 连接数。客户端向服务端发送了 syn 包，服务端收到后，会记录一下，该参数决定最多能记录几个这样的连接。我的 CentOS7 系统，默认是 256，当有 syn flood 攻击时，这个数值太小则很容易导致服务器瘫痪，实际上此时服务器并没有消耗太多资源（cpu、内存等），所以可以适当调大它。

### net.ipv4.tcp_syn_retries

该参数适用于客户端，它定义发起 syn 的最大重试次数，默认为 5.

### net.ipv4.tcp_synack_retries

该参数适用于服务端，它定义发起 syn+ack 的最大重试次数，默认为 52，可以适当预防 syn flood 攻击。

### net.ipv4.ip_local_port_range

该参数定义端口范围，系统默认保留端口为 1024 及以下，以上部分为自定义端口。这个参数适用于客户端，当客户端和服务端建立连接时，比如说访问服务端的 80 端口，客户端随机开启了一个端口和服务端发起连接，这个参数定义随机端口的范围。默认为 `32768 61000`。

### net.ipv4.tcp_fin_timeout

tcp 连接的状态中，客户端上有一个是 FIN-WAIT-2 状态，它是状态变迁为 timewait 前一个状态。该参数定义不属于任何进程的该连接状态的超时时间，默认值为 60。

### net.ipv4.tcp_keepalive_time

tcp 连接状态里，有一个是 keepalived 状态，只有在这个状态下，客户端和服务端才能通信。正常情况下，当通信完毕，客户端或服务端会告诉对方要关闭连接，此时状态就会变为 timewait，如果客户端没有告诉服务端，并且服务端也没有告诉客户端关闭的话（例如，客户端那边断网了），此时需要该参数来判定。

比如客户端已经断网了，但服务端上本次连接的状态依然是 keepalived，服务端为了确认客户端是否断网，就需要每隔一段时间去发一个探测包去确认一下看看对方是否在线。这个时间就由该参数决定。它的默认值为 7200（单位为秒）。

### net.ipv4.tcp_keepalive_intvl

该参数和上面的参数是一起的，服务端在规定时间内发起了探测，查看客户端是否在线，如果客户端并没有确认，此时服务端还不能认定为对方不在线，而是要尝试多次。该参数定义重新发送探测的时间，即第一次发现对方有问题后，过多久再次发起探测。

默认值为 75 （单位为秒）。

### net.ipv4.tcp_keepalive_probes

tcp_keepalive_time 和 tcp_keepalive_probes 参数规定了何时发起探测和探测失败后再过多久再发起探测，但并没有定义一共探测几次才算结束。该参数定义发起探测的包的数量。默认为 9，建议设置 2。

### net.core.somaxconn

定义了系统中每一个端口最大的监听队列的长度,这是个全局的参数,默认值为 128.限制了每个端口接收新 tcp 连接侦听队列的大小。对于一个经常处理新连接的高负载 web 服务环境来说，默认的 128 太小了。大多数环境这个值建议增加到 1024 或者更多。 服务进程会自己限制侦听队列的大小(例如 sendmail(8) 或者 Apache)，常常在它们的配置文件中有设置队列大小的选项。大的侦听队列对防止拒绝服务 DoS 攻击也会有所帮助。

### net.core. 

每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目，默认为1000

### net.ipv4.tcp_max_orphans

最大孤儿套接字(orphan sockets)数，单位是个，当 orphans 达到设定值一半的时候，会报
Out of socket memory，假设每个孤儿套接字占用 64kb 内存当 orphans 为 65536 时候占用内存大约 2GB，

### net.ipv4.route.gc_timeout

路由缓存刷新频率, 当一个路由失败后多长时间跳到另一个, 默认是 300 .

EOF