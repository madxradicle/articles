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
    <td>SELECT * FROM `phonebook` WHERE `surname` = 'tan' (0.0019 seconds)</td>
    <td>SELECT * FROM `phonebook` WHERE `surname` = 'tan' (0.0019 seconds)</td>
    <td>呃，show off失败，似乎没有什么改变（祈祷）。</td>
  </tr>
  <tr><td>SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee' (1.7565s)</td>
    <td>
SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee' (0.0006s)
</td><td>看到了吧？速度明显提升了。</td></tr>
  <tr><td>SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee' and phone_number='4568061964' (6.3696s)</td>
    <td>SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee' and phone_number='4568061964' (0.0011s)</td>
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

##WHERE CLAUSE错误写法

