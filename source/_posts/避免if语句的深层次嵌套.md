---
title: 避免if语句的深层次嵌套
date: 2018-11-15 20:10:33
categories: PHP
tags: PHP
---
# 

## 开发中，如何减少if else语句的使用

代码中嵌套的if/else结构往往导致代码不美观，也不易于理解。面向过程的开发中代码有大量的IF ELSE，在java中可以用一些设计模式替换掉这些逻辑，那么在js中是否也有类似的方法用来尽可能减少代码中的if/else嵌套呢？

有人认为：if else多就多呗，只要可读性强，维护起来方便。jQuery.fn.init里就是一堆if else判断，难道要质疑jQuery作者的水平了？并不是说if else多就不好，关键是看用的地方，jQuery.fn.init里除了if else判断简洁点，难道要改成switch？就算用工厂模式，还不是得做大量的if判断。

代码整洁强迫症患者必须要来个抛砖引玉：

### 1.

```
if(a为真){
    a=a
}else{
    a=b
}
```

可写成：a = a || b

### 2.

```
if(a==b){
    a=c
}else{
    a=d
}
```

可写成：a = (a==b) ? c : d

### 3.

后台接口通常会返回这种数据：

fruit: 0 // 0=苹果，1=梨子，2=桔子，3=柠檬，4=芒果...

这时写if...else是最痛苦的。从冲哥那偷来个方法：

```
var _f = ['苹果','梨子','桔子','柠檬','芒果'];
shuiguo = _f[fruit];
```

## 建议：

## 第一步：优化if逻辑

人们考虑的东西到时候，都会把最可能发生的情况先做好准备。优化if逻辑的时候也可以这样想：把最可能出现的条件放在前面，把最不可能出现的条件放在后面，这样程序执行时总会按照带啊名的先后顺序逐一检测所有的条件，知道发现匹配的条件才会停止继续检测。

`if` 的优化目标：最小化找到分支之前所判断条件体的数量。if优化的方法：将最常见的条件放在首位。

```
if (i < 5) {
    // 执行一些代码
 } else if (i > 5 && i < 10) {
    // 执行一些代码
 } else {
    // 执行一些代码
 }
```

例如上面这个例子，只有在 `i` 值经常出现小于5的时候是最优化的。如果i值经常大于或者等于10的话，那么在进入正确的分支之前，就必须两次运算条件体，导致表达式的平均运算时间增加。 `if` 中的条件体应该总是按照从最大概率到最小概率排列，以保证理论速度最快。

## 第二步：尽量少使用else

如果在函数中，可以使用 `if + return` ，先判断错误条件，然后立马结束函数，防止进入 `else` 分支。

举个简单的例子，后端返回数据，前端根据状态进行不同操作

```
$.ajax().done(function (res) {
    if (res.state === 'SUCCESS') {
        //TODO
    } else if (res.state === 'FAIL') {
        //TODO
    } else 
        //TODO
    }
});
```

这里用if else不挺好的么，有啥问题么？不过我个人倾向于switch。

解决方法：

### 1. switch/case

switch和if else在性能上是没有什么区别的，主要还是根据需求进行分析和选择。

- 如果条件较小的话选用if else比较合适。
- 相反，条件数量较大的话，就建议选用switch。

一般来说，if else适用于两个离散的值或者不同的值域。如果判断多个离散值，使用switch更加合适。

在大多数的情况下switch比if else运行的更加快。

在大多数情况下，switch的性能不会比if else低。switch的确在实质上跟if else if 完全一样的效果，不过在很多情况下，使用switch要比if else方便不少

比如经典的值等分支，匹配一些状态常量的时候，比if else结构方便许多，不用反复写xx == yy

```
$.ajax().done(function (res) {
    switch (res.state) {
        case 'SUCCESS':
            //TODO
            break;
        case 'FAIL':
            //TODO
            break;
        default :
            //TODO
    }
});
```

注意：千万不要忘记在每一个case语句后面放一个break语句。也可以放一个return或者throw。case语句匹配expression是用===而不是==。

### 2.hash 表

```
if (key == "Apple") {
    val = "Jobs";
} else if (key == "microsoft"){
    val = "Gates";
} else if (key == "Google"){
    val = "Larry";
} 
```

这个也可以用 `switch case` 解决，不过推荐的方法是 hash 表：

```
var ceos = {"Apple":"Jobs", "microsoft":"Gates", "Google":"Larry"};
val = ceos[key];
```

### 3.重构，用 OO 里面的继承或者组合

```
1.如果是狗，则汪汪
2.如果是猫，则喵喵
3.如果是羊，则咩咩
4.如果是鸭，则嘎嘎
```

可以重构一下，改成 OO。

```
*定义类： 动物（或者接口）
*定义方法：叫
*定义子类：狗、猫、羊、鸭
*重写方法 ----  叫
```

也就是说将同的判断按照功能，写成函数，这样也便于阅读

比如我有一个方法根据类型获取名称，用if else会这样

```
function getName(type) {
    if (type === 'monkey') {
        return 'monkey name';
    } else if (type === 'cat') {
        return 'cat name';
    } else {
        return 'dog name';
    }
}
```

如果写成函数，改成下面的会更好

```
function getName(type) {
    var data = {
        monkey: 'monkey name',
        cat: 'cat name',
        dog: 'dog name'
    }
    return data[type] ? data[type] : data['dog'];
}
```

硬要把设计模式添加到JS里不是不可以，但是要看情况。生搬硬套过来的东西然并卵啊。写代码记住三个字即可，短简易。代码短，读起来简单，维护容易，如果在性能和代码长度上二选一，我肯定选代码短，性能不行，加台服务器就是了。而冗长的代码并不是加个程序员就能搞定的。

保持着这个心态写代码，写出的东西离设计模式也不会差太多了。