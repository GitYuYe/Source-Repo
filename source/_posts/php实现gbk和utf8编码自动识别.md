---
title: php实现gbk和utf8编码自动识别
date: 2016-06-07 15:00:00
categories: PHP
tags: PHP
---
目前中文网页主流的编码为gbk和utf8两种编码。因此，我们做编码识别的前提是，编码不是gbk就是utf8.
编码自动识别的基本思想如下：
1.看给定的字节串是否符合utf8编码规则。如果不符合则为gbk编码。具体utf8编码规则件日志《[utf8编码规则](http://www.bo56.com/utf8%E7%BC%96%E7%A0%81%E8%A7%84%E5%88%99/)》。
2.如果给定的字节串中没有符合utf8三字节规则的，则为gbk编码。中文在utf8中占三个字节。
3.如果给定的字节串能对应上gbk编码中的中文，且无法对应上utf8编码中的中文，则为gbk编码。
4.特殊情况，特殊处理。如 “鏈條” 和 “瑷媄”。
总体思想是，先默认为utf8编码，再根据一些非utf8编码的情况逐步筛选。

准确率和效率还是不错的。20万多条query，1秒钟就可以处理完。

php代码如下：

```
function detect_encoding($str){
    $len = strlen($str);
    $encoding = "utf8";
    $is_utf8_chinese = false;
    for ($i = 0; $i < $len; $i++) { 
        if ( (ord($str[$i]) >> 7) > 0 ) { //非ascii字符
            if (ord($str[$i]) <= 191 ) {
                $encoding = "gbk0";
                break;
            } else if ( ord($str[$i]) <= 223 ) { //前两位为11
                if ( empty($str[$i+1]) or  ord($str[$i+1]) >> 6 != 2 ) { //紧跟后两位为10
                    $encoding = "gbk1";
                    break;
                } else {
                    $i += 1;
                }
            } else if ( ord($str[$i]) <= 239 ) { //前三位为111
                if ( empty($str[$i+1]) or  ord($str[$i+1]) >> 6 != 2 or empty($str[$i+2]) or  ord($str[$i+2]) >> 6 != 2) { //紧跟后两位为10
                    $encoding = "gbk2";
                    break;
                } else {
                    $i += 2;
                    $is_utf8_chinese = true;
                }
            } else if ( ord($str[$i]) <= 247 ) { //前四位为1111
                if ( empty($str[$i+1]) or  ord($str[$i+1]) >> 6 != 2 or empty($str[$i+2]) or  ord($str[$i+2]) >> 6 != 2 or empty($str[$i+3]) or  ord($str[$i+3]) >> 6 != 2) { //紧跟后两位为10
                    $encoding = "gbk3";
                    break;
                } else {
                    $i += 3;
                }
            } else if ( ord($str[$i]) <= 251 ) { //前五位为11111
                if ( empty($str[$i+1]) or  ord($str[$i+1]) >> 6 != 2 or empty($str[$i+2]) or  ord($str[$i+2]) >> 6 != 2 or empty($str[$i+3]) or  ord($str[$i+3]) >> 6 != 2 or empty($str[$i+4]) or  ord($str[$i+4]) >> 6 != 2) { //紧跟后两位为10
                    $encoding = "gbk4";
                    break;
                } else {
                    $i += 4;
                }
            } else if ( ord($str[$i]) <= 253 ) { //前六位为111111
                if ( empty($str[$i+1]) or  ord($str[$i+1]) >> 6 != 2 or empty($str[$i+2]) or  ord($str[$i+2]) >> 6 != 2 or empty($str[$i+3]) or  ord($str[$i+3]) >> 6 != 2 or empty($str[$i+4]) or  ord($str[$i+4]) >> 6 != 2 or empty($str[$i+5]) or  ord($str[$i+5]) >> 6 != 2 ) { //紧跟后两位为10
                    $encoding = "gbk5";
                    break;
                } else {
                    $i += 5;
                }
            } else {
                $encoding = "gbk6";
                break;
            }
        }
    }
 
    if ($is_utf8_chinese == false){
        $encoding = "gbk10";
    }
    if ($encoding == "utf8" && preg_match("/^[".chr(0xa1)."-".chr(0xff)."\x20-\x7f]+$/", $str) && !preg_match("/^[\x{4e00}-\x{9fa5}\x20-\x7f]+$/u", $str)) {
        $encoding = "gbk7";
    }
    //echo $encoding;
    if ($encoding == "utf8") {
        //echo "utf8";
        return ($str == "鏈條" || $str == "瑷媄")? $str: mb_convert_encoding($str, "gbk", "utf8");
    } else {
        //echo "gbk";
        return $str;
    }
}
```

记得代码最好在gbk编码中运行，因为代码中有汉字（$str == “鏈條” || $str == “瑷媄”）比较。当然你可以在utf8编码中运行，只要把汉字处理下。