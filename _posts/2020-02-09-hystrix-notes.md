---
title: Hystrix 学习笔记[0001]
key: 2020-02-09
tags: java hystrix
---

As The Titile

## 简介



在大中型分布式系统中，通常系统很多依赖。在高并发访问下,这些依赖的稳定性与否对系统的影响非常大,但是依赖有很多不可控问题:如网络连接缓慢，资源繁忙，暂时不可用，服务脱机等。在复杂的分布式架构的应用程序有很多的依赖，都会不可避免地在某些时候失败。高并发的依赖失败时如果没有隔离措施，当前应用服务就有被拖垮的风险。一般来说，随着服务依赖数量的变多，服务不稳定的概率会成指数性提高。例如：

一个依赖30个SOA服务的系统,每个服务99.99%可用。
99.99%的30次方 ≈ 99.7%，
0.3% 意味着一亿次请求 会有 3,000,00次失败，
换算成时间大约每月有2个小时服务不稳定。
解决这个问题的方案是对依赖进行隔离。Hystrix就是处理依赖隔离的框架,同时也是可以帮我们做依赖服务的治理和监控。Hystrix英文翻译就是豪猪，豪猪科动物以棘刺闻名，棘刺有保护御敌作用。Netflix自称Hystrix在其内部的使用规模如下：

The Netflix API processes 10+ billion HystrixCommand executions per day using thread isolation.
Each API instance has 40+ thread-pools with 5-20 threads in each (most are set to 10).
[Netflix API每天使用线程隔离处理100+亿次的HystrixCommand。
每个API实例都有40多个线程池，每个线程池都有5-20个线程(大多数都设置为10个线程)。] 

<!--more-->

## 原理

Hystrix使用命令模式(Command)包装依赖调用逻辑，每个命令在单独线程中/信号授权下执行。
可配置依赖调用超时时间,超时时间一般设为比99.5%平均时间略高即可。当调用超时时，直接返回或执行fallback（降级）逻辑。
为每个依赖提供一个小的线程池（或信号），如果线程池已满调用将被立即拒绝，默认不采用排队（使用SynchronousQueue和拒绝策略），加速失败判定时间(快速失败)。
依赖调用结果分：成功，失败（抛出异常），超时，线程拒绝，短路。请求失败(异常，拒绝，超时，短路)时执行fallback(降级)逻辑。
提供熔断器组件（下一个小节详细说明）。
提供近实时依赖的统计和监控（详细的metrics（度量）信息）。

## 熔断机制



下面简单说明一下 Hystrix 的熔断机制。

是否开启熔断器主要由依赖调用的错误比率决定的，依赖调用的错误比率 **=请求失败数 /  请求总数**。

Hystrix 中断路器打开的默认请求错误比率为 50%（这里暂时称为请求错误率），还有一个参数，用于设置在一个滚动窗口中，打开断路器的最少请求数（这里暂时称为滚动窗口最小请求数），这里举个具体的例子：如果滚动窗口最小请求数为 20，在一个窗口内（比如 10 秒，统计滚动窗口的时间可以设置，见下面的参数详解），收到 19 个请求，即使这19个请求都失败了，此时请求错误率高达95%，但是断路器也不会打开。

对于被熔断的请求，并不是永久被切断，而是被暂停一段时间（**默认是5000ms**）之后，允许部分请求通过，若请求都是健康的（`ResponseTime < 250ms`）则对请求健康恢复（取消熔断），如果不是健康的，则继续熔断。

（这里很容易出现一种错觉：多个请求失败但是没有触发熔断。这是因为在一个滚动窗口内的失败请求数没有达到打开断路器的最少请求数）


## 配置参数



1. 内置全局默认值（Global default from code）
   如果下面3种都没有设置，默认是使用此种，后面用”默认值”代指这种。

2. 动态全局默认属性（Dynamic global default property）
   可以通过属性配置来更改全局默认值，后面用”默认属性”代指这种。

3. 内置实例默认值（Instance default from code）
   在代码中，设置的属性值，后面用”实例默认”来代指这种。

4. 动态配置实例属性（Dynamic instance property）
   可以针对特定的实例，动态配置属性值，来代替前面三种，后面用”实例属性”来代指这种。

优先级：`1 < 2 < 3 < 4`。 这些配置基本上可以从 `com.netflix.hystrix.HystrixCommandProperties` 和 `com.netflix.hystrix.HystrixThreadPoolProperties` 查看。




## 基础属性配置



### CommandGroup

`CommandGroup` 是每个命令最少配置的必选参数，在不指定 `ThreadPoolKey `的情况下，字面值用于对不同依赖的线程池/信号区分，也就是在不指定 `ThreadPoolKey` 的情况下, `CommandGroup` 用于指定线程池的隔离。命令分组用于对依赖操作分组，便于统计、汇总等。

- 实例属性：`com.netflix.hystrix.HystrixCommandGroupKey`
- 实例配置：`HystrixCommand.Setter().withGroupKey (HystrixCommandGroupKey.Factory.asKey(“Group”));`
- 注解使用：`@HystrixCommand(groupKey = “Group”)`



### CommandKey





### ThreadPoolKey





## 命令属性配置




































<< EOF >>




原文链接：https://blog.csdn.net/zjcsuct/article/details/78198632

https://github.com/zjcscut/Reading-Notes-Repository/blob/master/%E5%85%B6%E4%BB%96/Hystrix.md



If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
