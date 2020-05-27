---
title: elasticsearch入门
date: 2020-05-26 15:05:02
tags: 
 - 技术
 - 扯淡
categories:
 - elasticsearch
---



## 1 基础概述

### 1.1 简介

`Elasticsearch`是用Java开发的，是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据



**Lucene与ES关系**

- `Lucene`只是一个库。想要使用它，你必须使用Java来作为开发语言并将其直接集成到你的应用中，更糟糕的是，`Lucene`非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的。

- `Elasticsearch`也使用Java开发并使用`Lucene`作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。



#### 1.1.1 版本介绍

| 版本 | 文档                                                         |
| ---- | ------------------------------------------------------------ |
| 0.9  | [0.9-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/0.9/getting-started.html) |
| 1.*  | [1.3-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/1.0/getting-started.html) |
| 2.*  | [2.0-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/2.0/getting-started.html) |
| 5.*  | [5.0-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/getting-started.html) |
| 6.*  | [6.0-Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/getting-started.html) |
| 7.*  | [Getting Started](https://www.elastic.co/guide/en/elastic-stack-get-started/current/index.html) |

5.0 应该是变化最大的版本，有兴趣关注5.0变化的请点击[大数据杂谈微课堂|Elasticsearch 5.0 新版本的特性与改进](https://www.infoq.cn/article/2016/08/Elasticsearch-5-0-Elastic/)

> 注：中间没有3.* 跟 4.* 跳过这两个版本是因为 elasticsearch 要保持全家桶版本一致
>
> 当前版本 7.3



#### 1.1.2 端口介绍

- 9200是http协议的RESTful接口
- 9300是tcp通讯端口，集群间和TCPClient都走的它

> 所以使用时需要在每个节点开启 9200、 9300端口，在指定一台机器上开启5601端口允许访问



### 1.2 特点

- 简单的restful api，天生的兼容多语言开发
- 分布式的实时文件存储，每个字段都被索引且可用于搜索
- 分布式的实时分析搜索引擎，海量数据下近实时秒级响应
- 易扩展，处理PB级结构化或非结构化数据



### 1.3 性能

压测工具 `esrally`, 介绍可点击参看 [Elasticsearch 压测方案之 esrally 简介](https://segmentfault.com/a/1190000011174694)

图片来源于网络

![](images/mysql-vs-es.png)



### 1.4 相对薄弱环节

**`多表关联`**

不能简单认为，将mysql同步到Elasticsearch就能解决问题了。

我们除了看到基于倒排索引Elasticsearch的全文检索的强大，也要看到Elasticsearch对于关系型数据库多表关联的支持相对薄弱，nested类型、Join类型的多表关联操作大数据场景下都会有性能问题。



**`深度分页`**

从性能角度考虑，Elasticsearch默认支持10000条数据的返回，除非修改`max_result_window`参数。

也就是会出现越往后翻页越慢的情况。这点，补救方案：scroll+scroll_after实现。

但是，更长远角度，建议：参考Google、百度的深度分页实现。



**`实时性`**

Elasticsearch是近实时的系统，不是准实时。受限于：`refresh_inteval` 设置，有最快1s延时。

准实时要求高的场景，建议选型注意



### 1.5 解决痛点

- 解决大规模数据检索问题
- 解决大规模数据统计分析问题
- 查询速度



### 1.6 适合场景

- 记录和日志分析
- 采集和组合公共数据
- 全文搜索
- 事件数据和指标
- 数据可视化

参考链接： [Elasticsearch Top5典型应用场景](https://blog.csdn.net/laoyang360/article/details/82728797)



### 1.7 前景光明

Elasticsearch在DBRanking 数据库排行榜搜索引擎部分近几年一直处于第一名的领先优势。

![](images/DBRanking.png)



### 1.8 谁在使用

基于Elastic的分布式、可扩展性、良好的性能，BAT、滴滴、美团、小米、华为、携程、360、有赞等几乎所有的主流互联网公司甚至婚庆网站的搜索引擎已经都已经转成ES了。



## 2 系统概述

### 2.1 基本概念

![](images/term.png)



- 集群(Cluster)
- 节点(Node)
- 索引(index)
- 分片(Shard)
- 副本(replicas)
- 分段
- 倒排索引
- 分析器
- ES的底层是lucene
- Mapping
- 索引别名
- 索引模板



#### 1.1.1 集群

多个使用相同`cluster-name`的ES实例构成一个集群



#### 1.1.2 节点

一个ES实例就是一个node, 一台机器上可以启动多个ES实例，但是大多数情况下每个node运行在一个独立的环境或虚拟机上

节点也有分类，ES有如下分类

**`Master-eligible node`**

node.master设置为true（默认）的节点，使其有资格被选为控制集群的主节点。

主节点负责轻量级群集范围的操作，例如创建或删除索引，跟踪哪些节点是群集的一部分，以及决定将哪些分片分配给哪些节点。集群运行状况对于拥有稳定的主节点非常重要

> 主节点必须能够访问数据/目录（就像数据节点一样），因为这是在节点重新启动之间保持集群状态的地方



**`Data node`**

node.data设置为true的节点（默认值）。 数据节点保存数据并执行与数据相关的操作，例如CRUD，搜索和聚合。

**`Ingest node`**

node.ingest设置为true的节点（默认值）。`Ingest node`能够将摄取管道应用于文档，以便在编制索引之前转换和丰富文档。 使用大量的摄取负载，使用专用的摄取节点并将主节点和数据节点标记为node.ingest：false是有意义的。

> 需要做monitoring cluster则monitoring cluster里必须有节点类型为 [ingest node](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/ingest.html)

**`Tribe node`**

通过`tribe.*settings`配置的tribe节点是一种特殊类型的仅协调节点，可以连接到多个集群并在所有连接的集群中执行搜索和其他操作

> `Tribe Node` 在7.0版本被完全移除

**`Machine learning node`**

将`xpack.ml.enabled`和`node.ml`设置为true的节点，这是Elasticsearch默认分发中的默认行为。 如果要使用机器学习功能，则群集中必须至少有一个机器学习节点。 有关机器学习功能的更多信息，请参阅[Machine learning in the Elastic Stack](https://www.elastic.co/guide/en/elastic-stack-overview/7.0/xpack-ml.html).

> `Machine learning node`是6.8版本后引入的



> ## Coordinating node
>
> 诸如搜索请求或批量索引请求之类的请求可能涉及保存在不同数据节点上的数据。 例如，搜索请求在两个阶段中执行，这两个阶段由接收客户端请求的节点 - 协调节点协调。
>
> 在分散阶段，协调节点将请求转发到保存数据的数据节点。 每个数据节点在本地执行请求并将其结果返回给协调节点。 在收集阶段，协调节点将每个数据节点的结果减少为单个全局结果集。
>
> 每个节点都隐式地是一个协调节点。 这意味着将所有三个node.master，node.data和node.ingest设置为false的节点仅用作协调节点，无法禁用该节点。 结果，这样的节点需要具有足够的存储器和CPU以便处理收集阶段


索引和搜索数据是CPU，内存和I / O密集型工作，可能会对节点资源造成压力。为确保主节点稳定且不受压力，在较大的群集中最好分割`专用主节点`和`专用数据节点`之间的角色

虽然主节点也可以表现为协调节点并将搜索和索引请求从客户端路由到数据节点，但最好不要将`专用主节点`用于此目的。对于集群的稳定性而言，符合主节点的节点尽可能少地工作是很重要的



参考链接：[官网7.0-Node](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/modules-node.html)



#### 1.1.3 索引

一系列文档的集合，相当于sql的库。数据的读取与写入都是基于对索引的直接操作。

索引是对使用者最可见最小单位



#### 1.1.4 分片

当有大量的文档时，由于内存的限制、磁盘处理能力不足、无法足够快的响应客户端的请求等，一个节点可能不够。这种情况下，数据可以分为较小的分片。每个分片放到不同的服务器上。

 ES是分布式搜索引擎，每个索引有一个或多个分片；不修改设置时，索引默认是5个分片，这个配置在索引**创建后不能修改**

当你查询的索引分布在多个分片上时，ES会把查询发送给每个相关的分片，并将结果组合在一起，而应用程序并不知道分片的存在。即：这个过程对用户来说是透明的。



分片很重要，主要基于下面两个原因

- 它允许您水平分割/缩放内容量
- 它允许您跨分片（可能在多个节点上）分布和并行化操作，从而提高性能/吞吐量



我们往 Elasticsearch 添加数据时需要用到索引 —— 保存相关数据的地方。 **索引实际上是指向一个或者多个物理 分片的逻辑命名空间 。** 

- 一个 分片 是一个底层的 工作单元 ，它仅保存了 `全部数据中的一部分`

- 每个分片是Luncene索引的一个实例，你可以把实例理解成自管理的搜索引擎，用于在Elasticsearch集群中对一部分数据进行索引和处理查询。

- 分片的数目决定理论上能够存放最大的数据量(实际中受数据，硬件，使用场景约束)

- 分片是独立的，对于一个Search Request的行为，每个分片都会执行这个Request。

- 每个分片都是一个Lucene Index，所以一个分片只能存放 Integer.MAX_VALUE-128 = 21,4748,3519 个文档



#### 1.1.5 副本

在任何环境跟任何时候都可以出现故障，故强烈建议使用故障转移机制，以防分片/节点以某种方式脱机或因任何原因消失。为此，Elasticsearch允许您将索引的分片的一个或多个副本制作成所谓的副本分片或简称副本



复制很重要，主要有两个原因

- 它在碎片/节点出现故障时提供高可用性。因此，请务必注意，副本分片永远不会在与从中复制的原始/主分片相同的节点上分配
- 它允许您扩展搜索量/吞吐量，因为可以在所有副本上并行执行搜索



每个(主)分片的副本数，默认值是 `1` 。对于活动的索引库，这个配置**可以随时修改**。

即5个分片数，则总的副本数就是1*5。一定要记得不是一个索引一个副本，而是对应到一个分片一个副本

参考连接 [官文档-basic_concepts](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/_basic_concepts.html)



#### 1.1.6 分段

elasticsearch中的每个分片包含多个segment，每一个`segment`都是一个`倒排索引`；在查询时，会把所有的segment查询结果汇总归并后最为最终的分片查询结果返回

在创建索引的时候，elasticsearch会把文档信息写到内存bugffer中(为了安全，也一起写到translog),定时(可配置)把数据写到segment缓存小文件中，然后刷新查询，使刚写入的segment可查。 

虽然写入的segment可查询，但是还没有持久化到磁盘上。因此，还是会存在丢失的可能性的。 所以，elasticsearch会执行flush操作，把segment持久化到磁盘上并清除translog的数据(因为这个时候，数据已经写到磁盘上，不在需要了)。 

当索引数据不断增长时，对应的segment也会不断的增多，查询性能可能就会下降。因此，Elasticsearch会触发segment合并的线程，把很多小的segment合并成更大的segment，然后删除小的segment。 

segment是不可变的，当我们更新一个文档时，会把老的数据打上已删除的标记，然后写一条新的文档。在执行flush操作的时候，才会把已删除的记录物理删除掉。

segment大小与`indexing buffer`, 参数详情请点击 [indexing buffer](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/indexing-buffer.html)



#### 1.1.7 倒排索引

倒排索引（Inverted Index）也叫反向索引，有反向索引必有正向索引。通俗地来讲，正向索引是通过key找value，反向索引则是通过value找key

当PUT一个JSON的对象，这个对象有多个字段，在插入这些数据到索引的同时，Elasticsearch还为这些字段建立索引——倒排索引，因为Elasticsearch最核心功能是搜索



那么，倒排索引是个什么样子呢？

![](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127172829635-1286260863.png)

首先解释下几个概念

**Term（单词）**：一段文本经过分析器分析以后就会输出一串单词，这一个一个的就叫做Term（直译为：单词）

**Term Dictionary（单词字典）**：顾名思义，它里面维护的是Term，可以理解为Term的集合

**Term Index（单词索引）**：为了更快的找到某个单词，我们为单词建立索引

**Posting List（倒排列表）**：倒排列表记录了`出现过某个单词的所有文档的文档列表`及`单词在该文档中出现的位置信息`，每条记录称为一个倒排项(Posting)。根据倒排列表，即可获知哪些文档包含某个单词。（PS：实际的倒排列表中并不只是存了文档ID这么简单，还有一些其它的信息，比如：词频（Term出现的次数）、偏移量（offset）等，可以想象成是Python中的元组，或者Java中的对象）



> 如果类比现代汉语词典的话，那么Term就相当于词语，Term Dictionary相当于汉语词典本身，Term Index相当于词典的目录索引

来个实际例子

假设有个user索引，它有四个字段：分别是name，gender，age，address。画出来的话，大概是下面这个样子，跟关系型数据库一样

![](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127173241683-1331385372.png)

数据存入ES后建立的索引大致如下:

**name字段：**

![](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127175423615-230290274.png)

**age字段：**

![](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127175627644-1013476663.png)

**gender字段：**

![](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127175809626-1224287371.png)

**address字段：**

![](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127180053644-1305820142.png)



Elasticsearch分别为每个字段都建立了一个倒排索引。比如，在上面“张三”、“北京市”、22 这些都是Term，而[1，3]就是Posting List。Posting list就是一个数组，存储了所有符合某个Term的文档ID。

只要知道文档ID，就能快速找到文档。可是，要怎样通过我们给定的关键词快速找到这个Term呢？

当然是建索引了，为Terms建立索引，最好的就是B-Tree索引（PS：MySQL就是B树索引最好的例子）。

首先，让我们来回忆一下MyISAM存储引擎中的索引是什么样的：

![](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127183454632-267154610.png)



![](images/btree.png)

我们查找Term的过程跟在MyISAM中记录ID的过程大致是一样的

MyISAM中，索引和数据是分开，通过索引可以找到记录的地址，进而可以找到这条记录

在倒排索引中，通过Term索引可以找到Term在Term Dictionary中的位置，进而找到Posting List，有了倒排列表就可以根据ID找到文档了

> 可以这样理解，类比MyISAM的话，Term Index相当于索引文件，Term Dictionary相当于数据文件
>
> 其实，前面我们分了三步，我们可以把Term Index和Term Dictionary看成一步，就是找Term。因此，可以这样理解倒排索引：通过单词找到对应的倒排列表，根据倒排列表中的倒排项进而可以找到文档记录

为了更进一步理解，下面从网上摘了两张图来具现化这一过程：

![](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127184959667-1135956344.png)



![](https://img2018.cnblogs.com/blog/874963/201901/874963-20190127185725607-2022920549.png)



参考链接: 

- [Elasticsearch倒排索引结构](https://www.cnblogs.com/cjsblog/p/10327673.html)
- [elasticsearch 倒排索引原理](https://zhuanlan.zhihu.com/p/33671444)



### 2.2 写操作

![](https://changsiyuan.github.io/images/elk/2-8.png)

1. 客户端向Node1发送写操作的请求。
2. Node1使用文档的_id来决定这个文档属于shard0，然后将请求路由至NODE3，P0所在的位置。
3. Node3在P0上执行了请求。如果请求成功，则将请求并行的路由至NODE1 NODE2的R0上。当所有的replicas报告成功后，NODE3向请求的node(NODE1)发送成功报告，NODE1再报告至Client。



当客户端收到执行成功后，操作已经在Primary shard和所有的replica shards上执行成功了



### 2.3 读操作

![](https://changsiyuan.github.io/images/elk/2-9.png)

1. 客户端发送Get请求到NODE1。
2. NODE1使用文档的_id决定文档属于shard 0，shard 0的所有拷贝存在于所有3个节点上。这次，它将请求路由至NODE2。
3. NODE2将文档返回给NODE1，NODE1将文档返回给客户端。 

对于读请求，请求节点(NODE1)将在每次请求到来时都选择一个不同的replica shard来达到负载均衡。使用轮询策略轮询所有的replica shards。



### 2.4 设计方式

#### 2.4.1 不变性

- 写到磁盘的倒序索引是不变的：自从写到磁盘就再也不变。这会有很多好处：
  - 不需要添加锁。如果你从来不用更新索引，那么你就不用担心多个进程在同一时间改变索引。
  - 一旦索引被内核的文件系统做了Cache，它就会待在那因为它不会改变。只要内核有足够的缓冲空间，绝大多数的读操作会直接从内存而不需要经过磁盘。这大大提升了性能。
  - 其他的缓存(例如fiter cache)在索引的生命周期内保持有效，它们不需要每次数据修改时重构，因为数据不变。
  - 写一个单一的大的倒序索引可以让数据压缩，减少了磁盘I/O的消耗以及缓存索引所需的RAM。
- 当然，索引的不变性也有缺点。如果你想让新修改过的文档可以被搜索到，你必须重新构建整个索引。这在一个index可以容纳的数据量和一个索引可以更新的频率上都是一个限制。



#### 2.4.2 动态更新索引

如何在不丢失不变形的好处下让倒序索引可以更改？

答案是：使用不只一个的索引。 

新添额外的索引来反映新的更改来替代重写所有倒序索引的方案。 Lucene引进了`per-segment`搜索的概念。一个segment是一个完整的倒序索引的子集，所以现在index在Lucene中的含义就是一个segments的集合，每个segment都包含一些提交点(commit point)。新的文档建立时首先在内存建立索引buffer，见Figure17。然后再被写入到磁盘的segment，见Figure18

![](https://changsiyuan.github.io/images/elk/3-1.png)

![](https://changsiyuan.github.io/images/elk/3-2.png)

- 一个per-segment的工作流程如下：
  - 新的文档在内存中组织，见Figure17。
  - 每隔一段时间，buffer将会被提交：一个新的segment(一个额外的新的倒序索引)将被写到磁盘 一个新的提交点(commit point)被写入磁盘，将包含新的segment的名称。 磁盘fsync，所有在内核文件系统中的数据等待被写入到磁盘，来保障它们被物理写入。
  - 新的segment被打开，使它包含的文档可以被索引。
  - 内存中的buffer将被清理，准备接收新的文档。
- 当一个新的请求来时，会遍历所有的segments。词条分析程序会聚合所有的segments来保障每个文档和词条相关性的准确。通过这种方式，新的文档轻量的可以被添加到对应的索引中。



#### 2.4.3 删除跟更新

- segments是不变的，所以文档不能从旧的segments中删除，也不能在旧的segments中更新来映射一个新的文档版本。取之的是，每一个提交点都会包含一个.del文件，列举了哪一个segmen的哪一个文档已经被删除了。 当一个文档被”删除”了，它仅仅是在.del文件里被标记了一下。被”删除”的文档依旧可以被索引到，但是它将会在最终结果返回时被移除掉。
- 文档的更新同理：当文档更新时，旧版本的文档将会被标记为删除，新版本的文档在新的segment中建立索引。也许新旧版本的文档都会本检索到，但是旧版本的文档会在最终结果返回时被移除



#### 2.4.4 实时搜索

- 在上述的per-segment搜索的机制下，新的文档会在分钟级内被索引，但是还不够快。 瓶颈在磁盘。将新的segment提交到磁盘需要fsync来保障物理写入。但是fsync是很耗时的。它不能在每次文档更新时就被调用，否则性能会很低。
- 现在需要一种轻便的方式能使新的文档可以被索引，这就意味着不能使用fsync来保障。 在ES和物理磁盘之间是`内核的文件系统缓存`。在内存中索引的文档会被写入到一个新的segment。但是现在我们将segment首先写入到内核的文件系统缓存，这个过程很轻量，然后再flush到磁盘，这个过程很耗时。但是一旦一个segment文件在内核的缓存中，它可以被打开被读取

参考链接: [官方2.*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/near-real-time.html)



#### 2.4.5 更新持久化

不使用fsync将数据flush到磁盘，我们不能保障在断电后或者进程死掉后数据不丢失。ES是可靠的，它可以保障数据被持久化到磁盘。一个完全的提交会将segments写入到磁盘，并且写一个提交点，列出所有已知的segments。当ES启动或者重新打开一个index时，它会利用这个提交点来决定哪些segments属于当前的shard。 如果在提交点时，文档被修改会怎么样？



当一个文档被索引时，它会被添加到in-memory buffer，并且添加到Translog日志中，见Figure21.

![](https://changsiyuan.github.io/images/elk/3-5.png)

refresh操作会让shard处于Figure22的状态：每秒中，shard都会被refreshed，在in-memory buffer中的文档会被写入到一个新的segment，但没有fsync，in-memory buffer被清空

![](https://changsiyuan.github.io/images/elk/3-6.png)

这个过程将会持续进行：新的文档将被添加到in-memory buffer和translog日志中，见Figure23

![](https://changsiyuan.github.io/images/elk/3-7.png)

一段时间后，当translog变得非常大时，索引将会被flush，新的translog将会建立，一个完全的提交进行完毕。见Figure24

![](https://changsiyuan.github.io/images/elk/3-8.png)

- 在in-memory中的所有文档将被写入到新的segment.
- 内核文件系统会被fsync到磁盘。
- 旧的translog日志被删除



translog日志提供了一个所有还未被flush到磁盘的操作的持久化记录。当ES启动的时候，它会使用最新的commit point从磁盘恢复所有已有的segments，然后将重现所有在translog里面的操作来添加更新，这些更新发生在最新的一次commit的记录之后还未被fsync。

translog日志也可以用来提供实时的CRUD。当你试图通过文档ID来读取、更新、删除一个文档时，它会首先检查translog日志看看有没有最新的更新，然后再从响应的segment中获得文档。这意味着它每次都会对最新版本的文档做操作，并且是实时的



#### 2.4.6 Segment合并

通过每隔一秒的自动刷新机制会创建一个新的segment，用不了多久就会有很多的segment。segment会消耗系统的文件句柄，内存，CPU时钟。最重要的是，每一次请求都会依次检查所有的segment。segment越多，检索就会越慢。

ES通过在后台merge这些segment的方式解决这个问题。小的segment merge到大的，大的merge到更大的

这个过程也是那些被”删除”的文档真正被清除出文件系统的过程，因为被标记为删除的文档不会被拷贝到大的segment中。合并过程如Figure25：

- 当在建立索引过程中，refresh进程会创建新的segments然后打开他们以供索引。
- merge进程会选择一些小的segments然后merge到一个大的segment中。这个过程不会打断检索和创建索引。
- Figure26，一旦merge完成，旧的segments将被删除：新的segment被flush到磁盘，一个新的提交点被写入，包括新的segment，排除旧的小的segments，新的segment打开以供索引旧的segments被删除

![](https://changsiyuan.github.io/images/elk/3-9.png)

![](https://changsiyuan.github.io/images/elk/3-10.png)

> merge大的segments会消耗大量的I/O和CPU，严重影响索引性能。默认，ES会节制merge过程来给留下足够多的系统资源



参考链接：

- [ElasticSearch原理简介](https://changsiyuan.github.io/2018/01/18/2018-1-18-ElasticSearch-Intro/)





## 3 基本操作

本章介绍 elasticsearch 的 CRUD 操作。 首先在ES是直接支持 DSL( Domain Specific Language)语言的, 但是在6.3版本开始 Kibana能直接支持SQL语言，比如

![](images/sql.png)



虽然Kibana也支持SQL，但是下面我们还是以DSL来介绍操作



### 3.1 创建Index

首先我们创建一个带mapping 的 index, 使用如下DSL语言

```
PUT user/
{
  "mappings": {
    "test": {
      "properties": {
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "byte"
        },
        "lable": {
          "type": "text"
        },
        "lable_kw": {
          "type": "keyword"
        }
      }
    }
  }
}
```

> mapping 是一个index的文档存储结构，如果不主动创建，ES会在首次存入数据时，自动根据字段值进行创建默认mapping

### 3.2 保存记录

保存记录有两种方式，一直是不指定id, 让ES自动生成id 

```
POST user/test
{
  "name": "test",
  "age": 20,
  "label": "This is a test",
  "label_kw": "This is a test"
}
```

> 这里只能用 POST 方式



当指定id时的DSL语言

```
POST user/test/1
{
  "name": "柯西莫.德.美第奇",
  "age": 19,
  "label": "这是一位佛罗伦萨人",
  "label_kw": "这是一位佛罗伦萨人"
}
```

>  这里可以换成PUT 方式



### 3.3 修改记录

修改也有三种方式

一种是通过 PUT/POST 的全覆盖方式，旧数据将被删除，以新的代替。

```
POST user/test/1
{
  "name": "柯西莫.德.美第奇",
  "age": 45,
  "label": "这是一位佛罗伦萨人",
  "label_kw": "这是一位佛罗伦萨人"
}
```



一种是通过POST方式，只对部分字段进行修改

```
POST user/test/1/_update
{
  "doc": {
    "age": 40
  }
}
```



第三种是批量修改通过 update_by_query

```
POST user/_update_by_query?conflicts=proceed
{
  "script": {
    "source": "ctx._source['age']=0 "
  },
  "query": {
    "term": {
      "name": "test"
    }
  }
}
```

> 通过在 source 指定修改的字段



### 3.4 删除记录

删除应该可以分三种

删除单条记录

```
DELETE user/test/1
```



删除多条符合条件的记录

```
POST user/test/_delete_by_query
{
  "query": {
    "range": {
      "age": {
        "gte": 20,
        "lte": 20
      }
    }
  }
}
```



删除整个index 

```
DELETE user
```



### 3.5 查询记录

查询是ES最核心的功能，所以支持的查询很多，简单的可以归为下面两类

1. 支持精确匹配查询的：term、range、exists、wildcard、prefix、fuzzy等。

2. 支持全文检索的：match、match_phrase、query_string、multi_match等

下面介绍部分常用的



**`term`**

完全匹配(精确查询),  被搜索文本`不进行分词`。

如何匹配: term 搜索时传递的文本就是就是一个token, 将这个文本(token)在ES里面查找



**`match`**

全文搜索, 被搜索字段默认会`被分词`，可以指定分词器。 只支持对一个字段搜索

如何匹配：拿拆分的token去ES里面搜索，返回的文档记录中至少一个token匹配上，匹配的token越多，相关性(score)就越高

还能支持操作模式(operator):  or, and. 默认or  当指定为 and 时 跟 match_phrase 效果一样



**`match_phrase`**

match_phrase: 短语匹配。 被搜索字段默认会被分词，可以指定分词器。 

如何匹配：拿拆分的token去ES里面搜索，返回的文档记录中要全部包含所有的token



**`multi_match`**

全文搜索, 被搜索字段默认会被分词，可以指定分词器。 支持对对多个字段搜索(可以匹配多个字段的match版 )



**`query_string`**

query_string在默认情况下，在包含多个文本字段的文本的[_all](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/mapping-all-field.html#mapping-all-field)字段上查询搜索。最重要的是，它被解析并支持一些运算符(AND / OR …)，通配符等



### 3.6 聚合

![](images/aggr.png)



**`Bucketing分桶聚合`**

举例：最常用的terms就类似Mysql group by功能



**`Metric计算聚合`**

举例：类比Mysql中的： MIN(), MAX(), SUM() 操作



**`Pipeline针对聚合结果聚合`**

举例：bucket_script实现类似Mysql的group by 后having的操作。



**`matrix`** 

未来将被弃用，故此不介绍

参考链接： [官方文档7.3-aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)



## 4 进阶篇



### 4.1 元数据

**`_index`**

 当前的记录所在的index

**`_type`**

当前记录的type. 在6.0版本已经被弃用了，但是后续版本都兼容该属性

**`_id`**

文档唯一标识

**`_source`**

默认情况下，Elasticsearch 用 JSON 字符串来表示文档主体保存在 `_source` 字段中。像其他保存的字段一样，`_source` 字段也会在写入硬盘前压缩。

这几乎始终是需要的功能，因为：

- 搜索结果中能得到完整的文档 —— 不需要额外去别的数据源中查询文档
- 如果缺少 `_source` 字段，部分 `更新` 请求不会起作用
- 当你的映射有变化，而且你需要重新索引数据时，你可以直接在 Elasticsearch 中操作而不需要重新从别的数据源中取回数据。
- 你可以从 `_source` 中通过 `get` 或 `search` 请求取回部分字段，而不是整个文档。
- 这样更容易排查错误，因为你可以准确的看到每个文档中包含的内容，而不是只能从一堆 ID 中猜测他们的内容。

即便如此，存储 `_source` 字段还是要占用硬盘空间的。假如上面的理由对你来说不重要，你可以用下面的映射禁用 `_source` 字段：

```
PUT /my_index
{
    "mappings": {
        "my_type": {
            "_source": {
                "enabled":  false
            }
        }
    }
}
```

在搜索请求中你可以通过限定 `_source` 字段来请求指定字段：

```
GET /_search
{
    "query":   { "match_all": {}},
    "_source": [ "title", "created" ]
}
```



### 4.2 索引模板

索引模板允许您定义在创建新索引时==自动应用的模板==。

模板包括设置和映射以及控制是否应将模板应用于新索引的简单模式模板。

> 注意!!!
> 模板仅在索引创建时应用。 更改模板不会对现有索引产生影响。 使用create index API时，作为create index调用的一部分定义的设置/映射将优先于模板中定义的任何匹配的设置/映射

示例

```
PUT /_template/china_weather
{
  "index_patterns": "china_weather_*",
  "aliases": {
    "china_weather": {}
  },
  "mappings": {
    "china": {
      "properties": {
        "air": {
          "type": "keyword"
        },
        "cityName": {
          "type": "keyword"
        }
      }
    }
  }
}
```



### 4.3 段策略

在建立索引的时候对性能影响最大的地方就是在将索引写入文件的时候, 所以在具体应用的时候就需要对此加以控制，段(Segment) 就是实现这种控制的。稍后详细描述段(Segment) 的控制策略



**段(Segment) 的控制策略**

在建立索引的时候对性能影响最大的地方就是在将索引写入文件的时候, 所以在具体应用的时候就需要对此加以控制: 

Lucene默认情况是每加入10份文档(Document)就从内存往index文件写入并生成一个段(Segment) ,然后每10个段(Segment)就合并成一个段(Segment). 这些控制的变量如下： 

| IndexWriter属性 | 默认值         | 描述                                      |
| --------------- | -------------- | ----------------------------------------- |
| MergeFactory    | 10             | 控制segment合并的频率和大小               |
| MaxMergeDocs    | Int32.MaxValue | 限制每个segment中包含的文档数             |
| MinMergeDocs    | 10             | 当内存中的文档达到多少的时候再写入segment |


MaxMergeDocs用于控制一个segment文件中最多包含的Document数.比如限制为100的话,即使当前有10个segment也不会合并,因为合并后的segment将包含1000个文档,超过了限制。

MinMergeDocs用于确定一个当内存中文档达到多少的时候才写入文件,该项对segment的数量和大小不会有什么影响,它仅仅影响内存的使用,进一步影响写索引的效率。



### 4.5 别名

**`官方解释`**

索引别名可以指向一个或多个索引，并且可以在任何需要索引名称的API中使用

别名为我们提供了极大的灵活性。它们允许我们执行以下操作：

- 在正在运行的集群上的一个索引和另一个索引之间透明切换
- 对多个索引进行分组组合（例如，`logstash_all`的索引别名：是过去3个月索引`logstash_201903`, `logstash_201904`, `logstash_201905`的组合
- 在索引中的文档子集上创建“视图”（结合业务场景，会提升检索效率）



**`通俗解释`**

索引别名类似：windows的快捷方式，linux的软链接，mysql的视图。

前提：Elasitcsearch创建索引后，索引名不允许改。很多业务场景下单一索引可能无法满足要求。

- 场景1：PB级别增量数据，借助rollover api实现，由基于日期的n个索引组成，显然，对外提供服务使用别名会很便捷。
- 场景2：试想，线上提供服务的某个索引出了问题，比如：某字段分词定义不准确，如何保证对外提供服务不停止（不更改业务代码）的前提下更换索引，显然，别名更合适。




对应别名与索引对应关系 (网)图

![](https://ask.qcloudimg.com/http-save/yehe-1903727/np6o18j3ol.jpeg?imageView2/2/w/1620)

**`创建别名与索引对应关系`**

如果一个别名对应多个索引时，默认情况下我们只具有只读操作，当设置了对某个索引具有写入操作时，此时才能通过别名插入数据，关键参数：is_write_index

```cql
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "metro_operating_detail_2019_02",
        "alias": "metro_operating_detail",
        "is_write_index": false
      }
    },
    {
      "add": {
        "index": "metro_operating_detail_2019_test",
        "alias": "metro_operating_detail",
        "is_write_index": true
      }
    }
  ]
}
```

> 通过 metro_operating_detail 别名插入的数据实际落在 metro_operating_detail 索引中, 可以通过curator来配置当前索引 `is_write_index`为Ture
>
> 如果想别名下的多个索引都能写入/更新数据，那么只通过别名是做不到的，只能操作每个物理索引进行写入/更新



**`索引别名使用的常见例子`**

实战中，索引的设计可能不是一步到位。
随着业务的扩展，可能会在开发的中后期，调整索引Mapping结构，
比如：

1. `ik_smart`改成`hanlp`分词以高效分词

2. long类型改成keyword以提升检索效率

3. 修改索引分片数以便于机器横向扩展

4. 索引分成更小粒度的索引等以提升性能。

通常的做法，都需要借助：reindex操作完成索引的迁移。如果要确保线上环境的可靠运行且用户无感知（即无需告知用户，不影响用户的业务），使用别名指向更改前和更改后的索引是绝佳方案。



**`思考`**

试想一下：如果不是基于时间的索引，而使用大索引，批量删除数据会发生什么？
<details>
<summary>点击查看原因</summary>
<pre><code>
删除索引数据只能使用：delete_by_query，相比删除索引，delete_by_query删除数据只是逻辑删除；
真正的删除实际是段合并后的物理删除分段，也就是delete_by_query后，有一段时间磁盘空间不降反升。此时的检索效率会非常低。
</code></pre>
</details>



参考链接：

- [官方文档6.6](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/indices-aliases.html#indices-aliases)
- [云社区-如何在Elasticsearch里面使用索引别名](https://cloud.tencent.com/developer/article/1122840)
- [干货 | Elasticsearch基础但非常有用的功能之一：别名](https://blog.csdn.net/laoyang360/article/details/90743369)

- [ElasticSearch 内部机制浅析（二）]([https://leonlibraries.github.io/2017/04/20/ElasticSearch%E5%86%85%E9%83%A8%E6%9C%BA%E5%88%B6%E6%B5%85%E6%9E%90%E4%BA%8C/](https://leonlibraries.github.io/2017/04/20/ElasticSearch内部机制浅析二/))



### 4.6 集群

#### 4.6.1 集群健康

Elasticsearch 的集群监控信息中包含了许多的统计数据，其中最为重要的一项就是 集群健康 ， 它在 status 字段中展示为 green 、 yellow 或者 red 

```
GET /_cluster/health
```

```
{
  "cluster_name" : "unisound-dev",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 4,
  "number_of_data_nodes" : 4,
  "active_primary_shards" : 64,
  "active_shards" : 127,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

> 查看更详细的集群状态： 
>
> - GET _cluster/health?level=indices
> - GET _cluster/health?level=shards
> - GET _cat/shards 
> - POST _cluster/reroute?pretty
> - GET /_cat/master    // 查当前master



响应信息中最重要的一块就是 `status` 字段。状态可能是下列三个值之一：

- `green`

  所有的主分片和副本分片都已分配。你的集群是 100% 可用的。

- `yellow`

  所有的主分片已经分片了，但至少还有一个副本是缺失的。不会有数据丢失，所以搜索结果依然是完整的。不过，你的高可用性在某种程度上被弱化。如果 *更多的* 分片消失，你就会丢数据了。把 `yellow` 想象成一个需要及时调查的警告。

- `red`

  至少一个主分片（以及它的全部副本）都在缺失中。这意味着你在缺少数据：搜索只能返回部分数据，而分配到这个分片上的写入请求会返回一个异常。



`green`/`yellow`/`red` 状态是一个概览你的集群并了解眼下正在发生什么的好办法。剩下来的指标给你列出来集群的状态概要：

- `number_of_nodes` 和 `number_of_data_nodes` 这个命名完全是自描述的。
- `active_primary_shards` 指出你集群中的主分片数量。这是涵盖了所有索引的汇总值。
- `active_shards` 是涵盖了所有索引的_所有_分片的汇总值，即包括副本分片。
- `relocating_shards` 显示当前正在从一个节点迁往其他节点的分片的数量。通常来说应该是 0，不过在 Elasticsearch 发现集群不太均衡时，该值会上涨。比如说：添加了一个新节点，或者下线了一个节点。
- `initializing_shards` 是刚刚创建的分片的个数。比如，当你刚创建第一个索引，分片都会短暂的处于 `initializing` 状态。这通常会是一个临时事件，分片不应该长期停留在 `initializing` 状态。你还可能在节点刚重启的时候看到 `initializing` 分片：当分片从磁盘上加载后，它们会从 `initializing` 状态开始。
- `unassigned_shards` 是已经在集群状态中存在的分片，但是实际在集群里又找不着。通常未分配分片的来源是未分配的副本。比如，一个有 5 分片和 1 副本的索引，在单节点集群上，就会有 5 个未分配副本分片。如果你的集群是 `red` 状态，也会长期保有未分配分片（因为缺少主分片）



> [彻底解决 es 的 unassigned shards 症状](https://toutiao.io/posts/na8zgp/preview)

修改复制分片数

> PUT logstash-2019.05.29/_settings
> {
>   "number_of_replicas": 0
> }
>
> 



#### 4.6.2 扩容

扩容有垂直扩容(机器性能)，水平扩容(机器个数)，通常es是在水平上进行扩容，使用ES的玩家一开始看简介，都知道ES是分布式，很方便进行扩展，但是ES的扩展到底怎么影响,并提供了性能呢



我们知道，索引一创建，就明确了分片数，分片数是决定理论上能够存放最大的数据量。一般设计上分片数会大于节点数，所以至少一个节点上至少会有2个分片。操作分片是需需要消耗机器性能的，当一个机器上节点太多，节点上的分片太多，就能导致性能存在问题。



而扩容不是让能存等多的数据(已有分片数确定)，而是让已有的数据能更快的检索出来。扩容会降低节点上的分片数，使节点的性能提高





### 4.7 配置

> - cluster.name: Yun_elasticsearch
> - node.name: node@192.168.3.243
> - node.master: true
> - node.data: true
> - node.ingest: false 
> - node.ml: false 
> - cluster.remote.connect: false 
> - path.data: /home/unisound/elasticsearch-6.5.2/data
> - path.logs: /home/unisound/elasticsearch-6.5.2/logs
> - network.host: 192.168.3.243
> - http.port: 9200
> - discovery.zen.ping.multicast.enabled 这个设置把组播的自动发现给关闭了，为了防止其他机器上的节点自动连入。
> - discovery.zen.fd.ping_timeout和discovery.zen.ping.timeout是设置了节点与节点之间的连接ping时长discovery.zen.minimum_master_nodes 这个设置为了避免脑裂。比如3个节点的集群，如果设置为2，那么当一台节点脱离后，不会自动成为master。
> - discovery.zen.ping.multicast.enabled: false     # 禁止多播
> - discovery.zen.ping.unicast.hosts 这个设置了自动发现的节点。
> - action.auto_create_index: false. 这个关闭了自动创建索引。为的也是安全考虑，否则即使是内网，也有很多扫描程序，一旦开启，扫描程序会自动给你创建很多索引。

- action.auto_create_index: false
- discovery.zen.minimum_master_nodes







在bin/elasticsearch里面增加两行：

```
ES_HEAP_SIZE=4g
MAX_OPEN_FILES=65535
```

这两行设置了节点可以使用的内存数和最大打开的文件描述符数。



参考连接

-  [elasticsearch 集群](https://www.cnblogs.com/yjf512/p/4865930.html)
- [ES官方文档](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/important-configuration-changes.html#important-configuration-changes)





## 5 常见错误



### unassigned 分片

unassigned 分片：表示未分配的分片



产生这个 unassigned 分片的原因很多，在一个科普网站看到作者罗列了12种，如下面

1. INDEX_CREATED：由于创建索引的API导致未分配。
2. CLUSTER_RECOVERED ：由于完全集群恢复导致未分配。
3. INDEX_REOPENED ：由于打开open或关闭close一个索引导致未分配。
4. DANGLING_INDEX_IMPORTED ：由于导入dangling索引的结果导致未分配。
5. NEW_INDEX_RESTORED ：由于恢复到新索引导致未分配。
6. EXISTING_INDEX_RESTORED ：由于恢复到已关闭的索引导致未分配。
7. REPLICA_ADDED：由于显式添加副本分片导致未分配。
8. ALLOCATION_FAILED ：由于分片分配失败导致未分配。
9. NODE_LEFT ：由于承载该分片的节点离开集群导致未分配。
10. REINITIALIZED ：由于当分片从开始移动到初始化时导致未分配（例如，使用影子shadow副本分片）。
11. REROUTE_CANCELLED ：作为显式取消重新路由命令的结果取消分配。
12. REALLOCATED_REPLICA ：确定更好的副本位置被标定使用，导致现有的副本分配被取消，出现未分配。



这么多种情况，那我们是具体怎么定位呢。 我们可以通过下面一个API 快速获取信息

```
GET /_cluster/allocation/explain
```

>## cluster-allocation-explain API 历史
>
>`cluster-allocation-explain API`在ElasticSearch 5.0中初次引入,并在5.2版本中进行了重构. 这个API主要是为了方便解决下面两个问题:
>
>- 对于不能指派(unassigned)的分片: 解释这些分片不能被指派(到某个节点)的原因.
>- 对于已指派的分片: 解决这些分片指派到特定节点的理由.
>
>

- [cluster-allocation-explain](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-allocation-explain.html)

- [【译】使用explain API摆脱ElasticSearch集群RED苦恼](https://segmentfault.com/a/1190000008956708)



## 感谢

本文大多借鉴参考了[铭毅天下](https://blog.csdn.net/wojiushiwo987/column/info/deep-elasticsearch),  其他知识来源于自己实践，还有官网，也有别的一些博客，在下面给出引用的连接



### 参考链接

- [Elasticsearch索引管理利器——Curator深入详解](https://www.javazhiyin.com/28072.html)

- [Mastering Elasticsearch(中文版)](https://doc.yonyoucloud.com/doc/mastering-elasticsearch/index.html)
- [Elasticsearch学习，请先看这一篇！](https://blog.csdn.net/laoyang360/article/details/52244917)










