---
title: Redis-数据类型
key: 20190222
tags: java, spring cloud, spring cloud ribbon
---

使用 Redis 进行设计和开发时，首先应该考虑 Redis 原生支持的哪些数据类型最适合我们的场景。

<!--more-->

## 字符串 string

字符串是 Redis 的基本数据类型之一，事实上，Redis 中所有的 key 都必须是字符串。

```bash
127.0.0.1:6379> set "mykey" chengchao
127.0.0.1:6379> get "mykey" 
"chengchao"
### get 不存在的 key 返回 nil：
127.0.0.1:6379> get yourkey
(nil)
### strlen 返回字符串长度：
127.0.0.1:6379> strlen mykey 
(integer) 9
### 不存在的 key 执行 strlen， 返回 0：
127.0.0.1:6379> strlen yourkey 
(integer) 0
127.0.0.1:6379> get "Extreme Pizza"
"300 Broadway"
### 使用 append 命令向一个 key 的字符串末尾追加字符串：
127.0.0.1:6379> append "Extreme Pizza" " 10011"
(integer) 18
127.0.0.1:6379> get "Extreme Pizza"
"300 Broadway 10011"
### 使用 setrange 命令可以覆盖字符串值的一部分：
127.0.0.1:6379> 
```

如果某个 key 已经存在，那么 `SET` 命令会覆盖之前的值。

如果不希望覆盖，可以使用 `exists` 命令来测试。

Redis 提供了 `setnx` 命令（简称不存在时 set），用于原子地、尽在 key 不存在时设置值。如果值设置成功，`setnx` 返回 1；如果 key 已经存在，返回 0 并且不覆盖原来的值。

`set` 命令中的 `NX` 选项与 `setnx` 一样。相反，`set` 命令的 `XX` 选项表示仅在 key 已经存在时才设置值。

```bash

127.0.0.1:6379> set chengchao "hello" NX
OK
127.0.0.1:6379> set chengchao "hello" NX
(nil)
127.0.0.1:6379> set chengchao "hello" NX
(nil)
127.0.0.1:6379> get chengchao
"hello"
127.0.0.1:6379> set chengchao "kao" XX
OK
127.0.0.1:6379> GET chengchao
"kao"
127.0.0.1:6379> setnx chengchao "wokao"
(integer) 0
127.0.0.1:6379> get chengchao
"kao"
127.0.0.1:6379> setnx handong "wokao"
(integer) 1
127.0.0.1:6379> setnx handong "wokao"
(integer) 0
127.0.0.1:6379> set handong "wokao2" NX
(nil)
127.0.0.1:6379> set handong2 "wokao2" nx
OK
127.0.0.1:6379> 
127.0.0.1:6379> get handong2
"wokao2"
127.0.0.1:6379> set handong2 "wokao" xx
OK
127.0.0.1:6379> get handong2
"wokao"
127.0.0.1:6379> 

```

**可以使用 `mset` 和 `mget` 命令来一次性地设置和获取多个 key 的值**

使用 `mset` 的优点在于整个操作是原子性的，意味着所有的 key 都是在客户机和服务器之间的一次通信中设置的。可以通过使用一个 `MSET` 节省网络开销。

```bash
MSET key value [key value ...]
MGET key [key value ...]
```

### 字符串在 Redis 内部的编码方式

Redis 使用了三种不同的编码方式来存储字符串对象，并会根据每个字符串值自动决定使用的编码方式：

- `int` ： 用于能够使用 64 位有符号整数表示的字符串。
- `embstr` : 用于长度小于与或等于 44 字节（3.x 中是 39 字节）的字符串；这种类型的编码在内存使用和性能方面更有效率。
- `raw` : 用于长度大于 44 字节的字符串。

使用 `object` 命令来查看与 key 相关联的 Redis 值对象的内部编码方式：

```bash
127.0.0.1:6379> object encoding chengchao
"embstr"
127.0.0.1:6379> set mykey 123
OK
127.0.0.1:6379> object encoding mykey
"int"
127.0.0.1:6379> get mykey
"123"
127.0.0.1:6379> object encoding "extreme Pizza"
"raw"
127.0.0.1:6379> object encoding "Extreme Pizza"
"raw"
127.0.0.1:6379> strlen "Extreme Pizza"
(integer) 21
```

更多内容：

