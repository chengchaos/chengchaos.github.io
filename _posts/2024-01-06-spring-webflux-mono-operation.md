---
title: Spring WebFlux 中的 Mono 操作
key: 2024-01-06
tags: java spring-webflux mono
---

2024 年上班后的第一个周末，本来想加班吧单位的事情做了，但是没动力。于是就整理一下常用的操作符。

Just so so.


<!--more-->

## 0x01 Flux Mono 是什么

`Flux<T>` 是一个标准的 `Publisher<T>`,  0 到 n 个发射项的异步序列。
`Mono<T>` 是一个专门的 `Publisher<T>`， 是一个发出（emit）  0 到 1 个元素的 Publisher，它最多发出一个项， 可选地以 `onComplete` 或 `onError` 信号终止。

Mono 值提供了 Flux 操作符的子集，并可以通过一些特殊操作符（特别是那些将 Mono 与另一个发布者结合的操作符）切换到 Flux。 

例如： `Mono#concatWith(Publisher)` 返回一个 Flux， 而 `Mono#then(Mono)` 则返回另一个 Mono。

## 0x02 创建

### 静态方法

#### 常见的

和 Flux 类似的：

- `Mono<T> just(T data)`
- `justOrEmpty(@Nullable Optional<? extends T> data)`
- `Mono<T> justOrEmpty(@Nullable T data)`
- `Mono.empty()`
- `Mono<T> never()`
- `Mono<T> defer(Supplier<? extends Mono<? extends T>> supplier)`

Mono 特有的：

- `fromCallable()`
- `fromCompletionStage()`
- `fromFuture()`
- `fromRunnable()`
- `fromSupplier()`
- `Mono<T> create(Consumer<MonoSink<T>> callback)`
- `Mono<Tuple2<T1, T2>> zip(Mono<? extends T1> p1, Mono<? extends T2> p2)`
- `Mono<T> ignoreElements(Publisher<T> source)`

#### `Mono.delay(Duration.ofSeconds(2))`

在指定的延迟时间后，产生数字 0 作为唯一值。(就是在指定时间后发出一个 0)



#### `ignoreElements(Publisher source)`

建一个Mono序列，忽略作为源的Publisher中的所有元素，只产生消息。就是创建出的结果不用了，中间过程还是会执行的


```c
   @Test
    void monoTest2() {
        Mono<String> mono4 = Mono.fromCallable(() -> {
            logger.info("*** this in mono4 ");
            return "mono4";
        });
        Mono<String> mono5 = Mono.ignoreElements(mono4);
        mono5.log()
                .subscribe();

    }
```

打印结果：


c.b.cheng.demo.webflux.WebFluxTest,monoTest2
15:29:06.459 [main] DEBUG reactor.util.Loggers - Using Slf4j logging framework
15:29:06.470 [main] INFO reactor.Mono.IgnorePublisher.1 - onSubscribe(MonoIgnoreElements.IgnoreElementsSubscriber)
15:29:06.472 [main] INFO reactor.Mono.IgnorePublisher.1 - request(unbounded)
15:29:06.472 [main] INFO c.b.cheng.demo.webflux.WebFluxTest - *** this in mono4 
15:29:06.472 [main] INFO reactor.Mono.IgnorePublisher.1 - onComplete()


#### `justOrEmpty(Optional<? extends T> data)` 和 `justOrEmpty(T data)`

从一个 Optional 对象或者可能为 null 的对象中创建 Mono。

```c
    @Test
    void test4() {
        Mono.fromSupplier(() -> "fromSupplier").subscribe(System.out::println);
        Mono.justOrEmpty(Optional.of("JustOrEmpty")).subscribe(System.out::println);
        Mono.create(sink -> sink.success("create")).subscribe(System.out::println);
        Mono.justOrEmpty(Optional.empty()).subscribe(System.out::println);
        Assertions.assertTrue(true);
    }
```


### 操作符

### `P as(Function<? super Mono<T>, P> transformer)`

转换这个 Mono 到一个目标类型

`mono.as(Flux::from).subscribe()`

