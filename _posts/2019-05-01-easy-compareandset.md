---
title: 浅析 CompareAndSet(CAS)
key: 20190507
tags: java CAS CompareAndSet
---

最近无意接触了AtomicInteger类compareAndSet(从JDK5开始)，搜了搜相关资料，整理了一下。


<!--more-->

首先要说一下，AtomicInteger 类 `compareAndSet` 通过原子操作实现了 CAS 操作，最底层基于汇编语言实现。

简单说一下原子操作的概念，“原子”代表最小的单位，所以原子操作可以看做最小的执行单位，该操作在执行完毕前不会被任何其他任务或事件打断。

CAS 是 Compare And Set 的一个简称，如下理解：

1，已知当前内存里面的值 current 和预期要修改成的值 new 传入

2，内存中 AtomicInteger 对象地址对应的真实值(因为有可能别修改) real 与 current 对比，相等表示 real 未被修改过，是“安全”的，将 new 赋给 real 结束然后返回；不相等说明 real 已经被修改，结束并重新执行1直到修改成功



CAS 相比 Synchronized，避免了锁的使用，总体性能比 Synchronized 高很多.

compareAndSet 典型使用为计数，如 `i++`, `++`i,这里以 `i++` 为例：

```java

    @Test
    public void testAtomicInt() {

        AtomicInteger ai = new AtomicInteger(0);

        int i = ai.incrementAndGet();

        System.out.println("i = "+ i);


    }
```


compareAndSet方法实现：

JDK文档对该方法的说明如下：如果当前状态值等于预期值，则以原子方式将同步状态设置为给定的更新值。

```java
    /**
     * Atomically sets the value to the given updated value
     * if the current value {@code ==} the expected value.
     *
     * @param expect the expected value
     * @param update the new value
     * @return {@code true} if successful. False return indicates that
     * the actual value was not equal to the expected value.
     */
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```

这里解释一下 valueOffset 变量，首先 valueOffset 的初始化在 static 静态代码块里面，代表相对起始内存地址的字节相对偏移量：

```java

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }


```

在生成一个 AtomicInteger 对象后，可以看做生成了一段内存，对象中各个字段按一定顺序放在这段内存中，字段可能不是连续放置的， `unsafe.objectFieldOffset(Field f)` 这个方法准确地告诉我们 "value" 字段相对于 AtomicInteger 对象的起始内存地址的字节相对偏移量。

```java

    private volatile int value;

    /**
     * Creates a new AtomicInteger with the given initial value.
     *
     * @param initialValue the initial value
     */
    public AtomicInteger(int initialValue) {
        value = initialValue;
    }

    /**
     * Creates a new AtomicInteger with initial value {@code 0}.
     */
    public AtomicInteger() {
    }


```

value 是一个 volatile 变量，不同线程对这个变量进行操作时具有可见性，修改与写入操作都会存入主存中，并通知其他 CPU 中该变量缓存行无效，保证了每次读取都是最新的值


找到sun.misc.Unsafe.java：

```java

    /**
     * Atomically update Java variable to <tt>x</tt> if it is currently
     * holding <tt>expected</tt>.
     * @return <tt>true</tt> if successful
     */
    public final native boolean compareAndSwapInt(Object o, long offset,
                                                  int expected,
                                                  int x);
```


继续查找unsafe.cpp,(http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/4614a598dae1/src/share/vm/prims/unsafe.cpp)：

```cpp


UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))

  UnsafeWrapper("Unsafe_CompareAndSwapInt");

  oop p = JNIHandles::resolve(obj);

  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);

  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;

UNSAFE_END

```

实现主要方法为 Atomic::cmpxchg , 这个本地方法的最终实现在 openjdk 的如下位置： D:\java\openjdk\hotspot\src\os_cpu\windows_x86\vm\atomicwindowsx86.inline.hpp（对应于windows操作系统，X86处理器） 

```cpp
// Adding a lock prefix to an instruction on MP machine
// VC++ doesn't like the lock prefix to be on a single line
// so we can't insert a label after the lock prefix.
// By emitting a lock prefix, we can define a label after it.
#define LOCK_IF_MP(mp) __asm cmp mp, 0  \
                       __asm je L0      \
                       __asm _emit 0xF0 \
                       __asm L0:
 
inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  // alternative for InterlockedCompareExchange
  int mp = os::is_MP();
  __asm {
    mov edx, dest
    mov ecx, exchange_value
    mov eax, compare_value
    LOCK_IF_MP(mp)
    cmpxchg dword ptr [edx], ecx
  }
}

```

如上面源代码所示，用嵌入的汇编实现的, CPU 指令是 `cmpxchg` ，程序会根据当前处理器的类型来决定是否为 `cmpxchg` 指令添加 lock 前缀。如果程序是在多处理器上运行，就为 `cmpxchg` 指令加上 lock 前缀( `lock cmpxchg`).反之，如果程序是在单处理器上运行，就省略 lock 前缀(单处理器自身会维护单处理器内的顺序一致性，不需要 lock 前缀提供的内存屏障效果).

> lock前缀的作用说明：
> - 1 禁止该指令与之前和之后的读和写指令重排序,
> - 2 把写缓冲区中的所有数据刷新到内存中。


总的来说，Atomic 实现了高效无锁(底层还是用到排它锁，不过底层处理比 java 层处理要快很多)与线程安全( volatile 变量特性)，CAS 一般适用于计数；多线程编程也适用，多个线程执行 AtomicXXX 类下面的方法，当某个线程执行的时候具有排他性，在执行方法中不会被打断，直至当前线程完成才会执行其他的线程。



参考文章：

https://blog.csdn.net/u013404471/article/details/47297123

http://www.infoq.com/cn/articles/java-memory-model-5

http://hllvm.group.iteye.com/group/topic/37940

http://www.cnblogs.com/dolphin0520/p/3920373.html




<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
