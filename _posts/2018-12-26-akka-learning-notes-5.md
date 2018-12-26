---
title: Akka 学习笔记 5 纵向扩展
key: 20181226
tags: Akka 
---

话题：

- 多核计算的出现
- 使用 Future 进行多核变成
- 使用 Router 和 Actor 进行多核变成
- 使用 Dispatcher 隔离性能风险

<!--more-->

“纵向扩展”的意思是：当我们给单个系统添加额外的资源（CPU 或内存）时，应用程序能够利用新添加的资源。

作为一名现代开发者，要能够安全、高效地使用更多的核，而不是单个计算速度更快的核。

其实可以把如何进行多核架构看作是一个分布式问题：我们需要在另一个 CPU 或者是另一台机器上完成一些任务。通过 Akka 来使用 Actor 时，纵向扩展和横向扩展之间的区别开始显得不那么明显。哦我们可以忽略另一台主机和另一个内核之间的区别，只把这个问题看作是向某个 Actor 发送一条消息。我们希望将一些人物发送到另一个地方来执行并完成，然后在某一时刻接受对于发出请求的相应。可以将学习纵向扩展作为第一步，帮助我们理解如何最终进行横向扩展——如果可以在 8 个核心上使用 Actor 完成工作，那么要在 8 台主机上使用 Actor 完成工作也就八九不离十了。


要利用多个内核，最基本的机制就是并行：应用程序必须同时执行不同的操作。本质上来说，我们希望将工作分割成独立的子任务，然后使用不同的核同时运行这些子任务，这样就能利用所有可用的内核了。

Akka 中提供了两种可以用来多核并行编程的抽象： Future 和 Actor。

## 5.3 选择 Future 或 Actor 进行并发编程

既然 Actor 和 Future 这两个抽象都可以用于并发编程，那么究竟应该使用哪个呢？

有句老话说的好：当你有了一把锤子后，所有的东西看起来都像是一颗钉子。

实际上，要决定到底使用 Actor 还是 Future 其实并不简单。我曾听别人说过一个通用准则：“Future 用于并发， Actor 用于状态。“

换句话说，如果需要处理状态，那么可能马上就会想到用 Actor。如果不需要处理状态，只需要并发的话，那么可以试着用 Future。虽然这个准则还不错，但是它将问题过于简化了。在一些情况下，使用 Actor 会使得设计更易于调试和维护。所以我们一定要具体问题具体分析，衡量两种方法的优劣，考虑设计是否简单。

## 5.4 并行编程


我们先模拟一个非常耗时的操作，假设是一个从网页解析出文本内容的例子：

```java
public class ArticleParser {
	public static Try<String> apply(String html) {
		return Try.ofFailable(
			() -> de.13s.boilerpipe.extractors
				.ArticleExtractor.INSTANCE.getText(html)
		);
	}
}
```

Scala 的例子：

```scala
object ArticleParser {
	def apply(html: String) : String = 
	    de.l3s.boilerpipe.extractors.
	    	.ArticleExtractor.INSTANCE.getText(html)
}
```

### 5.4.1 使用 Future 进行并行编程


Future 的可组合行很高，非常适合用来并行编程。

下面的例子中，有一个叫做 `articleList` 的字符串列表，包含所有需要解析的文章所在的 HTML 页面。我们可以使用 Future 来处理这个列表，达到高并发高效利用系统资源：

Java 的例子：

```java

List<ComposableFuture<String>> futures = articleLists
	.stream()
	.map(article -> CompletableFuture.supplyAsync(() -> ArticleParser.apply(article)))
	.collect(Collectors.toList());
Future<List<String>> articleFuture = com.jasongoodwin.monads.Futures.sequence(futures).get();

```

我们使用 `better-java-monads` 库来处理多个 Future， 这个库中包含一个 `sequence` 的方法，可以将一个 Future 列表转化成包含结果列表的单个 Future。使用这个库需要加入以下以来到  build.sbt:


```sbt
	"com.jason-goodwin" % "better-monads" % "0.2.1"
```

Scala 的例子也类似，不过 Scala 原生提供了 `sequence` 方法，用于对 Future 列表进行转换。

```scala

import scala.concurrent.ExecutionContext.Implicits.global

val futures = articleList.map(article => {
	Future(ArticleParse.apply(article))
})
val articleFutures: Future[List[String]] = Future.sequence(futures)

```


### 5.4.2 使用 Actor 进行并行编程


首先创建一个 Actor，负责调用 Future 例子中的 ArticleParser 的静态 apply 方法。

java 的例子：

