---
title: Spring MVC 中的异步支持
key: 2023-12-28
tags: Spring Cloud Asynchronized
---

不管未来会怎样，眼下 Java 开发还是我的饭碗，在有时间的时候，整理一下这些内容免得忘记。

<!--more-->

Spring 从 版本 3 开始提供了 `@Async`注解，该注解可以标注在方法上，在 spring-boot 的配置类中增加注解 `@EnableAsnyc` 可以开启 Spring 的异步方法执行能力支持。

## 0x01 Spring 自带的 TaskExecutor

- `SimpleAsyncTaskExecutor`: 不是真的线程池，这个类不重用线程，默认每次调用都会创建一个新的线程。
- `SyncTaskExecutor`: 这个类没有实现异步调用，只是一个同步操作。只适用于不需要多线程的地方。
- `ConcurrentTaskExecutor：` Executor 的适配类，不推荐使用。如果 `ThreadPoolTaskExecutor` 不满足要求时，才用考虑使用这个类。
- `SimpleThreadPoolTaskExecutor：` 是 Quartz 的 SimpleThreadPool 的类。线程池同时被 quartz 和非 quartz 使用，才需要使用此类。
- `ThreadPoolTaskExecutor` ：最常使用，推荐。 其实质是对 `java.util.concurrent.ThreadPoolExecutor` 的包装。

### 无返回值调用

```java
    /**
     * 带参数的异步调用 异步方法可以传入参数
     * 对于返回值是 void，异常会被 AsyncUncaughtExceptionHandler 处理掉
     */
    @Async
    public void asyncInvokeWithException(String s) {
        log.info("asyncInvokeWithParameter, parementer={}", s);
        throw new IllegalArgumentException(s);
    }
```

### 有返回值 Future 调用

```java
    /**
     * 异常调用返回 Future
     * 对于返回值是 Future，不会被 AsyncUncaughtExceptionHandler 处理，需要我们在方法中捕获异常并处理
     * 或者在调用方在调用 Futrue.get 时捕获异常进行处理
     */
    @Async
    public Future<String> asyncInvokeReturnFuture(int i) {
        log.info("asyncInvokeReturnFuture, parementer={}", i);
        Future<String> future;
        try {
            Thread.sleep(1000 * 1);
            future = new AsyncResult<String>("success:" + i);
            throw new IllegalArgumentException("a");
        } catch (InterruptedException e) {
            future = new AsyncResult<String>("error");
        } catch(IllegalArgumentException e){
            future = new AsyncResult<String>("error-IllegalArgumentException");
        }
        return future;
    }
```

### 有返回值CompletableFuture调用

`CompletableFuture` 并不使用 `@Async` 注解，可达到调用系统线程池处理业务的功能。

JDK5 新增了 Future 接口，用于描述一个异步计算的结果。虽然 Future 以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的 CPU 资源，而且也不能及时地得到计算结果。

CompletionStage代表异步计算过程中的某一个阶段，一个阶段完成以后可能会触发另外一个阶段

一个阶段的计算执行可以是一个 Function，Consumer 或者 Runnable。比如：

```java
stage.thenApply(x -> square(x))
    .thenAccept(x -> System.out.print(x))
    .thenRun(() -> System.out.println())
```

一个阶段的执行可能是被单个阶段的完成触发，也可能是由多个阶段一起触发

在 Java8 中，CompletableFuture 提供了非常强大的 Future 的扩展功能，可以帮助我们简化异步编程的复杂性，并且提供了函数式编程的能力，可以通过回调的方式处理计算结果，也提供了转换和组合 CompletableFuture 的方法。

它可能代表一个明确完成的 Future，也有可能代表一个完成阶段（ CompletionStage ），它支持在计算完成以后触发一些函数或执行某些动作。

它实现了Future和CompletionStage接口

