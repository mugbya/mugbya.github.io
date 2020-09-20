---
title: ES之doc_values与fielddata
date: 2020-06-02 11:46:18
toc: true
tags:
 - 技术
categories:
 - elasticsearch 
---

```yaml
 本文档适用范围：大部分描述适用于所有的es版本，部分内容适用于 5.* 版本以上
```


## 1. 自白

ES除了搜索十分优秀之外，聚合排序也还十分好用。

博主以前使用聚会功能场景不算太多，能写简答聚合语句。所以对里面很多东西知之甚少，偶然会遇到一些问题，能够找到解决方式，但是没有形成体系。

在一次部门技术负责人分享ES时，我才知道以前用的遇到的问题跟 `doc_values` 与 `fielddata` 相关；也更清楚知道聚合是如何实现的

本文先介绍聚合相关基础部分，慢慢由浅到深入


## 2. 基础

搜索我默认你都用过，而且也知道 `倒排索引` 是什么。

在聚合时需要另一种 **数据结构** 来支持聚合，这种数据结构就是 `Doc Values`，本篇只有基础。


### 2.1 Doc Values

- **概念**：是一种结构，存储 文档 与 `term` 的`正排索引`，也可以点击查看[官方定义](https://www.elastic.co/guide/en/elasticsearch/reference/current/doc-values.html)

- **作用与特性**：用于聚合，排序，大部分字段默认支持并开启 doc_values, `text`类型不支持

- **时机**：在文档被索引时，跟 `倒排索引是同时发生的`

- **存储地方**：磁盘

  > 对于这一点存在疑问，在 2.*版本提到 “当 `working set` 远小于系统的可用内存，系统会自动将 `Doc Values` 驻留在内存中，使得其读写十分快速；不过，当其远大于可用内存时，系统会根据需要从磁盘读取 `Doc Values`，然后选择性放到分页缓存中”
  >
  > 
  >
  > 但是在后别的版本并没有这样的描述，反而 `doc_values` 的表述更多跟 on-disk 联系在一起
  >
  > 
  >
  > 参考链接： [深入理解 Doc Values](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_deep_dive_on_doc_values.html)
  >
  > 

  
参考链接
- [官方doc_values](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/doc-values.html#doc-values)
- [doc_values介绍](https://www.cnblogs.com/bonelee/p/6401531.html)



### 2.2 fielddata

上面我们知道 text 类型不支持 `Doc Values`，故此ES又搞出一个查询时的数据结构 ，然后这个结构就叫 `fielddata`


官方文档解释

> Most fields can use index-time, on-disk `doc_values` for this data access pattern, but` text` fields do not support doc_values.
>
> Instead, text fields use a query-time in-memory data structure called fielddata. This data structure is built on demand the first time that a field is used for aggregations, sorting, or in a script. It is built by reading the entire inverted index for each segment from disk, inverting the term ↔︎ document relationship, and storing the result in memory, in the JVM heap.


根据上面的叙述 能够明确 `fielddata` 是 ` doc_values` 的补充

- **概念**：是一种结构，存储 文档 与 `term` 的`正排索引`

- **作用与特性**：用于聚合，排序，专门用来解决 `text` 类型不能统计，聚合

- **时机**：在文档被**查询**时

- **存储地方**：内存




继续看官方文档

>  **Fielddata is disabled on text fields by defaultedit**
>
> Fielddata can consume a lot of heap space, especially when loading high cardinality text fields. Once fielddata has been loaded into the heap, it remains there for the lifetime of the segment. Also, loading fielddata is an expensive process which can cause users to experience latency hits. This is why fielddata is disabled by default.
>
> If you try to sort, aggregate, or access values from a script on a text field, you will see this exception:
>
> Fielddata is disabled on text fields by default.  Set `fielddata=true` on
> [`your_field_name`] in order to load  fielddata in memory by uninverting the
> inverted index. Note that this can however use significant memory.

text 默认情况下也是 禁止了 Fielddata 的，因为开启会消耗大量 堆空间。简而言之开启的代价很昂贵，如果是在线系统开启，则有可能导致用户访问延迟



参考链接：[官方7.7 - fielddata](https://www.elastic.co/guide/en/elasticsearch/reference/current/fielddata.html)


## 3. 优化

在介绍基础时，顺便把相关的优化也介绍了。

由上文我们知道 `Doc Values` 这种数据结构跟 `倒排索引` 一样，都是在创建时就建立，需要消耗cpu,跟磁盘。单条插入时这个消耗其实并一定体现很大，如果是批量导入大量数据，可能会帮助你提升下。最好是你不要聚合，那么就可以关掉这个

再次提醒，除非你明确需求不会有聚合功能，否则不要禁用 `Doc Values`。

当然你也可以用一个索引只做搜索支持在线业务，然后有聚合需求时使用 `reindex` 搞一个线下的index来进行统计分析

如果你统计分析也需要提供线上服务，就别禁用了

放上禁用的例子

```yaml
PUT my_index
{
  "mappings": {
    "type": {
      "properties": {
        "id": {
          "type": "keyword",
          "doc_values": false
        },
        "valye": {
          "type": "keyword",
          "doc_values": false
        },
        "status": {
          "type": "integer",
          "doc_values": false
        },
        "desc": {
          "type": "keyword",
          "doc_values": false
        }
      }
    }
  }
}

```


## 4. 总结

> **总结**
> fielddata 与 doc_values 不同的地方在于 
> 1. 支持的类型
> 2. 创建的时机
> 3. 存储的地方
>


`text` 字段很特殊，为了能够支持 复杂搜索，所以ES做了 `倒排索引`来支持

但还要做另一不同需求 统计，聚合这种，却不能使用 `倒排索引`来做，又做了 `正排索引`（doc_values）


`doc_values` 默认支持大多数字段，但是不支持 `text`类型， 可以看下图

![](https://i.imgur.com/3wyMRa4.png)



然后ES 为了支持 `text`类型，又搞出 `fielddata`结构


但是 `text`类型不能被用于 聚合的问题 ES 还给出了另一个方案，那就是 给出 一个 `keyword`不分词的字段

```yaml
PUT my_index
{
  "mappings": {
    "properties": {
      "my_field": { 
        "type": "text",
        "fields": {
          "keyword": { 
            "type": "keyword"
          }
        }
      }
    }
  }
}
```



总之 `fielddata` 要慎用，想清楚再用