```c

    @Test
    void test20() {
        Integer i = Mono.just("1st")
                .log("mono-as")
                .subscribeOn(Schedulers.boundedElastic())
                .as(m -> {
                    String block = m.block();
                    if (block == null) {
                        return -2;
                    }
                    try {
                        return Integer.parseInt(block);
                    } catch (Exception e) {
                        logger.error(e.getMessage(), e);
                    }
                    return -1;
                });
        logger.info("i => {}", i);
        Assertions.assertTrue(true);
    }

```

#### `Mono<Void> and(Publisher<?> other)`

Join the termination signals from this mono and another source into the returned void mono

```c

    @Test
    void test21() {
        Mono<String> mono1 = Mono.fromCallable(() -> {
            TimeUnit.SECONDS.sleep(2L);
            logger.info("数据准备完成");
            return "mono1";
        });

        Flux.interval(Duration.ofMillis(0), Duration.ofMillis(100))
                .log("flux-2")
                .take(2)
                .reduce(Long::sum)
                .and(mono1)
                .subscribe();
        Assertions.assertTrue(true);
    }
```

#### `Mono<E> cast(Class<E> clazz)`

Cast the current Mono produced type into a target produced type.

```c
    @Test
    void test22() {
        Mono<Object> mono1 = Mono.fromCallable(() -> {
            logger.info("数据准备完成");
            return "mono1";
        });

        Mono<String> stringMono = mono1.cast(String.class);
        stringMono.subscribe(System.out::println);

        Assertions.assertTrue(true);
    }
```

#### `Mono<T> defaultIfEmpty(T defaultV)`

Provide a default single value if this mono is completed without any data.

```c
  @Test
    void test23() {

        Mono<Object> mono1 = Mono.fromSupplier(() -> {
            ThreadLocalRandom random = ThreadLocalRandom.current();
            int c = random.nextInt(100);
            logger.info("c = {}", c);
            if (c % 2 == 0){
                return "OK";
            }
            return null;
        });

        Mono<String> stringMono = mono1.cast(String.class)
                .defaultIfEmpty("Empty");

        stringMono.subscribe(System.out::println);

        Assertions.assertTrue(true);
    }
```

### `Flux<T> expand(Function<? super T, ? extends Publisher<? extends T>> expander)`

Recursively expand elements into a graph and emit all the resulting element using a breadth-first traversal strategy.



#### `Flux#buffer` 和 `Flix#bufferTimeout`

- 这两个操作符的作用是把当前流中的元素收集到集合中，并把集合对象作为流中的新元素。
- 在进行收集时可以指定不同的条件：所包含的元素的最大数量或收集的时间间隔。方法 `buffer()` 仅使用一个条件，而 `bufferTimeout()` 可以同时指定两个条件。。

除了元素数量和时间间隔外，还可以通过 `bufferUntil` 和`bufferWhile` 操作符来进行收集。这两个操作符的参数时表示每个集合中的元素索要满足的条件的 `Predicate` 对象。

- `bufferUntil`会一直收集直到 `Predicate` 返回 `true`, 当前收集完成， 开始下一次收集。
- 使得 `Predicate` 返回 `true` 的那个元素可以选择添加到当前集合或下一个集合中 （ `bufferUntil` 的第二个参数设置为 true ）；
- `bufferWhile` 则只有当 `Predicate` 返回 `true` 时才会收集。一旦为 `false`，会立即开始下一次收集。

```c
  @Test
    void test5() {

        Flux.range(1, 100)
                .buffer(20)
                .log()
                .subscribe(System.out::println);

        logger.info("===========");

        Flux.interval(Duration.ofMillis(100))
                .bufferTimeout(20, Duration.ofMillis(1001))
                .take(2)
                .toStream()
                .forEach(System.out::println);
        logger.info("===========");

        Flux.range(1, 10)
                .bufferUntil(i -> i % 2 == 0)
                .subscribe(System.out::println);
        logger.info("===========");

        Flux.range(1, 10)
                .bufferUntil(i -> i % 2 == 0, true)
                .subscribe(System.out::println);
        logger.info("===========");

        Flux.range(1, 20)
                .bufferWhile(i -> i % 3 == 0)
                .subscribe(System.out::println);
        waitting();

        Assertions.assertTrue(true);
    }
```

