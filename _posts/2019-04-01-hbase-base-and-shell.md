---
title: HBase 基础知识和 Shell
key: 20190330
tags: bigdata hbase
---

HBase 采用的是 Key/ Value 的存储方式， 这意味着，即使随着数据量增大， 也几乎不会导致查询能の下降。

> HBase 并不快， 只是当数据量很大的时候它慢的不明显。

凡事都不可能只有优点而没有缺点。数据分析是 HBase 的弱项，因为对于 HBase 乃至整个 NoSQL 生态圈来说， 基本上都是不支持表关联的。 当你想实现 group by 或者 order by 的时候，你会发现，你需要写很多的代码来实现 MapReduce。 因此，请不要盲目地使用 HBase。


当你的情况大体上符合以下任意一种的时候：

- 主要需求是数据分析，比如做报表。 
- 单表数据量不超过千万。 

请不要使用 HBase， 使用 MySQL 或者 Oracle 之类的产品可以让你的脑细胞免受折磨。

当你的情况是：

- 单表数据量超千万，而且并发还挺高。
- 数据分析需求较弱，或者不需要那么灵活或者实时。 

请使用 HBase，它不会让你失望的。

> 杨曦. HBase不睡觉书. 清华大学出版社. Kindle 版本. 

<!--more-->

## 部署架构

HBase 有两种服务器： Master 服务器和 RegionServer 服务器。

一个 HBase 集群有一个 Master 服务器和几个 RegionServer 服务器。Master 服务器负责维护表结构信息，实际的数据都存储在 RegionServer 服务器上。RegionServer 保存的表数据直接存储在 Hadoop 的 HDFS 上。


HBase 有一点很特殊： 客户端获取数据由客户端直连 RegionServer 的， 所以你会发现 Master 挂掉 之后依然可以查询数据， 但就是不能新建表了。

RegionServer 非常依赖 ZooKeeper 服务， 可以说没有 ZooKeeper 就没有 HBase。 ZooKeeper 在 HBase 中扮演的角色类似一个管家。 ZooKeeper 管理了 HBase 所有 RegionServer 的信息， 包括具体的数据段存放在 哪个 RegionServer 上。

客户端每次与 HBase 连接， 其实都是先与 ZooKeeper 通信， 查询出哪个 RegionServer 需要连接， 然后再连接 RegionServer。


### 什么是 Region

Region 就是一段数据的集合。 HBase 中的表一般拥有一个到多个 Region。 Region 有以下特性：

- Region 不能跨服务器， 一个 RegionServer 上有一个或者多个 Region。 
- 数据量小的时候， 一个 Region 足以存储所有数据； 但是， 当数据量大的时候，HBase 会拆分 Region。 
- 当 HBase 在进行负载均衡的时候， 也有可能会从一台 RegionServer 上把 Region 移动到 另一台 RegionServer 上。 
- Region 是基于 HDFS 的， 它的所有数据存取操作都是调用了 HDFS 的客户端接口来实现的。

### 什么是 RegionServer

RegionServer 就是存放 Region 的容器， 直观上说就是服务器上的一个服务。

## 存储架构

最基本的存储单位是列（`column`）， 一个列或者多个列形成一行（`row`）。 传统数据库是严格的行列对齐。比如这行有三个列 a、 b、 c， 下一行肯定也有三个列 a、 b、 c。 而在 HBase 中， 这一行有三个列 a、 b、 c， 下一个行也许是有 4 个列 a、 e、 f、 g。 在 HBase 中， 行跟行的列可以完全不一样，这个行的数据跟另外一个行的数据也可以存储在不同的机器 上， 甚至同一行内的列也可以存储在完全不同的机器上！ 

每个行（`row`）都拥有唯一的行键（`row key`）来标定这个行的唯一性。每个列都有多个版本，多个版本的值存储在单元格（`cell`）中。若干个列又可以被归类为一个**列族**。


### 什么是 RowKey

RowKey 是完全由用户指定的一串不重复的字符串。