```java
    /**
     * 数据查询线程池
     */
    private static final ThreadPoolExecutor SELECT_POOL_EXECUTOR = new ThreadPoolExecutor(10, 
            20, 
            5000,
            TimeUnit.MILLISECONDS, 
            new LinkedBlockingQueue<>(1024), 
            new ThreadFactoryBuilder().setNameFormat("selectThreadPoolExecutor-%d").build());

// tradeMapper.countTradeLog(tradeSearchBean)方法表示，获取数量，返回值为int
 // 获取总条数
        CompletableFuture<Integer> countFuture = CompletableFuture
                .supplyAsync(() -> tradeMapper.countTradeLog(tradeSearchBean), SELECT_POOL_EXECUTOR);
// 同步阻塞
    CompletableFuture.allOf(countFuture).join();
// 获取结果
 int count = countFuture.get();
```

### 默认线程池的弊端

在线程池应用中，参考阿里巴巴java开发规范：

> 线程池不允许使用 Executors 去创建，不允许使用系统默认的线程池，推荐通过 ThreadPoolExecutor的方式，这样的处理方式让开发的工程师更加明确线程池的运行规则，规避资源耗尽的风险。

Executors各个方法的弊端：

- newFixedThreadPool 和 newSingleThreadExecutor：主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。
- newCachedThreadPool 和 newScheduledThreadPool：要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。

@Async 默认异步配置使用的是 SimpleAsyncTaskExecutor，该线程池默认来一个任务创建一个线程，若系统中不断的创建线程，最终会导致系统占用内存过高，引发OutOfMemoryError错误。

针对线程创建问题，SimpleAsyncTaskExecutor 提供了限流机制，通过 concurrencyLimit 属性来控制开关，当 concurrencyLimit>=0 时开启限流机制，默认关闭限流机制即concurrencyLimit=-1，当关闭情况下，会不断创建新的线程来处理任务。基于默认配置，SimpleAsyncTaskExecutor 并不是严格意义的线程池，达不到线程复用的功能。

## 0x02 自定义线程池

自定义线程池，可对系统中线程池更加细粒度的控制，方便调整线程池大小配置，线程执行异常控制和处理。在设置系统自定义线程池代替默认线程池时，虽可通过多种模式设置，但替换默认线程池最终产生的线程池有且只能设置一个（不能设置多个类继承AsyncConfigurer）。自定义线程池有如下模式：

- 重新实现接口 AsyncConfigurer
- 继承 AsyncConfigurerSupport
- 配置由自定义的 TaskExecutor 替代内置的任务执行器

通过查看 Spring 源码关于 @Async 的默认调用规则，会优先查询源码中实现 AsyncConfigurer 这个接口的类，实现这个接口的类为 AsyncConfigurerSupport。但默认配置的线程池和异步处理方法均为空，所以，无论是继承或者重新实现接口，都需指定一个线程池。且重新实现 public Executor getAsyncExecutor()方法。

### 实现接口AsyncConfigurer

```java
    @Configuration
    public class AsyncConfiguration implements AsyncConfigurer {
    
        @Bean("kingAsyncExecutor")
        public ThreadPoolTaskExecutor executor() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            int corePoolSize = 10;
            executor.setCorePoolSize(corePoolSize);
            int maxPoolSize = 50;
            executor.setMaxPoolSize(maxPoolSize);
            int queueCapacity = 10;
            executor.setQueueCapacity(queueCapacity);
            executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
            String threadNamePrefix = "kingDeeAsyncExecutor-";
            executor.setThreadNamePrefix(threadNamePrefix);
            executor.setWaitForTasksToCompleteOnShutdown(true);
            // 使用自定义的跨线程的请求级别线程工厂类19         int awaitTerminationSeconds = 5;
            executor.setAwaitTerminationSeconds(awaitTerminationSeconds);
            executor.initialize();
            return executor;
        }

        @Override
        public Executor getAsyncExecutor() {
            return executor();
        }

        @Override
        public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
            return (ex, method, params) -> ErrorLogger.getInstance().log(String.format("执行异步任务'%s'", method), ex);
        }
}
```

### 继承AsyncConfigurerSupport

