---
title: http协议小记
date: 2015-11-06 18:15:00
categories: PHP
tags: PHP
---
> 1. http协议概述
>
> 超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。所有的WWW文件都必须遵守这个标准。1960年美国人Ted Nelson构思了一种通过计算机处理文本信息的方法，并称之为超文本（hypertext）
>
> 1. HTTP是一个客户端和服务器端请求和应答的标准
>
> 1. http协议特点
>
> a) **简单,**快速,只需要URL,传值的方法.
>
> b) **灵活**.
>
>  i. header(‘content-type:text/html;charset=utf-8’);
>
>  ii. header(‘content-type:image/jpeg’);
>
> **c)** 无连接, (服务器与客户端的连接总是要断开的)
>
> d) 无状态. http协议没有记忆功能. 页面与页面之间没有关系.
>
> 请求:
>
> ##### 1) HTTP请求的构成,
>
> 请求的构成: 请求行, 请求头, 空白行, 请求体
>
> ##### 2) 请求行的格式，
>
> 格式: 请求的方法 空格 URI信息 空格 协议的版本
>
> ##### 3) 请求头格式,
>
> 格式: key:空格value //属性: 空格 值
>
> **4) 请求体格式**
>
> get:无数据(get的参数在请求行中显示) post:携带数据信息
>
> 响应:
>
> ##### 1) http响应的构成
>
> 响应行 响应头 空白行 响应体
>
> ##### 2) 状态行的格式
>
> 响应行: 也称为状态行,消息行
>
> 格式: HTTP版本 空格 状态码 空格状态文本
>
> ##### 3) 常见的状态码:
>
> l 200 //响应成功.
>
> l 301 //域名的永久重定向
>
> l 302 //域名的临时重定向
>
> l 304 //本地加载资源
>
> l 403 //没有访问权限
>
> | 404 //访问的页面不存在
>
> 500 //指的是服务器内部的错误, 543,5XX
>
> php中发送请求的函数curl扩展函数
>
> curl发送请求主要分四步:
>
> 1) $curl = curl_init(url) 初始化请求的页面
>
> 2) curl_setopt($curl,参数,值);配置请求参数
>
> 3)result = curl _ exec ( $link ) 执行请求发送并接收返回的结果
>
> 4)curl_close ($link)关闭请求
>
> 常用几个参数:
>
> CURLOPT_HTTPHEADER：请求头，以数组形式设定，类似如下：
>
> array(‘host:url’,’Accept:text/html’,’Accept-Language:zh-cn,zh’)
>
> CURLOPT_HEADER：true/false, 设置返回的数据中是否带响应头信息
>
> CURLOPT_RETURNTRANSFER：true/false, 设定执行请求时是否返回数据，默认false就是直接输出数据
>
> (详细参数参考php官方网站…..)