#### `Flux#filter`

对流中包含的元素进行过滤，只留下满足 `Predicate` 条件的元素

```c
    Flux.range(1, 10)
            .filter(i -> i % 2 == 0)
            .subscribe(System.out::println);
```

#### `Flux#zipWith`

`zipWith` 操作符把当前流中的元素与另一个流中的元素按照一对一的方式进行合并。在合并时可以不做任何处理，由此得到的是一个元素类型为 `Tuple2` 的流；也可以通过一个 `BiFunction` 函数对合并的元素进行处理，所得到的流的元素类型为该函数的返回值。

```c
    @Test
    void test6() {

        // 打印出：
        //[a,1]
        //[b,2]
        Flux.just("a", "b")
                .zipWith(Flux.just("1", "2"))
                .subscribe(System.out::println);
        // 打印出： [1,上山打老虎]
        Flux.just(1, 2, 3, 4, 5)
                .zipWith(Flux.just("上山打老虎"))
                .subscribe(System.out::println);
        // 打印出：
        //1 -> 一
        //2 -> 二
        Flux.just("1", "2")
                .zipWith(Flux.just("一", "二"), (s1, s2) -> String.format("%s -> %s", s1, s2))
                .subscribe(System.out::println);
        Assertions.assertTrue(true);
    }
```

#### `Flux#take`

`take` 系列操作符用来从当前流中提取元素。提取方式如下：

- `take(long n)`，`take(Duration timespan)` 和 `takeMillis(long timespan)` : 按照指定的数量或时间间隔来提取
- `takeLast(long n)` ：提取流中的最后N个元素
- `takeUntil(Predicate<? super T> predicate)` ：提取元素直到 `Predicate`返回 `true` 后停止。
- `takeWhile(Predicate<? super T> continuePredicate)` ： 当 `Predicate` 返回`true` 时才进行提取。
- `takeUntilOther(Publisher<?> other)` ： 提取元素直到另外一个流开始产生元素

```c
   @Test
    void test7() {

        Flux.range(1, 100)
                .take(10).subscribe(System.out::println);

        Flux.range(1, 1000)
                .takeLast(10)
                .subscribe(System.out::println);
        Flux.range(1, 1000)
                .takeWhile(i -> i < 10)
                .subscribe(System.out::println);
        Flux.range(1, 1000)
                .takeUntil(i -> i == 10)
                .subscribe(System.out::println);
        Assertions.assertTrue(true);
    }

```

#### `Flux#reduce` 和 `Flux#reduceWith`

`reduce` 和 `reduceWith` 操作符对流中包含的所有元素进行累计操作，得到一个包含计算结果的 `Mono` 序列。累计操作是通过一个 `BiFunction` 来表示的。

在操作时可以指定一个初始值。若没有初始值，则序列的第一个元素作为初始值。

```c
    @Test
    void test8() {
        Flux.range(1, 100)
                .reduce((x, y) -> x + y)
                .subscribe(System.out::println);

        Flux.range(1, 100)
                .reduceWith(() -> 100, Integer::sum)
                .subscribe(System.out::println);
        Assertions.assertTrue(true);
    }
```

#### `Flux#merge` 和 `Flux#mergeSequential`

`merge` 和 `mergeSequential` 操作符用来把多个流合并成一个 Flux 序列。

`merge` 按照所有流中元素的实际产生序列来合并。

`mergeSequential` 按照所有流被订阅的顺序，以流为单位进行合并。

```c
    @Test
    void test9() {
        // 打印 0 0 1 1 2 
        // 然后一个 Flux 结束了，merge 也结束了。
        Flux.merge(Flux.interval(Duration.ofMillis(0), Duration.ofMillis(100)),
                        Flux.interval(Duration.ofMillis(50), Duration.ofMillis(100)))
                .take(5)
                .toStream()
                .forEach(System.out::println);
        // 打印 0 1 2 3 4 然后再打印另一个 Flux 的 0 1 2 3 4
        Flux.mergeSequential(Flux.interval(Duration.ofMillis(0), Duration.ofMillis(100)).take(5),
                        Flux.interval(Duration.ofMillis(50), Duration.ofMillis(100)).take(5))
                .toStream()
                .forEach(System.out::println);

        Assertions.assertTrue(true);
    }
```