[ https://redis.io/commands ]( https://redis.io/commands )


## 列表 （list）

列表能够存储一组对象，因此它也可以被用作栈和队列。

Redis 中，与 key 相关联的值可以是字符串组成的列表。

Redis 中的列表更像数据结构世界中的双向链表。

**用 `LPUSH` 在列表的左端插入**

```bash
127.0.0.1:6379> LPUSH favorite_site "PF Chang's" "Olive Garder"
(integer) 2
```

**用 `LRANGE` 获取列表中所有数据**

```bash
127.0.0.1:6379> LRANGE favorite_site 0 -1
1) "Olive Garder"
2) "PF Chang's"
```

**用 `RPUSH` 在列表的右端插入数据**

```bash
127.0.0.1:6379> RPUSH favorite_site "guijie" "nanluoguxiang"
(integer) 4
127.0.0.1:6379> lrange favorite_site 0 -1
1) "Olive Garder"
2) "PF Chang's"
3) "guijie"
4) "nanluoguxiang"
127.0.0.1:6379> 
```

**用 `LINSERT` 在指定值后面插入数据**

```bash
127.0.0.1:6379> LINSERT favorite_site AFTER "guijie" "wo-kao"
(integer) 6
127.0.0.1:6379> lrange favorite_site 0 -1
1) "Olive Garder"
2) "PF Chang's"
3) "guijie"
4) "wo-kao"
5) "wo-kao"
6) "nanluoguxiang"
```

**用 `LINDEX` 获取列表中指定索引的值**

```bash
127.0.0.1:6379> LINDEX favorite_site 3
"wo-kao"
```


### 原理

Redis 中的列表与双向链表类似，因此可以使用下面的三个命令将新元素添加到列表中：

- `LPUSH` : 将元素添加到列表的左端。
- `RPUSH` : 将元素添加到列表的右端。
- `LINSERT` : 将元素插入到俩表的支点/枢轴元素（pivotalelement) 之前或之后。

以上命令会返回插入后列表的长度。

想列表插入元素前，无需事先初始化一个空列表。如果我们向一个不存在的 key 中插入元素，Redis 将首先创建换一个空列表并将其与 key 关联。

不需要删除值为空列表的 key， Redis 会自动回收。


如果仅仅想在列表存在时才将元素插入到列表中，那么可以使用 `LPUSHX` 和 `RPUSHX` 命令。

可以使用 `LPOP` 或者 `RPOP` 从列表中删除一个元素。这两个命令会从列表的左端或右端移除第一个元素并返回其值。当 key 不存在时返回 `nil`。

可以使用 `LINDEX` 命令从列表中获取谓语指定索引处的元素

可以使用 `LRANGE` 命令获取一个范围内的元素。

**索引的定义**

在 Redis 中，假设列表有 n 个元素，则列表的索引可以按照从左到右的方式指定为 0 -- n-1； 也可按照从右到左的方式指定为 -1 -- -n。因此 0 -- -1 就表示整个列表。


`LTRIM` 命令可用于在删除列表中的多个元素时，只保留由 start 和 end 索引所指定范围内的元素：

```bash
127.0.0.1:6379> lrange favorite_site 0 -1
1) "Olive Garder"
2) "PF Chang's"
3) "guijie"
4) "wo-kao"
127.0.0.1:6379> ltirm favorite_site 1 -1
(error) ERR unknown command `ltirm`, with args beginning with: `favorite_site`, `1`, `-1`, 
127.0.0.1:6379> ltrim favorite_site 1 -1
OK
127.0.0.1:6379> lrange favorite_site 0 -1
1) "PF Chang's"
2) "guijie"
3) "wo-kao"
127.0.0.1:6379> 
```

**可以用 `LSET` 命令设置列表中指定索引位置处元素的值：**

```bash
127.0.0.1:6379> lrange favorite_site 0 -1
1) "PF Chang's"
2) "guijie"
3) "wo-kao"
127.0.0.1:6379> lset favorite_site 1 gui-jie
OK
127.0.0.1:6379> lrange favorite_site 0 -1
1) "PF Chang's"
2) "gui-jie"
3) "wo-kao"
127.0.0.1:6379> 
```


### 更多

`LPOP` 和 `RPOP` 命令有对应的阻塞版本：`BLPOP` 和 `BRPOP`。

与非阻塞版本类似，阻塞版本的命令也从列表的左端或者右端弹出元素；但是当列表为空时，阻塞版本会将客户端阻塞，必须在这些阻塞版本的命令中指定一个以秒为单位的超时时间，表示最长等待多少秒。

