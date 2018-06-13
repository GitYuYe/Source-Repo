---
title: 源码安装Coreseek Makefile.in找不到解决办法
date: 2015-09-20 20:15:22
categories: Linux
tags: Linux
---
1、出现如下错误：

config.status: creating Makefile
config.status: WARNING: ‘Makefile.in’ seems to ignore the –datarootdir setting
config.status: error: cannot find input file: src/Makefile.in

###### 通过yum安装autoconf automake

###### [root@linux mmseg-3.2.14]# yum -y install autoconf automake

###### [root@linux mmseg-3.2.14]# aclocal

###### aclocal:configure.in:26: warning: macro AM_PROG_LIBTOOL’ not found in library`

###### [root@linux mmseg-3.2.14]# yum -y install libtool`

###### [root@linux mmseg-3.2.14]# aclocal`

###### [root@linux mmseg-3.2.14]# libtoolize –forcePutting files in AC_CONFIG_AUX_DIR, `config’.

###### [root@linux mmseg-3.2.14]# automake –add-missing

###### [root@linux mmseg-3.2.14]# autoconf

###### [root@linux mmseg-3.2.14]# autoheader

###### [root@linux mmseg-3.2.14]# make clean

然后重新执行configure命令