#### `Flux#flatMap` 和 `Flux#flatMapSequential`

`flatMap` 和 `flatMapSequential` 操作符把流中的每个元素转换成一个流，再把所有流中的元素进行合并。

`flatMapSequential` 和 `flatMap` 之间的区别与 `mergeSequential` 和 `merge` 的区别是一样的。

```c
    @Test
    void test10() {

        Flux.just(5, 10)
                .log()
                .flatMap(x -> Flux.interval(Duration.ofMillis(x * 10), Duration.ofMillis(100))
                        .log()
                        .take(x))
                .toStream()
                .forEach(System.out::println);
        Assertions.assertTrue(true);
    }
```

#### `Flux#comcatMap`

`concatMap` 操作符的作用也是把流中的每个元素转换成一个流，再把所有流进行合并。c

`oncatMap` 会根据原始流中的元素顺序依次把转换之后的流进行合并，并且 `concatMap` 堆转换之后的流的订阅是动态进行的，而 `flatMapSequential` 在合并之前就已经订阅了所有的流。

```c
    @Test
    void test11() {
        Flux.just(5, 10)
                .concatMap(x -> Flux.interval(Duration.ofMillis(x * 10), Duration.ofMillis(100)).take(x))
                .toStream()
                // 先打印 1 to 5, 再打印 1 to 10
                .forEach(System.out::println);
        Assertions.assertTrue(true);
    }
```

#### `Flux#combineLatest`

`combineLatest` 操作符把所有流中的最新产生的元素合并成一个新的元素，作为返回结果流中的元素。

只要其中任何一个流中产生了新的元素，合并操作就会被执行一次，结果流中就会产生新的元素。

```c
  @Test
    void test12() {
        Flux.combineLatest(Arrays::toString,
                        Flux.interval(Duration.ofMillis(100)).take(5),
                        Flux.interval(Duration.ofMillis(50), Duration.ofMillis(100)).take(5))
                .toStream()
                .forEach(System.out::println);
        //[0, 0]
        //[0, 1]
        //[1, 1]
        //[1, 2]
        //[2, 2]
        //[2, 3]
        //[3, 3]
        //[3, 4]
        //[4, 4]
        //waitting();
        Assertions.assertTrue(true);
    }
```

#### `Mono<V> then(Mono<V> other)`

执行完当前的 Mono 然后执行另外一个。

> Let this Mono complete then play another Mono.

> In other words ignore element from this Mono and transform its completion signal into the emission and completion signal of a provided Mono<V>. Error signal is replayed in the resulting Mono<V>.

### 消息处理

#### `subscribe()`

需要处理 Flux 或者 Mono 中的消息是， 可以通过 `subscribe` 方法来添加响应的订阅逻辑。

在调用 `subscribe` 方法时可以指定需要处理的消息类型。

```c
    @Test
    void test13() {
        Flux.just(1, 2)
                .concatWith(Mono.just(18))
                .subscribe(System.out::println, System.err::println);

        Flux.just(1, 2)
                .concatWith(Mono.error(new IllegalStateException()))
                .subscribe(System.out::println, System.err::println);

        Flux.just(1, 2)
                .concatWith(Mono.error(new IllegalStateException()))
                .onErrorReturn(0)
                .subscribe(i -> System.out.println(":: "+ i));
        Assertions.assertTrue(true);
    }
```

#### `onErrorResume()`

还可以通过 `onErrorResume()` 方法来根据不同的异常类型来选择要使用的产生元素的流。

当出现错误时还可以使用 `retry` 操作符来进行重试。重试的动作是通过重新订阅序列来实现的。在使用 `retry` 操作时还可以指定重试的次数。