HBase 无法根据某个 column 来排序，系统永远是根据 RowKey 来排序。 RowKey 就是决定 row 存储顺序的唯一凭证。

RowKey 使用字典规则排序。

插入数据时，如果存在相同的 RowKey，则值会被覆盖。

旧值会放在 Cell 的历史纪录里面，查找旧值需要使用版本号来查找。


### Cell

Cell 时数据存储的最小单位，一个列上可以存储多个版本的 Cell。


### 列族（Column Family）

若干个列 （Column）可以组成一个列族（Column Family）

在建表的时候不需要指定列。因为列时可变的，唯一需要只当的是列族。

此外，表的很多属性，比如过期时间，数据块缓存以及是否压缩等都是定义在列族上，而不是定义在表上或者列上。同一个表里面的不同列族可以有完全不同的属性配置，但是同一个额列族内的所有列都会有相同的属性配置。

HBase 中一个列的名称前面总是带着它所属的列族，例如：

`brother:age`, `brother:name`

列族存在的意义是：HBase 会把相同列族的列尽量放在同一台机器上。

> 一个表要设置多少个列族比较合适？官方建议是：“越少越好”。


确定一条结果的表达式是：行键：列族：列：版本号：

```
rowkey:column-family:column:version

```

不过，版本号可以省略，省略时 HBase 默认获取最后一个版本的数据返回。

每个列或者单元格的值都被赋予一个时间戳，这个时间戳是由系统指定，也可以由用户显示指定。

### Region 和行的关系

一个 Region 就是多个行的集合，Region 中的行按照 RowKey 排序。


## Shell

HBase 的 Shell 是由 JRuby 开发的命令行工具。进入方式是：

```bash
$ $HBASE_HOME/bin/hbase shell
2019-04-01 13:07:25,146 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/works/apps/hbase-1.2.11/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/works/apps/hadoop-2.6.0-cdh5.7.0/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.11, rca53d58f5b7abde0c189c9f78baf4246bddffac3, Fri Feb 15 18:12:16 CST 2019

hbase(main):001:0> 
```

### 用 `create` 命令建表

`create '表名称','列族名称'`


```bash
hbase(main):001:0> create 'test','cf'
0 row(s) in 3.2770 seconds

=> Hbase::Table - test
hbase(main):002:0> 
```

### 用 `list` 命令查看数据库中的表

```bash
hbase(main):002:0> list
TABLE
FileTable
test                                                           
2 row(s) in 0.0410 seconds
=> ["FileTable", "test"]
hbase(main):003:0> 
```

### 用 `describe` 命令查看表的属性

```bash
hbase(main):008:0> describe 'test'
Table test is ENABLED
test
COLUMN FAMILIES DESCRIPTION
{NAME => 'cf', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_
BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BL
OCKSIZE => '65536', REPLICATION_SCOPE => '0'}
1 row(s) in 0.1660 seconds
```
NAME 属性表示的是列族的名称而不是表的名称。后面的属性都是针对列族的而不是针对表的。HBase 的表只有少数的几个属性，大部分的属性都在列族上。

### 用 `alter` 新增列族

在生产环境中执行这个命令之前，最好先通用（disable）表。因为对列族的所有操作都会同步到所有拥有这个表的 RegionServer 上，执行这个命令的时候，可以看到总共有多少个 RegionServer，当前执行了几个 RegionServer。当有很多客户端都在连接着的时候，直接新增一个列族对性能影响较大。


```bash
hbase(main):009:0> 
hbase(main):010:0* 
hbase(main):011:0* 
hbase(main):012:0* 
hbase(main):013:0* alter 'test','cf2'
Updating all regions with the new schema...
0/1 regions updated.
1/1 regions updated.
Done.
0 row(s) in 3.2340 seconds

hbase(main):014:0> describe 'test'
Table test is ENABLED
test
COLUMN FAMILIES DESCRIPTION 
{NAME => 'cf', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_
BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BL
OCKSIZE => '65536', REPLICATION_SCOPE => '0'} 
{NAME => 'cf2', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA
_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', B
LOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                                                                  
2 row(s) in 0.0820 seconds

hbase(main):015:0> 
```

