---
title: PHP过滤中文全角空格
date: 2017-08-10 10:10:33
categories: PHP
tags: PHP
---

做添加功能时过滤中文全角空格

`$params['name'] = preg_replace("/(\s|\ \;|　|\xc2\xa0)/", "", $params['name']);`

##### Html<textarea>标签默认有空格的解决方案

在Html页面中如果要定义一个多行文本的字段，如：“调整原因”、解决方案“”、“个人意见”等等，经常需要用到<textarea></textarea>标签。

正常情况下，我们都会根据编程习惯，将开始标签和结束标签分为两行来写，如下：`<textarea id="model" name="model" style="width:90%;height:60px;">

​                                </textarea>`

但是如果这么写，两个标签之间的任何文字甚至是空格都会被浏览器当做是textarea标签的value去进行加载，那么显示框在编辑时就会默认空几格

 如果要去掉空格应该这么写：`<textarea id="model" name="model" style="width:90%;height:60px;"></textarea>`

两个标签之间不管有没有参数都紧挨着就不会有默认的空格了