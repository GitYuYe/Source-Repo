---
title: PHP支持htaccess实现伪静态的注意事项
date: 2016-03-21 19:10:00
categories: PHP
tags: PHP
---
第一步：

在apache配置文件XXX.conf(默认的是httpd.conf,如果你手工新建的那就不是这个名字) 中找到#LoadModule rewrite_module modules/mod_rewrite.so去掉前面的#号

第二步：

在apache配置文件httpd.conf 中找到<Directory /与</Directory内的AllowOverride None改为AllowOverride All

第三步：创建.htaccess 文件

在网站根目录下新建 .htaccess文件。注意用记事本估计不太好建，用编辑器都能建(推荐phpdesigner或者Hbuilder)

.htaccess的文件示例

 RewriteEngine On
 RewriteRule ^reg /index.php?type=reg [L] **// 这时你访问 xxx/reg 出来的内容等同于你直接访问\****index.php?type=reg***\***
 RewriteCond %{QUERY_STRING} ^from=(.*)
 RewriteRule ^login$ /index.php?type=login&from=%1 [L]
 RewriteRule ^login /index.php?type=login [L]