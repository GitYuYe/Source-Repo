---
title: lamp环境的搭建及配置
date: 2015-08-10 20:10:33
categories: Linux
tags: Linux
---
想着怎么也得写点什么,把最近整理的linux笔记发上来把,也算做个备份..

1. 下载Ubuntu Server
2. 在虚拟机上安装Ubuntu Server。根据安装引导过程一步步安装，跟在自己电脑安装Windows操作系统类似。安装中会设置一个用户名和密码，安装成功后显示输入用户名的提示。大概是下图中的样子：
3. 一般情况下，我们会使用远程管理工具，我这里使用的是xShell。下载xShell并安装在自己电脑，直接百度xShell在百度软件中心下载就行。新建连接. 填写Name，Protocol选择SSH，Host填写Ubuntu Server的IP 地址（在Ubuntu Server中查看IP地址的命令为ifconfig），Port Number默认22，点击OK..

New Session（2）即为刚刚新建的session，点击connect发现连接失败，因为Ubuntu Server还没有下载openssh Server，在Ubuntu Server中使用如下命令下载并开启SSH服务:

　 更新软件列表：sudo apt-get update

　　安装openssh：sudo apt-get install openssh-server

　　启动服务:/etc/init.d/ssh start

　　本机测试是否能够成功登录：ssh -l 用户名 本机ip,此时再connect即可成功.

1. 安装Apache服务器

　　在Xshell中输入命令: sudo apt-get install apache2

　　这个时候遇到一个错误：E：could not get lock /var/lib/dpkg/lock -open

　　原因是：可能是有另外一个程序正在运行，导致资源被锁不可用。而导致资源被锁的原因，可能是上次安装时没正常完成，而导致出现此状况。

　　解决办法是：

　　　　sudo rm /var/cache/apt/archives/lock

　　　　sudo rm /var/lib/dpkg/lock

　　再重新安装即可成功。

　　在XShell输入命令：apache2 -v

　　如果出现以下提示说明安装成功：

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221112525167-849963900.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221112525167-849963900.png)

　　再在浏览器中测试下，在地址栏中输入Ubuntu Server的IP地址，出现以下页面：

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221112832620-1696824308.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221112832620-1696824308.png)

\5. 安装PHP

　　在XShell输入命令：sudo apt-get install php

　　安装完成后输入命令检验是否安装成功：php -v

　　显示如下版本信息即为安装成功：

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221143344839-1145471163.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221143344839-1145471163.png)

　　检验Apache是否可以调用php命令：cat /etc/apache2/mods-enabled/php.load

　　出现如下错误提示信息

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221145513073-1913919068.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221145513073-1913919068.png)

　　输入命令：sudo apt-get install libapache2-mod-php 后，再输入命令cat /etc/apache2/mods-enabled/php7.0.load，不过却提示和PHP5有冲突，这个先不管，截图如下

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221150853745-783053617.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221150853745-783053617.png)

　　注意：由于前面的提示信息是找不到php.load这个文件，所以我进入到相应文件夹去看了下，确实没有这个文件，在输入命令sudo apt-get install libapache2-mod-php后，我再到文件夹里面去看后，有php.7.0.load这个文件。所以我将命令稍作修改：cat /etc/apache2/mods-enabled/php7.0.load

\6. 安装MySQL

　　输入命令：sudo apt-get install mysql-server

　　中途会出现设置MySQL administrator密码的提示。

　　安装MySQL的扩展：sudo apt-get install php-mysql

　　之后输入命令：cat /etc/php/7.0/mods-available/mysqli.ini

　　出现下图的提示信息说明PHP和MySQL之间的连接好了（PHP版本不同命令会不同，研究了好久才研究出来，我参照的视频里面安装的是php5，命令是：cat /etc/php5/conf.d/mysql.ini）

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221162914854-799196551.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161221162914854-799196551.png)

\7. 安装完毕。重启MySQL，Apache，命令是：

　　sudo service mysql restart

　　sudo service apache2 restart

　　tip：安装LAMP系统的Apache，MySQL，PHP也可以使用一条命令：

　　　　sudo apt-get install apache2 php mysql-server php-mysql

　　或者sudo tasksel install lamp-server

\8. 创建phpinfo服务器探针

　　cd /var/www/html

　　vim info.php

　　进入info.php文件后输入代码如下：

```
<?php
phpinfo();
?>
```

　　编辑文件，保存退出，到浏览器输入ip/info.php可以显示如下内容。（IP为Ubuntu server IP地址）

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229155507570-1030997216.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229155507570-1030997216.png)

　　修改info.php文件代码为：

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229160958336-1308086275.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229160958336-1308086275.png)

　　Note：涂掉的是MySQL安装时输入的admin用户密码

