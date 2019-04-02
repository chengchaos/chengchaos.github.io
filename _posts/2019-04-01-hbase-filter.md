---
title: HBase 的过滤器
key: 20190330
tags: bigdata hbase
---

HBase 过滤器就是在 Get 或者 Scan 的时候过滤结果用的，可以理解为 HBase 中的 Where 语句。HBase 的过滤器会被序列化为可以在网络中传输的格式，然后分发给每个 RegionServer。在 Scan 的遍历过程中，不满足条件的结果就不会被返回个客户端。

<!--more-->

## 快速入门

HBase 过滤器需要实现 `Filter` 接口，HBase 同时还提供了 `FilterBase` 抽象类，其提供了 Filter 接口的默认实现，通常大部分的过滤器都继承自 FilterBase 类。过滤器不仅可以用于 Scan 还可以用于 Get。

### 值过滤器 `ValueFilter`

我们使用 SQL 的时候，用的最多的语句就是查某个列大于、等于、小于某个值了，HBase 中需要用过滤器来实现。


```java

    @Test
    public void valueFilterTest() {

        Filter filter = new ValueFilter(CompareFilter.CompareOp.EQUAL, new SubstringComparator("Wang"));

        ResultScanner scanner = HBaseUtil.getScanner("mytable", "row1", "row100", new FilterList(filter));

        for (Result result :scanner) {
            String str = Bytes.toString(result.getValue(Bytes.toBytes("mycf"), Bytes.toBytes("name")));
            LOGGER.info("str ==> {}", str);
        }
    }
```




`CompareFilter` 中包含一个枚举类 `CompareOp`，其中有以下值：

- `LESS` : 小于
- `LESS_OR_EQUAL` : 小于等于
- `EQUAL` : 等于
- `NOT_EQUAL` : 不等于
- `GREATER_OR_EQUAL` : 大于等于
- `GREATER` : 大于
- `NO_OP` : 无操作

`SubstringComparator` 是一个比较器。这个比较器可以判断目标字符串是否包含锁指定的字符串。


一个例子：


```java
package com.example.myscala002.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.CanUnbuffer;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.CompareFilter;
import org.apache.hadoop.hbase.filter.Filter;
import org.apache.hadoop.hbase.filter.SubstringComparator;
import org.apache.hadoop.hbase.filter.ValueFilter;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.hdfs.DistributedFileSystem;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;


public class ValueFilterTest {

    private static final Logger LOGGER = LoggerFactory.getLogger(ValueFilterTest.class);


    private Configuration createConfiguration() {

        Configuration config = HBaseConfiguration.create();
        try {

            // 添加必要的配置文件： hbase-site.xml, core-site.xml
            config.addResource(new Path(ClassLoader.getSystemResource("hbase-site.xml").toURI()));
            config.addResource(new Path(ClassLoader.getSystemResource("core-site.xml").toURI()));
            return config;
        } catch (Exception e) {
            LOGGER.error("", e);
            throw new RuntimeException(e);
        }
    }

    @Test
    public void valueFilterTest() {

        DistributedFileSystem distributedFileSystem;
        CanUnbuffer canUnbuffer;
        // org.apache.hadoop.hdfs.DistributedFileSystem
        try (Connection connection = ConnectionFactory.createConnection(createConfiguration())) {

            Table table = connection.getTable(TableName.valueOf("mytable"));

            Scan scan = new Scan();
            Filter filter = new ValueFilter(CompareFilter.CompareOp.EQUAL, new SubstringComparator("Wang"));

            scan.setFilter(filter);

            try (ResultScanner rs = table.getScanner(scan)) {

                String str;
                for (Result result: rs) {
                    str = Bytes.toString(result.getValue(Bytes.toBytes("mycf"), Bytes.toBytes("name")));
                    LOGGER.info("mycf:name ==> {}", str);
                    str = Bytes.toString(result.getValue(Bytes.toBytes("mycf"), Bytes.toBytes("teacher")));
                    LOGGER.info("mycf:teacher ==> {}", str);
                }
            }

        } catch (IOException e) {
            LOGGER.error("", e);
        }
    }

}

```

