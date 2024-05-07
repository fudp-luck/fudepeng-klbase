## 一、Zookeeper介绍及基本使用

### 1、官网

https://zookeeper.apache.org/

### 2、Zookeeper 简介

Zookeeper 是分布式应用程序的分布式协调服务，它公开了一组简单的原语，分布式应用程序可以基于这些原语来实现用于同步，并使用了按照文件系统熟悉的目录树结构样式设置的数据模型。

### 3、设计目标和集群

#### 【1】设计目标

1. ZooKeeper 很简单，它允许分布式进程通过共享的分层名称空间相互协调，该命名空间的组织方式类似于标准文件系统。名称空间由数据寄存器（在 ZooKeeper 看来，称为 znode）组成，它们类似于文件和目录。与设计用于存储的典型文件系统不同，ZooKeeper 数据保留在内存中，这意味着 ZooKeeper 可以实现高吞吐量和低延迟数。
2. ZooKeeper 实施对高性能，高可用性，严格有序访问加以重视。
3. ZooKeeper 的性能方面意味着它可以在大型的分布式系统中使用,可靠性方面使它不会成为单点故障,严格的排序意味着可以在客户端上实现复杂的同步原语。

#### 【2】集群

1. Zookeeper 类似于 Redis 的主从复制集群，也会存在单点故障问题，但 Zookeeper 是高性能、高可用的，所以它的单点故障可以很快的修复，官方压测为 200ms

  <img src="_media/middleware/Zookeeper/Zookeeper集群1.png"/>
2. 组成 ZooKeeper 服务的服务器都必须彼此了解，它们维护内存中的状态图像，以及持久存储中的事务日志和快照，只要大多数服务器可用，ZooKeeper 服务将可用。
3. 客户端连接到单个 ZooKeeper 服务器，客户端维护一个 TCP 连接，通过该连接发送请求，获取响应，获取监视事件并发送心跳，如果与服务器的 TCP 连接断开，则客户端将连接到其他服务器。

  <img src="_media/middleware/Zookeeper/Zookeeper集群2.png"/>
4. 如上所示，与 Redis 类似，集群情况下，写操作只能发生在主节点，读操作可以发生在任意节点，客户端可以随意连接任何一个节点，当发生写操作时要转到主节点上。
5. Zookeeper 读写分离，以读取为主的速度非常快，将客户端所有并发散落在集群的物理机上，在读写比为 10:1 的情况下效果最佳，官方压测 3 个节点可以支持 8w - 9w 的对外请求

### 4、数据模型和分层名称空间

ZooKeeper 提供的名称空间与标准文件系统的名称空间非常相似，名称是由斜杠（/）分隔的一系列路径元素。

ZooKeeper 命名空间中的每个节点都由路径标识。

<img src="_media/middleware/Zookeeper/Zookeeper的层次命名空间.png"/>

### 5、节点和临时节点

<img src="_media/middleware/Zookeeper/Zookeeper节点.png" alt="image-20210220171605432"  />

每一个 client 在访问 server 时都会有一个 session 会话来代表这个 client，当 client 结束或挂掉，其 session 会话也会消失。

Zookeeper 上面的临时节点就是由 session 会话来控制的

临时节点和持久节点都有序列节点的概念，序列节点不是一个单独的节点，它要么是持久的要么是临时的

参考 Redis 中分布式锁的方案（击穿方案），在 Redis 方案中需要引入多线程，一个线程持锁，一个线程监控过期时间，引入 Zookeeper 后可以通过临时节点 session 会话来控制，当一个请求取数据时，就在 zk 中创建 session 会话，不需要设置过期时间，只要这个请求还在，那么 session 就在，锁就一直在，当请求返回或挂掉，session 会话也会随之消失，让出锁，所以不需要引入多线程或单独写逻辑进行维护

节点上可以存储数据，大小为 1M ，是二进制安全的

### 6、Zookeeper 提供的保障

顺序一致性：来自客户端的更新将按照发送的顺序应用。
- 因为 Zookeeper 是单实例的，所以客户端发送的请求都要一个一个处理，类似 Redis
- 因为 Zookeeper 选择了相对简单的主从模型，所有的写都要发生在主机单点上，单点的好处在于发生的所有操作都由主机来决定
- 所以其顺序一致性由 zk 的主从模型来保证的

