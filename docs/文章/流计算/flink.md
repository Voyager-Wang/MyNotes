## Module

<img src="/Users/wangtao/notes/docs/文章/image-20200929115413536.png" alt="image-20200929115413536" style="zoom:50%;" />



*要求：AppMaster、JobManager、TaskManager等组件的功能和之间的交互*

当启动一个flink集群时，启动了哪些组件？这些组件的作用是什么

当执行一个任务时，启动了哪些组件？这些组件的作用是什么

![image-20200927162212055](/Users/wangtao/notes/docs/文章/image-20200927162212055.png)



# Stateful Stream Processing

有状态的流计算（Stateful Stream Processing）衍生出了对状态（State）和状态持久化（State Persistence）的需求，Flink状态持久化依赖的机制是Checkpointing。

有状态的计算需求举例：

- When an application searches for certain event patterns, the state will store the sequence of events encountered so far.
- When aggregating events per minute/hour/day, the state holds the pending aggregates.
- When training a machine learning model over a stream of data points, the state holds the current version of the model parameters.
- When historic data needs to be managed, the state allows efficient access to events that occurred in the past.

### State

#### Queryable state

To access state from outside of Flink during runtime.

https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/stream/state/queryable_state.html

#### State backends

Flink provides different state backends that specify **how and where state is stored**.

开箱即用的三种backends：

- *MemoryStateBackend*
- *FsStateBackend*
- *RocksDBStateBackend*

##### 1.The MemoryStateBackend

Upon checkpoints, this state backend will snapshot the state and send it as part of the checkpoint acknowledgement messages to the JobManager, which stores it on its heap as well.

保存在JobManager的内存中，随着ack消息一起发送给JobManager

默认是异步的

JobManager挂了持久化的信息即丢失？

有JobManager的高可用

##### 2.The FsStateBackend

The FsStateBackend holds in-flight data in the TaskManager’s memory. Upon checkpointing, it writes state snapshots into files in the configured file system and directory. Minimal metadata is stored in the JobManager’s memory (or, in high-availability mode, in the metadata checkpoint).

##### 3.The RocksDBStateBackend

