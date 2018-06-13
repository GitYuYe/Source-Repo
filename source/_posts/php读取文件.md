---
title: php读取文件
date: 2016-04-17 10:10:00
categories: PHP
tags: PHP
---
**首先，我们如果做个php网站，里面有个文件，要让php读取它，并给用户下载。很常见的方法是**

 $file = “aaa.rar”;
 header(“Content-type: application/octet-stream”); //必须更改头，否则浏览器无法识别
 header(‘Content-Disposition: attachment; filename=”‘ . basename($file) . ‘”‘); //弹出的下载框 需要文件名
 readfile($file); //对于这句话 大家注意一下

**这里实际上 有几个过程**

1、php首先IO读取

2、发送给apache缓冲区

3、apache发送给浏览器

然后用户才能下载。

这里面如果你网站PV比较高的话，那么php那一步一样的读取是很耗费IO资源的。所以今天我们要解决的是让apache直接读取，避开php那一步。

**于是我们就要用到一个apache（apache2系列)模块。 这个模块其实很简单、很好用。**

**windows部署。**

这个实在太简单。找到你的apache目录，找到modules目录。然后把下载下来的 mod_xsendfile.so 放进去

然后打开你的apache配置文件，随便找个犄角旮旯 放入这么一行

LoadModule xsendfile_module modules/mod_xsendfile.so **#前面不能有 ‘#’ 号，否则等于没放**

------

**重启 Apache 就算好了**

------

**linux部署。**

首先我们要安装apache的一个扩展工具–apxs， apxs是一个为Apache HTTP服务器编译和安装扩展模块的工具。

这里我们使用yum安装。 (yum是什么不知道的话，就不要看下去了。对身体不好)

yum -y install httpd-devel

装好后执行

which apxs

默认会出现 /use/sbin/apxs

结了。

然后利用 apxs 安装xsendfile模块到apache中

apxs -cia mod_xsendfile.c

注意，你必须吧这个mod_xsendfile.c 下载下来传到你的linux服务器中，然后cd到这个文件所在目录执行上述命令。

-cia选项表示编译（compile）、安装（install）以及启动（active）–> a选项会自动添加如下模块在配置文件httpd.conf中 .如果你不喜欢可以把a选项去掉。然后手工加入模块引用，请参照windows 方法。没啥区别

**重启 apache**

接下里就是写php代码了。

其实就一句话不一样。

 $file = “aaa.rar”;
 header(“Content-type: application/octet-stream”); //必须更改头，否则浏览器无法识别 你们这帮货在玩下载
 header(‘Content-Disposition: attachment; filename=”‘ . basename($file) . ‘”‘); //弹出的下载框 需要文件名
**header(“X-Sendfile: “.$file); //让xsendfile模块去发送这个文件，这样就绕过了 php读取这一层。效率明显提高了**

注意，之前还有一些教大家修改配置文件、修改模块的文章和方法。如果你改完后 **就是不记得重启apache**，我真的没法帮你更多了。