```c
    @Test
    void test15() {
        Flux.just(1, 2)
                .concatWith(Mono.error(new IllegalStateException()))
                .onErrorResume((e) -> {
                    logger.error(" ++++++++ >> ", e);
                    logger.error(" ++++++++ >> ", e);
                    if (e instanceof IllegalStateException) {
                        return Mono.just(18);
                    }
                    else if (e instanceof IllegalArgumentException) {
                        return Mono.just(-1);
                    }
                    return Mono.empty();
                })

                .subscribe(System.out::println);
        Assertions.assertTrue(true);
    }

    @Test
    void test16() {
        Flux.just(1, 2)
                .concatWith(Mono.error(new IllegalStateException()))
                .retry(2)
                .subscribe(System.out::println);
        Assertions.assertTrue(true);
    }
```

### 调度器 Scheduler

通过调度器可以指定操作执行的方式和所在的线程。有以下几种不同的调度器实现

- 当前线程，通过 `Schedulers.immediate()` 方法来创建。
- 单一的可复用的线程，通过 `Schedulers.single()` 方法来创建。
- 使用弹性的线程池，通过 `Schedulers.boundedElastic()` 方法来创建。线程池中的线程是可以复用的。当所需要时，新的线程会被创建。若一个线程闲置时间太长，则会被销毁。该调度器适用于 I/O 操作相关的流的处理。
- 使用对并行操作优化的线程池，通过 `Schedulers.parallel()`方法来创建。其中的线程数量取决于 CPU 的核的数量。该调度器适用于计算密集型的流的处理。
- 使用支持任务调度的调度器，通过 `Schedulers.timer()` 方法来创建。
- 从已有的 `ExecutorService` 对象中创建调度器，通过 `Schedulers.fromExecutorService()` 方法来创建。
- 自定义的调度器 `newBoundedElastic` 等方法创建。


通过 `publishOn()` 和 `subscribeOn()` 方法可以切换执行操作调度器。`publishOn()` 方法切换的是操作符的执行方式，而 `subscribeOn()` 方法切换的是产生流中元素时的执行方式


```c
 @Test
    void test17() {

        Flux.create(sink -> {
                    sink.next(Thread.currentThread().getName() + "::");
                    sink.complete();
                })
                .publishOn(Schedulers.single())
                .map(x -> String.format("::1 [%s] (%s) ", Thread.currentThread().getName(), x))
                .map(x -> String.format("::2 [%s] (%s) ", Thread.currentThread().getName(), x))
                .subscribeOn(Schedulers.newBoundedElastic(10, 10, "my-bounded"))
                .publishOn(Schedulers.boundedElastic())
                .map(x -> String.format("::3 [%s] (%s) ", Thread.currentThread().getName(), x))
                .subscribeOn(Schedulers.parallel())
                .map(x -> String.format("::4 [%s] (%s) ", Thread.currentThread().getName(), x))
                .subscribeOn(Schedulers.newBoundedElastic(10, 10, "my-bounded"))
                .map(x -> String.format("::5 [%s] (%s) ", Thread.currentThread().getName(), x))
                .subscribeOn(Schedulers.newBoundedElastic(10, 10, "my-bounded"))
                .map(x -> String.format("::6 [%s] (%s) ", Thread.currentThread().getName(), x))
                .log()
                .toStream()
                .forEach(System.out::println);
        Assertions.assertTrue(true);
    }

```

### 测试

`StepVerifier` 的作用是可以对序列中包含的元素进行逐一验证。

通过 `StepVerifier.create()` 方法对一个流进行包装之后再进行验证。`expectNext()` 方法用来声明测试时所期待的流中的下一个元素的值，而 `verifyComplete()` 方法则验证流是否正常结束。`verifyError()` 来验证流由于错误而终止。


`TestPublisher` 的作用在于可以控制流中元素的产生，甚至是违反反应流规范的情况。通过 `create()` 方法创建一个新的 `TestPublisher` 对象，然后使用 `next()` 方法来产生元素，使用 `complete()` 方法来结束流。

