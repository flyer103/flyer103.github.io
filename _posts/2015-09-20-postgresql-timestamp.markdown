---
layout: post
title: "postgresql 中的 "timestamp with time zone" 和 " timestamp without timezone" 的区别"
date: 2015-09-20 18:00:17
categories: theory, postgresql, time
---
迁移下 2014-12-29 写的一篇 blog。

在 [postgresql-9.4 Date/Time Types][postgresql-9.4 Date/Time Types] 文档中可以发现有这两种类型的 timestamp:  

* timestamp with time zone  
* timestamp without time zone  

在理解这两种类型的区别前，需要先了解下这两个概念:  

* absolute timestamp  
  即 UTC (UTC+0) 时间。  
* relative timestamp  
  即当地时间，如北京的当地时间是 UTC+8 的时间。  

使用的一个最佳实践是时间类型都设为 `timestamp with time zone` 类型，只有在根据 timestamp 进行 partition 时才使用 `timestamp without time zone` 类型，因为 partition 必须使用 immutable 数据 (即在任何情况下数据取出来都一样)，而 *timestamp with time zone* 的数据值与 postgres 配置的 timezone 有关。    

这两种数据类型的区别是:  

* 以当地时间存储数据到 `timestamp with time zone` 类型的字段时，postgres 底层会以 UTC 时间存储，展示数据时会根据 postgres 设置的 timezone 显示为当时时间  
* 以当地时间存储数据到 `timestamp without time zone` 类型的字段时，postgres 底层以输入的数据进行存储，展示时会原样展示，与 postgres 设置的时区无关  


参考:  

* [Difference between timestamps with/without time zone in PostgreSQL][Difference between timestamps with/without time zone in PostgreSQL]  
* [Always Use TIMESTAMP WITH TIME ZONE][Always Use TIMESTAMP WITH TIME ZONE]  
* [Partitioning and Constraint Exclusion][Partitioning and Constraint Exclusion]  
* [Constraint exclusion can't process simple constant expressions?][Constraint exclusion can't process simple constant expressions?]  
* [which timestamp type to choose in a postgresql database?][which timestamp type to choose in a postgresql database?]  
* [postgresql9.4 doc Date/Time Types][postgresql9.4 doc Date/Time Types]  
* [世界时区][世界时区]  
* [wiki UTC][wiki UTC]  


[Difference between timestamps with/without time zone in PostgreSQL]: http://stackoverflow.com/questions/5876218/difference-between-timestamps-with-without-time-zone-in-postgresql
[Always Use TIMESTAMP WITH TIME ZONE]: http://justatheory.com/computers/databases/postgresql/use-timestamptz.html  
[Partitioning and Constraint Exclusion]: http://www.postgresql.org/docs/9.4/static/ddl-partitioning.html#DDL-PARTITIONING-CONSTRAINT-EXCLUSION
[Constraint exclusion can't process simple constant expressions?]: http://comments.gmane.org/gmane.comp.db.postgresql.performance/29681
[which timestamp type to choose in a postgresql database?: http://stackoverflow.com/questions/6151084/which-timestamp-type-to-choose-in-a-postgresql-database
[postgresql9.4 doc Date/Time Types]: http://www.postgresql.org/docs/9.4/static/datatype-datetime.html
[世界时区]: http://www.worldtimezone.com/index_cn.php
[wiki UTC]: http://en.wikipedia.org/wiki/Coordinated_Universal_Time
[postgresql-9.4 Date/Time Types]: http://www.postgresql.org/docs/9.4/static/datatype-datetime.html
