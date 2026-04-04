---
title: Solr 学习笔记
key: 2021-12-06
tags: solr
---

有些事情，你没有办绕过去。

<!--more-->

## 安装

官网： [https://solr.apache.org/](https://solr.apache.org/)

前置条件： 

- java8
- solr

我们使用 `SOLR_INSTALL` 环境变量指向 solr 的安装位置，因为这是我从 *Solr 实战*这本书中学来的。

```bash
cd $SOLR_INSTALL
bin/solr start
*** [WARN] ***  Your Max Processes Limit is currently 2784. 
 It should be set to 65000 to avoid operational disruption. 
 If you no longer wish to see this warning, set SOLR_ULIMIT_CHECKS to false in your profile or solr.in.sh
Waiting up to 180 seconds to see Solr running on port 8983 [-]  
Started Solr server on port 8983 (pid=5079). Happy searching!

bin/solr status

Found 1 Solr nodes: 

Solr process 5079 running on port 8983
{
  "solr_home":"/Users/chengchao/apps/solr/server/solr",
  "version":"8.11.0 e912fdd5b632267a9088507a2a6bcbc75108f381 - jpountz - 2021-11-09 14:08:51",
  "startTime":"2021-12-06T08:30:54.256Z",
  "uptime":"0 days, 0 hours, 3 minutes, 7 seconds",
  "memory":"74.3 MB (%14.5) of 512 MB"}


```



更多的去看 README.txt 

http://localhost:8983/solr/#/



## 配置

大部分与 Solr 有关的配置都是在下面这三个 xml 文件中。

- solr.xml 定义管理、日志、分片和 SolrCloud 有关的属性。
- solrconfig.xml 定义 Solr 内核的主要配置。
- schema.xml 定义索引结构、包括字段及其数据类型。

## solrconfig.xml



## schema.xml

schema.xml 文档由三个主要部分组成：

- `<fields>` 元素包含 `<field>` 和 `<dynamicField>` 用来定义文档的基本结构。
- 其他元素，如 `<uniqueKey>` 和 `<copyField>` 位于 `<field>` 元素之后。
- `<types>` 元素下面的字段类型包括 Solr 能够处理的日期、数字和文本字段。

schema.xml 的 `<fields>` 定义了文档中所用到的 field 元素，Solr 根据这些字段定义来调用合适的字段分析器，将字段内容解析为词项，继而添加到到拍索引中。



so so .

## 参考

- [PostgreSQL新手入门](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)
- [openSUSE 安装 PostgreSQL](https://segmentfault.com/a/1190000010032424)
- [Java连接PostgreSQL数据库](https://www.yiibai.com/postgresql/postgresql_java.html)

EOF

