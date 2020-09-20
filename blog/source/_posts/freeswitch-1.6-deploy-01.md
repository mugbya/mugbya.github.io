---
title: freeswitch-1.6部署-01
toc: true
date: 2020-09-18 10:27:21
tags:
 - 技术
categories: 
 - freeswitch
---


```yaml
首先强调下环境： centos6  freeswitch1.6 
```

## 1.基础


一开始是很不想写这篇文章的，网上部署的文章还是蛮多，官方也给出了各个版本下的部署文档 [Archived Linux Installation Instructions](https://freeswitch.org/confluence/display/FREESWITCH/Archived+Linux+Installation+Instructions)

不过发现照着做还是搞了很多，主要是差了很多依赖。官方给出的依赖如下

```shell
yum install git gcc-c++ autoconf automake libtool wget python ncurses-devel zlib-devel libjpeg-devel openssl-devel e2fsprogs-devel sqlite-devel libcurl-devel pcre-devel speex-devel ldns-devel libedit-devel

```


不过在实际过程中遇到很多问题，如

- 错误提示: You must install libopus-dev to build mod_opus
- 错误提示: Neither yasm nor nasm have been found
- 错误提示:  找不到 lua.h 等lua的头文件


每安装一个依赖后，然后去make发现又有新的问题，所以这里在依赖给出更全面一点，但是不保证在任何环境都适用

```shell
yum install git gcc-c++ autoconf automake libtool wget python ncurses-devel zlib-devel libjpeg-devel openssl-devel e2fsprogs-devel sqlite-devel libcurl-devel pcre-devel speex-devel ldns-devel libedit-devel yasm lua lua-devel opus-devel
```


安装执行如下命令

```shell

$ cd /usr/src

$ git clone -b v1.6 https://github.com/signalwire/freeswitch.git

$ cd freeswitch

$ ./bootstrap.sh -j

$ ./configure -C

$ make && make install


$ make sounds-install

$ make moh-install

```

最终安装完后如下图

![](/images/freeswitch/install.png)


启动

```shell
$ cd /usr/local/freeswitch

$ ./bin/freeswitch

```

查看启动过程中是否出现 

```log
2020-09-18 16:58:44.243406 [INFO] sofia.c:5782 Setting MAX Auth Validity to 0 Attempts
2020-09-18 16:58:44.275457 [ERR] sofia.c:3146 Error Creating SIP UA for profile: internal-ipv6 (sip:mod_sofia@[::1]:5060;transport=udp,tcp) ATTEMPT 1 (RETRY IN 5 SEC)
2020-09-18 16:58:44.276436 [INFO] switch_core_sqldb.c:1693 sofia:external Starting SQL thread.
2020-09-18 16:58:44.276636 [NOTICE] sofia_reg.c:3398 Added gateway 'example.com' to profile 'external'
2020-09-18 16:58:44.278290 [ERR] sofia.c:3146 Error Creating SIP UA for profile: external-ipv6 (sip:mod_sofia@[::1]:5080;transport=udp,tcp) ATTEMPT 1 (RETRY IN 5 SEC)
2020-09-18 16:58:44.518954 [NOTICE] sofia.c:5949 Started Profile internal [sofia_reg_internal]
2020-09-18 16:58:44.519047 [WARNING] sofia.c:2206 MSG Thread 0 Started
```

里面 ERR 是因为机器没有开启 ipv6. 如何确定请参考： [Linux关闭、开启、配置IPv6](https://www.jianshu.com/p/bb951b2c76b6)

解决方式：

```shell
$ cd /usr/local/freeswitch/conf/sip_profiles

$ mkdir bak

$ mv *-ipv6* back

```


## 2.配置介绍

配置文件在 /usr/local/freeswitch/conf 下，下面是conf下的目录

```
├── autoload_configs
├── chatplan
├── dialplan
│   ├── default
│   ├── public
│   └── skinny-patterns
├── directory
│   └── default
├── ivr_menus
├── jingle_profiles
├── lang
├── mrcp_profiles
├── sip_profiles
│   ├── back
│   │   └── external-ipv6
│   └── external
└── skinny_profiles
```

- **autoload_configs**：存放自动加载的配置文件
- chatplan：存放的是聊天计划配置文件
- **dialplan**：存放的是拨号计划配置文件
- **directory**：用户目录，存储跟用户相关的信息
- ivr_menus：IVR菜单配置文件
- jingle_profiles：连接Google Talk的相关配置文件
- lang：多语言支持配置文件
- mrcp_profiles：MRCP的相关配置，用于跟第三方语音合成和语音识别系统对接。
- **sip_profiles**：sip配置文件
- skinny_profiles：思科SCCP协议话机的配置文件

> autoload_configs: 文件夹下的modules.conf.xml：表示当freeswitch启动时自动装载哪些模块。此文件夹下其它xml：一般来说都是对应每个模块的配置文件
> 
> directory: 此文件夹下的的defalut目录是默认的用户目录配置，default下的xml文件是对应每个sip用户的，每个sip用户都有一个配置文件
> 
> sip_profiles: 此文件夹下的internal.xml：一个SIP profile，或称作一个SIP-UA，监听在本地IP及端口5060.此文件夹下的externa.xml：另一个SIP-UA，用作外部连接，端口5080。




## 3.感谢

在安装过程中这篇博文帮助很大

- [FreeSWITCH安装报错](https://www.cnblogs.com/hezhixiong/p/4797511.html)
