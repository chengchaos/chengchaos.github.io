---
title: Elsstiicsearch 学习笔记
key: 2022-11-14
tags: elasticsearch es
---

建议关键字 elasticsearch es


<!--more-->

## 0x01 简介

omitted...

## 0x02 Elasticsearch 的索引 API

创建索引: PUT /索引名

验证： GET /_cat/索引名

```sh
curl -k -X PUT \
    --user elastic:XHCl5vRYe6hlh97CzpE3  \
    https://localhost:9200/chengchao

{"acknowledged":true,"shards_acknowledged":true,"index":"chengchao"}

curl -k -X GET \
    --user elastic:XHCl5vRYe6hlh97CzpE3  \
    https://localhost:9200/_cat/indices

yellow open chengchao sIcmT57ORhWk1wdTmqPOJg 1 1 0 0 225b 225b

curl -k -X POST \
    -H 'Content-Type: Application/json' \
    --user elastic:XHCl5vRYe6hlh97CzpE3 \
    https://localhost:9200/chengchao/_doc -d '
{"message" : "Hello Eko 2"}
'

curl -k -X GET \
    -H 'Content-Type: Application/json' \
    --user elastic:XHCl5vRYe6hlh97CzpE3 \
    https://localhost:9200/chengchao/_doc/1

curl -k -X GET \
    -H 'Content-Type: Application/json' \
    --user elastic:XHCl5vRYe6hlh97CzpE3  \
    https://localhost:9200/chengchao/_doc/_search
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "request [/chengchao/_doc/_search] contains unrecognized parameter: [q]"
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "request [/chengchao/_doc/_search] contains unrecognized parameter: [q]"
  },
  "status": 400
}

curl -k -X GET \
    -H 'Content-Type: application/json' \
    --user elastic:XHCl5vRYe6hlh97CzpE3  \
    https://localhost:9200/chengchao/_search \
    -d '
{
    "query" : {
        "match" : {
            "message" : "Eko"
        }
    }
}
'

curl -k -X GET \
    -H 'Content-Type: application/json' \
    --user elastic:XHCl5vRYe6hlh97CzpE3  \
    https://localhost:9200/chengchao/_search

curl -k -X GET \
    -H 'Content-Type: application/json' \
    --user elastic:XHCl5vRYe6hlh97CzpE3  \
    https://localhost:9200/chengchao/_search?q=Eko

curl -k -X GET \
    -H 'Content-Type: application/json' \
    --user elastic:XHCl5vRYe6hlh97CzpE3  \
    https://localhost:9200/chengchao/_doc/1

curl -k -X DELETE \
    --user elastic:XHCl5vRYe6hlh97CzpE3  \
    https://localhost:9200/chengchao
```

## 文档 API

```bash
PUT /index_name/1
{
    "name" : "Eko"
}

curl -k -X PUT \
    -H 'Content-Type: Application/json' \
    --user elastic:XHCl5vRYe6hlh97CzpE3 \
    https://localhost:9200/chengchao/_doc/1 \
    -d '
{"message" : "Hello Eko"}
'
```

## Elasticsearch 的查询


### Query String

get /index/doc/_search?q=john

### Query DSL

GET /index/doc/_search

{
    "query" : {
        "match" : {
            "name" : "john"
        }
    }
}

./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.23/elasticsearch-analysis-ik-6.8.23.zip

./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.4.1/elasticsearch-analysis-ik-8.4.1.zip

## 6.x

```bash
curl -X PUT 'http://localhost:9200/chengchao'
 
curl -X POST -H 'Content-Type:Application/json' \
'http://localhost:9200/chengchao/doc/1' -d '
{"message" : "Hello Eko"}
'

curl -X GET 'http://localhost:9200/chengchao/_mapping' -s | jq
{
  "chengchao": {
    "mappings": {
      "doc": {
        "properties": {
          "message": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}

curl -s -X PUT 'http://localhost:9200/chengchao3?pretty' \
    -H 'Content-Type:application/json' \
    -d '
{
    "mappings" : {
        "doc" : {
            "properties" : {
                "title" : {
                    "type" : "text",
                    "analyzer" : "ik_max_word"
                },
                "tags" : {
                    "type" : "text",
                    "analyzer" : "whitespace"
                },
                "content" : {
                    "type" : "text",
                    "analyzer" : "ik_smart"
                }
            }
        }
    }
}'

curl -s -X PUT -H "Content-Type:application/json" \
    'http://localhost:9200/chengchao3/doc/1' -d '
{
    "title" : "如何避免 Go 命令行执行产生“孤儿”进程？",
    "tags" : "bash url_encode url_decode",
    "content" : "在 Go 程序当中，如果我们要执行命令时，通常会使用 exec.Command ，也比较好用，通常状况下，可以达到我们的目的，如果我们逻辑当中，需要终止这个进程，则可以快速使用 cmd.Process.Kill() 方法来结束进程。但当我们要执行的命令会启动其他子进程来操作的时候，会发生什么情况？"
}'

curl 'http://localhost:9200/chengchao3/_search?q=孤儿'
curl 'http://localhost:9200/go-blog/_search?pretty' \
-H 'content-type:application/json' \
-d '
{
    "query" : {
        "match" : {
            "content" : "exec.Command"
        }
    }
}'


curl -X GET \
'http://localhost:9200/_analyze?pretty' \
    -H 'Content-Type:application/json' \
    -d '{"analyzer" : "ik_smart", "text" : "会发生什么情况"}' 



curl -X POST \
'http://localhost:9200/chengchao3/_analyze?pretty' \
-H 'content-type:application/json' \
-d '{"field" : "content" , "text" :"但当我们要执行的命令会启动其他子进程来操作的时候，会发生什么情况？"}'
```






EOF


