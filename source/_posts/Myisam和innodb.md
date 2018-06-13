---
title: Myisam和innodb
date: 2015-09-27 15:10:20
categories: MySQL
tags: MySQL
---
今天来写一点关于数据表的东西把…

## Innodb引擎

Innodb引擎提供了对数据库ACID事务的支持，并且实现了SQL标准的四种隔离级别，关于数据库事务与其隔离级别的内容请见数据库事务与其隔离级别这篇文章。该引擎还提供了行级锁和外键约束，它的设计目标是处理大容量数据库[系统](https://www.2cto.com/os/)，它本身其实就是基于MySQL后台的完整数据库系统，MySQL运行时Innodb会在内存中建立缓冲池，用于缓冲数据和索引。但是该引擎不支持FULLTEXT类型的索引，而且它没有保存表的行数，当SELECT COUNT(*) FROM TABLE时需要扫描全表。当需要使用数据库事务时，该引擎当然是首选。由于锁的粒度更小，写操作不会锁定全表，所以在并发较高时，使用Innodb引擎会提升效率。但是使用行级锁也不是绝对的，如果在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表。

## MyIASM引擎

MyIASM是MySQL默认的引擎，但是它没有提供对数据库事务的支持，也不支持行级锁和外键，因此当INSERT(插入)或UPDATE(更新)数据时即写操作需要锁定整个表，效率便会低一些。不过和Innodb不同，MyIASM中存储了表的行数，于是SELECT COUNT(*) FROM TABLE时只需要直接读取已经保存好的值而不需要进行全表扫描。如果表的读操作远远多于写操作且不需要数据库事务的支持，那么MyIASM也是很好的选择。

## 两种引擎的选择

大尺寸的数据集趋向于选择InnoDB引擎，因为它支持事务处理和故障恢复。数据库的大小决定了故障恢复的时间长短，InnoDB可以利用事务日志进行数据恢复，这会比较快。主键查询在InnoDB引擎下也会相当快，不过需要注意的是如果主键太长也会导致性能问题，关于这个问题我会在下文中讲到。大批的INSERT语句(在每个INSERT语句中写入多行，批量插入)在MyISAM下会快一些，但是UPDATE语句在InnoDB下则会更快一些，尤其是在并发量大的时候。

## Index——索引

索引（Index）是帮助MySQL高效获取数据的数据结构。MyIASM和Innodb都使用了树这种数据结构做为索引，关于树我也曾经写过一篇文章树是一种伟大的数据结构，只是自己的理解，有兴趣的朋友可以去[阅读](http://book.2cto.com/)。下面我接着讲这两种引擎使用的索引结构，讲到这里，首先应该谈一下B-Tree和B+Tree。

### B-Tree和B+Tree

B+Tree是B-Tree的变种，那么我就先讲B-Tree吧，相信大家都知道红黑树，这是我前段时间学《算法》一书时，实现的一颗红黑树，大家可以参考。其实红黑树类似2,3-查找树，这种树既有2叉结点又有3叉结点。B-Tree也与之类似，它的每个结点做多可以有d个分支（叉），d称为B-Tree的度，如下图所示，它的每个结点可以有4个元素，5个分支，于是它的度为5。B-Tree中的元素是有序的，比如图中元素7左边的指针指向的结点中的元素都小于7，而元素7和16之间的指针指向的结点中的元素都处于7和16之间，正是满足这样的关系，才能高效的查找：首先从根节点进行二分查找，找到就返回对应的值，否则就进入相应的区间结点递归的查找，直到找到对应的元素或找到null指针，找到null指针则表示查找失败。这个查找是十分高效的，其时间复杂度为O(logN)（以d为底，当d很大时，树的高度就很低），因为每次检索最多只需要检索树高h个结点。

[![B-Tree](https://www.2cto.com/uploadfile/Collfiles/20150327/2015032710000558.png)](https://www.2cto.com/uploadfile/Collfiles/20150327/2015032710000558.png)

接下来就该讲B+Tree了，它是B-Tree的变种，如下面两张图所示：

[![B+Tree](https://www.2cto.com/uploadfile/Collfiles/20150327/2015032710000561.png)](https://www.2cto.com/uploadfile/Collfiles/20150327/2015032710000561.png)
[![B+Tree2](https://www.2cto.com/uploadfile/Collfiles/20150327/2015032710000665.png)](https://www.2cto.com/uploadfile/Collfiles/20150327/2015032710000665.png)

从图中就可以看出，B+Tree的内部结点不存储数据，只存储指针，而叶子结点则只存储数据，不存储指针。并且在其每个叶子节点上增加了一个指向湘…�”/database/DB2/“ target=”_blank” class=”keylink”>DB2tK219O92rXjtcTWuNXro6zV4rj208W7r8zhuN/H+Lzkt8POyrXE0NTE3KOsscjI59Tatdq2/tXFzbzW0NKqsunRr7z8zqq00zE4tb00ObXEy/nT0Mr9vt2jrLWx1dK1vTE4uvOjrNa70OjLs9fFvdq147rN1rjV68uz0PKx6cD6vs2/ydLU0ru0ztDUt8POyrW9y/nT0Mr9vt292rXjo6y8q7TzzOG1[vc](https://www.2cto.com/kf/ware/vc/)HLx/i85LLp0a/Qp8LKoaM8L3A+DQo8aDMgaWQ9”myisam引擎的索引结构”>MyISAM引擎的索引结构

MyISAM引擎的索引结构为B+Tree，其中B+Tree的数据域存储的内容为实际数据的地址，也就是说它的索引和实际的数据是分开的，只不过是用索引指向了实际的数据，这种索引就是所谓的**非聚集索引**。

### Innodb引擎的索引结构

MyISAM引擎的索引结构同样也是B+Tree，但是Innodb的索引文件本身就是数据文件，即B+Tree的数据域存储的就是实际的数据，这种索引就是**聚集索引**。这个索引的key就是数据表的主键，因此InnoDB表数据文件本身就是主索引。

因为InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。

并且和MyISAM不同，InnoDB的辅助索引数据域存储的也是相应记录主键的值而不是地址，所以当以辅助索引查找时，会先根据辅助索引找到主键，再根据主键索引找到实际的数据。所以Innodb不建议使用过长的主键，否则会使辅助索引变得过大。建议使用自增的字段作为主键，这样B+Tree的每一个结点都会被顺序的填满，而不会频繁的分裂调整，会有效的提升插入数据的效率