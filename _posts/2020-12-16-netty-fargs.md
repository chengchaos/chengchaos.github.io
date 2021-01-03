---
title: Netty 实践
key: 2020-12-26
tags: Netty
---

原文 Netty中的坑(下篇)

<!--more-->

其实这篇应该叫Netty实践，但是为了与前一篇名字保持一致，所以还是用一下坑这个名字吧。

Netty是高性能Java NIO网络框架，在很多开源系统里都有她的身影，而在绝大多数互联网公司所实施的服务化，以及最近流行的MicroService中，她都作为基础中的基础出现。

Netty的出现让我们可以简单容易地就可以使用NIO带来的高性能网络编程的潜力。她用一种统一的流水线方式组织我们的业务代码，将底层网络繁杂的细节隐藏起来，让我们只需要关注业务代码即可。并且用这种机制将不同的业务划分到不同的handler里，比如将编码，连接管理，业务逻辑处理进行分开。Netty也力所能及的屏蔽了一些NIO bug，比如著名的epoll cpu 100% bug。而且，还提供了很多优化支持，比如使用buffer来提高网络吞吐量。

但是，和所有的框架一样，框架为我们屏蔽了底层细节，让我们可以很快上手。但是，并不表示我们不需要对框架所屏蔽的那一层进行了解。本文所涉及的几个地方就是Netty与底层网络结合的几个地方，看看我们使用的时候应该怎么处理，以及为什么要这么处理。

## autoread

在 Netty 4 里我觉得一个很有用的功能是 autoread。autoread 是一个开关，如果打开的时候 Netty 就会帮我们注册读事件(这个需要对NIO有些基本的了解)。当注册了读事件后，如果网络可读，则Netty就会从channel读取数据，然后我们的pipeline就会开始流动起来。那如果autoread关掉后，则Netty会不注册读事件，这样即使是对端发送数据过来了也不会触发读时间，从而也不会从channel读取到数据。那么这样一个功能到底有什么作用呢？

它的作用就是更精确的速率控制。那么这句话是什么意思呢？比如我们现在在使用Netty开发一个应用，这个应用从网络上发送过来的数据量非常大，大到有时我们都有点处理不过来了。而我们使用Netty开发应用往往是这样的安排方式：Netty的Worker线程处理网络事件，比如读取和写入，然后将读取后的数据交给pipeline处理，比如经过反序列化等最后到业务层。到业务层的时候如果业务层有阻塞操作，比如数据库IO等，可能还要将收到的数据交给另外一个线程池处理。因为我们绝对不能阻塞Worker线程，一旦阻塞就会影响网络处理效率，因为这些Worker是所有网络处理共享的，如果这里阻塞了，可能影响很多channel的网络处理。

但是，如果把接到的数据交给另外一个线程池处理就又涉及另外一个问题：速率匹配。

比如现在网络实在太忙了，接收到很多数据交给线程池。然后就出现两种情况：

1/ 由于开发的时候没有考虑到，这个线程池使用了某些无界资源。比如很多人对 ThreadPoolExecutor 的几个参数不是特别熟悉，就有可能用错，最后导致资源无节制使用，整个系统crash掉。
复制代码

```java
//比如开始的时候没有考虑到会有这么大量
//这种方式线程数是无界的，那么有可能创建大量的线程对系统稳定性造成影响
Executor executor = Executors.newCachedTheadPool();
executor.execute(requestWorker);

//或者使用这个
//这种queue是无界的，有可能会消耗太多内存，对系统稳定性造成影响
Executor executor = Executors.newFixedThreadPool(8);
executor.execute(requestWorker);
```

2/ 第二种情况就是限制了资源使用，所以只好把最老的或最新的数据丢弃。

```java
//线程池满后，将最老的数据丢弃
Executor executor = new ThreadPoolExecutor(8, 
        8, 
        1, 
        TimeUnit.MINUTES, 
        new ArrayBlockingQueue<Runnable>(1000), 
        namedFactory, 
        new ThreadPoolExecutor.DiscardOldestPolicy());
```

其实上面两种情况，不管哪一种都不是太合理。不过在 Netty 4 里我们就有了更好的解决办法了。如果我们的线程池暂时处理不过来，那么我们可以将 `autoread` 关闭，这样 Netty 就不再从 channel 上读取数据了。那么这样造成的影响是什么呢？这样 socket 在内核那一层的 read buffer 就会满了。因为 TCP 默认就是带 flow control 的，read buffer 变小之后，向对端发送 ACK 的时候，就会降低窗口大小，直至变成 0，这样对端就会自动的降低发送数据的速率了。等到我们又可以处理数据了，我们就可以将 `autoread` 又打开这样数据又源源不断的到来了。