### 使用 `alter` 命令调整时间戳

```bash
hbase(main):017:0> alter 'test',{NAME=>'cf',VERSIONS=>5}
Updating all regions with the new schema...
0/1 regions updated.
1/1 regions updated.
Done.
0 row(s) in 3.0760 seconds

```

### 使用 `put` 命令插入数据

在 HBase 中，如果一行有 10 列，那存储一行的数据需要些 10 行语句，这是因为 HBase 中行的每一个列都存储在不同的位置，因此必须指定要存储在哪个单元格；而单元格需要根据表、行、列这几个维度来定位。

插入数据的时候必须告诉 HBase 要把数据插入到哪个表的哪个列族的哪个行的哪个列。因此，put 命令都很长:


`put '<表名称>','<RowKey值>','<列族：列>',<数据值>`


```bash
hbase(main):015:0> put 'test','row1','cf:name','程超'
0 row(s) in 0.3070 seconds

hbase(main):016:0> scan 'test'
ROW                           COLUMN+CELL
row1                         column=cf:name, timestamp=1554096150762, value=\xE7\xA8\x8B\xE8\xB6\x85
1 row(s) in 0.0680 seconds

hbase(main):017:0> 

```

如果使用 put 命令时不指定时间戳，系统会用当前时间来指定，我们可以使用自定义时间戳。：

```bash
hbase(main):021:0> put 'test','row2','cf:name','ted'
0 row(s) in 0.0500 seconds

hbase(main):022:0> put 'test','row2','cf:name','billy',2222222222222
0 row(s) in 0.0300 seconds
hbase(main):025:0> scan 'test'
ROW                           COLUMN+CELL  
 row1                         column=cf:name, timestamp=1554096150762, value=\xE7\xA8\x8B\xE8\xB6\x85   
 row2                         column=cf:name, timestamp=2222222222222, value=billy                          
2 row(s) in 0.0420 seconds
hbase(main):027:0> get 'test','row2',{COLUMN=>'cf:name',VERSIONS=>3}
COLUMN                        CELL                                                                              
 cf:name                      timestamp=2222222222222, value=billy                                              
 cf:name                      timestamp=1554096641246, value=ted                                                
 cf:name                      timestamp=2222222222, value=billy                                                 
3 row(s) in 0.0800 seconds
```

HBase 没有专门的一个列族的栏来显示列族这个属性，它总是把列族和列显示为“列族：列”的形式。


### 用 `scan` 查看数据

最简单的用法时：

`scan '<表名称>'`

不过实际环境下，因为表的数据太大了，都会使用起始行（`STARTROW`）和结束行（`ENDROW`）来限制记录的条数。

`STARTROW` 和 `ENDROW` 都是可选参数，可以不输入，如果 `ENDROW` 不输入，就从 `STARTROW` 开始一直显示到表尾；都不输入，就从表开头一直显示到表结尾。如果都写了，就是显示 `>= STARTROW 并且 < ENDROW` 的数据。



```bash

hbase(main):028:0> scan 'test',{STARTROW=>'row2'}
ROW                           COLUMN+CELL                                                                       
 row2                         column=cf:name, timestamp=2222222222222, value=billy                              
1 row(s) in 0.0600 seconds

hbase(main):029:0> scan 'test',{ENDWOR => 'row4'}
NameError: uninitialized constant ENDWOR

hbase(main):030:0> scan 'test',{ENDROW => 'row4'}
ROW                           COLUMN+CELL                                                                       
 row1                         column=cf:name, timestamp=1554096150762, value=\xE7\xA8\x8B\xE8\xB6\x85           
 row2                         column=cf:name, timestamp=2222222222222, value=billy                              
2 row(s) in 0.0560 seconds

```

### 使用 `get` 命令获取单元格数据

`get` 命令只能查询一个单元格的记录。

