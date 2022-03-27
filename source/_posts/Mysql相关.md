---
layout: posts
title: Mysql相关
date: 2021-04-27 21:13:05
tags: [mysql]
categories: 面试篇
---

#### Mysql有哪些数据引擎，分别有什么特点

Innodb/MyISAM/Memory/Archive

<!--more -->

#### Innodb用的是B+树的数据结构，那它是如何定位到某一条记录的

![image-20200607161014044](image-20200607161014044.png)

Innodb中默认每一页大小是16kb，而页里面如果数据太多会进行分组，例如每6行记录为一组，页目录里面记录的就是每组的第一个记录的主键（索引）。

![image-20200607162147239](image-20200607162147239.png)

#### mvcc机制



#### redo和undo文件的区别



#### Mysql里面的执行计划--explain



#### 唯一索引和普通索引的区别

①对于查询来说：两种索引的性能是没差别，主要的差别是在更新索引上，mysql的读取是按页来的，每一页的大小是16k

细分以下两种情况：

1.需要更新的索引在内存中，修改的结果会保存到change buffer

1）唯一索引：找到对应的位置，判断有没有冲突，没有则将更新写入到change buffer，执行结束。

2）普通索引：找到对应的位置，不需要判断，将更新写入到change buffer，执行结束。

2.需要更新的索引不在内存中

1）唯一索引：先将磁盘数据读到内存中，找到对应的位置，判断冲突是否存在，不存在则写入change buffer，执行结束。

2）普通索引：找到对应的位置，不需要判断，将更新写入到change buffer，执行结束。





### 什么情况下会产生using temporary 和using filesort

**文件内排序**:单表查询时，如果order by的字段，无法走索引，此时便会产生文件内排序，filesort函数会优先使用内存(sort_buffer_size大小)排序，如果不够，便会产生临时文件。

**临时表**：①使用union查询

②当order by 和group by的字段不一致

③在多表连接下，order by 字段不是驱动表

④使用distinct查询并且加上了order by

⑤from里使用了子查询



临时表也分为两种：**磁盘临时表** 和 **内存临时表**

磁盘临时表一般由用户显式创建:create temporary table

内存临时表一般由优化器隐式创建，例如上面描述的几种情况。如果内存临时表大小超过了设定

> min(tmp_table_size,max_heap_table_size)

则会转化为磁盘临时表。



在5.6前，磁盘临时表使用的是MyISAM引擎，5.7以后可以选择Innodb或者MyISAM。

5.6和5.7中，内存临时表都只能使用Memory引擎，在8.0后可以选择Template引擎。