```java
public class ParseArticle {
	public final String htmlBody;
	public ParseArticle(String url) {
		this.htmlBody = url;
	}
}

public class ArticleParseActor extends AbstractActor {

	private ArticleParseActor() {
		receive(ReceiveBuilder.match(ParseArticle.class, x -> {
			sender().tell(ArticleParser.apply(x.htmlBody), self());
			})
			.build()
	    );
	}
}
```

Scala 的例子：

```scala

case class ParseArticle(htmlString: String)

class ArticleParseActor extends actor {
	override def receive: Receive = {
		case ParseArticle(htmlString) =>
			val body:String = ArticleParser(htmlString)
			sender9) ! body
	}
}

```

要并行完成任务，我们需要介绍 ”Router“的概念，用于及那个任务分发给不同的 Actor。

#### Router 介绍

在 Akka 中，Router 是一个用于负载均衡和路由的抽象。创建 Router 时，必须要传入一个 Actor Group，或者由 Router 来创建一个 Actor Pool。

注意 Group 和 Pool 这两个词的用法。为 Actor 创建 Router 时，一定要理解，有两种用来创建 Router 背后的 Actor 集合的机制：一种是由 Router来创建这些 Actor（一个 Pool）；另一种是把一个 Actor 列表（Group）传递给 Router。

在创建好了 Router 之后，当 Router 接收到消息时，就会将消息传递给 Group/Pool 中的一个或多个 Actor。有多种策略可以用来决定 Router 选择下一个消息发送对象的顺序。

在我们的例子中，所有的 Actor 都运行在本地，我们需要一个包含多个 Actor 的 Router 来支持使用多个 CPU 核心进行并行运算。如果 Actor 运行在远程机器上，也可以使用 Router 在服务器集群桑分发工作。

在我们使用本地 Actor 的例子中，可以选择 Actor Pool 的方式来创建 Router，由 Router 来创建我们所需的所有 Actor。在这种情况下使用 Router 非常简单：照常实例化一个 Actor，然后调用 `withRouter`，并传入一个路由策略，以及希望 Pool 中包含的 Actor 数量。Java 和 Scala 的代码逻辑相同：

Java 代码：

```java
ActorRef workerRouter = 
	system.actorOf(
		Props.create(ArticleParseActor.class)
			.withRouter(new RoundRobinPool(8)));
```

Scala 代码：

```scala
val workerRouter: ActorRef = 
	system.actorOf(
		Props.create(classOf[ArticleParseActor])
			.withRouter(new RoundRobinPool(8)))
```

也可以采用 Actor Group 的方式来创建 Router：传入一个包含 Actor 路径的列表：

```java
// java
ActorRef router = system.actorOf(new RoundRobinGroup(actors.map(actor -> actor.path()).props()));

// scala
val router = system.actorOf(new RoundRobinGroup(actors.map(actor => actor.path).props()))
```

现在就有了将工作分发给不同 CPU 内核所需的 Router 和 Actor。可以请求 Router 来处理列表中的每个消息，这样就可以并行执行操作了。

#### 路由逻辑

注意到我们使用了 RoundRobinPool/RoundRobinGroup，用于指定 Router 将消息发给各 Actor 的顺序。Akka 内置了一些路由策略，对于一般情况来说，RoundRobin 和 Random 都是不错的选择。

| 路由策略 | 功能 |
| ---- | ---- |
| Round Robin | 依次向 Pool/Group 中各个阶段发送消息，循环往复 |
| Random | 随机向各个节点发送消息 |
| Smallest Mailbox | 向当前包含消息数量最少的 Actor 发送消息。由于远程 Actor 的邮箱大小未知， 因此假设他么的队列中已经有消息在排队，所以会优先将消息发送给空闲的本地 Actor |
| Scatter Gather | 向 Group/Pool 中的所有 Actor 都发送消息，使用接收到的第一个响应，丢弃之后收到的任何其他响应。如果需要确保能够尽快收到一个响应，那么可以使用 scatter/gather 。 |
| Tail Chopping | 和 Scatter/Gather 类似，但是 Router 并不是一次性向 Group/Pool 中的所有 Actor 都发送一条消息，而是每向一个 Actor 发送消息后等待一小段时间。有着和 Scatter/Gather 类似的有点，但是相较而言有可能减少网络负载 |
| Consistent Hashing | 给 Router 提供一个 可以， Router 根据这个 key 生成哈希值。使用这个哈希值来决定给哪个节点发送数据。想要将特定的数据发送到特定的目标位置时，就可以使用哈希。 |
| Balancing Pool |BalancingPool 这个路由策略有点特殊。只可以用于本地 Actor。多个 Actor 共享同一个邮箱，一有空闲就处理邮箱中的任务。这种策略可以确保所有 Actor 都处于繁忙状态。对于本地集群来说，经常会优先选择这个路由策略 |


我们也可以实现自己的路由策略，不过大多数情况下并不需要这么做。


