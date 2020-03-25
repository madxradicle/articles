# Technical Articles
<p align="center">
  <a href="https://github.com/madxradicle/madxframework2.0">
    <img src="https://www.randomsystem.net/media/images/github/MR_logo.png" alt="Logo" width="200px" height="100px">
  </a>
  <h3 align="center">Technical Articles</h3>
  <p align="center">
   These articles are initially written in chinese and they're moved from google documents.
    <br />
    <a href="https://github.com/madxradicle/articles"><strong>Explore the docs »</strong></a>
    <br />
    <a href="https://github.com/madxradicle/articles/issues">Report Bug</a>
    ·
    <a href="https://github.com/madxradicle/articles/issues">Request Feature</a>
  </p>
</p>

<!-- TABLE OF CONTENTS -->
## Menu
* [System Lag](https://github.com/madxradicle/articles/tree/master/system_lag.md)
* [Duplicate Insert](https://github.com/madxradicle/articles/tree/master/duplicate_insert.md)
* [Performance - MYSQL Index](https://github.com/madxradicle/articles/tree/master/performance_mysql_index.md)

## Table of Contents
* [原因](#原因)
* [解决方法1 - 单一process处理事务](#解决方法1-单一process处理事务)
* [解决方法2 - 使用explicit lock at table-level](#解决方法2-使用explicit-lock-at-table-level)
* [解决方法3 - 使用INSERT NOT EXIST](#解决方法3-使用INSERT-NOT-EXIST)
* [如何避免duplicate of multi insert?](#如何避免duplicate-of-multi-insert?)
* [注意事项](#注意事项)

## 原因
上文SYSTEM_LAG提到duplicate inserts。明明你的php script只是执行一次insert, 结果table里却多过一条records, records的datetime column显示同一秒，也有多几秒可是少见。这造成公司蒙受严重损失。

<p align="center">
    <img src="https://github.madxradicle.com/duplicate_insert/figure1.png"/><br/>
   图1. Duplicate insert的结果
</p>   

不明，网上也是各有各说，那好我们整理出整条流程, browser -> cdn proxy (cloudflare) -> web server -> php -> mysql. 我们看到这5个方面皆有可能是肇事者。但最大可能性还是mysql本身。

我曾经使用apache benchmark 做stress test, 将concurrent connection 调成1000然后并发到我写好的php script，蓄意弄lag system 可惜无法还原同样情况。

## 解决方法1 单一process处理事务
程序以Multi-tier的方法设计,Frontend只做validation 和接收 request. Cron job (Backend)才做真正的处理。    

不变的真理，使用`unique` column 或 `unique` combination columns让mysql底层处理duplicate事件. 由于我们的系统都是member base的，所以善用user_id是关键。代码行过后我才解释这方法。

<p align="center">
    <img src="https://github.madxradicle.com/duplicate_insert/figure2.png"/><br/>
    图2. mg_prevent_duplicate表结构
</p>    

PHP伪代码
```sh
member/test.php
<?php
require_once("inc/inc.member.php");

//…………some codings………….
//…………some codings………….
//…………some codings………….

If ($_POST[‘a form has been submitted’] == true) {
    //..…… some validation codings ………
    //..…… some validation codings ………
    //..…… some validation codings ………

    If (!$err_msg) {
    if(!$o_db->num_rows( $o_db->query("SELECT pd_id FROM mg_prevent_duplicate WHERE user_id = '”.escape($user_id).”' AND pd_activity_type=’add_stock’") ) ) {
        /* lock to prevent duplicate insert */
        $r = $o_db->query("INSERT INTO mg_prevent_duplicate (user_id,pd_activity_type,pd_cdate) VALUES ( ‘“.escape('$user_id').”’,’add_stock’,’”.date("Y-m-d H:i:s").”’ )");
    );
    }
}

cron_folder/cron_process_request.php 每分钟执行一次。
<?php
require_once("inc/inc.cron.php");

//…………some codings………….
//…………some codings………….
//…………some codings………….

If (!$err_msg) {
    //purpose of run it full within 1 minute time
$expireTime = time() + 57;

while( time() < $expireTime ) {
$r = $o_db->query("SELECT * FROM mg_prevent_duplicate LIMIT 1");
        while($row = $o_db->fetch_assoc($r)) {
            $id = $row[‘pd_id’];    
            //……...your actual work ………
            //……...your actual work ………
            //……...your actual work ………
        }

//unlock
        $o_db->query("DELETE FROM mg_prevent_duplicate WHERE pd_id = ‘{$id}’");
    }
}
```
首先cron_process_request.php是由单一process 处理，而member/test.php如果用户死命乱按那么requests可能由多个 process处理就更容易发生duplicate insert。这么看来单一process是最安全的方法，因为目前我们没发现duplicate insert的事件发生。

优点: 
1) unique使用到implicit lock,mysql底层自动处理锁，性能一定好过explicit lock。
2) 由于frontend只做validation和接收request, php执行速度快促使connection 销毁速度也快。

缺点:
1) 由于是background/asynchronous process,用户无法即时体验request结果。补救方法是使用websocket将request结果直接打在客户端。

## 解决方法2 使用explicit lock at table level
```sh
PHP伪代码
<?php

//only session hold write table lock can read and write at this table
//other session cannot read and write until write table lock is released
$o_db->query("LOCK TABLE tableA,tableB,tableC WRITE");

$o_db->query("SELECT tableAcolumn FROM tableA WHERE user_id=123");

$o_db->query("INSERT INTO tableB SET tableBcolumn='programmer');

$o_db->query("INSERT INTO tableC SET tableCcolumn='soonyu');

$o_db->query("UNLOCK TABLES");
```
优点: 可以即时处理事务然后将结果打印给用户

缺点: 抢锁的过程将造成系统负担大，高并发时，这种设计系统容易被KO。不建议使用。

## 解决方法#3 使用INSERT NOT EXIST
如果不做Unique column,使用这种sql也可以解决duplicate insert.
```sh
$sql = "INSERT INTO `sometable` (some_type, some_id) SELECT 'add_product','{$get_d['id']}', FROM `sometable` WHERE NOT EXISTS (SELECT 1 FROM `sometable` where some_type='add_product' and some_id='{$get_d['id']}') LIMIT 1";
```
使用这种sql的前提是必须在sometable里出现一条dummy record。

优点: 可以即时处理事务然后将结果打印给用户。这种方法也比explicit lock好。

缺点: insert sql的可读性差，一个insert需要两次select的开销。

## 如何避免duplicate of multi insert?
<p align="center">
    <img src="https://github.madxradicle.com/duplicate_insert/figure3.png"/><br/>
    图3. Structure of multi insert sql
</p>

配合UNION和INSERT NOT EXIST, 可以用一行SQL解决这问题。
```sh
INSERT INTO mg_member_transaction (mt_type, user_id, ref_table, ref_pkid, mt_wallet, mt_add, mt_deduct, mt_cdate)
(SELECT 'transfer', '1', 'sometable', '1', 'cash', 0 , 12, '2020-03-06 12:00:00' FROM mg_member_transaction WHERE NOT EXISTS (SELECT 1 FROM `mg_member_transaction` where ref_table='sometable' and ref_pkid='1') LIMIT 1)
UNION ALL 
(SELECT 'transfer', '2', 'sometable', '2', 'cash', 12, 0, '2020-03-06 12:00:00' FROM mg_member_transaction WHERE NOT EXISTS (SELECT 1 FROM `mg_member_transaction` where ref_table='sometable' and ref_pkid='2') LIMIT 1)
```

## 注意事项
Unique是强大的，可是如果script是执行多个insert在不同的table,那么这些table通通都必须有unique的保护。举例错误示范。

PHP伪代码
```sh
<?php
//tableBcolumn is unique
$r = $o_db->query("INSERT INTO tableB SET tableBcolumn='programmer');

If ($o_db->affected_rows($r) > 0) {
    //tableCcolumn is not unique
    $o_db->query("INSERT INTO tableC SET tableCcolumn='soonyu');
}
```
这种情况当系统繁忙时，tableC还是会有几率发生duplicate inserts的! 