当超时时间设置为 0 时，表示永久等待。这个特性在任务调度场景中非常有用：多个任务执行程序会顶戴任务调度程序分派新的任务。任务执行程序秩序要对 Redis 中的列表使用 `BLPOP` 或 `BRPOP`，之后每当有新任务时，调度程序把任务插入到列表中，任务执行程序之一变回获取到该任务。


Redis 在内部使用 quicklist 存储列表对象。有两个配置选项可以调整列表对象的存储逻辑：

- `list-max-ziplist-size`: 一个列表条目中一个内部节点的最大值（quicklist 的每个节点都是一个 ziplist）。大多数情况下使用默认值即可。
- `list-compress-depth`: 列表压缩策略。

如果我们会用到 Redis 中列表首尾的元素， 那么可以利用这个 选项来获得更好的压缩比（ 译者注： 该参数表示 quicklist 两端不被压缩的节点的个数，当列表很长的时候最可能被访问的数据 是位于列表两端的数据，因此对这个参数精确地进行调优可以实现 在压缩比和其他因素之间的平衡）。



梅隆魁译. Redis 4.x Cookbook 中文版 (Kindle 位置 799-802). 电子工业出版社. Kindle 版本. 

## 哈希 （hash）

**使用 `HMSET` 命令设置值，`HMGET` 命令获取多个值**

```bash
127.0.0.1:6379> get chengchao
"CHENGCHAO"
127.0.0.1:6379> del chengchao
(integer) 1
127.0.0.1:6379> hmset chengchao email "chengchaos@gmail.com" phone 18514026899
OK
127.0.0.1:6379> hmget chengchao
(error) ERR wrong number of arguments for 'hmget' command
127.0.0.1:6379> hmget chengchao email
1) "chengchaos@gmail.com"
127.0.0.1:6379> hmget chengchao email phone
1) "chengchaos@gmail.com"
2) "18514026899"
127.0.0.1:6379> 
```

**使用 `HGET` 命令从一个哈希中获取某个字段对应的值，`HSET` 命令设置单个的值**

```bash
127.0.0.1:6379> get handong
"HANDONG"
127.0.0.1:6379> del handong
(integer) 1
127.0.0.1:6379> hset handong email handong66666@yahoo.com.cn
(integer) 1
127.0.0.1:6379> hget handong email
"handong66666@yahoo.com.cn"
```

**使用 `HEXISTS` 测试一个哈希中是否存在某个字段；`HGETALL` 命令获取一个哈希中的所有字段值**

```bash
127.0.0.1:6379> EXISTS CHENGCHAO
(integer) 0
127.0.0.1:6379> exists chengchao
(integer) 1
127.0.0.1:6379> hexists handong email
(integer) 1
127.0.0.1:6379> hexists handong phone
(integer) 0
127.0.0.1:6379> hgetall handong
1) "email"
2) "handong66666@yahoo.com.cn"
127.0.0.1:6379> 
```

**使用 `HDEL` 删除哈希中的字段。**


### 工作原理

类似 List，不需要在添加字段前先初始化一个空的哈希。当哈希编程空的时，Redis 会自动将其删除。

默认情况下，`HSET` 和 `HMSET` 会覆盖现有的字段。`HSETNX` 命令则仅在字段不存在的情况下才设置值，可用于防止 `HSET` 的默认覆盖行为。

对于不存在的 key 或者 field。 `HMGET' 和 `HGET` 将返回 `nil` 。

### 更多细节

一个哈希最多能容纳 `2^32 -1` 个字段。如果一个哈希的字段非常多，那么执行 `HGETALL` 命令时可能会阻塞 Redis 服务器。这种情况下，可以使用 `HSCAN` 来增量地获取所有字段和值。

`HSCAN` 是 Redis 中 `SCAN` 命令的一种（`SCAN`, `HSCAN`, `SSCAN`, `ZSCAN`）,该命令会增量地迭代遍历元素，从而不会造成服务器阻塞。

`HSCAN` 命令时一种基于指针的迭代器，因此我们需要在每次调用命令时指定一个游标（从 0 开始），当一次 `HSCAN` 运行结束后，Redis 将返回一个元素列表以及一个新的游标，这个游标可用于下一次迭代。

`HSCAN` 用法：

```
HSCAN key cursor [MATCH pattern] [COUNT number]
```

- 选项 `MATCH` 用来匹配满足指定 Glob 表达式的字段。
- 选项 `COUNT` 说明每次迭代中应该返回多少元素。仅仅是一个参考，默认 10. Redis 不保证返回的元素数量一定是 COUNT 个。

```bash
127.0.0.1:6379> HSCAN restaurant 0 MATCH *garden*
1) "309"
2) 1) "panda garden"
   2) "3.9"
   3) "chang's garden"
   4) "4.5"