#### 向同一个 Router Group/Pool 中发的所有 Actor 发送消息

无论是使用 Group 还是 Pool 的形式来创建 Router，都可以通过广播，将一条消息发送给所有 Actor。例如：如果 Actor 都连接到一个远程数据库，运行中的额系统由于发生了错误需要修改使用的数据库，那么就可以通过一条广播消息更新 Pool/Group 中的所有 Actor：

```java
// java
router.tell(new akka.routing.Broadcast(msg));
// scala
router ! akka.routing.Broadcast(msg)
```


#### 监督 Router Pool 中的路由对象

如果使用 Pool 的方式创建 Router，由 Router 负责创建 Actor，那么这些路由对象会成为 Router 的子节点。创建 Router 时，可以给 Router 提供一个自定义的监督策略。

创建 Router 时，可以调用 `withSupervisorStrategy` 方法指定 Router 对 Pool 中路由对象的监督策略。

```java
// java
ActorRef workerRouter = system.actorOf(
	Props.create(ArticleParseActor.class)
		.withRouter(new RoundRobinPool(8)
			.withSupervisorStrategy(strategy))
	);
// scala
val workerRouter: ActorRef = 
	system.actorOf(
		Props.create(classOf[ArticleParseActor])
			.withrouter(new RoundRobinPool(8)
				.withSupervisorStrategy(strategy)))
```

由于使用 Group方式创建 Router 的时候传入了事先已经能够存在的 Actor，所以没有办法用 Router 来监督 Group 中的 Actor。

监督 Pool 中的 Actor 是给 Router 指定监督策略的最常见的一种 使用场景。除此之外，还有另一个场景会用到这种做法。如果有一个顶层的 Actor （使用 ActorSystem 的 actorOf 方法创建），那么这个 Actor 会将由守护 Actor 来监督。如果需要为这个 Actor 指定一个自定义的监督策略，那么一种方法就是创建另一个 Actor 来负责监督。除此之外，我们可以直接创建一个 Router，然后传递一个自定义的 SupervisorStrategy，由 Router 负责监督 Actor。由于这种方法不需要定义任何 Actor 行为，所以是最简单的为顶层 Actor 提供自定义监督策略的方法。

### 5.5.1 Dispatcher 解析


Dispatcher 将如何执行任务与何时执行任务两者解耦。一般来说，Dispatcher 会包含一些线程，这些线程会负责调度并运行任务，比如处理 Actor 的消息以及线程中的 Future 事件。Dispatcher 是 Akka 能够支持响应式编程的关键，是负责完成任务的机制。


所有的 Actor 或者 Future 的工作都是由 Executor/Dispatcher 分配的资源来完成的。

Dispatcher 负责将工作分配给 Actor。除此之外 Dispatcher 还可以分配资源用于处理 Future 的回掉函数。我们会发现 Future API 接受 Executor/ExecutionContext 作为参数。由于 Akka 的 Dispatcher 扩展了这些 API,因此 Dispatcher 具备双重功能。

在 Akka 中，dispatcher 实现了 `scala.concurrent.ExecutionContextExecutor` 接口，而这个接口又扩展了 `java.util.concurrent.Executor` 和 `scala.concurrent.ExecutionContext`。可以将 Executor 传递给 Java 的 Future，吧 ExecutionContext 传递给 Scala 的 Future。

用于 Future 时，可以通过 ActorSystem 中的一个引用来获取 Dispatcher (`ActorSystem.dispatcher`)。可以在 ActorSystem 中通过 ID 查询得到配置文件中定义的某个 Dispatcher：

```java
// actor system's dispatcher
system.dispatcher 
// custom dispatcher
system.dispatchers.lookup("my-dispatcher") 
```

由于我们能够创建并获取这些基于 Executor 的 Dispatcher，因此可以使用它们来定义 ThreadPool/ForkJoinPool 来隔离运行任务的环境。


### 5.5.2 Executor

Dispatcher 基于 Executor，所以在具体介绍 Dispatcher 之前，我们先介绍两种主要的 Executor 类型： ForkJoinPool 和 ThreadPool。

ThreadPool Executor 有一个工作队列，队列中包含了要分配给各线程的工作。线程空闲时就会从队列中认领工作。由于线程资源的创建和销毁开销很大，而 ThreadPool 允许线程的重用，所以就可以减少创建和销毁线程的次数，提高效率。

ForkJoinPool Executor 使用一种分治算法，递归地将任务分割成更小的子任务，然后把子任务分配给不同的线程运行。接着再把运行结果组合起来。由于提交的任务不一定都能够被递归地分割成 ForkJoinTask，所以 ForkJoinPool Executor 有一个工作窃取算法，允许空闲的线程“窃取”分配给另一个线程的工作。由于工作可能无法平均分配并完成，所以工作窃取算法能够更高效地利用硬件资源。


