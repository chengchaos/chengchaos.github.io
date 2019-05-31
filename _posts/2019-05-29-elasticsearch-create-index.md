---
title: elasticsearch 创建索引
key: 2019-05-29
tags: elasticsearch-create-index
---

2000 年一月一日的样子我现在还记得.

<!--more-->
## 自动创建索引

https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html

默认情况下, 如果操作的索引不存在会被自动创建出来, 并且应用上一个配置好的 index templates. 此外, 若果映射不粗在, 索引操作也会创建一个来. 如果有需要, 新的字段和对象将会自动添加到映射定义中. 

自动创建索引的功能是由 `action.auto_create_index` 属性来设置的. 其默认值为 `true` 意味着索引总是会自动创建. 只允许对匹配某些模式的索引自动创建，方法是将该设置的值更改为这些模式的逗号分隔列表, 也可以通过在列表中使用 `+` 或 `-` 前缀模式显式地允许和禁止。如果设置为 `false` 则被禁用.

```javascript
// PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "twitter,index10,-index1*,+ind*" 
    }
}
// 允许twitter, index10 自动创建
// 不允许-index1 开头的自动创建
// 注意是由先后顺序的.

// PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "false" 
    }
}
// 完全禁止自动创建

// PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "true" 
    }
}
// 完全允许自动创建
```



## 保存原始数据


默认情况下，Elasticsearch 用 JSON 字符串来表示文档主体保存在 `_source` 字段中。像其他保存的字段一样，`_source` 字段也会在写入硬盘前压缩。

这几乎始终是需要的功能，因为：

- 搜索结果中能得到完整的文档 —— 不需要额外去别的数据源中查询文档
- 如果缺少 `_source` 字段，**部分更新** 请求不会起作用
- 当你的映射有变化，而且你需要重新索引数据时，你可以直接在 Elasticsearch 中操作而不需要重新从别的数据源中取回数据。
- 你可以从 _source 中通过 get 或 search 请求取回部分字段，而不是整个文档。
- 这样更容易排查错误，因为你可以准确的看到每个文档中包含的内容，而不是只能从一堆 ID 中猜测他们的内容。

即便如此，存储 _source 字段还是要占用硬盘空间的。假如上面的理由对你来说不重要，你可以用下面的映射禁用 _source 字段：

```
PUT /my_index
{
    "mappings": {
        "_source": {
            "enabled": false
        }
    }
}

// response
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "my_index"
}
```

在搜索请求中你可以通过限定 _source 字段来请求指定字段：

```
GET /_search
{
    "query":   { "match_all": {}},
    "_source": [ "title", "created" ]
}
```

如果你决定不再使用 _all 字段，你可以通过下面的映射禁用它：

```
PUT /my_index/_mapping/my_type
{
    "my_type": {
        "_all": { "enabled": false }
    }
}
```

## 映射

映射是定义文档及其包含的字段如何存储和索引的过程。

例如，使用映射来定义
- 哪些字符串字段应该被视为全文字段。
- 哪些字段包含数字、日期或地理位置。
- 日期值的格式。
- 用于控制动态添加字段的映射的自定义规则。

## 映射类型

每个索引都有一个映射类型，它决定文档将如何被索引。

## 字段数据类型

每个字段都有个数据类型, 可能是:

- 简单类型: `text`, `keyword`, `date`, `long`, `double`, `boolean`, `ip`.
- 带层级嵌套的 JSON `object`, `nested`
- 特殊类型: `geo_point`, `geo_shape`, `completion`.

为不同的目的以不同的方式索引相同的字段通常是有用的。
例如，字符串字段可以作为全文搜索的文本字段索引，也可以作为排序或聚合的关键字字段索引。或者，您可以使用标准分析器、英语分析器和法语分析器索引字符串字段。

这就是多领域的目的。大多数数据类型通过fields参数支持多字段。


## 更新映射

```
PUT twitter 
{}

PUT twitter/_mapping 
{
  "properties": {
    "email": {
      "type": "keyword"
    }
  }
}
```
更新字段的映射

通常来说, 字段已经存在的映射不能被更新, 不过有几个例外:

- 新属性可以添加到 Object datatype 字段中
- 新的 multi-fields 可以被添加到存在的字段中
- ignore_above 参数可以被更新


```
PUT my_index 
{
  "mappings": {
    "properties": {
      "name": {
        "properties": {
          "first": {
            "type": "text"
          }
        }
      },
      "user_id": {
        "type": "keyword"
      }
    }
  }
}

PUT my_index/_mapping
{
  "properties": {
    "name": {
      "properties": {
        "last": { 
          "type": "text"
        }
      }
    },
    "user_id": {
      "type": "keyword",
      "ignore_above": 100 
    }
  }
}
```

## 动态映射

ES中有一个非常重要的特性——动态映射，即索引文档前不需要创建索引、类型等信息，在索引的同时会自动完成索引、类型、映射的创建。那么什么是映射呢？映射就是描述字段的类型、如何进行分析、如何进行索引等内容。

当ES在文档中碰到一个以前没见过的字段时，它会利用动态映射来决定该字段的类型，并自动地对该字段添加映射。

有时这正是需要的行为，但有时不是，需要留意。你或许不知道在以后你的文档中会添加哪些字段，但是你想要它们能够被自动地索引。或许你只是想要忽略它们。或者，尤其当你将ES当做主要的数据存储使用时，大概你会希望这些未知的字段会抛出异常来提醒你注意这一问题。

幸运的是，你可以通过dynamic设置来控制这一行为，它能够接受以下的选项：

- true：默认值。动态添加字段
- false：忽略新字段
- strict：如果碰到陌生字段，抛出异常


dynamic设置可以适用在根对象上或者object类型的任意字段上。你应该默认地将dynamic设置为strict，但是为某个特定的内部对象启用它：
```
PUT /my_index
{
    "mappings": {
        "my_type": {
            "dynamic":      "strict", 
            "properties": {
                "title":  { "type": "string"},
                "stash":  {
                    "type":     "object",
                    "dynamic":  true 
                }
            }
        }
    }
}
```



<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