127.0.0.1:6379> HSCAN restaurant 309 MATCH *garden*
1) "0"
2) 1) "szechuwan garden"
   2) "4.9"
```

当服务器返回的游标为 0 时，表示整个遍历完成。

### 存储

Redis 在内部使用两种编码来存储哈希对象

- `ziplist` : 对于那些长度小于配置中 `hash-max-ziplist-entries` 选项配置的值（默认 512），并且所有元素得大小都小于配置中 `hash-max-ziplist-value` 选项配置的值（默认 64 字节）的哈希，采用此编码。ziplist 编码对于较小的哈希而言可以节省占用空间。
- `hashtable` : 当 ziplist 不适用时使用的默认编码。

## 集合（set）类型

集合类型是由唯一、无序对象组成的集合（collection）。它经常用于测试某个成员是否在集合中、去除重复项、集合运算（并集、交集、差集）。

Redis 的值对象可以是字符串集合。


**使用 `SADD` 命令添加数据**

```bash
127.0.0.1:6379> sadd tomcat java jsp servlet web cgi
(integer) 5
```

**使用 `SISMEMBER` 命令测试一个元素是否在集合中**

```bash
127.0.0.1:6379> sismember tomcat java
(integer) 1
127.0.0.1:6379> sismember tomcat php
(integer) 0
```

**使用 `SREM` 命令从结合中删除元素**

```bash
127.0.0.1:6379> srem tomcat web
(integer) 1
```

**使用 `SCARD` 命令获取集合中成员数量**

```base
127.0.0.1:6379> scard tomcat
(integer) 4
```

### 工作原理

与列表和哈希类似，Redis 在执行 `SADD` 命令时，不需要事先准备好一个空集合。同样，Redis 也会自动删除空集合对应的 Key。

### 更多细节

在 Redis 中，一个集合最多可以容纳 `2^32 -1` 个成员。

可以使用	`SMEMBERS` 命令列出集合中的所有元素。但是，类似哈希，在一个大的集合中使用 `SMEMBERS` 命令可能会阻塞服务器，因此建议此时使用 `SSCAN` 命令，与哈希的 `HSCAN` 命令类似。

Redis 提供了一组集合运算相关的命令， `SUNION` 和 `SUNIONSTORE` 用于计算并集；`SINTER` 和 `SINTERSTORE` 用于计算交集；`SDIFF` 和 `SDIFFSTORE` 用于计算差集。

不带 `STORE` 后缀的命令只返回相应操作的结果集合，而带 `STORE` 后缀的命令则会将结果存储到一个指定的 key 中。

```bash
127.0.0.1:6379> sadd jetty jsp servlet java
(integer) 3
127.0.0.1:6379> sscan jetty 0 match *
1) "0"
2) 1) "jsp"
   2) "java"
   3) "servlet"
