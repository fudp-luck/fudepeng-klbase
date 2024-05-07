# RocketMQ

## 第一章、RocketMQ 架构设计及概念

### 1、MQ 概念和分布式概念

#### 【1】MQ 概念

- 在 MQ 中有 producer 生产者、broker 存储、consumer 消费者，同时会有发布、订阅、主题的概念
- 当各业务线的发布订阅不同时，使用主题 topic 的形式来区分不同的数据，topic也是最顶层的隔离边界
- <img src="_media/middleware/RocketMq/MQ.png"/>
- MQ 最大的意义在于异步

#### 【2】结合 RocketMQ 分布式概念

- MQ 中也会存在单点故障等问题，所以需要根据 AKF 的概念设计分布式
  - X 轴解决单点故障：RocketMQ 中存在主从的概念，在主从同步时一般分为异步、同步两种同步方式，保证可靠性
  - Y 轴解决数据隔离：在增加 broker 服务器的基础上，可以将不同的项目组的数据按照不同的 topic 存储在不同的 broker 中；topic 也是最顶层的隔离边界，当两个生产者的数据互不影响时，可以分别存储到各自的 broker 中
  - Z 轴解决单点中数据复杂，压力过大：数据的复杂时依赖多个 queue
- 在队列中有 partition 分区和 queue 队列的概念
  - 在 Kafka 中 partition 等同于 queue，一个topic中存在多个partition； topic：（N）partition
  - 在RocketMQ中一个topic中存在多个队列； topic：（N）queue
    - 即使在单机情况下，可以将所有数据放在一个节点中，多个 queue，consumer也可以使用多线程取多个 queue 中的数据
    - 也可以将数据绑定到多个节点，且每个节点的 queue 的数量可以不一样，最终返回所有节点的 queue 的 list
- producer 只能向主 broker 推送数据；consumer 可以在主从 broker 进行消费
- <img src="_media/middleware/RocketMq/RockMq分布式.jpg)

#### 【3】总结 RocketMQ 基础概念

- RocketMQ 是一种 MQ ，具有生产者、消费者
- 是一个分布式的中间件，可以是一台或多台机器
- 是一种主从模型的，其主从间就是比较干净的传递，生产时只与 master 有关
- 在生产和消费时，围绕的是主题的形式，不同的部门可以创建不同的主题，主题是一个逻辑概念，主题下存在多个 queue，在创建 queue 时，可以创建在不同的物理服务器上（Z 轴沙丁分片的形式，起到负载的能力），且在每台物理服务器上 queue 的数量可以一样或不一样，最终这些队列的总和数是这个 topic 主题下所有的队列

### 2、完整框架模型

- 问题：producer 和 consumer 如何知道有多少个 broker，是如何发现和识别 topic 这些信息的
- 在 RocketMQ 中除了 producer、consumer、broker 这些角色以外，还有一类角色是 NameServer，类似于 Eureka
- NameServer 也是单独的服务器，当 broker 启动时，会与每一个 NameServer 进行通信，汇报自己持有的 queue，比如 broker1 汇报自己持有 topicB 中 3 个 queue；broker2 汇报自己持有 topicB 中 2 个 queue，最终 NameServer 会将这些信息统一格式化出来，给到外界完整的路由信息
- 这时一种松散机制，broker 启动后自主汇报给 NameServer，在汇报过程中捏合数据结果，是集群的元数据
- NameServer 是不需要持久化的
- <img src="_media/middleware/RocketMq/RocketMQ完整框架.png"/>

### 3、消费方式

- RocketMQ 提供了量大消费方式，一个是广播形式，一个是集群形式
- 广播形式（bradcast）：producer 生产了一条消息，可以有多个 consumer 都可以收到这一条消息，一对多
- 集群形式：一个 consumer 可以消费多个 queue，一个 queue 只能被一个 consumer 消费
- 组 group：在 consumer 消费是可以有一个组，组内会有若干的 consumer
- <img src="_media/middleware/RocketMq/消费方式.png"/>

### 4、消费特点和消息队列管道和 indexService

#### 【1】消费特点

