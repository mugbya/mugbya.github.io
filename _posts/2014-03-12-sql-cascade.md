---
layout: post
title: SQL外键中的级联关系
thread: 23
categories: 记录
tags: sql 
---
  

           环境：MariaDB 5.5 (mysql)   
           

#### 外键约束对子表的含义: 

  如果在父表中找不到候选键,则不允许在子表上进行insert/update 

#### 外键约束对父表的含义: 

  在父表上进行update/delete以更新或删除在子表中有一条或多条对应匹配行的候选键时,父表的行为取决于：在定义子表的外键时指定的on update/on delete子句, InnoDB支持５种方式, 分列如下 
  
- . cascade方式 
   在父表上update/delete记录时，同步update/delete掉子表的匹配记录 
   On delete cascade从mysql3.23.50开始可用; on update cascade从mysql4.0.8开始可用 

- . set null方式 
   在父表上update/delete记录时，将子表上匹配记录的列设为null 
   要注意子表的外键列不能为not null 
   On delete set null从mysql3.23.50开始可用; on update set null从mysql4.0.8开始可用 

- . No action方式 
   如果子表中有匹配的记录,则不允许对父表对应候选键进行update/delete操作 
   这个是ANSI SQL-92标准,从mysql4.0.8开始支持 

- . Restrict方式 
   同no action, 都是立即检查外键约束 

- . Set default方式 
   解析器认识这个action,但Innodb不能识别，不知道是什么意思．．． 
  
  注意：trigger不会受外键cascade行为的影响,即不会解发trigger 

在mysql中，与ＳＱＬ标准相违背的三点 
1.       如果在父表中有多个key值相同，那么在进行外键check时,会当成有相同key值的其他行不存在; 比如当定义了一个restrict行为外键时,一个子表行对应到多个父表行(具有相同key值), Innodb不允许删除父表上的所有这些行 

2.       父子表是同一个表,自我参照时不允许指定on update cascade, on update set null 
从mysql4.0.13开始，允许同一个表上的on delete set null 
从mysql4.0.21开始，允许同一个表上的on delete cascade 
但级联层次不能超出15 

3, Innodb在检查unique,constraint约束时，是row by row而不是语句或事务结束; 
  SQL标准中对constraint的检查是在语句执行完成时 
  
  参考链接：<a href="http://androider.iteye.com/blog/352332" target="_blank">androider</a>
  
  首次编辑于：2014-03-13
  
  
