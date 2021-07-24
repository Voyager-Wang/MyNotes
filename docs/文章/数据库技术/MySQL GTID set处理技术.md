## 写在前面：MySQL GTID 和 MySQL GTID set

*“GTID是什么”*不是本文要回答的问题，如果之前对MySQL GTID了解不多，可以先参考下官方文档

https://dev.mysql.com/doc/refman/5.7/en/replication-gtids-concepts.html

特别提到一点是，日常使用中经常将MySQL GTID和MySQL GTID set混用，而实际上官方文档中对两者的概念做了很明显的区分：GTID正如它的名字所描述的Global Transaction Identifier，代表的是一个事务；而GTID set是GTID的集合，常见的gtid_executed、gtid_purged实际上都是GTID set，当然，出于简便的原因，把它们都叫做GTID是无可厚非的，但是理解到GTID set本质上是一个集合，对于它处理会更加容易理解。

## 问题是什么？

**问题的背景**

从MySQL数据库中获取binlog日志时，常见的方案是模拟MySQL的主从复制协议，将自己的程序伪装成从库（slave），向MySQL数据库（master）发送dump请求，这样就可以获取到MySQL的binlog（这个过程有多种成熟的工具可以选择，如canal、tungsten等）。

从库向主库发送dump请求时，需要告诉主库要发送哪些binlog过来，如果主库开启了GTID模式，从库可以通过发送一组GTID set，告诉主库因该发送哪些binlog，这组GTID set的含义是*“我已经有这些事务了，请把**其他所有的**发给我”*。如果发送了不合适的GTID set，往往会引起复制出错，最常见出错的场景是请求了主库已经删除的binlog。基于以上，本文只回答一个问题

**问题**

如何设置恰当的GTID set，建立或者恢复主从复制？