- Kafka 可以理解为 store（存储）、mq
- RocketMQ 可以理解为 mq、db（既能存储，又有索引以及分析能力）
- RocketMQ 在发送的消息中，除了消息主体 body[] 以外，还包含了其归属于那个 topic，以及 tag 标志子主题和 properties<k,v>，因为包含了这些额外的内容，RocketMQ 就可以建立相应的索引，可以理解为是一个 db，这样整个系统更加灵活，但是增加了 broker 在内存和 CPU 上的开销，增加了负担
- Message 消息，就是业务中端点的信息，也就是生产端和消费端需要去关心这个消息是什么样的
- 在原始的 MQ 中，可以理解为就是一个管道，没有其他的附加功能

#### 【2】利用 MQ 实现 RPC 调用

- RPC 调用：一个主机中的一些线程，需要调用一些其他的服务，传统的 RPC 需要建立 Socket，如果网络断开后，连接就会断开
- 使用 MQ 实现可以通过双端侵入：在任意一个线程发送消息的时候，需要人工的在消息中加入一个 requestID，再加上真正的消息 Message，需要在自己业务代码中维护 requestID，且需要自己维护一个 map 映射，来映射这个 requestID 是哪个线程，或者 CallBack 函数是什么
- 每个线程拿到自己的 requestID 和 Message，并且 map 做好映射后，就可以通过一个 MQ 发送出去
- 在被调用服务中，在得到这个消息后，真正需要的是 Message，在 response 的时候还需要将 requestID 带回去，所以它需要支持 response 中带着 requestID 
- 以上，当 response 带回 requestID 到 MQ 后，主机中的线程通过 requestID 就可以得到回调，整个请求完整
- 而 RocketMQ 原生 API 中是已经封装了 requestID 的，不需要业务人员维护 requestID 和 map 映射，直接发送 Message，所以 RocketMQ 原生就是支持 RPC 调用
- <img src="_media/middleware/RocketMq/利用mq实现rpc.png"/>

#### 【3】indexService

- 当 RocketMQ 按照它的规则将数据发送给 broker 时，broker 会建立索引机制
- 比如在 consumer 消费时指出只要某一个 tag，那么它会只把这个 tag 返回；如果指给出一个 keys，那么它也会将这些 keys 先过滤一遍再返回，分为简单过滤和 sql 表达式过滤

### 5、顺序

- 顺序分为无序和有序
- 无序：消息的本身就是无序的，所有的消息都是无序的，大部分操作之间是没有关系的，在无序的一堆消息过来后，在 topic 中只要最终松散到 queue 队列中，queue 的数量越多，后续 consumer 的线程数和进程数就可以越多，吞吐量就上去了
- 有序：有序又分为全局有序和局部有序
  - 局部有序：因为在一个 topic 名下是有多个 queue 的，如果期望有序的消息进入到一个 queue 中，那么 consumer 在消费 queue 时可以保证这个 queue 是有序的，但无法保证 queue 与 queue 间有序，所以需要 producer 来控制将有序消息发送到一个 queue 中，可以做到局部有序
  - 全局有序：只能有一个 queue，一个 producer 的情况下制造出来的就是全局有序

### 6、消息存储

- 在 broker 服务器启动之后，其名下会有一个 CommitLog 文件，未来外界向这个 broker 推送的所有消息都 append 追加到一个日志文件中，并对应开辟出一些 queue 队列，将 topic 放进去，在设置 commitLogOffset 偏移、msgSize、tagsCode 等设置进去，组建成一个索引系统，当消费时就可以快速找到需要的那部分，不需要重复维护
- 而在 Kafka 中是每个 topic 是一个文件
- <img src="_media/middleware/RocketMq/消息存储.png"/>

## 第二章、RocketMQ 部署及开发

### 1、RockerMQ 的实验及结论

<img src="_media/middleware/RocketMq/rocketmq结论.png"/>

### 2、读写队列数量不一致

<img src="_media/middleware/RocketMq/队列数量不一致.png"/>

### 3、comsumerPull 和 consumerPush

#### 【1】comsumerPull

```java
@Test
    public void consumerPull() throws MQClientException {
        DefaultLitePullConsumer consumer = new DefaultLitePullConsumer("xxx");
        consumer.setNamesrvAddr("192.168.150.11.9876");
        consumer.start();
        Collection<MessageQueue> mqs = consumer.fetchMessageQueues("wula");// 查看消息队列，可以看到 topic 中有哪些 queues
        System.out.println("queues ......");
        mqs.forEach(System.out::println);

        System.out.println("poll ......");
//        consumer.assign(mqs); // 分配，此时可以消费到所有队列
        Collection<MessageQueue> queues = new ArrayList<>();
        MessageQueue qu = new MessageQueue("wula", "node01", 0);
        queues.add(qu);
        consumer.assign(queues); // 分配，此时可以消费node01中0号队列
        consumer.seek(qu, 4); // 偏移量，可以设置从哪里开始消费
        List<MessageExt> poll = consumer.poll();
        poll.forEach(System.out::println);
    }
```

