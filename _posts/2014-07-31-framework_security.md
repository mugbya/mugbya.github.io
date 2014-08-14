---
layout: post
title: "web框架安全学习"
categories: 记录
tags: 安全 

---

在java web学习中，一直没有太过关注安全的问题;如今有幸学习并了解。这里给出自己的学习链接：

>  - <a href="http://drops.wooyun.org/tips/236" target="_blank" >攻击JavaWeb应用[3]-SQL注入[1]</a>  


此篇博文对web 安全 / 框架安全有启示性的提示。 


在传统的方式操作数据库时，我们采用拼装sql语句，这样会导致SQL注入漏洞。所以在JDBC中有两种方式来操作数据库。我们一般参用预编译的方式，且用 '?'这种占位的方式来处理sql语句，不在拼装sql语句，这样，在一定层面上解决sql注入问题。  


在客户端将请求参数传到后端处理时，不管用Struts2 或者 SpringMVC这种框架来处理，其底层都会调用Servlet来处理我们的请求。而在Servlet中处理请求的方法有： doPost 、doGet 、 service 这三个方法。而service集前两种之和。所以这也隐约带给我们另一种安全隐患。就这个特性，可能被：




