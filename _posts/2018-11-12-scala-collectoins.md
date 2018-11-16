---
title: Scala 的集合
key: 20181112
tags: scala
---

本文是我学习[《Scala 编程实战》 （Alexander，A. 著）]( https://www.amazon.cn/dp/B06X9SDVB8/ref=sr_1_1?ie=UTF8&qid=1542361298&sr=8-1&keywords=Scala+%E7%BC%96%E7%A8%8B%E5%AE%9E%E6%88%98 )的读书笔记。

![Scala 编程实战 封面]( https://images-cn.ssl-images-amazon.com/images/I/417LU7wGg%2BL.jpg )
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

### 序列 (Sequences)

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

### Map 

和 Java 中的 Map 类似，下面是常见的 Map 类图：

```
Map ->

HashMap, WeakHashMap, SortedMap, TreeMap, LinkedHashMap, ListMap
```

当需要一个简单的不可变的 `map` 时，可以新建一个不需要 import：

```scala
scala> val m = Map(1 -> "a", 2 -> "b")
m: scala.collection.immutable.Map[Int, java.lang.String] = Map(1 -> a, 2 -> b)

```


可变的 `map` 默认不在开间范围，所以必须引入它（或者指定全路径）来使用它：

```scala
scala> val m = collection.mutable.Map(1 -> "a", 2 -> "b")
m: scala.collection.mutable.Map[Int, String] = Map(2 -> b, 1 -> a)
```



### Set

就像 Java 中的 Set， Scala 中的 Set 是一个没有重复元素的集合。

```scala
Set ->
    BitSet, HashSet, ListSet, SortedSet
    SortedSet -> TreeSet

```


不可变的 Set ：

```scala
scala> val set = Set(1, 2, 3)
set: scala.collection.immutable.Set[Int] = Set(1, 2)

```


可变的 Set ：

```scala
scala> val s = collection.mutable.Set(1, 2, 3)
s: scala.collection.mutable.Set[Int] = Set(1, 2, 3)
```


### 更多的集合类

还有很多集合特质和类，包括 `Stream`, `Quere`, `Stack`, `Range` 。都可以在集合类上创建视图（就像一个数据库视图），用迭代器，以及使用 `Option`, `Some` 和 `None` 


### 严格（strict）和惰性（lazy）集合


## 选择一个集合类

常见的问题主要有三种集合类可供选择：

- `Sequence`
- `Map`
- `Set`

`sequence` 是一种线性元素的集合，可能会是索引或者线性的（链表）。`map` 是包含键值对的集合，就像 Java 的 Map，`Set` 是包含无重复元素的集合。

除了这三个主要的集合类之外，还有其他有用的集合类型，如 `Stack`, `Queue` 和 `Range`。还有一些用起来像集合的类，如元组、枚举、`Option`/`Some`/`None` 以及 `Try/Success/Failure` 类。


### 选择 `Sequence`

当使用 sequence 是，有两个主要的选择：

- 这个序列是否应该被索引（就像一个数组），允许快速访问某些元素，或者是否应该用链表实现？
- 想要可变的还是不可变的集合？

*Scala 通用的序列集合类*

| - | 比可变 (Immutable) | 可变 (Mutable) |
| ---- | ---- | ---- |
| 索引 (Indexed) | Vector | ArrayBuffer |
| 线性链表 (Linked Lists) | List | ListBuffer |


通过这个表，如果想要一个不可变的索引集合，那么就一般会使用 `Vector` ; 如果想好可变索引集合，就使用 `ArrayBuffer` （等等）。

然而这只是通用的推荐的做法，此外还有很多选择。

*主要的不可变序列集合类*

| - | 索引序列 | 线性序列 | 描述 |
| ---- | ---- | ---- | ---- |
| List | - | √ | 一个单链表，适于拆分头和剩余列联表的递归算法 |
| Queue | - | √ | 先进先出数据结构 |
| Range | √ | - | 数据值范围 |
| Stack | - | √ | 先进后厨数据结构 |
| Stream | - | √ | 与链表相似，但是延迟并且持久，适用于大型或无限序列，与 Haskell 的链表类似 |
| String | √ | - | 可以被当做一个不可变的，索引的字符序列 |
| Vector | √ | - | “定位”不可变，可索引的序列，Scaladoc 这样描述它，“处理 Split 和 join 非常有效率的一组嵌套数组实现” |


* 主要的可变序列集合类*

| - | 索引序列 | 线性序列 | 描述 |
| ---- | ---- | ---- | ---- |
| Array | √ | - | 依靠于 Java的数组，其中元素是可变的，单不能改变大小 |
| ArrayBuffer | √ | - | 一个可变的序列集合的“定位”类，成本是常熟 |
| ArrayStack | √ | - | 先进后出的数据结构，在性能比较重要是 Stack 更好 |
| DoubleLinkedList | - | √ | 像一个单链表，但有一个 `prev` 方法，文档说“额外的连接让删除元素变得非常快。” |
| LinkedList | - | √ | 一个可变的单链表 |
| ListBuffer | - | √ | 像 ArrayBuffer，单依靠链表。文档说：“如果想把 buffer 转成 list，用 ListBuffer 而不是 ArrayBuffer。”前插后插开销都是常熟，其他的大部分的操作都是线性的。 |
| MutableList | - | √ | 一个可变的，单链表，后插开销是常数 |
| Stack | - | √ | 后进先出的数据结构（文档建议 ArrayStack 的效率稍微好些。） |
| StringBuilder | √ | - | 像在循环里构建字符串，如 Java 的 StringBuilder | 


*在 API 中常用的特质*

| 特质 | 描述 |
| ---- | ---- |
| IndexedSeq | 意味着随机访问元素是有效的 |
| LinearSeq | 意味着线性访问元素是有效的 |
| Seq | 在不需要支出序列是索引或者线性时使用 |


当然如果所有返回的集合可以非常的通用，那么可以把他们定义为 `Iterable` 或者 `Traversable`。这等同于在 Java 里返回 Collection 的做法。

也可以在 IDE 中通过查看方法的返回类型了解更多。例如，在 Eclipse 中创建一个新的 Vector 时，然后查看 Vector 实例的可用方法，看到方法返回的类型，有 `GenSeqLike`, `IndexedSeqLike`, `IterableLike`, `TraversableLike` 和 `TraversableOne` 。没有必要指明方法返回的类型 —— 当然不是在开始就指定，但通常识别出真正返回的东西的意图是个很好的实践，所以一旦习惯了就可以声明更具体的类型了。


### 选择 `map`

用 map 比用 sequence 更简单。有基本可变和不可变的 map 类，`SorteedMap` 特质用于将元素按键排序， `LinkedHashMap` 用于按插入顺序存储元素，还有一些其他的用于特殊用途的 map 类。


*常用的 map，包括可变和不可变的版本*

| - | 不可变 | 可变 | 描述 |
| ---- | ---- | ---- | ---- |
| HashMap | √ | √ | 不可变版本用 `hash trie`(哈希线索) 实现，可变版本用“哈希表”实现 |
| LinkedHashMap | - | √ | “用哈希表实现可变 map”，元素按插入顺序返回 |
| ListMap | √ | √ | 用链表数据结构实现的 map。元素按插入的相反顺序返回，因为每次插入的元素都放在 head |
| Map | √ | √ | 基础的 map，有可变和不可变的实现 |
| SortedMap | √ | - | 按序存键的一个基本特质。（当前用 SortedMap 创建一个变量返回 TreeMap） |
| TreeMap | √ | - | 不可变的，排序的 map，由红黑树实现 |
| WeakHashMap | - | √ | 一个 java.util.WeakHashMap 的包装，弱引用的 hashmap |

也可以混入 `SynchronizedMap` 特质来创建一个你想要的线程安全的 map 实现。


### 选择 `set`

用 set 和用 map 很相似。有基本的可变和不可变的 set，返回按键排序的 `SortedSet`，按插入顺序存储的 `LinkedHashSet` 以及一些其他特殊用途的 set 。

*常用 set，包括可变和不可变*

| - | 不可变 | 可变 | 描述 |
| ---- | ---- | ---- | ---- |
| BitSet | √ | √ | 非负整数表示为比特放入 64 位字节的可变尺寸数组。当有一组整数是来节省内存空间 |
| HashSet | √ | √ | 不可变版本用 `hash trie` (哈希线索)，可变版本用“哈希表”(hashtable) |
| LinkedHashSet | - | √ | 一个有 hashtab 实现的可变 Set。按插入顺序返回元素 |
| ListSet | √ | - | 用链表结构实现的 Set |
| TreeSet | √ | √ | 不可变版本用树实现。可变版本基于不可变的 AVL 树作为数据结构的 SortedSet |
| SortedSet | √ | √ | 一个基础特质（用 SortedSet 做变量，返回 TreeSet） |


### 表现类似集合的类型

Scala 提供了很多其他的集合类型，还有一些表现的想集合的类型，尽管他们不是集合。

| - | 描述 |
| ---- | ---- |
| `Enumeration` | 一个包含常数值的有限集合（比如一周的天或一年的周） |
| `Iterator` | 迭代器不是一个集合，它可以访问集合中的元素。可以在需要时把迭代器转换成一个集合 |
| `Option` | 包含一个或零个元素的集合。 `Some` 和 `None` 继承自 `Option` 。`Some` 是一个元素的容器，`None` 里没有元素。|
| `Tuple` | 支持异构的集合元素。没有一个叫“元组”的类，元组由 `Tuple1` 到 `Tuple22` 组成，支持从 1 到 22 个元素 |


### 严格（strict）与惰性（lazy）集合














---

traversable : /'trævɝsəbl/ adj. 能越过的，能横过的；可否认的，可反驳的；

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