这样整个系统就通过 TCP 的这个负反馈机制，和谐的运行着。那么 `autoread` 涉及的网络知识就是，发送端会根据对端ACK时候所携带的 advertises window 来调整自己发送的数据量。而 ACK 里的这 个window 的大小又跟接收端的 read buffer 有关系。而不注册读事件后，read buffer 里的数据没有被消费掉，就会达到控制发送端速度的目的。

不过设计关闭和打开 `autoread` 的策略也要注意，不要设计成我们不能处理任何数据了就立即关闭 `autoread`，而我们开始能处理了就立即打开 `autoread` 。这个地方应该留一个缓冲地带。也就是如果现在排队的数据达到我们预设置的一个高水位线的时候我们关闭autoread，而低于一个低水位线的时候才打开 autoread。不这么弄的话，有可能就会导致我们的 autoread 频繁打开和关闭。autoread 的每次调整都会涉及系统调用，对性能是有影响的。类似下面这样一个代码，在将任务提交到线程池之前，判断一下现在的排队量(注：本文的所有数字纯为演示作用，所有线程池，队列等大小数据要根据实际业务场景仔细设计和考量)。
复制代码

```java
int highReadWaterMarker = 900;
int lowReadWaterMarker = 600;

ThreadPoolExecutor executor = new ThreadPoolExecutor(8, 8, 1, TimeUnit.MINUTES, new ArrayBlockingQueue<Runnable>(1000), namedFactory, new ThreadPoolExecutor.DiscardOldestPolicy());

int queued = executor.getQueue().size();
if(queued > highReadWaterMarker){
    channel.config().setAutoRead(false);
}
if(queued < lowReadWaterMarker){
    channel.config().setAutoRead(true);
}
```

但是使用autoread也要注意一件事情。autoread如果关闭后，对端发送FIN的时候，接收端应用层也是感知不到的。这样带来一个后果就是对端发送了FIN，然后内核将这个socket的状态变成CLOSE_WAIT。但是因为应用层感知不到，所以应用层一直没有调用close。这样的socket就会长期处于CLOSE_WAIT状态。特别是一些使用连接池的应用，如果将连接归还给连接池后，一定要记着autoread一定是打开的。不然就会有大量的连接处于CLOSE_WAIT状态。

其实所有异步的场合都存在速率匹配的问题，而同步往往不存在这样的问题，因为同步本身就是带负反馈的。

## isWritable

isWritable其实在上一篇文章已经介绍了一点，不过这里我想结合网络层再啰嗦一下。上面我们讲的autoread一般是接收端的事情，而发送端也有速率控制的问题。Netty为了提高网络的吞吐量，在业务层与socket之间又增加了一个ChannelOutboundBuffer。在我们调用channel.write的时候，所有写出的数据其实并没有写到socket，而是先写到ChannelOutboundBuffer。当调用channel.flush的时候才真正的向socket写出。因为这中间有一个buffer，就存在速率匹配了，而且这个buffer还是无界的。也就是你如果没有控制channel.write的速度，会有大量的数据在这个buffer里堆积，而且如果碰到socket又『写不出』数据的时候，很有可能的结果就是资源耗尽。而且这里让这个事情更严重的是ChannelOutboundBuffer很多时候我们放到里面的是DirectByteBuffer，什么意思呢，意思是这些内存是放在GC Heap之外。如果我们仅仅是监控GC的话还监控不出来这个隐患。

那么说到这里，socket什么时候会写不出数据呢？在上一节我们了解到接收端有一个read buffer，其实发送端也有一个send buffer。我们调用socket的write的时候其实是向这个send buffer写数据，如果写进去了就表示成功了(所以这里千万不能将socket.write调用成功理解成数据已经到达接收端了)，如果send buffer满了，对于同步socket来讲，write就会阻塞直到超时或者send buffer又有空间(这么一看，其实我们可以将同步的socket.write理解为半同步嘛)。对于异步来讲这里是立即返回的。

那么进入send buffer的数据什么时候会减少呢？是发送到网络的数据就会从send buffer里去掉么？也不是这个样子的。还记得TCP有重传机制么，如果发送到网络的数据都从send buffer删除了，那么这个数据没有得到确认TCP怎么重传呢？所以send buffer的数据是等到接收端回复ACK确认后才删除。那么，如果接收端非常慢，比如CPU占用已经到100%了，而load也非常高的时候，很有可能来不及处理网络事件，这个时候send buffer就有可能会堆满。这就导致socket写不出数据了。而发送端的应用层在发送数据的时候往往判断socket是不是有效的(是否已经断开)，而忽略了是否可写，这个时候有可能就还一个劲的写数据，最后导致ChannelOutboundBuffer膨胀，造成系统不稳定。

