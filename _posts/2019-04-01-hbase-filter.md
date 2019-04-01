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
- `NOT_EQUAL` : 不等于
- `GREATER_OR_EQUAL` : 大于等于
- `GREATER` : 大于
- `NO_OP` : 无操作

`SubstringComparator` 是一个比较器。这个比较器可以判断目标字符串是否包含锁指定的字符串。

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

            ResultScanner rs = table.getScanner(scan);
            for (Result result: rs) {
                String str = Bytes.toString(result.getValue(Bytes.toBytes("mycf"), Bytes.toBytes("name")));
                LOGGER.info("str ==> {}", str);
            }

        } catch (IOException e) {
            LOGGER.error("", e);
        }
    }

}

```

### 单列值过滤器 SingleColumn`ValueFilter`


// TODO: 待续




<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
