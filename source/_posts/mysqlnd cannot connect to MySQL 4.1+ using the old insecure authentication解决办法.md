---
title: mysqlnd cannot connect to MySQL 4.1+ using the old insecure authentication解决办法
date: 2016-08-25 20:00:00
categories: PHP
tags: PHP
---
mysqlnd是个好东西。不仅可以提高与mysql数据库通信的效率，而且也可以方便的设置一些超时。如，连接超时，查询超时。
但是，使用mysqlnd的时候，有个地方需要注意。就是服务端的密码格式不能使用旧的16位的存储格式，而要使用新的41位的存储格式。
如果，服务端的密码格式是16位，那么就会报错。信息如下：
Fatal error: Uncaught exception ‘PDOException’ with message ‘SQLSTATE[HY000] [2000] mysqlnd cannot connect to MySQL 4.1+ using the old insecure authentication. Please use an administration tool to reset your password with the command SET PASSWORD = PASSWORD(‘your_existing_password’). This will store a new, and more secure, hash value in mysql.user. If this user is used in other scripts executed by PHP 5.2 or earlier you might need to remove the old-passwords flag from your my.cnf file’ in /home/hailong.xhl/test.php:8

如何查看自己的密码是否符合要求，so easy。

上面的密码是旧的16位格式。如果想改成新的41位格式，通过以下命令就可以。

修改完密码后，还需要在配置文件中修改下old_passwords选项。把值设置为0。即，
old_passwords=0
然后重启mysql。