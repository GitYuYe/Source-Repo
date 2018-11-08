---
title: JS中判断两个数字的大小
date: 2018-02-13 10:01:00
categories: 其他
tags: 其他
---
​	今天写了一个form表单，提交前会判断一下开始机器号是否比结束机器号大（开始必须小于等于结束的）

原始写法是：

var start = $("#input_edit_changeInfo_machineStartNo").val();//开始机器号
var end = $("#input_edit_changeInfo_machineEndNo").val();//结束机器号
if(start>end){
pffsNotify("开始机器号不可大于结束机器号","warning");//pffsNotify()方法是自定的提示框
return false;
}

当我测试时候发现开始机器号写“5”结束机器号写“20”会跳出提示框，当时我就懵了，只有“0”“1”写在开始上回通过！

也就是5>20,6>20?吗？？？？？？？

肯定不是，上网一查，因为js中的var定义的变量默认是字符串，如果单纯的比较字符串的话，会出现错误，需要先转化为int类型在做比较。

所以转换一下类型就OK啦（将var 类型转换为int类型）







解决方案：

代码换成这样

var start = $("#input_edit_changeInfo_machineStartNo").val();//开始机器号
var end = $("#input_edit_changeInfo_machineEndNo").val();//结束机器号
if(parseInt(start)>parseInt(end)){
pffsNotify("开始机器号不可大于结束机器号","warning");
return false;
}

就OK了，