```c

    @Test
    void test18() {

        StepVerifier.withVirtualTime(() -> Flux.interval(Duration.ofHours(4),
                        Duration.ofDays(1)).take(2))
                .expectSubscription()
                .expectNoEvent(Duration.ofHours(4))
                .expectNext(0L)
                .thenAwait(Duration.ofDays(1))
                .expectNext(1L)
                .verifyComplete();
        Assertions.assertTrue(true);
    }

    @Test
    void test19() {
        final TestPublisher<String> testPublisher = TestPublisher.create();
        testPublisher.next("a");
        testPublisher.next("b");
        testPublisher.complete();

        StepVerifier.create(testPublisher)
                .expectNext("a")
                .expectNext("b")
                //.verifyComplete();
                .expectComplete();
        
        Assertions.assertTrue(true);
    }
```


### 调试和日志

在调试模式启用之后，所有的操作符在执行时都会保存额外的与执行链相关的信息。当出现错误时，这些信息会被作为异常堆栈信息的一部分输出。


```c
Hooks.onOperator(providedHook -> providedHook.operatorStacktrace());

```

也可以通过 `checkpoint` 操作符来对特定的流处理链来启用调试模式，也可以通过添加log操作把流相关的事件记录在日志中。



```c
    @Test
    void test20() {

        Flux.just(1, 0)
                .map(x -> 1 / x)
                .log("Range")
                .checkpoint("test")
                .subscribe(System.out::println);
        
        Assertions.assertTrue(true);
    }
```

输出：

