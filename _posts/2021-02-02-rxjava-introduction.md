---
title: RxJava 介绍
key: 2021-02-02
tags: java rxjava 
---

As shown below. （shown 是 show 的过去分词）

<!--more-->

ReactiveX是一套异步编程模型，可以使用对偶性来推导。对偶性即导致相同的物理结果，而表面上不同的理论之间的对应。

Reactive Revolution

ReactiveX is more than an API, it's an idea and a breakthrough in programming. It has inspired several other APIs, frameworks, and even programming languages.

```java

public interface Iterable<T> {
    Iterator<T> getIterator(void);
}

public interface Iterator<T> {
    boolean getHasNext(void);

    T|error getNext(void);
}


public interface Iterable<T> {
    void setIterator(Iterator<T>);
}

public interface Iterable<T> {
    void setHasNext(boolean);
    void setNext(T|error);
}


public interface Iterable<T> {
    void setHasNext(false);
    void setNext(T);
    void setError(error);
}

public interface Observable<T> {
    void subscribe(Observer<? super T> observer);
}

public interface Observer<T> {
    void onNext(T value);
    void onError(Throwable e);
    void onComplete();
}

```

每个操作符类都是一个继承了 Observable 的实现类，然后又有一个内部类实现了 Observer 接口。 

在 RxJava 中，每个操作符内部实现了一整套基于 Push 的接口体系，从而可以在整个调用链中发挥承上启下的作用。

## 实现异步

线程操作符会将上游传入的数据缓存起来，然后开启一个线程，在线程中传递出去。

```java

public class ObserverOn<T> extends Observable<T> {

    // 指向上游传入的 Observable
    private Observable observable;

    private Scheduler scheduler;

    public ObserverOn(Observable observable, Scheduler scheduler) {
        this.observable = observable;
        this.scheduler = scheduler;
    }

    /**
     * 当真正的订阅发生时，这个方法会被调用，传入下游的 Observer。
     */
    @Override
    protected void subscribeActual(Observer observer) {
        // 先获得一个线程操作类，Work 封装了具体的线程
        Scheduler.Worker worker = scheduler.createWorker();
        // 创建一个内部类
        ObserverOnInner inner = ObserverOnInner(observer, worker);
        // 将整个内部类传递给上游
        // 上游的 Observable 获得内部类的引用
        // 当上游有数据时，会将数据传递给整个内部类。
        observable.subscribe(inner);
    }

    static final class ObserverOnInner<T> implements Observer<T>, Runnable, Disposable {
        // 持有下游的 observer
        private Observer<T> actual;

        // 切换的线程
        private Scheduler.Worker curWork;

        // 缓存的队列
        private Queue<T> cache;

        private boolean hasCalled = false;
        private boolean cancelled = false;
        private boolean done = false;
        private Throwable error = null;

        public ObserverOnInner(Observer<T> observer, Scheduler.Worker worker) {
            this.curWork = worker;
            actual = observer;
            cache = new ConcurrentLinkQueue<>();
        }

        @Override
        public void onSubscribe(Disposable d) {
            actual.onSubscribe(this);
        }

        /**
         * 上游的 Observable 有数据时，会转入数据进来。
         */
        @Override
        public void onNext(T value) {
            // 因为需要切换线程，因此暂时把数据缓存起来
            cache.offer(value)
            // 启动新线程
            schedule();
        }

        @Override
        public void onError(Throwable e) {
            // 上游发生错误时的处理
            error = e;
            done = true;
            schedule();
        }

        @Override
        public void onComplete() {
            // 完成的处理
            done = true;
            schedule();
        }

        void schedule() {
            // 这个方法只需要调用一次，因为只需要拉起一次线程
            if (!hasCalled) {
                hasCalled = true;
                // curWork 背后的线程会执行这个 runnable
                curWork.schedle(this);
            }
        }

        // 新线程启动后，会执行这个
        @Override
        public void run() {
            // 此时，代码已经在新得线程环境中执行了。
            for(;;) {
                if (cancelled) {
                    return;
                }
                // onError(), onComplete() 方法的处理
                if (done) {
                    if (error != null) {
                        actual.onError(error);
                        return;
                    }
                    actual.onComplete();
                    return;
                }
                T t = cache.poll();
                if (t != null) {
                    // 将数据传递给下游的 Observer
                    actual.onNext(t);
                }
            }
        }
        // 这个接口用于取消订阅
        @Override
        public void dispose() {
            cancelled = true;
        }
    }
}
```



EOF

---

Power by TeXt.
