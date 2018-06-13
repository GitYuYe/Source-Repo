---
title: php中如何设置mysql查询读取数据的超时时间
date: 2016-07-26 14:00:00
categories: PHP
tags: PHP
---
现象：
php能通过代理正常连接到mysql。但是，执行query后，一直等待，没有任何数据返回。
结果导致php-fpm进程全部阻塞在读取数据的地方。不能处理其他正常请求。

解决方法：
可以通过设置mysql查杀的超时时间来解决这个问题。
第一种设置mysql查询超时时间的方法是使用mysqlnd。
关于msyqlnd的介绍，大家可以看下这篇文章 [http://www.bo56.com/php-mysqlnd-简介/](http://www.bo56.com/php-mysqlnd-%E7%AE%80%E4%BB%8B/)
php启用mysqlnd扩展后，只要在php.ini文件中设置 mysqlnd.net_read_timeout 即可。
参数值的单位为秒。如：
mysqlnd.net_read_timeout = 3 表示每次mysql查询超时时间为3秒。如果超时，则会报错。
如下面的代码：

```
<?php
$dsn = 'mysql:dbname=demo;host=127.0.0.1;port=3306';
$user = 'demo';
$password = 'demo';
$dbh = new PDO($dsn, $user, $password);
$dbh->query("set names utf8");
$sql = "select sleep(5)";
$sth = $dbh->query($sql);
$row = $sth->fetch();
echo "over";
?>
```

则会报错误：
PHP Warning: PDO::query(): MySQL server has gone away
PHP Warning: PDO::query(): Error reading result set’s header
PHP Fatal error: Call to a member function fetch() on a non-object
由于出现了PHP Fatal error错误，导致fetch()之后的代码将无法执行。
因此代码需要对query的返回值做下判断，修改后的代码如下：

```
<?php
$dsn = 'mysql:dbname=demo;host=127.0.0.1;port=3306';
$user = 'demo';
$password = 'demo';
$dbh = new PDO($dsn, $user, $password);
$dbh->query("set names utf8");
$sql = "select sleep(5)";
$sth = $dbh->query($sql);
if(is_object($sth)){
    $row = $sth->fetch();
}
echo "over";
?>
```

注意：设置项 mysqlnd.net_read_timeout 的级别是PHP_INI_SYSTEM。所以在php代码中不能修改mysql查询的超时时间。

另一种方式是使用mysqli。
如果php没有启用mysqlnd，那么可以使用mysqli进行限制read的超时时间。
示例代码如下：

```
<?php
//自己定义读写超时常量
if (!defined('MYSQL_OPT_READ_TIMEOUT')) {
        define('MYSQL_OPT_READ_TIMEOUT',  11);
}
if (!defined('MYSQL_OPT_WRITE_TIMEOUT')) {
        define('MYSQL_OPT_WRITE_TIMEOUT', 12);
}
//设置超时
$mysqli = mysqli_init();
$mysqli->options(MYSQL_OPT_READ_TIMEOUT, 3);
$mysqli->options(MYSQL_OPT_WRITE_TIMEOUT, 1);
//连接数据库
$mysqli->real_connect("localhost", "root", "root", "test");
if (mysqli_connect_errno()) {
   printf("Connect failed: %s/n", mysqli_connect_error());
   exit();
}
//执行查询 sleep 1秒不超时
printf("Host information: %s/n", $mysqli->host_info);
if (!($res=$mysqli->query('select sleep(1)'))) {
    echo "query1 error: ". $mysqli->error ."/n";
} else {
    echo "Query1: query success/n";
}
//执行查询 sleep 9秒会超时
if (!($res=$mysqli->query('select sleep(9)'))) {
    echo "query2 error: ". $mysqli->error ."/n";
} else {
    echo "Query2: query success/n";
}
$mysqli->close();
echo "close mysql connection/n";
?>
```

注意：

1. 超时设置单位为秒，最少配置1秒
2. 但mysql底层的read会重试两次，所以实际会是 3 秒

重试两次 + 自身一次 = 3倍超时时间。
那么就是说最少超时时间是3秒，不会低于这个值，对于大部分应用来说可以接受，但是对于小部分应用需要优化。