原子性：更新成功或失败。没有部分结果。
- 当客户端要在集群中创建一个 a ，那么主节点会把创建 a 的这个操作复制给集群中所有节点，要么所有节点都成功，要么所有节点都失败
- 但上述条件是强一致性，强一致性会破坏可用性，而 zk 是高可用的，所以应类似于 Redis 是最终一致性的，集群中过半复制成功就算写入成功了

单个系统映像：无论客户端连接到哪个服务器，客户端都将看到相同的服务视图。也就是说，即使客户端故障转移到具有相同会话的其他服务器，客户端也永远不会看到系统的较旧视图。

可靠性：应用更新后，此更新将一直持续到客户端覆盖更新为止。zk 是内存级别的，所以会有持久化日志来保证可靠性

及时性：确保系统的客户视图在特定时间范围内是最新的，表示最终一致性，可能在客户端连接某一个 follower 时，并没有找到 a，因为集群中同步过半就算写成功了，但这时客户端会调用它的同步命令，这个 follower 会在很短的时间内同步这个 a 并返回

### 7、简单的 API

1. `create`：在树中的某个位置创建一个节点
2. `delete`：删除节点
3. `exists`：测试某个节点是否存在于某个位置
4. `get data`：获取数据，从节点读取数据
5. `set data`：将数据写入节点
6. `get childre`：获取子节点，检索节点的子节点列表
7. `sync`：等待数据传播

### 8、安装

准备四台机器 node01、02、03、04，四个节点中全部安装 jdk1.8，并设置环境变量

`wget https://downloads.apache.org/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2.tar.gz `在一个节点（node01）解压，之后向其他节点分发

`tar xf apache-zookeeper-3.6.2.tar.gz ` 解压

`mkdir /opt/soft/`

`mv apache-zookeeper-3.6.2  /opt/soft    --> cd /opt/soft/apache-zookeeper-3.6.2`
- bin  中会有相应的脚本
- conf 中是配置
  - 其中 zoo.sample.cfg 是启动配置模板
  - cp zoo.sample.cfg zoo.cfg   复制后 zoo.cfg 作为使用的启动配置

`vi zoo.cfg`
- `tickTime=2000`  表示心跳时间 2秒，用来确定其他节点活着的时间
- `initLimit=10`  表示在 follower 和 leader 建立连接时，leader 允许 2000 * 10 ，20秒的初始延迟，超过这个时间就认为 follower 有问题，就取消连接
- `syncLimit=5`表示同步，当 leader 与 follower 同步时，如果有 2000 * 5，10秒没有反馈，就认为 follower 有问题
- `dataDir=/tmp/zookeeper` 表示持久化目录，最好修改这个目录 /var/zookeeper
- `clientPort=2181` 表示客户端连接服务时使用的端口号
- `maxClientCnxns=60`  表示当前启动的 server 允许的最大连接数
- 自定义配置：
  - `server.1=node01:2888:3888`
  - `server.2=node02:2888:3888`
  - `server.3=node03:2888:3888`
  - `server.4=node04:2888:3888`
  - zk 不像 Redis 的哨兵可以通过订阅去发现其他的哨兵，zk 中只能手动的将集群中有哪些节点写进来，它们的行数除以二加一（行数/2 + 1）就是过半数
  - 当 leader 挂掉后，或者在第一次启动没有 leader 的时候，会在 3888 端口中建立连接，之后再这个 socket 中进行投票选举，选举出来后 leader 会起一个 2888 端口，其他的节点会连接到 2888 的 leader 上，后续发生的事物是在 2888 这个端口上进行通信的
  - server.1... ，这个 ID 号表示在选举中会出现自己选自己的情况，但 zk 中各节点暴露出自己的 ID 后会选最大
  - 或较大的节点过半数称为 leader

`mkdir /var/zookeeper  --> cd /var/zookeeper`

`vi myid `
- 每台机器小写自己配置的 ID 号
- 内容： 1

`cd /opt`

`scp -r ./soft/  node02(节点名称):'pwd'`发起远程拷贝

修改每台机器中 myid 的内容为自己所代表的 ID 号
- `mkdir /var/zookeeper`
- `echo 2 > /var/zookeeper/myid`
- 第三台、第四台与其一致

使 zk 命令在任何地方使用，需要将其加入到环境变量中
- `vi /etc/profile`
  - `export ZOOKEEPER_HOME=/opt/soft/apache-zookeeper-3.6.2`
  - `export PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin`
- `. /etc/profile`  加载
- 在系统任何位置  zk + tab 就会有其指令

分发环境变量文件
- `scp /etc/profile   node02：/etc`
- `scp /etc/profile   node03：/etc`
- `scp /etc/profile   node04：/etc`
- 在会话下面的命令行输入 `source /etc/profile`   在 Xshell 右下角三条横杠，选中发送全部会话

启动
- `zkServer.sh help` 可以看启动项
- `zkServer.sh start-foreground `  可以使 zk 在前台阻塞，将日志在前台实时打印
- 当 01 - 04 顺序启动时，03 为 leader ，因为启动到 03 时已经有三台了，可以进行过半投票了
- zkCli.sh 可以启动 zk 的客户端，可以在任意节点上启动，默认连接的是自己

### 9、Cli 演示

每一个客户端开启会创建一个 sessionID

<img src="_media/middleware/Zookeeper/sessionID.png"/>

help 可以打印出所有支持使用的命令

ls /   显示根目录

在根目录下创建一个节点
- `create /ooxx " "`
- `create /ooxx/xxoo " "`
- `create -e `创建临时节点
- `create -s `创建序列节点
- `create -s -e` 创建临时序列节点

获取节点数据
- `get /ooxx`

  <img src="F:\Java\images.assets\Zookeeper.assets\get.png" alt="image-20210222175537030"  />
- `cZxid = 0x200000002`，表示创建 ooxx 时，其事物号为 2 号，因为 zk 是单节点，所有维护一个递增的 ID 号很容易，而且这个 ID 为 16 进制
- `ctime` ，表示创建时间
- `mZxid`，表示修改的事物 ID，03号 为上述创建 xxoo 的过程
- `pZxid`，表示当前节点下，创建的最后一个节点的事物 ID
- `ephemeralOwner`，表示所有者，0x0 表示当前节点没有归属谁，属于持久节点

6. 设置节点数据：`set /ooxx "hello"`
7. 临时节点是和 session 绑定的，session 统一视图也是要消耗事物 ID 的
8. create /abc ，当很多个客户端都要在 /abc 下创建目录存数据，那么在分布式的情况下就会出现覆盖问题
   - 可以 `create -s /abc/xxx "aaaaaa"`，这样创建出来的目录会在后面拼入 ID 数值，在并发情况下，其他客户端也在 /abc 下创建 /xxx ，那么 ID 数值会递增
   - <img src="F:\Java\images.assets\Zookeeper.assets\并发数值1.png" alt="image-20210224234712208"  />
   - <img src="F:\Java\images.assets\Zookeeper.assets\并发数值2.png" alt="image-20210224234744958"  />
   - `create -s` ，可以规避覆盖问题，也实现了分布式下的同一命名，只要客户端记住了 ID 数值即可
   - 当删除 /abc 下的目录后再次创建，ID 编号不会从 0 开始，会继续递增，因为 leader 内部会维护这个数字到几了

### 10、场景及理论支撑

<img src="_media/middleware/Zookeeper/场景理论支撑.png"/>

### 11、socket 连接模型

netstat -natp | egrep '(2888|3888)' ， 显示 () 符合正则的连接

node01

<img src="_media/middleware/Zookeeper/node01连接.png"/>

- 开启了 3888 端口，并收到了来自 node02/03/04 的连接，之后自己连接到 14，因为 14 是 leader

node02

<img src="_media/middleware/Zookeeper/node02连接.png"/>

- 开启了 3888 端口，并收到了 node03/04 的连接，之后自己连接过 node01 ，并连接到 14

node03

<img src="_media/middleware/Zookeeper/node03连接.png"/>

- 以此类推，收到了 node04 的连接

node04

<img src="_media/middleware/Zookeeper/node04连接.png"/>

连接模型

<img src="F:\Java\images.assets\Zookeeper.assets\连接模型.png" alt="image-20210225012656163"  />



## 二、Zookeeper原理及API开发

### 1、Zookeeper 特性

<img src="_media/middleware/Zookeeper/Zookeeper特性.png"/>

### 2、paxos 协议

资料：https://www.douban.com/note/208430424/

paxos 对 Zookeeper 在 leader 选举方面没有过多解释

Zookeeper 在分布式下对 paxos 做了更简单的实现，ZAB 更容易实现数据在可用状态下的同步

### 3、ZAB 协议

#### 【1】ookeeper 本身的原子广播协议

原子：要么成功，要么失败，没有中间状态（基于 队列 + 两阶段提交）

广播：分布式多节点的，不代表全部知道，强调过半通过

队列：先进先出，顺序性

ZAB 协议作用在服务可用状态（有 leader）

#### 【2】Zookeeper 可用状态主从模型流程

<img src="_media/middleware/Zookeeper/zk主从模型流程.png"/>

图中流程解释

  - `1、`client01 访问 follower01 发起 create /ooxx 操作
  - `2、`follower01将 create /ooxx 转给 leader 处理
  - `3、`leader Zxid 自增事物 ID
  - `4-1、`leader 通过内部维护的发送队列向两个 follower 发起写操作日志，创建节点成功后返回 ok
    - Zookeeper 数据存在内存，日志存在磁盘
    - 这一步只触发了向磁盘中写日志，并没有在内存中立刻创建节点。
  - `4-2、`向 follower01 发送 write 写提交，成功并返回 ok：因为此时 leader 和 follower01 此时已达成过半，所以即使 follower02 没有返回 ok 也会向其发送写提交，只要 follower02 最终将队列消费完并同步数据即可，这也是最终一致性的体现
  - `over ok` 最终 leader 会向 follower01 返回 ok，follower01 向 client01 返回 ok

注：当 client02 访问 follower02 时可能会出现数据不同步的情况，这时调用 sync 可以使其同步，sync 是可选项

如果 leader 还没来得及发送写操作就挂了，那么数据会回滚

#### 【3】Zookeeper 不可用状态主从模型

不可用状态的两种场景
1. 第一次启动集群
2. leader 挂掉后重启集群

集群节点的维度
1. 每个节点会有自己的 myid
2. 事物 id - Zxid

新 leader 选举依据：
1. 经验最丰富的 Zxid，拥有最大事物 id 的节点
2. myid 大的
3. 过半通过的数据才是真数据，只要见到可用的 Zxid，且挂掉的节点没有过半，那么这个 zxid 一定是被过半通过的

第一次启动集群模型

<img src="_media/middleware/Zookeeper/启动leader模型.png"/>

leader 挂掉后重启集群模型

<img src="_media/middleware/Zookeeper/leader重启.png"/>



例如：node04 是初始 leader，且在 leader 挂掉前 node03 没有完成同步，所以 zxid 还是上一次同步的数据，假设 node03 率先发现 leader 挂掉，并发起投票

node03 会将自己的 myid、zxid 发送到 node01/02 比较，并给自己投一票，此时 node01:0 ；node02:0 ；node03:1

node01/02 发起自己的投票，分别给自己投一票，之后比较 node03 发现 node03 的 zxid 小于自己，所以分别带着自己的 myid、zxid 驳回 node03，之后 node03 给 node01/02 投了一票 ，此时 node01:2 ；node02:2 ；node03:1

node01 带着自己的 myid、zxid 向 node02 发送比较，或者 node02 向 node01 发送比较，比较后发现二者 zxid 相同，node02 的 myid 大于 node01，所以 node01 给 node02 投一票，此时 node01:2 ；node02:3 ；node03:1，node02 当选 leader

### 4、watch 观察

<img src="_media/middleware/Zookeeper/watch观察.png"/>

当两个客互相提供服务，且要监控对方是否可用时
1. 可以在客户端开启一个线程做心跳验证
2. 通过 zk，使 client02 watch 监控a 节点（其实监控的是产生的事件），如果 client01 挂掉，那么 a 节点也会消失，并产生事件，之后会回调 client02

注：两者区别在于方向性和时效性，zk 方式要比心跳验证方式的时效性要好，假设每 3s 验证一次，也会存在将近 3s 的延迟，而 zk 会立刻回调

### 5、API & callback 

```java
// 依赖 注意：引入的版本要与服务的版本一致
<dependency>
   <groupId>org.apache.zookeeper</groupId>
   <artifactId>zookeeper</artifactId>
   <version>3.6.2</version>
</dependency>
  
public class ZookeeperApp {
    public static void main(String[] args) {
        /*
          1、zk 是有 session 概念的，没有连接池的概念，
          2、客户端不用准备连接池，每个连接会拿到一个独立的 session
          3、watch 观察回调 分为两类
            1、第一类：new zk 时候，传入的 watch，这个 watch 是session级别的，和 path、node 没有关系，
                它是收不到事件的，只能收到当前连接以及当别的 server 断的时候，连接别人的过程
            2、第二类：getData 时传入的 watch ，可以获取 path、node
          4、watch 的注册只发生在读类型的调用， get 、exites .....
         */
        CountDownLatch countDownLatch = new CountDownLatch(1);// 使程序在完成 zk 连接时回调
        String server = "192.168.60.132:2181,192.168.60.133:2181,192.168.60.134:2181,192.168.60.135:2181";
        Integer sessionTimeout = 3000; //会决定临时节点消失的时间
        try {
            ZooKeeper zooKeeper = new ZooKeeper(server, sessionTimeout, new Watcher() {
                // watch 的回调方法
                @Override
                public void process(WatchedEvent watchedEvent) {
                    Event.KeeperState state = watchedEvent.getState(); //连接状态
                    Event.EventType type = watchedEvent.getType(); // 事件类型
                    String path = watchedEvent.getPath();
                    System.out.println("new zk watch : "+watchedEvent.toString());
                    switch (state) {
                        case Unknown:
                            break;
                        case Disconnected:
                            break;
                        case NoSyncConnected:
                            break;
                        case SyncConnected:
                            System.out.println("connected");
                            break;
                        case AuthFailed:
                            break;
                        case ConnectedReadOnly:
                            break;
                        case SaslAuthenticated:
                            break;
                        case Expired:
                            break;
                        case Closed:
                            break;
                    }
                    switch (type) {
                        case None:
                            break;
                        case NodeCreated:
                            break;
                        case NodeDeleted:
                            break;
                        case NodeDataChanged:
                            break;
                        case NodeChildrenChanged:
                            break;
                        case DataWatchRemoved:
                            break;
                        case ChildWatchRemoved:
                            break;
                        case PersistentWatchRemoved:
                            break;
                    }
                }
            });
//            new ZooKeeper(server, sessionTimeout, (watchedEvent)->{ });  // lambda

            countDownLatch.await();
            ZooKeeper.States state = zooKeeper.getState();
            switch (state) {
                case CONNECTING:
                    // 启动后被打印，zk 是异步处理的
                    System.out.println("ing ......");
                    break;
                case ASSOCIATING:
                    break;
                case CONNECTED:
                    System.out.println("connected");
                    countDownLatch.countDown();
                    break;
                case CONNECTEDREADONLY:
                    break;
                case CLOSED:
                    break;
                case AUTH_FAILED:
                    break;
                case NOT_CONNECTED:
                    break;
            }
            // 新增节点，同步阻塞
            String pathName = zooKeeper.create("/xxoo", "olddata".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);

            // 获取节点,同步阻塞
            Stat stat = new Stat();
            byte[] node = zooKeeper.getData("xxoo", new Watcher() {
                @Override
                public void process(WatchedEvent watchedEvent) {
                    System.out.println("getData watch: " + watchedEvent.toString());
                    try {
                        // 可以让 watch 重新注册，继续监控
                        // true default watch 被重新注册 是 new zk 的那个 watch
                        zooKeeper.getData("/xxoo",true,stat);
                        // this, 使用的是当前的 watch
//                        zooKeeper.getData("/xxoo",this,stat);
                    } catch (KeeperException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, stat);
            System.out.println(new String(node));

            // 修改节点数据, 会触发回调
            Stat stat1 = zooKeeper.setData("/xxoo", "newdata".getBytes(), 0);
            // 不会触发回调，watch 是一次性的，需要重新注册，参考上述 109 111 code
            Stat stat2 = zooKeeper.setData("/xxoo", "newdata01".getBytes(), stat1.getVersion());

            // 获取数据 ，异步回调
            System.out.println("-----------async start------------");
            zooKeeper.getData("/xxoo", false, new AsyncCallback.DataCallback() {
                @Override
                public void processResult(int i, String s, Object o, byte[] bytes, Stat stat) {
                    System.out.println("-----------async call back------------");
                    System.out.println(new String(bytes));
                    System.out.println(o.toString());
                }
            }, "abc");
            System.out.println("-----------async over------------");

        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }
}
```

## 三、Zookeeper 分布式协调案例

### 1、分布式配置

#### 【1】流程思路

当有一台或多台服务器，在启动时需要一些配置，以及它们在运行时有一些配置被变更了，它们需要第一时间获知，这时成本一定要降到最低，所以引入 Zookeeper

<img src="_media/middleware/Zookeeper/分布式配置思路.png"/>

我们要将服务器配置数据 data 存到 Zookeeper 中，服务器可以通过 get 取回 zk 中的配置信息的，同时通过 watch 监控，一旦外界对数据做出了修改，这个修改就会造成 watch 的回调

#### 【2】代码验证

TestConf  业务代码测试类

```java
package com.zookeeper.config;

import org.apache.zookeeper.ZooKeeper;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

public class TestConfig {

    ZooKeeper zk;

    @Before
    public void conn(){
        zk = ZKUtils.getZk();
    }

    @After
    public void close(){
        try {
            zk.close();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void getConf(){
        WatchCallBack watchCallBack = new WatchCallBack();
        watchCallBack.setZk(zk);
        MyConf myConf = new MyConf();
        watchCallBack.setConf(myConf);
        // 控制：只有得到数据后，再进行，否则阻塞
        watchCallBack.aWait();
        /*
        可能性：
        1、节点不存在
        2、节点存在
         */

        while (true){
            if (myConf.getConf().equals("")){
                System.out.println("conf diu le  ....");
                watchCallBack.aWait();
            } else {
                System.out.println(myConf.getConf());
            }
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

ZKUtils   Zookeeper 工具类 获取 zk 连接

```java
package com.zookeeper.config;

import org.apache.zookeeper.ZooKeeper;

import java.io.IOException;
import java.util.concurrent.CountDownLatch;

public class ZKUtils {
    private static ZooKeeper zk;

    // 使用 /testConf 作为根，其他真正根目录下与 testConf 平级的目录是看不到的
    private static String address =
       "192.168.60.132:2181,192.168.60.133:2181,192.168.60.134:2181,192.168.60.135:2181/testConf";

    private static DefaultWatch watch = new DefaultWatch();

    private static CountDownLatch init = new CountDownLatch(1);

    public static ZooKeeper getZk() {
        try {
            zk = new ZooKeeper(address, 1000, watch);
            watch.setCc(init);
            init.await();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return zk;
    }
}
```

DefaultWatch  默认的 watch ，zk连接时 session 级别的 watch

```java
package com.zookeeper.config;

import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;

import java.util.concurrent.CountDownLatch;

public class DefaultWatch implements Watcher {
    // session 级别的 watch  与 path 无关
    CountDownLatch cc;

    public void setCc(CountDownLatch cc) {
        this.cc = cc;
    }

    @Override
    public void process(WatchedEvent watchedEvent) {
        System.out.println(watchedEvent.toString());

        switch (watchedEvent.getState()) {
            case Unknown:
                break;
            case Disconnected:
                break;
            case NoSyncConnected:
                break;
            case SyncConnected:
                cc.countDown();
                break;
            case AuthFailed:
                break;
            case ConnectedReadOnly:
                break;
            case SaslAuthenticated:
                break;
            case Expired:
                break;
            case Closed:
                break;
        }
    }
}
```

WatchCallBack  封装了所有回调方法

```java
package com.zookeeper.config;

import org.apache.zookeeper.AsyncCallback;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;

import java.util.concurrent.CountDownLatch;

public class WatchCallBack implements Watcher,
        AsyncCallback.StatCallback,
        AsyncCallback.DataCallback {

    ZooKeeper zk;
    MyConf conf = new MyConf();
    CountDownLatch ccc = new CountDownLatch(1);

    // 得到数据前阻塞
    public void aWait() {
        zk.exists("/AppConf", this, this,"abc");
        try {
            ccc.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void process(WatchedEvent watchedEvent) {
        switch (watchedEvent.getType()) {
            case None:
                break;
            case NodeCreated:
                // 从启动到现在没有创建过这个节点
                // 在其它地方创建这个节点后，这个事件会被启动
                zk.getData("/AppConf", this, this,"abc");
                break;
            case NodeDeleted:
                // 节点被删除，这是一个容忍性的问题，
                // 被删除的前提是已经获取过这个节点的数据，如果节点删除不影响就不做处理
                // 如果一定要求数据一致性，那么就节点删除后，这个数据不能用了
                conf.setConf("");
                ccc = new CountDownLatch(1);
                break;
            case NodeDataChanged://节点被修改了，将数据再取一遍
                zk.getData("/AppConf", this, this,"abc");
                break;
            case NodeChildrenChanged:
                break;
            case DataWatchRemoved:
                break;
            case ChildWatchRemoved:
                break;
            case PersistentWatchRemoved:
                break;
        }
    }

    @Override
    public void processResult(int i, String path, Object o, byte[] data, Stat stat) {
        if(data != null){
            String s = new String(data);
            conf.setConf(s);
            ccc.countDown();
        }
    }

    @Override
    public void processResult(int i, String s, Object o, Stat stat) {
        if(stat != null){
            zk.getData("/AppConf",this, this, "xxx");
//            zk.getChildren()  // dubbo 可以将别人注册的子节点拿出来使用 ，数据为 ip
        }
    }

    public ZooKeeper getZk() {
        return zk;
    }

    public void setZk(ZooKeeper zk) {
        this.zk = zk;
    }

    public MyConf getConf() {
        return conf;
    }

    public void setConf(MyConf conf) {
        this.conf = conf;
    }
}
```

MyConf  封装业务所需数据

```java
package com.zookeeper.config;

// 这个 class 是未来最关心的地方，会有各种信息是业务需要的
public class MyConf {
    private String conf;

    public String getConf() {
        return conf;
    }

    public void setConf(String conf) {
        this.conf = conf;
    }
}
```

### 2、分布式锁

#### 【1】锁流程

当两个客户端需要使用同一个资源，在同一时间只能一个客户端访问，这时就需要锁，在实现分布式锁的技术中，Zookeeper 是最好的

<img src="F:\Java\images.assets\Zookeeper.assets\分布式锁.png" alt="image-20210227230940449" />

- 1- 争抢锁，只有一个客户端能够获得锁
- 2- 如果获得锁的挂掉了，zk 中采用临时节点（session 一消即消）解决
- 3- 获得锁的成功处理后，释放锁
- 4- 2 & 3 中锁的释放，其它人是怎么知道的？
  
  -   4.1、主动轮询、心跳检测，每隔一秒看一下是否被释放，弊端：存在延迟和压力（很多人同时心跳检测）
  -   4.2、watch 监控，回调，解决了延迟问题，弊端：存在压力（一旦释放，很多人同时回调抢锁）
  -   4.3、sequence + Enode + watch （临时序列节点 + watch监控），给每一个都编上序列号，后一个去 watch 前一个，始终是最小的获得锁，一旦最小的释放锁，其成本只是 zk 给第二个发回调，如果中间有挂掉的，触发事件回调后，后一个重新排序找到前一个

#### 【2】代码

ZKUtils

```java
package com.zookeeper.config;

import org.apache.zookeeper.ZooKeeper;

import java.io.IOException;
import java.util.concurrent.CountDownLatch;

public class ZKUtils {
    private static ZooKeeper zk;

    // 使用 /testConf 作为根，其他真正根目录下与 testConf 平级的目录是看不到的
    private static String address =            "192.168.60.132:2181,192.168.60.133:2181,192.168.60.134:2181,192.168.60.135:2181/testLock";

    private static DefaultWatch watch = new DefaultWatch();

    private static CountDownLatch init = new CountDownLatch(1);

    public static ZooKeeper getZk() {
        try {
            zk = new ZooKeeper(address, 1000, watch);
            watch.setCc(init);
            init.await();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return zk;
    }
}
```

WatchCallBack

```java
package com.zookeeper.lock;

import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;

public class WatchCallBack implements Watcher, AsyncCallback.StringCallback,
        AsyncCallback.Children2Callback, AsyncCallback.StatCallback {

    ZooKeeper zk;
    String threadName;
    List tasks;
    String pathName;
    CountDownLatch cc = new CountDownLatch(1);

    // 抢占锁
    public void tryLock() {
        try {
            // 重入锁，这里可以判断当前根数据是否是自己的线程名，如果是就直接进入业务
            // .... 补充 ....
            zk.create("/lock", threadName.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.EPHEMERAL_SEQUENTIAL, this, "abc");
            cc.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    // 释放锁
    public void unLock() {
        try {
            zk.delete(pathName, -1);
            System.out.println(threadName + "over work ...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void process(WatchedEvent watchedEvent) {
        // 如果第一个锁释放了，其实只有第二个收到了回调事件，
        // 如果，不是第一个，是某一个，也能造成它后面的收到通知，从而让它后面的去跟watch挂掉的前一个
        switch (watchedEvent.getType()) {
            case None:
                break;
            case NodeCreated:
                break;
            case NodeDeleted:
                zk.getChildren("/", false, this, "sdf");
                break;
            case NodeDataChanged:
                break;
            case NodeChildrenChanged:
                break;
            case DataWatchRemoved:
                break;
            case ChildWatchRemoved:
                break;
            case PersistentWatchRemoved:
                break;
        }
    }

    @Override
    public void processResult(int i, String s, Object o, String name) {
        if (name != null) {
            pathName = name;
            // 获得所有子节点，看自己是不是最小的,父节点不需要监控
            zk.getChildren("/", false, this, "sdf");
        }
    }

    // getChildren callback,
    // 进到这个方法的前提，一定是所有节点创建完了，且能看到自己前面的所有节点
    @Override
    public void processResult(int i, String s, Object o, List<String> list, Stat stat) {
        System.out.println(threadName + "look locks ....");
//        for (String child : list) {
//            System.out.println(child);
//        }
        Collections.sort(list);
        int i1 = list.indexOf(pathName.substring(1));
        // 是不是第一个节点
        if (i == 0){
            // yes
            System.out.println(threadName + "i am first ...");
            try {
                zk.setData("/", threadName.getBytes(), -1);
            } catch (KeeperException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            cc.countDown();
        } else {
            // no  监控了自己前面的节点，当前面节点发生删除事件时，就回调自己
            zk.exists("/"+list.get(i-1),this,this,"sdf");
        }
    }
    @Override
    public void processResult(int i, String s, Object o, Stat stat) {

    }

    public ZooKeeper getZk() {
        return zk;
    }

    public void setZk(ZooKeeper zk) {
        this.zk = zk;
    }

    public String getThreadName() {
        return threadName;
    }

    public void setThreadName(String threadName) {
        this.threadName = threadName;
    }

    public List getTasks() {
        return tasks;
    }

    public void setTasks(List tasks) {
        this.tasks = tasks;
    }
}
```

TestLock

```java
package com.zookeeper.lock;

import com.zookeeper.config.ZKUtils;
import org.apache.zookeeper.ZooKeeper;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

public class TestLock {

    ZooKeeper zk;

    @Before
    public void conn() {
        zk = ZKUtils.getZk();
    }

    @After
    public void close() {
        try {
            zk.close();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void getConf() {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                WatchCallBack watchCallBack = new WatchCallBack();
                watchCallBack.setZk(zk);
                String threadName = Thread.currentThread().getName();
                watchCallBack.setThreadName(threadName);
                // 每一个线程需要的工作
                //抢锁
                watchCallBack.tryLock();
                //干活
                System.out.println(threadName + "working....");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //释放锁
                watchCallBack.unLock();
            }, "Thread：" + i).start();
        }
    }
}
```
