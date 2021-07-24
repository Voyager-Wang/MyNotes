## 1.LogMiner是什么

如果要细致学习，推荐直接看官方文档：https://docs.oracle.com/cd/B12037_01/server.101/b10825/logminer.htm#BEGIN

Oracle LogMiner是Oracle公司从产品8i以后提供的一个实际非常有用的日志分析工具，使用该工具可以轻松获得Oracle在线/离线日志（或者叫做重做/归档日志）文件中的具体内容，比如操作数据库的DML和DDL语句。LogMiner本质上是集成在Oracle数据库的一个日志读取、分析工具，不仅可以分析自己数据库的重作日志文件，也可以用来分析其它Oracle数据库的重作日志文件（需要使用该数据库的字典文件）。在默认情况下，Oracle已经安装了LogMiner工具，用户可以建立Oracle数据库的连接后，通过执行sql使用LogMiner。

总的说来，LogMiner工具的主要用途有：

- 1.**跟踪数据库的变化**：可以离线地跟踪数据库的变化，而不会影响在线系统的性能

- 2.回退数据库的变化：回退特定的变化数据，减少Point-In-Time Recovery的执行

- 3.优化和扩容计划：可通过分析日志文件中的数据以分析数据的增长模式

  ...

### 补充：重做日志和归档日志

重做日志（redo log）和归档日志（archive log）是Oracle用于做数据恢复的两种日志，都是物理文件，存放在不同的目录中。归档日志只有在数据库开启了归档模式后才生成（线上使用的数据库一般都会开启）。

重做日志也常称作在线日志（联机日志），是LGWR进程从redo log buffer写入的，是一个内存到磁盘的写入，由于只有在事务的请求被写到重做日志后，请求才能被完成，所以最大限度的提高重做日志的吞吐量是oracle性能优化首先考虑的因素。重做日志最大的特点是循环覆盖写入，重做日志分为多个group（默认3个，可以增加删除），每个group对应不同的文件，一个文件写满后会写入并覆盖下一个group的文件，正常情况下group对应的文件名是不变的。

归档日志是ACR进程从重做日志备份生成的，是一个磁盘到磁盘到写入，可以并发地执行，当一个重做日志的文件写满后，就会触发重做日志写入到归档日志，重做日志备份为归档日志后，系统就会把重做日志的内容清空，但文件依然存在，准备下一次使用；如果某个重做日志要被使用时，该重做日志备份为归档日志还没有完成，那么数据库就会挂起，直到归档日志写完。归档日志没有覆盖写入的情况，它的文件名是由系统自动生成的。

## 2.具体使用方式

在安装了LogMiner工具的Oracle数据库下使用LogMiner是非常简单的，如果没有安装LogMiner工具，需要联系DBA安装配置。LogMiner分析日志的过程是一个session级别的操作，大体上的流程上是添加日志->开始分析->查询结果->结束分析，LogMiner只提供了2个PL/SQL包和若干视图来完成整个日志分析过程，2个PL/SQL包是DBMS_LOGMNR_D包和DBMS_LOGMNR包，包含如下5个方法：

- **DBMS_LOGMNR_D.BUILD**：用来构建数据字典，也就是表的元数据信息，如果没有数据字典，无法从日志中解析得到字段名称等信息；但是这一步构建也不是必须的，完全可以在不构建的数据字典的情况下直接使用在线的数据字典，当然在线数据字典在使用中也存在一些限制，比如缺少对表结构变化的感知
- **DBMS_LOGMNR_D.SET_TABLESPACE**：修改LogMiner视图使用的表空间，默认是SYSTEM空间，最好修改到其他表空间中使用，这一步也不是必须的。
- **DBMS_LOGMNR.ADD_LOGFILE**：用来添加或删除用于分析的日志文件，这一步是必须的，由于LogMiner本质上是一个日志分析工具，因此必须要指定分析哪些日志，提供这些日志的目录和文件名，获取日志的目录和文件名可以通过Oracle提供其他函数进行，比如 SELECT * FROM v$logfile;
- **DBMS_LOGMNR.START_LOGMNR**：用来开启日志分析，调用时可以传入一系列配置参数和过滤条件，这一步也是必须的
- **DBMS_LOGMNR.END_LOGMNR**：用来终止日志分析，它将回收LogMiner所占用的内存

上述提供的方法没有说明如何查询获取的结果，因为查询结果时通过提供的视图实现的，与LogMiner相关的视图：

- **V$LOGHIST**：显示历史日志文件的一些信息

- **V$LOGMNR_DICTIONARY**：因为LOGMINER可以有多个字典文件，所以该视图显示字典文件信息

- **V$LOGMNR_PARAMETERS**：显示LOGMINER的参数

- **V$LOGMNR_LOGS**：显示用于分析的日志列表信息

- **V$LOGMNR_CONTENTS**：LOGMINER结果，在开启日志分析之后，结束日志分析之前，可以从这个视图中获取分析的结果，包括DML DDL等，其他时候查询会报错；另外需要说明的是，这里的SQL不一定是用户的原始DDL，比如

### 典型的LogMiner步骤

- 1.进行初始化设置：开启附加日志，设置LogMiner的表空间等；

- 2.提取一个字典：将字典文件提取为Flat File或Redo日志，或者跳过这一步，后续直接使用Online Catalog数据字典；

- 3.指定需要分析的Redo日志文件：利用DBMS_LOGMNR.ADD_LOGFILE来添加日志；

- 4.开始LogMiner：执行DBMS_LOGMNR.START_LOGMNR来启动LogMiner；