#### 【2】consumerPush

```java
@Test
    public void consumerPush() throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_ox");
        consumer.setNamesrvAddr("192.168.150.11.9876");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET); // 从哪开始push
        consumer.subscribe("wula", "*"); // 订阅
        // 这里的消息是及时消费，是别人推过来的，所以需要监听器，应该是异步的
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            // 数据的集合
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                list.forEach(msg -> {
                    System.out.println(msg);
                    byte[] body = msg.getBody();
                    String msge = new String(body);
                    System.out.println(msge);
                });
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
    }
```

## 第三章、RocketMQ_E.QRocketMQ_Plus_Kafka

### 1、集群搭建案例

#### (一)前置

1. 4 台机器（node01 ~ 04），两主两从
2. 集群中的主从使用的是 sync 同步方式（async 异步）
   1. sync：需要主从同步后再返回
   2. async：只需要主收到信息后直接返回
   3. rocket 中的 message.setWaitStoreMsgOK(true); 默认为 true 同步的，如果是 false 就约等于 kafka 中 ACK = 0
3. 在准备 Linux 时，防火墙关闭，并配置主机名、host，强调主机名不要包含下划线
4. 定义四个配置文件：
   1. 让 broker 自动去 name server 上注册
   2. 相应的 storepath 规划
   3. logs 文件路径的规划
5. 定义目录：mkdir -p /var/rocketmq/{logs,store/{commitlog,consumequeue,index}}

### 【2】配置文件中主从差异

- 两主两从配置文件目录
  - <img src="_media/middleware/RocketMq/两主两从配置文件目录.png"/>
- 配置文件
  - <img src="_media/middleware/RocketMq/主配置文件.png"/>
  - brokeRole = SYNC_MASTER | ASYNC_MASTER
    - 在 message.setWaitStoreMsgOK(true); 时
      - SYNC_MASTER：producer send 到 master 后，master 同步给 slave，然后给 producer 返回 ok
      -  ASYNC_MASTER：producer send 到 master 后，master 异步给 slave 同步，并给 producer 返回 ok
    - 在 message.setWaitStoreMsgOK(false); 时，集群的同步形式同上，只不过 ok 不需要等了，更容易丢数据
  - <img src="_media/middleware/RocketMq/从配置文件.png"/>
  - 注意对 4 个配置文件进行相关配置
- 配置 logs 路径： sed -i 's#${user.home}#/var/rocketmq#g' *.xml
- 修改堆内存大小，改为 1g，否则会无法启动：vi runbroker.sh -> JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g"
- 之后在每一台机器上重复上述配置：scp -r ./rocketm1-4.9.2/ node02:/opt （发送到其他机器上）
- 可以配置环境变量：
  - pwd 获取安装路径
  - vi etc/profile -> export ROCKETMQ_HOME=[安装路径] -> 最后一行加入 :$ROCKETMQ_HOME/bin
  - 加载 ./etc/profile

### 2、启动 SimpleRocketMq

### 【1】启动

1. 启动时先启动 nameserver（node01 ~ 03）；命令：mqnamesrv
2. 之后是 broker ；命令：mqbroker -c /opt/rocketmq-4.9.2/conf/22conf/broker-a.properties
   1. node01 ~ 02 broker-a 0,1 master slave
   2. node03 ~ 04 broker-a 0,1 master slave

### 【2】API-admin

```java
@Test
    public void admin() throws Exception {
        DefaultMQAdminExt admin = new DefaultMQAdminExt();
        admin.setNamesrvAddr("1192.168.150.11:9876;192.168.150.12:9876;192.168.150.13:9876");
        admin.start();
        ClusterInfo clusterInfo = admin.examineBrokerClusterInfo();
        HashMap<String, BrokerData> brokerAddrTable = clusterInfo.getBrokerAddrTable();
        Set<Map.Entry<String, BrokerData>> entries = brokerAddrTable.entrySet();
        Iterator<Map.Entry<String, BrokerData>> iterator = entries.iterator();
        while (iterator.hasNext()) {
            Map.Entry<String, BrokerData> next = iterator.next();
            System.out.println(next.getKey() + "  " + next.getValue());
        }
    }
```

