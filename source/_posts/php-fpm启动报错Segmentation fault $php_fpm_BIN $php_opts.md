---
title: php-fpm启动报错Segmentation fault $php_fpm_BIN $php_opts
date: 2016-05-30 19:00:00
categories: PHP
tags: PHP
---
今天QA使用php-fpm启动php报错。具体信息如下：

-bash-3.2$ ~/script/client/php-fpm restart
Gracefully shutting down php-fpm . done
Starting php-fpm [29-Jul-2013 16:11:55] NOTICE: [pool www] ‘user’ directive is ignored when FPM is not running as root
**/home/script/client/php-fpm: line 52: 11678 Segmentation fault $php_fpm_BIN $php_opts**
failed

最终发现是php中同时启用了apc和zend opcache两个扩展导致的。猜想应该是这两个扩展冲突，同时启用就会报上面的错误。
另外，一些扩展库用的编译库来运行库版本不一致引起的 ，比如gd用的是libpng15编译的，但运行环境却是libpng12的话 就容易有这类的问题。