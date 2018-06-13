---
title: merge引擎
date: 2015-10-07 18:00:00
categories: MySQL
tags: MySQL
---
[Mysql MERGE引擎简介](http://www.cnblogs.com/luojianqun/p/4903743.html)

一. 什么是MERGE**引擎**MERGE存储引擎把一组MyISAM数据表当做一个逻辑单元来对待，让我们可以同时对他们进行查询。

**二. 应用场景**如果需要把日志纪录不停的录入MySQL数据库，并且每天、每周或者每个月都创建一个单一的表，而且要时常进行来自多个表的合计查询，MERGE表这时会非常简单有效。

**三. 举例**假设有如下两表

```
1 CREATE TABLE `t1` (   
2 `id` int(10) unsigned NOT NULL AUTO_INCREMENT,   
3 `log` varchar(45) ,   
4 PRIMARY KEY (`id`) 
5 ) ENGINE=MyISAM;
```

```
1 CREATE TABLE `t2`(   
2 `id` int(10) unsigned NOT NULL AUTO_INCREMENT,   
3 `log` varchar(45) ,   
4 PRIMARY KEY (`id`) 
5 ) ENGINE=MyISAM;
```

假设t1，t2中都有如下记录

+—-+——-+

| id | log |

+—-+——-+

| 1 | test1 |

| 2 | test2 |

| 3 | test3 |

+—-+——-+

建立MERGE表

```
1 CREATE TABLE `t` (   
2 `id` int(10) unsigned NOT NULL AUTO_INCREMENT,   
3 `log` varchar(45) NOT NULL,   
4 PRIMARY KEY (`id`) 
5 ) ENGINE=MERGE UNION=(t1, t2) INSERT_METHOD=LAST;
```

执行select * from t;将会得到如下结果
+—-+——-+
| id | log |
+—-+——-+
| 1 | test1 |
| 2 | test2 |
| 3 | test3 |
| 1 | test1 |
| 2 | test2 |
| 3 | test3|
+—-+——-+

从效果上看，t1,t2的记录如同在一张表里一样被罗列了出来。当然，看了这个结果你一定会有一些疑问，在下一节里我们会慢慢解释。现在我们主要来解释一下上面MERGE表的建表语句。
1）ENGINE=MERGE
指明使用MERGE引擎，有些同学可能见到过ENGINE=MRG_MyISAM的例子，也是对的，它们是一回事。
2）UNION=(t1, t2)
指明了MERGE表中挂接了些哪表，可以通过alter table的方式修改UNION的值，以实现增删MERGE表子表的功能。
3）INSERT_METHOD=LAST
INSERT_METHOD指明插入方式，取值可以是：0 不允许插入；FIRST 插入到UNION中的第一个表； LAST 插入到UNION中的最后一个表。
4）MERGE表及构成MERGE数据表结构的各成员数据表必须具有完全一样的结构。每一个成员数据表的数据列必须按照同样的顺序定义同样的名字和类型，索引也必须按照同样的顺序和同样的方式定义。
**四. Cookie问答**1）建表时UNION指明的子表如果存在相同主键的记录会怎么样？
相同主键的记录会同时存在于MERGE中，就像第三节中的例子所示。但如果继续向MERGE表中插入数据，若数据主键已存在则无法插入。换言之，MERGE表只对建表之后的操作负责。
2）若MREGE后存在重复主键，按主键查询会是什么结果？
顺序查询，只出现一条查询记录即停止。比如第三节中的例子，如果执行

```
1 select * from t where id=1;
```

只会得到结果
+—-+——–+
| id | log |
+—-+——–+
| 1 | test1 |
+—-+——–+

3）直接删除一个子表会出现什么情况，正确删除的方式是怎样的？
MERGE表会被破坏，正确方式是用alter table方式先将子表从MERGE表中去除，再删除子表。
以第三节中的例子为例，执行如下操作

```
1 alter table t ENGINE=MRG_MyISAM UNION=(t1) INSERT_METHOD=LAST;
```

可以从MERGE表中去除t2，这里你可以安全的对t2进行任何操作了。

4）误删子表时，如何恢复MERGE表？
误删子表时，MERGE表上将无法进行任何操作。
方法1，drop MERGE表，重建。重建时注意在UNION部分去掉误删的子表。
方法2，建立MERGE表时，会在数据库目录下生成一个.MRG文件，比如设表名为t，则文件名为t.MRG。
文件内容类似：
t1
t2
\#INSERT_METHOD=LAST
指明了MGEGE表的子表构成及插入方式。
可以直接修改此文件，去掉误删表的表名。然后执行flush tables即可修复MERGE表。

5）MERGE的子表中之前有记录，且有自增主键，则MERGE表创建后，向其插入记录时主键以什么规则自增？
以各表中的AUTO_INCREMENT最大值做为下一次插入记录的主键值。
比如t1的自增ID至6，t2至4，则创建MERGE表后，插入的下一条记录ID将会是7

6）两个结构完全相同的但已存在数据的表，是否一定可以合成一个MEREGE表？
从实验的结果看，不是这样的，有时创建出的表，无法进行任何操作。
所以，推荐的使用方法是先有一个MERGE表，里面只包含一张表，当一个这个表的的大小增长到一定程度（比如200w）时，创建另一张空表，将其挂入MERGE表，然后继续插入记录。

7）删除MERGE表是否会对子表产生影响？
不会

8）MREGE表的子表的ENGIN是否有要求？
有的，必须是MyISAM表

附:
官方给出的关于MERGE表存在的一些问题
<http://dev.mysql.com/doc/refman/5.1/zh/storage-engines.html#merge-table-problems>
如果需要把日志纪录不停的录入MySQL数据库，并且每天、每周或者每个月都创建一个单一的表，而且要时常进行来自多个表的合计查询，MERGE表这时会非常简单有效