---
title: mysql索引合并,一条sql可以使用多个索引
date: 2016-11-07 18:11:00
categories: MySQL
tags: MySQL
---
mysql的索引合并并不是什么新特性。早在mysql5.0版本就已经实现。之所以还写这篇博文，是因为好多人还一直保留着一条sql语句只能使用一个索引的错误观念。本文会通过一些示例来说明如何使用索引合并。

##### 什么是索引合并

下面我们看下mysql文档中对索引合并的说明：
The Index Merge method is used to retrieve rows with several range scans and to merge their results into one. The merge can produce unions, intersections, or unions-of-intersections of its underlying scans. This access method merges index scans from a single table; it does not merge scans across multiple tables.
根据官方文档中的说明，我们可以了解到：
1、索引合并是把几个索引的范围扫描合并成一个索引。
2、索引合并的时候，会对索引进行并集，交集或者先交集再并集操作，以便合并成一个索引。
3、这些需要合并的索引只能是一个表的。不能对多表进行索引合并。

##### 使用索引合并有啥收益

简单的说，索引合并，让一条sql可以使用多个索引。对这些索引取交集，并集，或者先取交集再取并集。从而减少从数据表中取数据的次数，提高查询效率。

##### 怎么确定使用了索引合并

在使用explain对sql语句进行操作时，如果使用了索引合并，那么在输出内容的type列会显示 index_merge，key列会显示出所有使用的索引。如下：
[![index_merge_sql](http://www.bo56.com/wp-content/uploads/2015/06/index_merge_sql.png)](http://www.bo56.com/wp-content/uploads/2015/06/index_merge_sql.png)

在explain的extra字段中会以下几种：
Using union 索引取并集
Using sort_union 先对取出的数据按rowid排序，然后再取并集
Using intersect 索引取交集

你会发现并没有 sort_intersect，因为根据目前的实现，想索引取交集，必须保证通过索引取出的数据顺序和rowid顺序是一致的。所以，也就没必要sort了。

##### sort_union索引合并的示例

### 数据表结构

### 数据

### 使用索引合并的案例

### 未使用索引合并的案例

### sort_union总结

从上面的两个案例大家可以发现，相同模式的sql语句，可能有时能使用索引，有时不能使用索引。是否能使用索引，取决于mysql查询优化器对统计数据分析后，是否认为使用索引更快。
因此，单纯的讨论一条sql是否可以使用索引有点片面，还需要考虑数据。

##### union索引合并使用案例

### 数据表结构

数据结构和之前有所调整。主要调整有如下两方面：
1、引擎从myisam改为了innodb。
2、组合索引中增加了id，并把id放在最后。

### 数据

数据和上面的数据一样。

### 使用索引合并的案例

### union总结

相同的数据，相同的sql语句，只是数据表结构有所调整，就从sort_union变为了union。有以下几个原因：
1、只要通过索引取出的数据已经按rowid进行了排序，就可以使用union。
2、组合索引中在最后加id字段，目的就是通过索引前两个字段取出的数据是按id排序。
3、把引擎从myisam改为innodb，目的就是让id和rowid的顺序一致。

##### intersect使用案例

数据结构和数据和union案例中的一致。

### 使用索引合并的案例

```
mysql> explain select * from test where (key1_part1=1 and key1_part2=1) and (key2_part1=1 and key2_part2=1)\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: test
         type: index_merge
possible_keys: key1,key2
          key: key2,key1
      key_len: 8,8
          ref: NULL
         rows: 3
        Extra: Using intersect(key2,key1); Using where; Using index
1 row in set (0.02 sec)
```