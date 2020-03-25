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
* [System Lag](https://github.com/madxradicle/articles/tree/master/system_lag.md)
* [Duplicate Insert](https://github.com/madxradicle/articles/tree/master/duplicate_insert.md)
* [Performance - MYSQL Index](https://github.com/madxradicle/articles/tree/master/performance_mysql_index.md)

<p align="center">
    <img src="https://github.madxradicle.com/duplicate_insert/figure1.png"/><br/>
   图1. Duplicate insert的结果
</p>    
上文SYSTEM_LAG提到duplicate inserts。明明你的php script只是执行一次insert, 结果table里却多过一条records, records的datetime column显示同一秒，也有多几秒可是少见。这造成公司蒙受严重损失。

* [原因](#原因)
* [解决方法1 - 单一process处理事务](#解决方法1-单一process处理事务)
* [解决方法2 - 使用explicit lock at table-level](#解决方法2-使用explicit lock at table-level)
* [解决方法3 - 使用INSERT NOT EXIST](#解决方法3-使用INSERT NOT EXIST)
* [如何避免duplicate of multi insert?](#如何避免duplicate of multi insert?)
* [注意事项](#注意事项)


