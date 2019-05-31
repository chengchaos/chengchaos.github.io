---
title: Future 和 Promises
key: 2019-05-31
tags: Scala Future Promises
---
原文： [https://docs.scala-lang.org/overviews/core/futures.html](https://docs.scala-lang.org/overviews/core/futures.html)
作者： Philipp Haller, Aleksandar Prokopec, Heather Miller, Viktor Klang, Roland Kuhn, and Vojin Jovanovic

<!--more-->

## Introduction

Future 提供了一种有效的、非阻塞的方式，并行执行许多操作。`Future` 对象是一个值对象的占位符代表可能还没有计算出来的值。通常情况下 Future 的值是由并发的程序提供的，并随后（subsequently）被使用。以这种方式组合并发任务往往会导致更快、异步、非阻塞的并行代码。

默认情况下，Future 和 Promise 是非阻塞的，使用回调而不是典型的阻塞操作。
为了在语法 (syntactically) 和概念 (conceptually)上简化回调的使用，Scala提供了组合符，如 `flatMap`、`foreach` 和 `filter`，用于以非阻塞方式组合 future。阻塞仍然是可能的——对于绝对必要的情况，可以阻塞 future (尽管不建议这样做)。

一个典型的 future 看起来是酱紫的:

```scala
val inverseFuture: Future[Matrix] = Future {
  fatMatrix.inverse() // 非阻塞的长久的计算
}(executionContext)
```

或者更傻瓜化一点的:

```scala
implicit val ec: ExecutionContext = ...

val inverseFuture: Future[Matrix] = Future {
  fatMatrix.inverse()
} // ec 是隐式提供的
```

这两个代码片段都将 `fatMatrix.reverse()` 的执行委托给 ExecutionContext，并在 `inverseFuture` 中体现计算结果。


## Execution Context

Future 和 promise 以一个或多个 (revolve around) `ExecutionContext` 为中心，负责执行计算。

`ExecutionContext` 类似于 `Executor`: 可以在新线程、池化线程或当前线程中自由地执行计算(尽管不建议在当前线程中执行计算—更多内容见下文)。


`cala.concurrent` 包中提供了一个 `ExecutionContext` 实现，一个全局静态线程池。还可以将 `Executor` 转换为 `ExecutionContext`。最后，用户可以自由地扩展 `ExecutionContext` trait 来实现他们自己的执行上下文，尽管这只在极少数情况下才应该这样做( in rare cases)。

### The Global Execution Context

`ExecutionContext.global` 是一个 `ForkJoinPool` 支持的 ExecutionContext。
它应该足以应付大多数情况，但需要注意: `ForkJoinPool` 管理有限的线程数量(称为*并行级别*的最大线程数量)。
只有每个阻塞调用都封装在一个 `blocking` 调用中时(下面将详细介绍)，并发阻塞计算的数量才可以超过并行性级别。
否则，全局执行上下文中的线程池可能会耗尽(starved)，无法进行计算。

默认情况下是 `ExecutionContext.global` 将其底层 fork-join 池的并行性级别设置为可用处理器的数量(`Runtime.availableProcessors`)。可以通过设置以下VM属性中的一个(或多个)覆盖此配置:

- scala.concurrent.context.minThreads - 默认值是 `Runtime.availableProcessors`
- scala.concurrent.context.numThreads - 可以是“xN”形式的数字或乘数(N); 默认值是 `Runtime.availableProcessors`
- scala.concurrent.context.maxThreads - 默认值是 `Runtime.availableProcessors`

The parallelism level will be set to `numThreads` as long as it remains within `[minThreads; maxThreads]`.

如上所述，在存在阻塞计算的情况下，`ForkJoinPool` 可以将线程数量增加到其 `parallelismLevel` 之外。
正如在 `ForkJoinPool` API中所解释的，只有在显式通知池时才有可能:

```scala
import scala.concurrent.Future
import scala.concurrent.forkjoin._

// the following is equivalent to `implicit val ec = ExecutionContext.global`
import ExecutionContext.Implicits.global

Future {
  ForkJoinPool.managedBlock(
    new ManagedBlocker {
      var done = false
      def block() : Boolean = {
        try {
          myLock.lock()
          // ...
        } finally {
          done = true
        }
        true
      }
      def isReleaseable: Boolean = done
    }
  )
}
```
幸运的是(fortunately)，并发包提供了一种方便(convenient )的方法:

```scala
import scala.concurrent.Future
import scala.concurrent.blocking

Future {
  blocking {
    myLock.lock()
    // ...
  }
}
```
> 注意，`blocking` 是一个通用的 construct ，后面将更深入地讨论它。

Last but not least，必须记住，`ForkJoinPool` 不是为长久的阻塞操作而设计的。
即使在被通知 `blocking` 时，可能也不会像您所期望的那样生成新工作程序，而在创建新工作程序时，它们可能多达32767个。为了让您有个概念，下面的代码将使用32000个线程:


```scala

implicit val ec = ExecutionContext.global

for (i <- 1 to 32000) {
  Future {
    blocking {
      Thread.sleep(999999)
    }
  }
}

```

如果需要包装持久的阻塞操作，我们建议使用专用的(dedicated ) `ExecutionContext`，例如包装一个 Java 的 `Executor`。


### Adapting a Java Executor

使用 `ExecutionContext.fromExecutor` 方法可以包装一个 Java Executor 到一个 ExecutionContext 中.

例如:


```scala
ExecutionContext.fromExecutor(new ThreadPoolExecutor( /* 你自己配置就好 */ ))
```



### Synchronous Execution Context

你可能会想(One might be tempted to)要一个 `ExecutionContext` 在当前线程中运行计算:

```scala
val currentThreadExecutionContext = ExecutionContext.fromExecutor(
  new Executor {
    // Do not do this!
    // 最好别这么干
    def execute(runnable: Runnable) { runnable.run() }
})

```

This should be avoided as it introduces non-determinism in the execution of your future.

```scala
Future {
  doSomething
}(ExecutionContext.global).map {
  doSomethingElse
}(currentThreadExecutionContext)
```

`doSomethingElse` 调用可以在 `doSomething` 的线程中执行，也可以在主线程中执行，因此可以是异步的，也可以是同步的。
正如[这里](http://blog.ometer.com/2011/07/24/callbacks-synchronous-and-asynchronous/)所解释的，回调不应该两者都是。



## Future

`Future` 是一个持有某个值的对象，该值可能在某个时候可用。这个值通常是其他一些计算的结果:

1. 如果计算还没有完成，我们说 `Future` 是 **not completed** 的。
2. 如果计算已完成, 结果可能是一个值或一个异常，我们说 `Future` 是 **completed**的。

Completion can take one of two forms (完成可以是两种形式之一):

1. 当 `Future` 完成并得到一个值时，我们说 future 是 **successfully completed** 的
2. 当 `Future` 完成并抛出了一个异常，我们说 future 是 **failed** 的

`Future` 有一个重要的属性，它只能被分配(assigned )一次。一旦一个 `Future` 对象被赋予一个值或一个异常，它实际上就成为不可变的——它永远不会被覆盖。

创建 future 对象的最简单方法是调用 `Future.apply` 方法，该方法启动异步计算并返回保存该计算结果的 future。一旦 future 完成结果就可用了。

> 注意 `Future[T]` 是表示(denotes )将来对象的类型，而(whereas) `Future.apply` 是一个方法，它创建并调度一个异步计算，然后返回一个将来的对象，该对象将与该计算的结果一起完成。

This is best shown through an example.


让我们假设(assume )有一个流行的社交网络(social network), 我们想要使用它的(假定的)API来为指定的用户获取好友列表。我们将打开一个新的会话，然后发送一个请求，以获取特定用户的好友列表:

```scala

import scala.concurrent._
import ExecutionContext.Implicits.global

val session = socialNetwork.createSessionFor("user", credentials)
val f: Future[List[Friend]] = Future {
  session.getFriends()
}

```

上面，我们首先导入 `scala.concurrent ` 包的内容，使类型Future可见。我们将很快解释第二个导入(We will explain the second import shortly.)。

然后，我们使用一个假想的 `createSessionFor` 方法初始化一个 session 变量，该变量将用于向服务器发送请求。要获得用户的好友列表，必须通过网络发送请求，这可能需要很长时间。这可以通过调用 `getFriends` 方法来表示，该方法返回 `List[Friend]`。为了更好地利用(utilize ) CPU 直到响应到达(arrives)前，我们不应该阻塞程序的其余部分——这个计算应该异步调度。`Future.apply` 方法就是这样做的——它并行执行指定的计算块，在本例中，它向服务器发送请求并等待响应。

一旦服务器响应，在 future 变量 `f` 中的这个朋友列表就可用了。

不成功的尝试可能导致异常。在下面的示例中，session 变量没有真确的初始化，因此在 Future 块中的计算将抛出一个 NullPointerException。这个 Future `f` 会失败，得到一个 Exception，而不是成功完成:

```scala
val session = null
val f: Future[List[Friend]] = Future {
  session.getFriends
}
```

前面的代码中, `import ExecutionContext.Implicits` 行导入默认的全局执行上下文。执行上下文执行提交给它们的任务，您可以将执行上下文看作线程池。它们对 `Future.apply` 方法至关重要(essential )，因为它们处理如何以及何时执行异步计算。您可以定义自己的执行上下文并在 `Future` 中使用它们，但是现在只要知道您可以导入如上所示的默认执行上下文就够了(sufficient )。


我们的示例基于一个假设的社交网络 API，其中计算包括发送网络请求和等待响应。It is fair to offer an example involving an asynchronous computation which you can try out of the box。
假设(Assume )您有一个文本文件，并且希望找到特定关键字第一次出现的位置。在从磁盘检索文件内容时，这种计算可能会涉及阻塞，因此与计算的其余部分同时执行是有意义的。

```scala
val firstOccurrence: Future[Int] = Future {
  val source = scala.io.Source.fromFile("myText.txt")
  source.toSeq.indexOfSlice("myKeyword")
}

```

### Callbacks

现在我们知道如何启动异步计算来创建一个新的 Future，但是我们还没有展示如何在结果可用时使用它，这样我们就可以用它做一些有用的事情。我们常常对计算结果感兴趣，而不仅仅是它的副作用(side-effects)。


在许多 Future 的实现中，一旦 Future 的客户端对它的结果感兴趣，它就必须阻塞自己的计算，并等待 Future 完成——只有这样，它才能使用 Future 的值继续自己的计算。虽然Scala Future API允许这样做(我们将在稍后展示)，但是从性能的角度来看，更好的方法是完全非阻塞的方式，即注册一个回调函数到 Future 中。一旦 Future 完成，此回调将被异步调用。如果在注册回调时 Future 已经完成，那么回调可以异步执行，也可以在同一线程上顺序执行。

注册回调函数的最常见形式是使用 `onComplete` 方法, 使用类型为 `Try[T] => U` 的回调函数.如果 Future 成功完成，则回调应用于 `Success[T]` 类型的值，否则应用于 `Failure[T]` 类型的值。

类型 `Try[T]` 类似于 `Option[T]` 或 `Option[T, S]`，因为它是一个单子(monad )，可能(potentially )持有某种类型的值。然而，它被专门设计成要么保存一个值，要么保存某个可抛出的对象。当一个 `Option[T]` 可以是一个值(例如 `Some[T]`)或者根本没有值(例如 `None`)时，`Try[T]` 在它拥有一个值时是 `Success[T]`，否则 `Failure[T]` 则持有一个异常。`Failure[T]` 包含的信息不仅仅是一个简单的 `None`，它说明了为什么没有值。同时，您可以将 `Try[T]` 看作是 `Either[Throwable, T]` 的一个特殊版本，专门用于左值为 `Throwable` 的情况。

回到我们的社交网络示例，假设我们想获取我们自己最近的帖子列表，并将它们呈现到屏幕上。我们通过调用 `getRecentPosts` 方法来实现这一点，该方法返回一个 `List[String]` —— 一个最近的文本文章列表:

```scala
import scala.util.{Success, Failure}

val f: Future[List[String]] = Future {
  session.getRecentPosts
}

f onComplete {
  case Success(posts) => for (post <- posts) println(post)
  case Failure(t) => println("An error has occurred: " + t.getMessage)
}

```

`onComplete` 方法是通用的，因为它允许客户机处理将来计算失败和成功的结果。如果只需要处理成功的结果，可以使用 `foreach` 回调:

```scala
val f: Future[List[String]] = Future {
  session.getRecentPosts
}

f foreach { posts =>
  for (post <- posts) println(post)
}
```

Future 提供了一种只处理失败结果的干净方法，使用 `failed` 的投影将一个 `Failure[Throwable]` 转换为一个 `Success[Throwable]` 。
下面关于[投影 projection3](https://docs.scala-lang.org/overviews/core/futures.html#projections)的一节提供了这样做的一个例子。


回到前面的例子，搜索关键字的第一次出现，你可能想把关键字的位置打印到屏幕上:


```scala

val firstOccurrence: Future[Int] = Future {
  val source = scala.io.Source.fromFile("myText.txt")
  source.toSeq.indexOfSlice("myKeyword")
}

firstOccurrence onComplete {
  case Success(idx) => println("The keyword first appears at position: " + idx)
  case Failure(t) => println("Could not process file: " + t.getMessage)
}

```

`onComplete` 和 `foreach` 方法的结果类型是 `Unit`，这意味着不能链式调用这些方法。注意，这种设计是有意的，以避免暗示链式调用可能意味着对已注册回调的执行进行排序(在相同将来注册的回调是无序的)。

也就是说，我们现在应该对**何时**调用回调进行注释。因为它需要将来的值可用，所以只能在 Future 完成后调用它。但是，不能保证完成 future 调用的线程或创建回调的线程会调用它。相反，回调由某个线程执行，在 future 对象完成后的某个时候执行。我们说回调最**终会(eventually)**执行。

此外(Furthermore)，即使在同一应用程序的不同运行之间，执行回调的顺序也不是预先定义的。实际上，回调可能不会一个接一个地依次调用，但可以同时并发执行。这意味着在下面的示例中，变量 `totalA` 可能没有设置为计算文本中小写字母和大写字母 `a` 的正确数字。

```scala
@volatile var totalA = 0

val text = Future {
  "na" * 16 + "BATMAN!!!"
}

text foreach { txt =>
  totalA += txt.count(_ == 'a')
}

text foreach { txt =>
  totalA += txt.count(_ == 'A')
}
```


上面，两个回调可能会一个接一个地执行，在这种情况下，变量 totalA 保存期望值18。但是，它们也可以并发执行，因此 totalA 可以是 16 或 2，因为 `+=` 不是原子操作(即它包含一个读和一个写步骤，这个步骤可能与其他读和写任意交错)。

为了完整起见，回调的语义列在这里:


1. 在 future 上注册一个 `onComplete` 回调函数可以确保在 future 最终完成后调用相应的闭包。
2. 注册一个 `foreach` 回调函数与 `onComplete` 具有相同的语义，不同之处在于只有在 Future 成功完成时才调用闭包。
3. 在 Future 上注册一个已经完成的回调将最终执行回调(如1所示)。
4. 如果 Future 上注册了多个回调，则不定义它们执行的顺序。实际上，回调可以彼此并发执行。然而，特定的 `ExecutionContext` 实现可能会导致定义良好的顺序。
5. 如果某些回调引发异常，则无论如何(regardless)都会执行其他回调。
6. 如果某些回调函数从未完成(例如回调函数包含无限循环(infinite loop))，则其他回调函数可能根本不会执行。在这些情况下，潜在(potentially )的阻塞回调必须使用 `blocking` 结构(construct )(参见下面)。
7. 一旦执行，回调将从 future 对象中删除，从而符合 GC 的条件。


### Functional Composition and For-Comprehensions

<div style="color: #f00;">未完待续</div>





<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