ForkJoinPool Executor 几乎总是比 ThreadPool 的 Executor 效率更高，是我们的默认选择。


### 5.5.3 创建 Dispatcher


要在 application.conf 中定义一个 Dispatcher，需要指定 Dispatcher 的类型和 Executor。还可以指定 Executor 的具体配置细节，比如使用线程的数量，或是每个 Actor 一次性处理的消息的数量。


```conf
my-dispatcher {
	type = Dispatcher
	executor = "fork-join-executor"

	fork-join-executor {
		parallelism-min = 2 # minimum threads
		parallelism-factor = 2.0 # Maximum threads per core
		parallelism-max = 10 # Maximum total threads
	}
	throughput = 100 # Max message to process in a actor before moving on.
}
```

有四种类型的 Dispatcher 可以用于描述如何在 Actor 之间共享线程：

- **Dispatcher**：默认的 Dispatcher 类型。将会使用定义的 Executor 在 Actor 中处理消息。在大多数情况下这种类型能够提供最好的性能。
- **PinnedDispatcher**：给每个 Actor 都分配其自己独有的线程。这种类型的 Dispatcher 为每个 Actor 都创建一个 ThreadPool Executor，每个 Executor 中都包含一个线程。如果希望确保每个 Actor 都能够立即响应，那么这似乎是个不错的方法。不过 PinnedDispatcher 比其他共享资源的方法效率更高的情况其实并不多。可以在单个 Actor 必须处理很多重要工作的时候试试这种类型的 Dispatcher，否则的话不推荐使用。
- **CallingThreadDispatcher**： 这个 Dispatcher 比较特殊，它没有 Executor，而是在发起调用的线程上执行工作。这种 Dispatcher 主要用于测试，特备是调试。由于发起调用的线程负责返程工作，可以清楚地看到栈追踪轨迹信息，了解所执行方法的完整上下文。这对于理解异常是非常有用的。每个 Actor 会获取一个锁，所以每次只有一个线程可以在 Actor 中执行代码，而如果doge线程向一个 Actor 发送信息的话，就会导致除了拥有锁的线程之外的所有线程处于等待状态。 TestActorRef 就是介于 CallingThreadDispatcher 实现在测试中同步执行工作的。
- **BalancingDispatcher**：哦我们会在一些 Akka 文档中看到 BalancingDispatcher。现在已经不推荐直接使用了。应该使用之前介绍过的 BalancingPool Router。不过 Akka 中任然使用了 BalancingDispatcher，但是只会通过 Router 简介受用。 BalancingDispatcher 有一点很特殊：Pool 中的所有 Actor 都共享同一个邮箱，并且会为 Pool 中的每个 Actor 都创建一个线程。使用 BalancingDispatcher 的 Actor 从邮箱中拉去信息，所以只要有 Actor 处于空闲状态，就不会有任何 Actor 的工作队列中存在任务。这是工作窃取的一个变种，所有 Actor 都会从一个共享的邮箱中拉取任务。两者在性能上的优点也类似。


创建 Actor 的时候，可以给 `Props` 提供在 application.conf 中配置好的 Dispatcher 名称：

```scala
system.actorOf(Props[MyActor].withDispatcher("my-pinned-dispatcher"))
``` 

### 5.5.4 决定合适使用哪种 Dispatcher


进行纵向扩展的第一步是理解哪些情况的响应及时性最重要，以及对这些重要的请求做出响应时可能会发生资源竞争的地方。

只使用一个资源池来随意分配工作可能会导致应用程序的某些很耗费资源的操作占尽了所有资源，而最重要的基本使用场景却无法得到资源。

要甘山这种情况，可以把用于遵行高风险任务的资源和运行重要任务的资源隔离开来。如果我们新建一些 Dispatcher，把运行时间比较长或者是会阻塞线程的任务都分配给这些 Dispatcher，就可以确保应用程序的剩余部分仍然能够保持响应的即时性。我们希望能够把所有需要大量计算、运行时间较长的任务分离到单独的 Dispatcher 中，确保在糟糕的情况下仍然能够有资源区运行其他人物。

要采取这种方法，就必须首先分析应用程序的性能，理解应用程序在什么地方可能会阻塞线程，耗尽系统资源。我们需要对应用程序执行的任务进行分类。

阻塞 IO 会有它自己的 Dispatcher，包含 50 或 100 个线程，会阻塞线程、等待 IO 的操作要和异步线程池分隔开来，原因在于一定拿所有线程都处于等待 IO 的状态，那么应用程序中的其他操作都无法继续执行。者可能是最重要的一点：**千万不要把阻塞 IO 操作放在 akka 的 Dispatcher 中**。





- 


If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
