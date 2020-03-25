上文SYSTEM_LAG提到duplicate inserts。明明你的php script只是执行一次insert, 结果table里却多过一条records, records的datetime column显示同一秒，也有多几秒可是少见。这造成公司蒙受严重损失。

这里将概括 
a) 原因 
b) 解决方法#1 - 单一process处理事务 
c) 解决方法#2 - 使用explicit lock at table-level
d) 解决方法#3 - 使用INSERT …  NOT EXIST
e) 如何避免duplicate of multi insert? 
f) 注意事项

a) 原因
不明，网上也是各有各说，那好我们整理出整条流程, browser -> cdn proxy (cloudflare) -> web server -> php -> mysql. 我们看到这5个方面皆有可能是肇事者。但最大可能性还是mysql本身。

我曾经使用apache benchmark 做stress test, 将concurrent connection 调成1000然后并发到我写好的php script，蓄意弄lag system 可惜无法还原同样情况。

b) 解决方法#1 - 单一process处理事务 
程序以Multi-tier的方法设计,Frontend只做validation 和接收 request. Cron job (Backend)才做真正的处理。    

不变的真理，使用unique column 或 unique combination columns让mysql底层处理duplicate事件. 由于我们的系统都是member base的，所以善用user_id是关键。代码行过后我才解释这方法。



