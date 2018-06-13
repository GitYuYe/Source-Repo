---
title: PHP返回内容过长时被nginx截断的解决办法
date: 2017-02-01 17:05:00
categories: PHP
tags: PHP
---
查看了html源代码，发现html源代码被截断了。因此，导致网页内容显示不全。
之后的整个分析过程绕了一大圈，即是tcpdump，又是用tcpflow进行网络包分析。最后，还是从nginx的错误日志中发现了端倪。
在nginx的错误日志中发现如下信息：

```
2016/03/29 06:08:10 [crit] 7042#0: *3 open() "/var/lib/nginx/tmp/fastcgi/4/00/0000000004" failed (13: Permission denied) while reading upstream, client: 117.72.224.240, server: www.bo56.com, request: "GET /wp-admin/post-new.php HTTP/1.1", upstream: "fastcgi://127.0.0.1:9010", host: "www.bo56.com", referrer: "http://www.bo56.com/wp-admin/edit.php"
```

错误日志显示， Permission denied，说明没权限。把这个目录设置为 775 权限问题解决。网页也就正常显示了。
如果PHP返回的内容过大，nginx会把一部分内容先存到文本文件中，等全部内容都接收完毕后，再一并发送到客户端。

在使用tcpdump抓包分析的时候，发现nginx进程主动向PHP进程发送了reset标识，终止通信。可能，nginx发现权限有问题，就关闭了连接。只是猜测，没有验证。图如下：

[![tcpdump_reset2](http://www.bo56.com/wp-content/uploads/2016/03/tcpdump_reset2.png)](http://www.bo56.com/wp-content/uploads/2016/03/tcpdump_reset2.png)

[使用tcpdump抓包教程](http://www.bo56.com/%E8%B0%83%E8%AF%95%E5%88%A9%E5%99%A8%E4%B9%8Btcpdump/)

## 经验教训

这次算是一次比较失败的问题排查。本来可以几分钟解决的事情，却花费了很长的时间。
切记，遇到问题，不要盲目的不断尝试。首先，要看日志查找线索。利用掌握的知识合理的推理分析。