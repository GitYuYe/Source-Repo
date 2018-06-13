---
title: php中mysql数据库异步查询实现
date: 2016-09-24 16:43:00
categories: PHP
tags: PHP
---
##### 问题

通常一个web应用的性能瓶颈在数据库。因为，通常情况下php中mysql查询是串行的。也就是说，如果指定两条sql语句时，第二条sql语句会等到第一条sql语句执行完毕再去执行。这个时候，如果执行2条sql语句，每条执行时间为50ms，全部执行完毕可能需要100ms。既然，主要原因是sql的串行执行导致。那我们是不是可以改变执行方式来提高性能呢？答案是，可以的。我们可以通过异步执行的方式来提高性能。

##### 异步

如果通过异步的方式去执行，可能性能会有很大提升。如果是采用异步的方式，两条sql语句会并发执行，可能就需要60ms就可以执行完毕。

##### 实现

mysqli + mysqlnd。php官方实现的mysqlnd中提供了异步查询的方法。分别是：
mysqlnd_async_query 发送查询请求
mysqlnd_reap_async_query 获取查询结果
这样就可以不必每次发送完查询请求后，一直阻塞等待查询结果了。

实现代码如下：

```
<?php
  
$host       = '127.0.0.1';
$user       = 'root';
$password   = '';
$database   = 'test';
  
/**
 * 期望得到额结果
 * array(
 *  1 => int,
 *  2 => int,
 *  3 => int
 * )
 */
$result = array(1=>0, 2=>0, 3=>0);
  
//异步方式[并发请求]
$time_start = microtime(true);
$links = array();
  
foreach ($result as $key=>$value) {
    $obj = new mysqli($host, $user, $password, $database);
    $links[spl_object_hash($obj)] = array('value'=>$key, 'link'=>$obj);
}
$done = 0;
$total = count($links);
  
foreach ($links as $value) {
    $value['link']->query("SELECT COUNT(*) AS `total` FROM `demo` WHERE `value`={$value['value']}", MYSQLI_ASYNC);
}
  
do {
  
    $tmp = array();
    foreach ($links as $value) {
        $tmp[] = $value['link'];
    }
  
    $read = $errors = $reject = $tmp;
    $re = mysqli_poll($read, $errors, $reject, 1);
    if (false === $re) {
        die('mysqli_poll failed');
    } elseif ($re < 1) {
        continue;
    }
  
    foreach ($read as $link) {
        $sql_result = $link->reap_async_query();
        if (is_object($sql_result)) {
            $sql_result_array = $sql_result->fetch_array(MYSQLI_ASSOC);//只有一行
            $sql_result->free();
            $hash = spl_object_hash($link);
            $key_in_result = $links[$hash]['value'];
            $result[$key_in_result] = $sql_result_array['total'];
        } else {
            echo $link->error, "\n";
        }
        $done++;
    }
  
    foreach ($errors as $link) {
        echo $link->error, "1\n";
        $done++;
    }
  
    foreach ($reject as $link) {
        printf("server is busy, client was rejected.\n", $link->connect_error, $link->error);
        //这个地方别再$done++了。
    }
} while ($done<$total);
var_dump($result);
echo "ASYNC_QUERY_TIME:", microtime(true)-$time_start, "\n";
  
$link = end($links);
$link = $link['link'];
echo "\n";
```

##### 结语

mysql数据库对于每个查询请求都是单独启动一个线程进行处理。如果mysql服务器启动线程过多，必然会造成线程切换引起系统负载过高。如果在mysql数据库负载不高的情况下，使用异步查询还是不错的选择。