127.0.0.1:6379> smembers tomcat
1) "jsp"
2) "servlet"
3) "java"
4) "cgi"
127.0.0.1:6379> smembers jetty
1) "jsp"
2) "java"
3) "servlet"
127.0.0.1:6379> sinter tomcat jetty
1) "jsp"
2) "java"
3) "servlet"
127.0.0.1:6379> sinterstore common tomcat jetty
(integer) 3
127.0.0.1:6379> smembers common
1) "jsp"
2) "servlet"
3) "java"
```

### 存储

Redis 内部使用两种编码凡是来存储集合对象：

- `intset` : 对于那些元素都是整数，并且元素个数小于配置中 `set-max-intset-entries` 选项的值（默认 512）的集合，采用此种编码
- `hashtable` : 对于 intset 不适用时的默认编码。


## 有序集合（storted set）

**使用 `ZADD` 命令添加数据**

```bash
127.0.0.1:6379> zadd ranking 100 "java" 80 "php" 70 "c++" 90 "c"
(integer) 4
```

**使用 `ZREVRANGE` 命令获取排名**

```bash
127.0.0.1:6379> zrevrange ranking 0 -1
1) "java"
2) "c"
3) "php"
4) "c++"
127.0.0.1:6379> zrevrange ranking 0 -1 withscores
1) "java"
2) "100"
3) "c"
4) "90"
5) "php"
6) "80"
7) "c++"
8) "70"
```

**使用 `ZINCRBY` 命令增加得分**

```bash
127.0.0.1:6379> zincrby ranking 1 c
"91"
127.0.0.1:6379> zrevrange ranking 0 -1 withscores
1) "java"
2) "100"
3) "c"
4) "91"
5) "php"
6) "80"
7) "c++"
8) "70"
```

**使用 `ZREVRANK` 命令和 `ZSCORE` 命令浏览特定的排名和得分**

```bash
127.0.0.1:6379> zrevrank ranking java
(integer) 0
127.0.0.1:6379> zscore ranking java
"100"
127.0.0.1:6379> zrevrank ranking java2
(nil)
```

**使用 `ZUNIONSTORE` 合并两个有序集合**

### 工作原理

在 `ZADD` 命令中使用 `NX` 选项，能够实现在不更新有才能在成员的情况下只添加新成员；`XX` 选项允许不添加新元素的情况先更新集合（只更新存在的成员而不添加新成员）。这些选项只使用与 Redis 3.0.2 及更高版本。

多个不同的成员可能具有相同的权重，此时 Redis 将按照字段顺序进行排序。

`ZUNIONSTORE` 命令英语将两个有序集合的并集保存到指定的 Key 中，并且可以指定各个有序几个的不同权重。

```bash
ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] AGGREGATE SUM|MIN|MAX]
```

### 存储

Redis 在内部使用两种编码凡是存储有序集合对象：

- `ziplist` : 对于那些长度小于配置中 `zset-max-ziplist-entries` 选项配置的值（默认128），并且素有元素的大小都小于配置中 `zset-max-ziplist-value` 选型配置的值（默认 64 字节）的有序集合，采用此编码。ziplist 用于节省较小集合所占用的空间
- `skiplist` : 当 ziplist 不适用时使用的默认编码。


## HyperLogLog 类型


在日常的各种数据处理场景中，“唯一计数”所以一向常见的任务。在 Redis 中，虽然我们可以使用集合来进行唯一计数，但是当数据量增大到上千万是，就需要考虑内存消耗和性能下降的问题了。如果我们不需要获取数据集合的内容，而只是想得到不同值的个数，那么久可以使用 `HyperLogLog (HLL)` 数据类型来优化使用集合类型时存在的内存和性能问题。


假设需要计算访问 Olive Garden 餐厅的不同客户数：

使用 `PFADD` 命令吧用户 ID 加入到一个 HyperLogLog 中。然后使用 `PFCOUNT` 命令获取该餐厅的额不同到访者数量：

```bash
127.0.0.1:6379> pfadd "counting:Olive Garden" "0000123"
(integer) 1
127.0.0.1:6379> pfadd "counting:Olive Garden" "0023992"
(integer) 1
127.0.0.1:6379> pfcount "counting:Olive Garden"
(integer) 2
127.0.0.1:6379> 
```

如果想要展示 Olive Garden 在一个星期中额独立访客数，并将其作为每周流行度的标志，那么，可以每天生成一个 HLL，然后用 `PFMERGE` 命令把这 7 天的数据合并为一个：

```
PFADD "Counting:Olive Garden:20170903" "0023992" "0023991" "0045992"
PFADD "Counting:Olive Garden:20170904" "0023992" "0023991" "0045992"
PFADD "Counting:Olive Garden:20170905" "0024492" "0023211" "0045992"
PFADD "Counting:Olive Garden:20170906" "0023999" "0023991" "0045992"
PFADD "Counting:Olive Garden:20170907" "0023992" "0063991" "0045992"
PFADD "Counting:Olive Garden:20170908" "0023292" "0023991" "0045991"
PFADD "Counting:Olive Garden:20170909" "0023282" "0023984" "0045092"


PFMERGE "Counting:Olive Garden:20170903week" "Counting:Olive Garden:20170903" "Counting:Olive Garden:20170904" "Counting:Olive Garden:20170905" "Counting:Olive Garden:20170906" "Counting:Olive Garden:20170907" "Counting:Olive Garden:20170908" "Counting:Olive Garden:20170909"

PFCOUNT "Counting:Olive Garden:20170903week" 