<img src="_media/middleware/RocketMq/集群信息.png"/>

### 3、ExampleRocketMQ 和 DefaultMQPushCosumer 案例

- NameServer 地址

  - ```java
    private static final String ADDRESS = "1192.168.150.11:9876;192.168.150.12:9876;192.168.150.13:9876";
    ```

- 管理工具API

  - ```java
    @Test
        public void admin() throws Exception {
            DefaultMQAdminExt admin = new DefaultMQAdminExt();
            admin.setNamesrvAddr(ADDRESS);
            admin.start();
            ClusterInfo clusterInfo = admin.examineBrokerClusterInfo();
            HashMap<String, BrokerData> brokerAddrTable = clusterInfo.getBrokerAddrTable();
            Set<Map.Entry<String, BrokerData>> entries = brokerAddrTable.entrySet();
            Iterator<Map.Entry<String, BrokerData>> iterator = entries.iterator();
            while (iterator.hasNext()) {
                Map.Entry<String, BrokerData> next = iterator.next();
                System.out.println(next.getKey() + "  " + next.getValue());
            }
            TopicList topicList = admin.fetchAllTopicList(); // 拿到所有 topic 的集合
            topicList.getTopicList().forEach(System.out::println);
        }
    ```

- producer

  - ```java
     @Test
        public void producer() throws Exception {
            DefaultMQProducer producer = new DefaultMQProducer("ooxx");
            producer.setNamesrvAddr(ADDRESS);
            producer.start();
            return producer;
            // 批量发送
            ArrayList<Message> messages = new ArrayList<>();
            for (int i = 0; i < 10; i++) { // 10条消息
                messages.add(new Message("bala", "TagA", "key" + i, ("message" + i).getBytes()));//key要全局唯一
            }
            SendResult send = producer.send(messages);
            System.out.println(send);
        }
    ```

- consumer

  - ```java
    /**
         * 配置项 messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
         * 18个延迟级别，应用场景：
         * ConsumeConcurrentlyStatus.RECONSUME_LATER; 重试每次时间间隔
         * consumer.setMaxReconsumeTimes(2); 最大重试几次
         * %RETRY%xxx、%DLQ%xxx 是rocketmq维护的重试队列和死信队列，是以topic形式，且以consumerGroup为单位的
         */
        @Test
        public void consumer() throws MQClientException {
            DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("xxx");
            consumer.setNamesrvAddr(ADDRESS);
            consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
            consumer.setMaxReconsumeTimes(2);
            consumer.setConsumeMessageBatchMaxSize(1);
            consumer.subscribe("bala", "*");
            consumer.registerMessageListener(new MessageListenerConcurrently() {
                @Override
                public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                    MessageExt msg = list.get(0);
                    try {
                        System.out.println(new String(msg.getBody()));
                        if (msg.getKeys().equals("key1")) {
                            int a = 1 / 0;
                        }
                    } catch (Exception e) {
                        return ConsumeConcurrentlyStatus.RECONSUME_LATER; // 表示重试
                    }
                    return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                }
            });
            /*
            new MessageListenerOrderly()  有序消息处理
            new MessageListenerConcurrently() 无序消息处理，线程安全的，消息的消费成功与否不影响后序的消息，可以常识重复的去消费这条消息
            最大区别在于消息顺序的依赖性
            */
            consumer.start();
        }
    ```

- orderProducer

  - ```java
    @Test
        public void orderProducer() throws Exception {
            DefaultMQProducer producer = defualtProducer("ooxx");
            // 解决下面的性能问题，此处是方法级的，在堆中只有一个对象，复用即可
            MyMessageQueueSelector selector = new MyMessageQueueSelector();
            for (int i = 0; i < 10; i++) {
                // 将取模为0的消息顺序放到一个队列中
                Message msg = new Message(
                        "order_topic",
                        "TagA",
                        ("message body : " + i + "tye : " + i % 3).getBytes()
                );
                SendResult res = producer.send(msg, selector, i % 3);
    //            producer.send(msg, new MessageQueueSelector() { // 此处存在性能问题
    //                @Override
    //                public MessageQueue select(List<MessageQueue> list, Message message, Object arg) {
    //                    Integer index = (Integer) arg;
    //                    int taget = index % list.size();
    //                    return list.get(taget);
    ////                    MessageQueue target = list.get(index);
    ////                    return target
    //                }
    //            }, i % 3);
            }
        }
    ```
    
    ```java
    public class MyMessageQueueSelector implements MessageQueueSelector {
        MyMessageQueueSelector(){
            System.out.println("create selector....");
        }
        @Override
        public MessageQueue select(List<MessageQueue> mqs, Message message, Object arg) {
            Integer index = (Integer) arg;
            int target = index % mqs.size();
            return mqs.get(target);
        }
    }
    ```