```java
@Configuration  
@EnableAsync  
class SpringAsyncConfigurer extends AsyncConfigurerSupport {  
  
    @Bean  
    public ThreadPoolTaskExecutor asyncExecutor() {  
        ThreadPoolTaskExecutor threadPool = new ThreadPoolTaskExecutor();  
        threadPool.setCorePoolSize(3);  
        threadPool.setMaxPoolSize(3);  
        threadPool.setWaitForTasksToCompleteOnShutdown(true);  
        threadPool.setAwaitTerminationSeconds(60 * 15);  
        return threadPool;  
    }  
  
    @Override  
    public Executor getAsyncExecutor() {  
        return asyncExecutor;  
}  

    @Override  
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) -> ErrorLogger.getInstance().log(String.format("执行异步任务'%s'", method), ex);
    }
}
```

### 配置自定义的TaskExecutor

由于 AsyncConfigurer 的默认线程池在源码中为空，Spring 通过 `beanFactory.getBean(TaskExecutor.class)` 先查看是否有线程池，未配置时，又通过 `beanFactory.getBean(DEFAULT_TASK_EXECUTOR_BEAN_NAME, Executor.class)` 查询是否存在默认名称为 `TaskExecutor` 的线程池。所以可在项目中，定义名称为 `TaskExecutor` 的 bean 生成一个默认线程池。也可不指定线程池的名称，申明一个线程池，本身底层是基于 `TaskExecutor.class` 便可。

比如：

```
Executor.class: 

ThreadPoolExecutorAdapter -> 
    ThreadPoolExecutor -> 
        AbstractExecutorService -> 
            ExecutorService -> Executor
```

（这样的模式，最终底层为 Executor.class，在替换默认的线程池时，需设置默认的线程池名称为 `TaskExecutor`）
 
```
TaskExecutor.class: 

ThreadPoolTaskExecutor -> 
    SchedulingTaskExecutor -> 
        AsyncTaskExecutor -> TaskExecutor
```
（这样的模式，最终底层为 TaskExecutor.class，在替换默认的线程池时，可不指定线程池名称。）

```java
    @EnableAsync
    @Configuration
    public class TaskPoolConfig {
        @Bean(name = AsyncExecutionAspectSupport.DEFAULT_TASK_EXECUTOR_BEAN_NAME)
        public Executor taskExecutor() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
             //核心线程池大小
            executor.setCorePoolSize(10);
            //最大线程数
            executor.setMaxPoolSize(20);
            //队列容量
            executor.setQueueCapacity(200);
            //活跃时间
            executor.setKeepAliveSeconds(60);
            //线程名字前缀
            executor.setThreadNamePrefix("taskExecutor-");
            executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
            return executor;
        }
      
        @Bean(name = "new_task")
        public Executor taskExecutor() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            //核心线程池大小
            executor.setCorePoolSize(10);
            //最大线程数
            executor.setMaxPoolSize(20);
            //队列容量
            executor.setQueueCapacity(200);
            //活跃时间
            executor.setKeepAliveSeconds(60);
            //线程名字前缀
            executor.setThreadNamePrefix("taskExecutor-");
            executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
            return executor;
        }
    }
```


### 多个线程池

`@Async` 注解，使用系统默认或者自定义的线程池（代替默认线程池）。可在项目中设置多个线程池，在异步调用时，指明需要调用的线程池名称，如 `@Async("new_task")`。


## 0x02 Service 层 @Async 

1， 配置类。

```java
// 基于Java配置的启用方式：
@Configuration
@EnableAsync
public class AsyncConfig {

}

// Spring boot
@EnableAsync
@EnableTransactionManagement
public class SettlementApplication {
    public static void main(String[] args) {
        SpringApplication.run(SettlementApplication.class, args);
    }
}
```

