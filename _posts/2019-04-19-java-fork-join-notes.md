---
title: java 多任务并行框架之 Fork-Join
key: 20190419
tags: java fork-join
---

Fork/Join 框架是 Java7 中新增的一项特性，也是 Java7 平台的其中一项主要改进。

在实际情况中，很多时候我们都需要面对经典的“分治”问题。要解决这类问题，主要任务通常被分解为多个任务块（分解阶段），其后每一小块任务被独立并行计算。一旦计算任务完成，每一快的结果会被合并或者解决（解决阶段）。

“分治”问题可以很容易地通过 Callable 线程的 Executor 接口来解决。通过为每个任务实例化一个 Callable 实例，并在 ExecutorService 类中汇总计算结果来得出最终结果可以实现这一目的。那么自然而然想到的问题就是，如果这一 接口已经做得不错了，我们为什么还需要 Java 7 的其他框架？



<!--more-->
使用 ExecutorService和Callable的主要问题是，Callable实例在本质上是阻塞的。一旦一个Callable实例开始执行，其他所有Callable都会被阻塞。由于队列后面的Callable实例在前一实例未执行完成的时候不会被执行，因此许多资源无法得到利用。Fork/Join框架被引入来解决这一并行问题，而Executor解决的是并发问题（译者注：并发和并行的区别就是一个处理器同时处理多个任务和多个处理器或者是多核的处理器同时处理多个不同的任务）。

## Work stealing

Fork/Join 框架在 `java.util.concurrent` 包中加入了两个主要的类：

- `ForkJoinPool`
- `ForkJoinTask`

ForkJoinPool 类是 ForkJoinTask 实例的执行者，ForkJoinPool 的主要任务就是”**工作窃取**”，其线程尝试发现和执行其他任务创建的子任务。ForkJoinTask 实例与普通 Java 线程相比是非常轻量的。

一 旦 ForkJoinTask 被启动，就会启动其子任务并等待它们执行完成。执行者 ForkJoinPool 负责将子任务赋予线程池中处于等待任务状态的另一线程。线程池中的活动线程会尝试执行其他任务所创建的子任务。ForkJoinPool 会尝试在任何时候都维持与可用的处理器数目一样数目的活动线程数。

除了几个其他API方法以外，ForkJoinTask有两个主要的方法：

- `fork()` – 这个方法决定了 ForkJoinTask 的异步执行，凭借这个方法可以创建新的任务。
- `join()` – 该方法负责在计算完成侯返回结果，因此允许一个任务等待另一任务执行完成。


![fork-join](http://www.developer.com/imagesvr_ce/3378/join-fork-image001.png)
 

 参考： https://www.oracle.com/technetwork/articles/java/fork-join-422606.html


<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
