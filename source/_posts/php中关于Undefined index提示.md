---
title: php中关于Undefined index提示
date: 2016-03-09 21:10:00
categories: PHP
tags: PHP
---
我们在获取Get或者Post参数时，会发现页面会

报Undefined index提示,如下:

$type=$_GET[“type”];

注意：这是一个提示，而并不是错误。原因在于$_GET数组里面没有这个key，那么我们直接获取就会报这个提示。

**两种解决方法：**

**1、修改代码**

if(isset($_GET[“type”])

 $type=$_GET[“type”];

也就是做一次判断，防止报这个提示

**2、修改php.ini**

打开php.ini 。搜索error_reporting = E_ALL

如果找到了，把 这句话改成

error_reporting = E_ALL & ~E_NOTICE (注意前面 如果有分号也要去掉),

E_ALL 代表所有错误、警告、提示都爆出来。 ~ 这个符号代表排除，和E_NOTICE配合就是 排除“提示”

重启apache

个人推荐第二种。 要个毛提示啊，我这么写自然有我的道理，关你php什么事。

还增加apache 日志容量，增加了压力，还能省掉我们写代码的时间。多好