值过滤器会过滤所有的列，所以，会把所有包含我们设定值的所有行都返回回来。

```bash
20:08:03.367 + INFO  c.e.m.h.ValueFilterTest -line 70 - mycf:name ==> billyWangpaul - [main]
20:08:03.368 + INFO  c.e.m.h.ValueFilterTest -line 72 - mycf:teacher ==> null - [main]
20:08:03.368 + INFO  c.e.m.h.ValueFilterTest -line 70 - mycf:name ==> andyWang - [main]
20:08:03.369 + INFO  c.e.m.h.ValueFilterTest -line 72 - mycf:teacher ==> null - [main]
20:08:03.369 + INFO  c.e.m.h.ValueFilterTest -line 70 - mycf:name ==> kateWang - [main]
20:08:03.369 + INFO  c.e.m.h.ValueFilterTest -line 72 - mycf:teacher ==> null - [main]
20:08:03.370 + INFO  c.e.m.h.ValueFilterTest -line 70 - mycf:name ==> null - [main]
20:08:03.370 + INFO  c.e.m.h.ValueFilterTest -line 72 - mycf:teacher ==> WangQiang - [main]
```


### 单列值过滤器 `SingleColumn`ValueFilter`


单值过滤器可以当作值过滤器的升级版，通过前两个参数指定要比较的列，比如：


```java

    private void withTable(String tableName, Consumer<Table> tableConsumer) {

        Configuration conf = this.createConfiguration();
        try (Connection conn = ConnectionFactory.createConnection(conf)) {
            Table table = conn.getTable(TableName.valueOf(Bytes.toBytes(tableName)));
            tableConsumer.accept(table);
        } catch (IOException e) {
            LOGGER.error("", e);
        }
    }


    @Test
    public void singleColumnValueFilterTest() {


        Filter filter = new SingleColumnValueFilter(Bytes.toBytes("mycf"),
                Bytes.toBytes("name"),
                CompareFilter.CompareOp.EQUAL,
                new SubstringComparator("Wang"));

        final Scan scan = new Scan().setFilter(filter);

        this.withTable("mytable", table -> {

            try ( ResultScanner rs = table.getScanner(scan)) {

                for (Result result: rs) {
                    String str = Bytes.toString(result.getValue(Bytes.toBytes("mycf"), Bytes.toBytes("name")));
                    LOGGER.info("str ==> {}", str);
                }
            } catch (IOException e) {
                LOGGER.error("", e);
            }
        });

    }
```

这样就可以过滤指定的列，但是单列值过滤器发现改行记录没有想比较的列的时候，会把整行数据放到结果集中，所以，要安全使用单列值过滤器，务必保证<span style="color:red">**每行记录都包含**</span>将要比较的列。


如果无法保证，可以这样：

- 在遍历结果集的时候，再次判断结果中是否包含我们要比较的列，没有就不适用这条记录。
- 使用过滤器列表 （FilterList),将列族过滤器（FamilyFilter）、列过滤器（QualifierFilter）和值过滤器放入过滤器列表，同时进行过滤。