　　　　　按insert或者I进入到编辑模式，按ESC退出编辑模式，再按：wq，即保存退出；q！强制退出。

 编辑文件，保存退出，到浏览器输入ip/info.php可以显示如下内容。

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229161600523-2075807427.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229161600523-2075807427.png)

　　还可以安装一些php的扩展，如php7.0-gd,安装好后可在测试页面看到：

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229213141929-124078906.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229213141929-124078906.png)

\9. 一些Linux命令：

　　修改时区为上海：root@yaoxiao:/etc# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

　　设置代理　

　　1）通过export http代理使用apt-get（临时有效）

 　　　　在使用apt-get之前，在终端中输入以下命令

 　　　　export http_proxy=[http://proxy_addr:proxy_port](http://gityuanye.cn/2014/08/14/lamp%E7%8E%AF%E5%A2%83%E7%9A%84%E9%85%8D%E7%BD%AE/)

　　　　 取消代理使用

　　　　 export http_proxy=””

　　 2）apt.conf文件中配置http代理信息（永久有效）

 　　　　sudo gedit /etc/apt/apt.conf在您的apt.conf文件中加入下面这行

 　　　　Acquire::http::Proxy “[http://proxy_addr:proxy_port](http://gityuanye.cn/2014/08/14/lamp%E7%8E%AF%E5%A2%83%E7%9A%84%E9%85%8D%E7%BD%AE/)“;

 　　　　保存apt.conf文件即可

　　3）.bashrc文件中配置代理信息(apt-get, wget 等等)（全局有效）

　　　　 gedit ~/.bashrc 在.bashrc文件末尾添加如下内容

 　　　　export http_proxy=”[http://proxy_addr:proxy_port](http://gityuanye.cn/2014/08/14/lamp%E7%8E%AF%E5%A2%83%E7%9A%84%E9%85%8D%E7%BD%AE/)“

 　　　　保存文件，重新开启终端。

　　复制文件：cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 语法：cp {源文件} {目标文件}

\10. 使用filezilla管理Linux服务器上的文件

　　首先下载filezilla，配置：

　　[![img](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229211310648-440461883.png)](http://images2015.cnblogs.com/blog/1035135/201612/1035135-20161229211310648-440461883.png)

　　可能会遇到无法连接到root用户，解决办法如下：

　　 进入到配置文件　　vim /etc/ssh/sshd_config

　　 修改　　permitrootlogin：yes

　　 重启ssh服务　　service ssh restart

\11. 手动配置DNS

　　进入到配置文件hosts: sudo vim /etc/hosts　

　　在文件底部添加你需要的项，格式： .

　　　　　　　　　　示例: 192.168.190.130 video.imooc.com

　　　　　　　　　　　　　192.168.190.130 bbs.imooc.com

　　　　　　　　　　　　　192.168.190.130 oa.imooc.com

　　完成后保存退出，重启网络：sudo /etc/init.d/networking restart

　　到浏览器中输入video.imooc.com等三个域名检验，发现跟输入192.168.190.130指向同一个页面。最开始设置好以后在自己电脑上怎么也打不开“It Works”页面，后来发现在Windows客户端查看的话要设置Windows端的hosts文件：打开C:\Windows\System32\drivers\etc\hosts，加入三台虚拟主机的配置：192.168.190.130 video.imooc.com。

```
12. 用xshell远程连接到Ubuntu服务器创建虚拟主机文件
```

 （1）批量创建文件夹：sudo mkdir -p /var/www/html/{video,bbs,oa}

 （2）创建完文件夹后分别为这三个文件夹创建示例文件：

 　　d /video/

 　　sudo vim index.html

 （3） 编辑文件：“Welcome to Video Site”

　　　　重复（2）（3）两步给bbs、oa创建示例文件。

　　（5）创建虚拟主机文件：

　　　　默认情况下，apache有一个默认的虚拟主机文件叫000-default.conf。我们将会复制000-default.conf文件内容到我们新的虚拟主机配置文件中。

　　　　sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/video.local.conf

　　　　sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/bbs.local.conf

　　　　sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/oa.local.conf

　　（6）修改虚拟主机文件

　　　　确保虚拟主机配置文件末尾包含.conf扩展名。现在，修改video.local.conf文件以符合需求:

　　　　sudo vi /etc/apache2/sites-available/video.local.conf

　　　　修改后的文件如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<VirtualHost *:80>

　　　　# The ServerName directive sets the request scheme, hostname and port that

　　　　# the server uses to identify itself. This is used when creating

　　　　# redirection URLs. In the context of virtual hosts, the ServerName

　　　　# specifies what hostname must appear in the request's Host: header to

　　　　# match this virtual host. For the default virtual host (this file) this

　　　　# value is not decisive as it is used as a last resort host regardless.

　　　　# However, you must set it for any further virtual host explicitly.

　　　　#ServerName www.example.com

　　　　ServerAdmin webmaster@video.local
　　　　ServerName video.imooc.com
　　　　DocumentRoot /var/www/html/video/public_html

　　　　# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,

　　　　# error, crit, alert, emerg.

　　　　# It is also possible to configure the loglevel for particular

　　　　# modules, e.g.

　　　　#LogLevel info ssl:warn

　　　　ErrorLog ${APACHE_LOG_DIR}/error.log

　　　　CustomLog ${APACHE_LOG_DIR}/access.log combined

　　　　# For most configuration files from conf-available/, which are

　　　　# enabled or disabled at a global level, it is possible to

　　　　# include a line for only one particular virtual host. For example the

　　　　# following line enables the CGI configuration for this host only

　　　　# after it has been globally disabled with "a2disconf".

　　　　#Include conf-available/serve-cgi-bin.conf

　　　　</VirtualHost>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　　　重复（6）步骤修改bbs.local.conf oa.local.conf

　　　 （7）修改虚拟主机文件后，禁用默认的虚拟主机配置（000.default.conf)，然后启用新的虚拟主机配置，如下所示。

　　　　sudo a2dissite 000-default.conf

　　　　sudo a2ensite video.local.conf

　　　　sudo a2ensite bbs.local.conf

　　　　sudo a2ensite oa.local.conf

　　　　（8）重启Apache服务器，测试网页

　　　　sudo service apache2 restart

　　　　打开地址video.imooc.com bbs.imooc.com oa.imooc.com，页面如下：

　　　　[![img](http://images2015.cnblogs.com/blog/1035135/201703/1035135-20170313151018682-938984166.png)](http://images2015.cnblogs.com/blog/1035135/201703/1035135-20170313151018682-938984166.png)

```
13. MySQL数据存储目录迁移：
```

（1）停止数据库服务：sudo service mysql stop

（2）创建与MySQL文件夹权限一致的文件夹（一般是挂载硬盘，但目前学习阶段没有挂载硬盘）来存放备份：sudo mkdir /mysqldata

[![img](http://images2015.cnblogs.com/blog/1035135/201703/1035135-20170328111056279-1533018983.png)](http://images2015.cnblogs.com/blog/1035135/201703/1035135-20170328111056279-1533018983.png)

（3）修改mysqldata文件夹属性：sudo chown -vR mysql:mysql /mysqlcopy/mysql/

（4）修改mysqldata文件夹权限：sudo chmod -vR 700 /mysqlcopy/mysql/

（5）切换到root账户：su

（6）复制：cp -av /var/lib/mysql/* /mysqlcopy/mysql/

（7）退出root账户：exit（或Ctrl+D）

（8）修改MySQL配置文件：sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

输入/datadir（在文件中查找字符）找到MySQL的默认数据存放文件夹，修改为指向刚刚创建的/mysqlcopy/mysql/文件夹。

\14. 安装phpmyadmin

phpmyadmin是非常流行的MySQL管理工具，有两种方式安装：

（1）使用apt-get安装，然后创建一个软连接

　　sudo apt-get install phpmyadmin

　　sudo ln -s /usr/share/phpmyadmin/ /var/www/html/pma

（2）下载好phpmyadmin安装包，然后使用filezilla将安装包拖到 /var/www/html/ 下面

使用第二种方式安装完成后，打开[http://192.168.190.130/pma/试验一下是否安装成功。提示有如下错误。](http://192.168.190.130/pma/%E8%AF%95%E9%AA%8C%E4%B8%80%E4%B8%8B%E6%98%AF%E5%90%A6%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F%E3%80%82%E6%8F%90%E7%A4%BA%E6%9C%89%E5%A6%82%E4%B8%8B%E9%94%99%E8%AF%AF%E3%80%82)

[![img](http://images2015.cnblogs.com/blog/1035135/201703/1035135-20170328161938295-1644023818.png)](http://images2015.cnblogs.com/blog/1035135/201703/1035135-20170328161938295-1644023818.png)

安装php-mbstring即可解决：sudo apt-get install php-mbstring

```
15. MySql远程访问
```

[![img](http://images2015.cnblogs.com/blog/1035135/201703/1035135-20170328222120811-865189431.png)](http://images2015.cnblogs.com/blog/1035135/201703/1035135-20170328222120811-865189431.png)

输入用户名，选择权限后，新增用户成功。重启MySQL 服务。

用Navicat Premium远程连接,此时不能成功，我们需要修改如下文件：

　　sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

　　注释掉 bind-address：127.0.0.1

此时连接成功。