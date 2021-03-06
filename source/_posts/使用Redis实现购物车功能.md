---
title: 使用Redis实现购物车功能
date: 2017-06-13 18:05:00
categories: PHP
tags: PHP
---
#### 使用Redis实现购物车功能

一般情况下购物车功能都是使用SESSION/COOKIE实现的，也就是将整个购物车数据都存储到SESSION中。这样做的好处就是不用操作数据库就可以实现，同时用户可以不同登录就可以将商品加入到购物车中，缺点就是1. 导致SESSION过于臃肿 2. SESSION数据默认是存储到文件中的，所以操作SESSION是相对比较慢的。而将购物车数据存放到Redis中，可以加快购物车的读写性能，从而提高用户体验，缺点就是Redis数据是存放到内存，相对成本高

### 核心代码如下

Cart.class.php代码:

```
 <?php

//使用redis实现一个购物车功能
class Cart
{

    /**
     *  购物车有功能： 1、 将商品添加到购物车中  2、改变购物车商品数量  3、显示购物车的信息
     *
     *
     * 将商品添加到购物车中功能分析如下：
     * 1. 接收到商品ID
     * 2. 根据商品ID查询商品信息
     * 3. 将商品信息加入到购物车中
     *
     *         a. 判断购物车是否已有对应商品
     *         b. 如果购物车中没有对应的商品，直接加入
     *         c. 如果购物车中有对应的商品，只要修改商品数量
     */

    public function __construct()
    {
        //如果成员属性没有声明，默认就是公有属性
        $this->redis = new Redis;
        $this->redis->connect('127.0.0.1', 6379);
    }

    public function addToCart($gid, $cartNum=1)
    {

        session_start();
        if ($gid <= 0) {

            throw new Exception("请输入商品ID");

        }

        //根据商品ID查询商品数据
        $goodData = $this->goodsData($gid);

        $key = 'cart:'.session_id().':'.$gid;//id 说明：1、不仅仅要区分商品  2、 用户

        // $data = $this->redis->hget($key, 'id');
        $data = $this->redis->exists($key);


        //判断购物车中是否有无商品，然后根据情况加入购物车
        if (!$data) {

            //购物车之前没有对应的商品的

            //购物车的商品数量
            $goodData['num'] = $cartNum;

            //将商品数据存放到redis中hash
            $this-&gt;redis-&gt;hmset($key, $goodData);


            $key1 = 'cart:ids:set:'.session_id();

            //将商品ID存放集合中,是为了更好将用户的购物车的商品给遍历出来
            $this-&gt;redis-&gt;sadd($key1, $gid);

        } else {

            //购物车有对应的商品，只需要添加对应商品的数量
            $originNum = $this->redis->hget($key, 'num');

            //原来的数量加上用户新加入的数量
            $newNum = $originNum + $cartNum;

            $this-&gt;redis-&gt;hset($key, 'num', $newNum);


        }

    }

    //显示用户购物车的所有商品
    public function showCartList()
    {

        session_start();

        $sessId = session_id();


        $key = 'cart:ids:set:'.session_id();

        //先根据集合拿到商品ID
        $idArr =  $this->redis->sMembers($key);


        for ($i=0; $i<count($idArr); $i++) {

            $k  = 'cart:'.session_id().':'.$idArr[$i];//id 

            // echo $k,'<br/>';
            $list[] = $this->redis->hGetAll($k);
        }

        include './View/show.php';

    }

    public function goodsData($gid)
    {

        $goodsData = array(

            1 => array(
                'id' => 1,
                'gname' => 'xxoo',
                'price' => '1.5'
            ),

            2 => array(
                'id' => 2,
                'gname' => 'xxoo22',
                'price' => '221.5'
            ),
            3 => array(
                'id' => 3,
                'gname' => 'xxoo33',
                'price' => '331.5'
            ),
            4 => array(
                'id' => 4,
                'gname' => 'xxoo44',
                'price' => '4441.5'
            ),    
        );

        return $goodsData[$gid];
    }
}
```