```java



    @Test
    public void filterListTest01() {

        FilterList filterList = new FilterList(FilterList.Operator.MUST_PASS_ALL);

        /* 只有列族为 mycf 的记录才放入结果集 */
        Filter familyFilter = new FamilyFilter(CompareFilter.CompareOp.EQUAL,
                new BinaryComparator(Bytes.toBytes("mycf")));

        filterList.addFilter(familyFilter);
        /*只有列为 teacher 的记录才放入结果集 */

        Filter colFilter = new QualifierFilter(CompareFilter.CompareOp.EQUAL,
                new BinaryComparator(Bytes.toBytes("teacher")));
        filterList.addFilter(colFilter);

        /* 只有包含 Wang 的记录才放入结果集 */
        Filter valueFilter = new ValueFilter(CompareFilter.CompareOp.EQUAL,
                new SubstringComparator("Wang"));
        filterList.addFilter(valueFilter);

        Scan scan = new Scan().setFilter(filterList);

        this.withTable("mytable", table ->{
            try (ResultScanner rs = table.getScanner(scan)) {

                for (Result result :rs) {
                    String str = Bytes.toString(result.getValue(Bytes.toBytes("mycf"), Bytes.toBytes("teacher")));
                    LOGGER.info("mycf:teacher ==> {}", str);
                }
            } catch (IOException e){ {

                LOGGER.error("", e);
            }}
        });
    }

```


这个方案的缺点就是：使用了 3 个过滤器，执行的速度比直接使用单列值过滤器更慢。所以实际工作中可以针对在每行中都存在的列使用单列值过滤器，对于不确定是否存在的列使用过滤器列表。


## 比较运算符入门

###　字符串完全匹配

使用 `BinaryComparator` 比较器.

```java

        Filter filter = new SingleColumnValueFilter(Bytes.toBytes("mycf"),
                Bytes.toBytes("name"),
                CompareFilter.CompareOp.EQUAL,
                new BinaryComparator(Bytes.toBytes("Wang")));
```


### 大于

使用 `CompareFilter.CompareOp.GREATER` 比较操作.

```java

        Filter filter = new SingleColumnValueFilter(Bytes.toBytes("mycf"),
                Bytes.toBytes("name"),
                CompareFilter.CompareOp.GREATER,
                new BinaryComparator(Bytes.toBytes("Wang")));
```

### 比较数字


如果用个 hbase shel 来保存数字的话,其实保存起来的都是字符串,如果用 BinaryComparator 来比较, 就会出现 22 比 3 小,所以不要用 hbase shell 来插入数字类型, 为了保证我们存储的真的是数字形式的数据, 要使用 Java api 来保存数字类型.




## 比较器

所有的比较器都继承自抽象类 `ByteArrayComparable` ,除了前面的 `BinaryComparator` 和 `SubstringComparator` ,HBase 还有以下一些比较器:

- 正则表达式比较器  `RegexStringComparator`: 与之搭配使用的比较关系为 `CompareOp.EQUAL`
- 空值比较器 `NullComparator` : 一般和 `SingleColumnValueFilter` 一起使用, 搭配关系为 `CompareOp.EQUAL` 或 `CompareOp.NOT_EQUAL`
- 数字比较器 `LongComparator` 
- 比特位比较器 `BitComparator` : 其构造器需要传入两个参数: 要计算的比特数组和计算方法. 计算方法的可选值由 `BitComparator.BitwiseOp` 枚举类提供,有 `AND`, `OR`, `XOR` 可选。 HBase 会将你传入的比特数组通过你要求的计算方法跟数据库中的值进行比特位计算。当比较关系为 EQUAL 的时候，结果集中包含的那些运算结果为非全 0 的结果。当比较关系为 NOT_EQUAL 的时候，只有运算结果为全 0 的记录会被放入结果集。
- 字节数组前缀比较器 `BinaryPrefixComparator` : 你提供一段字节数组,然后字节数组前缀比较器会帮你挑出所有以这段字节数组开头的记录.


## 分页过滤器 `PageFilter`

分页过滤器的构造函数是:

```java
  /**
   * Constructor that takes a maximum page size.
   *
   * @param pageSize Maximum result size.
   */
  public PageFilter(final long pageSize) {
    Preconditions.checkArgument(pageSize >= 0, "must be positive %s", pageSize);
    this.pageSize = pageSize;
  }
```

参数 pageSize 即每页的记录数.

