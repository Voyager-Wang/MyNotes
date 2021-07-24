## Flink SQL CDC简介

Flink 1.11 引入了 Flink SQL CDC，基于社区的开源组件 flink-cdc-connectors 实现，这是一个可以直接从 MySQL、PostgreSQL 等数据库直接读取全量数据和增量变更数据的 source 组件。

### CDC定义：

#### 概念

CDC 全称是 Change Data Capture ，它是一个比较广义的概念，只要能捕获变更的数据，我们都可以称为 CDC。

In [databases](https://en.wikipedia.org/wiki/Database), **change data capture** (**CDC**) is a set of software [design patterns](https://en.wikipedia.org/wiki/Design_pattern_(computer_science)) used to determine and track the data that has changed so that action can be taken using the changed data.

#### 基于日志的CDC

Flink CDC 是基于增量日志实现的，优势在于：

- 能够捕获所有数据的变化，捕获完整的变更记录。在异地容灾，数据备份等场景中得到广泛应用，如果是基于查询的 CDC 有可能导致两次查询的中间一部分数据丢失
- 每次 DML 操作均有记录无需像查询 CDC 这样发起全表扫描进行过滤，拥有更高的效率和性能，具有低延迟，不增加数据库负载的优势
- 无需入侵业务，业务解耦，无需更改业务模型
- 捕获删除事件和捕获旧记录的状态，在查询 CDC 中，周期的查询无法感知中间数据是否删除

### flink-cdc-connectors



![image-20210112213013283](https://tinkerer-pic.oss-cn-hangzhou.aliyuncs.com/picgo/image-20210112213013283.png)

如上图，借助flink-cdc-connectors可以实现 Flink SQL 采集+计算+传输（ETL）一体化，这样做的优点有以下：
- 开箱即用，简单易上手
- 减少维护的组件，简化实时链路，减轻部署成本
- 减小端到端延迟
- Flink 自身支持 Exactly Once 的读取和计算
- 数据不落地，减少存储成本
- 支持全量和增量流式读取
- binlog 采集位点可回溯*

### 功能特点

flink-cdc-connectors本质上是包装了Debezium工具的一个flink-source，处理利用Debezium获取增量数据的功能，还借助Debezium实现了全量数据的初始化（读快照数据）。

需要关注的特性主要有：

- 实现了全量（快照读）+增量同步，通过一个快速的锁表，保证增量不会重做全量数据
- 支持SQLAPI、TableAPI、StreamAPI
- 不依赖kafka等消息队列中间件
- 每张表读取一份binlog，缺少共享机制，使用大量表的场景容易产生瓶颈
- 位置点信息保存在flink state中，借用flink checkpointing机制保证exactly-once

补充：常见的CDC工具主要有，maxwell、canal、debezium、flinkx，简单比较如下：

| 组件                 | Canal                                                | Maxwell                                             | Debezium                                                     | Flinx                                               |
| -------------------- | ---------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------- |
| 开源方               | 阿里                                                 | zendesk                                             | redhat                                                       | 袋鼠云                                              |
| 开发语言             | Java                                                 | Java                                                | Java                                                         | Java                                                |
| 支持数据库           | MySQL                                                | MySQL                                               | MongoDB、MySQL、PostgreSQL、SQL Server 、Oracle( 孵化)、DB2( 孵化)、Cassandra( 孵化) | MongoDB、MySQL、PostgreSQL                          |
| 是否支持bootstrap    | 否                                                   | 是                                                  | 是                                                           | 否                                                  |
| 是否支持解析DDL同步  | 是                                                   | 是                                                  | 是                                                           | 是                                                  |
| 是否支持HA           | 是                                                   | 需定制                                              | 基于kafka-connector                                          | 是                                                  |
| 社区活跃(2020.07.20) | release:2019.09.02,star:14.8k,last-commit:2020.03.13 | release:2020.07.01,star:2.2k,last-commit:2020.07.02 | release:2020.07.16,star:3.4k,last-commit:2020.07.16          | release:2020.07.14,star:1.4k,last-commit:2020.07.17 |
| 文档                 | 中文，百度可以解决                                   | 英文，官方文档                                      | 英文，官方文档十分详细                                       | 中文，github readme 文档                            |
| MQ集成               | RocketMQ、Kafka                                      | kafka                                               | kafka                                                        | Emqx、kafka                                         |

### 未来方向

- FLIP-132 ：Temporal Table DDL（基于 CDC 的维表关联）
- Upsert 数据输出到 Kafka
- 更多的 CDC formats 支持（debezium-avro, OGG, Maxwell）
- 批模式支持处理 CDC 数据
- flink-cdc-connectors 支持更多数据库

比较值得关注的是基于 CDC 的维表关联：

通过 CDC 把维表的数据导入到维表 Join 的状态里面，在这个 State 里面因为它是一个分布式的 State ，里面保存了 Database 里面实时的数据库维表镜像，当消息队列数据过来时候无需再次查询远程的数据库了，直接查询本地磁盘的 State ，避免了 IO 操作，实现了低延迟、高吞吐，更精准

![image-20210112214523498](https://tinkerer-pic.oss-cn-hangzhou.aliyuncs.com/picgo/image-20210112214523498.png)