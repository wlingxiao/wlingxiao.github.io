---
title: ElasticSearch 索引配置
date: 2017-08-20 19:29:57
tags: [ElasticSearch]
categories: ElasticSearch
---
索引与 `ik` 分词器配置
<!-- more -->
#### 新建简单索引
`curl -XPUT [address]/index_name`
#### 带参数新建索引
```shell
curl -XPUT [address]/index_name -d '{
  "settings": {
     "refresh_interval": "5s",
     "number_of_shards" :   1, // 一个主节点
     "number_of_replicas" : 0 // 0个副本，后面可以加
  },
  "mappings": {
    "_default_":{
      "_all": { "enabled":  false } // 关闭_all字段，因为我们只搜索title字段
    },
    "resource": {
      "dynamic": false, // 关闭“动态修改索引”
      "properties": {
        "title": { // 需要索引的字段
          "type": "string",
          "index": "analyzed",
          "fields": {
            "cn": {
              "type": "string",
              "analyzer": "ik_max_word", // 启用 ik 分词,
              "search_analyzer": "ik_max_word"
            },
            "en": {
              "type": "string",
              "analyzer": "english"
            }
          }
        }
      }
    }
  }
}'
```

#### 配置 `ik` 分词器（ElasticSearch 版本 2.4.5）
按照上面的方法配置 `ik` 分词器不成功，需要按照 elasticsearch-analysis-ik 官方 README 方法才能成功

#### 删除索引
`curl -XDELETE [address]/index_name`

#### 错误
+ [No handler for type [text] declared on field [content]](http://www.cnblogs.com/softidea/p/6081326.html)

建立映射
```shell
curl -XPOST http://localhost:9200/index/fulltext/_mapping -d'
{
    "fulltext": {
             "_all": {
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_max_word",
            "term_vector": "no",
            "store": "false"
        },
        "properties": {
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word",
                "include_in_all": "true",
                "boost": 8
            }
        }
    }
}'
```
响应为 `400`
```json
{
    "error": {
        "root_cause": [
            {
                "type": "mapper_parsing_exception",
                "reason": "No handler for type [text] declared on field [content]"
            }
        ],
        "type": "mapper_parsing_exception",
        "reason": "No handler for type [text] declared on field [content]"
    },
    "status": 400
}
```
> 上面这个请求在2.3.4版本的es服务器上会执行失败，原因是es 2.3.4上面还没有text这种类型，"type":"text" 改为 "type":"String"即可
>
参考文章：  
> + [为Elasticsearch添加中文分词，对比分词器效果](http://keenwon.com/1404.html)  
> + [elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik)