官网示例库：https://dev.mysql.com/doc/index-other.html

## 一、性能监控

### 1、MySQL 基础架构

#### 【1】架构图

<img src="_media/database/SQL调优-MySQL/基础架构图.png"/>

SQL 的优化发生在分析器和优化器

#### 【2】连接器

连接器负责跟客户端建立连接，获取权限、维持和管理连接

  - 用户名密码验证
  - 查询权限信息，分配对应的权限
  - 可以使用show processlist查看现在的连接
  - 如果太长时间没有动静，就会自动断开，通过wait_timeout控制，默认8小时

连接可以分为两类：

  - 长连接：推荐使用，但是要周期性的断开长连接
  - 短链接

#### 【3】分析器

词法分析：Mysql需要把输入的字符串进行识别每个部分代表什么意思

  - 把字符串 T 识别成 表名 T 
  - 把字符串 ID 识别成 列 ID

语法分析：根据语法规则判断这个sql语句是否满足mysql的语法，如果不符合就会报错 `You have an error in your SQL synta`

#### 【4】优化器

在具体执行SQL语句之前，要先经过优化器的处理

  - 当表中有多个索引的时候，决定用哪个索引
  - 当sql语句需要做多表关联的时候，决定表的连接顺序 ， 等等

不同的执行方式对SQL语句的执行效率影响很大

  - RBO：基于规则的优化
  - CBO：基于成本的优化 （大多使用）

#### 【5】查询缓存

当执行查询语句的时候，会先去查询缓存中查看结果，之前执行过的sql语句及其结果可能以key-value的形式存储在缓存中，如果能找到则直接返回，如果找不到，就继续执行后续的阶段。

但是，不推荐使用查询缓存：

1. 查询缓存的失效比较频繁，只要表更新，缓存就会清空
2. 缓存对应新更新的数据命中率比较低

注：在 MySQL8.0 后就取消了查询缓存

### 2、profiles

官网文档：https://dev.mysql.com/doc/refman/8.0/en/show-profile.html，在未来版本会被淘汰

常规查询语句显示的时间为小数点后两位，当速度特别快时，会显示 0.00

  <img src="_media/database/SQL调优-MySQL/sql1常规语句.png" alt="image-20210301095808888"    />

`set profiling=1` 可以监控到 sql 的具体时间

常规查询后，使用 `show profiles` ，可以查看 sql 执行时间

  <img src="_media/database/SQL调优-MySQL/showprofiles.png" alt="image-20210301100242490"    />

常规查询后，使用 `show profile `，可以查看 sql 每一步具体执行时间，`show profile for query 1` ，来指定具体查看哪条 SQL

  <img src="_media/database/SQL调优-MySQL/showprofile.png" alt="image-20210301100320800"    />

其他性能属性

| all              | show profile all for query n              | 显示所有性能信息               |
| ---------------- | ----------------------------------------- | ------------------------------ |
| block io         | show profile block io for query n         | 显示块io操作的次数             |
| context switches | show profile context switches for query n | 显示上下文切换次数，被动和主动 |
| cpu              | show profile cpu for query n              | 显示用户CPU时间、系统CPU时间   |
| IPC              | show profile IPC for query n              | 显示发送和接受的消息数量       |
| Memory           |                                           | 暂未实现                       |
| page faults      | show profile page faultsl for query n     | 显示页错误数量                 |
| source           | show profile source for query n           | 显示源码中的函数名称与位置     |
| swaps            | show profile swaps for query n            | 显示swap的此时                 |

### 3、Performance Schema

#### 【1】Performance Schema 介绍

官网文档：https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html

performance schema 用于监控 MySQL server 在一个较低级别的运行过程中的资源消耗、资源等待等情况。

show database ，里面会有一个 performance_schema，里面有 87 张表

Performance Schema 的状态

  - 默认是开启的，可以通过 SHOW VARIABLES LIKE 'performance_schema';  来查看其状态
  - 状态不可以通过命令修改，是只读的，如果修改只能通过配置文件修改

#### 【2】Performance Schema 特点

提供了一种在数据库运行时实时检查server的内部执行情况的方法。performance_schema 数据库中的表使用performance_schema 存储引擎。该数据库主要关注数据库运行过程中的性能相关的数据，与information_schema不同，information_schema主要关注server运行过程中的元数据信息

performance_schema 通过监视 server 的事件来实现监视server内部运行情况， “事件” 就是server内部活动中所做的任何事情以及对应的时间消耗，利用这些信息来判断server中的相关资源消耗在了哪里？一般来说，事件可以是函数调用、操作系统的等待、SQL语句执行的阶段（如sql语句执行过程中的parsing 或 sorting阶段）或者整个SQL语句与SQL语句集合。事件的采集可以方便的提供server中的相关存储引擎对磁盘文件、表I/O、表锁等资源的同步调用信息。

performance_schema 中的事件与写入二进制日志中的事件（描述数据修改的 events）、事件计划调度程序（这是一种存储程序）的事件不同。performance_schema 中的事件记录的是server执行某些活动对某些资源的消耗、耗时、这些活动执行的次数等情况。

performance_schema 中的事件只记录在本地 server 的 performance_schema 中，其下的这些表中数据发生变化时不会被写入 binlog 中，也不会通过复制机制被复制到其他 server 中。

当前活跃事件、历史事件和事件摘要相关的表中记录的信息。能提供某个事件的执行次数、使用时长。进而可用于分析某个特定线程、特定对象（如 mutex 或 file）相关联的活动。

PERFORMANCE_SCHEMA 存储引擎使用 server 源代码中的“检测点”来实现事件数据的收集。对于performance_schema 实现机制本身的代码没有相关的单独线程来检测，这与其他功能（如复制或事件计划程序）不同

收集的事件数据存储在 performance_schema 数据库的表中。这些表可以使用 SELECT 语句查询，也可以使用SQL语句更新 performance_schema 数据库中的表记录（如动态修改 performance_schema 的 setup_* 开头的几个配置表，但要注意：配置表的更改会立即生效，这会影响数据收集）

performance_schema 的表中的数据不会持久化存储在磁盘中，而是保存在内存中，一旦服务器重启，这些数据会丢失（包括配置表在内的整个 performance_schema 下的所有数据）

MySQL 支持的所有平台中事件监控功能都可用，但不同平台中用于统计事件时间开销的计时器类型可能会有所差异。

#### 【3】Performance Schema 操作入门

在mysql的5.7版本中，性能模式是默认开启的，如果想要显式的关闭的话需要修改配置文件，不能直接进行修改，会报错 `Variable 'performance_schema' is a read only variable`

```sql
--查看performance_schema的属性
mysql> SHOW VARIABLES LIKE 'performance_schema';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| performance_schema | ON    |
+--------------------+-------+
1 row in set (0.01 sec)

--在配置文件中修改performance_schema的属性值，on表示开启，off表示关闭
[mysqld]
performance_schema=ON

--切换数据库
use performance_schema;

--查看当前数据库下的所有表,会看到有很多表存储着相关的信息
show tables;

--可以通过show create table tablename来查看创建表的时候的表结构
mysql> show create table setup_consumers;
+-----------------+---------------------------------
| Table           | Create Table                    
+-----------------+---------------------------------
| setup_consumers | CREATE TABLE `setup_consumers` (
  `NAME` varchar(64) NOT NULL,                      
  `ENABLED` enum('YES','NO') NOT NULL               
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8 |  
+-----------------+---------------------------------
1 row in set (0.00 sec)                  
```

基本概念：

1. instruments：生产者，用于采集mysql中各种各样的操作产生的事件信息，对应配置表中的配置项我们可以称为监控采集配置项。
2. consumers：消费者，对应的消费者表用于存储来自instruments采集的数据，对应配置表中的配置项我们可以称为消费存储配置项。

#### 【4】performance_schema表的分类

performance_schema库下的表可以按照监视不同的纬度就行分组。

```sql
--语句事件记录表，这些表记录了语句事件信息，当前语句事件表events_statements_current、历史语句事件表events_statements_history和长语句历史事件表events_statements_history_long、以及聚合后的摘要表summary，其中，summary表还可以根据帐号(account)，主机(host)，程序(program)，线程(thread)，用户(user)和全局(global)再进行细分)
show tables like '%statement%';

--等待事件记录表，与语句事件类型的相关记录表类似：
show tables like '%wait%';

--阶段事件记录表，记录语句执行的阶段事件的表
show tables like '%stage%';

--事务事件记录表，记录事务相关的事件的表
show tables like '%transaction%';

--监控文件系统层调用的表
show tables like '%file%';

--监视内存使用的表
show tables like '%memory%';

--动态对performance_schema进行配置的配置表
show tables like '%setup%';
```


#### 【5】performance_schema的简单配置与使用

据库刚刚初始化并启动时，并非所有 instruments（事件采集项，在采集项的配置表中每一项都有一个开关字段，或为YES，或为NO）和 consumers（与采集项类似，也有一个对应的事件类型保存表配置项，为YES就表示对应的表保存性能数据，为NO就表示对应的表不保存性能数据）都启用了，所以默认不会收集所有的事件，可能你需要检测的事件并没有打开，需要进行设置，可以使用如下两个语句打开对应的instruments和consumers（行计数可能会因MySQL版本而异）。

```sql
--打开等待事件的采集器配置项开关，需要修改setup_instruments配置表中对应的采集器配置项
UPDATE setup_instruments SET ENABLED = 'YES', TIMED = 'YES'where name like 'wait%';

--打开等待事件的保存表配置开关，修改setup_consumers配置表中对应的配置项
UPDATE setup_consumers SET ENABLED = 'YES'where name like '%wait%';

--当配置完成之后可以查看当前server正在做什么，可以通过查询events_waits_current表来得知，该表中每个线程只包含一行数据，用于显示每个线程的最新监视事件
select * from events_waits_current\G
*************************** 1. row ***************************
            THREAD_ID: 11
             EVENT_ID: 570
         END_EVENT_ID: 570
           EVENT_NAME: wait/synch/mutex/innodb/buf_dblwr_mutex
               SOURCE: 
          TIMER_START: 4508505105239280
            TIMER_END: 4508505105270160
           TIMER_WAIT: 30880
                SPINS: NULL
        OBJECT_SCHEMA: NULL
          OBJECT_NAME: NULL
           INDEX_NAME: NULL
          OBJECT_TYPE: NULL
OBJECT_INSTANCE_BEGIN: 67918392
     NESTING_EVENT_ID: NULL
   NESTING_EVENT_TYPE: NULL
            OPERATION: lock
      NUMBER_OF_BYTES: NULL
                FLAGS: NULL
/*该信息表示线程id为11的线程正在等待buf_dblwr_mutex锁，等待事件为30880
属性说明：
	id:事件来自哪个线程，事件编号是多少
	event_name:表示检测到的具体的内容
	source:表示这个检测代码在哪个源文件中以及行号
	timer_start:表示该事件的开始时间
	timer_end:表示该事件的结束时间
	timer_wait:表示该事件总的花费时间
注意：_current表中每个线程只保留一条记录，一旦线程完成工作，该表中不会再记录该线程的事件信息
*/

/*
_history表中记录每个线程应该执行完成的事件信息，但每个线程的事件信息只会记录10条，再多就会被覆盖，*_history_long表中记录所有线程的事件信息，但总记录数量是10000，超过就会被覆盖掉
*/
select thread_id,event_id,event_name,timer_wait from events_waits_history order by thread_id limit 21;

/*
summary表提供所有事件的汇总信息，该组中的表以不同的方式汇总事件数据（如：按用户，按主机，按线程等等）。例如：要查看哪些instruments占用最多的时间，可以通过对events_waits_summary_global_by_event_name表的COUNT_STAR或SUM_TIMER_WAIT列进行查询（这两列是对事件的记录数执行COUNT（*）、事件记录的TIMER_WAIT列执行SUM（TIMER_WAIT）统计而来）
*/
SELECT EVENT_NAME,COUNT_STAR FROM events_waits_summary_global_by_event_name  ORDER BY COUNT_STAR DESC LIMIT 10;

/*
instance表记录了哪些类型的对象会被检测。这些对象在被server使用时，在该表中将会产生一条事件记录，例如，file_instances表列出了文件I/O操作及其关联文件名
*/
select * from file_instances limit 20; 
```


#### 【6】常用配置项的参数说明

启动选项

```sql
performance_schema_consumer_events_statements_current=TRUE
/*是否在mysql server启动时就开启events_statements_current表的记录功能(该表记录当前的语句事件信息)，启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新setup_consumers配置表中的events_statements_current配置项，默认值为TRUE*/

performance_schema_consumer_events_statements_history=TRUE
/*与performance_schema_consumer_events_statements_current选项类似，但该选项是用于配置是否记录语句事件短历史信息，默认为TRUE*/

performance_schema_consumer_events_stages_history_long=FALSE
/*与performance_schema_consumer_events_statements_current选项类似，但该选项是用于配置是否记录语句事件长历史信息，默认为FALSE*/

/*除了statement(语句)事件之外，还支持：wait(等待)事件、state(阶段)事件、transaction(事务)事件，他们与statement事件一样都有三个启动项分别进行配置，但这些等待事件默认未启用，如果需要在MySQL Server启动时一同启动，则通常需要写进my.cnf配置文件中*/

performance_schema_consumer_global_instrumentation=TRUE
/*是否在MySQL Server启动时就开启全局表（如：mutex_instances、rwlock_instances、cond_instances、file_instances、users、hostsaccounts、socket_summary_by_event_name、file_summary_by_instance等大部分的全局对象计数统计和事件汇总统计信息表 ）的记录功能，启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新全局配置项，默认值为TRUE*/

/*performance_schema_consumer_statements_digest=TRUE
是否在MySQL Server启动时就开启events_statements_summary_by_digest 表的记录功能，启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新digest配置项
默认值为TRUE*/

performance_schema_consumer_thread_instrumentation=TRUE
/*是否在MySQL Server启动时就开启

events_xxx_summary_by_yyy_by_event_name表的记录功能，启动之后也可以在setup_consumers表中使用UPDATE语句进行动态更新线程配置项
默认值为TRUE*/

performance_schema_instrument[=name]
/*是否在MySQL Server启动时就启用某些采集器，由于instruments配置项多达数千个，所以该配置项支持key-value模式，还支持%号进行通配等，如下:*/

# [=name]可以指定为具体的Instruments名称（但是这样如果有多个需要指定的时候，就需要使用该选项多次），也可以使用通配符，可以指定instruments相同的前缀+通配符，也可以使用%代表所有的instruments

## 指定开启单个instruments

--performance-schema-instrument= 'instrument_name=value'

## 使用通配符指定开启多个instruments

--performance-schema-instrument= 'wait/synch/cond/%=COUNTED'

## 开关所有的instruments

--performance-schema-instrument= '%=ON'

--performance-schema-instrument= '%=OFF'

注意，这些启动选项要生效的前提是，需要设置performance_schema=ON。另外，这些启动选项虽然无法使用show variables语句查看，但我们可以通过setup_instruments和setup_consumers表查询这些选项指定的值。
```

系统变量

```sql
show variables like '%performance_schema%';
--重要的属性解释
performance_schema=ON
/*
控制performance_schema功能的开关，要使用MySQL的performance_schema，需要在mysqld启动时启用，以启用事件收集功能
该参数在5.7.x之前支持performance_schema的版本中默认关闭，5.7.x版本开始默认开启
注意：如果mysqld在初始化performance_schema时发现无法分配任何相关的内部缓冲区，则performance_schema将自动禁用，并将performance_schema设置为OFF
*/

performance_schema_digests_size=10000
/*
控制events_statements_summary_by_digest表中的最大行数。如果产生的语句摘要信息超过此最大值，便无法继续存入该表，此时performance_schema会增加状态变量
*/
performance_schema_events_statements_history_long_size=10000
/*
控制events_statements_history_long表中的最大行数，该参数控制所有会话在events_statements_history_long表中能够存放的总事件记录数，超过这个限制之后，最早的记录将被覆盖
全局变量，只读变量，整型值，5.6.3版本引入 * 5.6.x版本中，5.6.5及其之前的版本默认为10000，5.6.6及其之后的版本默认值为-1，通常情况下，自动计算的值都是10000 * 5.7.x版本中，默认值为-1，通常情况下，自动计算的值都是10000
*/
performance_schema_events_statements_history_size=10
/*
控制events_statements_history表中单个线程（会话）的最大行数，该参数控制单个会话在events_statements_history表中能够存放的事件记录数，超过这个限制之后，单个会话最早的记录将被覆盖
全局变量，只读变量，整型值，5.6.3版本引入 * 5.6.x版本中，5.6.5及其之前的版本默认为10，5.6.6及其之后的版本默认值为-1，通常情况下，自动计算的值都是10 * 5.7.x版本中，默认值为-1，通常情况下，自动计算的值都是10
除了statement(语句)事件之外，wait(等待)事件、state(阶段)事件、transaction(事务)事件，他们与statement事件一样都有三个参数分别进行存储限制配置，有兴趣的同学自行研究，这里不再赘述
*/
performance_schema_max_digest_length=1024
/*
用于控制标准化形式的SQL语句文本在存入performance_schema时的限制长度，该变量与max_digest_length变量相关(max_digest_length变量含义请自行查阅相关资料)
全局变量，只读变量，默认值1024字节，整型值，取值范围0~1048576
*/
performance_schema_max_sql_text_length=1024
/*
控制存入events_statements_current，events_statements_history和events_statements_history_long语句事件表中的SQL_TEXT列的最大SQL长度字节数。 超出系统变量performance_schema_max_sql_text_length的部分将被丢弃，不会记录，一般情况下不需要调整该参数，除非被截断的部分与其他SQL比起来有很大差异
全局变量，只读变量，整型值，默认值为1024字节，取值范围为0~1048576，5.7.6版本引入
降低系统变量performance_schema_max_sql_text_length值可以减少内存使用，但如果汇总的SQL中，被截断部分有较大差异，会导致没有办法再对这些有较大差异的SQL进行区分。 增加该系统变量值会增加内存使用，但对于汇总SQL来讲可以更精准地区分不同的部分。
*/
```


#### 【7】重要配置表的相关说明

配置表之间存在相互关联关系，按照配置影响的先后顺序，可添加为

<img src="_media/database/SQL调优-MySQL/配置表.png" alt="image-20210301155214736"    />

```sql
/*
performance_timers表中记录了server中有哪些可用的事件计时器
字段解释：
	timer_name:表示可用计时器名称，CYCLE是基于CPU周期计数器的定时器
	timer_frequency:表示每秒钟对应的计时器单位的数量,CYCLE计时器的换算值与CPU的频率相关、
	timer_resolution:计时器精度值，表示在每个计时器被调用时额外增加的值
	timer_overhead:表示在使用定时器获取事件时开销的最小周期值
*/
select * from performance_timers;

/*
setup_timers表中记录当前使用的事件计时器信息
字段解释：
	name:计时器类型，对应某个事件类别
	timer_name:计时器类型名称
*/
select * from setup_timers;

/*
setup_consumers表中列出了consumers可配置列表项
字段解释：
	NAME：consumers配置名称
	ENABLED：consumers是否启用，有效值为YES或NO，此列可以使用UPDATE语句修改。
*/
select * from setup_consumers;

/*
setup_instruments 表列出了instruments 列表配置项，即代表了哪些事件支持被收集：
字段解释：
	NAME：instruments名称，instruments名称可能具有多个部分并形成层次结构
	ENABLED：instrumetns是否启用，有效值为YES或NO，此列可以使用UPDATE语句修改。如果设置为NO，则这个instruments不会被执行，不会产生任何的事件信息
	TIMED：instruments是否收集时间信息，有效值为YES或NO，此列可以使用UPDATE语句修改，如果设置为NO，则这个instruments不会收集时间信息
*/
SELECT * FROM setup_instruments;

/*
setup_actors表的初始内容是匹配任何用户和主机，因此对于所有前台线程，默认情况下启用监视和历史事件收集功能
字段解释：
	HOST：与grant语句类似的主机名，一个具体的字符串名字，或使用“％”表示“任何主机”
	USER：一个具体的字符串名称，或使用“％”表示“任何用户”
	ROLE：当前未使用，MySQL 8.0中才启用角色功能
	ENABLED：是否启用与HOST，USER，ROLE匹配的前台线程的监控功能，有效值为：YES或NO
	HISTORY：是否启用与HOST， USER，ROLE匹配的前台线程的历史事件记录功能，有效值为：YES或NO
*/
SELECT * FROM setup_actors;

/*
setup_objects表控制performance_schema是否监视特定对象。默认情况下，此表的最大行数为100行。
字段解释：
	OBJECT_TYPE：instruments类型，有效值为：“EVENT”（事件调度器事件）、“FUNCTION”（存储函数）、“PROCEDURE”（存储过程）、“TABLE”（基表）、“TRIGGER”（触发器），TABLE对象类型的配置会影响表I/O事件（wait/io/table/sql/handler instrument）和表锁事件（wait/lock/table/sql/handler instrument）的收集
	OBJECT_SCHEMA：某个监视类型对象涵盖的数据库名称，一个字符串名称，或“％”(表示“任何数据库”)
	OBJECT_NAME：某个监视类型对象涵盖的表名，一个字符串名称，或“％”(表示“任何数据库内的对象”)
	ENABLED：是否开启对某个类型对象的监视功能，有效值为：YES或NO。此列可以修改
	TIMED：是否开启对某个类型对象的时间收集功能，有效值为：YES或NO，此列可以修改
*/
SELECT * FROM setup_objects;

/*
threads表对于每个server线程生成一行包含线程相关的信息，
字段解释：
	THREAD_ID：线程的唯一标识符（ID）
	NAME：与server中的线程检测代码相关联的名称(注意，这里不是instruments名称)
	TYPE：线程类型，有效值为：FOREGROUND、BACKGROUND。分别表示前台线程和后台线程
	PROCESSLIST_ID：对应INFORMATION_SCHEMA.PROCESSLIST表中的ID列。
	PROCESSLIST_USER：与前台线程相关联的用户名，对于后台线程为NULL。
	PROCESSLIST_HOST：与前台线程关联的客户端的主机名，对于后台线程为NULL。
	PROCESSLIST_DB：线程的默认数据库，如果没有，则为NULL。
	PROCESSLIST_COMMAND：对于前台线程，该值代表着当前客户端正在执行的command类型，如果是sleep则表示当前会话处于空闲状态
	PROCESSLIST_TIME：当前线程已处于当前线程状态的持续时间（秒）
	PROCESSLIST_STATE：表示线程正在做什么事情。
	PROCESSLIST_INFO：线程正在执行的语句，如果没有执行任何语句，则为NULL。
	PARENT_THREAD_ID：如果这个线程是一个子线程（由另一个线程生成），那么该字段显示其父线程ID
	ROLE：暂未使用
	INSTRUMENTED：线程执行的事件是否被检测。有效值：YES、NO 
	HISTORY：是否记录线程的历史事件。有效值：YES、NO * 
	THREAD_OS_ID：由操作系统层定义的线程或任务标识符（ID）：
*/
select * from threads
```

注意：在performance_schema库中还包含了很多其他的库和表，能对数据库的性能做完整的监控，大家需要参考官网详细了解。

#### 【8】performance_schema实践操作

```sql
--1、哪类的SQL执行最多？
SELECT DIGEST_TEXT,COUNT_STAR,FIRST_SEEN,LAST_SEEN FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
--2、哪类SQL的平均响应时间最多？
SELECT DIGEST_TEXT,AVG_TIMER_WAIT FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
--3、哪类SQL排序记录数最多？
SELECT DIGEST_TEXT,SUM_SORT_ROWS FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
--4、哪类SQL扫描记录数最多？
SELECT DIGEST_TEXT,SUM_ROWS_EXAMINED FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
--5、哪类SQL使用临时表最多？
SELECT DIGEST_TEXT,SUM_CREATED_TMP_TABLES,SUM_CREATED_TMP_DISK_TABLES FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
--6、哪类SQL返回结果集最多？
SELECT DIGEST_TEXT,SUM_ROWS_SENT FROM events_statements_summary_by_digest ORDER BY COUNT_STAR DESC
--7、哪个表物理IO最多？
SELECT file_name,event_name,SUM_NUMBER_OF_BYTES_READ,SUM_NUMBER_OF_BYTES_WRITE FROM file_summary_by_instance ORDER BY SUM_NUMBER_OF_BYTES_READ + SUM_NUMBER_OF_BYTES_WRITE DESC
--8、哪个表逻辑IO最多？
SELECT object_name,COUNT_READ,COUNT_WRITE,COUNT_FETCH,SUM_TIMER_WAIT FROM table_io_waits_summary_by_table ORDER BY sum_timer_wait DESC
--9、哪个索引访问最多？
SELECT OBJECT_NAME,INDEX_NAME,COUNT_FETCH,COUNT_INSERT,COUNT_UPDATE,COUNT_DELETE FROM table_io_waits_summary_by_index_usage ORDER BY SUM_TIMER_WAIT DESC
--10、哪个索引从来没有用过？
SELECT OBJECT_SCHEMA,OBJECT_NAME,INDEX_NAME FROM table_io_waits_summary_by_index_usage WHERE INDEX_NAME IS NOT NULL AND COUNT_STAR = 0 AND OBJECT_SCHEMA <> 'mysql' ORDER BY OBJECT_SCHEMA,OBJECT_NAME;
--11、哪个等待事件消耗时间最多？
SELECT EVENT_NAME,COUNT_STAR,SUM_TIMER_WAIT,AVG_TIMER_WAIT FROM events_waits_summary_global_by_event_name WHERE event_name != 'idle' ORDER BY SUM_TIMER_WAIT DESC
--12-1、剖析某条SQL的执行情况，包括statement信息，stege信息，wait信息
SELECT EVENT_ID,sql_text FROM events_statements_history WHERE sql_text LIKE '%count(*)%';
--12-2、查看每个阶段的时间消耗
SELECT event_id,EVENT_NAME,SOURCE,TIMER_END - TIMER_START FROM events_stages_history_long WHERE NESTING_EVENT_ID = 1553;
--12-3、查看每个阶段的锁等待情况
SELECT event_id,event_name,source,timer_wait,object_name,index_name,operation,nesting_event_id FROM events_waits_history_longWHERE nesting_event_id = 1553;
```

### 4、processlist

官网文档：https://dev.mysql.com/doc/refman/8.0/en/show-processlist.html

使用show processlist查看连接的线程个数，来观察是否有大量线程处于不正常的状态或者其他不正常的特征

<img src="_media/database/SQL调优-MySQL/连接线程数.png" style="zoom:200%;" />

属性说明：

```shell
 id  # 表示session id
 user  # 表示操作的用户
 host  # 表示操作的主机
 db  # 表示操作的数据库
 command  # 表示当前状态
 	sleep  # 线程正在等待客户端发送新的请求
 	query  # 线程正在执行查询或正在将结果发送给客户端
 	locked  # 在mysql的服务层，该线程正在等待表锁
 	analyzing and statistics  # 线程正在收集存储引擎的统计信息，并生成查询的执行计划
 	Copying to tmp table  # 线程正在执行查询，并且将其结果集都复制到一个临时表中
 	sorting result  # 线程正在对结果集进行排序
 	sending data  # 线程可能在多个状态之间传送数据，或者在生成结果集或者向客户端返回数据
 info  # 表示详细的sql语句
 time  # 表示相应命令执行时间
 state  # 表示命令执行状态
```

一般情况用的比较少，有连接池的存在不会出现大量连接的情况

## 二、数据类型和schema优化

### 1、数据类型的优化

#### 【1】更小的通常更好

应该尽量使用可以正确存储数据的最小数据类型，更小的数据类型通常更快，因为它们占用更少的磁盘、内存和CPU缓存，并且处理时需要的CPU周期更少，但是要确保没有低估需要存储的值的范围，如果无法确认哪个数据类型，就选择你认为不会超过范围的最小类型

案例：设计两张表，设计不同的数据类型，查看表的容量

#### 【2】简单就好

简单数据类型的操作通常需要更少的CPU周期，例如：

1. 整型比字符操作代价更低，因为字符集和校对规则是字符比较比整型比较更复杂，
2. 使用mysql自建类型而不是字符串来存储日期和时间
3. 用整型存储IP地址
   - `select INET_ATON('192.160.60.11') ` -> 转为整形  3232250891

     <img src="_media/database/SQL调优-MySQL/IP转int.png" alt="image-20210301163733640"    />
   - `select INET_NTOA('3232250891') ` -> 转为 IP 3232250891

     <img src="_media/database/SQL调优-MySQL/int转ip.png" alt="image-20210301163939050"    />
   - 最大可以到 255.255.255.255

案例：创建两张相同的表，改变日期的数据类型，查看SQL语句执行的速度，权衡（节省数据存储空间，还是提高查询效率）

#### 【3】尽量避免 null

在数据库中 null ≠ null

如果查询中包含可为NULL的列，对mysql来说很难优化，因为可为null的列使得索引、索引统计和值比较都更加复杂

坦白来说，通常情况下null的列改为not null带来的性能提升比较小，所有没有必要将所有的表的schema进行修改，但是应该尽量避免设计成可为null的列

#### 【4】实际细则 —— 整数类型

可以使用的几种整数类型：
- TINYINT —— 8位，
- SMALLINT —— 16位，
- MEDIUMINT —— 24位，
- INT —— 32位，
- BIGINT —— 64位

尽量使用满足需求的最小数据类型

#### 【5】实际细则 —— 字符串和字符类型

varchar：
1. varchar可变程度，可以设置最大长度；最大空间是65535个字节，适合用在长度可变的属性
2. varchar根据实际内容长度保存数据
   1. 使用最小的符合需求的长度。
   2. varchar(n) n小于等于255使用额外一个字节保存长度，n>255使用额外两个字节保存长度。
   3. varchar(5)与varchar(255)保存同样的内容，硬盘存储空间相同，但内存空间占用不同，是指定的大小 。
   4. varchar在mysql5.6之前变更长度，或者从255一下变更到255以上时时，都会导致锁表。
3. 应用场景
   1. 存储长度波动较大的数据，如：文章，有的会很短有的会很长
   2. 字符串很少更新的场景，每次更新后都会重算并使用额外存储空间保存长度
   3. 适合保存多字节字符，如：汉字，特殊字符等

char：
1. char长度固定，即每条数据占用等长字节空间；最大长度是255个字符，适合用在身份证号、手机号等定长字符串
2. char固定长度的字符串
   1. 最大长度：255
   2. 会自动删除末尾的空格
   3. 检索效率、写效率 会比varchar高，以空间换时间
3. 应用场景
   1. 存储长度波动不大的数据，如：md5摘要
   2. 存储短字符串、经常更新的字符串

#### 【6】实际细则 —— BLOB 和 TEXT 类型

MySQL 把每个 BLOB 和 TEXT 值当作一个独立的对象处理。

两者都是为了存储很大数据而设计的字符串类型，分别采用二进制和字符方式存储。

text不设置长度，当不知道属性的最大长度时，适合用text

按照查询速度：char>varchar>text

#### 【7】实际细则 —— datetime 和 timesstamp

细则：
1. 不要使用字符串类型来存储日期时间数据
2. 日期时间类型通常比字符串占用的存储空间小
3. 日期时间类型在进行查找过滤时可以利用日期来进行比对
4. 日期时间类型还有着丰富的处理函数，可以方便的对时间类型进行日期计算
5. 使用int存储日期时间不如使用timestamp类型

datetime
1. 占用8个字节
2. 与时区无关，数据库底层时区配置，对datetime无效
3. 可保存到毫秒
4. 可保存时间范围大
5. 不要使用字符串存储日期类型，占用空间大，损失日期类型函数的便捷性

timestamp （多数使用）
1. 占用4个字节
2. 时间范围：1970-01-01 到 2038-01-19
3. 精确到秒
4. 采用整形存储
5. 依赖数据库设置的时区
6. 自动更新 timestamp 列的值

date
1. 占用的字节数比使用字符串、datetime、int 存储要少，使用 date 类型只需要 3 个字节
2. 使用 date 类型还可以利用日期时间函数进行日期之间的计算
3. date 类型用于保存 1000-01-01 到 9999-12-31 之间的日期

#### 【8】实际细则 —— 使用枚举代替字符串类型

有时可以使用枚举类代替常用的字符串类型，mysql存储枚举类型会非常紧凑，会根据列表值的数据压缩到一个或两个字节中，mysql在内部会将每个值在列表中的位置保存为整数，并且在表的 .frm 文件中保存“数字 - 字符串”映射关系的查找表
-  `create table enum_test(e enum('fish','apple','dog') not null);`
-  `insert into enum_test(e) values('fish'),('dog'),('apple');`
-  `select e+0 from enum_test;`

- 表中数据不会更改可以使用，性别，国家，省市区  ...

#### 【9】实际细则 —— 特殊类型数据

人们经常使用 varchar(15) 来存储 ip 地址，然而，它的本质是 32 位无符号整数不是字符串，可以使用 `INET_ATON()` 和 `INET_NTOA` 函数在这两种表示方法之间转换

案例：`select inet_aton('1.1.1.1')`   \  `select inet_ntoa(16843009)`

### 2、合理使用范式和反范式

三范式：
1. 列不可分
2. 不能存在传递依赖
3. 表中其它列的值必须依赖于主键

范式
1. 优点：
   1. 范式化的更新通常比反范式要快
   2. 当数据较好的范式化后，很少或者没有重复的数据
   3. 范式化的数据比较小，可以放在内存中，操作比较快
2. 缺点：通常需要进行关联

反范式
1. 优点
   1. 所有的数据都在同一张表中，可以避免关联
   2. 可以设计有效的索引；
2. 缺点：表格内的冗余较多，删除数据时候会造成表有些有用的信息丢失

在企业中很好能做到严格意义上的范式或者反范式，一般需要混合使用
1. 在一个网站实例中，这个网站，允许用户发送消息，并且一些用户是付费用户。现在想查看付费用户最近的10条信息。  在user表和message表中都存储用户类型(account_type)而不用完全的反范式化。这避免了完全反范式化的插入和删除问题，因为即使没有消息的时候也绝不会丢失用户的信息。这样也不会把user_message表搞得太大，有利于高效地获取数据。
2. 另一个从父表冗余一些数据到子表的理由是排序的需要。
3. 缓存衍生值也是有用的。如果需要显示每个用户发了多少消息（类似论坛的），可以每次执行一个昂贵的自查询来计算并显示它；也可以在user表中建一个num_messages列，每当用户发新消息时更新这个值。

案例
1. 范式设计

   <img src="_media/database/SQL调优-MySQL/范式设计.png" alt="image-20210301214230731" style="zoom: 67%;" />
2. 反范式设计

   <img src="_media/database/SQL调优-MySQL/反范式设计.png" alt="image-20210301214258651"    />

### 3、主键的选择

代理主键：与业务无关的，无意义的数字序列

自然主键：事物属性中的自然唯一标识

推荐使用代理主键
- 它们不与业务耦合，因此更容易维护
- 一个大多数表，最好是全部表，通用的键策略能够减少需要编写的源码数量，减少系统的总体拥有成本
- 可以写一个主键生成器，用一个规则使主键不重复，所有类都可以使用，如果使用与业务相关的主键，不方便维护

### 4、字符集的选择

字符集直接决定了数据在 MySQL 中的存储编码方式，由于同样的内容使用不同字符集表示所占用的空间大小会有较大的差异，所以通过使用合适的字符集，可以帮助我们尽可能减少数据量，进而减少 IO 操作次数。

纯拉丁字符能表示的内容，没必要选择 latin1 之外的其他字符编码，因为这会节省大量的存储空间。

如果我们可以确定不需要存放多种语言，就没必要非得使用 UTF8 或者其他 UNICODE 字符类型，这回造成大量的存储空间浪费。

MySQL 的数据类型可以精确到字段，所以当我们需要大型数据库中存放多字节数据的时候，可以通过对不同表不同字段使用不同的数据类型来较大程度减小数据存储量，进而降低 IO 操作次数并提高缓存命中率。

utf8 只能存两个字节的中文，可以更换为 utf8mb4

### 5、存储引擎的选择

存储引擎的对比

<img src="_media/database/SQL调优-MySQL/存储引擎的对比.png"/>

默认为 InnoDB

<img src="_media/database/SQL调优-MySQL/myini.png"/>

### 6、适当的数据冗余

被频繁引用且只能通过 Join 2张(或者更多)大表的方式才能得到的独立小字段。

这样的场景由于每次Join仅仅只是为了取得某个小字段的值，Join到的记录又大，会造成大量不必要的 IO，完全可以通过空间换取时间的方式来优化。不过，冗余的同时需要确保数据的一致性不会遭到破坏，确保更新的同时冗余字段也被更新。

### 7、适当拆分

当我们的表中存在类似于 TEXT 或者是很大的 VARCHAR类型的大字段的时候，如果我们大部分访问这张表的时候都不需要这个字段，我们就该义无反顾的将其拆分到另外的独立表中，以减少常用数据所占用的存储空间。

这样做的一个明显好处就是每个数据块中可以存储的数据条数可以大大增加，既减少物理 IO 次数，也能大大提高内存中的缓存命中率。

垂直切分：按照不同业务进行切分，将表放在不同的物理服务器里，这样请求会分散到不同的服务器，减少压力

水平切分：按照表中数据进行切分，1 - 1000 放到一个物理服务器，1001 - 2000 放到一个服务器，减少压力

### 8、执行计划 explain

#### 【1】执行计划概述

官网文档：https://dev.mysql.com/doc/refman/8.0/en/explain.html

在企业的应用场景中，为了知道优化SQL语句的执行，需要查看SQL语句的具体执行过程，以加快SQL语句的执行效率。

可以使用explain+SQL语句来模拟优化器执行SQL查询语句，从而知道mysql是如何处理sql语句的。

#### 【2】包含的信息

| Column        | Meaning                                        |
| ------------- | ---------------------------------------------- |
| id            | The `SELECT` identifier                        |
| select_type   | The `SELECT` type                              |
| table         | The table for the output row                   |
| partitions    | The matching partitions                        |
| type          | The join type                                  |
| possible_keys | The possible indexes to choose                 |
| key           | The index actually chosen                      |
| key_len       | The length of the chosen key                   |
| ref           | The columns compared to the index              |
| rows          | Estimate of rows to be examined                |
| filtered      | Percentage of rows filtered by table condition |
| extra         | Additional information                         |

<img src="_media/database/SQL调优-MySQL/查询计划.png"/>

#### 【3】内容分析

id：select 查询的序列号，包含一组数字，表示查询中执行select子句或者操作表的顺序

1. 如果id相同，那么执行顺序从上到下

   ```sql
   explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal; 
   ```

2. 如果id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

   ```sql
   explain select * from emp e where e.deptno in (select d.deptno from dept d where d.dname = 'SALES'); 
   ```

3. id相同和不同的，同时存在：相同的可以认为是一组，从上往下顺序执行，在所有组中，id值越大，优先级越高，越先执行

   ```sql
   explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal where e.deptno in (select d.deptno from dept d where d.dname = 'SALES'); 
   ```

select_type：主要用来分辨查询的类型，是普通查询还是联合查询还是子查询

| Value                | Meaning                                                      |
| -------------------- | ------------------------------------------------------------ |
| SIMPLE               | Simple SELECT (not using UNION or subqueries)                |
| PRIMARY              | Outermost SELECT                                             |
| UNION                | Second or later SELECT statement in a UNION                  |
| DEPENDENT UNION      | Second or later SELECT statement in a UNION, dependent on outer query |
| UNION RESULT         | Result of a UNION.                                           |
| SUBQUERY             | First SELECT in subquery                                     |
| DEPENDENT SUBQUERY   | First SELECT in subquery, dependent on outer query           |
| DERIVED              | Derived table                                                |
| UNCACHEABLE SUBQUERY | A subquery for which the result cannot be cached and must be re-evaluated for each row of the outer query |
| UNCACHEABLE UNION    | The second or later select in a UNION that belongs to an uncacheable subquery (see UNCACHEABLE SUBQUERY) |

```sql
--sample:简单的查询，不包含子查询和union
explain select * from emp;

--primary:查询中若包含任何复杂的子查询，最外层查询则被标记为Primary
explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;

--union:若第二个select出现在union之后，则被标记为union
explain select * from emp where deptno = 10 union select * from emp where sal >2000;

--dependent union:跟union类似，此处的depentent表示union或union all联合而成的结果会受外部表影响
explain select * from emp e where e.empno  in ( select empno from emp where deptno = 10 union select empno from emp where sal >2000)

--union result:从union表获取结果的select
explain select * from emp where deptno = 10 union select * from emp where sal >2000;

--subquery:在select或者where列表中包含子查询
explain select * from emp where sal > (select avg(sal) from emp) ;

--dependent subquery:subquery的子查询要受到外部表查询的影响
explain select * from emp e where e.deptno in (select distinct deptno from dept);

--DERIVED: from子句中出现的子查询，也叫做派生类，
explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;

--UNCACHEABLE SUBQUERY：表示使用子查询的结果不能被缓存
 explain select * from emp where empno = (select empno from emp where deptno=@@sort_buffer_size);
 
--uncacheable union:表示union的查询结果不能被缓存：sql语句未验证
```

table ：对应行正在访问哪一个表，表名或者别名，可能是临时表或者 union 合并结果集

1. 如果是具体的表名，则表明从实际的物理表中获取数据，当然也可以是表的别名
2. 表名是 derivedN 的形式，表示使用了id为N的查询产生的衍生表
3. 当有 union result 的时候，表名是union n1,n2等的形式，n1,n2表示参与union的id

type：显示的是访问类型

1. 访问类型表示我是以何种方式去访问我们的数据，最容易想的是全表扫描，直接暴力的遍历一张表去寻找需要的数据，效率非常低下

2. 访问的类型有很多，效率从最好到最坏依次是：

   - `system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL`

   - 一般情况下，得保证查询至少达到range级别，最好能达到ref

   ```sql
   --all:全表扫描，一般情况下出现这样的sql语句而且数据量比较大的话那么就需要进行优化。
   explain select * from emp;
   
   --index：全索引扫描这个比all的效率要好，主要有两种情况，一种是当前的查询时覆盖索引，即我们需要的数据在索引中就可以索取，或者是使用了索引进行排序，这样就避免数据的重排序
   explain  select empno from emp;
   
   --range：表示利用索引查询的时候限制了范围，在指定范围内进行查询，这样避免了index的全索引扫描，适用的操作符： =, <>, >, >=, <, <=, IS NULL, BETWEEN, LIKE, or IN() 
   explain select * from emp where empno between 7000 and 7500;
   
   --index_subquery：利用索引来关联子查询，不再扫描全表
   explain select * from emp where emp.job in (select job from t_job);
   
   --unique_subquery:该连接类型类似与index_subquery,使用的是唯一索引
    explain select * from emp e where e.deptno in (select distinct deptno from dept);
    
   --index_merge：在查询过程中需要多个索引组合使用，没有模拟出来
   
   --ref_or_null：对于某个字段即需要关联条件，也需要null值的情况下，查询优化器会选择这种访问方式
   explain select * from emp e where  e.mgr is null or e.mgr=7369;
   
   --ref：使用了非唯一性索引进行数据的查找
    create index idx_3 on emp(deptno);
    explain select * from emp e,dept d where e.deptno =d.deptno;
   
   --eq_ref ：使用唯一性索引进行数据查找
   explain select * from emp,emp2 where emp.empno = emp2.empno;
   
   --const：这个表至多有一个匹配行，
   explain select * from emp where empno = 7369;
    
   --system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现
   ```

possible_keys：显示可能应用在这张表中的索引，一个或多个，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10; 
```

key：实际使用的索引，如果为null，则没有使用索引，查询中若使用了覆盖索引，则该索引和查询的select字段重叠。

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

key_len：表示索引中使用的字节数，可以通过key_len计算查询中使用的索引长度，在不损失精度的情况下长度越短越好。

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10; 
```

ref：显示索引的哪一列被使用了，如果可能的话，是一个常数

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

rows：根据表的统计信息及索引使用情况，大致估算出找出所需记录需要读取的行数，此参数很重要，直接反应的sql找了多少数据，在完成目的的情况下越少越好

```sql
explain select * from emp;
```

Extra：包含额外的信息。

```sql
--using filesort:说明mysql无法利用索引进行排序，只能利用排序算法进行排序，会消耗额外的位置
explain select * from emp order by sal;

--using temporary:建立临时表来保存中间结果，查询完成之后把临时表删除
explain select ename,count(*) from emp where deptno = 10 group by ename;

--using index:这个表示当前的查询时覆盖索引的，直接从索引中读取数据，而不用访问数据表。如果同时出现using where 表名索引被用来执行索引键值的查找，如果没有，表面索引被用来读取数据，而不是真的查找
explain select deptno,count(*) from emp group by deptno limit 10;

--using where:使用where进行条件过滤
explain select * from t_user where id = 1;

--using join buffer:使用连接缓存，情况没有模拟出来

--impossible where：where语句的结果总是false
explain select * from emp where empno = 7469;
```

## 三、索引原理及优化

### 1、索引的原理，索引为什么选择 B+ 树

#### 【1】哈希表和二叉树

哈希表结构只有精确匹配索引所有列的查询才有效，所以只适用于 memory

二叉树结构会造成对应的树节点过深，变成了链表，且每个节点都是一次 IO，越深意味着 IO 次数越多，IO 会造成瓶颈，所以不合适

<img src="_media/database/SQL调优-MySQL/二叉树.png" alt="image-20210308141329196"    />

二叉搜索树（BST）结构，内部有排序操作，可以进行二分查找，可能出现一边长一边短的问题

#### 【2】平衡树

平衡树（AVL）结构，也是一个二叉树，有左旋右旋的概念，需要要保证当前最短子树和最长子树高度差不能超过 1，如果超过 1 那么在严格意义上来讲就不是一个平衡树了，在向树中插入数据时，插入的越多，就会有 1 - n次的旋转，这个旋转过程是比较浪费性能的，所以其插入删除效率极低，查询效率是比较高的，且数依然会较深

<img src="_media/database/SQL调优-MySQL/平衡树.png" alt="image-20210308141617871"    />

#### 【3】红黑树

红黑树结构，最长子树只要不超过最短子树的两倍即可，通过内部的旋转 + 变色，使插入和查询达到一个平衡值，减少了旋转次数，而变色条件是，任何一个单分支里面不能有两个连续的红色节点，且从根到任意节点要保证所有数据路里面黑色节点数保持一致，是 AVL 的变种，牺牲查询性能来满足插入性能的提升

<img src="_media/database/SQL调优-MySQL/红黑树.png" alt="image-20210308142154434"    />

#### 【4】B 树结构，多叉树

其特点是：
1. 所有键值都分布在整棵树中
2. 搜索有可能在非叶子节点结束，在关键字全集内做一次查找，性能逼近二叉树
3. 每个节点最多拥有 m 个子树
4. 根节点至少有两个子树
5. 分支节点至少拥有 m/2 棵子树（除根节点和叶子节点外都是分支节点）
6. 所有叶子节点都在同一层，每个节点最多可以有 m-1 个 key，并且以升序排序

<img src="_media/database/SQL调优-MySQL/B树.png"/>

上图说明：
1. 每个节点占用一个磁盘块，一个节点上有两个升序排序的关键字和三个指向子树根节点的指针，指针存储的是子节点所在磁盘块的地址。
2. 两个关键词划分成的三个范围域对应三个指针指向的子树的数据的范围域。
3. 以根节点为例，关键字为16和34，P1指针指向的子树的数据范围为小于16，P2指针指向的子树的数据范围为16~34，P3指针指向的子树的数据范围为大于 34。

查找关健字过程：
1. 根据根节点找到磁盘块1，读入内存。【磁盘I/O操作第1次】
2. 比较关健字28在区间( 16,34 )，找到磁盘块1的指针P2。
3. 根据P2指针找到磁盘块3，读入内存。【磁盘I/O操作第2次】
4. 比较关健字28在区间（25,31 )，找到磁盘块3的指针P2。
5. 根据P2指针找到磁盘块8，读入内存。【磁盘I/O操作第3次】
6. 在磁盘块8中的关健宁列表中找到关健字28。

缺点：
1. 每个节点都有 key，同时也包含 data，而每个页存储空间是有限的，如果 data 比较大的话会导致每个节点存储的 key 数量变小
2. 当存储的数据量很大的时候会导致深度较大，增大查询时磁盘 IO 次数，进而影响查询性能

#### 【5】B+Tree 是在 BTree 的基础之上做的一种优化

变化如下：
1. B+Tree每个节点可以包含更多的节点，这个做的原因有两个：
   1. 第一个原因是为了降低树的亮度，
   2. 第二个原因是持数据范围变为多个区间，区间越多，数据检索越快
2. 非叶子节点存 key ，叶子节点存储 key 和数据
3. 叶子节点两两指针相互连接(符合磁盘的预读特性〕，顺序查询性能更高

<img src="_media/database/SQL调优-MySQL/B+树.png"/>

注意一：
1. 在B+Tree上有两个头指针，一个指向根节点，另一个指向关键字最小的叶子节点，而且所有叶子节点（即数据节点)之间是一种链式环结构。
2. 因此可以对B+Tree进行两种查找运算︰一种是对于主键的范围查找和分页查找，另一种是从根节点开始，进行随机查找。

InnoDB 存储引擎中索引和数据是放在一起的，且叶子节点中就是实际的一整行数据