默认情况下， spring 会查找相关的线程池的定义， 要么时在 context 中唯一定义的`org.springframework.core.task.TaskExecutor` bean 或者命名为 ’taskExecutor‘ 的 `java.util.concurrent.Executor` bean, 如果两个都没有找到， 将使用一个 `org.springframework.core.task.SimpleAsyncTaskExecutor` 处理异步方法的调用，Besides, 返回类型为 `void` 的带注解的方法不能讲任何异常传回给调用者。默认情况下，此列未捕获的异常只能记录在 log 里。

> 以上参考 EnableAsync 源码

2，在 Service 类的方法上增加 `@Async` 注解

3，修改方法的返回值，例如我们返回一个 `Future` 包装的业务数据

```java
@Service
public class OldServices {

    @Async
    public Future<String> getName() {
        // return new AsyncResult<>("chengchao");
        return AsyncResult.forValue("chengchao");
    }
}
```
就像 `@EnableAsync` 源码中的注释说明的那样，如果不定义一个 TaskExecutor，默认会使用 SimpleAsyncTaskExecutor 来处理，不过，SimpleAsyncTaskExecutor 类不会共享任何线程，每次调用都创建一个新的线程去执行， 并且默认情况下并发线程数量时无限的。

> `TaskExecutor` implementation that fires up a new Thread for each task, executing it asynchronously.
> Supports limiting concurrent threads through the "concurrencyLimit" bean property. By default, the number of concurrent threads is unlimited.
> **NOTE**: This implementation does not reuse threads! Consider a thread-pooling TaskExecutor implementation instead, in particular for executing a large number of short-lived tasks.

所以我们通常要定义一个 TaskExecutor

4, 自定义 Executor

```java
@Configuration
@EnableAsync
public class AsyncConfig extends AsyncConfigurerSupport {

    /**
     * 在使用这个 Executor 的方法上添加 {@code @Async("线程名称")} 就
     * 可以异步执行。
     * <br />
     * But, 如果配置类继承了 AsyncConfigurerSupport 并复写了
     * {@code getAsyncExecutor()} 方法，并在方法中返回我们自定义的 executor
     * 就会在全局的 Async 注解中使用我们自定以的 Executor。
     * <br />
     * 通常异步方法都会返回一个 {@code Future<原来的数据类型>}
     * <br />
     * Controller 中使用 {@code DeferrerResult<原来的数据类型>} 进行返回
     * @return Executor
     */
    @Bean(name="asyncExecutor")
    public Executor asyncExecutor() {
        // 默认 Spring 框架使用 SimpleAsyncTaskExecutor
        // 但是它不会复用任何线程。
        //
        // 因此我们通常使用 ThreadPoolTaskExecutor
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.setThreadNamePrefix("Async-1-");
        executor.setThreadGroupName("AsyncMvc-");
        executor.setAwaitTerminationSeconds(5);
        executor.initialize();

        return executor;

    }

    @Override
    public Executor getAsyncExecutor() {
        return this.asyncExecutor();
    }
}
```

## 0x02 Controller 层

Controller 层返回一步结果包装的业务数据就可以，通常有以下几种：

1， 返回 `DeferredResult<R>`

```java

    @GetMapping("/v1/name/1")
    public DeferredResult<String> getName1() {

        DeferredResult<String> result = new DeferredResult<>();
        ForkJoinPool.commonPool().submit(() -> {
            String name = null;
            try {
                name = this.oldServices.getName().get();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } catch (ExecutionException e) {
                LOGGER.error(e.getMessage(), e);
            }
            result.setResult(name);
        });
        return result;
    }
```

2， 返回 `Callable<R>`

```java
    @GetMapping("/v1/name/2")
    public Callable<String> getName2() {
        return () -> this.oldServices.getName().get();
    }
```

1， 返回 `WebAsyncTask<R>`

```java
    @GetMapping("/v1/name/3")
    public WebAsyncTask<String> getName3() {
        Callable<String> c =  () -> this.oldServices.getName().get();
        return new WebAsyncTask<>(c);
    }
```

## 0x03 限制

使用 `@Async` 注解的方法。

## 参考和抄袭:

- [Spring使用@Async注解](https://www.cnblogs.com/wlandwl/p/async.html)