```
20:15:56.437 [main] DEBUG reactor.util.Loggers - Using Slf4j logging framework
20:15:56.448 [main] INFO Range - | onSubscribe([Fuseable] FluxMapFuseable.MapFuseableSubscriber)
20:15:56.450 [main] INFO Range - | request(unbounded)
20:15:56.450 [main] INFO Range - | onNext(1)
1
20:15:56.453 [main] ERROR Range - | onError(java.lang.ArithmeticException: / by zero)
20:15:56.454 [main] ERROR Range - 
java.lang.ArithmeticException: / by zero
    at c.b.cheng.demo.webflux.WebFluxTest.lambda$test20$36(WebFluxTest.java:425)
    at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:113)
    at reactor.core.publisher.FluxArray$ArraySubscription.fastPath(FluxArray.java:172)
    at reactor.core.publisher.FluxArray$ArraySubscription.request(FluxArray.java:97)
    at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.request(FluxMapFuseable.java:171)
    at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.request(FluxPeekFuseable.java:144)
    at reactor.core.publisher.FluxOnAssembly$OnAssemblySubscriber.request(FluxOnAssembly.java:649)
    at reactor.core.publisher.LambdaSubscriber.onSubscribe(LambdaSubscriber.java:119)
    at reactor.core.publisher.FluxOnAssembly$OnAssemblySubscriber.onSubscribe(FluxOnAssembly.java:633)
    at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.onSubscribe(FluxPeekFuseable.java:178)
    at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onSubscribe(FluxMapFuseable.java:96)
    at reactor.core.publisher.FluxArray.subscribe(FluxArray.java:53)
    at reactor.core.publisher.FluxArray.subscribe(FluxArray.java:59)
    at reactor.core.publisher.Flux.subscribe(Flux.java:8642)
    at reactor.core.publisher.Flux.subscribeWith(Flux.java:8815)
    at reactor.core.publisher.Flux.subscribe(Flux.java:8608)
    at reactor.core.publisher.Flux.subscribe(Flux.java:8532)
    at reactor.core.publisher.Flux.subscribe(Flux.java:8475)
    at c.b.cheng.demo.webflux.WebFluxTest.test20(WebFluxTest.java:428)
    at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
    at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.base/java.lang.reflect.Method.invoke(Method.java:568)
    at org.junit.platform.commons.util.ReflectionUtils.invokeMethod(ReflectionUtils.java:725)
    at org.junit.jupiter.engine.execution.MethodInvocation.proceed(MethodInvocation.java:60)
    at org.junit.jupiter.engine.execution.InvocationInterceptorChain$ValidatingInvocation.proceed(InvocationInterceptorChain.java:131)
    at org.junit.jupiter.engine.extension.TimeoutExtension.intercept(TimeoutExtension.java:149)
    at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestableMethod(TimeoutExtension.java:140)
    at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestMethod(TimeoutExtension.java:84)
    at org.junit.jupiter.engine.execution.ExecutableInvoker$ReflectiveInterceptorCall.lambda$ofVoidMethod$0(ExecutableInvoker.java:115)
    at org.junit.jupiter.engine.execution.ExecutableInvoker.lambda$invoke$0(ExecutableInvoker.java:105)
    at org.junit.jupiter.engine.execution.InvocationInterceptorChain$InterceptedInvocation.proceed(InvocationInterceptorChain.java:106)
    at org.junit.jupiter.engine.execution.InvocationInterceptorChain.proceed(InvocationInterceptorChain.java:64)
    at org.junit.jupiter.engine.execution.InvocationInterceptorChain.chainAndInvoke(InvocationInterceptorChain.java:45)
    at org.junit.jupiter.engine.execution.InvocationInterceptorChain.invoke(InvocationInterceptorChain.java:37)
    at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:104)
    at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:98)
    at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.lambda$invokeTestMethod$7(TestMethodTestDescriptor.java:214)
    at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
    at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.invokeTestMethod(TestMethodTestDescriptor.java:210)
    at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:135)
    at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:66)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:151)
    at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
    at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
    at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
    at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
    at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
    at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
    at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
    at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
    at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
    at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
    at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
    at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
    at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
    at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
    at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.submit(SameThreadHierarchicalTestExecutorService.java:35)
    at org.junit.platform.engine.support.hierarchical.HierarchicalTestExecutor.execute(HierarchicalTestExecutor.java:57)
    at org.junit.platform.engine.support.hierarchical.HierarchicalTestEngine.execute(HierarchicalTestEngine.java:54)
    at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:107)
    at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:88)
    at org.junit.platform.launcher.core.EngineExecutionOrchestrator.lambda$execute$0(EngineExecutionOrchestrator.java:54)
    at org.junit.platform.launcher.core.EngineExecutionOrchestrator.withInterceptedStreams(EngineExecutionOrchestrator.java:67)
    at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:52)
    at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:114)
    at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:86)
    at org.junit.platform.launcher.core.DefaultLauncherSession$DelegatingLauncher.execute(DefaultLauncherSession.java:86)
    at org.junit.platform.launcher.core.SessionPerRequestLauncher.execute(SessionPerRequestLauncher.java:53)
    at com.intellij.junit5.JUnit5IdeaTestRunner.startRunnerWithArgs(JUnit5IdeaTestRunner.java:57)
    at com.intellij.rt.junit.IdeaTestRunner$Repeater$1.execute(IdeaTestRunner.java:38)
    at com.intellij.rt.execution.junit.TestsRepeater.repeat(TestsRepeater.java:11)
    at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:35)
    at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:232)
    at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:55)
20:15:56.458 [main] ERROR reactor.core.publisher.Operators - Operator called default onErrorDropped
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.ArithmeticException: / by zero
Caused by: java.lang.ArithmeticException: / by zero
    at c.b.cheng.demo.webflux.WebFluxTest.lambda$test20$36(WebFluxTest.java:425)
    Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Error has been observed at the following site(s):
    *__checkpoint ⇢ test
Original Stack Trace:
        at c.b.cheng.demo.webflux.WebFluxTest.lambda$test20$36(WebFluxTest.java:425)
        at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:113)
        at reactor.core.publisher.FluxArray$ArraySubscription.fastPath(FluxArray.java:172)
        at reactor.core.publisher.FluxArray$ArraySubscription.request(FluxArray.java:97)
        at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.request(FluxMapFuseable.java:171)
        at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.request(FluxPeekFuseable.java:144)
        at reactor.core.publisher.LambdaSubscriber.onSubscribe(LambdaSubscriber.java:119)
        at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.onSubscribe(FluxPeekFuseable.java:178)
        at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onSubscribe(FluxMapFuseable.java:96)
        at reactor.core.publisher.FluxArray.subscribe(FluxArray.java:53)
        at reactor.core.publisher.FluxArray.subscribe(FluxArray.java:59)
        at reactor.core.publisher.Flux.subscribe(Flux.java:8642)
        at reactor.core.publisher.Flux.subscribeWith(Flux.java:8815)
        at reactor.core.publisher.Flux.subscribe(Flux.java:8608)
        at reactor.core.publisher.Flux.subscribe(Flux.java:8532)
        at reactor.core.publisher.Flux.subscribe(Flux.java:8475)
        at c.b.cheng.demo.webflux.WebFluxTest.test20(WebFluxTest.java:428)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.base/java.lang.reflect.Method.invoke(Method.java:568)
        at org.junit.platform.commons.util.ReflectionUtils.invokeMethod(ReflectionUtils.java:725)
        at org.junit.jupiter.engine.execution.MethodInvocation.proceed(MethodInvocation.java:60)
        at org.junit.jupiter.engine.execution.InvocationInterceptorChain$ValidatingInvocation.proceed(InvocationInterceptorChain.java:131)
        at org.junit.jupiter.engine.extension.TimeoutExtension.intercept(TimeoutExtension.java:149)
        at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestableMethod(TimeoutExtension.java:140)
        at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestMethod(TimeoutExtension.java:84)
        at org.junit.jupiter.engine.execution.ExecutableInvoker$ReflectiveInterceptorCall.lambda$ofVoidMethod$0(ExecutableInvoker.java:115)
        at org.junit.jupiter.engine.execution.ExecutableInvoker.lambda$invoke$0(ExecutableInvoker.java:105)
        at org.junit.jupiter.engine.execution.InvocationInterceptorChain$InterceptedInvocation.proceed(InvocationInterceptorChain.java:106)
        at org.junit.jupiter.engine.execution.InvocationInterceptorChain.proceed(InvocationInterceptorChain.java:64)
        at org.junit.jupiter.engine.execution.InvocationInterceptorChain.chainAndInvoke(InvocationInterceptorChain.java:45)
        at org.junit.jupiter.engine.execution.InvocationInterceptorChain.invoke(InvocationInterceptorChain.java:37)
        at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:104)
        at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:98)
        at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.lambda$invokeTestMethod$7(TestMethodTestDescriptor.java:214)
        at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
        at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.invokeTestMethod(TestMethodTestDescriptor.java:210)
        at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:135)
        at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:66)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:151)
        at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
        at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
        at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
        at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
        at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
        at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
        at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
        at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
        at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
        at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
        at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
        at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
        at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
        at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
        at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.submit(SameThreadHierarchicalTestExecutorService.java:35)
        at org.junit.platform.engine.support.hierarchical.HierarchicalTestExecutor.execute(HierarchicalTestExecutor.java:57)
        at org.junit.platform.engine.support.hierarchical.HierarchicalTestEngine.execute(HierarchicalTestEngine.java:54)
        at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:107)
        at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:88)
        at org.junit.platform.launcher.core.EngineExecutionOrchestrator.lambda$execute$0(EngineExecutionOrchestrator.java:54)
        at org.junit.platform.launcher.core.EngineExecutionOrchestrator.withInterceptedStreams(EngineExecutionOrchestrator.java:67)
        at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:52)
        at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:114)
        at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:86)
        at org.junit.platform.launcher.core.DefaultLauncherSession$DelegatingLauncher.execute(DefaultLauncherSession.java:86)
        at org.junit.platform.launcher.core.SessionPerRequestLauncher.execute(SessionPerRequestLauncher.java:53)
        at com.intellij.junit5.JUnit5IdeaTestRunner.startRunnerWithArgs(JUnit5IdeaTestRunner.java:57)
        at com.intellij.rt.junit.IdeaTestRunner$Repeater$1.execute(IdeaTestRunner.java:38)
        at com.intellij.rt.execution.junit.TestsRepeater.repeat(TestsRepeater.java:11)
        at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:35)
        at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:232)
        at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:55)

Process finished with exit code 0
```

## 参考和抄袭:




