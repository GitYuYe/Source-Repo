---
title: 注意sql语句中的通配符，别掉坑里面!
date: 2016-10-10 10:21:00
categories: MySQL
tags: MySQL
---
**现象：**
有一个表 action_conf，数据如下：
[![table](http://www.bo56.com/wp-content/uploads/2014/09/table.jpg)](http://www.bo56.com/wp-content/uploads/2014/09/table.jpg)

如果想获取以**exp_site10**开头的en_name的记录，sql语句该如何写？
so easy！

很自信的在idb中执行了这条sql，就会发现结果并不是所预期的。
你会发现，执行上面的sql会把所有以 **exp_site_10**开头的记录都列出来了。

**原因：**
其实，这都是sql中的通配符在作怪。在sql中，下划线_是一个通配符，能匹配任何单一字符。
既然直到原因，修改sql就很容易了。正确的sql应该是：

在通配符前面增加转移字符后，mysql就会把通配符视为普通字符。

**进阶：**
通配符整理：
通配符 说明
% 替代一个或多个字符
_ 仅替代一个字符
[charlist] 字符列中的任何单一字符
[^charlist]或[!charlist] 不在字符列中的任何单一字符