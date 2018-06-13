---
title: session和cookie
date: 2015-11-30 08:15:00
categories: PHP
tags: PHP
---
session和cookie

1.设置cookie

Setcookie(名,值,过期时间,有效路径,有效域名)

 说明:

1) 名称: cookie名称, 为获取cookie值作准备

2) 值: cookie的值, 注意cookie值的类型. 只能是字符串

1. 字符串:

2. 数组形式:

   a) 设置形式：setcookie(‘c1[k1]’,值)

   b) 读取形式：$_COOKIE [‘c1’][‘k1’]

2.读取cookie

 通过超全局数组$_COOKIE获取

3.删除COOKIE的方法

1) 将时间过期; Setcookie(‘名’,值,time()-1);

2) 将时间过期; Setcookie(‘名’,值,time()-999999);

3) Setcookie(‘名’);

4) Setcookie(‘名’,’’);

2.session

1. 开启SESSION会话功能 session_start();

2. 获取当前的SESSION的ID值 Session_id();

3. 设置session的值 $_SESSION[‘名称’] = 值; //值的类型没有限制

4. 读取SESSION数据

5. 删除session Unset($_SESSION[‘username’])

6. 销毁session_destroy();

   

session的php.ini配置选项

　　php.ini文件和Session有关的几个常用配置选项：

　　session.auto_start = 0 ; 在请求启动时初始化session

　　session.cache_expire = 180 ; 设置缓存中的会话文档在 n 分钟后过时

　　session.cookie_lifetime = 0 ; 设置按秒记的cookie的保存时间，相当于设置Session的过期时间，为0时表示直到浏览器被重启

　　session.auto_start=1，这样就无需每次使用session之前都要调用session_start()不建议使用.但启用该选项也有一些限制，如果确实启用了 session.auto_start，则不能将对象放入会话中，因为类定义必须在启动会话之前加载以在会话中重建对象。

　　session.cookie_path = / ; cookie的有效路径

　　session.cookie_domain = ; cookie的有效域

　　session.name = PHPSESSID; 用在cookie里的session的名字

　　session.save_handler = files ; 用于保存/取回数据的控制方式

　　session.save_path = /tmp ; 在 save_handler 设为文件时传给控制器的参数， 这是数据文件将保存的路径.

　　session.use_cookies = 1 ; 是否使用cookies

　　session.setMaxInactiveInterval(30 * 60);//设置单位为秒，设置为-1永不过期

如何设置一个30分钟过期的Session?

 使用memcache, redis等, okey, 这种答案是一种正确答案

 1. 设置Cookie过期时间30分钟, 并设置Session的lifetime也为30分钟.

 2. 自己为每一个Session值增加Time stamp.

 3. 每次访问之前, 判断时间戳.