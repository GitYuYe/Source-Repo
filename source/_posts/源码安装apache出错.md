---
title: 源码安装apache出错
date: 2015-08-30 17:11:22
categories: Linux
tags: Linux
---
 今日编译apache时出错：

\#./configure –prefix……检查编辑环境时出现：

checking for APR… no
configure: error: APR not found . Please read the documentation

解决办法：

1.下载所需软件包：

```
wget http://archive.apache.org/dist/apr/apr-1.4.5.tar.gz  wget http://archive.apache.org/dist/apr/apr-util-1.3.12.tar.gz  wget http://jaist.dl.sourceforge.net/project/pcre/pcre/8.10/pcre-8.10.zip
```

2.编译安装：

```
yum remove apr-util-devel apr apr-util-mysql apr-docs apr-devel apr-util apr-util-docs
```

具体步骤如下:

a:解决apr not found问题>>>>>>

```
[root@xt test]# tar -zxf apr-1.4.5.tar.gz  [root@xt test]# cd  apr-1.4.5  [root@xt apr-1.4.5]# ./configure --prefix=/usr/local/apr  [root@xt apr-1.4.5]# make && make install
```

b:解决APR-util not found问题>>>>

```
[root@xt test]# tar -zxf apr-util-1.3.12.tar.gz  [root@xt test]# cd apr-util-1.3.12  [root@xt apr-util-1.3.12]# ./configure --prefix=/usr/local/apr-util -with- apr=/usr/local/apr/bin/apr-1-config  [root@xt apr-util-1.3.12]# make && make install
```

c:解决pcre问题>>>>>>>>>

```
[root@xt test]#unzip -o pcre-8.10.zip  [root@xt test]#cd pcre-8.10  [root@xt pcre-8.10]#./configure --prefix=/usr/local/pcre  [root@xt pcre-8.10]#make && make install
```

4.最后编译Apache时加上：

–with-apr=/usr/local/apr \

–with-apr-util=/usr/local/apr-util/ \

–with-pcre=/usr/local/pcre

成功编译完成~