- 5.查询V$LOGMNR_CONTENTS视图；

- 6.结束LogMiner：通过执行EXECUTE DBMS_LOGMNR.END_LOGMNR来结束分析。

### 数据字典的设置

数据字典保存了表的元数据信息，如果不使用数据字典，LogMiner将使用16进制字符显示内部对象ID，不具有可读性。

LogMiner提供了3种提取字典文件的方式：

- 1.将字典文件提取为一个Flat File（平面文件或中间接口文件）

- 2.将字典文件提取为Redo Log

- 3.使用Online Catalog（联机日志）

三种方式不在这里展开说明，可以直接参考https://blog.csdn.net/yes_is_ok/article/details/79296614等分享。

### 使用示例

```sql
/* 开启附加日志： 非必需*/
 ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;

/* 使用过程 */
/* 1.设置字典：元数据 */

/* 2.添加需要分析的日志 */
/* 查看日志路径 */
 SELECT * FROM v$logfile;
EXECUTE dbms_logmnr.add_logfile(logfilename=>'E:\APP\WANGTAO3\ORADATA\DAPAOPAO\REDO01.LOG',options=>dbms_logmnr.NEW);
EXECUTE dbms_logmnr.add_logfile(logfilename=>'E:\APP\WANGTAO3\ORADATA\DAPAOPAO\REDO02.LOG',options=>dbms_logmnr.ADDFILE);
EXECUTE dbms_logmnr.add_logfile(logfilename=>'E:\APP\WANGTAO3\ORADATA\DAPAOPAO\REDO03.LOG',options=>dbms_logmnr.ADDFILE);

/* 查看添加的日志 */
select filename from V$LOGMNR_LOGS;

/* 3.开始分析 */
EXECUTE DBMS_LOGMNR.START_LOGMNR(OPTIONS => DBMS_LOGMNR.DICT_FROM_ONLINE_CATALOG);

/* 4.获取结果 */

SELECT TABLE_NAME,USERNAME, SCN,TO_CHAR(timestamp,'yyyy-mm-dd hh:mi:ss am'),SQL_REDO FROM V$LOGMNR_CONTENTS where SCN > 79164067 AND table_name = 'TEST' ORDER BY timestamp DESC;
SELECT TABLE_NAME,USERNAME, SCN,timestamp,SQL_REDO FROM V$LOGMNR_CONTENTS ORDER BY timestamp DESC;
select sql_redo,timestamp,username,session_info from v$logmnr_contents where table_name='TEST';
SELECT * FROM  V$LOGMNR_CONTENTS;

/* 5.结束 */
EXECUTE DBMS_LOGMNR.END_LOGMNR
/* 结束 */

/* DBA用户下操作 */
/* 执行：sqlplus/nolog */
/* 查看是否开启归档日志 */
ARCHIVE LOG LIST;

/* 数据操作 */
INSERT INTO "DTS"."TEST" (ID, NAME) VALUES ('13', 'logminer2');

DELETE FROM "DTS"."TEST";
```

结果展示：

![image-20200816180537861](https://tinkerer-pic.oss-cn-hangzhou.aliyuncs.com/picgo/image-20200816180537861.png)

## 3.LogMiner用于数据同步

LogMiner用于数据同步是具有可行性的，但是还需要回答几个关键问题才能形成完善的解决方案：

### 3.1 分析哪些日志？

LogMiner既可以分析重做日志也可以分析归档日志，分析重做日志可以获取最新的变更，并且由于重做日志文件名是不变的，不需要为LogMiner重新配置需要解析的文件名；但是一旦涉及到历史变更的获取，就必须要分析归档日志，归档日志的问题在于：只有在重做日志组写满后才写入，因此具有延迟性，不可能只依赖归档日志做到实时的数据同步，另外归档日志不断增加，需要不断调整LogMiner的文件列表，关于LogMiner是否可以同时分析归档日志和重做日志（并做日志的链接和去重），以及自动追加归档日志的列表，还需要进一步探讨，目前没有看到这样的方案。

### 3.2 如何解决日志分析造成的延迟？

开启LogMiner日志分析后，从V$LOGMNR_CONTENTS视图中查询结果，具有一定的延迟（比如上述3个日志文件查询使用了3.5s），这个延迟会直接造成数据同步的延迟。延迟是如何造成的，是否会随着日志增大而增大，还需要进一步研究。

可以关注相关性能测试https://www.cnblogs.com/shishanyuan/p/3142674.html

### 3.3 如何支持DDL？

采用恰当的数据字典方案，可以保证在发生表结构变更时，LogMiner解析得到正确的DML，但是不同的数据字典方案还存在一些其他限制，比如是否需要重启数据库（这在很多时候是不能接受的），是否需要开启附加日志等，在方案的选择上还要做一些探讨和权衡。

### 3.4 是否支持LOB类型？

需要进行测试

### 3.5 如何进行异构数据库的同步？

LogMiner获取的数据变更是以SQL的形式展示的，异构数据库的传输必然需要进行SQL的解析和再拼接，对每一个DML进行解析引入的代价有多高，是否会成为瓶颈，以及是否需要引入并发的机制进行处理，还需要进行分析或测试。

## 参考资料

https://www.cnblogs.com/shishanyuan/p/3142713.html

https://www.cnblogs.com/shishanyuan/p/3142674.html

https://blog.csdn.net/yes_is_ok/article/details/79296614

https://docs.oracle.com/cd/E11882_01/server.112/e22490/logminer.htm#SUTIL019

