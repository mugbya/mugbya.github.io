---
title: ES之简介close与freeze
date: 2020-09-17 14:46:18
toc: true
tags:
 - 技术
categories:
 - elasticsearch 
---


## 1.缘起


上次在群里看到彬哥分享的 ILM 后, 自己使用7.9版本是试用了下，很棒，解决之前索引管理的麻烦(以前都是程序中处理)

然后我把ILM分享给部门负责人，然后让我演示其用法，在演示后，部门负责人提出一个问题，在cold节点的index能否close, 因为IML是一直在增加索引的，担心索引太多造成集群性能问题，如果能把cold节点的index关闭就更好了


## 2.探索及结果

拿到这个问题，我还真的第一时间就朝着 close index 去了, 去翻了 [Index lifecycle actions](https://www.elastic.co/guide/en/elasticsearch/reference/current/ilm-actions.html), 发现没有 close 这个行为。也去群里问了，结果群友推荐 curator, 我查了下，并不适用于iml中(因为自动管理后，我也不知道哪些index在什么时候就到cold节点了)

然后继续搜索，发现 [使用 Elasticsearch Freeze index API 创建冻结索引](https://www.elastic.co/cn/blog/creating-frozen-indices-with-the-elasticsearch-freeze-index-api) 这篇文章, 发现 freeze index，同样消耗集群资源很少，只需要维护一些元数据


在官方 [freeze-index-api](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/freeze-index-api.html) 也有说明

```yaml
A frozen index has almost no overhead on the cluster (except for maintaining its metadata in memory), and is blocked for write operations.
```


实践也发现，直接按照日常搜索是检索不到位于cold节点的数据，只要加了 `ignore_throttled=false` 后，才能检索到cold 节点的数据


这个完全能达到部门负责人的需求，直接反馈



## 3.对比

**close**

当close index后，集群几乎没有开销，除了在内存中维护元数据，同时index即不能写，也不能读，要操作需要open index。

> **注意**
>
> Elasticsearch不会在任何close的index中重建丢失的分片副本。 随着时间的推移，替换群集中的节点时，所有封闭索引的分片数据将逐渐丢失。


**freeze**

freeze index后，集群几乎没有开销，除了在内存中维护元数据，此时index可以读，但是不能写入

> freeze 是在ES 6.6 版本引入的


总结: 偶尔要被检索的cold数据就用freeze就行，不需要在去close。 iml close policy 可以配置成 freeze, 可以iml真的是好用啊


