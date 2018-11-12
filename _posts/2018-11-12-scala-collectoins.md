---
title: Scala 的集合
key: 20181112
tags: scala
---

Scala 的集合类博大精深，与 Java 的集合类相比大相径庭，这会减慢 Java 开发人员转向 Scala 的学习速度。

<!--more-->

### 几个重要的概念

- 谓词
- 匿名函数
- 隐式循环 (implied loops)


## 理解集合的层级结构

以 `Vector` 为例，`Vector` 继承了一大堆的特质，但是 `Vector` 的使用还是很简单的：

```scala
val v = Vector(1, 2, 3)
v.sum            // 6
v.filter(_ > 1) // Vector(2, 3)
v.map(_ * 2)    // Vector(2, 4, 6)
```

宏观地讲，Scala 的集合类从 `Traversable` 和 `Iterable` 这些特质开始，扩展到三个主要的类别： `Seq`, `Set`, `Map` 。序列进一步分支成索引序列和线性序列：

```
    | Traversable |
    | Iterable    |
  | Seq |  | Set |  | Map |
| INdexedSeq | | LLinearSeq |
```

`Traversable` 特质遍历整个集合，Scaladoc 说它实现了一个就 `foreach` 方法而言的所有集合的通用方法，这样就可以反复遍历集合。

`Iterable`  特质定义了一个迭代器，可以一次性循环一个集合的元素，当使用迭代器时，集合只可以被循环一次，因为在迭代的过程中每个元素都被改变了。

#### 序列 (Sequences)

Scala 包含了大量的序列。

```
Seq --> IndexedSeq, Buffer, LinearSeq

IndexedSeq -> StringBuilder, String, ArrayBuffer
           -> Array, Range, Vector

Buffer -> ArrayBuffer, 
       -> ListBuffer

LinearSeq -> List, LinkedList, MutableList
          -> Queue, Stack, Stream

```

序列分支为两个大类：索引序列和线性序列（链表）。索引序列 (`IndexSeq`) 意味着随机存取是高效的，比如读取数组的元素： `arr(5000)` 。默认情况下（Scala 2.10.x) 版本中创建 `Vector` 时会认为是一个所以你序列：

```scala
scala> val x = IndexedSeq(1, 2, 3)
x: IndexedSeq[Int] = Vector(1, 2, 3)
```

线性序列 (`LinearSeq`) 说明集合可以很方便地被分为头尾两部分，并且用 `head`, `tail`, `isEmpty` 方法是很常见的。当创建一个 `LinearSeq` 时会创建一个 `List` （列表）：

```scala
scala> val seq = scala.collection.immutable.LinearSeq(1, 2, 3)
seq: scala.collection.immutable.LinearSeq[Int] = List(1, 2, 3)
```

#### Map 



---

traversable : /'trævɝsəbl/ adj. 能越过的，能横过的；可否认的，可反驳的；

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
