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
* [简介](#简介)
* [使用EXPLAIN](#使用EXPLAIN)
* [WHERE CLAUSE错误写法](#WHERE-CLAUSE错误写法)
* [ORDER BY较差写法 ](#ORDER-BY较差写法)
* [SELECT 最好写法](#SELECT-最好写法)
* [其他](#其他)

## 简介
<p align="center">
    <img src="https://github.madxradicle.com/mysql_index/figure1.png"/><br/>
    图1. 简单的phonebook table structure，我随机添加了10mil行记录。
</p>    

<table>
  <tr><th>未加INDEX</th><th>已加INDEX</th><th>备注</th></tr>
  <tr>
    <td>SELECT * FROM `phonebook` WHERE `surname` = 'tan'<br/>(0.0019s)</td>
    <td>SELECT * FROM `phonebook` WHERE `surname` = 'tan'<br/>(0.0019s)</td>
    <td>呃，show off失败，似乎没有什么改变（祈祷）。</td>
  </tr>
  <tr><td>SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee'<br/>(1.7565s)</td>
    <td>
SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee'<br/>(0.0006s)
</td><td>看到了吧？速度明显提升了。</td></tr>
  <tr><td>SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee' and phone_number='4568061964'<br/>(6.3696s)</td>
    <td>SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee' and phone_number='4568061964'<br/>(0.0011s)</td>
    <td>看到了吧！！速度大幅度提升了！！</td></tr>
</table>

好，这里讲到设计index,首先重复data最少的column排在第一位,重复最多的排在最后一位。请参考图2。
<p align="center">
    <img src="https://github.madxradicle.com/mysql_index/figure2_1.png"/><img src="https://github.madxradicle.com/mysql_index/figure2_2.png"/><br/>
    图2. 用phpmyadmin新建一个combination index。别为各个column加index,没效果。
</p>    

## 使用EXPLAIN
较差的SQL设计，EXPLAIN里会看到。
<p align="center">
    <img src="https://github.madxradicle.com/mysql_index/figure3.png"/><br/>
    图3. 用phpmyadmin query explain
</p>    

1) type：ALL， 扫描整个table。
2) key: NULL, 没有用到index。
3) 估计扫描记录 9398158,占了超过总记录的90%。
4) Using filesort,此order by没有用到index。

较好的SQL设计，EXPLAIN里会看到。
<p align="center">
    <img src="https://github.madxradicle.com/mysql_index/figure4.png"/><br/>
    图4. 用phpmyadmin query explain
</p> 

1) type: ref, 此sql使用到index。
2) key: surname_givenname_phoneno，使用到这index。
3) 仅仅扫描了104个记录。
4) Using Index, 此order by用到index。

## WHERE CLAUSE错误写法
如果添加Index的column包含依序a,b,c, 那么写WHERE CLAUSE的最佳的效果便是：
```sh
WHERE a=? AND b=? AND c=? 或 ✅
WHERE a=? AND b=? 或 ✅
WHERE a=? ✅
```

请避免以下写法：
```sh
WHERE a=? AND c=? 或 ❌
WHERE b=? AND c=? 或 ❌
WHERE b=? 或 ❌
WHERE c=? ❌
```

## ORDER BY较差写法 
这里判断较差在于EXPLAIN的EXTRA出现了filesort,简单来说filesort代表index用不到。除此之外，前提还要 SELECT a,b,c或SELECT a,b 或 SELECT a ,不可使用SELECT *

```sh
IF WHERE clause is empty THEN
-------------------------------
ORDER BY a ASC, b ASC, c ASC ✅
ORDER BY a ASC, b ASC✅
ORDER BY a ASC✅

ORDER BY a ASC, c ASC❌
ORDER BY b ASC, c ASC❌
ORDER BY b ASC ❌
ORDER BY c ASC ❌
```

```sh
IF WHERE a=? THEN`
-------------------------------
ORDER BY a ASC, b ASC, c ASC ✅
ORDER BY a ASC, b ASC ✅
ORDER BY a ASC✅
ORDER BY b ASC ✅
ORDER BY b ASC, c ASC ✅

ORDER BY c ASC ❌
ORDER BY a ASC, c ASC ❌
```

```sh
IF WHERE a=? AND b=? THEN
-------------------------------
ORDER BY a ASC, b ASC, c ASC ✅
ORDER BY a ASC, b ASC ✅
ORDER BY a ASC✅
ORDER BY b ASC ✅
ORDER BY c ASC✅
ORDER BY a ASC, c ASC✅ 
ORDER BY b ASC, c ASC ✅
```

ASC (ascending) 是natural order, 不需要额外工作所以最快，一旦用到DESC,不管column有没有被Index,explain里都会显示using filesort, 这时mysql需要额外开销。

是不是说永远都用不到DESC? 其实只要用WHERE 缩小search range，还是可以用到DESC的。所以programmer必须一直用explain检查并调到最佳水平。

## SELECT 最好写法
如果添加Index的column包含依序a,b,c, 那么写SELECT的最佳的效果便是：

```sh
SELECT a,b,c ✅
SELECT a,b ✅
SELECT a  ✅
SELECT b,c ✅
SELECT a,c ✅
SELECT b ✅
SELECT c ✅

SELECT * ❌
```

当你使用index(a_b_c) 找到record时,直接取出indexed columns: a,b,c就变得轻易许多。如果select 的column不在index里，那么mysql需要额外开销到table file再取出column value.

## 其他
本文所提到的都是index最佳的使用方法。所以以下的commands都必须避免。

```SH
OR ❌
>=, <=, >, <❌
NOT IN❌
BETWEEN❌
LIKE❌
!=❌
```
Index是针对select sql而设的,需知大部分的系统都是以select占大多数的。

更多内容，可以从这里学习 https://www.slideshare.net/billkarwin/how-to-design-indexes-really


