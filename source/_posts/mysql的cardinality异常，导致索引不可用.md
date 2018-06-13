---
title: mysql的cardinality异常，导致索引不可用
date: 2017-02-21 09:05:00
categories: MySQL
tags: MySQL
---
前段时间，一大早上，就收到报警，警告php-fpm进程的数量超过阈值。最终发现是一条sql没用到索引，导致执行数据库查询慢了，最终导致php-fpm进程数增加。最终通过analyze table feed_comment_info_id_0000 命令更新了Cardinality ，才能再次用到索引。
**排查过程如下：**
sql语句：

```
select id from feed_comment_info_id_0000 where obj_id=101 and type=1;
```

索引信息：

```
show index from feed_comment_info_id_0000
+---------------------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment |
+---------------------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
| feed_comment_info_id_0000 | 0 | PRIMARY  | 1 | id      | A | 6216 | NULL | NULL |     | BTREE | | 
| feed_comment_info_id_0000 | 1 | obj_type | 1 | obj_id  | A | 6216 | NULL | NULL |     | BTREE | | 
| feed_comment_info_id_0000 | 1 | obj_type | 2 | type    | A | 6216 | NULL | NULL | YES | BTREE | | 
| feed_comment_info_id_0000 | 1 | user_id  | 1 | user_id | A | 6216 | NULL | NULL |     | BTREE | | 
+---------------------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+
5 rows in set (0.00 sec)
```

通过explian查看时，发现sql用的是主键PRIMARY，而不是obj_type索引。通过show index 查看索引的Cardinality值，发现这个值是实际数据的两倍。感觉这个Cardinality值已经不正常，因此通过analyzea table命令对这个值从新进行了计算。命令执行完毕后，就可用使用索引了。

**Cardinality解释**
官方文档的解释：
An estimate of the number of unique values in the index. This is updated by running ANALYZE TABLE or myisamchk -a. Cardinality is counted based on statistics stored as integers, so the value is not necessarily exact even for small tables. The higher the cardinality, the greater the chance that MySQL uses the index when doing
总结一下：
1、它代表的是索引中唯一值的数目的估计值。如果是myisam引擎，这个值是一个准确的值。如果是innodb引擎，这个值是一个估算的值，每次执行show index 时，可能会不一样
2、创建Index时（primary key除外），MyISAM的表Cardinality的值为null，InnoDB的表Cardinality的值大概为行数；
3、值的大小会影响到索引的选择
4、创建Index时，MyISAM的表Cardinality的值为null，InnoDB的表Cardinality的值大概为行数。
5、可以通过Analyze table来更新一张表或者mysqlcheck -Aa来进行更新整个数据库
6、可以通过 show index 查看其值