```java


    @Test
    public void PageFilterTest() {

        Filter filter = new PageFilter(2);

        final Scan scan = new Scan().setFilter(filter);

        this.withTable("mytable", table -> {

            try ( ResultScanner rs = table.getScanner(scan)) {

                for (Result result: rs) {
                    String str = Bytes.toString(result.getValue(Bytes.toBytes("mycf"), Bytes.toBytes("name")));
                    String rowkey = Bytes.toString(result.getRow());
                    LOGGER.info("rowkey ==>{}, mycf:name ==> {}", rowkey, str);
                }
            } catch (IOException e) {
                LOGGER.error("", e);
            }
        });


    }

```

PageFilter 只能实现相当于 limit n 的功能,如果要翻页,需要把上一次翻页的最后一个 rowkey 记录下来,并最为下一次 scan 的 startRowKey.


```java

    private byte[] printResult(ResultScanner rs) {

        byte[] lastRowKey = null;

        for (Result result: rs) {
            byte[] rowKey = result.getRow();
            String name = Bytes.toString(result.getValue(Bytes.toBytes("mycf"), Bytes.toBytes("name")));
            LOGGER.info("name ==> {}", name);
            lastRowKey = rowKey;
        }

        return lastRowKey;
    }



    @Test
    public void PageFilterTest02() {

        Filter filter = new PageFilter(2);

        final Scan scan = new Scan().setFilter(filter);

        this.withTable("mytable", table -> {

            byte[] lastRowKey;

            try ( ResultScanner rs = table.getScanner(scan)) {

                lastRowKey = this.printResult(rs);

                // 第二页
                byte[] startRowKey = Bytes.add(lastRowKey, new byte[1]);
                scan.setStartRow(startRowKey);


            } catch (IOException e) {
                LOGGER.error("", e);
            }

            // 第二页
            try ( ResultScanner rs = table.getScanner(scan)) {

                lastRowKey = this.printResult(rs);

                byte[] startRowKey = Bytes.add(lastRowKey, new byte[1]);
                scan.setStartRow(startRowKey);


            } catch (IOException e) {
                LOGGER.error("", e);
            }
        });


    }
```
> 由于我们的 Filter 是被分发到不同的 Region 中去执行的，所以各个 RegionScanner 并不知道别的 RegionScanner 获取到了几条数据，所以如果你的数据是分布在不同的 RegionServer 上的话，返回的结果会比你定义的分页数更大。
> 
> 如果想要精确地只返回 n 条结果的话，在 Scan 返回结果后，还需要对数据进行后续处理。比如对结果进行排序，并丢弃超过 n 条的记录。

## 过滤器列表


构造方法:


```java


  /**
   * Constructor that takes a set of {@link Filter}s. The default operator
   * MUST_PASS_ALL is assumed.
   *
   * @param rowFilters list of filters
   */
  public FilterList(final List<Filter> rowFilters) {
    if (rowFilters instanceof ArrayList) {
      this.filters = rowFilters;
    } else {
      this.filters = new ArrayList<Filter>(rowFilters);
    }
  }

  /**
   * Constructor that takes an operator.
   *
   * @param operator Operator to process filter set with.
   */
  public FilterList(final Operator operator) {
    this.operator = operator;
  }

```


过滤器列表内的执行是有先后顺序的.


### 多个过滤器间的 and / or 关系

```java
  /**
   * Constructor that takes a set of {@link Filter}s and an operator.
   *
   * @param operator Operator to process filter set with.
   * @param rowFilters Set of row filters.
   */
  public FilterList(final Operator operator, final List<Filter> rowFilters) {
    this.filters = new ArrayList<Filter>(rowFilters);
    this.operator = operator;
  }
```

第一个参数被称作运算符, 它代表过滤器列表中的各个过滤器对于扫描器的扫描过程所起的左右, 取值如下:

- `MUST_PASS_ALL` : (默认), 所有都通过才加入结果集, 相当于 AND 连接
- `MUST_PASS_ONE` : 只要一个过滤器通过,就加入结果集,相当于 OR 连接