```bash
hbase(main):031:0> get 'test','row1','cf:name'
COLUMN                        CELL                                                                              
 cf:name                      timestamp=1554096150762, value=\xE7\xA8\x8B\xE8\xB6\x85                           
1 row(s) in 0.0790 seconds

hbase(main):032:0> get 'test','row2','cf:name'
COLUMN                        CELL                                                                              
 cf:name                      timestamp=2222222222222, value=billy                                              
1 row(s) in 0.0340 seconds

hbase(main):033:0> get 'test','row2',{COLUMN=>'cf:name',VERSIONS=>5}
COLUMN                        CELL                                                                              
 cf:name                      timestamp=2222222222222, value=billy                                              
 cf:name                      timestamp=1554096641246, value=ted                                                
 cf:name                      timestamp=2222222222, value=billy                                                 
3 row(s) in 0.0300 seconds

hbase(main):034:0> scan 'test',{VERSION=>5}
ROW                           COLUMN+CELL                                                                       
 row1                         column=cf:name, timestamp=1554096150762, value=\xE7\xA8\x8B\xE8\xB6\x85           
 row2                         column=cf:name, timestamp=2222222222222, value=billy                              
2 row(s) in 0.0610 seconds

```

### 使用 `delete` 命令删除单元格数据

delete 是删除单元格的命令


最简单的用法：

`delete '<表明成>','<rowkey>','<列族：列>'`

根据版本删除：

`delete '<表明成>','<rowkey>','<列族：列>',ts`


```bash
hbase(main):035:0> delete 'test','row2','cf:name',2222222222
0 row(s) in 0.0480 seconds
hbase(main):036:0> scan 'test',{VERSION=>5}
ROW                           COLUMN+CELL
row1                         column=cf:name, timestamp=1554096150762, value=\xE7\xA8\x8B\xE8\xB6\x85  
row2                         column=cf:name, timestamp=2222222222222, value=billy      
2 row(s) in 0.0640 seconds

hbase(main):037:0> 
```

HBase 的删除记录操作并不是真的删除了数据，而是放置了一个墓碑标记（tombstone marker），把这个版本连同之前的版本都标记为不可见了。

HBase 会在做自动合并（compation, HBase 整理存储文件时的一个操作，会把多个文件块合并成一个文件）的时候执行真的删除操作。

数据在真的删除之前还是可以查到的，需要在 scan 命令后跟上 `RAW=>true` 参数和适当的 VERSIONS 参数。

```bash
hbase(main):038:0* scan 'test',{VERSIONS=>5,RAW=>true}
ROW                           COLUMN+CELL
 row1                         column=cf:name, timestamp=1554096150762, value=\xE7\xA8\x8B\xE8\xB6\x85
 row2                         column=cf:name, timestamp=2222222222222, value=billy
 row2                         column=cf:name, timestamp=1554096641246, value=ted
 row2                         column=cf:name, timestamp=2222222222, type=Delete
 row2                         column=cf:name, timestamp=2222222222, value=billy
2 row(s) in 0.0500 seconds

hbase(main):039:0> 
```

### 使用 'deleteall` 命令删除整行记录

`deleteall '<表名称>','<rowkey>'`


### 使用 `disable` 命令停用表

表在删除之前需要先使其停用，因为可能好多客户端正在连着，也有可能 HBase 正在做合并或者分裂操作。因此需要先做一个 disable 的操作，意思时把这个表停用并且下线。

```bash
hbase(main):039:0>  disable 'test'
0 row(s) in 2.3450 seconds
hbase(main):040:0> scan 'test'
ROW                           COLUMN+CELL                                                                       

ERROR: test is disabled.
......
```

在 没有什么数据或者没有什么人使用的情况下这条命令执行得很快，但如果在系统已经上线了，并且负载很大的情况下 disable 命令会执行得很慢， 因为 disable 要通知所有的 RegionServer 来下线这个表，并且有很多涉及该表的 操作需要被停用掉，以保证该表真的已经完全不参与任何工作了。当停用掉一个 表后， 可以用 scan 测试一下表是不是真的被关闭了.

### 使用 `drop` 命令删除表