所以，Netty已经为我们考虑了这点。channel有一个isWritable属性，可以来控制ChannelOutboundBuffer，不让其无限制膨胀。至于isWritable的实现机制可以参考前一篇。
序列化

所有讲TCP的书都会有这么一个介绍：TCP provides a connection-oriented, reliable, byte stream service。前面两个这里就不关心了，那么这个byte stream到底是什么意思呢？我们在发送端发送数据的时候，对于应用层来说我们发送的是一个个对象，然后序列化成一个个字节数组，但无论怎样，我们发送的是一个个『包』。每个都是独立的。那么接收端是不是也像发送端一样，接收到一个个独立的『包』呢？很遗憾，不是的。这就是byte stream的意思。接收端没有『包』的概念了。

这对于应用层编码的人员来说可能有点困惑。比如我使用Netty开发，我的handler的channelRead这次明明传递给我的是一个ByteBuf啊，是一个『独立』的包啊，如果是byte stream的话难道不应该传递我一个Stream么。但是这个ByteBuf和发送端的ByteBuf一点关系都没有。比如：

```java
public class Decorder extends ChannelInboundHandlerAdapter{
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //这里的msg和发送端channel.write(msg)时候的msg没有任何关系
    } 
}    
```

这个ByteBuf可能包含发送端多个ByteBuf，也可能只包含发送端半个ByteBuf。但是别担心，TCP的可靠性会确保接收端的顺序和发送端的顺序是一致的。这样的byte stream协议对我们的反序列化工作就带来了一些挑战。在反序列化的时候我们要时刻记着这一点。对于半个ByteBuf我们按照设计的协议如果解不出一个完整对象，我们要留着，和下次收到的ByteBuf拼凑在一起再次解析，而收到的多个ByteBuf我们要根据协议解析出多个完整对象，而很有可能最后一个也是不完整的。不过幸运的是，我们有了Netty。Netty为我们已经提供了很多种协议解析的方式，并且对于这种半包粘包也已经有考虑，我们可以参考ByteToMessageDecoder以及它的一连串子类来实现自己的反序列化机制。而在反序列化的时候我们可能经常要取ByteBuf中的一个片段，这个时候建议使用ByteBuf的readSlice方法而不是使用copy。

另外，Netty还提供了两个ByteBuf的流封装：ByteBufInputStream, ByteBufOutputStream。比如我们在使用一些序列化工具，比如Hessian之类的时候，我们往往需要传递一个InputStream(反序列化)，OutputStream(序列化)到这些工具。而很多协议的实现都涉及大量的内存copy。比如对于反序列化，先将ByteBuf里的数据读取到byte[]，然后包装成ByteArrayInputStream，而序列化的时候是先将对象序列化成ByteArrayOutputStream再copy到ByteBuf。而使用ByteBufInputStream和ByteBufOutputStream就不再有这样的内存拷贝了，大大节约了内存开销。

另外，因为socket.write和socket.read都需要一个direct byte buffer(即使你传入的是一个heap byte buffer，socket内部也会将内容copy到direct byte buffer)。如果我们直接使用ByteBufInputStream和ByteBufOutputStream封装的direct byte buffer再加上Netty 4的内存池，那么内存将更有效的使用。这里提一个问题：为什么socket.read和socket.write都需要direct byte buffer呢？heap byte buffer不行么？

总结起来，对于序列化和反序列化来讲就是两条：1 减少内存拷贝 2 处理好TCP的粘包和半包问题

## 后记

作为一个应用层程序员，往往是幸福的。因为我们有丰富的框架和工具为我们屏蔽下层的细节，这样我们可以更容易的解决很多业务问题。但是目前程序设计并没有发展到不需要了解所有下层的知识就可以写出更有效率的程序，所以我们在使用一个框架的时候最好要对它所屏蔽和所依赖的知识进行一些了解，这样在碰到一些问题的时候我们可以根据这些理论知识去分析原因。这就是理论和实践的相结合。

转自：

- [https://www.cnblogs.com/yuyijq/p/4431798.html](https://www.cnblogs.com/yuyijq/p/4431798.html)

EOF

---

Power by TeXt.