- orderConsumer

  - ```java
    /**
         * 保证有序性：
         * 1、producer，其实就是自定义分区器（队列选择器）
         * 2、consumer，要使用MessageListenerOrderly，
         * 才能实现生产的时候，有顺序依赖的msg进入一个队列，并且，消费者也会再多线程情况下保证单线程对应的queue能按顺序消费
         * 如果，只创建一个queue，就全局有序
         */
    
        // 有序的消费
        @Test
        public void orderConsumer() throws MQClientException {
            DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("xoxo");
            consumer.setNamesrvAddr(ADDRESS);
            consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
            try {
                consumer.subscribe("order_topic", "*");
            } catch (MQClientException e) {
                e.printStackTrace();
            }
            consumer.registerMessageListener(new MessageListenerOrderly() {
                @Override
                public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext consumeOrderlyContext) {
                    list.forEach(msg -> {
                        System.out.println(Thread.currentThread().getName() + " : " + new String(msg.getBody()));
                    });
                    return ConsumeOrderlyStatus.SUCCESS;
                }
            });
            consumer.start();
            consumer.shutdown();
        }
    ```

## 第四章、事物 / 死信 / 延时 / 组 / 偏移原理

### 1、广播

```java
// 在 consumer 端自己规划
consumer.setMessagemodel(MessageModel.BROADCASTING);
```

### 2、延时队列

- 场景：有时候向队列里发送消息，很快就会被消费端消费到，我们希望的是隔一段时间之后再让消费端看到这个消息
- 延迟队列的实现：RocketMq 采用粗粒度
  - 粗粒度：比如在 RocketMq 的重试机制 messageDelayLevel=1s 5s 10s 30s 1m 2m .... ，按照时间规则准备多个 topic，比如 1s_topic、30s_topic，生产的时候，只要消息带着延迟级别，就会将这个消息放到相应的中间队列里，如果期望在 30s 后收到消息，那么首先这个消息会被放到 30s_topic 中，由 RocketMq broker 在进程内部盯着这个队列，到了时间后，再推到目标队列 topic 中
  - 细粒度：采用时间轮，就是类似钟表一样秒针转动一圈，分针走一下，但其中存在排序成本，需要根据传入的各个延迟时间进行排序，不适合处理并发量大的情况，在一些特殊情况下，时间轮是最优解

```java
    private static final String ADDRESS = "1192.168.150.11:9876;192.168.150.12:9876;192.168.150.13:9876";

    /**
     * 主要体现在 producer
     */
    @Test
    public void consumerDelay() throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("delayconsumer");
        consumer.setNamesrvAddr(ADDRESS);
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET); // 从哪开始push
        consumer.subscribe("topic_delay", "*");
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                msgs.forEach(msg -> {
                    System.out.println(msg);
                });
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
    }

    @Test
    public void producerDelay() throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("delay_producer");
        producer.setNamesrvAddr(ADDRESS);
        producer.start();
        for (int i = 0; i < 10; i++) {
            Message msga = new Message(
                    "topic_delay",
                    "TagD",
                    ("message" + i).getBytes()
            );
            // 延迟是以消息为级别的
            msga.setDelayTimeLevel(i % 18);
            SendResult send = producer.send(msga); // 没有直接进入目标topic的队列，二十进入延迟队列
            System.out.println(send);
        }
    }
```

### 3、事物和分布式事物

#### 【1】什么是事务

- 事务是一批操作，要么成功要么失败
- 在分布式事务中，就分为 MQ 的写入是一件事情，包括数据更新，调用一些接口，这些事情要么成功要么全部回滚
- 所以 RocketMq 事务不止解决自己生产消息的事务能力

#### 【2】事务过程

- 问题：
  - 当 service 先调用接口和修改数据库数据时，mq可能推送失败，那么下游就收不到消息
  - 当 service 先推送消息再地通用接口和修改数据库时，接口可能挂掉导致数据库的数据没有机试修改