The RocksDBStateBackend holds in-flight data in a [RocksDB](http://rocksdb.org/) database that is (per default) stored in the TaskManager data directories. Upon checkpointing, the whole RocksDB database will be checkpointed into the configured file system and directory. Minimal metadata is stored in the JobManager’s memory (or, in high-availability mode, in the metadata checkpoint).

和FsStateBackend的区别在于 in-flight data保存的方式

更多细节：待补充





### Checkpointing机制

#### 概念：Checkpoint

Checkpoint是一种快照（snapshot），另外，savepoint也是一种快照（snapshot），这里的快照（snapshot）指的是所有Operator的State的全局快照（分布式系统的快照）。实际上，checkpoint和savepoint几乎是相同的，仅在以下两个方面不同：

Savepoints are **triggered by the user** and **don’t automatically expire**

- savepoint由用户手动触发生成
- savepoint不会过期自动清理

checkpoint生成机制概览

<img src="/Users/wangtao/notes/docs/文章/image-20201014142102064.png" alt="image-20201014142102064" style="zoom:50%;" />

保存快照是异步的

Keep in mind that everything to do with checkpointing can be done asynchronously. The checkpoint barriers don’t travel in lock step and operations can asynchronously snapshot their state.

checkpoint需要手动设置：

- 定义state：Stream API 有多种方式定义state：windows、key/value state、CheckpointedFunction，其他API怎么使用checkpoint？

- 主动开启checkpoint，默认是不开启checkpoint的

#### Barriers

Barrier是一个特殊的消息，触发所有Operator保存snapshot的标识消息，和其他消息一起按照顺序流经整个系统，由JobManager触发各个source生成。

<img src="/Users/wangtao/notes/docs/文章/image-20200929104440302.png" alt="image-20200929104440302" style="zoom:50%;" />



#### Operator生成快照的方式之一：Aligned checkpoints

![image-20200929113822288](/Users/wangtao/notes/docs/文章/image-20200929113822288.png)



这种Aligned（对齐） checkpoints的机制可以做到Exactly-once，还有其他的checkpoints的方式，这里不展开。

Aligned checkpoints的特点是有多个输入流的Operator要等到所有的Barrier到齐后，再向下游发送Barrier，如果一个流的Barrier先到了，这个流的Barrier之后的所有消息都会被阻塞，直到所有Barrier都到齐并向下游发送了Barrier之后才结束阻塞状态。

保存snapshot的时机

Operators snapshot their state at the point in time when they have received all snapshot barriers from their input streams, and before emitting the barriers to their output streams. 

也可以关闭上述阻塞机制，但是保存snapshot的时机是一样的，这时提供的是At-least-once保证，带来的好处更低的延迟。

![image-20201014141630078](/Users/wangtao/notes/docs/文章/image-20201014141630078.png)

图中Checkpoint data保存在：

in the JobManager’s memory (or, in high-availability mode, in the metadata checkpoint)



#### Operator生成快照的方式之二：Unaligned checkpoints（Flink 1.11引入）

![image-20201014153141208](/Users/wangtao/notes/docs/文章/image-20201014153141208.png)

#### Recovery（恢复）

Recovery under this mechanism is straightforward: Upon a failure, Flink selects the latest completed checkpoint *k*. The system then re-deploys the entire distributed dataflow, and gives each operator the state that was snapshotted as part of checkpoint *k*. The sources are set to start reading the stream from position *Sk*. For example in Apache Kafka, that means telling the consumer to start fetching from offset *Sk*.

具体有不同的Restart Strategies

https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/task_failure_recovery.html#restart-strategies

#### 如何做到Exactly-once的？

三种常见的场景

- Flink makes no effort to recover from failures (*at most once*)
- Nothing is lost, but you may experience duplicated results (*at least once*)
- Nothing is lost or duplicated (*exactly once*)

**Exactly-once的含义：**

Given that Flink recovers from faults by rewinding and replaying the source data streams, when the ideal situation is described as **exactly once** this does *not* mean that every event will be processed exactly once. Instead, it means that *every event will affect the state being managed by Flink exactly once*.

Exactly-once并不意味着在失败和恢复的过程中，消息只会被处理一次，更确切的说，消息的处理是遵循at least once的；所谓的exactly-once是指消息对“状态”的作用效果是严格的只执行一次，比如对多个消息进行求和，求和的结果就是一种“状态”，exactly-once保证的是在恢复之后，最终求和的结果一定是正确的，即最终效果是一个消息既不会丢失、也不会重复统计到结果中（exactly-once），但是，对这条消息对处理可能会重复执行。

**Exactly-once 和 Exactly Once End-to-end有区别：**

To achieve exactly once end-to-end, so that every event from the sources affects the sinks exactly once, the following must be true:

1. your sources must be replayable, and（源端可以重放数据，保证Exactly-once的要求）
2. your sinks must be transactional (or idempotent)（目标端支持事务或者是幂等的，保证Exactly Once End-to-end的要求）

从本质上来理解，exactly-once的作用范围限制在所有Operator中，snapshot也只是记录了所有Operator的状态快照，不会也不能对外部系统做快照，外部系统指的是sink写入的目标端系统；而Exactly Once End-to-end（端到端的exactly-once），包含了外部系统在内，其涉及的范围是超过exactly-once的作用范围的，因此单纯的exactly-once不可能保证Exactly Once End-to-end。要想实现Exactly Once End-to-end，需要在exactly-once的基础上，对外部系统额外作出一些限制，这个限制是：

- 目标端支持事务或者是幂等的

这里要对目标端支持事务做单独的说明，对一个支持事务（不是幂等的）的目标端，要想做到Exactly Once End-to-end，必须要严格地每生成一个checkpoint做一次commit，这会带来不可避免的延迟，这个延迟取决于checkpoint的生成间隔。

（commit失败如何处理？）

![image-20201014143636996](/Users/wangtao/notes/docs/文章/image-20201014143636996.png)

![image-20201014144157771](/Users/wangtao/notes/docs/文章/image-20201014144157771.png)

