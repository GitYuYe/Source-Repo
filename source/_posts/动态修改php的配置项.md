---
title: 动态修改php的配置项
date: 2016-08-13 14:00:00
categories: PHP
tags: PHP
---
我们一般修改php的配置项都是在php.ini中修改。在php,ini中的修改会影响到所有使用php的程序。假如我想让修改只在某个域名下生效，该如何做呢?

**使用ini_set()**
首先想到的可能是使用ini_set()方法在脚本中修改。但是这个只能修改作用域为PHP_INI_USER和PHP_INI_ALL的配置项。具体配置项作用域说明请查看 [PHP配置指令作用域说明](http://www.bo56.com/php%E9%85%8D%E7%BD%AE%E6%8C%87%E4%BB%A4%E4%BD%9C%E7%94%A8%E5%9F%9F%E8%AF%B4%E6%98%8E/)

**使用php_value**
如果我访问url时，程序每次执行都自动加载一个header.php文件。但是，如果是通过shell脚本方式执行，就不要加载这个文件了。要实现这个需求，我们需要用到 auto_prepend_file 这个配置想。这个配置想的作用域是 PHP_INI_PERDIR 。 也就是说不能通过ini_set()方法设置。那我们可以通过php_value进行设置。

如果是apache+php的组合，我们可以在apache的配置文件中加入如下指令即可。

> Php_value auto_prepend_file /home/www/bo56.com/header.php

如果是nginx+php组合，可以加入如下指令

> fastcgi_param PHP_VALUE “auto_prepend_file=/home/www/bo56.com/header.php”;

注意，nginx中多次使用 PHP_VALUE时，最后的一个会覆盖之前的。如果想设置多个配置项，需要写在一起，然后用换行分割。如：

> fastcgi_param PHP_VALUE “auto_prepend_file=/home/www/bo56.com/header.php \n auto_append_file=/home/www/bo56.com/external/footer.php”;