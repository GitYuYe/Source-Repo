---
title: vmware安装的centos7中没有出现eth0网卡，也没有ip，不能上网
date: 2017-07-15 20:10:33
categories: Linux
tags: Linux
---
解决办法: 

1.编辑网卡的配置文件 vi /etc/sysconfig/network-scripts/ifcfg-ens33 将里面的NAME和DEVICE项修改为eth0 

2.重命名网卡配置文件ifcfg-ens33为ifcfg-eth0 

3.编辑/etc/default/grub并加入“net.ifnames=0 biosdevname=0 ”到GRUBCMDLINELINUX变量 

4.运行命令grub2-mkconfig -o /boot/grub2/grub.cfg 来重新生成GRUB配置并更新内核参数。 

5.重启系统 