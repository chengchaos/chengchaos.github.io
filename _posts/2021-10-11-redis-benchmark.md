---
title: Redis Benchmark
key: 2021-10-11
tags: redis benchmark
---

## 性能指标

通过我们看各大厂商提供的指标，我们不难发现，主要是 **QPS** 。

看一下网上的 Redis 和 MySQL 的性能对比:

1C 1GB 配置

Redis 1C 1GB 主从版，提供 80000 QPS

MySQL 1C 1GB 通用型，提供 465 QPS

相差 172 倍左右的性能

16C 128G 配置

Redis 16C 128G 集群版（单副本），提供 1280000 QPS

MySQL 16C 128G 独享版，提供 48102 QPS

相差 26 倍左右的性能

<!--more-->

## 测试工具

Redis 的性能测试工具，目前主流使用的是 redis-benchmark 。为什么这么说呢？

- 在我们 Google 搜索 “Redis 性能测试”时，清一色的文章选择的工具，清一色的都是 redis-benchmark 。
- 我翻看了云厂商（腾讯云、UCloud 等），提供的测试方法，都是基于 redis-benchmark 。

当然，也是有其它工具：

- memtier_benchmark ：目前阿里云提供的测试方法，是基于它来实现，具体可以看看 [阿里云 Redis —— 测试工具](https://help.aliyun.com/document_detail/100347.html) 。
- jMeter: 测试都会用这个。
- YCSB ：YCSB 能够测试的服务特别多（回头再说）。

## redis-benchmark

Redis 自带了一个叫 redis-benchmark 的工具来模拟 N 个客户端同时发出 M 个请求。（类似于 Apache ab 程序）。

### 安装

redis-benchmark 是 Redis 自带的，所以不需要专门去安装.

### 使用

redis-benchmark 的使用非常简单，只要了解它每个参数的作用，就可以非常方便的执行一次性能测试。我们来一起看看有哪些参数。执行 `redis-benchmark --help` 命令，返回参数列表：

```bash
redis-benchmark --help
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests>] [-k <boolean>]

 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 3)
 --dbnum <db>       SELECT the specified db number (default 0)
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
  Using this option the benchmark will expand the string __rand_int__
  inside an argument with a 12 digits number in the specified range
  from 0 to keyspacelen-1. The substitution changes every time a command
  is executed. Default tests use this to hit random keys in the
  specified range.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -e                 If server replies with errors, show them on stdout.
                    (no more than 1 error per second is displayed)
 -q                 Quiet. Just show query/sec values
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.

Examples:

 Run the benchmark with the default configuration against 127.0.0.1:6379:
   $ redis-benchmark

 Use 20 parallel clients, for a total of 100k requests, against 192.168.1.1:
   $ redis-benchmark -h 192.168.1.1 -p 6379 -n 100000 -c 20

 Fill 127.0.0.1:6379 with about 1 million keys only using the SET test:
   $ redis-benchmark -t set -n 1000000 -r 100000000

 Benchmark 127.0.0.1:6379 for a few commands producing CSV output:
   $ redis-benchmark -t ping,set,get -n 100000 --csv

 Benchmark a specific command line:
   $ redis-benchmark -r 10000 -n 10000 eval 'return redis.call("ping")' 0

 Fill a list with 10000 random elements:
   $ redis-benchmark -r 10000 -n 10000 lpush mylist __rand_int__

 On user specified command lines __rand_int__ is replaced with a random integer
 with a range of values selected by the -r option.



-h ：Redis 服务主机地址，默认为 127.0.0.1 。
-p ：Redis 服务端口，默认为 6379 。
-s ：指定连接的 Redis 服务地址，用于覆盖 -h 和 -p 参数。一般情况下，我们并不会使用。
-a ：Redis 认证密码。
--dbnum ：选择 Redis 数据库编号。
k ：是否保持连接。默认会持续保持连接。
请求相关参数
默认情况下，使用 rand_int 作为 KEY 。
通过设置 -r 参数，可以设置 KEY 的随机范围。例如说，-r 10 生成的 KEY 范围是 [0, 9) 。
重要：一般情况下，我们会自动如下参数，以达到不同场景下的性能测试。
-c ：并发的客户端数（每个客户端，等于一个并发）。
-n ：总共发起的操作（请求）数。例如说，一次 GET 命令，算作一次操作。
-d ：指定 SET/GET 操作的数据大小，单位：字节。
-r ：SET/GET/INCR 使用随机 KEY ，SADD 使用随机值。
-P ：默认情况下，Redis 客户端一次请求只发起一个命令。通过 -P 参数，可以设置使用 pipelining功能，一次发起指定个请求，从而提升 QPS 。
-l ：循环，一直执行基准测试。
-t ：指定需要测试的 Redis 命令，多个命令通过逗号分隔。默认情况下，测试 PING_INLINE/PING_BULK/SET/GET 等等命令。如果胖友只想测试 SET/GET 命令，则可以 -t SET,GET 来指定。
-I ：Idle 模式。仅仅打开 N 个 Redis Idle 个连接，然后等待，啥也不做。不是很理解这个参数的目的，目前猜测，仅仅用于占用 Redis 连接。

输出相关：
-e ：如果 Redis Server 返回错误，是否将错误打印出来。默认情况下不打印，通过该参数开启。
-q ：精简输出结果。即只展示每个命令的 QPS 测试结果。如果不理解的胖友，跑下这个参数就可以很好的明白了。
--csv ：按照 CSV 的格式，输出结果。
```

### 快速测试

```bash
redis-benchmark
```

在安装 Redis 的服务器上，直接执行，不带任何参数，即可进行测试。

我的服务器上返回:

```bash
Welcome to Alibaba Cloud Elastic Compute Service !

chengchao@web1:~
> redis-benchmark
====== PING_INLINE ======
  100000 requests completed in 1.22 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.96% <= 1 milliseconds
100.00% <= 1 milliseconds
82169.27 requests per second

====== PING_BULK ======
  100000 requests completed in 1.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.91% <= 1 milliseconds
99.98% <= 17 milliseconds
100.00% <= 17 milliseconds
66844.91 requests per second

====== SET ======
  100000 requests completed in 1.48 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.92% <= 1 milliseconds
99.99% <= 2 milliseconds
100.00% <= 2 milliseconds
67430.88 requests per second

====== GET ======
  100000 requests completed in 1.43 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.95% <= 1 milliseconds
100.00% <= 1 milliseconds
69979.01 requests per second

====== INCR ======
  100000 requests completed in 1.34 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.94% <= 1 milliseconds
99.95% <= 2 milliseconds
100.00% <= 2 milliseconds
74515.65 requests per second

====== LPUSH ======
  100000 requests completed in 1.37 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.88% <= 1 milliseconds
99.98% <= 14 milliseconds
100.00% <= 14 milliseconds
73152.89 requests per second

====== RPUSH ======
  100000 requests completed in 1.31 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.96% <= 1 milliseconds
100.00% <= 1 milliseconds
76103.50 requests per second

====== LPOP ======
  100000 requests completed in 1.32 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.91% <= 1 milliseconds
99.99% <= 2 milliseconds
100.00% <= 2 milliseconds
75872.54 requests per second

====== RPOP ======
  100000 requests completed in 1.47 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.94% <= 1 milliseconds
100.00% <= 1 milliseconds
68027.21 requests per second

====== SADD ======
  100000 requests completed in 1.59 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.91% <= 1 milliseconds
99.97% <= 15 milliseconds
100.00% <= 15 milliseconds
62774.64 requests per second

====== HSET ======
  100000 requests completed in 1.48 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.93% <= 1 milliseconds
100.00% <= 1 milliseconds
67567.57 requests per second

====== SPOP ======
  100000 requests completed in 1.61 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.93% <= 1 milliseconds
99.95% <= 2 milliseconds
100.00% <= 2 milliseconds
62305.30 requests per second

====== LPUSH (needed to benchmark LRANGE) ======
  100000 requests completed in 1.50 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.88% <= 1 milliseconds
99.99% <= 2 milliseconds
100.00% <= 2 milliseconds
66445.18 requests per second

====== LRANGE_100 (first 100 elements) ======
  100000 requests completed in 1.58 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.92% <= 1 milliseconds
99.97% <= 14 milliseconds
99.98% <= 15 milliseconds
100.00% <= 15 milliseconds
63291.14 requests per second

====== LRANGE_300 (first 300 elements) ======
  100000 requests completed in 1.60 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.97% <= 1 milliseconds
100.00% <= 1 milliseconds
62383.03 requests per second

====== LRANGE_500 (first 450 elements) ======
  100000 requests completed in 1.40 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.83% <= 1 milliseconds
99.95% <= 2 milliseconds
99.98% <= 3 milliseconds
100.00% <= 3 milliseconds
71225.07 requests per second

====== LRANGE_600 (first 600 elements) ======
  100000 requests completed in 1.18 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.93% <= 1 milliseconds
100.00% <= 1 milliseconds
84530.86 requests per second

====== MSET (10 keys) ======
  100000 requests completed in 1.61 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.94% <= 1 milliseconds
99.98% <= 15 milliseconds
100.00% <= 15 milliseconds
62227.75 requests per second


```

### 精简测试

```bash
redis-benchmark -t set,get,incr -n 1000000 -q -a sinogold
SET: 79948.83 requests per second
GET: 81017.58 requests per second
INCR: 71022.73 requests per second

```

### pipeline 测试

在一些业务场景，我们希望通过 Redis pipeline 功能，批量提交命令给 Redis Server ，从而提升性能。那么，我们就来测试下

```bash
redis-benchmark -t set,get,incr -n 1000000 -q -P 10 -a sinogold
SET: 591715.94 requests per second
GET: 740192.50 requests per second
INCR: 724637.69 requests per second

```

### 随机 Key 测试

我们主要来看看 `-r` 参数的使用。为了更好的对比，我们来先看看未使用的 `-r` 的情况，然后再测试使用 `-r` 的情况。

我们来查看 Redis 中，有哪些 KEY ：

```bash
redis-cli -a sinogold flushdb
Warning: Using a password with '-a' option on the command line interface may not be safe.
OK

redis-benchmark -t set -n 1000 -q -a sinogold
SET: 83333.34 requests per second

redis-cli -a sinogold keys \*
Warning: Using a password with '-a' option on the command line interface may not be safe.
1) "key:__rand_int__"

```

只有一个以 key: 开头，结尾是 rand_int" 的 KEY 。这说明，整个测试过程，使用的都是这个 KEY 。

```bash
redis-cli -a sinogold flushdb
Warning: Using a password with '-a' option on the command line interface may not be safe.
OK
redis-cli -a sinogold dbsize
Warning: Using a password with '-a' option on the command line interface may not be safe.
(integer) 0
#### 通过 -r 10 参数，设置 KEY 的随机范围为 -r 10 。
redis-benchmark -a sinogold -t set -n 1000 -q -r 10
SET: 66666.67 requests per second

redis-cli -a sinogold keys \*
Warning: Using a password with '-a' option on the command line interface may not be safe.
 1) "key:000000000005"
 2) "key:000000000004"
 3) "key:000000000009"
 4) "key:000000000002"
 5) "key:000000000007"
 6) "key:000000000008"
 7) "key:000000000006"
 8) "key:000000000001"
 9) "key:000000000000"
10) "key:000000000003"

```

通过 -r 参数，我们可以测试随机 KEY 的情况下的性能。

```bash
 redis-benchmark -t set,get,incr -n 1000000 -q -P 10 -a sinogold -r 10
SET: 675219.50 requests per second
GET: 640615.00 requests per second
INCR: 647249.19 requests per second
```

## jMeter (Redis Data Set)

由于 jmeter 本身并没有带有 Redis 的测试入口，我们需要去安装 Redis 插件。

### 下载

首先，我们下载 jmeter-plugins-manager-1.6.jar 文件，放到 jmeter 的 lib 的 ext 文件夹中，然后重启 jmeter。

下载地址是： [https://repo1.maven.org/maven2/kg/apc/jmeter-plugins-manager/1.6/jmeter-plugins-manager-1.6.jar](https://repo1.maven.org/maven2/kg/apc/jmeter-plugins-manager/1.6/jmeter-plugins-manager-1.6.jar)。

但是我用浏览器下载不下来, 用 wget 也不行, 后来我看这个地址是个 maven 的仓库,就把它放到一个 java 项目的依赖里面, 用 maven 下载下来用. 手动狗头.

```xml
        <dependency>
            <groupId>kg.apc</groupId>
            <artifactId>jmeter-plugins-manager</artifactId>
            <version>1.6</version>
        </dependency>
```

打开本地仓库的目录 localRepository/kg/apc/jmeter-plugins-manager/1.6 就可以复制过来了.

windows 的:  localRepository\kg\apc\jmeter-plugins-manager\1.6 .

### 安装

- 启动 jMeter , 点 Options 菜单, 点最下面的 Plugins Manager.
- 点选 Available Plugins 标签页. 在 Search... 输入框中输入 Redis. 按回车.
- 会显示找到一个 Redis Data Set , 然后勾选它前面的 checkbox, 再点击右下角的 Apply Changes and Restart jMeter 按钮
- 等待下载并重启 jMeter 进程.

```sh
Redis Data Set
Vendor: JMeter-Plugins.org
Allows reading test data from Redis storage
Documentation: https://jmeter-plugins.org/wiki/RedisDataSet/
What is new in version 0.5: fixed ssl config in redis dataset so it actually gets used
Maven groupId: kg.apc, artifactId: jmeter-plugins-redis, version: 0.5

一个截图
 
Location: /C:/works/local/apache-jmeter-5.4/lib/ext/jmeter-plugins-redis-0.5.jar
Libraries: [jedis, jmeter-plugins-cmn-jmeter, commons-pool2]
```

### 使用

- 新建一个测试计划. 例如叫 redis 保存.
- 在测试计划上点右键, 选 add , Threads (Users) , Thread Group . 命名为 redis-test-thread-group .
- 在 Thread Group 上点右键, 选 add, Config Element, jp@gc - Redis Data Set .

在 Data Configuration group 中添加主机的信息

#### Data Configuration

- Redis key：Redis 中的 key，Redis 数据库中列表（有序数据）或集（无序数据）的名称
- Variable Names：由数据集导出到测试元素的变量的名称（设置取出来的 value 存放在哪个变量中）
- Delimiter：存储在 Redis 列表或集合中的行中使用的分隔符（取出的 value 有多个值时，变量名之间的分隔符）
- Date Sources Type：数据源类型，有 List、Set 两种选择
- Recycle data on Flase： 数据是否重复使用

#### Connection Configuration

- Redis server host：Redis 服务器 IP 地址
- Redis server port：Redis 服务端口
- Timeout for connect in ms： 连接超时时间，默认 2000 ms
- Password for connection：连接 Redis 的密码
- Database：数据库名称，连接 Redis 的第几个数据库，默认为 0

#### Redis Pool Configuration

| 字段 | 用法| 默认值 |
| ---- | ---- | ---- |
| minIdle | 至少有多少个处于空闲状态的 Redis 实例 | 0 |
| maxIdle | 一个线程池最多有多少个处于空闲状态的 Redis 实例 | 10 |
| maxActive | 控制一个 pool 可分配多少个 Redis 实例，通过 pool.getResource()来获取；如果赋值为-1，则表示不限制；如果 pool 已经分配了 maxActive 个 jedis 实例，则此时 pool 的状态就成 exhausted | 20 |
| maxWait | 表示当 borrow 一个 Redis 实例时，最大的等待时间，如果超过等待时间，则直接抛出 JedisConnectionException | 30000 |
| whenExhaustedAction | 表示当 pool 中的 Redis 实例都被 allocated 完时，pool 要采取的操作；默认有三种 WHEN_EXHAUSTED_FAIL（表示无 Redis 实例时，直接抛出 NoSuchElementException）、WHEN_EXHAUSTED_BLOCK（则表示阻塞住，或者达到 maxWait 时抛出 JedisConnectionException）、WHEN_EXHAUSTED_GROW（则表示新建一个 jedis 实例，也就说设置的 maxActive 无用 | GROW |
| testOnBorrow | 在 borrow 一个 Redis 实例时，是否提前进行 alidate 操作；如果为 true，则得到的 Redis 实例均是可用的 | False |
| testOnReturn | 在 return 给 pool 时，是否提前进行 validate 操作 | False |
| testWhileIdle | 如果为 true，表示有一个 idle object evitor 线程对 idle object 进行扫描，如果 validate 失败，此 object 会被从 pool 中 drop 掉；这一项只有在 timeBetweenEvictionRunsMillis 大于 0 时才有意义 | False |
| timeBetweenEvictionRunsMillis | 表示 idle object evitor 两次扫描之间要 sleep 的毫秒数 | 30000 |
| numTestsPerEvictionRun | 表示 idle object evitor 每次扫描的最多的对象数 | 0 |
| minEvictableIdleTimeMillis | 表示一个对象至少停留在 idle 状态的最短时间，然后才能被 idle object evitor 扫描并驱逐；这一项只有在 timeBetweenEvictionRunsMillis 大于 0 时才有意义 | 60000 |
| softMinEvictableIdleTimeMillis | 在 minEvictableIdleTimeMillis 基础上，加入了至少 minIdle 个对象已经在 pool 里面了。如果为-1，evicted 不会根据 idle time 驱逐任何对象。如果 minEvictableIdleTimeMillis>0，则此项设置无意义，且只有在 timeBetweenEvictionRunsMillis 大于 0 时才有意义 | 60000 |

接下来我们添加调试取样器，在名称中引用 Redis 变量名称。然后，线程组循环次数设置多次。

右键点击线程组, 选 Add, Sampler, Debug Sampler .
右键点击线程组, 选 Add, Listener, View Results Tree .
点击线程组, 修改 Loo Count 为 5 .

添加 BeanShell Samplter

在 Redis 中创建一个 key = key:000000000000, value = string 数据, 放 jedis 的 jar 到 jMeter 的 lib 目录中. 写一个脚本:

```java
import redis.clients.jedis.Jedis;
import org.apache.commons.lang3.StringUtils;

String host = "47.114.98.28";
        int port = 6379;
        String password = "sinogold";
        int index = 0;
        String key = "key:000000000000";
        Jedis jedis = new Jedis(host, port);
        if (StringUtils.isNotBlank(password)) {
            jedis.auth(password);
        }
        jedis.select(index);
        for (int i = 0; i < 1; i++ ) {
            redisData = jedis.get(key);
            vars.put("redisData", redisData);
        }
        jedis.close();
```

## 参考链接

- [https://zhuanlan.zhihu.com/p/78034665](https://zhuanlan.zhihu.com/p/78034665)
- [http://testingpai.com/article/1619004387649](http://testingpai.com/article/1619004387649)

EOF
