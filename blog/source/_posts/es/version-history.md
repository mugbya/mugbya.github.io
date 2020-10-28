---
title: ES版本变化史
date: 2020-05-26 10:36:05
toc: true
tags:
 - 技术
categories:
 - elasticsearch 
---


```yaml 
本篇讲述ES的变化带来的影响，不同版本可能有些东西不一定，需要注意
```

## 1. 版本介绍

| 版本 | 文档                                                         |
| ---- | ------------------------------------------------------------ |
| 0.9  | [0.9-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/0.9/getting-started.html) |
| 1.*  | [1.3-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/1.0/getting-started.html) |
| 2.*  | [2.0-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/2.0/getting-started.html) |
| 5.*  | [5.0-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/getting-started.html) |
| 6.*  | [6.0-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/getting-started.html) |
| 7.*  | [Getting Started](https://www.elastic.co/guide/en/elastic-stack-get-started/current/index.html) |

5.0 应该是变化最大的版本，有兴趣关注5.0变化的请点击[大数据杂谈微课堂|Elasticsearch 5.0 新版本的特性与改进](https://www.infoq.cn/article/2016/08/Elasticsearch-5-0-Elastic/)

> **提示**
>
> 中间没有3.* 跟 4.* 跳过这两个版本是因为 elasticsearch 要保持全家桶版本一致
>
> ~~当前版本 7.3~~
> ~~当前版本 7.8~~
> 当前版本 7.9




在使用的时候经常 看到有人用 `string`类型，然后再在基础上写博客。当然这不是说它本身是错的，但是因为 ES 版本变化原因，导致很多细节已经在别的版本中不能在适用了，故本文对这种变更东西进行梳理



## 2. 详情

### 2.1 type





### 2.2 string 类型

在 5版本 以前都有 `string`类型，但是在 5.*版本开始已经移除了 该类型。


>  ### 移除背景及缘由
>
> 因为ElasticSearch对字符串拥有两种完全不同的搜索方式. 你可以按照整个文本进行匹配, 即关键词搜索(keyword search), 也可以按单个字符匹配, 即全文搜索(full-text search). 对ElasticSearch稍有了解的人都知道, 前者的字符串被称为`not-analyzed`字符, 而后者被称作`analyzed`字符串.
>
> 同一种类型用于应对两种不同的使用场景是会让人崩溃的, 因为有些选项只对其一的场景设置有效.例如`position_increment_gap`对`not-analyzed`字符就不会起作用, 而像`ignore_above`对于`analyzed`字符串就很难区分它到底是对整个字符串的值有效还是对单独的每个分词有效(在这种场景, ignore_above确实只对整个字符串值有效, 而对单个分词的限制可以使用`limit`设置).


最终 `string`字段被拆分成两种新的数据类型: `text`用于全文搜索的, 而`keyword`用于关键词搜索.

------

在7.9 增加 wildcard 类型。 用来通配搜索，该类型字段的统配搜索要比以前更快, 详情参考[Wildcard field type](https://www.elastic.co/guide/en/elasticsearch/reference/master/wildcard.html)



参考链接：[ElasticSearch数据类型--string类型已死, 字符串数据永生](https://segmentfault.com/a/1190000008897731)



### 3 参数

- [集群自动发现](http://blog.mugbya.me/2020/09/18/ES%E4%B9%8B%E9%9B%86%E7%BE%A4%E8%87%AA%E5%8A%A8%E5%8F%91%E7%8E%B0/)








## changelog

- 2020-07-13以前: 略
- 2020-07-13: 自动发现
- 2020-09-18: 移除自动发现详情, 增加 wildcard 类型




