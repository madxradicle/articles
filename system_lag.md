通常系统异常变慢，可以说90%都是SQL的问题。这里概括 a) 如何检查，b) 解决方法, c) 原理，d) 并发症

a) 如何检查
- 检查mysql process
    - whm > phpmyadmin > status > processes 或 whm > show mysql processes里检查long execute sql
    - 在process list > status column如果发现 Creating sort index,基本断定这个sql要optimize了
- 检查 mg_trace
    - mg_trace's exec_time是指整个php script所消耗的时间
    - mg_trace's query_exec_time是指总sql的运行时间
    - 如果query_exec_time = exec_time,那一定是SQL性能问题
    - 如果exec_time 不寻常的大过 query_exec_time，这有几个可能
        - 使用了curl触发api, 对方response慢让php script白等
        - 使用了 *_connect() function 链接tcp protocol/service (Such as ftp,email etc)
        - php script里存在 sleep() function
        - 过度调用了heavy php function such as image handling functions


b) 解决方法
- Mysql 解决方法
    - Index好Select SQL (新篇专门介绍index，中文叫索引)
    - Join/union select/sql’s subquery没有设计好，建议拆成几个单条SQL
    - 改用innodb engine
    - setup mysql proxy做读写分离 (master-slave model), 需知99% of sql 是SELECT， 请参考 whm > phpmyadmin > status > query statistics
    - avoid explicit lock, for example sql LOCK sometable 和 UNLOCK sometable.
- PHP 解决方法
    - 不要在php script里做太多东西,否则建议script只接受request,让cron job做background process
    - curl或任何关系到第三方的链接, 可以放到cron job做background process
    - 做 dns load balancer, cloudflare可以实现 (操作难度-简单)
    - 做 server load balancer,公司用过 ha proxy和ibm load balancer (操作难度-较难)

c) 原理
- 不变的真理, Connection(链接）的生成如果大于销毁注定系统就要崩溃。
- PHP说明
    - 多个browser/client request链接到webserver(apache/nginx)的connection由多个process处理，每个process就是一个webserver程序，程序里调用php module执行php代码。
- Mysql说明
    - 什么是creating sort index? index一个table column,mysql就会生成两个files, 1) table file, 2) index file, 如果一个 sql 里使用到的columns都index过那就很快因为mysql只需读取index file，可是如果order by 里的column没有存在于index file,系统将往table file读取，这时你就会看到 creating sort index。同时读两个files,mysql开销变大速度变慢。

d) 并发症
通常系统lag后，mysql将发生 duplicate inserts (新篇专门介绍解决方法)
Mysql’s myisam typed table broken, phpmyadmin > database > structure > size column有时会显示 NOT IN USE, 解决方法执行 repair table
Rsync 发生 broken pipe error
Ftp / cpanel 登入不了
