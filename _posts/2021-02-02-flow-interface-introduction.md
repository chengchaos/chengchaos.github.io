---
title: Flow 中各个接口组件功能介绍
key: 2021-02-02
tags: java rxjava 
---

As show below.

<!--more-->

Publisher

通过此接口来发布一个元素给有需求的消费之。

Subscriber

每一个订阅者（消费者）从 Publisher 那里获取所需的元素进行消费。

Subscription

Subscription 属于 Subscriber 和 Publisher 进行交流的中间人，在订阅者请求的时候需要用到它（请求消费和不要元素消费时）

Processor

Processor 同时属于 Subscriber 和 Publisher 这两个角色。

`Flow.Publisher<T>` 接口有 1 方法：

```java

/**
 * Subscribers 接收的 items 的生产者（也包括其他控制消息）。
 * 每个当前的 Flow.Subscriber （通过 onNext 方法）按一样的顺序接收一样的 items，
 * 除非丢弃或者遇到错误。
 * 如果 Publisher 遇到错误也不打算下发给 Subscriber，则 Subscriber 接收到 onError,
 * 然而并不能接收跟多的消息（and then receives no further messages)。
 * 否则，当知道不再向订阅者发送消息时，订阅者将接收到 onComplete。
 * Publisher 要确保 Subscriber 调用的么个订阅都是严格按照 happens-before 顺序 
 */
@FunctionalInterface
public static interface Publisher<T> {
    // 添加一个 Subscriber
    // 如果已经 subscribed 或者尝试订阅失败则调用 Subscriber 的 onError 方法抛出一个 IllegalStateException。
    // 否则，传入一个新的  Flow.Subscription 对象调用 Subscriber 的 onSubscribe 方法，
    // Subscriber 通过调用其所属的 Flow.SubScription 的 request 方法来获取元素。
    // 也可以调用它的 cancel 方法来解除一个订阅。
    public void subscriber(Subscriber<? super T> subscriber);
}
```

`Flow.Subscriber<T>` 接口有 4 个方法：

```java

public final class Flow {


    public static interface Subscriber<T> {

        /**
        * 这个方法先于（prior）给定的 Subscription 调用 Subscriber 其他任何 Subscriber 的方法。
        * 如果这个方法抛出异常，不能保证结果的行为。可能的原因是订阅没有被建立，或者已经被取消。
        */
        public void onSubscribe(Subscription subscription);

        /**
        * 传入 Subscription 的下一个元素。
        * 如果有异常，同上。
        */
        public void onNext(T item);
        
        /**
        * 这个方法发生在 Publisher 或者 Subscription 遇到（encountered）一个
        * 不可恢复的错误，之上（upon）。
        * 当 Publisher 或者 Subscription 遇到了不可恢复的错误时，调用此方法。
        * 然后 Subscription 就不能再调用 Subscriber 的其他方法了。
        */
        public void onError(Throwable throwable);

        /**
        * 调用这个方法后， Subscription 就不能再调用 Subscriber 的其他方法了。
        */
        public void onComplete();
    }
}

```

`Flow. Subscription` 接口有两个方法。

```java
public static interface Subscription {

    /**
     * 将给定的 n 个 Item 添加到该订阅的未满足的需求中。
     * 如果 n <= 0 则 Subscriber 将收到一个带有 IllegalArgumentException 参数的 onError 信号。
     * 否则，Subscriber 将接收最多 n 个额外的 noNext 调用。
     */
    public void request(long n);

    /**
     * 使订阅者(最终)停止接收消息。
     * 实现是尽最大的努力 —— 在调用此方法之后可能会收到额外的消息。
     * 一个取消的订阅不需要收到 onComplete 或 onError 信号。
     */
    public void cancel();
}
```

EOF

---

Power by TeXt.
