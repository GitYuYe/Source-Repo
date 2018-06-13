---
title: php-fpm的慢执行日志slow log，分析php性能问题
date: 2016-07-16 14:00:00
categories: PHP
tags: PHP
---
众所周知，mysql有slow query log，根据慢查询日志，我们可以知道那些sql语句有性能问题。作为mysql的好搭档，php也有这样的功能。如果你使用php-fpm来管理php的话，你可以通过如下选项开启。
PHP 5.3.3 之前设置如下：

5s

 

logs/php-fpm-slowlog.log

PHP 5.3.3 之后设置以下如下：
request_slowlog_timeout = 5s
slowlog = /usr/local/php/log/php-fpm-slowlog.log

说明：
request_slowlog_timeout 是脚本超过多长时间 就可以记录到日志文件
slowlog 是日志文件的路径

开启后，如果有脚本执行超过指定的时间，就会在指定的日志文件中写入类似如下的信息：

> [13-Mar-2015 16:54:49][pool www] pid 18575
> script_filename = /home/web/htdocs/sandbox_canglong/test/tt.php
> [0x0000000003a00dc8] curl_exec() /home/web/htdocs/sandbox_canglong/test/tt.php:2
> [0x0000000003a00cd0] exfilter_curl_get() /home/web/htdocs/sandbox_canglong/test/tt.php:6

日志说明：
script_filename 是入口文件
curl_exec() ： 说明是执行这个方法的时候超过执行时间的。
exfilter_curl_get() ：说明调用curl_exec()的方法是exfilter_curl_get() 。
每行冒号后面的数字是行号。

开启后，在错误日志文件中也有相关记录。如下：

> [13-Mar-2015 15:55:37] WARNING: [pool www] child 18575, script ‘/home/web/htdocs/sandbox_canglong/test/tt.php’ (request: “GET /test/tt.php”) executing too slow (1.006222 sec), logging
> [13-Mar-2015 15:55:37] NOTICE: child 18575 stopped for tracing
> [13-Mar-2015 15:55:37] NOTICE: about to trace 18575
> [13-Mar-2015 15:55:37] NOTICE: finished trace of 18575