---
title: Spark 学习笔记04 Spark-SQL
key: 20190408
tags: bigdata spark saprk-sql
---

Spark SQL 提供了一下三大功能：

- 从结构化数据源（JSON、Hive、Parquet）中读取数据
- 使用 SQL 语句进行查询
- SQL 和代码整合



<!--more-->

为实现这些功能，Spark SQL 提供了一种特殊的 RDD 叫 **SchemaRDD**。SchemaRDD 是存放 **Row** 对象的 RDD，每个 Row 对象代表一行记录。

SchemaRDd 还包含记录的结构信息（数据字段）。在 SchemaRDD 内部可以利用结构信息更加高效地存储数据。SchemaRDD 支持运行 SQL 查询。

SchemaRDD 可以从外部数据源创建，也可以从查询结果或普通 RDD 中创建。


在 1.3.0 以及后续版本中， SchemaRDD 已经被 DataFrame 所取代。

The largest change that users will notice when upgrading to Spark SQL 1.3 is that SchemaRDD has been renamed to DataFrame. This is primarily because DataFrames no longer inherit from RDD directly, but instead provide most of the functionality that RDDs provide though their own implementation. DataFrames can still be converted to RDDs by calling the .rdd method.

In Scala there is a type alias from SchemaRDD to DataFrame to provide source compatibility for some use cases. It is still recommended that users update their code to use DataFrame instead. Java and Python users will need to update their code.

当升级到 Spark SQL 1.3 时，用户会注意到的最大变化是 SchemaRDD 被重命名为 DataFrame。这主要是因为 DataFrames 不再直接从 RDD 继承，而是通过自己的实现提供 RDDs 提供的大部分功能。仍然可以通过调用 `.rdd` 方法将数据流转换为 RDDs。

在 Scala 中，有一个从 SchemaRDD 到 DataFrame 的类型别名，用于为某些用例提供源代码兼容性。仍然建议用户更新代码以使用 DataFrame。Java 和 Python 用户将需要更新他们的代码。













<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
