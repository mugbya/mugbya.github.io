---
title: ES之词频统计
date: 2020-10-28 17:25:38
toc: true
tags:
 - 技术
categories:
 - elasticsearch 
---

## 1. 缘起

今天在群里遇到一个需求：**统计北京、天津、河北出现的总数（不是文档数目)**

![](/images/es/term-freq-01.png)

期望得到 北京：3； 天津：5；河北：1


## 2. 实践

这个需求以前没有接触过，然后去搜索下了，发现这是一个词频统计的问题

相关链接

- [es中怎么判断同一条信息中同一个词出现的次数](https://elasticsearch.cn/question/6834)
- [Elasticsearch词频统计实现与原理解读](https://developer.aliyun.com/article/707393)


然后我也简单实现了下：

```yaml
POST _bulk
{"index":{"_index":"test","_type":"_doc","_id":"1"}}
{"cities":["北京"]}
{"index":{"_index":"test","_type":"_doc","_id":"2"}}
{"cities":["北京",  "天津"]}
{"index":{"_index":"test","_type":"_doc","_id":"3"}}
{"cities":["河北",  "天津"]}
{"index":{"_index":"test","_type":"_doc","_id":"4"}}
{"cities":["天津"]}
{"index":{"_index":"test","_type":"_doc","_id":"5"}}
{"cities":["北京", "北京",  "天津"]}
{"index":{"_index":"test","_type":"_doc","_id":"6"}}
{"cities":["天津"]}


GET test/_search
{
  "size": 0, 
  "aggs": {
    "term_sum": {
      "terms": {
         "order": {
          "_term": "desc"
        },
        "field": "cities.keyword",
        "size": 10
      }
    }
  }
}
```

> 这里使用es自带的mappings, es会自动给text类型再增加一个keyword类型，因为 text 类型不能用于聚合，以前文章有相关内容


聚合结果也是题主想要的

```yaml

  "aggregations" : {
    "term_sum" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "河北",
          "doc_count" : 1
        },
        {
          "key" : "天津",
          "doc_count" : 5
        },
        {
          "key" : "北京",
          "doc_count" : 3
        }
      ]
    }
  }

```

我以为自此这个需求我就实现了，本质是**数据进入es会被拆成token(词元), 然后就是对词元的一个频率统计**




结果一会群主补了一篇文章 [星球文章](https://wx.zsxq.com/dweb2/index/topic_detail/582245525418184)

一看是一个我不曾用过的api, 看了下[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/docs-multi-termvectors.html)，竟然1.3 开始就有了。 然后我对照着实现了下，

```yaml

PUT my_index_003
{
  "mappings": {
    "properties": {
      "title": {
        "type": "keyword"
      },
      "tags": {
        "type": "keyword"
      }
    }
  }
}

GET my_index_003/_search

POST my_index_003/_bulk
{"index":{"_id":1}}
{"tags":["北京","北京","天津"]}
{"index":{"_id":2}}
{"tags":"北京"}
{"index":{"_id":3}}
{"tags":["北京","天津"]}
{"index":{"_id":4}}
{"tags":["河北","天津"]}
{"index":{"_id":5}}
{"tags":"天津"}
{"index":{"_id":6}}
{"tags":"天津"}



POST my_index_003/_search

GET my_index_003/_mtermvectors
{
  "ids": [
    1,
    2,
    3,
    4,
    5,
    6
  ],
  "parameters": {
    "fields": [
      "tags"
    ],
    "term_statistics": true,
    "offsets": false,
    "payloads": false,
    "positions": false
  }
}
```

结果一看结果明白它更具体的场景，它是用在 统计文档某个字段的词频情况，也给出了该词出现在多少文档中


## 总结

1. 如果需要统计词频在总文档出现的次数，用 terms 更方便直接
2. 如果需要查看某个文档下某字段有多少词元，及出现的频率；或者在给定文档范围情况下，统计词元 **在该文档出现的次数** 跟 **出现在多少文档中**，并不直接提供统计词元出现的总数