- 注意：推送的消息能否被consumer看到，取决于接口和数据库的返回结果，如果调用成功则一定要被看到，反之则不被看到
- RocketMq 提出了两阶段提交和状态的判断和重试
- 两阶段提交：表示在调用接口准备修改数据库数据时，就将数据提交到Mq，但不会被consumer看到，当接口和数据修改成功后再做第二次提交，反之则回滚
- <img src="_media/middleware/RocketMq/两阶段提交.png"/>
- 此时出现的问题：出现单点故障，在推送消息的过程中服务挂掉了
- RocketMq 中提供了回调检查机制，需要提供一个方法去获取数据库中记载的service本地事务和事务ID，之后轮询询问service是否活过来了，注意挂掉的service的producer group 名要和重启后的组名一致，最终决定消息是否提交，满足了最终一致性
- <img src="_media/middleware/RocketMq/回调检查.png"/>

#### 【3】事务结论

- consumer 能否看到 RocketMq 事务消息，完全取决于 producer 所在的服务的本地事务成功与否
- 那么在 service 的事务开始到结束状态到发送消息中，会有各种情况发生：任意一步可能失败，服务宕机，网络断开
- 所以 RocketMq 在设计时采用了：两阶段提交（预写（半消息，half）+ 本地事务）+回查机制（规避了service挂机）+ 最终事务是否 commit/rolback
- <img src="_media/middleware/RocketMq/事务总结.png"/>

#### 【4】consumer & producer

```java
private static final String ADDRESS = "1192.168.150.11:9876;192.168.150.12:9876;192.168.150.13:9876";

    /**
     * 主要体现在 producer
     * 尽量不要自己去做一些属性，这样会自己加入状态，一旦机器重启后，那么自定义的这些状态就都没有了
     * 所以在做下面代码时，尽量用mq 中的以有的api，包括内部类
     */
    @Test
    public void consumerTransaction() throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("transaction_consumer");
        consumer.setNamesrvAddr(ADDRESS);
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET); // 从哪开始push
        consumer.subscribe("topic_transaction", "*");
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                msgs.forEach(msg -> {
                    System.out.println(msg);
                });
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
    }
```

