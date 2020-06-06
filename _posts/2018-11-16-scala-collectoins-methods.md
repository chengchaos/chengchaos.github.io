---
title: Scala 的集合的方法
key: 20181116
tags: scala scala-collection
---

本文是我学习[《Scala 编程实战》 （Alexander，A. 著）]( https://www.amazon.cn/dp/B06X9SDVB8/ref=sr_1_1?ie=UTF8&qid=1542361298&sr=8-1&keywords=Scala+%E7%BC%96%E7%A8%8B%E5%AE%9E%E6%88%98 )的读书笔记。

![Scala 编程实战 封面]( https://images-cn.ssl-images-amazon.com/images/I/417LU7wGg%2BL.jpg )


Scala 集合里有大量的方法可以使用个，那如何选择这些方法来解决问题呢？

<!--more-->

Scala 集合类提供了丰富的用来操作数据的方法，绝大多数方法以一个函数或者谓词作为参数。

## 按类别划分方法

### 过滤方法

可以用来过滤一个集合的方法，包括： `collect`, `diff`, `distinct`, `drop`, `dropWhile`, `filter`, `filterNot`, `find`, `foldLeft`, `foldRight`, `head`, `headOption`, `init`, `intersect`, `last`, `lastOption`, `reduceLeft`, `reduceRight`, `remove`, `slice`, `tail`, `take`, `takeWhile`, `union` 。

### 转换方法

转换方法最少有一个集合作为输入然后通过提供一个算法创建一个新的集合作为输出。

转换方法包括：`+`, `++`, `--`, `diff`, `distinct`, `collect`, `flatMap`, `map`, `reverse`, `sortWith`, `takeWhile`, `zip`, `zipWithIndex` 。

### 分组方法

分组方法会让你根据一个已有的集合创建多个分组。

分组方法包括： `groypBy`, `partition`, `sliding`, `span`, `splitAt`, `unzip` 。

### 信息和数学方法

这些方法提供关于一个集合的信息，包括： `canEqual`, `contains`, `containsSlice`, `count`, `endsWith`, `exists`, `find`, `forAll`, `hasDefiniteSize`, `indexOf`, `indexOfSlice`, `indexWhere`, `isDefinedAt`, `isEmpty`, `lastIndexOf`, `lastIndexOfSlice`, `size`, `startsWith`, `sum`。

和 `foldLeft`, `foldRight`, `reduceLeft`, `reduceRight` 这样的方法也可以通过提供一个函数去获取集合的信息。

### 其他

一些方法很难分类，例如： `par`, `view`, `flatten`, `foreach`, `mkString` 等。

- `par` 会根据已有的集合创建一个平行的集合。
- `view` 在集合上创建一个惰性视图。
- `flatten` 将包含列表的列表转换成一个列表。
- `foreach` 就像循环，遍历集合中的每个元素。
- `mkString` 会根据一个集合声场一个字符创。

还有更多未列出的方法。例如，有 `to*` 的一系列的方法，可以将当前的集合（如 List）转换成其他集合类型（如 Array，Buffer，Vector 等）。


## 常用的集合方法

下表列出了常用的集合的方法，表中的字符说明如下：

- `c` 代表一个集合。
- `f` 代表一个函数。
- `p` 代表一个谓词。
- `n` 代表一个数字。
- `op` 代表一个简单的操作（通常是一个简单的函数）。


*可遍历的集合的常用方法*

| 方法 | 描述 |
|---- | ---- |
| `c collect f` | 教集合里所有元素应用一个偏函数来构建一个由这个函数定义的新集合 |
| `c count p` | 对集合里满足谓词条件的元素进行计数 |
| `c1 diff c2` | 返回 c1 里与 c2 不同的元素 |
| `c drop n` | 返回集合里去除前 n 个元素的剩余元素 |
| `c dropWhile p` | 返回“满足谓词的最长前缀元素”的集合（看不懂！） |
| `c exists ` | 如果集合中的每个元素都让谓词为真，返回 `true` |
| `c filter p` | 返回集合中让谓词为真的所有元素 |
| `c filterNot p` | 返回集合中让维持为假的所有元素 |
| `c find p` | 查找第一个匹配谓词的元素，找到返回 `Some[A]` 否则返回 `None` |
| `c flatten` | 把一个集合的集合变成一个单一的集合 |
| `c flatMap f` | ？？ |
| `c foldLeft(z)(op)` | 把操作依次应用到元素上，从左到右，由 z 开始 |
| `c foldRight(z)(op)` | 把操作依次应用到元素上，从右到做，由 z 开始 |
| `c forAll p` | 如果所有元素都妈祖谓词返回 `true` ，否则返回 `false` |
| `c foreach f` | 把函数应用到集合的所有元素 |
| `c groupBy f` | 根据函数把集合归类成一个 Map |
| `c hasDefiniteSize` | 测试集合是否是有限的（比如，对 Stream 或 Iterator 返回 `false` ） |
| `c head` | 返回集合的第一个元素，如果集合为空则抛出 `NoSuchElementException` |
| `c headOption` |返回集合的第一个元素，如果元素存在则返回 `Some[A]` 否则返回 `None` |
| `c init` | 选择集合中出去最后一个元素的所有元素，集合为空则抛出 `UnsupportedOperationException` |
| `c1 intersect c2` | 在支持的集合上，返回两个集合的交集（共同拥有的元素） |
| `c isEmpty` | 如果集合为空返回 `true`，否则返回 `false`。|
| `c last` | 返回集合的最后一个元素，为空则抛出 `NoSuchElementException` |
| `c lastOption` | 返回集合的最后一个元素，如果元素存在返回 `Some[A]` 否则返回 `None` |
| `c map f` | 把函数应用到集合中的每一个元素，返回一个新的集合 |
| `c max` | 返回集合中的最大元素 |
| `c min` | 返回集合总的最小元素 |
| `c nonEmpty` | 如果集合为空返回 `false`，否则返回 `true`。 |
| `c par` | 返回一个集合的并行实现（比如，Array 返回 ParArray） |
| `c partition p` | 根据谓词算法返回两个集合 |
| `c product` | 返回集合中所有元素的乘积 |
| `c reduceLeft op` | 和 `foldLeft` 一样，单从集合的的一个元素开始 |
| `c reduceRight op` | 和 `foldRight` 一样，但从集合的最后一个元素开始 |
| `c reverse` | 返回集合的倒序（ Traversable 上不可行，但大多数集合都可以用，来自 `GenSeqLike`） |
| `c size` | 返回集合的大小 |
| `c slice(from, to)` | 返回 from 到 to 元素的间隔 |
| `c sortWith f` | 返回按函数比较的集合版本 |
| `c span p` | 返回由两个集合组成的一个集合， 第一个来自 `c.takeWhile(p)`, 第二个来自 `c.dropWhile(p)` |
| `c splitAt n` | 通过 n 所在集合的集合 c 的位置拆分一个集合 |
| `c sum` | 返回集合所有元素的和 |
| `c tail` | 返回集合中除第一个元素以外的其他所有元素 |
| `c take n` | 返回集合中前 n 个元素 |
| `c takeWhile p` | 当谓词判断为 `true` 是返回集合中的元素，为 `false` 时停止。 |
| `c1 union c2` | 返回两个集合的并集 |
| `c unzip` | 与 `zip` 相反，把每个元素分成两份从而将一个集合一分为二，好像把 Tuple2 的位元素的集合拆分开来。 |
| `c view` | 返回集合的一个非严格（惰性）视图 |
| `c1 zip c2` | 通过按对匹配来创建一个 `pair` 集合。比如 c1 中的 0 匹配 c2 中的0 …… |
| `c zipWithIndex` | 按照下标遍历集合 |


### 可变集合的方法


*可变集合中常用的操作符（方法）*

| 操作符（方法） | 描述 |
| ---- | ---- |
| `c += x` | 把 x 元素添加到集合 c |
| `c += (x, y, z)` | 把 x, y, z 元素加入到集合 c |
| `c1 ++= c2` | 把 c2 里的额元素添加到 c1 |
| `c -= x` | 从集合中删除元素 x |
| `c -= (x, y, z)` | 从集合 c 中删除元素 x, y, z |
| `c1 --= c2` | 从 c1 中删除 c2 中的元素 |
| `c(n) = x` | 把值 x 赋给元素 c(n) |
| `c clear` | 删除集合中的所有元素 |
| `c remove n` | 删除位置为 n 的元素 |
| `c.remove(n, len)` | 删除第 n 位开头并且持续 len 长度的元素 |

还有一些额外的方法，不过这些事最常见的。


### 不可变集合操作

*不可变集合中的常用操作符（方法）*

| 操作符（方法） | 描述 |
| ---- | ---- |
| `c1 ++ c2` | 把 c2 集合里的元素附加到 c1 里创造一个新的集合 |
| `c :+ e` | 返回把元素 e 附加到集合 c 的新集合 |
| `e +: c` | 返回一个把元素 e 插到集合 c 前面的新集合 |
| `e :: list` | 返回一个把 e 前插到名叫 list 列表的集合（`::` 只能在 List 上使用） |
| `c drop n ` | `-` 和 `--` 两个方法都被弃用了。 |
| `c dropWhile p` | 删除某个元素，返回新的集合。 |
| `c filter p` | - |
| `c filterNot p` | - |
| `c head` | - |
| `c tail` | - |
| `c take n` | - |
| `c takeWhile p` | - |

此表只是列出了不可变集合中最常用的方法。


### Mpas

Maps 中还有一些附加的方法，如下表所示，表中的符号说明如下：

- `m` 代表一个 map。
- `mm` 代表一个可变 map。
- `k` 代表一个键。
- `p` 代表一个谓词。
- `v` 代表一个 map 中的值。
- `c` 代表一个集合

*可变和不可变 map 中的方法*

| Map 方法 | 描述 |
| ---- | ---- |
|**不可变 map 的方法** | - |
| `m - k` | 返回删除 key （以及它相应的 value）后的 map |
| `m - (k1, k2, k3)` | 同上，删除了多个 可以 后的 map |
| `m -- c` | 同上，参数是任何序列集合 |
| `m -- List(k1, k2)` | 同上 |
| **可变 map 的方法**| - |
| `mm += (k -> v)` | 添加 |
| `mm += (k1 -> v1, k2 -> v2)` | 同上 |
| `mm ++= c` | 把集合 c 中的元素添加到可变 map mm 中 |
| `mm ++= List(3 -> c)` | 同上 |
| `mm -= k` | 从可变 map mm 中删掉 key 为 k 的键值对 |
| `mm -= (k1, k2, k3)` | 同上 …… |
| `mm -= c` | 同上 …… |
| **可变和不可变都有的方法**| - |
| `m(k)` | 返回 k 的 value |
| `m contains k` | 判断是否包含 k |
| `m filter p` | 返回满足谓词 p 条件的键 |
| `m filterKeys p` | 返回包含匹配谓词 p 条件的 key 的 map |
| `m get k` | 如果 k 存在返回 `Some[A]` 否则返回 `None` |
| `m getOrElse(k, d)` | 同上，只是不存在是返回 d |
| `m is DefinedAt k` | 如果 map 包含 k 返回 true |
| `m keys ` | 把 map 中的 keys 作为 Iterable |
| `m keyIterator` | 把 map 中的 key 作为 Iterator返回 |
| `m keySet` | 把 map 中的 keys 作为 Set 返回 |
| `m mapValues f` | 把函数 f 应用到 map 中的每一个值，然后发返回一个新 map |
| `m values` | 把 map 中的value 作为 Iterable 返回 |
| `m valuesIterator ` | 把 map 中的值最为 Iterator 返回 |









---


If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
