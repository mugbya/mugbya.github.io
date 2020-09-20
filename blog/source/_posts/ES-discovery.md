---
title: ES之集群自动发现
toc: true
date: 2020-09-18 20:55:16
tags:
 - 技术
categories:
 - elasticsearch 
---



> 该篇是从 [ES版本变化史](http://blog.mugbya.me/2020/05/26/ES%E7%89%88%E6%9C%AC%E5%8F%98%E5%8C%96%E5%8F%B2/) 中单独抽出来的



## 1. 历史

在 5、6版本中，集群发现由 `Zen discovery` 内置模块负责，它提供单播和基于文件的发现，也可以通过扩展插件支持云环境和其他形式的发现。

`Zen discovery` 包含如下一些模块

- ping: 使用ping方式发现别的机器
- seed nodes: 简而言之，所有节点机器配置相同的 初始种子节点(master类型的节点)，然后种子节点先启动，后面别的节点启动后就找种子节点，通过种子节点来完成集群发现

  > 使用 cassandra 的用户应该很熟悉 seed nodes 这个配置



seed nodes 实现分为两种方式：Unicast(单播)、file-based(文件方式)

- Unicast: 使用该参数配置 `discovery.zen.ping.unicast.hosts`

- file-based：使用外部文件来配置种子节点。当文件更改时，ES无需重启而会自动加载该文件



| 发现方式         | 差异                           |
| ---------------- | ------------------------------ |
| Unicast(单播)    | 静态：改动所有节点`需要`会重启 |
| file-based(文件) | 动态：改动所有节点 `无需` 重启 |

    
<br>

`Unicast(单播)` 的配置形式如下:


```yaml
discovery.zen.ping.unicast.hosts: ["12.12.12.12:10801"]
```

<br>
                  

要是用文件方式，则需要先在 `elasticsearch.yml`进行如下配置：

```yaml
discovery.zen.hosts_provider: file
```



`file-based`的配置路径在： `$ES_PATH_CONF/unicast_hosts.txt`, 形式如下：

```yaml
10.10.10.5
10.10.10.6:9305
10.10.10.5:10005
# an IPv6 address
[2001:0db8:85a3:0000:0000:8a2e:0370:7334]:9301

```

参考链接：[ES官方6.8](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/modules-discovery-zen.html)


## 2. 新版



在 7 以后的版本 就不在提 `Zen discovery`, 不过还是使用的 `seed nodes` 方式，也支持静态更动态两种配置。不同的是 **`参数名发生了变化`**。



在 7 版本就不提Unicast(单播) 了，而是 `settings-based` ，file-based(文件)则不变。



`settings-based` 的配置形式如下:

```yaml
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 
   - seeds.mydomain.com 
```

>**注**
>
>如果不指定端口，则默认为 transport.port



要是用文件方式，则需要先在 `elasticsearch.yml`进行如下配置：

```yaml
discovery.seed_providers: file
```



`file-based`的配置路径在： `$ES_PATH_CONF/unicast_hosts.txt`, 形式如下：

```yaml
10.10.10.5
10.10.10.6:9305
10.10.10.5:10005
# an IPv6 address
[2001:0db8:85a3:0000:0000:8a2e:0370:7334]:9301
```

参考链接：[ES官方7.0](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/modules-discovery-hosts-providers.html)








