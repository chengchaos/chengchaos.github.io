---
title: Java 8 CompletableFuture 学习笔记
key: 20190102
tags: java8, CompletableFuture
---

CompletableFuture 是 Java 8 引入的相当碉堡的函数式异步编程辅助类。

<!--more-->

## Future 接口

Future 接口在 Java 5 中被引入，设计初衷是对将来某个时刻会发生的结果进行建模。它建模了一种异步计算，返回一个执行运算结果的引用，当运算结束后，这个引用被返回给调用方。在 Future 中触发那些潜在耗时的操作把调用线程解放出来，让它能继续执行其他有价值的工作，不再需要呆呆等待耗时的操作完成。

Future 的另一个优点是它比更底层的 Thread 更易用。要使用 Future，通常你只需要将耗时的操作封装在一个 Callable 对象中，再将它提交给 ExecutorService，就万事大吉了。

使用 Future 以异步方式执行一个耗时的操作：

```java

    private Double doSomeLongComputation() {

        try {
            TimeUnit.SECONDS.sleep(2L);
            return 42D;
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        return 0D;
    }

    private void doSomethingElse() {
        System.out.println("hai ...");
    }

    @Test
    public void testFuture5() {

        ExecutorService executor = Executors.newCachedThreadPool();

        Future<Double> future = executor.submit(
                new Callable<Double>() {
                    @Override
                    public Double call() throws Exception {
                        return doSomeLongComputation();
                    }
                }
        );

        boolean done = future.isDone();

        doSomethingElse();
        System.out.println("is done ==> "+ done);

        try {
            Double result = future.get(3L, TimeUnit.SECONDS);
            Double expected = 42D;
            Assert.assertEquals("不相等", expected, result);
            System.out.println("result ==> "+ result);
            done = future.isDone();
            boolean cancelled = future.isCancelled();

            System.out.println("is done ==> "+ done +", is cancelled ==> "+ cancelled);

            TimeUnit.SECONDS.sleep(5L);
        } catch (ExecutionException ee) {
            /* 计算抛出一个异常 */
        } catch (InterruptedException ie) {
            /* 被中断异常 */
        } catch (TimeoutException te) {
            /* Future 完成前超时 */
        }
    }
```

可以使用 `isDone` 方法检查计算是否完成，或者使用 `get` 阻塞住调用线程，直到计算完成返回结果，你也可以使用 `cancel` 方法停止任务的执行。

虽然 Future 以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的 CPU 资源，而且也不能及时地得到计算结果，为什么不能用观察者设计模式当计算结果完成及时通知监听者呢？

很多语言，比如 Node.js，采用回调的方式实现异步编程。Java 的一些框架，比如 Netty，自己扩展了 Java 的 Future 接口，提供了 `addListener` 等多个扩展方法：

```java
	ChannelFuture future = bootstrap.connect(new InetSocketAddress(host, port));
    future.addListener(new ChannelFutureListener() {
		@Override
		public void operationComplete(ChannelFuture future) throws Exception {
			if (future.isSuccess()) {
				// SUCCESS
			} else {
				 // FAILURE
			}
		}
	});

```

