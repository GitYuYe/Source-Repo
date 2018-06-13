---
title: h5+css3
date: 2015-12-13 09:15:00
categories: HTML
tags: HTML
---
html+html5+css+css3

1.怪异模式和标准模式(box-sizing)

首先，两种模式可以利用box-sizing属性进行自行选择：

　　标准模式：box-sizing:**content-box;**

　　怪异模式：box-sizing:**border-box;**

**两种模式的区别：**

　　**标准模式会被设置的padding撑开，而怪异模式则相当于将盒子的大小固定好，再将内容装入盒子。盒子的大小并不会被padding所撑开。**

例：

　　 .box{
 box-sizing:border-box; //没有添加时，按照标准模式计算， 添加时按照怪异模式解析
 width:200px;
 height:200px;
 border:2px solid black;
 padding:50px;
 margin:50px;
 }

　　 标准模式：盒子总宽度/高度 = **内容区宽度 /高度+padding+border + margin**。

 盒子总宽度/高度 = **width/height + margin。**

2.关于sessionstorage和localstorage

**1、web Storage**

web Storage 是HTML 5引入的一个重要的功能，在前端开发的过程中会经常用到，它可以在客户端本地存储数据，类似cookie，但其功能却比cookie强大的多。cookie的大小只有4Kb左右（浏览器不同，大小也不同），而web Storage的大小有5MB。其API提供的方法有以下几种：

 setItem (key, value) —— 保存数据，以键值对的方式储存信息。

 getItem (key) —— 获取数据，将键值传入，即可获取到对应的value值。

 removeItem (key) —— 删除单个数据，根据键值移除对应的信息。

 clear () —— 删除所有的数据

 key (index) —— 获取某个索引的key

**2、localStorage**

 localStorage的生命周期是永久性的。假若使用localStorage存储数据，即使关闭浏览器，也不会让数据消失，除非主动的去删除数据，使用的方法如上所示。localStorage有length属性，可以查看其有多少条记录的数据。

 localStorage 相对sessionStorage简单一点，需要注意的地方不是很多。

 **3、sessionStorage**

sessionStorage 的生命周期是在浏览器关闭前。也就是说，在整个浏览器未关闭前，其数据一直都是存在的。sessionStorage也有length属性，其基本的判断和使用方法和localStorage的使用是一致的。

 <1> 页面刷新不会消除数据

 <2>只有在当前页面打开的链接，才可以访sessionStorage的数据；

 <3>使用window.open打开页面和改变localtion.href方式都可以获取到sessionStorage内部的数据

- **sessionStorage 、localStorage 和 cookie 之间的区别**
  共同点：都是保存在浏览器端，且同源的。
- 区别：cookie数据始终在同源的http请求中携带（即使不需要），即cookie在浏览器和服务器间来回传递；cookie数据还有路径（path）的概念，可以限制cookie只属于某个路径下。存储大小限制也不同，cookie数据不能超过4k，同时因为每次http请求都会携带cookie，所以cookie只适合保存很小的数据，如会话标识。
- 而sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存。sessionStorage和localStorage 虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大。
- 数据有效期不同，sessionStorage：仅在当前浏览器窗口关闭前有效，自然也就不可能持久保持；localStorage：始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据；cookie只在设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭。
- 作用域不同，sessionStorage**不在**不同的浏览器窗口中共享，即使是同一个页面；localStorage 在所有同源窗口中都是共享的；cookie也是在所有同源窗口中都是共享的。Web Storage 支持事件通知机制，可以将数据更新的通知发送给监听者。Web Storage 的 api 接口使用更方便。

缓存处理

dns 缓存

当你需要多次浏览一个网站的时候,需要多次解析这个网站的dns,这个时候本地就会产生dns缓存,当每次请求时候先确认你的缓存有没有过期,没有过期则先加载缓存中的数据

cdn 缓存

访问一个网站服务器 的时候,采取就近原则,即寻找最近的节点来访问,减少跨域,跨区请求的次数

浏览器缓存

浏览器会保存你访问过的网页的静态文件信息,再次访问优先读取缓存,如果缓存过期或者被清除则请求服务器

服务器缓存

指的是将需要频繁访问的网络内容存放在离用户较近、访问速度更快的系统中，以提高内容访问速度的一种技术。缓存服务器就是存放频繁访问内容的服务器。

图片预处理技术

图片懒加载，在页面上的未可视区域可以添加一个滚动条事件，判断图片位置与浏览器顶端的距离与页面的距离，
如果前者小于后者，优先加载。
如果为幻灯片、相册等，可以使用图片预加载技术，将当前展示图片的前一张和后一张优先下载。
如果图片为 css 图片，可以使用 CSSsprite，SVGsprite，Iconfont、Base64 等技术。
如果图片过大，可以使用特殊编码的图片，加载时会先加载一张压缩的特别厉害的缩略图，以提高用户体验。
如果图片展示区域小于图片的真实大小，则因在服务器端根据业务需要先行进行图片压缩，图片压缩后大小与展
示一致。