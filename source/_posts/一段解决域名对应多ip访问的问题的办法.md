---
title: 一段解决域名对应多ip访问的问题的办法
date: 2017-04-11 21:05:00
categories: PHP
tags: PHP
---
一段解决域名对应多ip访问的问题的办法

PHP获取远程网页内容有多种方式，例如用自带的file_get_contents、fopen等函数。
引用

```
<?php   
echo file_get_contents("http://demo56.com/demo.php");   
?>
```

　　但是，在DNS轮询等负载均衡中，同一域名，可能对应多台服务器，多个IP。假设demo56.com被DNS解析到72.249.146.213、72.249.146.214、72.249.146.215三个IP，用户每次访问demo56.com，系统会根据负载均衡的相应算法访问其中的一台服务器。

　　上周做一个视频项目时，就碰到这样一类需求：需要依次访问每台服务器上的一个PHP接口程序（假设为abc.php），查询这台服务器的传输状态。

　　这时就不能直接用file_get_contents访问demo56.com/abc.php了，因为它可能一直重复访问某一台服务器。

　　而采用依次访问[http://72.249.146.213/abc.php、http://72.249.146.214/abc.php、http://72.249.146.215/abc.php的方法，在这三台服务器上的Web](http://72.249.146.213/abc.php%E3%80%81http://72.249.146.214/abc.php%E3%80%81http://72.249.146.215/abc.php%E7%9A%84%E6%96%B9%E6%B3%95%EF%BC%8C%E5%9C%A8%E8%BF%99%E4%B8%89%E5%8F%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84Web) Server配有多个虚拟主机时，也是不行的。

　　通过设置本地hosts也不行，因为hosts不能设置多个IP对应同一个域名。

　　那就只有通过PHP和HTTP协议来实现：访问abc.php时，在header头中加上demo56.com域名。于是，我写了下面这个PHP函数

```
<?php  
/************************ 
* 函数用途：同一域名对应多个IP时，获取指定服务器的远程网页内容 
* 参数说明： 
*    $ip   服务器的IP地址 
*    $host   服务器的host名称 
*    $url   服务器的URL地址（不含域名） 
* 返回值： 
*    获取到的远程网页内容 
*    false   访问远程网页失败 
************************/  
function HttpVisit($ip, $host, $url)     
{     
    $errstr = '';     
    $errno = '';  
    $fp = fsockopen ($ip, 80, $errno, $errstr, 90);  
    if (!$fp)     
    {     
         return false;     
    }     
    else    
    {     
        $out = "GET {$url} HTTP/1.1\r\n";  
        $out .= "Host:{$host}\r\n";     
        $out .= "Connection: close\r\n\r\n";  
        fputs ($fp, $out);     
  
        while($line = fread($fp, 4096)){  
           $response .= $line;  
        }  
        fclose( $fp );  
  
        //去掉Header头信息  
        $pos = strpos($response, "\r\n\r\n");  
        $response = substr($response, $pos + 4);  
      
        return $response;     
    }     
}  
  
//调用方法：  
$server_info1 = HttpVisit("72.249.146.213", "demo56.com", "/abc.php");  
$server_info2 = HttpVisit("72.249.146.214", "demo56.com", "/abc.php");  
$server_info3 = HttpVisit("72.249.146.215", "demo56.com", "/abc.php");  
?>
```