比如 Google 的 guava 也提供了通用的扩展 Future: [ListenableFuture](http://google.github.io/guava/releases/19.0/api/docs/com/google/common/util/concurrent/ListenableFuture.html)、[SettableFuture](http://google.github.io/guava/releases/19.0/api/docs/com/google/common/util/concurrent/SettableFuture.html) 以及辅助类 [Futures](http://google.github.io/guava/releases/19.0/api/docs/com/google/common/util/concurrent/Futures.html) 等,方便异步编程。

```java
final String name = "MyName";
inFlight.add(name);
ListenableFuture<Result> future = service.query(name);
future.addListener(new Runnable() {
	public void run() {
		processedCount.incrementAndGet();
		inFlight.remove(name);
		lastProcessed.set(name);
		logger.info("Done with {0}", name);
	}
}, executor);
```

比如 Scala 也提供了简单易用且功能强大的 [Future](http://docs.scala-lang.org/overviews/core/futures.html)/Promise 异步编程模式。

作为正统的 Java 类库，是不是应该做点什么，加强一下自身库的功能呢？

在 Java 8 中, 新增加了一个包含 50 个方法左右的类: `CompletableFuture`，实现了 `CompletionStage` 和 `Future` 接口，可以帮助我们简化异步编程的复杂性，提供了函数式编程的能力，可以通过回调的方式处理计算结果，并且提供了转换和组合 CompletableFuture 的方法。


## 创建 CompletableFuture 对象



### New 一个



```java
public CompletableFuture<String> ask() {
    final CompletableFuture<String> future = new CompletableFuture<>();
    //...
    return future;
}

```

这个 future 和 Callable 没有任何联系，没有线程池也不是异步工作。如果现在客户端代码调用 `ask().get()` 它将永远阻塞。直到执行了：

```java	
future.complete("42")
```

此时此刻所有客户端 `Future.get()` 将得到字符串的结果，并且完成的回调会立即生效。这对于想表达一个未来完成 的任务时是非常方便的，而且没有必要让计算任务在其他线程上执行。

`CompletableFuture.complete()` 只能调用一次，后续调用将被忽略。但也有一个后门叫做 `CompletableFuture.obtrudeValue(…)` 覆盖一个新Future之前的价值，请小心使用。

如果想传递一些异常，可以用 `CompletableFuture.completeExceptionally(ex)` (或者用 `obtrudeException(ex)`这样更强大的方法覆盖前面的异常)。 


CompletableFuture 类实现了 CompletionStage 和 Future 接口，所以你还是可以像以前一样通过阻塞或者轮询的方式获得结果，尽管这种方式不推荐使用。

```java
public T get()
public T get(long timeout, TimeUnit unit)
public T getNow(T valueIfAbsent)
public T join()
```

CompletableFuture 类中的 `join` 方法和 Future 接口中的 `get` 有相同的含义，并且也声明在
Future 接口中，它们唯一的不同是 `join` 不会抛出任何检测到的异常。



### 工厂方法

`CompletableFuture.completedFuture` 是一个静态辅助方法，用来返回一个已经计算好的 CompletableFuture。


```java
public static <U> CompletableFuture<U> completedFuture(U value)

```

而以下四个静态方法用来为一段异步执行的代码创建 CompletableFuture 对象：

```java
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```

以 Async 结尾并且没有指定 Executor 的方法会使用 `ForkJoinPool.commonPool()` 作为它的线程池执行异步代码。

`runAsync` 方法也好理解，它以 `Runnable` 函数式接口类型为参数，所以 `CompletableFuture` 的计算结果为空。

`supplyAsync` 方法以 `Supplier<U>` 函数式接口类型为参数，CompletableFuture 的计算结果类型为 `U`。


## 计算结果完成时的处理

> `whenComplete` 和 `exceptionally` 和 `handle`

当 CompletableFuture 的计算结果完成，或者抛出异常的时候，我们可以执行特定的 Action。主要是下面的方法：

```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
```


可以看到 Action 的类型是 `BiConsumer<? super T,? super Throwable>`，它可以处理正常的计算结果，或者异常情况。

方法不以 Async 结尾，意味着 Action 使用相同的线程执行，而 Async 可能会使用其它的线程去执行(如果使用相同的线程池，也可能会被同一个线程选中执行)。

注意这几个方法都会返回 CompletableFuture，当 Action 执行完毕后它的结果返回原始的 CompletableFuture 的计算结果或者返回异常。

`exceptionally` 方法返回一个新的 CompletableFuture，当原始的 CompletableFuture 抛出异常的时候，就会触发这个 CompletableFuture 的计算，调用 `function` 计算值，否则如果原始的 CompletableFuture 正常计算完后，这个新的 CompletableFuture 也计算完成，它的值和原始的 CompletableFuture 的计算的值相同。也就是这个 `exceptionally` 方法用来处理异常的情况。

```java

	/**
	 * 抛出异常是返回 -1， 否则返回 100
	 */
    @Test
    public void testEx() {

        final int x = new Random().nextInt(2);
        CompletableFuture<Integer> future= CompletableFuture.supplyAsync(
                () -> {
                    System.out.println("stmt = 1 / "+ x);
                    int i = 1/x;
                    return 100;
                })
                .exceptionally((t) ->  -1);

        try {
            Integer result = future.get();
            System.out.println("result ==> "+ result);
            waitSometime(2L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

        System.out.println("-- end --");

    }
```

下面一组方法虽然也返回 CompletableFuture 对象，但是对象的值和原来的 CompletableFuture 计算的值不同。当原先的 CompletableFuture 的值计算完成或者抛出异常的时候，会触发这个 CompletableFuture 对象的计算，结果由 BiFunction 参数计算而得。因此这组方法兼有 `whenComplete` 和转换的两个功能。



```java

public <U> CompletableFuture<U> handle(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T,Throwable,? extends U> fn)
public <U> CompletableFuture<U> handleAsync(BiFunction<? super T,Throwable,? extends U> fn, Executor executor)
```

同样，不以 Async 结尾的方法由原来的线程计算，以 Async 结尾的方法由默认的线程池 `ForkJoinPool.commonPool()` 或者指定的线程池 `executor` 运行。


## 转换

> `thenApply`


CompletableFuture 可以作为 monad （单子）和 functor （函子，起作用的东西）。由于回调风格的实现，我们不必因为等待一个计算完成而阻塞着调用线程，而是告诉CompletableFuture当计算完成的时候请执行某个function。而且我们还可以将这些操作串联起来，或者将CompletableFuture组合起来。

```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
```


这一组函数的功能是当原来的 CompletableFuture 计算完后，将结果传递给函数 `fn`，将 `fn` 的结果作为新的 CompletableFuture 计算结果。因此它的功能相当于将 `CompletableFuture<T>`转换成 `CompletableFuture<U>`。

这三个函数的区别和上面介绍的一样，不以 Async 结尾的方法由原来的线程计算，以 Async 结尾的方法由默认的线程池 `ForkJoinPool.commonPool()` 或者指定的线程池 `executor` 运行。Java 的 CompletableFuture 类总是遵循这样的原则，下面就不一一赘述了。


```java

CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    return 100;
});
CompletableFuture<String> f = future.thenApplyAsync(i -> i * 10)
		.thenApply(i -> i.toString());
System.out.println(f.get()); //"1000"

```

一个例子：

```java

    /**
     * thenApply 的功能相当于将 CompletableFuture<T> 转换成 CompletableFuture<U>
     */
    @Test
    public void test002() {

        CompletableFuture<String> future =
                CompletableFuture.supplyAsync(() -> "Hello")
                        .thenApply(input -> input + " world")
                        .thenApply(String::toUpperCase);

        CompletableFuture<Double> f2 =
                CompletableFuture.supplyAsync(() -> "10")
                        .thenApply(Integer::parseInt)
                        .thenApply(i -> i * 10.0D);
        try {
            String output = future.get();
            System.out.println(output); /* HELLO WORLD */
            Double d = f2.get();
            System.out.println(d); /* 100.0 */
        } catch (InterruptedException e) {
            e.printStackTrace();
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

    }
```

需要注意的是，这些转换并不是马上执行的，也不会阻塞，而是在前一个 stage 完成后继续执行。

它们与 `handle` 方法的区别在于 `handle` 方法会处理正常计算值和异常，因此它可以屏蔽异常，避免异常继续抛出。而 `thenApply` 方法只是用来处理正常值，因此一旦有异常就会抛出。


## 消费

前面的方法是当计算完成的时候，会生成新的计算结果(`thenApply`, `handle`)，或者返回同样的计算结果 `whenComplete`，CompletableFuture 还提供了一种处理结果的方法，只对结果执行 Action,而不返回新的计算值，因此计算值为 `Void`:

```java
public CompletableFuture<Void> thenAccept(Consumer<? super T> action)
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action)
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor)
```

看它的参数类型也就明白了，它们是函数式接口 Consumer，这个接口只有输入，没有返回值。

```java

    @Test
    public void testAccept() throws Exception {

        CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> {
            return 100;
        });
        CompletableFuture<Void> f =  future2.thenAcceptAsync(i -> {
            this.waitSometime(1L);
            System.out.println(i); // (2) 100
        });
        System.out.println("f is done ==> "+ f.isDone()); // (1) f is done ==> false
        System.out.println(f.get()); // (3) null
        System.out.println("f is done ==> "+ f.isDone()); // (4) f is done ==> true
        this.waitSometime(2L);
    }
```

`thenAcceptBoth` 以及相关方法提供了类似的功能，当两个 `CompletionStage`都正常完成计算的时候，就会执行提供的 `action`，它用来组合另外一个异步的结果。

`runAfterBoth` 是当两个 `CompletionStage` 都正常完成计算的时候，执行一个 `Runnable` ，这个 `Runnable` 并不使用计算的结果。


```java
public <U> CompletableFuture<Void> thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)
public <U> CompletableFuture<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action, Executor executor)
public     CompletableFuture<Void> runAfterBoth(CompletionStage<?> other,  Runnable action)
```

一个例子：

```java

    /**
     * thenAcceptBoth 和 thenCombin 类似
     * 只是返回的是 CompletableFuture<Void> 类型
     */
    @Test
    public void acceptBothTest() {

        CompletableFuture<String> f1 =
                CompletableFuture.supplyAsync(() -> "100");
        
        CompletableFuture<Integer> f2 =
                CompletableFuture.supplyAsync(() -> 10);

        CompletableFuture<Void> f3 =
                f1.thenAcceptBothAsync(f2, (s, i) -> System.out.println(Double.parseDouble(s) * i));
        try {
            f3.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
```

更彻底地，下面一组方法当计算完成的时候会执行一个 `Runnable`,与 `thenAccept` 不同，`Runnable` 并不使用 `CompletableFuture` 计算的结果。

```java
public CompletableFuture<Void> thenRun(Runnable action)
public CompletableFuture<Void> thenRunAsync(Runnable action)
public CompletableFuture<Void> thenRunAsync(Runnable action, Executor executor)
```


因此先前的 `CompletableFuture` 计算的结果被忽略了,这个方法返回 `CompletableFuture<Void>`类型的对象。


> 因此，你可以根据方法的参数的类型来加速你的记忆。`Runnable` 类型的参数会忽略计算的结果，`Consumer` 是纯消费计算结果，`BiConsumer` 会组合另外一个 `CompletionStage` 纯消费，`Function` 会对计算结果做转换，`BiFunction` 会组合另外一个 `CompletionStage` 的计算结果做转换。


## 组合 Compose

```java
public <U> CompletableFuture<U> thenCompose(Function<? super T,? extends CompletionStage<U>> fn)
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn)
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn, Executor executor)
```


这一组方法接受一个 Function 作为参数，这个 Function 的输入是当前的 CompletableFuture 的计算值，返回结果将是一个新的 CompletableFuture （将前一个结果作为下一个计算的参数，它们之间存在着先后顺序）。因此它的功能类似:

```
    A +--> B +---> C
```

记住，`thenCompose` 返回的对象并不是函数 `fn` 返回的对象，如果原来的 CompletableFuture 还没有计算出来，它就会生成一个新的组合后的 CompletableFuture。

```java


    /**
     * thenCompose 可以用于组合多个 CompletableFuture
     * 将前一个结果作为下一个计算的参数.
     * <p>
     * 它们之间存在着先后顺序.
     * </p>
     */
    @Test
    public void test003() {

        CompletableFuture<String> future =
                CompletableFuture.supplyAsync(() -> "hello")
                        .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " world"));

        CompletableFuture<Double> f2 =
                CompletableFuture.supplyAsync(() -> "10")
                        .thenCompose(s -> CompletableFuture.supplyAsync(() -> Double.parseDouble(s)))
                        .thenCompose(d -> CompletableFuture.supplyAsync(() -> d * 10));

        try {
            System.out.println(future.get()); // hello world
            System.out.println(f2.get()); // 100.0
        } catch (InterruptedException e) {
            e.printStackTrace();
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

```



```java
public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
public <U,V> CompletableFuture<V> 	thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)
```

这一组方法 `thenCombine` 用来复合另外一个` CompletionStage` 的结果。它的功能类似：

```
	A +
	  |
	  +------> C
	  +------^
	B +
```


两个 CompletionStage 是并行执行的，它们之间并没有先后依赖顺序，`other` 并不会等待先前的 `CompletableFuture` 执行完毕后再执行。

其实从功能上来讲,它们的功能更类似 `thenAcceptBoth`，只不过 `thenAcceptBoth` 是纯消费，它的函数参数没有返回值，而 `thenCombine` 的函数参数 `fn` 有返回值。

```java

    /**
     * 使用 thenCombine() 之后
     * f1 和 f2 之间是并行的.
     * 这一点和 thenCompose 方法不同.
     */
    @Test
    public void combinTest() {
        CompletableFuture<String> f1 =
                CompletableFuture.supplyAsync(() -> "100");
        CompletableFuture<Integer> f2 =
                CompletableFuture.supplyAsync(() -> 10);
        CompletableFuture<Double> f3 =
                f1.thenCombine(f2, (s, i) -> Double.parseDouble(s) * i);
        try {
            Double aDouble = f3.get();
            System.out.println(aDouble);
        } catch (InterruptedException e) {
            e.printStackTrace();
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }


```


## 任何一个 （Either）


`thenAcceptBoth` 和 `runAfterBoth` 是当两个 `CompletableFuture` 都计算完成，而我们下面要了解的方法是当任意一个 `CompletableFuture` 计算完成的时候就会执行。

```java
public CompletableFuture<Void> acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action, Executor executor)
public <U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, Function<? super T,U> fn)
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn)
public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn, Executor executor)
```


`acceptEither` 方法是当任意一个 `CompletionStage` 完成的时候，`action` 这个消费者就会被执行。这个方法返回 `CompletableFuture<Void>`。


`applyToEither` 方法是当任意一个 `CompletionStage` 完成的时候，`fn` 会被执行，它的返回值会当作新的 `CompletableFuture<U>` 的计算结果。

```java

    /**
     * Either 表示的是两个 CompletableFuture
     *
     * 当其中任何一个计算完成都会执行.
     *
     * 和它类似的是 applyToEither .
     */
    @Test
    public void eitherTest() {

        CompletableFuture<String> f1 =
                CompletableFuture.supplyAsync(() -> "10");
        CompletableFuture<String> f2 =
                CompletableFuture.supplyAsync(() -> "20");

        CompletableFuture<Void> f3 = f1.acceptEither(f2,
                System.out::println);
    }
```

## 辅助方法 `allOf` 和 `anyOf`


前面我们已经介绍了几个静态方法：`completedFuture`、`runAsync`、`supplyAsync`,下面介绍的这两个方法用来组合多个 CompletableFuture。

```java
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs)
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)
```

`allOf` 方法是当所有的 `CompletableFuture` 都执行完后执行计算。

`anyOf` 方法是当任意一个 `CompletableFuture` 执行完后就会执行计算，计算的结果相同。

下面的代码运行结果有时是100,有时是"abc"。但是 `anyOf` 和 `applyToEither` 不同。`anyOf` 接受任意多的 CompletableFuture 而 `applyToEither`只是判断两个 CompletableFuture；`anyOf` 返回值的计算结果是参数中其中一个 CompletableFuture 的计算结果，`applyToEither` 返回值的计算结果却是要经过 `fn` 处理的。当然还有静态方法的区别，线程池的选择等。

```java
Random rand = new Random();
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(10000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return 100;
});
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(10000 + rand.nextInt(1000));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "abc";
});
//CompletableFuture<Void> f =  CompletableFuture.allOf(future1,future2);
CompletableFuture<Object> f =  CompletableFuture.anyOf(future1,future2);
System.out.println(f.get());
```

## 更进一步

如果你用过 `Guava` 的 `Future` 类，你就会知道它的 `Futures` 辅助类提供了很多便利方法，用来处理多个 `Future`，而不像 `Java` 的 `CompletableFuture`，只提供了 `allOf`、`anyOf` 两个方法。 

比如有这样一个需求，将多个 `CompletableFuture` 组合成一个 `CompletableFuture`，这个组合后的 `CompletableFuture` 的计算结果是个 `List` ,它包含前面所有的 `CompletableFuture` 的计算结果，`guava` 的 `Futures.allAsList` 可以实现这样的功能，但是对于 java CompletableFuture，我们需要一些辅助方法：

```java
   public static <T> CompletableFuture<List<T>> sequence(List<CompletableFuture<T>> futures) {
       CompletableFuture<Void> allDoneFuture = CompletableFuture.allOf(futures.toArray(new CompletableFuture[futures.size()]));
       return allDoneFuture.thenApply(v -> futures.stream().map(CompletableFuture::join).collect(Collectors.<T>toList()));
   }

	public static <T> CompletableFuture<Stream<T>> sequence(Stream<CompletableFuture<T>> futures) {
       List<CompletableFuture<T>> futureList = futures.filter(f -> f != null).collect(Collectors.toList());
       return sequence(futureList);
   }
```


或者 Java Future 转 CompletableFuture:

```java
public static <T> CompletableFuture<T> toCompletable(Future<T> future, Executor executor) {
    return CompletableFuture.supplyAsync(() -> {
        try {
            return future.get();
        } catch (InterruptedException | ExecutionException e) {
            throw new RuntimeException(e);
        }
    }, executor);
}
```

github 有多个项目可以实现 Java CompletableFuture 与其它 Future (如 Guava ListenableFuture)之间的转换，如 spotify/futures-extra、future-converter、scala/scala-java8-compat 等。



## 使用定制的执行器

Brian Goetz建议，线程池大小与处理器的利用率之比可以使用下面的公式进行估算：

```
Nthreads = NCPU * UCPU * (1 + W/C)
```
其中：
- NCPU 是处理器的核的数目，可以通过 `Runtime.getRuntime().availableProcessors()`得到
- UCPU 是期望的 CPU 利用率（该值应该介于0和1之间）
- W/C 是等待时间与计算时间的比率

```java

private final Executor executor = Executors.newFixedThreadPool(Math.min(size(), 100),
	new ThreadFactory() {
		public Thread newThread(Runnable r) {
			Thread t = new Thread(4);
			t.setDaemon(true); // 不会阻止程序关停
			return t;
		}
	});

CompletableFuture.supplyAsync(() -> someWork(), executor);


```

使用 `Stream` 还是 `CompletableFutures`？

- 如果你进行的是计算密集型的操作，并且没有I/O，那么推荐使用 Stream 接口，因为实现简单，同时效率也可能是最高的（如果所有的线程都是计算密集型的，那就没有必要创建比处理器核数更多的线程）。
- 反之，如果你并行的工作单元还涉及等待I/O的操作（包括网络连接等待），那么使用 `CompletableFuture` 灵活性更好，你可以像前文讨论的那样，依据等待/计算，或者 W/C 的比率设定需要使用的线程数。这种情况不使用并行流的另一个原因是，处理流的流水线中如果发生 I/O 等待，流的延迟特性会让我们很难判断到底什么时候触发了等待。



参考： 

- Java 8 in Action 
- [Java CompletableFuture 详解](https://colobu.com/2016/02/29/Java-CompletableFuture/)
- [Java 8: Definitive guide to CompletableFuture](https://www.javacodegeeks.com/2013/05/java-8-definitive-guide-to-completablefuture.html)


If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