###过滤器列表嵌套

过滤器列表也是一种过滤器，所以我们可以把一个过滤器列表设置到另外一个过滤器列表中来实现嵌套查询。



## RowKey 过滤器

### 行过滤器 `RowFilter`


```java


        Filter filter = new RowFilter(CompareFilter.CompareOp.GREATER,
                new BinaryComparator(Bytes.toBytes("rowe")));
        
```

可以搭配 `SubstringComparator` 和 `RegexStringComparator` 等比较器一起使用.



### 多行范围过滤器 `MultiRowRangeFilter`

是行过滤器的扩展版本, 可以使用其来指定多个 RowKey 搜索范围. 构造函数需要传入多个 RowKey 范围实例作为参数:


```java
MultiRowRangeFilter(List<RowRange> list)
```

RowRange 实例可以包含起始行(StartRow),结束行(EndRow)信息,

### 前缀过滤器 `PrefixFilter`


准确的说应该叫做行键前缀过滤器, 其可以根据 RowKey 的前缀进行匹配.


> 就算用了前缀过滤器也依然要结合上 STARTROW 使用，否则 scan 还是会从第一条记录开始扫描，浪费了大量的性能。



### 模糊行键过滤器 'FuzzyRowFilter'

```java

public FuzzyRowFilter(List<Pair<byte[], byte[]>>fuzzyKeysData)

```

fuzzyKeysData 是用于模糊匹配的表达式. 模糊匹配的表达式由两个部分组成: 行键和行键掩码(fuzzy info).

- 行键: 对于需要模糊匹配的字符所在的位置, 可以使用任意字符.
- 行键掩码: 行键掩码的长度必须和 RowKey 相同, 在需要模糊匹配的位置标记为 1, 其他位置标记为 0.


```java
FuzzyRowFilter filter = new FuzzyRowFilter(Arrays.asList(
    new Pair<byte[], byte[]>(
        Bytes.toBytesBinary("2016_??_??_4567"),
        new byte[]{0, 0, 0, 0, 0, 1, 1, 0, 1, 1, 0, 0, 0, 0, 0})));
```

### 包含结尾过滤器 

默认 scan 扫描数据, 如果使用 `STOPROW` 来指定终止行, 结果中并不会包含终止行. 如果想包含, 可以:

- 在终止行的 rowkey 上增加一个字节的数据,然后把增加了一个字节的 rowkey 作为 `STOPROW`.
- 使用 `InclusiveStopFilter`

### 随机行过滤器

`RandomRowFilter` , 是哦用于数据分析时对系统数据进行采样.




## 列过滤器

### 列族过滤器

`FamilyFilter` 


### 列过滤器

`QualifierFilter`

### 依赖过滤去

### 列前缀过滤器

`ColumnPrefixFilter`

### 多列前缀过滤器

`MultipleColumnPrefixFilter`


### 列键过滤器

`KeyOnlyFilter` 


```java
CellUtil.cloneQualifier(cell);


for (Result r :rs) {
    List<Cell> cells = r.listCells();
    List<String> strList = new ArrayList<>();
    byte[] rowkey = r.getRow();
    strList.add("row="+ Bytes.toString(rowkey));
    for (Cell cell :cells) {
        strList.add("column="+ new String(CellUtil.cloneQualifier(cell)));
    }
    Sytstem.out.println(StrintUtils.join(strList, ", "));
}
```


### 首次列键过滤器

`FirstKeyOnlyFilter`

### 列名范围过滤器

 `ColumnRangeFilter`


### 列数量过滤器

`ColumnCountGetFilter`


### 列翻页过滤器

`ColumnPaginationFilter`


## 单元格过滤器

### 时间戳过滤器

`TimestampsFilter`

### 装饰过滤器

### 全匹配过滤器


## 自定义过滤器



<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