停用后的表，可以用 drop 命令删除：

```bash
hbase(main):041:0> drop 'test'
0 row(s) in 1.3100 seconds

hbase(main):042:0> list
TABLE                                                                                                           
FileTable                                                                                                       
1 row(s) in 0.0520 seconds

=> ["FileTable"]
```


### 使用 `help` 命令获得帮助


```bash

hbase(main):086:0* help 'get'
Get row or cell contents; pass table name, row, and optionally
a dictionary of column(s), timestamp, timerange and versions. Examples:

  hbase> get 'ns1:t1', 'r1'
  hbase> get 't1', 'r1'
  hbase> get 't1', 'r1', {TIMERANGE => [ts1, ts2]}
  hbase> get 't1', 'r1', {COLUMN => 'c1'}
  hbase> get 't1', 'r1', {COLUMN => ['c1', 'c2', 'c3']}
  hbase> get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1}
  hbase> get 't1', 'r1', {COLUMN => 'c1', TIMERANGE => [ts1, ts2], VERSIONS => 4}
  hbase> get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1, VERSIONS => 4}
  hbase> get 't1', 'r1', {FILTER => "ValueFilter(=, 'binary:abc')"}
  hbase> get 't1', 'r1', 'c1'
  hbase> get 't1', 'r1', 'c1', 'c2'
  hbase> get 't1', 'r1', ['c1', 'c2']
  hbase> get 't1', 'r1', {COLUMN => 'c1', ATTRIBUTES => {'mykey'=>'myvalue'}}
  hbase> get 't1', 'r1', {COLUMN => 'c1', AUTHORIZATIONS => ['PRIVATE','SECRET']}
  hbase> get 't1', 'r1', {CONSISTENCY => 'TIMELINE'}
  hbase> get 't1', 'r1', {CONSISTENCY => 'TIMELINE', REGION_REPLICA_ID => 1}

Besides the default 'toStringBinary' format, 'get' also supports custom formatting by
column.  A user can define a FORMATTER by adding it to the column name in the get
specification.  The FORMATTER can be stipulated: 

 1. either as a org.apache.hadoop.hbase.util.Bytes method name (e.g, toInt, toString)
 2. or as a custom class followed by method name: e.g. 'c(MyFormatterClass).format'.

Example formatting cf:qualifier1 and cf:qualifier2 both as Integers: 
  hbase> get 't1', 'r1' {COLUMN => ['cf:qualifier1:toInt',
    'cf:qualifier2:c(org.apache.hadoop.hbase.util.Bytes).toInt'] } 

Note that you can specify a FORMATTER by column only (cf:qualifier).  You cannot specify
a FORMATTER for all columns of a column family.
    
The same commands also can be run on a reference to a table (obtained via get_table or
create_table). Suppose you had a reference t to table 't1', the corresponding commands
would be:

  hbase> t.get 'r1'
  hbase> t.get 'r1', {TIMERANGE => [ts1, ts2]}
  hbase> t.get 'r1', {COLUMN => 'c1'}
  hbase> t.get 'r1', {COLUMN => ['c1', 'c2', 'c3']}
  hbase> t.get 'r1', {COLUMN => 'c1', TIMESTAMP => ts1}
  hbase> t.get 'r1', {COLUMN => 'c1', TIMERANGE => [ts1, ts2], VERSIONS => 4}
  hbase> t.get 'r1', {COLUMN => 'c1', TIMESTAMP => ts1, VERSIONS => 4}
  hbase> t.get 'r1', {FILTER => "ValueFilter(=, 'binary:abc')"}
  hbase> t.get 'r1', 'c1'
  hbase> t.get 'r1', 'c1', 'c2'
  hbase> t.get 'r1', ['c1', 'c2']
  hbase> t.get 'r1', {CONSISTENCY => 'TIMELINE'}
  hbase> t.get 'r1', {CONSISTENCY => 'TIMELINE', REGION_REPLICA_ID => 1}
```


## shell 命令列表

// TODO: 待续.







<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