127.0.0.1:6379> PFADD "Counting:Olive Garden:20170903" "0023992" "0023991" "0045992"
(integer) 1
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170904" "0023992" "0023991" "0045992"
(integer) 1
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170905" "0024492" "0023211" "0045992"
(integer) 1
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170906" "0023999" "0023991" "0045992"
Invalid argument(s)
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170907" "0023992" "0063991" "0045992"
(integer) 1
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170906" "0023999" "0023991" "0045992"
(integer) 1
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170908" "0023292" "0023991" "0045991"
(integer) 1
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170909" "0023282" "0023984" "0045092"
(integer) 1
127.0.0.1:6379> PFMERGE "Counting:Olive Garden:20170903week" "Counting:Olive Garden:20170903" "Counting:Olive Garden:20170904" "Counting:Olive Garden:20170905" "Counting:Olive Garden:20170906" "Counting:Olive Garden:20170907" "Counting:Olive Garden:20170908" "Counting:Olive Garden:20170909"
OK
127.0.0.1:6379> PFCOUNT "Counting:Olive Garden:20170903week" 
(integer) 12
```

### 工作原理

HHL 类型相关所有命令都是以 `PF` 开头的，用来向 HLL 数据结构的发明者 Philipe Flajolet 致敬。

Redis 中 HLL 的优势在于能够使用固定数量的内存（每个 HyperLogLog 类型的 Key 只需要占用 128KB 内存，却可以计算最多 `2^64` 个不同元素的基数）和常熟时间复杂度（每个 Key  `O(1)`）进行唯一计数。

不过，由于 HLL 算法返回的基数（HLL 实际是 cardinality counting 基数计算的一种）可能不准确（标准差小于 1%），因此在决定是否使用 HLL时需要进行权衡。

### 更多细节

（待续）





## Geo 类型


（待续）

## Key 管理


```bash

$ sudo pip install redis fake2db
$ pip
$ sudo apt install python-pip
$ sudo pip install redis fake2db
$ sudo pip install --upgrade pip
$ fake2db --rows 10000 --db redis

```

**获取 Key 的个数**

```sh
127.0.0.1:6379> DBSIZE
(integer) 50000
```

**获取 Redis 中所有的 Key(1) `KEYS`**

```sh
127.0.0.1:6379> KEYS *
...
49997) "user_agent:336"
49998) "detailed_registration:9720"
49999) "detailed_registration:1256"
50000) "company:1084"
(6.89s)
```

**获取 Redis 中所有的 Key(2) `SCAN`**


```sh
127.0.0.1:6379> scan 0
1) "57344"
2)  1) "company:2914"
    2) "user_agent:878"
    3) "customer:1063"
    4) "simple_registration:4186"
    5) "customer:1264"
    6) "user_agent:5763"
    7) "detailed_registration:5061"
    8) "company:4318"
    9) "simple_registration:9342"
   10) "simple_registration:3351"
127.0.0.1:6379> scan 57344
1) "61440"
2)  1) "user_agent:8854"
    2) "customer:2289"
    3) "detailed_registration:7221"
    4) "company:1017"
    5) "customer:8992"
    6) "detailed_registration:8530"
    7) "customer:4082"
    8) "user_agent:159"
    9) "simple_registration:3825"
   10) "simple_registration:4324"
   11) "simple_registration:4771"
   12) "company:1112"
127.0.0.1:6379> 
```

**删除 Redis 中的 Key `DEL` / `UNLINK`**

`DEL` 和 `UNLINK` (4.0 以上版本)

```sh
127.0.0.1:6379> DEL "detailed_registration:1665" "simple_registration:6411"
(integer) 2
127.0.0.1:6379> unlink "company:1664"
(integer) 1
```

**判断一个 Key 是否存在 `EXISTS`**

```sh
127.0.0.1:6379> EXISTS "simple_registration:7681"
(integer) 1
127.0.0.1:6379> EXISTS "simple_registration:99999"
(integer) 0
```

**获取数据类型 `TYPE`**

```sh

127.0.0.1:6379> TYPE "company:3859"
hash
```

**重命名一个 Key: `RENAME`**

```sh
127.0.0.1:6379> RENAME "customer:6591" "customer:6591:renamed"
OK
```

### 工作原理

生产环境执行 `KEYS *` 是一个危险操作, 建议使用 `SCAN` 替代

另外,对于非字符串数据, 使用个 `DEL` 命令需要留意, 如果元素数据量很大,可能会导致服务器延迟, 为避免, 应该使用 `UNLINK` 替代. `UNLINK` 会在另一个线程而不是主事件循环线程中执行删除操作.

`RENAME` 命令在目标 Key 存在时会将其删除. 或可能引发 `DEL` 命令引起的高延迟. 因此, 重命名的最佳实践是: 如果目标 Key 已经存在, 则先对其执行 `UNLINK`, 然后再重命名.


### 更多细节

`DUMP` / `RESTORE` 命令可以用于序列号和反序列化.可以用这两个命令对 Redis 进行备份工作.













If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