```java
@Test
    public void producerTransaction() throws Exception {
        TransactionMQProducer producer = new TransactionMQProducer("transaction_producer");
        producer.setNamesrvAddr(ADDRESS);
        // 1、半消息成功了才能执行本地事务，也需要坚挺；
        // 2、半消息的回调需要被监听到
        producer.setTransactionListener(new TransactionListener() {
            /*
            有三个地方可以传导业务需要的参数
                1、message 的 body ，存在网络带宽的成本，增加了对body编码解码的过程
                2、userProperty，也是通过网络传递给consumer
                3、arge 方式，arge可以是一个很复杂的内容，甚至可以将一个方法对象封装进去，consumer得到后只执行本地方法
             */
            @Override
            public LocalTransactionState executeLocalTransaction(Message msg, Object arg) { // 此处的Message msg 是RocketMq推送成功之后返回的message，里面包括了transactionid
                // send ok half
//                String action = new String(msg.getBody());
//                String action = msg.getProperty("action");
                String action = (String) arg;
                String transactionId = msg.getTransactionId();
                System.out.println("transactionId: " + transactionId);
                /*
                RocketMq 中事务状态有两个
                    1、RocketMq 的 half 半消息，这个状态驱动 RocketMq 回查 producer
                    2、service 应该是无状态的，那么应该把 transactionId 随着本地事务的执行写入事件状态表
                        比如将mysql 中扣减库存需要修改的表中数据以及transactionId加到另一个事件表中 或者
                        调用了对方接口之后，可以将transactionId放到一个表里，回调的时候碰撞transactionId，看上面要查询的是哪个接口
                 */
                switch (action) {
                    case "0":
                        // 此处可以理解为：调用了一个接口，但不知道什么时候返回，所以需要被检查，最终返回 commit，或者是抛出了异常
                        System.out.println(Thread.currentThread().getName() + "send half：Async api call...action：0，异步调用");
                        return LocalTransactionState.UNKNOW; // RocketMq 会回调，去 check
                    case "1":
                        System.out.println(Thread.currentThread().getName() + "send half：localTransaction faild...action：1，表示失败了");
                        /*
                        transaction.begin
                        throw...
                        transaction.rollback
                         */
                        // consumer 是消费不到 rollback 的 message 的
                        return LocalTransactionState.ROLLBACK_MESSAGE;
                    case "2":
                        System.out.println(Thread.currentThread().getName() + "send half：localTransaction ok...action：2，表示成功");
                        /*
                        transaction.begin
                        throw...
                        transaction.commit ..ok
                         */
                        // consumer 是肯定消费的到的，只不过还要验证，中途会不会做check
                        try {
                            Thread.sleep(1000 * 10);
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                        return LocalTransactionState.COMMIT_MESSAGE;
                }
                return null;
            }

            @Override
            public LocalTransactionState checkLocalTransaction(MessageExt msg) {
                // call back check
                String transactionId = msg.getTransactionId();
                String action = msg.getProperty("action");
                int times = msg.getReconsumeTimes();

                switch (action) {
                    case "0":
                        System.out.println(Thread.currentThread().getName() + "check half：Async api call...action：0，UNKNOW" + times);
                        // 模拟多次查询
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                        return LocalTransactionState.COMMIT_MESSAGE;
                    case "1":
                        System.out.println(Thread.currentThread().getName() + "check ：1 ROLLBACK" + times);
                        // 观察事务表，如果没有发现状态变化就直接返回 unknow
                        return LocalTransactionState.UNKNOW;
                    case "2":
                        System.out.println(Thread.currentThread().getName() + "check ：2 COMMIT" + times);
                        // check 都是观察事务表的，在上面10s内没有成功，此处就是未知状态
                        return LocalTransactionState.UNKNOW;
                        // 只有在未来盯着事务表得到结果，就可以 commit 或 rollback
                    // 做了一个长时间的事务过程中，生产者挂了，首先要 check 关注事务表，如果producer重启了一下还能否响应这个检查
                }
                return null;
            }
        });

        // 异步回调线程，不是必须要配置的，在源码中会准备一个
        producer.setExecutorService(new ThreadPoolExecutor(
                1,
                Runtime.getRuntime().availableProcessors(),
                2000,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(2000),
                new ThreadFactory() {
                    @Override
                    public Thread newThread(Runnable r) { // loop worker -> 消费你提供的 queue中的 runnable
                        return new Thread(r, "Transaction thread");
                    }
                }
        ));
        producer.start();

        for (int i = 0; i < 10; i++) {
            Message msgt = new Message(
                    "topic_transaction",
                    "TagT",
                    "key:" + i,
                    ("message" + i).getBytes()
            );
            msgt.putUserProperty("action", i % 3 + "");
            // 此处发送的是半消息
            SendResult send = producer.sendMessageInTransaction(msgt, i % 3 + "");
            System.out.println(send);
        }
    }
```

## 第五章、Producer 源码分析

<img src="_media/middleware/RocketMq/RocketMq源码1.png"/>

<img src="_media/middleware/RocketMq/RocketMq源码2.png"/>

### 1、Producer 主要流程

<img src="_media/middleware/RocketMq/producer主要流程.png"/>

### 2、Producer 的启动

#### 【1】DefaultMQProducer

<img src="_media/middleware/RocketMq/DefaultMQProducer.png"/>

#### 【2】producer.start() -- MQClientInstance 和 APIImpl

- 源码流程参考

<img src="_media/middleware/RocketMq/MQClientInstance 和 APIImpl.png"/>

- buildMQClientId（为客户端定义ID）：由主机IP地址+进程ID号组成
  - 在一个主机里 IP 地址是一样的，在一个JVM里，进程ID号是一样的
  - 如果在一个 JVM 里有 producer 和 consumer ，那么这一个实例就可以进行通信了
  - 在一台主机上有多个 JVM，那么每个 JVM 有自己的单例，他们的 IP 是一样的，但进程 ID 号是不一样的，所以在外界看来，就是独立分割的两个实例

#### 【3】mQClientFactory.start() -- NettyRemtingClient 和 NioEventLoop

### 3、设计 -- 资源隔离

1. RocketMq 的资源隔离：通过code码来区分消息的类型，分别抛给不同的线程池
2. Kafka 的资源隔离：先将所有消息打到一个通道里，之后分配线程，然后再识别调用的是哪个 API 之后对应的处理
3. 区别就是 RocketMq 对资源隔离开了，且在代码逻辑上比较清晰；Kafka 没有资源隔离，但 Kafka 开的线程池少，不浪费资源

<img src="_media/middleware/RocketMq/资源隔离.png"/>