![image-20210308162359327](_media/database/SQL调优-MySQL/B+树2.png"/>

注意二：
1. InnoDB是通过B+Tree结构对主键创建索引，然后叶子节点中存储记录，如果没有主键，那么会选择唯一键，如果没有唯一健，那么会生成—个6位的row_id来作为主健
2. 如果创建索引的键是其他字段，那么在叶子节点中存储的是该记录的主键，然后再通过主键索引找到对应的记录，叫做回表

MyISAM 存储引擎中，索引和数据放在两个文件中，且叶子节点中放的是数据所在的文件的地址

<img src="_media/database/SQL调优-MySQL/B+树myisam.png"/>

### 2、索引基本知识

1. 概述：

   1. 官网文档：https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html
   2. 索引使用 B+ Tree 实现的，官网中写的是 B-Tree
   3. 索引也是数据

2. 索引的优点

   1. 大大减少了服务器需要扫描的数据量
   2. 帮助服务器避免排序和临时表
   3. 将随机 io 变成顺序 io 

3. 索引的用处

   1. 快速查找匹配WHERE子句的行
   2. 从 consideration 中消除行，如果可以在多个索引之间进行选择，mysql 通常会使用找到最少行的索引
   3. 如果表具有多列索引，则优化器可以使用索引的任何最左前缀来查找行
   4. 当有表连接的时候，从其他表检索行数据
   5. 查找特定索引列的 min 或 max 值
   6. 如果排序或分组时在可用索引的最左前缀上完成的，则对表进行排序和分组
   7. 在某些情况下，可以优化查询以检索值而无需查询数据行

4. 索引的分类

   1. 主键索引
   2. 唯一索引：数据库会默认给唯一键建索引，并非主键，主键叫唯一且非空
   3. 普通索引：普通的列建索引
   4. 全文索引：一般在 varchar、char、text 中建全文索引，用的很少
   5. 组合索引：将两个经常查询的列组合一起建立索引
   
5. 面试技术名词

   ![image-20210308170309721](_media/database/SQL调优-MySQL/名词.png"/>

   1. 回表
      - 在为普通列（比如 name 列）创建索引时，其最后叶子节点中存放的并不是整行的数据，而是放的主键
      - 当按照 name 列进行查询索引时，首先会根据 name 列的 B+Tree 取到主键
      - 再根据主键去查找主键的  B+Tree ，在从这个 B+Tree 中取出整行数据
      - 比如 select ... where id in ....  ； select * from emp whrer name = ma
   2. 覆盖索引
      - 回表时遍历了两次  B+Tree ，产生了两次 IO，当有一条 SQL 语句依据 ID 进行查询时，因为 ID 在 name 列   B+Tree 中已经存在，所以就不需要再去主键查询一次  B+Tree 了，回表就不存在了，这个过程叫覆盖索引
      - 比如 select id from emp whrer name = ma
      - 执行计划中：Extra = Useing Inde
   3. 最左匹配
      - 当有两个列 name、age 经常查询，那么就根据这两个列来创建索引
      - 当 select * from emp where name =? and age = ? ，是先匹配 name 再匹配 age
      - 当 select * from emp where age = ? ，索引匹配失效
      - 优化案例：
        - 以下三条 sql 应如何建立索引？
          - select * from emp where name =? and age = ?
          - select * from emp where age = ?
          - select * from emp where name = ?
        - 方案一：建立 name、age 组合索引，再单独建 age 索引
        - 方案二：建立 age、name 组合索引，再单独建 name 索引
        - 分析：应选取方案一，因为索引也是数据，也会占中磁盘空间，方案一中 age 占用的磁盘空间更小
   4. 索引下推
      - 老版本数据库中，（name,age）索引，匹配数据时，先将所有 name 的值从存储引擎中取出，之后在 server 层过滤 age
      - 新版本数据库中，在从存储引擎中获取组合数据，在取 name 值时，直接将 age 过滤了，也就是在 server 层不需要再做 age 的判断了，这意味着从 server 层读取 IO 数据时 IO 量少了，效率提升

6. 索引采用的数据结构

   1. 哈希表
   2. B+树

7. 索引匹配方式

   ```sql
   create table staffs(
   
       id int primary key auto_increment,
   
       name varchar(24) not null default '' comment '姓名',
   
       age int not null default 0 comment '年龄',
   
       pos varchar(20) not null default '' comment '职位',
   
       add_time timestamp not null default current_timestamp comment '入职时间'
   
     ) charset utf8 comment '员工记录表';
   
   -----------alter table staffs add index idx_nap(name, age, pos); 创建组合索引
   ```

   1. 全值匹配：全值匹配指的是和索引中的所有列进行匹配

      ```sql
      explain select * from staffs where name = 'July' and age = '23' and pos = 'dev';
      ```

   2. 匹配最左前缀：只匹配前面的几列

      ```sql
      explain select * from staffs where name = 'July' and age = '23';
      explain select * from staffs where name = 'July';
      ```

   3. 匹配列前缀：可以匹配某一列的值的开头部分

      ```sql
      explain select * from staffs where name like 'J%';
      explain select * from staffs where name like '%y';
      ```

   4. 匹配范围值：可以查找某一个范围的数据

      ```sql
      explain select * from staffs where name > 'Mary';
      ```

   5. 精确匹配某一列并范围匹配另外一列：可以查询第一列的全部和第二列的部分

      ```sql
      explain select * from staffs where name = 'July' and age > 25;
      ```

   6. 只访问索引的查询：查询的时候只需要访问索引，不需要访问数据行，本质上就是覆盖索引

      ```sql
      explain select name,age,pos from staffs where name = 'July' and age = 25 and pos = 'dev';
      ```

### 3、哈希索引

1. 基于哈希表的实现，只有精确匹配索引所有列的查询才有效
2. 在 mysql 中，只有 memory 的存储引擎显式支持哈希索引。（memory 不能持久化）
   - 当有大量常量值不做修改，但经常取这些常量值，可以将这些数据放在 memory
3. 哈希索引自身只需存储对应的 hash 值，所以索引的结构十分紧凑，这让哈希索引查找的速度非常快
4. 哈希索引的限制
   1. 哈希索引只包含哈希值和行指针，而不存储字段值，索引不能使用索引中的值来避免读取行
   2. 哈希索引数据并不是按照索引值顺序存储的，所以无法进行排序
   3. 哈希索引不支持部分列匹配查找，哈希索引是使用索引列的全部内容来计算哈希值
   4. 哈希索引支持等值比较查询，也不支持任何范围查询
   5. 访问哈希索引的数据非常快，除非有很多哈希冲突，当出现哈希冲突的时候，存储引擎必须遍历链表中的所有行指针，逐行进行比较，直到找到所有符合条件的行
   6. 哈希冲突比较多的话，维护的代价也会很高
5. 避免哈希冲突：
   1. 开放寻址法：核心思想是，如果出现了散列冲突，我们就重新探测一个空闲位置，将其插入。
      1. 线性探测：我们就从当前位置开始，依次往后查找，看是否有空闲位置，直到找到为止。
         1. 查找的时候，一旦我们通过线性探测方法，找到一个空闲位置，我们就可以认定散列表中不存在这个数据。但是，如果这个空闲位置是我们后来删除的，就会导致原来的查找算法失效。本来存在的数据，会被认定为不存在。
         2. 我们可以将删除的元素，特殊标记为 deleted。当线性探测查找的时候，遇到标记为 deleted 的空间，并不是停下来，而是继续往下探测
      2. 二次探测：探测步长不同，二次探测探测的步长就变成了原来的“二次方”，也就是说，它探测的下标序列就是 hash(key)+0，hash(key)+12，hash(key)+22
      3. 双重散列:有冲突就找第二个散列函数，再有再加
   2. 链表法：是一种更加常用的散列冲突解决办法，相比开放寻址法，它要简单很多。在散列表中，每个“桶（bucket）”或者“槽（slot）”会对应一条链表，所有散列值相同的元素我们都放到相同槽位对应的链表中。
   3. 优缺点：
      1. 开放寻址法：都在数组里，序列化简单，冲突代价高，删除麻烦，装载因子不能太大，浪费内存空间
      2. 链表：内存利用率高，存储小对象的时候，你还要存储指针，比较消耗内存。我总结一下，基于链表的散列冲突处理方法比较适合存储大对象、大数据量的散列表，而且，比起开放寻址法，它更加灵活，支持更多的优化策略，比如用红黑树代替链表。
6. 案例
   - 当需要存储大量的URL，并且根据 URL 进行搜索查找，如果使用 B+ 树，存储的内容就会很大
     - select id from url where url=""
   - 也可以利用将url使用CRC32做哈希，可以使用以下查询方式：
     - select id fom url where url="" and url_crc=CRC32("")
     - 此查询性能较高原因是使用体积很小的索引来完成查找

### 4、组合索引

1. 当包含多个列作为索引，需要注意的是正确的顺序依赖于该索引的查询，同时需要考虑如何更好的满足排序和分组的需要
2. 案例，建立组合索引 a,b,c，不同SQL语句使用索引情况
   - ![image-20210308135741949](_media/database/SQL调优-MySQL/3、索引原理及优化\组合索引.png"/>

### 5、聚簇索引与非聚簇索引

1. 聚簇索引：不是单独的索引类型，而是一种数据存储方式，指的是数据行跟相邻的键值紧凑的存储在一起
   1. 优点
      1. 可以把相关数据保存在一起
      2. 数据访问更快，因为索引和数据保存在同一个树中
      3. 使用覆盖索引扫描的查询可以直接使用页节点中的主键值
   2. 缺点
      1. 聚簇数据最大限度地提高了IO密集型应用的性能，如果数据全部在内存，那么聚簇索引就没有什么优势
      2. 插入速度严重依赖于插入顺序，按照主键的顺序插入是最快的方式
      3. 更新聚簇索引列的代价很高，因为会强制将每个被更新的行移动到新的位置
      4. 基于聚簇索引的表在插入新行，或者主键被更新导致需要移动行的时候，可能面临页分裂的问题
      5. 聚簇索引可能导致全表扫描变慢，尤其是行比较稀疏，或者由于页分裂导致数据存储不连续的时候
2. 非聚簇索引：数据文件跟索引文件分开存放

### 6、覆盖索引

1. 基本介绍

   1. 如果一个索引包含所有需要查询的字段的值，我们称之为覆盖索引
   2. 不是所有类型的索引都可以称为覆盖索引，覆盖索引必须要存储索引列的值
   3. 不同的存储实现覆盖索引的方式不同，不是所有的引擎都支持覆盖索引，memory不支持覆盖索引

2. 优势

   1. 索引条目通常远小于数据行大小，如果只需要读取索引，那么mysql就会极大的较少数据访问量
   2. 因为索引是按照列值顺序存储的，所以对于IO密集型的范围查询会比随机从磁盘读取每一行数据的IO要少的多
   3. 一些存储引擎如MYISAM在内存中只缓存索引，数据则依赖于操作系统来缓存，因此要访问数据需要一次系统调用，这可能会导致严重的性能问题
   4. 由于INNODB的聚簇索引，覆盖索引对INNODB表特别有用

3. 案例演示

   1. 当发起一个被索引覆盖的查询时，在explain的extra列可以看到using index的信息，此时就使用了覆盖索引

      ```sql
      mysql> explain select store_id,film_id from inventory\G
      *************************** 1. row ***************************
                 id: 1
        select_type: SIMPLE
              table: inventory
         partitions: NULL
               type: index
      possible_keys: NULL
                key: idx_store_id_film_id
            key_len: 3
                ref: NULL
               rows: 4581
           filtered: 100.00
              Extra: Using index
      1 row in set, 1 warning (0.01 sec)
      ```

   2. 在大多数存储引擎中，覆盖索引只能覆盖那些只访问索引中部分列的查询。不过，可以进一步的进行优化，可以使用innodb的二级索引来覆盖查询。

      - 例如：actor使用innodb存储引擎，并在last_name字段又二级索引，虽然该索引的列不包括主键actor_id，但也能够用于对actor_id做覆盖查询

        ```sql
        mysql> explain select actor_id,last_name from actor where last_name='HOPPER'\G
        *************************** 1. row ***************************
                   id: 1
          select_type: SIMPLE
                table: actor
           partitions: NULL
                 type: ref
        possible_keys: idx_actor_last_name
                  key: idx_actor_last_name
              key_len: 137
                  ref: const
                 rows: 2
             filtered: 100.00
                Extra: Using index
        1 row in set, 1 warning (0.00 sec)
        ```



## 四、索引优化细节

### 1、表达式计算

- 当使用索引列进行查询的时候尽量不要使用表达式，把计算放到业务层而不是数据库层

```sql
select actor_id from actor where actor_id=4;
select actor_id from actor where actor_id+1=5;
```

### 2、使用主键索

1. 引尽量使用主键查询，而不是其他索引，因此主键查询不会触发回表查询

### 3、使用前缀索引

1. 有时候需要索引很长的字符串，这会让索引变的大且慢，通常情况下可以使用某个列开始的部分字符串，这样大大的节约索引空间，从而提高索引效率，但这会降低索引的选择性，索引的选择性是指不重复的索引值和数据表记录总数的比值，范围从 1/#T 到 1 之间。索引的选择性越高则查询效率越高，因为选择性更高的索引可以让mysql 在查找的时候过滤掉更多的行。

2. 一般情况下某个列前缀的选择性也是足够高的，足以满足查询的性能，但是对应 BLOB、TEXT、VARCHAR 类型的列，必须要使用前缀索引，因为mysql不允许索引这些列的完整长度，使用该方法的诀窍在于要选择足够长的前缀以保证较高的选择性，通过又不能太长。

3. 案例：

   ```sql
   --创建数据表
   create table citydemo(city varchar(50) not null);
   insert into citydemo(city) select city from city;
   
   --重复执行5次下面的sql语句
   insert into citydemo(city) select city from citydemo;
   
   --更新城市表的名称
   update citydemo set city=(select city from city order by rand() limit 1);
   
   --查找最常见的城市列表，发现每个值都出现45-65次，
   select count(*) as cnt,city from citydemo group by city order by cnt desc limit 10;
   
   --查找最频繁出现的城市前缀，先从3个前缀字母开始，发现比原来出现的次数更多，可以分别截取多个字符查看城市出现的次数
   select count(*) as cnt,left(city,3) as pref from citydemo group by pref order by cnt desc limit 10;
   select count(*) as cnt,left(city,7) as pref from citydemo group by pref order by cnt desc limit 10;
   --此时前缀的选择性接近于完整列的选择性
   
   --还可以通过另外一种方式来计算完整列的选择性，可以看到当前缀长度到达7之后，再增加前缀长度，选择性提升的幅度已经很小了
   select count(distinct left(city,3))/count(*) as sel3,
   count(distinct left(city,4))/count(*) as sel4,
   count(distinct left(city,5))/count(*) as sel5,
   count(distinct left(city,6))/count(*) as sel6,
   count(distinct left(city,7))/count(*) as sel7,
   count(distinct left(city,8))/count(*) as sel8 
   from citydemo;
   
   --计算完成之后可以创建前缀索引
   alter table citydemo add key(city(7));
   
   --注意：前缀索引是一种能使索引更小更快的有效方法，但是也包含缺点：mysql无法使用前缀索引做order by 和 group by。 
   ```

4. 案例效果

   - <img src="_media/database/SQL调优-MySQL/3、索引原理及优化\案例效果.png" alt="image-20210310214852578"    />

5. 查看表中的索引

   - ![image-20210310215149664](_media/database/SQL调优-MySQL/3、索引原理及优化\查看表中索引.png"/>

6. Cardinality 基数解释

   1. 基数表示列中去重后的唯一值，比如 （a,b,c,d）的基数为 4，（a,b,c,d,a）的基数也是 4
   2. 基数是一个近似值，一般需要根据这个值来判断与哪个值进行关联
   3. 表关联时，基数值越小，检索关联的效率越高
   4. 其算法使用的是 HyperLogLog ，经常在数据库中使用来计算基数值（Distinct Value）
   5. 在 OLAP 项目中经常使用，例如电商网站的历史数据，需要统计分析以应对新一年的业务及广告投放，这时因有大量数据，以及各类表关联查询，所以需要统计基数

### 4、使用索引扫描来排序

- mysql 有两种方式可以生成有序的结果:通过排序操作或者按索引顺序扫描，如果 explain 出来的 type 列的值为 index，则说明 mysql 使用了索引扫描来做排序

- 扫描索引本身是很快的，因为只需要从一条索引记录移动到紧接着的下一条记录。但如果索引不能覆盖查询所需的全部列，那么就不得不每扫描一条索引记录就得回表查询一次对应的行，这基本都是随机 IO，因此按索引顺序读取数据的速度通常要比顺序地全表扫描慢

- mysql 可以使用同一个索引即满足排序，又用于查找行，如果可能的话，设计索引时应该尽可能地同时满足两种任务。

- 只有当索引的列顺序和 order by 子句的顺序完全一致，并且所有列的排序方式都一样时，mysql 才能够使用索引来对结果进行排序，如果查询需要关联多张表，则只有当orderby子句引用的字段全部为第一张表时，才能使用索引做排序。order by 子句和查找型查询的限制是一样的，需要满足索引的最左前缀的要求，否则，mysql 都需要执行顺序操作，而无法利用索引排序

- ```sql
  --sakila数据库中renta1表在rental_date,inventory_id,customer_id上有rental_date的索引
  --使用rental_date索引为下面的查询做排序
  explain select rental_id,staff_id from rental where rental_date='2005-05-25' order by inventory_id ,customer_id\G;
  -- mysql 5.7
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: ref
  possible_keys: rental_date
            key: rental_date
        key_len: 5
            ref: const
           rows: 1
       filtered: 100.00
          Extra: Using index condition  -- 使用索引中的功能
  1 row in set, 1 warning (0.00 sec)
  -- mysql 8.0
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: ref
  possible_keys: rental_date
            key: rental_date
        key_len: 5
            ref: const
           rows: 1
       filtered: 100.00
          Extra: NULL
  1 row in set, 1 warning (0.00 sec)
  ```

  ```sql
  --order by子句不满足索引的最左前缀的要求，也可以用于查询排序，这是因为所以你的第一列被指定为一个常数
  --该查询为索引的第一列提供了常量条件，而使用第二列进行排序，将两个列组合在一起，就形成了索引的最左前缀
  explain select rental_id,staff_id from rental where rental_date='2005-05-25' order by inventory_id desc\G;
  -- mysql 5.7
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: ref
  possible_keys: rental_date
            key: rental_date
        key_len: 5
            ref: const
           rows: 1
       filtered: 100.00
          Extra: Using where  -- 使用了where条件，使用的是左前缀
  1 row in set, 1 warning (0.01 sec)
  -- mysql 8.0 中对返向索引进行了优化
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: ref
  possible_keys: rental_date
            key: rental_date
        key_len: 5
            ref: const
           rows: 1
       filtered: 100.00
          Extra: Backward index scan -- 使用的是返向索引
  1 row in set, 1 warning (0.00 sec)
  ```

  ```sql
  --下面的查询在 mysql 5.7 中不会利用索引，在mysql 8.0 中 order by 后面的条件符合复合索引条件
  explain select rental_id,staff_id from rental where rental_date> '2005-05-25' order by rental_date ,inventory_id\G;
  -- mysql 5.7
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: ALL
  possible_keys: rental_date
            key: NULL
        key_len: NULL
            ref: NULL
           rows: 16005
       filtered: 50.00
          Extra: Using where; Using filesort -- 使用的全文件扫描
  1 row in set, 1 warning (0.00 sec)
  -- mysql 8.0
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: range
  possible_keys: rental_date
            key: rental_date
        key_len: 5
            ref: NULL
           rows: 1
       filtered: 100.00
          Extra: Using index condition  -- 使用索引
  1 row in set, 1 warning (0.00 sec)
  ```

  ```sql
  -- 该查询中使用了两种不同的排序方向，索引中默认排序是升序，以下不会使用索引，应保持同方向
  explain select rental_id,staff_id from rental where rental_date='2005-05-25' order by inventory_id asc ,customer_id desc\G;
  -- mysql 5.7
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: ALL
  possible_keys: rental_date
            key: NULL
        key_len: NULL
            ref: NULL
           rows: 16005
       filtered: 50.00
          Extra: Using where; Using filesort
  1 row in set, 1 warning (0.00 sec)
  -- mysql 8.0
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: range
  possible_keys: rental_date
            key: rental_date
        key_len: 5
            ref: NULL
           rows: 1
       filtered: 100.00
          Extra: Using index condition; Using filesort
  1 row in set, 1 warning (0.00 sec)
  ```

  ```sql
  -- 该查询中引入了一个不在索引中的列
  explain select rental_id,staff_id from rental where rental_date>'2005-05-25' order by inventory_id desc ,staff_id\G;
  -- mysql 5.7
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: ALL
  possible_keys: rental_date
            key: NULL
        key_len: NULL
            ref: NULL
           rows: 16005
       filtered: 50.00
          Extra: Using where; Using filesort
  1 row in set, 1 warning (0.00 sec)
  -- mysql 8.0
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: rental
     partitions: NULL
           type: range
  possible_keys: rental_date
            key: rental_date
        key_len: 5
            ref: NULL
           rows: 1
       filtered: 100.00
          Extra: Using index condition; Using filesort
  1 row in set, 1 warning (0.00 sec)
  ```

  

### 5、推荐使用 in

- union all、in、or都能够使用索引，但是推荐使用 in

  ```sql
  explain select * from actor where actor_id = 1 union all select * from actor where actor_id = 2;
  explain select * from actor where actor_id in (1,2);
  explain select * from actor where actor_id = 1 or actor_id =2;
  
  -- exist
  select * from emp e where exists(select deptno from dept d where (deptno = 20 or deptno = 30) and e.deptno = d.deptno);
  -- and 的优先级比 or 高，所以需要括起来
  -- 多表查询时多用 join 
  -- 当 a 表条件来自 b 表，而 b 表字段不在查询显示中凸显，使用 exist 且速度比较快
  ```

### 6、范围列可以用到索引

1. 范围条件是：<、>
2. 范围列可以用到索引，但是范围列后面的列无法用到索引，索引最多用于一个范围列

### 7、强制类型转换会全表扫描

1. 不会触发索引

   ```sql
   explain select * from user where phone=13800001234;
   ```

2. 触发索引

   ```sql
   explain select * from user where phone='13800001234';
   ```

### 8、不宜建立索引

- 更新十分频繁，数据区分度不高的字段上不宜建立索引
  1. 更新会变更B+树，更新频繁的字段建议索引会大大降低数据库性能
  2. 类似于性别这类区分不大的属性，建立索引是没有意义的，不能有效的过滤数据，
  3. 一般区分度在80%以上的时候就可以建立索引，区分度可以使用 count(distinct(列名))/count(*) 来计算

### 9、其他细节

1. 创建索引的列，不允许为null，可能会得到不符合预期的结果
2. 当需要进行表连接的时候，最好不要超过三张表，因为需要 join 的字段，数据类型必须一致，join 在 mysql 中的实现有很多种方式，其使用的 nested-loop 嵌套循环算法，就是将 A 表中的每一行都与 B 表中的全部做匹配，其分为以下几种方式：
   - Simple Nested-Loop Join 效率较低，一般不使用
     - ![]_media/database/SQL调优-MySQL/join原理1.png"/>
   - Index Nested-Loop Join， 使用索引
     - ![]_media/database/SQL调优-MySQL/join原理2.png"/>
   - Block Nested-Loop Join ，当索引方式效率不高时，可以把对应关联列的数据放到内存中
     - ![image-20210319143441789]_media/database/SQL调优-MySQL/block join.png"/>
     - show variables like '%join_buffer%'  查看 join 内存大小
3. 能使用 limit 的时候尽量使用 limit
   - limit 不能叫分页，应该称为限制输出
   - 如果明确知道只有一条结果返回，limit 1 能够提高效率，因为 limit 是向下遍历的过程，如果直接指定 limit 1 那么就不会在向下做判断，省了一次判断的过程
4. 单表索引建议控制在5个以内
   - 索引也是数据，会占用很多存储空间，索引越多代表着存储空间越大，IO 就大了，同时也不一定起到优化效果
5. 单索引字段数不允许超过5个（组合索引）
   - 组合索引中有最左匹配原则，如果索引列过多，后面的列查询的时候，会先将前面的索引列匹配完整，会浪费存储空间
6. 创建索引的时候应该避免以下错误概念
   1. 索引越多越好
   2. 过早优化，在不了解系统的情况下进行优化

## 五、索引监控及索引使用案例

### 1、索引监控

1. show status like 'Handler_read%';
2. 参数解释
   1. Handler_read_first：读取索引第一个条目的次数
   2. Handler_read_key：通过index获取数据的次数
   3. Handler_read_last：读取索引最后一个条目的次数
   4. Handler_read_next：通过索引读取下一条数据的次数
   5. Handler_read_prev：通过索引读取上一条数据的次数
   6. Handler_read_rnd：从固定位置读取数据的次数
   7. Handler_read_rnd_next：从数据节点读取下一条数据的次数

### 2、索引使用案例

1. 预先准备好数据

   ```sql
   SET FOREIGN_KEY_CHECKS=0;
   DROP TABLE IF EXISTS `itdragon_order_list`;
   CREATE TABLE `itdragon_order_list` (
     `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT '主键id，默认自增长',
     `transaction_id` varchar(150) DEFAULT NULL COMMENT '交易号',
     `gross` double DEFAULT NULL COMMENT '毛收入(RMB)',
     `net` double DEFAULT NULL COMMENT '净收入(RMB)',
     `stock_id` int(11) DEFAULT NULL COMMENT '发货仓库',
     `order_status` int(11) DEFAULT NULL COMMENT '订单状态',
     `descript` varchar(255) DEFAULT NULL COMMENT '客服备注',
     `finance_descript` varchar(255) DEFAULT NULL COMMENT '财务备注',
     `create_type` varchar(100) DEFAULT NULL COMMENT '创建类型',
     `order_level` int(11) DEFAULT NULL COMMENT '订单级别',
     `input_user` varchar(20) DEFAULT NULL COMMENT '录入人',
     `input_date` varchar(20) DEFAULT NULL COMMENT '录入时间',
     PRIMARY KEY (`id`)
   ) ENGINE=InnoDB AUTO_INCREMENT=10003 DEFAULT CHARSET=utf8;
   
   INSERT INTO itdragon_order_list VALUES ('10000', '81X97310V32236260E', '6.6', '6.13', '1', '10', 'ok', 'ok', 'auto', '1', 'itdragon', '2017-08-28 17:01:49');
   INSERT INTO itdragon_order_list VALUES ('10001', '61525478BB371361Q', '18.88', '18.79', '1', '10', 'ok', 'ok', 'auto', '1', 'itdragon', '2017-08-18 17:01:50');
   INSERT INTO itdragon_order_list VALUES ('10002', '5RT64180WE555861V', '20.18', '20.17', '1', '10', 'ok', 'ok', 'auto', '1', 'itdragon', '2017-09-08 17:01:49');
   ```

2. 第一个案例

   ```sql
   select * from itdragon_order_list where transaction_id = "81X97310V32236260E";
   --通过查看执行计划发现type=all,需要进行全表扫描
   explain select * from itdragon_order_list where transaction_id = "81X97310V32236260E";
   
   --优化一、为transaction_id创建唯一索引
    create unique index idx_order_transaID on itdragon_order_list (transaction_id);
   --当创建索引之后，唯一索引对应的type是const，通过索引一次就可以找到结果，普通索引对应的type是ref，表示非唯一性索引赛秒，找到值还要进行扫描，直到将索引文件扫描完为止，显而易见，const的性能要高于ref
    explain select * from itdragon_order_list where transaction_id = "81X97310V32236260E";
    
    --优化二、使用覆盖索引，查询的结果变成 transaction_id,当extra出现using index,表示使用了覆盖索引
    explain select transaction_id from itdragon_order_list where transaction_id = "81X97310V32236260E";
   ```

3. 第二个案例

   ```sql
   --创建复合索引
   create index idx_order_levelDate on itdragon_order_list (order_level,input_date);
   
   --创建索引之后发现跟没有创建索引一样，都是全表扫描，都是文件排序
   explain select * from itdragon_order_list order by order_level,input_date;
   
   --可以使用force index强制指定索引
   explain select * from itdragon_order_list force index(idx_order_levelDate) order by order_level,input_date;
   --其实给订单排序意义不大，给订单级别添加索引意义也不大，因此可以先确定order_level的值，然后再给input_date排序
   explain select * from itdragon_order_list where order_level=3 order by input_date;
   ```

## 六、查询优化分析

在编写快速的查询之前，需要清楚一点，真正重要的是响应时间，而且要知道在整个SQL语句的执行过程中每个步骤都花费了多长时间，要知道哪些步骤是拖垮执行效率的关键步骤，想要做到这点，必须要知道查询的生命周期，然后进行优化，不同的应用场景有不同的优化方式，不要一概而论，具体情况具体分析。

### 1、查询慢的原因

1. 网络
2. CPU
3. IO
4. 上下文切换
5. 系统调用
6. 生成统计信息
7. 锁等待时间

### 2、优化数据访问

1. 查询性能低下的主要原因是访问的数据太多，某些查询不可避免的需要筛选大量的数据，我们可以通过减少访问数据量的方式进行优化

   1. 确认应用程序是否在检索大量超过需要的数据
   2. 确认mysql服务器层是否在分析大量超过需要的数据行

2. 是否向数据库请求了不需要的数据

   1. 查询不需要的记录

      - 我们常常会误以为 mysql 会只返回需要的数据，实际上 mysql 却是先返回全部结果再进行计算，在日常的开发习惯中，经常是先用select语句查询大量的结果，然后获取前面的N行后关闭结果集。
      - 优化方式是在查询后面添加limit

   2. 多表关联时返回全部列

      ```sql
      select * from actor inner join film_actor using(actor_id) inner join film using(film_id) where film.title='Academy Dinosaur';
      
      select actor.* from actor...;
      ```

   3. 总是取出全部列

      - 在公司的企业需求中，禁止使用select *,虽然这种方式能够简化开发，但是会影响查询的性能，所以尽量不要使用

   4. 重复查询相同的数据

      - 如果需要不断的重复执行相同的查询，且每次返回完全相同的数据，因此，基于这样的应用场景，我们可以将这部分数据缓存起来，这样的话能够提高查询效率

### 3、执行过程的优化  —— 查询缓存

- 在解析一个查询语句之前：
  - 如果查询缓存是打开的，那么 mysql 会优先检查这个查询是否命中查询缓存中的数据；
  - 如果查询恰好命中了查询缓存，那么会在返回结果之前会检查用户权限；
  - 如果权限没有问题，那么 mysql 会跳过所有的阶段，就直接从缓存中拿到结果并返回给客户端

### 4、执行过程的优化  —— 查询优化处理

1. #### 语法解析器和预处理

   - mysql 通过关键字将 SQL 语句进行解析，并生成一棵解析树，mysql 解析器将使用 mysql 语法规则验证和解析查询
     - 例如验证使用使用了错误的关键字或者顺序是否正确等等，预处理器会进一步检查解析树是否合法
       - 例如表名和列名是否存在，是否有歧义，还会验证权限等等

2. #### 查询优化器

   1. 当语法树没有问题之后，相应的要由优化器将其转成执行计划，一条查询语句可以使用非常多的执行方式，最后都可以得到对应的结果，但是不同的执行方式带来的效率是不同的，优化器的最主要目的就是要选择最有效的执行计划

      - mysql使用的是基于成本的优化器，在优化的时候会尝试预测一个查询使用某种查询计划时候的成本，并选择其中成本最小的一个
      
   2. #### select count(*) from film_actor;     show status like 'last_query_cost';
   
      - <img src="_media/database/SQL调优-MySQL/6、查询优化分析\成本.png" alt="image-20210320214201671"    />
   - 可以看到这条查询语句大概需要做1104个数据页才能找到对应的数据，这是经过一系列的统计信息计算来的
        - 每个表或者索引的页面个数
     - 索引的基数
        - 索引和数据行的长度
        - 索引的分布情况
   
   3. #### 在很多情况下mysql会选择错误的执行计划，原因如下：
   
      1. 统计信息不准确：
         - InnoDB因为其mvcc的架构，并不能维护一个数据表的行数的精确统计信息
      2. 执行计划的成本估算不等同于实际执行的成本
         - 有时候某个执行计划虽然需要读取更多的页面，但是他的成本却更小，因为如果这些页面都是顺序读或者这些页面都已经在内存中的话，那么它的访问成本将很小，mysql 层面并不知道哪些页面在内存中，哪些在磁盘，所以查询之际执行过程中到底需要多少次 IO 是无法得知的
   3. mysql的最优可能跟你想的不一样
      
         - mysql 的优化是基于成本模型的优化，但是有可能不是最快的优化
   4. mysql不考虑其他并发执行的查询
      5. mysql不会考虑不受其控制的操作成本
         - 执行存储过程或者用户自定义函数的成本
   
4. #### 优化器的优化策略
   
   1. 静态优化：直接对解析树进行分析，并完成优化
      2. 动态优化：动态优化与查询的上下文有关，也可能跟取值、索引对应的行数有关
   3. mysql对查询的静态优化只需要一次，但对动态优化在每次执行时都需要重新评估
   
5. #### 优化器的优化类型
   
   1. 重新定义关联表的顺序
   
      - 数据表的关联并不总是按照在查询中指定的顺序进行，决定关联顺序时优化器很重要的功能
   
   2. 将外连接转化成内连接，内连接获取数据量少于外连接，内连接的效率要高于外连接
   
   3. 使用等价变换规则，mysql可以使用一些等价变化来简化并规划表达式
   
   4. 优化count()，min()，max()
   
      - 索引和列是否可以为空通常可以帮助mysql优化这类表达式：例如，要找到某一列的最小值，只需要查询索引的最左端的记录即可，不需要全文扫描比较
   
   5. 预估并转化为常数表达式，当mysql检测到一个表达式可以转化为常数的时候，就会一直把该表达式作为常数进行处理
   
      - > explain select film.film_id,film_actor.actor_id from film inner join film_actor using(film_id) where film.film_id = 1
   
   6. 索引覆盖扫描，当索引中的列包含所有查询中需要使用的列的时候，可以使用覆盖索引
   
   7. 子查询优化
   
      - mysql在某些情况下可以将子查询转换一种效率更高的形式，从而减少多个查询多次对数据进行访问，例如将经常查询的数据放入到缓存中
   
      8. 等值传播
   
         - 如果两个列的值通过等式关联，那么mysql能够把其中一个列的where条件传递到另一个上：
   
           ```sql
           explain select film.film_id from film inner join film_actor using(film_id
           
           ) where film.film_id > 500;
           
           这里使用film_id字段进行等值关联，film_id这个列不仅适用于film表而且适用于film_actor表
           
           ```
   
      
   
     ```sql
   explain select film.film_id from film inner join film_actor using(film_id) where film.film_id > 500 and film_actor.film_id > 500;
        
     ```
   
   6. #### 关联查询
   
      1. mysql的关联查询很重要，但其实关联查询执行的策略比较简单：
   
         - mysql对任何关联都执行嵌套循环关联操作，即mysql先在一张表中循环取出单条数据，然后再嵌套到下一个表中寻找匹配的行，依次下去，直到找到所有表中匹配的行为止。
         - 然后根据各个表匹配的行，返回查询中需要的各个列。mysql会尝试再最后一个关联表中找到所有匹配的行，如果最后一个关联表无法找到更多的行之后，mysql返回到上一层次关联表，看是否能够找到更多的匹配记录，以此类推迭代执行。整体的思路如此，但是要注意实际的执行过程中有多个变种形式：
   
      2. join 的实现方式原理
   
         1. ![image-20210309182042832]_media/database/SQL调优-MySQL/join原理-1.png"/>
         2. ![image-20210309182142904]_media/database/SQL调优-MySQL/join原理-2.png"/>
         3. ![image-20210309182157603]_media/database/SQL调优-MySQL/join原理-3.png"/>
         - Join Buffer会缓存所有参与查询的列而不是只有Join的列。
            - 可以通过调整join_buffer_size缓存大小
         - join_buffer_size的默认值是256K，join_buffer_size的最大值在MySQL 5.1.22版本前是4G-1，而之后的版本才能在64位操作系统下申请大于4G的Join Buffer空间。
            - 使用Block Nested-Loop Join算法需要开启优化器管理配置的optimizer_switch的设置block_nested_loop为on，默认为开启。
         - show variables like '%optimizer_switch%'
   
   3. 案例演示
   
      - 查看不同的顺序执行方式对查询性能的影响：
   
        **explain select film.film_id,film.title,film.release_year,actor.actor_id,actor.first_name,actor.last_name from film inner join f**
   
        **ilm_actor using(film_id) inner join actor using(actor_id);**
   
        查看执行的成本：
   
        show status like 'last_query_cost'; 
   
        按照自己预想的规定顺序执行：
   
        **explain select straight_join film.film_id,film.title,film.release_year,actor.actor_id,actor.first_name,actor.last_name from fil**
   
        **m inner join film_actor using(film_id) inner join actor using(actor_id);**
   
        查看执行的成本：
   
           show status like 'last_query_cost'; 
   
   7. #### 排序优化
   
      - 无论如何排序都是一个成本很高的操作，所以从性能的角度出发，应该尽可能避免排序或者尽可能避免对大量数据进行排序。
      - 推荐使用利用索引进行排序，但是当不能使用索引的时候，mysql就需要自己进行排序，如果数据量小则再内存中进行，如果数据量大就需要使用磁盘，mysql中称之为filesort。
      - 如果需要排序的数据量小于排序缓冲区(show variables like '%sort_buffer_size%';),mysql使用内存进行快速排序操作，如果内存不够排序，那么mysql就会先将树分块，对每个独立的块使用快速排序进行排序，并将各个块的排序结果存放再磁盘上，然后将各个排好序的块进行合并，最后返回排序结果
      - 排序的算法
        - 两次传输排序
          - 第一次数据读取是将需要排序的字段读取出来，然后进行排序
          - 第二次是将排好序的结果按照需要去读取数据行。这种方式效率比较低
            - 原因是第二次读取数据的时候因为已经排好序，需要去读取所有记录而此时更多的是随机IO，读取数据成本会比较高
          - 两次传输的优势，在排序的时候存储尽可能少的数据，让排序缓冲区可以尽可能多的容纳行数来进行排序操作
        - 单次传输排序
          - 先读取查询所需要的所有列，然后再根据给定列进行排序，最后直接返回排序结果，此方式只需要一次顺序IO读取所有的数据，而无须任何的随机IO，
          - 问题在于查询的列特别多的时候，会占用大量的存储空间，无法存储大量的数据
        - 当需要排序的列的总大小超过max_length_for_sort_data定义的字节，mysql会选择双次排序，反之使用单次排序，当然，用户可以设置此参数的值来选择排序的方式

### 4、优化特定类型的查询

1. #### 优化 count() 查询

   1. count() 是特殊的函数，有两种不同的作用，一种是某个列值的数量，也可以统计行数
   2. myisam 的 count 函数比较快，但是有前提条件的，只有没有任何 where 条件的 count(*) 才是比较快的
   3. 使用近似值
      - 在某些应用场景中，不需要完全精确的值，可以参考使用近似值来代替，比如可以使用 explain 来获取近似的值
      - 其实在很多 OLAP 的应用中，需要计算某一个列值的基数，有一个计算近似值的算法叫 hyperloglog。
   4. 更复杂的优化
      - 一般情况下，count() 需要扫描大量的行才能获取精确的数据，其实很难优化，在实际操作的时候可以考虑使用索引覆盖扫描，或者增加汇总表，或者增加外部缓存系统。

2. #### 优化关联查询

   1. 确保 on 或者 using 子句中的列上有索引，在创建索引的时候就要考虑到关联的顺序
      - 当表 A 和表 B 使用列 C 关联的时候，如果优化器的关联顺序是B、A，那么就不需要再B表的对应列上建上索引，没有用到的索引只会带来额外的负担，一般情况下来说，只需要在关联顺序中的第二个表的相应列上创建索引
   2. 确保任何的groupby和order by中的表达式只涉及到一个表中的列，这样mysql才有可能使用索引来优化这个过程

3. #### 优化子查询：

   - 子查询的优化最重要的优化建议是尽可能使用关联查询代替
   - 子查询的数据会暂时放到临时表中，临时表也是一次 IO
   - join 的临时表是放最终数据的

4. 如果对关联查询做分组，并且是按照查找表中的某个列进行分组，那么可以采用查找表的标识列分组的效率比其他列更高，需要特定情况：当表中数据不重复时使用，但本身无意义，只是 mysql 提供这种方式的语法不会出错

   ```sql
   select actot.first_name,actor.last_name,count(*) from film_actor inner join actor using(actor_id) group by actor.first_name,actor.last_name
   
   select actot.first_name,actor.last_name,count(*) from film_actor inner join actor using(actor_id) group by actor.actor_id
   ```

   

5. #### 优化 limit 分页

   1. 在很多应用场景中我们需要将数据进行分页，一般会使用limit加上偏移量的方法实现，同时加上合适的orderby 的子句，如果这种方式有索引的帮助，效率通常不错，否则的化需要进行大量的文件排序操作，还有一种情况，当偏移量非常大的时候，前面的大部分数据都会被抛弃，这样的代价太高。要优化这种查询的话，要么是在页面中限制分页的数量，要么优化大偏移量的性能

   2. 优化此类查询的最简单的办法就是尽可能地使用覆盖索引，而不是查询所有的列

      - 查看执行计划查看扫描行数

        ```sql
        select film_id,description from film order by title limit 50,5
        explain select film.film_id,film.description from film inner join (select film_id from film order by title limit 50,5) as lim using(film_id);
        ```

6. #### 优化union查询

   1. mysql总是通过创建并填充临时表的方式来执行union查询，因此很多优化策略在union查询中都没法很好的使用。经常需要手工的将where、limit、order by等子句下推到各个子查询中，以便优化器可以充分利用这些条件进行优化
   2. 除非确实需要服务器消除重复的行，否则一定要使用union all，因此没有all关键字，mysql会在查询的时候给临时表加上distinct的关键字，这个操作的代价很高

7. #### 推荐使用用户自定义变量

   - 用户自定义变量是一个容易被遗忘的mysql特性，但是如果能够用好，在某些场景下可以写出非常高效的查询语句，在查询中混合使用过程化和关系话逻辑的时候，自定义变量会非常有用。
   - 用户自定义变量是一个用来存储内容的临时容器，在连接mysql的整个过程中都存在。

   1. 自定义变量的使用

      - > set @one :=1

      - > set @min_actor :=(select min(actor_id) from actor)

      - > set @last_week :=current_date-interval 1 week;

   2. 自定义变量的限制

      1. 无法使用查询缓存
      2. 不能在使用常量或者标识符的地方使用自定义变量，例如表名、列名或者limit子句
      3. 用户自定义变量的生命周期是在一个连接中有效，所以不能用它们来做连接间的通信
      4. 不能显式地声明自定义变量地类型
      5. mysql优化器在某些场景下可能会将这些变量优化掉，这可能导致代码不按预想地方式运行
      6. 赋值符号：=的优先级非常低，所以在使用赋值表达式的时候应该明确的使用括号
      7. 使用未定义变量不会产生任何语法错误

   3. 自定义变量的使用案例

      1. 优化排名语句

         - 在给一个变量赋值的同时使用这个变量

           - > select actor_id,@rownum:=@rownum+1 as rownum from actor limit 10;

         - 查询获取演过最多电影的前10名演员，然后根据出演电影次数做一个排名

           - > select actor_id,count(*) as cnt from film_actor group by actor_id order by cnt desc limit 10;

      2. 避免重新查询刚刚更新的数据

         - 当需要高效的更新一条记录的时间戳，同时希望查询当前记录中存放的时间戳是什么

           - > update t1 set  lastUpdated=now() where id =1;
             > select lastUpdated from t1 where id =1;

           - > update t1 set lastupdated = now() where id = 1 and @now:=now();
             > select @now;

      3. 确定取值的顺序：在赋值和读取变量的时候可能是在查询的不同阶段

         - > set @rownum:=0;
           > select actor_id,@rownum:=@rownum+1 as cnt from actor where @rownum<=1;

           - 因为where和select在查询的不同阶段执行，所以看到查询到两条记录，这不符合预期

         - > set @rownum:=0;
           > select actor_id,@rownum:=@rownum+1 as cnt from actor where @rownum<=1 order by first_name

           - 当引入了orde;r by之后，发现打印出了全部结果，这是因为order by引入了文件排序，而where条件是在文件排序操作之前取值的  

         - 解决这个问题的关键在于让变量的赋值和取值发生在执行查询的同一阶段：

           - > set @rownum:=0;
             > select actor_id,@rownum as cnt from actor where (@rownum:=@rownum+1)<=1;

## 七、分区设计及优化

对于用户而言，分区表是一个独立的逻辑表，但是底层是由多个物理子表组成。分区表对于用户而言是一个完全封装底层实现的黑盒子，对用户而言是透明的，从文件系统中可以看到多个使用 # 分隔命名的表文件。

mysql 在创建表时使用 partition by 子句定义每个分区存放的数据，在执行查询的时候，优化器会根据分区定义过滤那些没有我们需要数据的分区，这样查询就无须扫描所有分区。

分区的主要目的是将数据安好一个较粗的力度分在不同的表中，这样可以将相关的数据存放在一起。

### 1、分区表的应用场景

1. 表非常大以至于无法全部都放在内存中，或者只在表的最后部分有热点数据，其他均是历史数据
2. 分区表的数据更容易维护
   - 批量删除大量数据可以使用清除整个分区的方式
   - 对一个独立分区进行优化、检查、修复等操作
3. 分区表的数据可以分布在不同的物理设备上，从而高效地利用多个硬件设备
4. 可以使用分区表来避免某些特殊的瓶颈
   - innodb 的单个索引的互斥访问
   - ext3 文件系统的inode锁竞争
5. 可以备份和恢复独立的分区

### 2、分区表的限制

1. 一个表最多只能有1024个分区，在 5.7 版本的时候可以支持 8196 个分区
2. 在早期的 mysql 中，分区表达式必须是整数或者是返回整数的表达式，在 mysql5.5 中，某些场景可以直接使用列来进行分区
3. 如果分区字段中有主键或者唯一索引的列，那么所有主键列和唯一索引列都必须包含进来
4. 分区表无法使用外键约束

### 3、分区表的原理

- 分区表由多个相关的底层表实现，这个底层表也是由句柄对象标识，我们可以直接访问各个分区。存储引擎管理分区的各个底层表和管理普通表一样（所有的底层表都必须使用相同的存储引擎），分区表的索引知识在各个底层表上各自加上一个完全相同的索引。从存储引擎的角度来看，底层表和普通表没有任何不同，存储引擎也无须知道这是一个普通表还是一个分区表的一部分。
- 分区表的操作按照以下的操作逻辑进行：
  - select：当查询一个分区表的时候，分区层先打开并锁住所有的底层表，优化器先判断是否可以过滤部分分区，然后再调用对应的存储引擎接口访问各个分区的数据
  - insert：当写入一条记录的时候，分区层先打开并锁住所有的底层表，然后确定哪个分区接受这条记录，再将记录写入对应底层表
  - delete：当删除一条记录时，分区层先打开并锁住所有的底层表，然后确定数据对应的分区，最后对相应底层表进行删除操作
  - update：当更新一条记录时，分区层先打开并锁住所有的底层表，mysql先确定需要更新的记录再哪个分区，然后取出数据并更新，再判断更新后的数据应该再哪个分区，最后对底层表进行写入操作，并对源数据所在的底层表进行删除操作
- 有些操作时支持过滤的，例如，当删除一条记录时，MySQL需要先找到这条记录，如果where条件恰好和分区表达式匹配，就可以将所有不包含这条记录的分区都过滤掉，这对update同样有效。如果是insert操作，则本身就是只命中一个分区，其他分区都会被过滤掉。mysql先确定这条记录属于哪个分区，再将记录写入对应得曾分区表，无须对任何其他分区进行操作
- 虽然每个操作都会“先打开并锁住所有的底层表”，但这并不是说分区表在处理过程中是锁住全表的，如果存储引擎能够自己实现行级锁，例如innodb，则会在分区层释放对应表锁。

### 4、分区表的类型

1. 范围分区

   - 根据列值在给定范围内将行分配给分区

   - 范围分区表的分区方式是：每个分区都包含行数据且分区的表达式在给定的范围内，分区的范围应该是连续的且不能重叠，可以使用values less than运算符来定义。

     

   1. 创建普通的表

      ```sql
      CREATE TABLE employees (
          id INT NOT NULL,
          fname VARCHAR(30),
          lname VARCHAR(30),
          hired DATE NOT NULL DEFAULT '1970-01-01',
          separated DATE NOT NULL DEFAULT '9999-12-31',
          job_code INT NOT NULL,
          store_id INT NOT NULL
      );
      ```

   2. 创建带分区的表，下面建表的语句是按照store_id来进行分区的，指定了4个分区

      ```sql
      CREATE TABLE employees (
          id INT NOT NULL,
          fname VARCHAR(30),
          lname VARCHAR(30),
          hired DATE NOT NULL DEFAULT '1970-01-01',
          separated DATE NOT NULL DEFAULT '9999-12-31',
          job_code INT NOT NULL,
          store_id INT NOT NULL
      )
      PARTITION BY RANGE (store_id) (
          PARTITION p0 VALUES LESS THAN (6),
          PARTITION p1 VALUES LESS THAN (11),
          PARTITION p2 VALUES LESS THAN (16),
          PARTITION p3 VALUES LESS THAN (21)
      );
      --在当前的建表语句中可以看到，store_id的值在1-5的在p0分区，6-10的在p1分区，11-15的在p3分区，16-20的在p4分区，但是如果插入超过20的值就会报错，因为mysql不知道将数据放在哪个分区
      ```

   3. 可以使用less than maxvalue来避免此种情况

      ```sql
      CREATE TABLE employees (
          id INT NOT NULL,
          fname VARCHAR(30),
          lname VARCHAR(30),
          hired DATE NOT NULL DEFAULT '1970-01-01',
          separated DATE NOT NULL DEFAULT '9999-12-31',
          job_code INT NOT NULL,
          store_id INT NOT NULL
      )
      PARTITION BY RANGE (store_id) (
          PARTITION p0 VALUES LESS THAN (6),
          PARTITION p1 VALUES LESS THAN (11),
          PARTITION p2 VALUES LESS THAN (16),
          PARTITION p3 VALUES LESS THAN MAXVALUE
      );
      --maxvalue表示始终大于等于最大可能整数值的整数值
      ```

   4. 可以使用相同的方式根据员工的职务代码对表进行分区

      ```sql
      CREATE TABLE employees (
          id INT NOT NULL,
          fname VARCHAR(30),
          lname VARCHAR(30),
          hired DATE NOT NULL DEFAULT '1970-01-01',
          separated DATE NOT NULL DEFAULT '9999-12-31',
          job_code INT NOT NULL,
          store_id INT NOT NULL
      )
      PARTITION BY RANGE (job_code) (
          PARTITION p0 VALUES LESS THAN (100),
          PARTITION p1 VALUES LESS THAN (1000),
          PARTITION p2 VALUES LESS THAN (10000)
      );
      ```

   5. 可以使用date类型进行分区：如虚妄根据每个员工离开公司的年份进行划分，如year(separated)

      ```sql
      CREATE TABLE employees (
          id INT NOT NULL,
          fname VARCHAR(30),
          lname VARCHAR(30),
          hired DATE NOT NULL DEFAULT '1970-01-01',
          separated DATE NOT NULL DEFAULT '9999-12-31',
          job_code INT,
          store_id INT
      )
      PARTITION BY RANGE ( YEAR(separated) ) (
          PARTITION p0 VALUES LESS THAN (1991),
          PARTITION p1 VALUES LESS THAN (1996),
          PARTITION p2 VALUES LESS THAN (2001),
          PARTITION p3 VALUES LESS THAN MAXVALUE
      );
      ```

   6. 可以使用函数根据range的值来对表进行分区，如timestampunix_timestamp()

      ```sql
      CREATE TABLE quarterly_report_status (
          report_id INT NOT NULL,
          report_status VARCHAR(20) NOT NULL,
          report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
      )
      PARTITION BY RANGE ( UNIX_TIMESTAMP(report_updated) ) (
          PARTITION p0 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-01-01 00:00:00') ),
          PARTITION p1 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-04-01 00:00:00') ),
          PARTITION p2 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-07-01 00:00:00') ),
          PARTITION p3 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-10-01 00:00:00') ),
          PARTITION p4 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-01-01 00:00:00') ),
          PARTITION p5 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-04-01 00:00:00') ),
          PARTITION p6 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-07-01 00:00:00') ),
          PARTITION p7 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-10-01 00:00:00') ),
          PARTITION p8 VALUES LESS THAN ( UNIX_TIMESTAMP('2010-01-01 00:00:00') ),
          PARTITION p9 VALUES LESS THAN (MAXVALUE)
      );
      --timestamp不允许使用任何其他涉及值的表达式
      ```

   7. 基于时间间隔的分区方案，在mysql5.7中，可以基于范围或事件间隔实现分区方案，有两种选择

      1. 基于范围的分区，对于分区表达式，可以使用操作函数基于date、time、或者datatime列来返回一个整数值

         ```sql
         CREATE TABLE members (
             firstname VARCHAR(25) NOT NULL,
             lastname VARCHAR(25) NOT NULL,
             username VARCHAR(16) NOT NULL,
             email VARCHAR(35),
             joined DATE NOT NULL
         )
         PARTITION BY RANGE( YEAR(joined) ) (
             PARTITION p0 VALUES LESS THAN (1960),
             PARTITION p1 VALUES LESS THAN (1970),
             PARTITION p2 VALUES LESS THAN (1980),
             PARTITION p3 VALUES LESS THAN (1990),
             PARTITION p4 VALUES LESS THAN MAXVALUE
         );
         
         CREATE TABLE quarterly_report_status (
             report_id INT NOT NULL,
             report_status VARCHAR(20) NOT NULL,
             report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
         )
         PARTITION BY RANGE ( UNIX_TIMESTAMP(report_updated) ) (
             PARTITION p0 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-01-01 00:00:00') ),
             PARTITION p1 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-04-01 00:00:00') ),
             PARTITION p2 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-07-01 00:00:00') ),
             PARTITION p3 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-10-01 00:00:00') ),
             PARTITION p4 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-01-01 00:00:00') ),
             PARTITION p5 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-04-01 00:00:00') ),
             PARTITION p6 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-07-01 00:00:00') ),
             PARTITION p7 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-10-01 00:00:00') ),
             PARTITION p8 VALUES LESS THAN ( UNIX_TIMESTAMP('2010-01-01 00:00:00') ),
             PARTITION p9 VALUES LESS THAN (MAXVALUE)
         );
         ```

      2. 基于范围列的分区，使用date或者datatime列作为分区列

         ```sql
         CREATE TABLE members (
             firstname VARCHAR(25) NOT NULL,
             lastname VARCHAR(25) NOT NULL,
             username VARCHAR(16) NOT NULL,
             email VARCHAR(35),
             joined DATE NOT NULL
         )
         PARTITION BY RANGE COLUMNS(joined) (
             PARTITION p0 VALUES LESS THAN ('1960-01-01'),
             PARTITION p1 VALUES LESS THAN ('1970-01-01'),
             PARTITION p2 VALUES LESS THAN ('1980-01-01'),
             PARTITION p3 VALUES LESS THAN ('1990-01-01'),
             PARTITION p4 VALUES LESS THAN MAXVALUE
         );
         ```

   8. 真实案例：

      ```sql
      #不分区的表
      CREATE TABLE no_part_tab
      (id INT DEFAULT NULL,
      remark VARCHAR(50) DEFAULT NULL,
      d_date DATE DEFAULT NULL
      )ENGINE=MYISAM;
      #分区的表
      CREATE TABLE part_tab
      (id INT DEFAULT NULL,
      remark VARCHAR(50) DEFAULT NULL,
      d_date DATE DEFAULT NULL
      )ENGINE=MYISAM
      PARTITION BY RANGE(YEAR(d_date))(
      PARTITION p0 VALUES LESS THAN(1995),
      PARTITION p1 VALUES LESS THAN(1996),
      PARTITION p2 VALUES LESS THAN(1997),
      PARTITION p3 VALUES LESS THAN(1998),
      PARTITION p4 VALUES LESS THAN(1999),
      PARTITION p5 VALUES LESS THAN(2000),
      PARTITION p6 VALUES LESS THAN(2001),
      PARTITION p7 VALUES LESS THAN(2002),
      PARTITION p8 VALUES LESS THAN(2003),
      PARTITION p9 VALUES LESS THAN(2004),
      PARTITION p10 VALUES LESS THAN maxvalue);
      #插入未分区表记录
      DROP PROCEDURE IF EXISTS no_load_part;
       
      
      DELIMITER//
      CREATE PROCEDURE no_load_part()
      BEGIN
          DECLARE i INT;
          SET i =1;
          WHILE i<80001
          DO
          INSERT INTO no_part_tab VALUES(i,'no',ADDDATE('1995-01-01',(RAND(i)*36520) MOD 3652));
          SET i=i+1;
          END WHILE;
      END//
      DELIMITER ;
       
      CALL no_load_part;
      #插入分区表记录
      DROP PROCEDURE IF EXISTS load_part;
       
      DELIMITER&& 
      CREATE PROCEDURE load_part()
      BEGIN
          DECLARE i INT;
          SET i=1;
          WHILE i<80001
          DO
          INSERT INTO part_tab VALUES(i,'partition',ADDDATE('1995-01-01',(RAND(i)*36520) MOD 3652));
          SET i=i+1;
          END WHILE;
      END&&
      DELIMITER ;
       
      CALL load_part;
      ```

2. 列表分区

   - 类似于按 range分区，区别在于list分区是基于列值匹配一个离散值集合中的某个值来进行选择

     ```sql
     CREATE TABLE employees (
         id INT NOT NULL,
         fname VARCHAR(30),
         lname VARCHAR(30),
         hired DATE NOT NULL DEFAULT '1970-01-01',
         separated DATE NOT NULL DEFAULT '9999-12-31',
         job_code INT,
         store_id INT
     )
     
     PARTITION BY LIST(store_id) (
         PARTITION pNorth VALUES IN (3,5,6,9,17),
         PARTITION pEast VALUES IN (1,2,10,11,19,20),
         PARTITION pWest VALUES IN (4,12,13,14,18),
         PARTITION pCentral VALUES IN (7,8,15,16)
     );
     ```

3. 列分区

   - mysql 从 5.5 开始支持 column 分区，可以认为 i 是 range 和 list 的升级版，在 5.5 之后，可以使用 column 分区替代 range 和 list，但是 column 分区只接受普通列不接受表达式

     ```sql
      CREATE TABLE `list_c` (
      `c1` int(11) DEFAULT NULL,
      `c2` int(11) DEFAULT NULL
     ) ENGINE=InnoDB DEFAULT CHARSET=latin1
     /*!50500 PARTITION BY RANGE COLUMNS(c1)
     (PARTITION p0 VALUES LESS THAN (5) ENGINE = InnoDB,
      PARTITION p1 VALUES LESS THAN (10) ENGINE = InnoDB) */
     
      CREATE TABLE `list_c` (
      `c1` int(11) DEFAULT NULL,
      `c2` int(11) DEFAULT NULL,
      `c3` char(20) DEFAULT NULL
     ) ENGINE=InnoDB DEFAULT CHARSET=latin1
     /*!50500 PARTITION BY RANGE COLUMNS(c1,c3)
     (PARTITION p0 VALUES LESS THAN (5,'aaa') ENGINE = InnoDB,
      PARTITION p1 VALUES LESS THAN (10,'bbb') ENGINE = InnoDB) */
     
      CREATE TABLE `list_c` (
      `c1` int(11) DEFAULT NULL,
      `c2` int(11) DEFAULT NULL,
      `c3` char(20) DEFAULT NULL
     ) ENGINE=InnoDB DEFAULT CHARSET=latin1
     /*!50500 PARTITION BY LIST COLUMNS(c3)
     (PARTITION p0 VALUES IN ('aaa') ENGINE = InnoDB,
      PARTITION p1 VALUES IN ('bbb') ENGINE = InnoDB) */
     ```

4. hash分区

   - 基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含 myql 中有效的、产生非负整数值的任何表达式

     ```sql
     CREATE TABLE employees (
         id INT NOT NULL,
         fname VARCHAR(30),
         lname VARCHAR(30),
         hired DATE NOT NULL DEFAULT '1970-01-01',
         separated DATE NOT NULL DEFAULT '9999-12-31',
         job_code INT,
         store_id INT
     )
     PARTITION BY HASH(store_id)
     PARTITIONS 4;
     
     CREATE TABLE employees (
         id INT NOT NULL,
         fname VARCHAR(30),
         lname VARCHAR(30),
         hired DATE NOT NULL DEFAULT '1970-01-01',
         separated DATE NOT NULL DEFAULT '9999-12-31',
         job_code INT,
         store_id INT
     )
     PARTITION BY LINEAR HASH(YEAR(hired))
     PARTITIONS 4;
     ```

5. key分区

   - 类似于hash分区，区别在于key分区只支持一列或多列，且mysql服务器提供其自身的哈希函数，必须有一列或多列包含整数值

     ```sql
     CREATE TABLE tk (
         col1 INT NOT NULL,
         col2 CHAR(5),
         col3 DATE
     )
     PARTITION BY LINEAR KEY (col1)
     PARTITIONS 3;
     ```

6. 子分区

   - 在分区的基础之上，再进行分区后存储

     ```sql
     CREATE TABLE `t_partition_by_subpart`
     (
       `id` INT AUTO_INCREMENT,
       `sName` VARCHAR(10) NOT NULL,
       `sAge` INT(2) UNSIGNED ZEROFILL NOT NULL,
       `sAddr` VARCHAR(20) DEFAULT NULL,
       `sGrade` INT(2) NOT NULL,
       `sStuId` INT(8) DEFAULT NULL,
       `sSex` INT(1) UNSIGNED DEFAULT NULL,
       PRIMARY KEY (`id`, `sGrade`)
     )  ENGINE = INNODB
     PARTITION BY RANGE(id)
     SUBPARTITION BY HASH(sGrade) SUBPARTITIONS 2
     (
     PARTITION p0 VALUES LESS THAN(5),
     PARTITION p1 VALUES LESS THAN(10),
     PARTITION p2 VALUES LESS THAN(15)
     );
     ```

### 5、如何使用分区表

1. 如果需要从非常大的表中查询出某一段时间的记录，而这张表中包含很多年的历史数据，数据是按照时间排序的，此时应该如何查询数据呢？
   - 因为数据量巨大，肯定不能在每次查询的时候都扫描全表。考虑到索引在空间和维护上的消耗，也不希望使用索引，即使使用索引，会发现会产生大量的碎片，还会产生大量的随机IO，但是当数据量超大的时候，索引也就无法起作用了，此时可以考虑使用分区来进行解决
2. 全量扫描数据，不要任何索引
   - 使用简单的分区方式存放表，不要任何索引，根据分区规则大致定位需要的数据为止，通过使用 where 条件将需要的数据限制在少数分区中，这种策略适用于以正常的方式访问大量数据
3. 索引数据，并分离热点
   - 如果数据有明显的热点，而且除了这部分数据，其他数据很少被访问到，那么可以将这部分热点数据单独放在一个分区中，让这个分区的数据能够有机会都缓存在内存中，这样查询就可以只访问一个很小的分区表，能够使用索引，也能够有效的使用缓存

### 6、注意事项

1. null值会使分区过滤无效
2. 分区列和索引列不匹配，会导致查询无法进行分区过滤
3. 选择分区的成本可能很高
4. 打开并锁住所有底层表的成本可能很高
5. 维护分区的成本可能很高

## 八、参数设计优化

### 1、general

1. datadir=/var/lib/mysql
   - 数据文件存放的目录
2. socket=/var/lib/mysql/mysql.sock
   - mysql.socket 表示 server 和 client 在同一台服务器，并且使用 localhost 进行连接，就会使用socket 进行连接
3. pid_file=/var/lib/mysql/mysql.pid
   - 存储 mysql 的 pid
4. port=3306
   - mysql 服务的端口号
5. default_storage_engine=InnoDB
   - mysql 存储引擎
6. skip-grant-tables
   - 当忘记 mysql 的用户名密码的时候，可以在 mysql 配置文件中配置该参数，跳过权限表验证，不需要密码即可登录 mysql

### 2、character

1. character_set_client
   - 客户端数据的字符集
2. character_set_connection
   - mysql处理客户端发来的信息时，会把这些数据转换成连接的字符集格式
3. character_set_results
   - mysql发送给客户端的结果集所用的字符集
4. character_set_database
   - 数据库默认的字符集
5. character_set_server
   - mysql server的默认字符集

### 3、connection

1. max_connections
   - mysql 的最大连接数，如果数据库的并发连接请求比较大，应该调高该值
   - show variables like '%max_connection%';   当前默认是 151
   - set global max_connection=1024
2. max_user_connections
   - 限制每个用户的连接个数
3. back_log
   - mysql 能够暂存的连接数量，当 mysql 的线程在一个很短时间内得到非常多的连接请求时，就会起作用，如果mysql的连接数量达到max_connections时，新的请求会被存储在堆栈中，以等待某一个连接释放资源，如果等待连接的数量超过 back_log，则不再接受连接资源
   - 值越大，意味着当前缓存等待的请求就越多，客户端就会得不到响应，不如直接拒绝连接，不宜过大
4. wait_timeout
   - mysql在关闭一个非交互的连接之前需要等待的时长
5. interactive_timeout
   - 关闭一个交互连接之前需要等待的秒数

### 4、log

1. log_error
   - 指定错误日志文件名称，用于记录当mysqld启动和停止时，以及服务器在运行中发生任何严重错误时的相关信息
2. log_bin
   - 指定二进制日志文件名称，用于记录对数据造成更改的所有查询语句
   - log_bin=master-bin 建议打开
3. binlog_do_db
   - 将更新记录到二进制日志的数据库，其他所有没有显式指定的数据库更新将忽略，不记录在日志中
4. binlog_ignore_db
   - 指定不将更新记录到二进制日志的数据库
5. sync_binlog
   - 指定多少次写日志后同步磁盘
   - 默认是 1
6. general_log
   - 是否开启查询日志记录
   - 当业务较为复杂时，建议打开
7. general_log_file
   - 指定查询日志文件名，用于记录所有的查询语句
8. slow_query_log
   - 是否开启慢查询日志记录
   - 当 sql 语句过慢时会记录到慢查询日志中，建议开启
9. slow_query_log_file
   - 指定慢查询日志文件名称，用于记录耗时比较长的查询语句
10. long_query_time
    - 设置慢查询的时间，超过这个时间的查询语句才会记录日志
11. log_slow_admin_statements
    - 是否将管理语句写入慢查询日志

### 5、cache

查询缓存（新版本被取消）、join（join buffer、索引）、索引、排序

1. key_buffer_size
   - 索引缓存区的大小（只对 myisam 表起作用）
   - 默认 8M
2. query_cache
   1. query_cache_size：查询缓存的大小，未来版本被删除
      1. show status like '%Qcache%'; ：查看缓存的相关属性
      2. Qcache_free_blocks：缓存中相邻内存块的个数，如果值比较大，那么查询缓存中碎片比较多
      3. Qcache_free_memory：查询缓存中剩余的内存大小
      4. Qcache_hits：表示有多少此命中缓存
      5. Qcache_inserts：表示多少次未命中而插入
      6. Qcache_lowmen_prunes：多少条query因为内存不足而被移除cache
      7. Qcache_queries_in_cache：当前cache中缓存的query数量
      8. Qcache_total_blocks：当前cache中block的数量
   2. query_cache_limit
      		超出此大小的查询将不被缓存
   3. query_cache_min_res_unit
      		缓存块最小大小
   4. query_cache_type：缓存类型，决定缓存什么样的查询
      - 0 表示禁用
      - 1 表示将缓存所有结果，除非sql语句中使用sql_no_cache禁用查询缓存
      - 2 表示只缓存select语句中通过sql_cache指定需要缓存的查询
3. sort_buffer_size：每个需要排序的线程分派该大小的缓冲区
   1. innodb 默认 1M
   2. myisam 默认 8M
4. max_allowed_packet=32M：限制server接受的数据包大小
5. join_buffer_size=2M：表示关联缓存的大小
6. thread_cache_size：服务器线程缓存，这个值表示可以重新利用保存再缓存中的线程数量，当断开连接时，那么客户端的线程将被放到缓存中以响应下一个客户而不是销毁，如果线程重新被请求，那么请求将从缓存中读取，如果缓存中是空的或者是新的请求，这个线程将被重新请求，那么这个线程将被重新创建，如果有很多新的线程，增加这个值即可
   1. Threads_cached：代表当前此时此刻线程缓存中有多少空闲线程
   2. Threads_connected：代表当前已建立连接的数量
   3. Threads_created：代表最近一次服务启动，已创建现成的数量，如果该值比较大，那么服务器会一直再创建线程
   4. Threads_running：代表当前激活的线程数

### 6、INNODB

1. innodb_buffer_pool_size=
   - 该参数指定大小的内存来缓冲数据和索引，最大可以设置为物理内存的80%
2. innodb_flush_log_at_trx_commit
   - 主要控制innodb将log buffer中的数据写入日志文件并flush磁盘的时间点，值分别为0，1，2
3. innodb_thread_concurrency
   - 设置innodb线程的并发数，默认为0表示不受限制，如果要设置建议跟服务器的cpu核心数一致或者是cpu核心数的两倍
4. innodb_log_buffer_size
   - 此参数确定日志文件所用的内存大小，以M为单位
5. innodb_log_file_size
   - 此参数确定数据日志文件的大小，以M为单位
6. innodb_log_files_in_group
   - 以循环方式将日志文件写到多个文件中
7. read_buffer_size
   - mysql读入缓冲区大小，对表进行顺序扫描的请求将分配到一个读入缓冲区
8. read_rnd_buffer_size
   - mysql随机读的缓冲区大小
9. innodb_file_per_table
   - 此参数确定为每张表分配一个新的文件，建议打开

## 九、mysql 日志详解

### 1、mysql 三种日志

1. redo（归属于Innodb）
   - 当发生数据修改的时候，innodb 引擎会先将记录写到 redo log 中，并更新内存，此时更新就算是完成了， 同时 innodb 引擎会在合适的时机将记录操作到磁盘中
   - redolog 是固定大小的，是循环写的过程
   - 有了 redolog 之后， innodb 就可以保证即使数据库发生异常重启，之前的记录也不会丢失，叫做 crash-safe
2. undo（归属于Innodb）
   - Undo Log是为了实现事务的原子性，在MySQL数据库lnnoDB存储引擎中，还用Undo Log来实现多版本并发控制(简称：MVCC)
3. binlog（归属于 mysql server 层面），主要做mysql功能层面的事情
   - 与 redo 日志的区别：
     1. redo 是 innodb 独有的，binlog 是所有引擎都可以使用的
     2. redo 是物理日志，记录的是在某个数据页上做了什么修改，binlog 是逻辑日志，记录的是这个语句的原始逻辑
     3. redo 是循环写的，空间会用完，binlog 是可以追加写的，不会覆盖之前的日志信息，顺序写的效率很高
   - Binlog中会记录所有的逻辑，并且采用追加写的方式
   - 一般在企业中数据库会 有备份系统，可以定期执行备份， 备份的周期可以自己设置
   - 恢复数据的过程：
     1. 找到最近一次的全量备份数据
     2. 从备份的时间点开始，将备份的 binlog 取出来，重放到要恢复的那个时刻

### 2、数据更新流程

- <img src="_media/database/SQL调优-MySQL/8、参数化设计\数据更新流程.png" alt="image-20210322164412394"    />

1. 执行器先从引擎中找到数据，如果在内存中直接返回，如果不在内存中，查询后返回
2. 执行器拿到数据之后会先修改数据，然后调用引擎接口重新写入数据
3. 引擎将数据更新到内存，同时写数据到 redo 中，此时处于 prepare 阶段，并通知执行器执行完成，随时可以操作
4. 执行器生成这个操作的 binlog
5. 执行器调用引擎的事务提交接口，引擎把刚刚写完的redo改成commit状态，更新完成

### 3、事物相关的日志实现

- mysql 的事物
  -  A（原子性）C（一致性）I（隔离性）D（持久性）
  - 一致性是最终要实现的追求
- A（原子性）是通过 undo log 实现的
  - 在操作任何数据之前，首先将数据备份到一个地方(这个存储数据备份的地方称为Undo Log)。然后进行数据的修改。如果出现了错误或者用户执行了 ROLLBACK 语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态
    注：undo log是逻辑日志，可以理解为:
    - 当 delete 一条记录时，undo log 中会记录一条对应的 insert 记录
    - 当 insert 一条记录时，undo log 中会记录一条对应的 delete 记录
    - 当 update 一条记录时，它记录一条对应相反的 update 记录
- I（隔离性）对应一些隔离级别，是通过锁来实现的
- D（持久性）是通过 redo log 实现的
  - <img src="_media/database/SQL调优-MySQL/原子性.png"/>
  - 当每次写完 DML 语句后，Innodb 会将数据提交到 log buffer 的内存空间，之后再提交到 OS Buffer 并调用 fsync() 写入磁盘
  - 图二中，0、2 两种方式性能更高，因为每秒相对于 CPU 来说是批量提交，但会存在数据丢失 1 秒的可能，其中 2 的性能更高，因为会直接写到 OS Buffer，减少一次内存间复制的过程，1 的安全性更高
  - 每次在写的时候，能够保证日志的持久化，之后就算实际的数据文件没有写到磁盘，也可以通过日志文件进行恢复，从而达到持久性
- C（一致性）是通过 A I D 来实现的

## 十、锁

MySQL 的锁应与存储引擎挂钩，其中 myisam 中的锁叫共享读锁和独占写锁，而 Innodb 中叫共享锁和排它锁

### 1、基本介绍

- 锁是计算机协调多个进程或线程并发访问某一资源的机制。
- 在数据库中，除传统的 计算资源（如CPU、RAM、I/O等）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一 个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。
- 相对其他数据库而言，MySQL 的锁机制比较简单，其最 显著的特点是不同的**存储引擎**支持不同的锁机制。比如，MyISAM 和 MEMORY 存储引擎采用的是表级锁（table-level locking）；InnoDB存储引擎既支持行级锁（row-level locking），也支持表级锁，但默认情况下是采用行级锁。
- **表级锁：**开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。 
- **行级锁：**开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
- 从上述特点可见，很难笼统地说哪种锁更好，只能就具体应用的特点来说哪种锁更合适！仅从锁的角度 来说：表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web应用；而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有 并发查询的应用，如一些在线事务处理（OLTP）系统。

### 2、MyISAM 表锁

- MySQL的表级锁有两种模式：
  - **表共享读锁（Table Read Lock）**
  - **表独占写锁（Table Write Lock）**。
- 对 MyISAM 表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；对 MyISAM 表的写操作，则会阻塞其他用户对同一表的读和写操作；MyISAM 表的读操作与写操作之间，以及写操作之间是串行的！

### 3、MyISAM  案例及注意

- 建表语句

  ```sql
  CREATE TABLE `mylock` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `NAME` varchar(20) DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=MyISAM DEFAULT CHARSET=utf8;
  
  INSERT INTO `mylock` (`id`, `NAME`) VALUES ('1', 'a');
  INSERT INTO `mylock` (`id`, `NAME`) VALUES ('2', 'b');
  INSERT INTO `mylock` (`id`, `NAME`) VALUES ('3', 'c');
  INSERT INTO `mylock` (`id`, `NAME`) VALUES ('4', 'd');
  ```

- MyISAM 写阻塞读案例

  - 当一个线程获得对一个表的写锁之后，只有持有锁的线程可以对表进行更新操作。其他线程的读写操作都会等待，直到锁释放为止。（独占锁）

  - | session1                                                     | session2                                             |
    | ------------------------------------------------------------ | ---------------------------------------------------- |
    | 获取表的write锁定 lock table mylock write;                   |                                                      |
    | 当前session对表的查询，插入，更新操作都可以执行 select * from mylock; insert into mylock values(5,'e'); | 当前session对表的查询会被阻塞 select * from mylock； |
    | 释放锁： unlock tables；                                     | 当前session能够立刻执行，并返回对应结果              |

- MyISAM 读阻塞写案例

  - 一个 session 使用 lock table 给表加读锁，这个 session 可以锁定表中的记录，但更新和访问其他表都会提示错误，同时，另一个 session 可以查询表中的记录，但更新就会出现锁等待。

  - | session1                                                     | session2                                                     |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | 获得表的read锁定 lock table mylock read;                     |                                                              |
    | 当前session可以查询该表记录： select * from mylock;          | 当前session可以查询该表记录： select * from mylock;          |
    | 当前session不能查询没有锁定的表 select * from person Table 'person' was not locked with LOCK TABLES | 当前session可以查询或者更新未锁定的表 select * from mylock insert into person values(1,'zhangsan'); |
    | 当前session插入或者更新表会提示错误 insert into mylock values(6,'f') Table 'mylock' was locked with a READ lock and can't be updated update mylock set name='aa' where id = 1; Table 'mylock' was locked with a READ lock and can't be updated | 当前session插入数据会等待获得锁 insert into mylock values(6,'f'); |
    | 释放锁 unlock tables;                                        | 获得锁，更新成功                                             |

- 注意

  - **MyISAM在执行查询语句之前，会自动给涉及的所有表加读锁，在执行更新操作前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此用户一般不需要使用命令来显式加锁，上例中的加锁时为了演示效果。**

  - **MyISAM的并发插入问题**

    - MyISAM表的读和写是串行的，这是就总体而言的，在一定条件下，MyISAM也支持查询和插入操作的并发执行

    - | session1                                                     | session2                                                     |
      | ------------------------------------------------------------ | ------------------------------------------------------------ |
      | 获取表的read local锁定 lock table mylock read local          |                                                              |
      | 当前session不能对表进行更新或者插入操作 insert into mylock values(6,'f') Table 'mylock' was locked with a READ lock and can't be updated update mylock set name='aa' where id = 1; Table 'mylock' was locked with a READ lock and can't be updated | 其他session可以查询该表的记录 select* from mylock            |
      | 当前session不能查询没有锁定的表 select * from person Table 'person' was not locked with LOCK TABLES | 其他session可以进行插入操作，但是更新会阻塞 update mylock set name = 'aa' where id = 1; |
      | 当前session不能访问其他session插入的记录；                   |                                                              |
      | 释放锁资源：unlock tables                                    | 当前session获取锁，更新操作完成                              |
      | 当前session可以查看其他session插入的记录                     |                                                              |

    - 可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺：

      ```sql
      mysql> show status like 'table%';
      +-----------------------+-------+
      | Variable_name         | Value |
      +-----------------------+-------+
      | Table_locks_immediate | 352   |
      | Table_locks_waited    | 2     |
      +-----------------------+-------+
      --如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。
      ```

### 4、事物及存在的问题

1. 事物及其 ACID 属性

   - 事务是由一组SQL语句组成的逻辑处理单元，事务具有4属性，通常称为事务的ACID属性。
   - 原子性（Actomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。 
   - 一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。
   - 隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。 
   - 持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

2. 并发事物带来的问题

   - 相对于串行处理来说，并发事务处理能大大增加数据库资源的利用率，提高数据库系统的事务吞吐量，从而可以支持更多用户的并发操作，但与此同时，会带来一下问题，以下

     - **脏读**： 一个事务正在对一条记录做修改，在这个事务并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”的数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做“脏读”
     - **不可重复读**：一个事务在读取某些数据已经发生了改变、或某些记录已经被删除了！这种现象叫做“不可重复读”。
     - **幻读**： 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读”

   - 上述出现的问题都是数据库读一致性的问题，可以通过事务的隔离机制来进行保证。

   - 数据库的事务隔离越严格，并发副作用就越小，但付出的代价也就越大，因为事务隔离本质上就是使事务在一定程度上串行化，需要根据具体的业务需求来决定使用哪种隔离级别

     - | 脏读             | 不可重复读 | 幻读 |      |
       | ---------------- | ---------- | ---- | ---- |
       | read uncommitted | √          | √    | √    |
       | read committed   |            | √    | √    |
       | repeatable read  |            |      | √    |
       | serializable     |            |      |      |

   - 可以通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况：

     ```sql
     mysql> show status like 'innodb_row_lock%';
     +-------------------------------+-------+
     | Variable_name                 | Value |
     +-------------------------------+-------+
     | Innodb_row_lock_current_waits | 0     |
     | Innodb_row_lock_time          | 18702 |
     | Innodb_row_lock_time_avg      | 18702 |
     | Innodb_row_lock_time_max      | 18702 |
     | Innodb_row_lock_waits         | 1     |
     +-------------------------------+-------+
     --如果发现锁争用比较严重，如InnoDB_row_lock_waits和InnoDB_row_lock_time_avg的值比较高
     ```

### 5、InnoDB 行锁模式

- **共享锁（s）**：又称读锁。
  - 允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。
  - 若事务 T 对数据对象 A 加上 S 锁，则事务T可以读 A 但不能修改 A，其他事务只能再对 A 加S 锁，而不能加 X 锁，直到 T 释放 A 上的 S 锁。这保证了其他事务可以读 A，但在 T 释放 A上的 S锁之前不能对 A 做任何修改。 		
- **排他锁（x）**：又称写锁。
  - 允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。
  - 若事务 T 对数据对象 A 加上 X 锁，事务T可以读 A 也可以修改 A，其他事务不能再对A加任何锁，直到 T 释放 A 上的锁。
- mysql InnoDB引擎默认的修改数据语句：**update、delete、insert 都会自动给涉及到的数据加上排他锁，select 语句默认不会加任何锁类型**，如果加排他锁可以使用select …for update语句，加共享锁可以使用select … lock in share mode语句。
- **所以加过排他锁的数据行在其他事务种是不能修改数据的，也不能通过 for update 和 lock in share mode 锁的方式查询数据，但可以直接通过 select …from… 查询数据，因为普通查询没有任何锁机制。**

### 6、InnoDB 行锁实现方式

- InnoDB 行锁是通过给**索引**上的索引项加锁来实现的，这一点 MySQL 与 Oracle 不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB 这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB 才使用行级锁，**否则，InnoDB 将使用表锁！**

1. 在不通过索引条件查询的时候，innodb使用的是表锁而不是行锁

   - ```sql
     create table tab_no_index(id int,name varchar(10)) engine=innodb;
     insert into tab_no_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
     ```

   - | session1                                                  | session2                                                |
     | --------------------------------------------------------- | ------------------------------------------------------- |
     | set autocommit=0 select * from tab_no_index where id = 1; | set autocommit=0 select * from tab_no_index where id =2 |
     | select * from tab_no_index where id = 1 for update        |                                                         |
     |                                                           | select * from tab_no_index where id = 2 for update;     |

   - session1只给一行加了排他锁，但是session2在请求其他行的排他锁的时候，会出现锁等待。原因是在没有索引的情况下，innodb只能使用表锁。

2. 创建带索引的表进行条件查询，innodb使用的是行锁

   - ```sql
     create table tab_with_index(id int,name varchar(10)) engine=innodb;
     alter table tab_with_index add index id(id);
     insert into tab_with_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
     ```

   - | session1                                                   | session2                                                 |
     | ---------------------------------------------------------- | -------------------------------------------------------- |
     | set autocommit=0 select * from tab_with_indexwhere id = 1; | set autocommit=0 select * from tab_with_indexwhere id =2 |
     | select * from tab_with_indexwhere id = 1 for update        |                                                          |
     |                                                            | select * from tab_with_indexwhere id = 2 for update;     |

3. 由于mysql的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现冲突的，依然无法访问具体的数据

   - ```sql
     alter table tab_with_index drop index id;
     insert into tab_with_index  values(1,'4');
     ```

   - | session1                                                     | session2                                                     |
     | ------------------------------------------------------------ | ------------------------------------------------------------ |
     | set autocommit=0                                             | set autocommit=0                                             |
     | select * from tab_with_index where id = 1 and name='1' for update |                                                              |
     |                                                              | select * from tab_with_index where id = 1 and name='4' for update 虽然session2访问的是和session1不同的记录，但是因为使用了相同的索引，锁的是具体的表，所以需要等待锁 |

### 7、总结

1. **对于MyISAM的表锁，主要讨论了以下几点：** 
   1. 共享读锁（S）之间是兼容的，但共享读锁（S）与排他写锁（X）之间，以及排他写锁（X）之间是互斥的，也就是说读和写是串行的。
   2. 在一定条件下，MyISAM 允许查询和插入并发执行，我们可以利用这一点来解决应用中对同一表查询和插入的锁争用问题。 
   3. MyISAM 默认的锁调度机制是写优先，这并不一定适合所有应用，用户可以通过设置LOW_PRIORITY_UPDATES 参数，或在 INSERT、UPDATE、DELETE 语句中指定LOW_PRIORITY 选项来调节读写锁的争用。 
   4. 由于表锁的锁定粒度大，读写之间又是串行的，因此，如果更新操作较多，MyISAM 表可能会出现严重的锁等待，可以考虑采用 InnoDB 表来减少锁冲突。
2. **对于InnoDB，本文主要讨论了以下几项内容：** 
   1. InnoDB的行锁是基于索引实现的，如果不通过索引访问数据，InnoDB会使用表锁。 
   2. 在不同的隔离级别下，InnoDB的锁机制和一致性读策略不同。
   3. 当发生死锁，mysql 会自己进行释放锁的机制，先释放后加锁的行
3. 在了解InnoDB锁特性后，用户可以通过设计和SQL调整等措施减少锁冲突和死锁，包括：
   1. 尽量使用较低的隔离级别； 精心设计索引，并尽量使用索引访问数据，使加锁更精确，从而减少锁冲突的机会；
   2. 选择合理的事务大小，小事务发生锁冲突的几率也更小；
   3. 给记录集显式加锁时，最好一次性请求足够级别的锁。比如要修改数据的话，最好直接申请排他锁，而不是先申请共享锁，修改时再请求排他锁，这样容易产生死锁；
   4. 不同的程序访问一组表时，应尽量约定以相同的顺序访问各表，对一个表而言，尽可能以固定的顺序存取表中的行。这样可以大大减少死锁的机会；
   5. 尽量用相等条件访问数据，这样可以避免间隙锁对并发插入的影响； 不要申请超过实际需要的锁级别；除非必须，查询时不要显示加锁；
   6. 对于一些特定的事务，可以使用表锁来提高处理速度或减少死锁的可能。

## 十一、主从复制

### 1、为什么需要主从复制

1. 在业务复杂的系统中，有这么一个情景，有一句sql语句需要锁表，导致暂时不能使用读的服务，那么就很影响运行中的业务，使用主从复制，让主库负责写，从库负责读，这样，即使主库出现了锁表的情景，通过读从库也可以保证业务的正常运作。
2. 做数据的热备
3. 架构的扩展。业务量越来越大，I/O访问频率过高，单机无法满足，此时做多库的存储，降低磁盘I/O访问的频率，提高单个机器的I/O性能。

### 2、什么是 MySQL 的主从复制

- MySQL 主从复制是指数据可以从一个MySQL数据库服务器主节点复制到一个或多个从节点。
- MySQL 默认采用异步复制方式，这样从节点不用一直访问主服务器来更新自己的数据，数据的更新可以在远程连接上进行，从节点可以复制主数据库中的所有数据库或者特定的数据库，或者特定的表。

### 3、MySQL 复制的原理

1. 原理
   1. master 服务器将数据的改变记录二进制 binlog 日志，当 master 上的数据发生改变时，则将其改变写入二进制日志中；
   2. slave 服务器会在一定时间间隔内对 master 二进制日志进行探测其是否发生改变，如果发生改变，则开始一个 I/O Thread 请求 master 二进制事件
   3. 同时主节点为每个 I/O 线程启动一个 dump 线程，用于向其发送二进制事件，并保存至从节点本地的中继日志中，从节点将启动 SQL 线程从中继日志中读取二进制日志，在本地重放，使得其数据和主节点的保持一致，最后 I/OThread 和 SQLThread 将进入睡眠状态，等待下一次被唤醒。
2. 解释
   1. 从库会生成两个线程，一个 I/O 线程,，一个 SQL 线程；
   2. I/O 线程会去请求主库的 binlog，并将得到的 binlog 写到本地的 relay-log (中继日志)文件中；
   3. 主库会生成一个 log dump 线程,用来给从库 I/O 线程传 binlog；
   4. SQL 线程,会读取 relay log 文件中的日志,并解析成sql语句逐一执行；
3. 注意
   1. master 将操作语句记录到 binlog 日志中，然后授予 slave 远程连接的权限（master 一定要开启binlog 二进制日志功能；通常为了数据安全考虑，slave 也开启 binlog 功能）。 
   2. slave 开启两个线程：IO 线程和SQL线程。其中：
      - IO 线程负责读取 master 的 binlog 内容到中继日志 relay log 里；
      - SQL 线程负责从 relay log 日志里读出 binlog 内容，并更新到 slave 的数据库里，这样就能保证 slave 数据和 master 数据保持一致了。
   3. Mysql 复制至少需要两个 Mysql 的服务，当然 Mysql 服务可以分布在不同的服务器上，也可以在一台服务器上启动多个服务。 
   4. Mysql 复制最好确保 master 和 slave 服务器上的 Mysql 版本相同（如果不能满足版本一致，那么要保证 master 主节点的版本低于 slave 从节点的版本） 
   5. master 和 slave 两节点间时间需同步

### 4、MySQL 复制具体步骤

1. 从库通过手工执行 change  master to 语句连接主库，提供了连接的用户一切条件（user 、password、port、ip），并且让从库知道，二进制日志的起点位置（file名 position 号）；    start  slave
2. 从库的 IO 线程和主库的 dump 线程建立连接。
3. 从库根据 change  master  to 语句提供的 file 名和 position 号，IO线 程向主库发起 binlog 的请求。
4. 主库 dump 线程根据从库的请求，将本地 binlog 以 events 的方式发给从库IO线程。
5. 从库 IO 线程接收 binlog  events，并存放到本地 relay-log 中，传送过来的信息，会记录到master.info中
6. 从库 SQL 线程应用 relay-log，并且把应用过的记录到 relay-log.info 中，默认情况下，已经应用过的 relay 会自动被清理 purge

### 5、MySQL 主从形式

1. 一主一从
   - ![image-20210323144847761](_media/database/SQL调优-MySQL/一主一从.png"/>
2. 主主复制
   - ![image-20210323144922479](_media/database/SQL调优-MySQL/主主复制.png"/>
3. 一主多从
   - ![image-20210323144937580](_media/database/SQL调优-MySQL/一主多从.png"/>
4. 多主一从
   - ![image-20210323144954427](_media/database/SQL调优-MySQL/多主一从.png"/>
5. 联级复制
   - ![image-20210323145012062](_media/database/SQL调优-MySQL/联级复制.png"/>

### 6、MySQL 同步延时分析

- 同步延时：
  - mysql 的主从复制都是单线程的操作，主库对所有 DDL 和 DML 产生的日志写进 binlog，由于binlog 是顺序写，所以效率很高，slave 的 sql thread 线程将主库的 DDL 和 DML 操作事件在slave 中重放。DML 和 DDL 的 IO 操作是随机的，不是顺序，所以成本要高很多，另一方面，由于 sql thread 也是单线程的，当主库的并发较高时，产生的 DML 数量超过 slave 的SQL thread 所能处理的速度，或者当 slave中 有大型 query 语句产生了锁等待，那么延时就产生了。
- 解决方案
  1. 业务的持久化层的实现采用分库架构，mysql 服务可平行扩展，分散压力。
  2. 单个库读写分离，一主多从，主写从读，分散压力。这样从库压力比主库高，保护主库。
  3. 服务的基础架构在业务和 mysql 之间加入 memcache 或者 redis 的 cache 层。降低 mysql 的读压力。 
  4. 不同业务的 mysql 物理上放在不同机器，分散压力。
  5. 使用比主库更好的硬件设备作为 slave，mysql 压力小，延迟自然会变小。
  6. 使用更加强劲的硬件设备
- mysql5.7之后使用 MTS 并行复制技术，永久解决复制延时问题
  - https://blog.csdn.net/weixin_30080745/article/details/113305346

### 7、MySQL 主从复制安装配置

1. 基础设置准备

   ```shell
   #操作系统：
   centos6.5
   #mysql版本：
   5.7
   #两台虚拟机：
   node1:192.168.85.111（主）
   node2:192.168.85.112（从）
   ```

2. 安装 mysql 数据库

3. 在两台数据库中分别创建数据库

   ```sql
   --注意两台必须全部执行
   create database msb;
   ```

4. 在主（node01）服务器配置如下

   ```shell
   #修改配置文件，执行以下命令打开mysql配置文件
   vi /etc/my.cnf
   #在mysqld模块中添加如下配置信息
   log-bin=master-bin #二进制文件名称
   binlog-format=ROW  #二进制日志格式，有row、statement、mixed三种格式，row指的是把改变的内容复制过去，而不是把命令在从服务器上执行一遍，statement指的是在主服务器上执行的SQL语句，在从服务器上执行同样的语句。MySQL默认采用基于语句的复制，效率比较高。mixed指的是默认采用基于语句的复制，一旦发现基于语句的无法精确的复制时，就会采用基于行的复制。
   server-id=1		   #要求各个服务器的id必须不一样
   binlog-do-db=msb   #同步的数据库名称
   ```

5. 配置从服务器登录主服务器的账号授权

   ```sql
   --授权操作
   set global validate_password_policy=0;
   set global validate_password_length=1;
   grant replication slave on *.* to 'root'@'%' identified by '123456';
   --刷新权限
   flush privileges;
   ```

6. 从服务器的配置

   ```shell
   #修改配置文件，执行以下命令打开mysql配置文件
   vi /etc/my.cnf
   #在mysqld模块中添加如下配置信息
   log-bin=master-bin	#二进制文件的名称
   binlog-format=ROW	#二进制文件的格式
   server-id=2			#服务器的id
   ```

7. 重启主服务器的 mysqlId 服务

   ```shell
   #重启mysql服务
   service mysqld restart
   #登录mysql数据库
   mysql -uroot -p
   #查看master的状态
   show master status；
   ```

## 十二、总结及补充

### 1、mysql 8.0 & mysql 5.7 区别

- https://blog.csdn.net/zwj1030711290/article/details/80025981

### 2、事物测试

1. 打开 mysql 的命令行，将自动提交事物关闭

   ```sql
   --查看是否是自动提交 1表示开启，0表示关闭
   select @@autocommit;
   --设置关闭
   set autocommit = 0;
   ```

2. 数据准备

   ```sql
   --创建数据库
   create database tran;
   --切换数据库 两个窗口都执行
   use tran;
   --准备数据
    create table psn(id int primary key,name varchar(10)) engine=innodb;
   --插入数据
   insert into psn values(1,'zhangsan');
   insert into psn values(2,'lisi');
   insert into psn values(3,'wangwu');
   commit;
   ```

3. 测试事物

   - ```sql
     --事务包含四个隔离级别：从上往下，隔离级别越来越高，意味着数据越来越安全
     read uncommitted; 	--读未提交
     read commited;		--读已提交
     repeatable read;	--可重复读
     (seariable)			--序列化执行，串行执行
     --产生数据不一致的情况：
     脏读
     不可重复读
     幻读
     ```

   - | 隔离级别 | 异常情况 |            | 异常情况 |
     | -------- | -------- | ---------- | -------- |
     | 读未提交 | 脏读     | 不可重复读 | 幻读     |
     | 读已提交 |          | 不可重复读 | 幻读     |
     | 可重复读 |          |            | 幻读     |
     | 序列化   |          |            |          |

4. 测试 - 01：脏读 read uncommitted

   ```sql
   set session transaction isolation level read uncommitted;
   A:start transaction;
   A:select * from psn;
   B:start transaction;
   B:select * from psn;
   A:update psn set name='msb';
   A:selecet * from psn
   B:select * from psn;  --读取的结果msb。产生脏读，因为A事务并没有commit，读取到了不存在的数据
   A:commit;
   B:select * from psn; --读取的数据是msb,因为A事务已经commit，数据永久的被修改
   ```

5. 测试 - 02：当使用read committed的时候，就不会出现脏读的情况了，当时会出现不可重复读的问题

   ```sql
   set session transaction isolation level read committed;
   A:start transaction;
   A:select * from psn;
   B:start transaction;
   B:select * from psn;
   --执行到此处的时候发现，两个窗口读取的数据是一致的
   A:update psn set name ='zhangsan' where id = 1;
   A:select * from psn;
   B:select * from psn;
   --执行到此处发现两个窗口读取的数据不一致，B窗口中读取不到更新的数据
   A:commit;
   A:select * from psn;--读取到更新的数据
   B:select * from psn;--也读取到更新的数据
   --发现同一个事务中多次读取数据出现不一致的情况
   ```

6. 测试 - 03：当使用repeatable read的时候(按照上面的步骤操作)，就不会出现不可重复读的问题，但是会出现幻读的问题

   ```sql
   set session transaction isolation level repeatable read;
   A:start transaction;
   A:select * from psn;
   B:start transaction;
   B:select * from psn;
   --此时两个窗口读取的数据是一致的
   A:insert into psn values(4,'sisi');
   A:commit;
   A:select * from psn;--读取到添加的数据
   B:select * from psn;--读取不到添加的数据
   B:insert into psn values(4,'sisi');--报错，无法插入数据
   --此时发现读取不到数据，但是在插入的时候不允许插入，出现了幻读，设置更高级别的隔离级别即可解决
   ```

### 3、MySQL 5.7 Linux 安装

1. 更换 yum 源

   1. 打开 mirrors.aliyun.com，选择centos的系统，点击帮助

   2. 执行命令：yum install wget -y

   3. 改变某些文件的名称

      ```shell
      mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
      ```

   4. 执行更换yum源的命令

      ```shell
      wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
      ```

   5. 更新本地缓存

      - > yum clean all

      - > yum makecache

2. 查看系统中是否自带安装 mysql

   - ```shell
     yum list installed | grep mysql
     ```

   - ![image-20210323151437285](_media/database/SQL调优-MySQL/检查系统mysql.png"/>

3. 删除系统自带的 mysql 及其依赖 （防止冲突）

   - ```shell
     yum -y remove mysql-libs.x86_64
     ```

   - ![image-20210323151541738](_media/database/SQL调优-MySQL/删除系统mysql.png"/>

4. 安装 wget 命令

   - ```shell
     yum install wget -y 
     ```

   - ![image-20210323151612320](_media/database/SQL调优-MySQL/安装wget.png"/>

5. 给 CentOS 添加 rpm 源，且选择较新的源

   - ```shell
     wget dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm 
     ```

   - ![image-20210323151658758](_media/database/SQL调优-MySQL/rpm源.png"/>

6. 安装下载好的 rpm 文件

   - ```shell
     yum install mysql-community-release-el6-5.noarch.rpm -y
     ```

   - ![image-20210323151755078](_media/database/SQL调优-MySQL/安装rpm.png"/>

7. 安装成功之后，会在/etc/yum.repos.d/文件夹下增加两个文件

   - ![image-20210323151824759](_media/database/SQL调优-MySQL/新增文件.png"/>

8. 修改mysql-community.repo文件

   - 源文件
     - ![image-20210323151907496](_media/database/SQL调优-MySQL/原文件.png"/>
   - 修改后
     - ![image-20210323151927873](_media/database/SQL调优-MySQL/修改后文件.png"/>

9. 使用 yum 安装 mysql

   - ```shell
     yum install mysql-community-server -y
     ```

   - ![image-20210323152020899](_media/database/SQL调优-MySQL/安装mysql.png"/>

10. 启动 mysql 服务并设置开机启动

    ```shell
    #启动之前需要生成临时密码，需要用到证书，可能证书过期，需要进行更新操作
    yum update -y
    #启动mysql服务
    service mysqld start
    #设置mysql开机启动
    chkconfig mysqld on
    ```

11. 获取 mysql 的临时密码

    - ```shell
      grep "password" /var/log/mysqld.log
      ```

    - ![image-20210323152119633](_media/database/SQL调优-MySQL/临时密码.png"/>

12. 使用临时密码登录

    ```shell
    mysql -uroot -p
    #输入密码
    ```

13. 修改密码

    ```sql
    set global validate_password_policy=0;
    set global validate_password_length=1;
    ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
    ```

14. 修改远程访问权限

    ```sql
    grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
    flush privileges;
    ```

15. 设置字符集为utf-8

    ```shell
    #在[mysqld]部分添加：
    character-set-server=utf8
    #在文件末尾新增[client]段，并在[client]段添加：
    default-character-set=utf8
    ```

### 4、Linux MySQL 卸载

1. 查看 mysql 的安装情况

   - ```shell
     rpm -qa | grep -i mysql
     ```

   - ![image-20210323152446996](_media/database/SQL调优-MySQL/mysql安装情况.png"/>

2. 删除上图安装的软件

   ```shell
   rpm -ev mysql-community-libs-5.7.27-1.el6.x86_64 --nodeps
   ```

3. 都删除成功之后，查找相关的mysql的文件

   - ```shell
     find / -name mysql
     ```

   - ![image-20210323152515252](_media/database/SQL调优-MySQL/查找相关文件.png"/>

4. 删除全部文件

   ```shell
   rm -rf /var/lib/mysql
   rm -rf /var/lib/mysql/mysql
   rm -rf /etc/logrotate.d/mysql
   rm -rf /usr/share/mysql
   rm -rf /usr/bin/mysql
   rm -rf /usr/lib64/mysql
   ```

5. 再次执行命令

   ```shell
   rpm -qa | grep -i mysql
   #如果没有显式则表示卸载完成
   ```

### 5、MySQL SQL  基础语法

```sql
--给表添加注释
comment on table emp is '雇员表';
--给列添加注释
comment on column emp.ename is '雇员姓名';

/*sql语句学习

SELECT [DISTINCT] {*,column alias,..}
FROM table alias
Where 条件表达式

*/

--查询雇员表中部门编号是10的员工
select empno,ename,job from emp where deptno = 10;
--dinstinct 去除重复数据
select distinct deptno from emp;
--去重也可以针对多个字段，多个字段值只要有一个不匹配就算是不同的记录
select distinct deptno,sal from emp;


--在查询的过程中可以给列添加别名，同时也可以给表添加别名
select e.empno 雇员编号,e.ename 雇员名称,e.job 雇员工作 from emp e where e.deptno = 10;
--给列起别名可以加as，也可以不加，看你心情
select e.empno as 雇员编号,e.ename  as 雇员名称,e.job as 雇员工作 from emp e where e.deptno = 10;
--给列起别名，如果别名中包含空格，那么需要将别名整体用“”包含起来
select e.empno as "雇员 编号",e.ename  as "雇员 名称",e.job as "雇员 工作" from emp e where e.deptno = 10;
--查询表中的所有字段,可以使用*,但是在项目中千万不要随便使用*,容易被打死
select * from emp;


/*
＝，！＝,<>，<,>,<=,>=,any,some,all
is null,is not null
between x and y
in（list），not in（list）
exists（sub－query）
like  _ ,%,escape ‘\‘   _\% escape ‘\’

*/
-- =
select * from emp where deptno = 20;
--!=
select * from emp where deptno !=20;
--<> 不等于
select * from emp where deptno <> 20;
--<,
select sal from emp where sal <1500;
-->,
select sal from emp where sal >1500;
--<=,
select sal from emp where sal <=1500;
-->=,
select sal from emp where sal >=1500;
--any,取其中任意一个
select sal from emp where sal > any(1000,1500,3000);
--some,some跟any是同一个效果，只要大于其中某一个值都会成立
select sal from emp where sal > some(1000,1500,3000);
--all，大于所有的值才会成立
select sal from emp where sal > all(1000,1500,3000);
--is null,在sql的语法中，null表示一个特殊的含义，null != null,不能使用=，！=判断，需要使用is ,is not
select * from emp where comm is null;
--,is not null
select * from emp where comm is not null;
select * from emp where null is null;
--between x and y,包含x和y的值
select * from emp where sal between 1500 and 3000;
select * from emp where sal >=1500 and sal <=3000;
--需要进行某些值的等值判断的时候可以使用in和not in
--in（list），
select * from emp where deptno in(10,20);
--可是用and 和or这样的关键字，and相当于是与操作，or相当于是或操作
--and和or可能出现在同一个sql语句中，此时需要注意and和or的优先级
--and 的优先级要高于or，所以一定要将or的相关操作用（）括起来，提高优先级
select * from emp where deptno = 10 or deptno = 20;
--not in（list）
select * from emp where deptno not in(10,20);
select * from emp where deptno != 10 and deptno !=20;
/*exists（sub－query）,当exists中的子查询语句能查到对应结果的时候，
意味着条件满足
相当于双层for循环
--现在要查询部门编号为10和20的员工，要求使用exists实现
*/
select * from emp where deptno = 10 or deptno = 20;
--通过外层循环来规范内层循环
select *
  from emp e
 where exists (select deptno
          from dept d
         where (d.deptno = 10 or d.deptno = 20)
           and e.deptno = d.deptno)
/*
模糊查询：
like  _ ,%,escape ‘\‘   _\% escape ‘\’

在like的语句中，需要使用占位符或者通配符
_,某个字符或者数字仅出现一次
%，任意字符出现任意次数
escape,使用转义字符,可以自己规定转义字符

使用like的时候要慎重，因为like的效率比较低
使用like可以参考使用索引，但是要求不能以%开头
涉及到大文本的检索的时候，可以使用某些框架 luence，solr，elastic search
*/
--查询名字以S开头的用户
select * from emp where ename like('S%')
--查询名字以S开头且倒数第二个字符为T的用户
select * from emp where ename like('S%T_');
select * from emp where ename like('S%T%');
--查询名字中带%的用户
select * from emp where ename like('%\%%') escape('\')
/*

order by进行排序操作
默认情况下完成的是升序的操作，
asc:是默认的排序方式，表示升序
desc：降序的排序方式

排序是按照自然顺序进行排序的
如果是数值，那么按照从大到小
如果是字符串，那么按照字典序排序

在进行排序的时候可以指定多个字段，而且多个字段可以使用不同的排序方式

每次在执行order by的时候相当于是做了全排序，思考全排序的效率
会比较耗费系统的资源，因此选择在业务不太繁忙的时候进行
*/
select * from emp order by sal;
select * from emp order by sal desc;
select * from emp order by ename;
select * from emp order by sal desc,ename asc;
--使用计算字段
--字符串连接符
select 'my name is '||ename name from emp;
select concat('my name is ',ename) from emp;
--计算所有员工的年薪
select ename,(e.sal+e.comm)*12 from emp e;
--null是比较特殊的存在，null做任何运算都还是为null，因此要将空进行转换
--引入函数nvl，nvl(arg1,arg2),如果arg1是空，那么返回arg2，如果不是空，则返回原来的值
select ename,(e.sal+nvl(e.comm,0))*12  from emp e;
--dual是oracle数据库中的一张虚拟表，没有实际的数据，可以用来做测试
select 100+null from dual;
--A
select * from emp where deptno =30;
--B
select * from emp where sal >1000; 
--并集，将两个集合中的所有数据都进行显示，但是不包含重复的数据
select * from emp where deptno =30 union
select * from emp where sal >1000;
--全集，将两个集合的数据全部显示，不会完成去重的操作
select * from emp where deptno =30 union all
select * from emp where sal >1000;
--交集，两个集合中交叉的数据集，只显示一次
select * from emp where deptno =30 intersect 
select * from emp where sal >1000;
--差集,包含在A集合而不包含在B集合中的数据，跟A和B的集合顺序相关
select * from emp where deptno =30 minus 
select * from emp where sal >1000;
```

### 6、MySQL SQL  函数

```sql
--函数的测试
/*
组函数又称为聚合函数
  输入多个值，最终只会返回一个值
  组函数仅可用于选择列表或查询的having子句
单行函数
  输入一个值，输出一个值


*/

--查询所有员工的薪水总和
select sum(sal) from emp;
--查看表中有多少条记录
select deptno,count(*) from emp group by deptno where count(*) >3;

--字符函数
--concat：表示字符串的连接  等同于||
select concat('my name is ', ename) from emp;
--将字符串的首字母大写
select initcap(ename) from emp;
--将字符串全部转换为大写
select upper(ename) from emp;
--将字符串全部转换为小写
select lower(ename) from emp;
--填充字符串
select lpad(ename,10,'*') from emp;
select rpad(ename,10,'*') from emp;
--去除空格
select trim(ename) from emp;
select ltrim(ename) from emp;
select rtrim(ename) from emp;
--查找指定字符串的位置
select instr('ABABCDEF','A') from emp;
--查看字符串的长度
select length(ename) from emp;
--截取字符串的操作
select substr(ename,0,2) from emp;
--替换操作
select replace('ababefg','ab','hehe') from emp;

--数值函数
--给小数进行四舍五入操作，可以指定小数部分的位数
select round(123.123,2) from dual;
select round(123.128,2) from dual;
select round(-123.128,2) from dual;

--截断数据,按照位数去进行截取，但是不会进行四舍五入的操作
select trunc(123.128,2) from dual;
--取模操作
select mod(10,4) from dual;
select mod(-10,4) from dual;
--向上取整
select ceil(12.12) from dual;
--向下取整
select floor(13.99) from dual;
--取绝对值
select abs(-100) from dual;
--获取正负值
select sign(-100) from dual;
--x的y次幂
select power(2,3) from dual;

--日期函数
select sysdate from dual;
select current_date from dual;
--add_months,添加指定的月份
select add_months(hiredate,2),hiredate from emp;
--返回输入日期所在月份的最后一天
select last_day(sysdate) from dual;
--两个日期相间隔的月份
select months_between(sysdate,hiredate) from emp;
--返回四舍五入的第一天
select sysdate 当时日期,
round(sysdate) 最近0点日期,
round(sysdate,'day') 最近星期日,
round(sysdate,'month') 最近月初,
round(sysdate,'q') 最近季初日期, 
round(sysdate,'year') 最近年初日期 from dual;
--返回下周的星期几
select next_day(sysdate,'星期一') from dual;
--提取日期中的时间
select 
extract(hour from timestamp '2001-2-16 2:38:40 ' ) 小时,
extract(minute from timestamp '2001-2-16 2:38:40 ' ) 分钟,
extract(second from timestamp '2001-2-16 2:38:40 ' ) 秒,
extract(DAY from timestamp '2001-2-16 2:38:40 ' ) 日,
extract(MONTH from timestamp '2001-2-16 2:38:40 ' ) 月,
extract(YEAR from timestamp '2001-2-16 2:38:40 ' ) 年
 from dual;
--返回日期的时间戳
select localtimestamp from dual;
select current_date from dual;
select current_timestamp from dual;
--给指定的时间单位增加数值
select
trunc(sysdate)+(interval '1' second), --加1秒(1/24/60/60)
trunc(sysdate)+(interval '1' minute), --加1分钟(1/24/60)
trunc(sysdate)+(interval '1' hour), --加1小时(1/24)
trunc(sysdate)+(INTERVAL '1' DAY),  --加1天(1)
trunc(sysdate)+(INTERVAL '1' MONTH), --加1月
trunc(sysdate)+(INTERVAL '1' YEAR), --加1年
trunc(sysdate)+(interval '01:02:03' hour to second), --加指定小时到秒
trunc(sysdate)+(interval '01:02' minute to second), --加指定分钟到秒
trunc(sysdate)+(interval '01:02' hour to minute), --加指定小时到分钟
trunc(sysdate)+(interval '2 01:02' day to minute) --加指定天数到分钟
from dual;


/*

转换函数
     在oracle中存在数值的隐式转换和显式转换
     隐式转换指的是字符串可以转换为数值或者日期
显式转换：
    to_char: 当由数值或者日期转成字符串的时候，必须要规定格式
*/
select '999'+10 from dual;
--date ：to_char
select to_char(sysdate,'YYYY-MI-SS HH24:MI:SS') from dual;
-- number : to_char
select to_char(123.456789,'9999') from dual;
select to_char(123.456789,'0000.00') from dual;
select to_char(123.456789,'$0000.00') from dual;
select to_char(123.456789,'L0000.00') from dual;
select to_char(123456789,'999,999,999,999') from dual;
--to_date:转换之后都是固定的格式
select to_date('2019/10/10 10:10:10','YYYY-MM-DD HH24:MI:SS') from dual;
--to_number:转成数字
select to_number('123,456,789','999,999,999') from dual;


--显示没有上级管理的公司首脑
select ename,nvl(to_char(mgr),'boss') from emp where mgr is null;
--显示员工雇佣期满6个月后下一个星期五的日期
select hiredate,next_day(add_months(hiredate,6),'星期五') from emp;

--条件函数
--decode,case when

--给不同部门的人员涨薪，10部门涨10%，20部门涨20%，30部门涨30%
select ename,sal,deptno,decode(deptno,10,sal*1.1,20,sal*1.2,30,sal*1.3) from emp;
select ename,
       sal,
       deptno,
       case deptno
         when 10 then
          sal * 1.1
         when 20 then
          sal * 1.2
         when 30 then
          sal * 1.3
       end
  from emp;
------------------------------

create table test(
   id number(10) primary key,
   type number(10) ,
   t_id number(10),
   value varchar2(5)
);
insert into test values(100,1,1,'张三');
insert into test values(200,2,1,'男');
insert into test values(300,3,1,'50');

insert into test values(101,1,2,'刘二');
insert into test values(201,2,2,'男');
insert into test values(301,3,2,'30');

insert into test values(102,1,3,'刘三');
insert into test values(202,2,3,'女');
insert into test values(302,3,3,'10');

select * from test;
/*
需求
将表的显示转换为
姓名      性别     年龄
--------- -------- ----
张三       男        50
*/
select decode(type, 1, value) 姓名,
       decode(type, 2, value) 性别,
       decode(type, 3, value) 年龄
  from test;
select min(decode(type, 1, value)) 姓名,
       min(decode(type, 2, value)) 性别,
       min(decode(type, 3, value)) 年龄
  from test group by t_id; 

/*
组函数,一般情况下，组函数都要和groupby组合使用
组函数一般用于选择列表或者having条件判断
常用的组函数有5个
avg()  平均值,只用于数值类型的数据
min()  最小值，适用于任何类型
max()  最大值，适用于任何类型
count() 记录数,处理的时候会跳过空值而处理非空值
    count一般用来获取表中的记录条数，获取条数的时候可以使用*或者某一个具体的列
       甚至可以使用纯数字来代替，但是从运行效率的角度考虑，建议使用数字或者某一个具体的列
       而不要使用*
       
sum()   求和，只适合数值类型的数据
*/
select avg(sal) from emp;
select min(sal) from emp;
select max(sal) from emp;
select count(sal) from emp;
select sum(sal) from emp;
--group by,按照某些相同的值去进行分组操作
--group进行分组操作的时候，可以指定一个列或者多个列，但是当使用了groupby 之后，
--选择列表中只能包含组函数的值或者group by 的普通字段
--求每个部门的平均薪水
select avg(sal) from emp group by deptno;
--求平均新书大于2000的部门
select avg(sal),deptno from emp where sal is not null group by deptno having avg(sal) >2000 order by avg(sal);

select count(10000) from emp;
--部门下雇员的工资>2000 人数
select deptno,count(1) from emp where sal>2000 group by deptno
--部门薪水最高
select deptno,max(sal) from emp group by deptno;
--部门里面 工龄最小和最大的人找出来,知道姓名
select deptno,min(hiredate),max(hiredate) from emp group by deptno;
select ename, deptno
  from emp e
 where hiredate in (select min(hiredate) from emp group by deptno)
    or hiredate in (select max(hiredate) from emp group by deptno)
select * from emp

select mm2.deptno, e1.ename, e1.hiredate
  from emp e1,
       (select min(e.hiredate) mind, max(e.hiredate) maxd, e.deptno
          from emp e
         group by e.deptno) mm2
 where (e1.hiredate = mm2.mind
    or e1.hiredate = mm2.maxd)
    and e1.deptno = mm2.deptno;
```

### 7、MySQL SQL  关联查询

```sql
--关联查询
/*
select t1.c1,t2.c2 from t1,t2 where t1.c3 = t2.c4
在进行连接的时候，可以使用等值连接，可以使用非等值连接

*/
--查询雇员的名称和部门的名称
select ename,dname from emp,dept where emp.deptno = dept.deptno;
--查询雇员名称以及自己的薪水等级
select e.ename,e.sal,sg.grade from emp e,salgrade sg where e.sal between sg.losal and sg.hisal;

--等值连接，两个表中包含相同的列名
--非等值连接，两个表中没有相同的列名，但是某一个列在另一张表的列的范围之中
--外连接
select * from emp;
select * from dept;
--需要将雇员表中的所有数据都进行显示,利用等值连接的话只会把关联到的数据显示，
--没有关联到的数据不会显示，此时需要外连接
--分类：左外连接（把左表的全部数据显示）和右外连接（把右表的全部数据显示）
select * from emp e,dept d where e.deptno = d.deptno；--等值连接
select * from emp e,dept d where e.deptno = d.deptno(+);--左外连接
select * from emp e,dept d where e.deptno(+) = d.deptno;--右外连接
--自连接,将一张表当成不同的表来看待，自己关联自己
--将雇员和他经理的名称查出来
select e.ename,m.ename from emp e,emp m where e.mgr = m.empno;
--笛卡尔积,当关联多张表，但是不指定连接条件的时候，会进行笛卡尔积，
--关联后的总记录条数为M*n，一般不要使用
select * from emp e,dept d;

--92的表连接语法有什么问题？？？？
--在92语法中，多张表的连接条件会方法where子句中，同时where需要对表进行条件过滤
--因此，相当于将过滤条件和连接条件揉到一起，太乱了，因此出现了99语法
```

### 8、MySQL SQL  行转列

```sql
create table tmp(rq varchar2(10),shengfu varchar2(5));

insert into tmp values('2005-05-09','胜');
insert into tmp values('2005-05-09','胜');
insert into tmp values('2005-05-09','负');
insert into tmp values('2005-05-09','负');
insert into tmp values('2005-05-10','胜');
insert into tmp values('2005-05-10','负');
insert into tmp values('2005-05-10','负');

/*
          胜 负
2005-05-09 2 2
2005-05-10 1 2

*/

select rq,decode(shengfu,'胜',1),decode(shengfu,'负',2) from tmp;

select rq,
       count(decode(shengfu, '胜', 1)) 胜,
       count(decode(shengfu, '负', 2)) 负
  from tmp
 group by rq;


create table STUDENT_SCORE
(
  name    VARCHAR2(20),
  subject VARCHAR2(20),
  score   NUMBER(4,1)
);
insert into student_score (NAME, SUBJECT, SCORE) values ('张三', '语文', 78.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('张三', '数学', 88.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('张三', '英语', 98.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('李四', '语文', 89.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('李四', '数学', 76.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('李四', '英语', 90.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('王五', '语文', 99.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('王五', '数学', 66.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('王五', '英语', 91.0);


/*
姓名   语文  数学  英语
王五    89    56    89
*/
--至少使用4中方式下写出
--decode
select ss.name,
       max(decode(ss.subject, '语文', ss.score)) 语文,
       max(decode(ss.subject, '数学', ss.score)) 数学,
       max(decode(ss.subject, '英语', ss.score)) 英语
  from student_score ss group by ss.name
--case when
select ss.name,
       max(case ss.subject
             when '语文' then
              ss.score
           end) 语文,
       max(case ss.subject
             when '数学' then
              ss.score
           end) 数学,
       max(case ss.subject
             when '英语' then
              ss.score
           end) 英语
  from student_score ss
 group by ss.name;
--join
select ss.name,ss.score from student_score ss where ss.subject='语文';
select ss.name,ss.score from student_score ss where ss.subject='数学';
select ss.name,ss.score from student_score ss where ss.subject='英语';

select ss01.name, ss01.score 语文, ss02.score 数学, ss03.score 英语
  from (select ss.name, ss.score
          from student_score ss
         where ss.subject = '语文') ss01
  join (select ss.name, ss.score
          from student_score ss
         where ss.subject = '数学') ss02
    on ss01.name = ss02.name
  join (select ss.name, ss.score
          from student_score ss
         where ss.subject = '英语') ss03
    on ss01.name = ss03.name;

--union all
select t.name,sum(t.语文),sum(t.数学),sum(t.英语) from (select ss01.name,ss01.score 语文,0 数学,0 英语 from student_score ss01 where ss01.subject='语文' union all
select ss02.name,0 语文,ss02.score 数学,0 英语 from student_score ss02 where ss02.subject='数学' union all
select ss03.name,0 语文,0 数学,ss03.score 英语 from student_score ss03 where ss03.subject='英语') t group by t.name
```

### 9、MySQL SQL  视图

```sql
/*
CREATE [OR REPLACE] VIEW view
[(alias[, alias]...)]
AS subquery
[WITH READ ONLY];

*/
--如果普通用户第一次创建视图，提示没有权限，要使用管理员去修改权限
grant create view to scott;

--创建视图
create view v_emp as select * from emp where deptno = 30;
--视图的使用
select * from v_emp;
--向视图中添加数据,执行成功之后，需要提交事务，绿色表示提交事务，让数据生效，红色表示回滚事务，让数据恢复原状态
insert into v_emp(empno,ename) values(1111,'zhangsan');
select * from emp;
--如果定义的视图是非只读视图的话，可以通过视图向表中插入数据，如果是只读视图，则不可以插入数据
create view v_emp2 as select * from emp with read only;
select * from v_emp2;
--只读视图只提供查询的需求，无法进行增删改操作
insert into v_emp2(empno,ename) values(1234,'lisi');
--删除视图
drop view v_emp2;
--当删除视图中的数据的时候，如果数据来源于多个基表，则此时不能全部进行删除，只能删除一个表中的数据

--我们要求平均薪水的等级最低的部门，它的部门名称是什么，我们完全使用子查询
--1、求平均薪水
select e.deptno,avg(e.sal) from emp e group by e.deptno;
--2、求平均薪水的等级
select t.deptno,sg.grade gd
  from salgrade sg
  join (select e.deptno, avg(e.sal) vsal from emp e group by e.deptno) t
    on t.vsal between sg.losal and sg.hisal;
--3、求平均薪水的等级最低的部门
select min(t.gd) from (select t.deptno,sg.grade gd
  from salgrade sg
  join (select e.deptno, avg(e.sal) vsal from emp e group by e.deptno) t
    on t.vsal between sg.losal and sg.hisal) t
--4、求平均薪水的等级最低的部门的部门名称

select d.dname, d.deptno
  from dept d
  join (select t.deptno, sg.grade gd
          from salgrade sg
          join (select e.deptno, avg(e.sal) vsal from emp e group by e.deptno) t
            on t.vsal between sg.losal and sg.hisal) t
    on t.deptno = d.deptno
 where t.gd =
       (select min(t.gd)
          from (select t.deptno, sg.grade gd
                  from salgrade sg
                  join (select e.deptno, avg(e.sal) vsal
                         from emp e
                        group by e.deptno) t
                    on t.vsal between sg.losal and sg.hisal) t);
--查看sql语句能够发现，sql中有很多的重复的sql子查询，可以通过视图将重复的语句给抽象出来
--创建视图
create view v_deptno_grade as select t.deptno, sg.grade gd
          from salgrade sg
          join (select e.deptno, avg(e.sal) vsal from emp e group by e.deptno) t
            on t.vsal between sg.losal and sg.hisal;
--使用视图替换

select d.dname, d.deptno
  from dept d
  join v_deptno_grade t
    on t.deptno = d.deptno
 where t.gd =
       (select min(t.gd)
          from v_deptno_grade t);
```

### 10、MySQL SQL  序列

```sql
--在oracle中如果需要完成一个列的自增操作，必须要使用序列
/*
create sequence seq_name
  increment by n  每次增长几
  start with n    从哪个值开始增长
  maxvalue n|nomaxvalue 10^27 or -1  最大值
  minvalue n|no minvalue  最小值
	cycle|nocycle           是否有循环
	cache n|nocache          是否有缓存

*/
create sequence my_sequence
increment by 2
start with 1

--如何使用？
--注意，如果创建好序列之后，没有经过任何的使用，那么不能获取当前的值，必须要先执行nextval之后才能获取当前值
--dual是oracle中提供的一张虚拟表，不表示任何意义，在测试的时候可以随意使用
--查看当前序列的值
select my_sequence.currval from dual;
--获取序列的下一个值
select my_sequence.nextval from dual;

insert into emp(empno,ename) values(my_sequence.nextval,'hehe');
select * from emp;
```

### 11、MySQL SQL  数据更新

```sql
--DML：数据库操作语言
--增
--删
--改

--在实际项目中，使用最多的是读取操作，但是插入数据和删除数据同等重要，而修改操作相对较少

/*
插入操作：
  元组值的插入
  查询结果的插入

*/
--最基本的插入方式
--insert into tablename values(val1,val2,....) 如果表名之后没有列，那么只能将所有的列都插入
--insert into tablename(col1,col2,...) values(val1,val2,...) 可以指定向哪些列中插入数据

insert into emp values(2222,'haha','clerk',7902,to_date('2019-11-2','YYYY-MM-dd'),1000,500,10);
select * from emp;
--向部分列插入数据的时候，不是想向哪个列插入就插入的，要遵循创建表的时候定义的规范
insert into emp(empno,ename) values(3333,'wangwu')

--创建表的其他方式
--复制表同时复制表数据，不会复制约束
create table emp2 as select * from emp;
--复制表结构但是不复制表数据，不会复制约束
create table emp3 as select * from emp where 1=2;
--如果有一个集合的数据，把集合中的所有数据都挨条插入的话，效率如何？一般在实际的操作中，很少一条条插入，更多的是批量插入

/*
删除操作：
 delete from tablename where condition

*/
--删除满足条件的数据
delete from emp2 where deptno = 10;
--把整张表的数据全部清空
delete from emp2;
--truncate ,跟delete有所不同，delete在进行删除的时候经过事务，而truncate不经过事务，一旦删除就是永久删除，不具备回滚的操作
--效率比较高，但是容易发生误操作，所以不建议使用
truncate table emp2

/*
修改操作：
   update tablename set col = val1,col2 = val2 where condition;
   可以更新或者修改满足条件的一个列或者多个列
*/
--更新单列
update emp set ename = 'heihei' where ename = 'hehe';
--更新多个列的值
update emp set job='teacher',mgr=7902 where empno = 15;


/*
增删改是数据库的常用操作，在进行操作的时候都需要《事务》的保证， 也就是说每次在pl/sql中执行sql语句之后都需要完成commit的操作
事务变得非常关键：
    最主要的目的是为了数据一致性
    如果同一份数据，在同一个时刻只能有一个人访问，就不会出现数据错乱的问题，但是在现在的项目中，更多的是并发访问
    并发访问的同时带来的就是数据的不安全，也就是不一致
    如果要保证数据的安全，最主要的方式就是加锁的方式，MVCC
    
    事务的延申：
        最基本的数据库事务
        声明式事务
        分布式事务
    为了提高效率，有可能多个操作会在同一个事务中执行，那么就有可能部分成功，部门失败，基于这样的情况就需要事务的控制。
    select * from emp where id = 7902 for update
    select * from emp where id = 7902 lock in share mode.
    
    如果不保证事务的话，会造成脏读，不可重复读，幻读。
*/
```

### 12、MySQL SQL  事物

```sql
--事务：表示操作集合，不可分割，要么全部成功，要么全部失败

--事务的开始取决于一个DML语句
/*
事务的结束
  1、正常的commit（使数据修改生效）或者rollback（将数据恢复到上一个状态）
  2、自动提交，但是一般情况下要将自动提交进行关闭，效率太低
  3、用户关闭会话之后，会自动提交事务
  4、系统崩溃或者断电的时候回回滚事务，也就是将数据恢复到上一个状态
*/
insert into emp(empno,ename) values(2222,'zhangsan');
--commit;
--rollback;
select * from emp;

--savepoint  保存点
--当一个操作集合中包含多条SQL语句，但是只想让其中某部分成功，某部分失败，此时可以使用保存点
--此时如果需要回滚到某一个状态的话使用 rollback to sp1;
delete from emp where empno = 1111;
delete from emp where empno = 2222;
savepoint sp1;
delete from emp where empno = 1234;
rollback to sp1;
commit;
/*
事务的四个特性：ACID
  原子性：表示不可分割，一个操作集合要么全部成功，要么全部失败，不可以从中间做切分
  一致性：最终是为了保证数据的一致性，当经过N多个操作之后，数据的状态不会改变（转账）
          从一个一致性状态到另一个一致性状态，也就是数据不可以发生错乱
  隔离性：各个事务之间相关不会产生影响，（隔离级别）
          严格的隔离性会导致效率降低，在某些情况下为了提高程序的执行效率，需要降低隔离的级别
          隔离级别：
            读未提交
            读已提交
            可重复读
            序列化
          数据不一致的问题：
            脏读
            不可重复读
            幻读
  持久性：所有数据的修改都必须要持久化到存储介质中，不会因为应用程序的关闭而导致数据丢失

  四个特性中，哪个是最关键的？
     所有的特性中都是为了保证数据的一致性，所以一致性是最终的追求
     事务中的一致性是通过原子性、隔离性、持久性来保证的

     锁的机制：
     为了解决在并发访问的时候，数据不一致的问题，需要给数据加锁
     加锁的同时需要考虑《粒度》的问题：
         操作的对象
            数据库
            表
            行
     一般情况下，锁的粒度越小，效率越高，粒度越大，效率越低 
            在实际的工作环境中，大部分的操作都是行级锁  

*/
```

### 13、MySQL SQL  索引

```sql
--索引：加快数据的检索
--创建索引
create index i_ename on emp(ename);
--删除索引
drop index i_ename;
select * from emp where ename = 'SMITH';
```

### 14、MySQL SQL  建表操作

```sql
/*

CREATE TABLE [schema.]table
  (column datatype [DEFAULT expr] , …
	);

*/

--设计要求：建立一张用来存储学生信息的表，表中的字段包含了学生的学号、姓名、年龄、入学日期、年级、班级、email等信息，
--并且为grade指定了默认值为1，如果在插入数据时不指定grade得值，就代表是一年级的学生

create table student
(
stu_id number(10),
name varchar2(20),
age number(3),
hiredate date,
grade varchar2(10) default 1,
classes varchar2(10),
email varchar2(50)
);
insert into student values(20191109,'zhangsan',22,to_date('2019-11-09','YYYY-MM-DD'),'2','1','123@qq.com');
insert into student(stu_id,name,age,hiredate,classes,email) values(20191109,'zhangsan',22,to_date('2019-11-09','YYYY-MM-DD'),'1','123@qq.com');

select * from student;
--正规的表结构设计需要使用第三方工具 powerdesigner
--再添加表的列的时候，不能允许设置成not null
alter table student add address varchar2(100);
alter table student drop column address;
alter table student modify(email varchar2(100));
--重新命名表
rename student to stu;
--删除表
/*
在删除表的时候，经常会遇到多个表关联的情况，多个表关联的时候不能随意删除，需要使用级联删除
cascade:如果A,B,A中的某一个字段跟B表中的某一个字段做关联，那么再删除表A的时候，需要先将表B删除
set null:再删除的时候，把表的关联字段设置成空
*/
 drop table stu;
 
 --创建表的时候可以给表中的数据添加数据校验规则，这些规则称之为约束
 /*
 约束分为五大类
 not null: 非空约束，插入数据的时候某些列不允许为空
 unique key:唯一键约束，可以限定某一个列的值是唯一的，唯一键的列一般被用作索引列。
 primary key:主键：非空且唯一，任何一张表一般情况下最好有主键，用来唯一的标识一行记录，
 foreign key:外键，当多个表之间有关联关系（一个表的某个列的值依赖与另一张表的某个值）的时候，需要使用外键
 check约束:可以根据用户自己的需求去限定某些列的值
 */
 --个人建议：再创建表的时候直接将各个表的约束条件添加好，如果包含外键约束的话，最好先把外键关联表的数据优先插入
 
 insert into emp(empno,ename,deptno) values(9999,'hehe',50);
 
 create table student
(
stu_id number(10) primary key,
name varchar2(20) not null,
age number(3) check(age>0 and age<126),
hiredate date,
grade varchar2(10) default 1,
classes varchar2(10),
email varchar2(50) unique,
deptno number(2)
);
insert into student(stu_id,name,age,hiredate,classes,email,deptno) values(20191109,'zhansgan',111,to_date('2019-11-09','YYYY-MM-DD'),'1','12443@qq.com',10);

alter table student add constraint fk_0001 foreign key(deptno) references dept(deptno);
```

### 15、SQL 示例

1. 表结构

   ```sql
   -- 1.学生表 
   student(s_id,s_name,s_birth,s_sex) –学生编号,学生姓名, 出生年月,学生性别 
   -- 2.课程表 
   course(c_id,c_name,t_id) – –课程编号, 课程名称, 教师编号 
   -- 3.教师表 
   Teacher(t_id,t_name) –教师编号,教师姓名 
   -- 4.成绩表 
   Score(s_id,c_id,s_score) –学生编号,课程编号,分数
   ```

2. 测试数据

   ```sql
   --建表
   --学生表
   CREATE TABLE `student`(
       `s_id` VARCHAR(20),
       `s_name` VARCHAR(20) NOT NULL DEFAULT '',
       `s_birth` VARCHAR(20) NOT NULL DEFAULT '',
       `s_sex` VARCHAR(10) NOT NULL DEFAULT '',
       PRIMARY KEY(`s_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
   --课程表
   CREATE TABLE `course`(
       `c_id`  VARCHAR(20),
       `c_name` VARCHAR(20) NOT NULL DEFAULT '',
       `t_id` VARCHAR(20) NOT NULL,
       PRIMARY KEY(`c_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
   --教师表
   CREATE TABLE `teacher`(
       `t_id` VARCHAR(20),
       `t_name` VARCHAR(20) NOT NULL DEFAULT '',
       PRIMARY KEY(`t_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
   --成绩表
   CREATE TABLE `score`(
       `s_id` VARCHAR(20),
       `c_id`  VARCHAR(20),
       `s_score` INT(3),
       PRIMARY KEY(`s_id`,`c_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
   --插入学生表测试数据
   insert into student values('01' , '赵雷' , '1990-01-01' , '男');
   insert into student values('02' , '钱电' , '1990-12-21' , '男');
   insert into student values('03' , '孙风' , '1990-05-20' , '男');
   insert into student values('04' , '李云' , '1990-08-06' , '男');
   insert into student values('05' , '周梅' , '1991-12-01' , '女');
   insert into student values('06' , '吴兰' , '1992-03-01' , '女');
   insert into student values('07' , '郑竹' , '1989-07-01' , '女');
   insert into student values('08' , '王菊' , '1990-01-20' , '女');
   --课程表测试数据
   insert into course values('01' , '语文' , '02');
   insert into course values('02' , '数学' , '01');
   insert into course values('03' , '英语' , '03');
   --教师表测试数据
   insert into teacher values('01' , '张三');
   insert into teacher values('02' , '李四');
   insert into teacher values('03' , '王五');
   --成绩表测试数据
   insert into score values('01' , '01' , 80);
   insert into score values('01' , '02' , 90);
   insert into score values('01' , '03' , 99);
   insert into score values('02' , '01' , 70);
   insert into score values('02' , '02' , 60);
   insert into score values('02' , '03' , 80);
   insert into score values('03' , '01' , 80);
   insert into score values('03' , '02' , 80);
   insert into score values('03' , '03' , 80);
   insert into score values('04' , '01' , 50);
   insert into score values('04' , '02' , 30);
   insert into score values('04' , '03' , 20);
   insert into score values('05' , '01' , 76);
   insert into score values('05' , '02' , 87);
   insert into score values('06' , '01' , 31);
   insert into score values('06' , '03' , 34);
   insert into score values('07' , '02' , 89);
   insert into score values('07' , '03' , 98);
   ```

3. 示例题

   ```sql
   -- 1、查询"01"课程比"02"课程成绩高的学生的信息及课程分数  
   select a.* ,b.s_score as 01_score,c.s_score as 02_score from 
       student a 
       join Score b on a.s_id=b.s_id and b.c_id='01'
       left join Score c on a.s_id=c.s_id and c.c_id='02' or c.c_id = NULL where b.s_score>c.s_score
   ```

   ```sql
   -- 2、查询"01"课程比"02"课程成绩低的学生的信息及课程分数 
   select a.* ,b.s_score as 01_score,c.s_score as 02_score from 
       student a left join score b on a.s_id=b.s_id and b.c_id='01' or b.c_id=NULL 
        join score c on a.s_id=c.s_id and c.c_id='02' where b.s_score<c.s_score
   ```

   ```sql
   -- 3、查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩
   select b.s_id,b.s_name,ROUND(AVG(a.s_score),2) as avg_score from 
       student b 
       join score a on b.s_id = a.s_id
       GROUP BY b.s_id,b.s_name HAVING ROUND(AVG(a.s_score),2)>=60; 
   ```

   ```sql
   -- 4、查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩
           -- (包括有成绩的和无成绩的) 
   select b.s_id,b.s_name,ROUND(AVG(a.s_score),2) as avg_score from 
       student b 
       left join score a on b.s_id = a.s_id
       GROUP BY b.s_id,b.s_name HAVING ROUND(AVG(a.s_score),2)<60
       union
   select a.s_id,a.s_name,0 as avg_score from 
       student a 
       where a.s_id not in (
                   select distinct s_id from score);
   ```

   ```sql
   -- 5、查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩
   select a.s_id,a.s_name,count(b.c_id) as sum_course,sum(b.s_score) as sum_score from 
       student a 
       left join score b on a.s_id=b.s_id
       GROUP BY a.s_id,a.s_name;
   ```

   ```sql
   -- 6、查询"李"姓老师的数量 
   select count(t_id) from teacher where t_name like '李%';
   ```

   ```sql
   -- 7、查询学过"张三"老师授课的同学的信息 
   select a.* from 
       student a 
       join score b on a.s_id=b.s_id where b.c_id in(
           select c_id from course where t_id =(
               select t_id from teacher where t_name = '张三'));
   ```

   ```sql
   -- 8、查询没学过"张三"老师授课的同学的信息 
   select * from 
       student c 
       where c.s_id not in(
           select a.s_id from student a join score b on a.s_id=b.s_id where b.c_id in(
               select c_id from course where t_id =(
                   select t_id from teacher where t_name = '张三')));
   ```

   ```sql
   -- 9、查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息 
   select a.* from 
       student a,score b,score c 
       where a.s_id = b.s_id  and a.s_id = c.s_id and b.c_id='01' and c.c_id='02';
   ```

   ```sql
   -- 10、查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息
   select a.* from 
       student a 
       where a.s_id in (select s_id from score where c_id='01' ) and a.s_id not in(select s_id from score where c_id='02')
   ```

   ```sql
   -- 11、查询没有学全所有课程的同学的信息 
   select s.* from 
       student s where s.s_id in(
           select s_id from score where s_id not in(
               select a.s_id from score a 
                   join score b on a.s_id = b.s_id and b.c_id='02'
                   join score c on a.s_id = c.s_id and c.c_id='03'
               where a.c_id='01'))
   ```

   ```sql
   -- 12、查询至少有一门课与学号为"01"的同学所学相同的同学的信息 
   select * from student where s_id in(
       select distinct a.s_id from score a where a.c_id in(select a.c_id from score a where a.s_id='01')
       );
   ```

   ```sql
   -- 13、查询和"01"号的同学学习的课程完全相同的其他同学的信息  
   select a.* from student a where a.s_id in(
       select distinct s_id from score where s_id!='01' and c_id in(select c_id from score where s_id='01')
       group by s_id 
       having count(1)=(select count(1) from score where s_id='01'));
   ```

   ```sql
   -- 14、查询没学过"张三"老师讲授的任一门课程的学生姓名 
   select a.s_name from student a where a.s_id not in (
       select s_id from score where c_id = 
                   (select c_id from course where t_id =(
                       select t_id from teacher where t_name = '张三')) 
                   group by s_id);
   ```

   ```sql
   -- 15、查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩 
   select a.s_id,a.s_name,ROUND(AVG(b.s_score)) from 
       student a 
       left join score b on a.s_id = b.s_id
       where a.s_id in(
               select s_id from score where s_score<60 GROUP BY  s_id having count(1)>=2)
       GROUP BY a.s_id,a.s_name 
   ```

   ```sql
   -- 16、检索"01"课程分数小于60，按分数降序排列的学生信息
   select a.*,b.c_id,b.s_score from 
       student a,score b 
       where a.s_id = b.s_id and b.c_id='01' and b.s_score<60 ORDER BY b.s_score DESC;
   ```

   ```sql
   -- 17、按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
   select a.s_id,(select s_score from score where s_id=a.s_id and c_id='01') as 语文,
                   (select s_score from score where s_id=a.s_id and c_id='02') as 数学,
                   (select s_score from score where s_id=a.s_id and c_id='03') as 英语,
               round(avg(s_score),2) as 平均分 from score a  GROUP BY a.s_id ORDER BY 平均分 DESC;
   ```

   ```sql
   -- 18.查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
   --及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
   select a.c_id,b.c_name,MAX(s_score),MIN(s_score),ROUND(AVG(s_score),2),
       ROUND(100*(SUM(case when a.s_score>=60 then 1 else 0 end)/SUM(case when a.s_score then 1 else 0 end)),2) as 及格率,
       ROUND(100*(SUM(case when a.s_score>=70 and a.s_score<=80 then 1 else 0 end)/SUM(case when a.s_score then 1 else 0 end)),2) as 中等率,
       ROUND(100*(SUM(case when a.s_score>=80 and a.s_score<=90 then 1 else 0 end)/SUM(case when a.s_score then 1 else 0 end)),2) as 优良率,
       ROUND(100*(SUM(case when a.s_score>=90 then 1 else 0 end)/SUM(case when a.s_score then 1 else 0 end)),2) as 优秀率
       from score a left join course b on a.c_id = b.c_id GROUP BY a.c_id,b.c_name
   ```

   ```sql
   -- 19、按各科成绩进行排序，并显示排名(实现不完全)
   -- mysql没有rank函数
       select a.s_id,a.c_id,
           @i:=@i +1 as i保留排名,
           @k:=(case when @score=a.s_score then @k else @i end) as rank不保留排名,
           @score:=a.s_score as score
       from (
           select s_id,c_id,s_score from score WHERE c_id='01' GROUP BY s_id,c_id,s_score ORDER BY s_score DESC
   )a,(select @k:=0,@i:=0,@score:=0)s
       union
       select a.s_id,a.c_id,
           @i:=@i +1 as i,
           @k:=(case when @score=a.s_score then @k else @i end) as rank,
           @score:=a.s_score as score
       from (
           select s_id,c_id,s_score from score WHERE c_id='02' GROUP BY s_id,c_id,s_score ORDER BY s_score DESC
   )a,(select @k:=0,@i:=0,@score:=0)s
       union
       select a.s_id,a.c_id,
           @i:=@i +1 as i,
           @k:=(case when @score=a.s_score then @k else @i end) as rank,
           @score:=a.s_score as score
       from (
           select s_id,c_id,s_score from score WHERE c_id='03' GROUP BY s_id,c_id,s_score ORDER BY s_score DESC
   )a,(select @k:=0,@i:=0,@score:=0)s
   ```

   ```sql
   -- 20、查询学生的总成绩并进行排名
   select a.s_id,
       @i:=@i+1 as i,
       @k:=(case when @score=a.sum_score then @k else @i end) as rank,
       @score:=a.sum_score as score
   from (select s_id,SUM(s_score) as sum_score from score GROUP BY s_id ORDER BY sum_score DESC)a,
       (select @k:=0,@i:=0,@score:=0)s
   ```

   ```sql
   -- 21、查询不同老师所教不同课程平均分从高到低显示 
       select a.t_id,c.t_name,a.c_id,ROUND(avg(s_score),2) as avg_score from course a
           left join score b on a.c_id=b.c_id 
           left join teacher c on a.t_id=c.t_id
           GROUP BY a.c_id,a.t_id,c.t_name ORDER BY avg_score DESC;
   ```

   ```sql
   -- 22、查询所有课程的成绩第2名到第3名的学生信息及该课程成绩 
               select d.*,c.排名,c.s_score,c.c_id from (
                   select a.s_id,a.s_score,a.c_id,@i:=@i+1 as 排名 from score a,(select @i:=0)s where a.c_id='01'    
               )c
               left join student d on c.s_id=d.s_id
               where 排名 BETWEEN 2 AND 3
               UNION
               select d.*,c.排名,c.s_score,c.c_id from (
                   select a.s_id,a.s_score,a.c_id,@j:=@j+1 as 排名 from score a,(select @j:=0)s where a.c_id='02'    
               )c
               left join student d on c.s_id=d.s_id
               where 排名 BETWEEN 2 AND 3
               UNION
               select d.*,c.排名,c.s_score,c.c_id from (
                   select a.s_id,a.s_score,a.c_id,@k:=@k+1 as 排名 from score a,(select @k:=0)s where a.c_id='03'    
               )c
               left join student d on c.s_id=d.s_id
               where 排名 BETWEEN 2 AND 3;
   ```

   ```sql
   -- 23、统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比
           select distinct f.c_name,a.c_id,b.`85-100`,b.百分比,c.`70-85`,c.百分比,d.`60-70`,d.百分比,e.`0-60`,e.百分比 from score a
                   left join (select c_id,SUM(case when s_score >85 and s_score <=100 then 1 else 0 end) as `85-100`,
                                               ROUND(100*(SUM(case when s_score >85 and s_score <=100 then 1 else 0 end)/count(*)),2) as 百分比
                                   from score GROUP BY c_id)b on a.c_id=b.c_id
                   left join (select c_id,SUM(case when s_score >70 and s_score <=85 then 1 else 0 end) as `70-85`,
                                               ROUND(100*(SUM(case when s_score >70 and s_score <=85 then 1 else 0 end)/count(*)),2) as 百分比
                                   from score GROUP BY c_id)c on a.c_id=c.c_id
                   left join (select c_id,SUM(case when s_score >60 and s_score <=70 then 1 else 0 end) as `60-70`,
                                               ROUND(100*(SUM(case when s_score >60 and s_score <=70 then 1 else 0 end)/count(*)),2) as 百分比
                                   from score GROUP BY c_id)d on a.c_id=d.c_id
                   left join (select c_id,SUM(case when s_score >=0 and s_score <=60 then 1 else 0 end) as `0-60`,
                                               ROUND(100*(SUM(case when s_score >=0 and s_score <=60 then 1 else 0 end)/count(*)),2) as 百分比
                                   from score GROUP BY c_id)e on a.c_id=e.c_id
                   left join course f on a.c_id = f.c_id
   ```

   ```sql
   -- 24、查询学生平均成绩及其名次 
           select a.s_id,
                   @i:=@i+1 as '不保留空缺排名',
                   @k:=(case when @avg_score=a.avg_s then @k else @i end) as '保留空缺排名',
                   @avg_score:=avg_s as '平均分'
           from (select s_id,ROUND(AVG(s_score),2) as avg_s from score GROUP BY s_id)a,(select @avg_score:=0,@i:=0,@k:=0)b;
   ```

   ```sql
   -- 25、查询各科成绩前三名的记录
               -- 1.选出b表比a表成绩大的所有组
               -- 2.选出比当前id成绩大的 小于三个的
           select a.s_id,a.c_id,a.s_score from score a 
               left join score b on a.c_id = b.c_id and a.s_score<b.s_score
               group by a.s_id,a.c_id,a.s_score HAVING COUNT(b.s_id)<3
               ORDER BY a.c_id,a.s_score DESC 
   ```

   ```sql
   -- 26、查询每门课程被选修的学生数  
           select c_id,count(s_id) from score a GROUP BY c_id 
   ```

   ```sql
   -- 27、查询出只有两门课程的全部学生的学号和姓名 
           select s_id,s_name from student where s_id in(
                   select s_id from score GROUP BY s_id HAVING COUNT(c_id)=2); 
   ```

   ```sql
   -- 28、查询男生、女生人数 
           select s_sex,COUNT(s_sex) as 人数  from student GROUP BY s_sex
   ```

   ```sql
   -- 29、查询名字中含有"风"字的学生信息
           select * from student where s_name like '%风%';
   ```

   ```sql
   -- 30、查询同名同性学生名单，并统计同名人数 
           select a.s_name,a.s_sex,count(*) from student a  JOIN 
                       student b on a.s_id !=b.s_id and a.s_name = b.s_name and a.s_sex = b.s_sex
           GROUP BY a.s_name,a.s_sex
   ```

   ```sql
   -- 31、查询1990年出生的学生名单 
           select s_name from student where s_birth like '1990%' 
   ```

   ```sql
   -- 32、查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列 
    
       select c_id,ROUND(AVG(s_score),2) as avg_score from score GROUP BY c_id ORDER BY avg_score DESC,c_id ASC
   ```

   ```sql
   -- 33、查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩 
    
       select a.s_id,b.s_name,ROUND(avg(a.s_score),2) as avg_score from score a
           left join student b on a.s_id=b.s_id GROUP BY s_id HAVING avg_score>=85
   ```

   ```sql
   -- 34、查询课程名称为"数学"，且分数低于60的学生姓名和分数 
    
           select a.s_name,b.s_score from score b LEFT JOIN student a on a.s_id=b.s_id where b.c_id=(
                       select c_id from course where c_name ='数学') and b.s_score<60
   ```

   ```sql
   -- 35、查询所有学生的课程及分数情况； 
           select a.s_id,a.s_name,
                       SUM(case c.c_name when '语文' then b.s_score else 0 end) as '语文',
                       SUM(case c.c_name when '数学' then b.s_score else 0 end) as '数学',
                       SUM(case c.c_name when '英语' then b.s_score else 0 end) as '英语',
                       SUM(b.s_score) as  '总分'
           from student a left join score b on a.s_id = b.s_id 
           left join course c on b.c_id = c.c_id 
           GROUP BY a.s_id,a.s_name
   ```

   ```sql
   -- 36、查询任何一门课程成绩在70分以上的姓名、课程名称和分数； 
               select a.s_name,b.c_name,c.s_score from course b left join score c on b.c_id = c.c_id
                   left join student a on a.s_id=c.s_id where c.s_score>=70
   ```

   ```sql
   -- 37、查询不及格的课程
           select a.s_id,a.c_id,b.c_name,a.s_score from score a left join course b on a.c_id = b.c_id
               where a.s_score<60 
   ```

   ```sql
   --38、查询课程编号为01且课程成绩在80分以上的学生的学号和姓名； 
           select a.s_id,b.s_name from score a LEFT JOIN student b on a.s_id = b.s_id
               where a.c_id = '01' and a.s_score>80
   ```

   ```sql
   -- 39、求每门课程的学生人数 
           select count(*) from score GROUP BY c_id; 
   ```

   ```sql
   -- 40、查询选修"张三"老师所授课程的学生中，成绩最高的学生信息及其成绩
           -- 查询老师id   
           select c_id from course c,teacher d where c.t_id=d.t_id and d.t_name='张三'
           -- 查询最高分（可能有相同分数）
           select MAX(s_score) from score where c_id='02'
           -- 查询信息
           select a.*,b.s_score,b.c_id,c.c_name from student a
               LEFT JOIN Score b on a.s_id = b.s_id
               LEFT JOIN course c on b.c_id=c.c_id
               where b.c_id =(select c_id from course c,Teacher d where c.t_id=d.t_id and d.t_name='张三')
               and b.s_score in (select MAX(s_score) from Score where c_id=(select c_id from course c,Teacher d where c.t_id=d.t_id and d.t_name='张三'))
   ```

   ```sql
   -- 41、查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩 
       select DISTINCT b.s_id,b.c_id,b.s_score from score a,score b where a.c_id != b.c_id and a.s_score = b.s_score
   ```

   ```sql
   -- 42、查询每门功成绩最好的前两名 
           -- 牛逼的写法
       select a.s_id,a.c_id,a.s_score from Score a
           where (select COUNT(1) from Score b where b.c_id=a.c_id and b.s_score>=a.s_score)<=2 ORDER BY a.c_id
   ```

   ```sql
   -- 43、统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列  
           select c_id,count(*) as total from score GROUP BY c_id HAVING total>5 ORDER BY total,c_id ASC
   ```

   ```sql
   -- 44、检索至少选修两门课程的学生学号 
           select s_id,count(*) as sel from score GROUP BY s_id HAVING sel>=2
   ```

   ```sql
   -- 45、查询选修了全部课程的学生信息 
           select * from student where s_id in(        
               select s_id from score GROUP BY s_id HAVING count(*)=(select count(*) from course))
   ```

   ```sql
   --46、查询各学生的年龄
       -- 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一
       select s_birth,(DATE_FORMAT(NOW(),'%Y')-DATE_FORMAT(s_birth,'%Y') - 
                   (case when DATE_FORMAT(NOW(),'%m%d')>DATE_FORMAT(s_birth,'%m%d') then 0 else 1 end)) as age
           from student;
   ```

   ```sql
   -- 47、查询本周过生日的学生
       select * from student where WEEK(DATE_FORMAT(NOW(),'%Y%m%d'))=WEEK(s_birth)
       select * from student where YEARWEEK(s_birth)=YEARWEEK(DATE_FORMAT(NOW(),'%Y%m%d'))
       select WEEK(DATE_FORMAT(NOW(),'%Y%m%d')) 
   ```

   ```sql
   -- 48、查询下周过生日的学生
       select * from student where WEEK(DATE_FORMAT(NOW(),'%Y%m%d'))+1 =WEEK(s_birth) 
   ```

   ```sql
   -- 49、查询本月过生日的学生
       select * from student where MONTH(DATE_FORMAT(NOW(),'%Y%m%d')) =MONTH(s_birth)
   ```

   ```sql
   -- 50、查询下月过生日的学生
       select * from student where MONTH(DATE_FORMAT(NOW(),'%Y%m%d'))+1 =MONTH(s_birth)
   ```

