---
title: Netty 学习笔记 002 (Bootstrap)
key: 2019-06-18
tags: netty bootstrap
---



Bootstrap 是 Netty 提供的客户端连接工具类，主要用于简化客户端的创建。



<!--more-->

## 设置 EventLoopGroup 线程池

通常多个连接会共享一个 `EventLoopGrup`，默认 `EventLoopGroup` 的大小是 CPU 内核数的 2 倍，也可以根据业务实际情况进行调整，如果性能和连接数都要求不高，建议设置为 1 。



## TCP 参数设置接口

`Bootstrap` 的客户端 TCP 参数设置接口：（`AbstractBootstrap`）



```java
public <T> option(ChannelOption<T> option, T value) {
    if (option == null) {
        throw new NullPointerException("option");
    }
    if (value == null) {
        synchronized (options) {
            options.remove(option);
        }
    } else {
        synchronized (options) {
            options.put(option, value);
        }
    }
    return self();
}
```



Netty 提供的主要 TCP 参数如下:



- `SO_TIMEOUT` : 控制读取操作将阻塞多少毫秒. 如果返回值为 0, 计时器就被禁止, 该线程将无限期阻塞.
- `SO_SNDBUF` :  套接字使用的发送缓冲区大小.
- `SO_REUSEADDR` : 决定当网络上仍然有数据向旧的 ServerSocket 传输数据时,  是否允许新的 ServerSocket 绑定到与旧的 ServerSocket 同样的端口上. 这个选项的默认值与操作系统有关.
- `CONNECT_TIMEOUT_MILLIS` : 客户端连接超时时间, 由于 NIO 原生的客户端并不提供设置连接超时的接口, 因此 Netty 采用自定义连接超时定时器负责监测是否超时和进行超时控制.
- `TCP_NODELAY` :  激活或者禁止 TCP_NODELAY  套接字选项, 它决定是否使用 Nagle 算法. 如果是时延敏感的应用, 建议关闭 Nagle 算法.

## Channel 接口

用于指定客户端使用的 Channel 接口, 对于 TCP 客户端连接,默认使用 `NioSocketChannel`. `BootstrapChannelFactory` 利用  `channelClass` 类型信息, 通过反射机制创建 `NioSocketChannel` 对象.



Bootstrap 为了简化 `ChannelHandler` 的编排, 提供了 `ChannelInitializer`, 他继承了 `ChannelHandlerAdapter`, TCP 链路注册成功后, 调用 `initChannel` 接口, 用于设置用户 `ChannelHandler` , 代码(`ChannelInitializer`)如下:



```java
private boolean initChannel(ChannelHandlerContext ctx) throws Exception {
    if (initMap.putIfAbsent(ctx, Boolean.TRUE) == null) {
        try {
            initChannel((C)ctx.channel());
        } catch (Throwable cause) {
            exceptionCaught(ctx, cause);
        } finally {
            remove(ctx);
        }
        return true;
    } 
    return false;
}
```



它的功能主要有两个:



1. 执行客户端初始化时应用实现的 `initChannel(SocketChannel ch)` 将应用自定义的 `ChannelHandler` 添加到 `ChannelPipeline` 中.
2. 将 Bootstrap 注册到 `ChannelPipeline` 用于初始化应用 `ChannelHandler` 的 `ChannelInitializer` 删除掉, 完成用户 `ChannelPipeline` 的初始化工作.

## 连接



最后一个重要接口就是调用 `connect` 方法连接服务端,  应采用异步的方式调用, 即获取 `ChannelFuture` 后组测监听器, 异步处理连接操作结果, 不要阻塞调用方的线程.



参考: 《Netty进阶之路 跟着案例学 Netty》(李林峰)















<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
