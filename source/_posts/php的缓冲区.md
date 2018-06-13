---
title: php的缓冲区
date: 2016-05-18 20:00:00
categories: PHP
tags: PHP
---
大家应该都知道php文件最终在浏览器上显示，走过3个缓冲阶段：
php buffer=》web server buffer=》浏览器buffer。

先从php buffer开始讲起。
**php buffer**
php运行的结果先放入缓冲区（buffer），只有当缓冲区满了或者php运行完毕，才将数据输出去。

缓冲区是通过php.ini中的output_buffering变量控制。output_buffering的默认值是off，可以设置大于0的数值来打开buffer。
但是这里需要注意的是：
1）使用ini_set是无法修改buffer的设置。
2）不管php.ini中output_buffering设置，cli模式下的php始终默认是output buffering为关闭的。但是你可以通过ob_start()将buffer打开。

大家都说：ob_start()是将php buffer打开，ob_end_flush()是将php buffer关闭。但是php.ini中php buffer是关闭的，再次调用ob_end_flush()会报warning。

另外，ob_*系列的函数是操作php本身的输出缓冲区。可以使用ob_flush()将php 缓冲区的内容强制输出。

**web server buffer**
这里主要将apache和nginx的缓冲区。
1、apache buffer
当php的输出数据给apache服务器时，它也会做一层buffer（也将数据放入它的缓冲区，当缓冲区数据满或执行完毕时，才输出数据）。

若想关闭缓冲区，可以在php层使用flush()来强制将缓冲区数据输出。
fulsh() 的工作原理：在apache module的sapi下, flush会通过调用sapi_module的flush成员函数指针, 间接的调用apache的api: ap_rflush刷新apache的输出缓冲区, 当然手册中也说了, 有一些apache的其他模块, 可能会改变这个动作的结果.例如mod_gzip，可能自己进行输出缓冲区，这将导致flush()函数产生的结果不会立即被发送到客户端浏览器。

2、nginx buffer
nginx使用fastcgi缓冲区来缓冲数据。很遗憾的是，fastcgi是强制将buffer打开的，无法关闭缓冲区。

有人有可能会想，无法关闭可以将buffer设置的足够小，来使缓冲数据输出，达到无缓冲的效果。但是这个想法无法实现。
原因一：fastcgi buffer无法识别小于1k的数值。
原因二：受参数之间大小关系的影响。

具体可以看看fastcgi的一些buffer设置。
fastcgi_buffer_size是用来存储response的header数据。
fastcgi_buffers是用来存储response的内容数据.
fastcgi_busy_buffers_size是用来控制同时传输到客户端的buffer数量。一旦fastcgi_buffers设置的 buffer被写入，直到buffer里面的数据被完整的传输完（传输到客户端），这些buffer将会一直处在busy状态，我们不能对这些 buffer进行任何别的操作。所有处在busy状态的buffer size加起来不能超过fastcgi_busy_buffers_size。

参数之间大小关系：
fastcgi_busy_buffers_size<(all fastcgi_buffers - one buffer) 并且fastcgi_busy_buffers_size>=max (fastcgi_buffer_size, one fastcgi _buffers)。
例如,在nginx.conf配置中有:
fastcgi_buffers 4 128k
fastcgi_buffer_size 256k
那么fastcgi_busy_buffers_size<(4*128k - 4k) 并且fastcgi_busy_buffers_size>=max(256k, 128k)
其中，4k（one buffer的大小）是linux系统默认的缓存大小，即一个内存页。

若fastcgi_buffer_size设置的很小，会导致header过小的错误。你也同样无法保证设置的值会满足所有的情况。

要注意的是：
1）flush, 严格来讲, 这个只有在PHP做为apache的Module(handler或者filter)安装的时候, 才有实际作用. 它是刷新WebServer(可以认为特指apache)的缓冲区.所以在nginx下，flush()函数是无法起作用的。

**browser buffer**
在 php端无法关闭浏览器buffer(至少目前我没有研究出来，若有哪位高人研究出来了，请一定要记得分享)。为了使得数据及时输出，可以在发送真正内容 数据前，发送一些空格来填满浏览器的buffer。浏览器的buffer一满，就会将其他新输出的数据输出。但是不同的浏览器会设置不同的buffer大 小。为了保险期间，可以发送4096个空格，因为目前比较流行的浏览器的buffer还没有超过4k(一个内页大小)。