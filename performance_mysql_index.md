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
  <tr><td>SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee' (1.7565 seconds)</td>
    <td>
SELECT * FROM `phonebook` WHERE `surname` = 'tan' and given_name='wen wee' (0.0006 seconds)
</td><td>看到了吧？速度明显提升了。</td></tr>
  <tr><td></td><td></td><td></td></tr>
  
  
  
</table>
