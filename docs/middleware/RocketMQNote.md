# RocketMQ基础实战

## 一、RocketMQ 入门分析

### 1、消息中间件及其使用场景

#### 【1】消息中间件

消息中间件（MessageQueue），其主要功能是异步处理，解决同步处理下流程阻塞问题。

同步处理需要等待下游程序的应答才能继续执行其他逻辑，如果没有应答，那么程序就会阻塞。

<img src="_media/middleware/RocketMq/同步调用模型.png" style="zoom:50%;" />

异步处理就可以将消息发送到消息中间件中，无需等待其他程序系统的应答，而该消息可以被其他系统所消费执行。

<img src="_media/middleware/RocketMq/异步处理模型.png" style="zoom:50%;" />

#### 【2】使用场景

1. 异步解耦，在各个子系统之间存在耦合性，耦合性越高则容错性越低，使用MQ后即使某个子系统宕机，也不会影响其他系统的运行。

   <img src="_media/middleware/RocketMq/异步解耦.png" alt="image-20231223183455816" style="zoom:50%;" />

2. 流量削峰，比如互联网的大促场景，如果没有消息中间件，那么系统在面对大量并发的情况下受制于数据库的读写能力，就会有大量请求堆积，引入MQ之后，可以在前端将消息写入MQ中，使系统逐一处理请求，是一种以时间换空间的做法从而确保系统稳定性。

   <img src="_media/middleware/RocketMq/流量削峰.png" style="zoom:50%;" />

3. 数据分发，在场景中一个系统需要对应多个系统，如果使用传统的RPC调用，当其他系统出现功能的增删需要数据的传递时，那么就需要修改代码来适配功能的增删。应用MQ后，系统只需要关注于自身业务，将生产的数据发送到MQ，其他系统功能的增删可以到MQ中消费所需的数据，便于数据的分发，减少代码修改，提高团队配合的效率。

   <img src="_media/middleware/RocketMq/数据分发.png" style="zoom:50%;" />

### 2、RocketMQ产品分析

#### 【1】各角色介绍

NameSever：相当于服务注册与发现中心，启动RocketMQ必须要先启动NameSever；

Broker：主机，里面包括主题、队列等相关信息，启动后注册到NameSever，主要负责消息的存储；

producer和consumer：需要到NameSever中拿到主机地址之后再进行消息的发送和消费。

<img src="_media/middleware/RocketMq/RocketMQ角色.png"/>

#### 【2】基本概念

主题（Topic）：标识一类消息，比如电商场景中一个产品类目就可以标识为一个主题；

分组（Group）：生产者标识同一类消息的，通常发送逻辑一致；消费者表示对一类消息的消费并且消费逻辑一致，比如针对订单消息，消费者方可以制定物流组和通知组分别对订单消息进行处理，这两类之间是相互隔离的；

消息队列（Queue）：一个主题有若干个Queue，发送时无需区分消息在哪个队列中，一般会分散到不同的Queue中，在消费时，一个消费者可以启动多个节点订阅不同的Queue，提高消费的并发性；

标签（Tag）：在处理消息时，消息往往会归于一类，在同类下会存在不同的标签，比如电商场景中收集类别下会有“国产手机”、“苹果手机”等不同的标签，为消息打上对应标签有助于消费时进行筛选，比如订阅“Topic手机”时加入“安卓手机”标签条件，就可以完成消费的过滤；

偏移量（Offset）：一般不指定偏移量时所代表的是Queue中的consumerOffset消费者偏移量，另外对于主机中的每个队列也有它的最大偏移量，可以代表最大有多少条数据。

<img src="_media/middleware/RocketMq/RocketMQ概念.png"/>

### 3、RocketMQ安装

#### 【1】Windows下安装

官网：[下载 | RocketMQ (apache.org)](https://rocketmq.apache.org/download/)

<img src="_media/middleware/RocketMq/Windows下设置环境变量1.png"/>

启动broker时：修改runbroker.cmd -> set "JAVA_OPT=%JAVA_OPT% -cp "%CLASSPATH%""

#### 【2】Linux下安装

```shell
cd rocketmq-all-4.8.0-bin-release/
cd bin/
nohup sh mqnamesrv & #启动NameSever
tail -f ~/logs/rocketmqlogs/namesrv.log

cd ../conf/
vi broker.conf
	brokerIP1=192.168.1.123 #配置为外网可以访问的地址
cd ../bin/
nohup sh mqbroker -c ../conf/broker.conf -n 192.168.1.123:9876 autoCreateTopicEnable=true & #指定ns地址
tail -f ~/logs/rocketmqlogs/broker.log
# 修改broker堆空间默认值，改小
vi runbroker.sh
	JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m"
nohup sh mqbroker -c ../conf/broker.conf -n 192.168.1.123:9876 autoCreateTopicEnable=true &
```

#### 【3】源码的安装

官网：source目录

执行：mvn install -Dmaven.test.skip=true

启动NameSever：namesrv -> NamesrvStartup.java

运行前配置环境变量：该地址文件下包含distribution下conf，此外需要新建logs文件以及store文件用于存储消息

<img src="_media/middleware/RocketMq/idea环境变量.png"/>

启动NameServer

在Broker工程目录下找到BrokerStartup，启动 -c指定目录

![image-20240104181004728](_media/middleware/RocketMq/启动broker参数.png"/>

#### 【4】控制台安装

地址1：[codeload.github.com/apache/rocketmq-externals/zip/master]()

地址2：[github.com/apache/rocketmq-dashboard]()

```properties
## rocketmq-externals-master > rocketmq-console > src > main > resources > application.properties
## 当配置在不同服务器时，需要修改地址
rocketmq.config.namesrvAddr=localhost:9876; 
```

将rocketmq-console项目打包后在target下命令行启动

```cmd
java -jar rocketmq-console-ng-2.0.0.jar
```

访问：127.0.0.1:8089

## 二、RocketMQ 实战操作

### 1、普通消息发送

#### 【1】消息发送流程

1. 导入MQ客户端依赖

   ```xml
   <dependency>
   	<groupId>org.apache.rocketmq</groupId>
   	<artifactId>rocketmq-client</artifactId>
   	<version>4.8.0</version>
   </dependency>
   ```

2. 消息发送者步骤

   1. 创建消息生产者producer，并指定生产者组名
   2. 指定Nameserver地址
   3. 启动producer
   4. 创建消息对象，指定Topic、Tag和消息体
   5. 发送消息
   6. 关闭生产者producer

3. 消息消费者步骤

   1. 创建消费者Consumer，指定消费者组名
   2. 指定Nameserver地址
   3. 订阅主题Topic和Tag
   4. 设置回调函数，处理消息
   5. 启动消费者consumer

#### 【2】普通消息同步发送

同步消息发送可以确保消息发送成功，可以用于一些比较重要的消息，比如消息通知和短信通知

<img src="_media/middleware/RocketMq/同步发送时序图.png" style="zoom:50%;" />

```java
/**
 * 同步发送  原生的API :SpringBoot   封装-> 原生
 */
public class SyncProducer {
    public static void main(String[] args) throws Exception{
        // 实例化消息生产者Producer
        DefaultMQProducer producer = new DefaultMQProducer("group_test");

        // 设置NameServer的地址（Broker有多台，  2主（对生产消费）2从（数据备份）的架构）  避免：单点故障
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();
       // producer.setDefaultTopicQueueNums(8);

        for (int i = 0; i < 10; i++) { //发送100条消息
        // 创建消息，并指定Topic（消息进行分类： 衣服、电器、手机），Tag（男装、女装、童装   -》消费环节：过滤）和消息体
            Message msg = new Message("TopicTest" /* Topic */,
                    "TagA" /* Tag */,
                    ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            // 发送消息到一个Broker
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        //如果不再发送消息，关闭Producer实例。
        producer.shutdown();
    }
}
```

#### 【3】普通消息异步发送

作为发送者不能容忍等待响应的时间，通过回调接口方式接受响应，适应发送量比较大的场景

<img src="_media/middleware/RocketMq/异步发送时序图.png" style="zoom:50%;" />

```java
/**
 * 异步发送--生产者
 */
public class AsyncProducer {
    public static void main(String[] args) throws Exception{
        // 实例化消息生产者Producer
        DefaultMQProducer producer = new DefaultMQProducer("group_test");
        // 设置NameServer的地址
        producer.setNamesrvAddr("127.0.0.1:9876");
        // 启动Producer实例
        producer.start();
        for (int i = 0; i < 10; i++) {
            final int index = i;
            // 创建消息，并指定Topic，Tag和消息体
            Message msg = new Message("topicA", "TagA", "OrderID888",
                    "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
            // SendCallback接收异步返回结果的回调
            producer.send(msg, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.printf("%s%n", sendResult);
                }
                @Override
                public void onException(Throwable e) {
                    System.out.printf("%-10d Exception %s %n", index, e);
                    e.printStackTrace();
                }
            });
        }
        Thread.sleep(10000);
        // 如果不再发送消息，关闭Producer实例。
        producer.shutdown();
    }
}
```

控制台参数：

1. Message ID：消息的全局唯一标识（内部机制的ID生成是使用机器IP和消息偏移量的组成，所以有可能重复，如果是幂等性还是最好考虑Key），由消息队列 MQ 系统自动生成，唯一标识某条消息。
2. SendStatus：发送的标识。成功，失败等
3. Queue：相当于是Topic的分区；用于并行发送和接收消息

#### 【4】普通消息的单项发送

发送方只关注发送，不管是否发送成功，比如日志收集

<img src="_media/middleware/RocketMq/单项发送时序图.png" style="zoom:50%;" />

```java
/**
 * 单向发送
 */
public class OnewayProducer {
    public static void main(String[] args) throws Exception{
        // 实例化消息生产者Producer对象
        DefaultMQProducer producer = new DefaultMQProducer("group_test");
        // 设置NameServer的地址
        producer.setNamesrvAddr("127.0.0.1:9876");
        // 启动Producer实例
        producer.start();
        for (int i = 0; i < 10; i++) {
            // 创建消息，并指定Topic，Tag和消息体
            Message msg = new Message("TopicTest" /* Topic */,
                    "TagA" /* Tag */,
                    ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            // 发送单向消息，没有任何返回结果
            producer.sendOneway(msg);

        }
        // 如果不再发送消息，关闭Producer实例。
        producer.shutdown();
    }
}
```

```java
/**
 * 单向发送
 */
public class OnewayProducer2 {
    public static void main(String[] args) throws Exception{
        // 实例化消息生产者Producer   对象。
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
        // 设置NameServer的地址
        producer.setNamesrvAddr("106.55.246.66:9876");//106.55.246.66
        // 启动Producer实例
        producer.start();
        for (int i = 0; i < 100; i++) {
            // 创建消息，并指定Topic，Tag和消息体
            Message msg = new Message("TopicTest" /* Topic */,
                    "TagA" /* Tag */,
                    ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            // 发送单向消息，没有任何返回结果
            producer.sendOneway(msg);

        }
        // 如果不再发送消息，关闭Producer实例。
        producer.shutdown();
    }
}

```

#### 【5】消息发送方式选择

|   发送方式   | 发送TPS | 发送结果反馈 |  可靠性  |                           适用场景                           |
| :----------: | :-----: | :----------: | :------: | :----------------------------------------------------------: |
| 同步可靠发送 |   快    |      有      |  不丢失  |            重要通知邮件、报名短信通知、营销短信等            |
| 异步可靠发送 |   快    |      有      |  不丢失  | 用户视频上传后通知启动转码服务，转码完成后通知推送转码结果等 |
|   单项发送   |  最快   |      无      | 可能丢失 | 适用于某些耗时非常短，但对可靠性要求并不高的场景，例如日志收集 |



### 2、消费模式

#### 【1】集群消费模式

一个消费者集群中的各个Consumer实例分摊去消费消息，即一条消息只会投递到一个Consumer Group下面的一个实例。

每个实例都平均分摊拉取消费Message Queue中的消息。例如某个Topic有3条Q，其中一个Consumer Group 有 3 个实例（可能是 3 个进程，或者 3 台机器），那么每个实例只消费其中的1条Q。

由于Producer发送消息的时候是轮询发送所有的Q，所以消息会平均散落在不同的Q上，可以认为Q上的消息是平均的。那么实例也就平均地消费消息了。

这种模式下，消费进度(Consumer Offset)的存储会持久化到Broker。

<img src="_media/middleware/RocketMq/集群消费模型.png" style="zoom:50%;" />

```java
public class BalanceComuser {
    public static void main(String[] args) throws Exception {
        // 实例化消费者,指定组名:  TopicTest  10条消息 group_consumer  ，  lijin 8(2)
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group_consumer");
        // 指定Namesrv地址信息.
        consumer.setUnitName("consumer1");  //一个订阅服务器A

        consumer.setUnitName("consumer2"); //一个订阅服务器B
        consumer.setNamesrvAddr("127.0.0.1:9876");
        // 订阅Topic
        consumer.subscribe("TopicTest", "*"); //tag  tagA|TagB|TagC
        //consumer.setConsumeFromWhere();

        //集群模式消费
        consumer.setMessageModel(MessageModel.CLUSTERING);

        //取消
        //consumer.unsubscribe("TopicTest");
        //再次订阅Topic即可
        consumer.subscribe("TopicTest", "*"); //tag  tagA|TagB|TagC

        // 注册回调函数，处理消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                            ConsumeConcurrentlyContext context) {
                try {
                    for(MessageExt msg : msgs) {
                        String topic = msg.getTopic();
                        String msgBody = new String(msg.getBody(), "utf-8");
                        String tags = msg.getTags();
                        Thread.sleep(1000);
                        System.out.println("收到消息：" + " topic :" + topic + 
                                " ,tags : " + tags + " ,msg : " + msgBody);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;

                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        //启动消息者
        consumer.start();
        //注销Consumer
        //consumer.shutdown();
        System.out.printf("Consumer Started.%n");
    }
}
```

#### 【2】广播消费模式

消息将对一个消费集群下的各个Consumer实例都投递一遍。即使这些 Consumer 属于同一个Consumer Group，消息也会被Consumer Group 中的每个Consumer都消费一次。

实际上，是一个消费组下的每个消费者实例都获取到了topic下面的每个Message Queue去拉取消费。所以消息会投递到每个消费者实例。

这种模式下，消费进度(Consumer Offset)会存储持久化到实例本地。

<img src="_media/middleware/RocketMq/广播消费模式.png" style="zoom:50%;" />

```java
public class BroadcastComuser {
    public static void main(String[] args) throws Exception {
        // 实例化消息生产者,指定组名
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("C-king");
        // 指定Namesrv地址信息.
        consumer.setNamesrvAddr("127.0.0.1:9876");//106.55.246.66
        // 订阅Topic
        consumer.subscribe("TopicTest", "*");
        //广播模式消费
        consumer.setMessageModel(MessageModel.BROADCASTING);
        // 如果非第一次启动，那么按照上次消费的位置继续消费
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
        // 注册回调函数，处理消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                             ConsumeConcurrentlyContext context) {
                try {
                    for(MessageExt msg : msgs) {
                        String topic = msg.getTopic();
                        String msgBody = new String(msg.getBody(), "utf-8");
                        String tags = msg.getTags();
                        System.out.println("收到消息：" + " topic :" + topic + 
                                " ,tags : " + tags + " ,msg : " + msgBody);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;

                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //启动消息者
        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}
```

#### 【3】消费模式的选择

集群模式：适用场景&注意事项 

1. 消费端集群化部署，每条消息只需要被处理一次。
2. 由于消费进度在服务端维护，可靠性更高。
3. 集群消费模式下，每一条消息都只会被分发到一台机器上处理。如果需要被集群下的每一台机器都处理，请使用广播模式。
4. 集群消费模式下，不保证每一次失败重投的消息路由到同一台机器上，因此处理消息时不应该做任何确定性假设。

广播模式：适用场景&注意事项 

1. 广播消费模式下不支持顺序消息。
2. 广播消费模式下不支持重置消费位点。
3. 每条消息都需要被相同逻辑的多台机器处理。
4. 消费进度在客户端维护，出现重复的概率稍大于集群模式。
5. 广播模式下，消息队列 RocketMQ 保证每条消息至少被每台客户端消费一次，但是并不会对消费失败的消息进行失败重投，因此业务方需要关注消费失败的情况。
6. 广播模式下，**客户端每一次重启都会从最新消息消费。客户端在被停止期间发送至服务端的消息将会被自动跳过，请谨慎选择。**
7. 广播模式下，每条消息都会被大量的客户端重复处理，因此推荐尽可能使用集群模式。
8. 目前仅 Java 客户端支持广播模式。
9. 广播模式下服务端不维护消费进度，所以消息队列 RocketMQ 控制台不支持消息堆积查询、消息堆积报警和订阅关系查询功能。

### 3、生产与消费

#### 【1】顺序消息的生产与消费

在默认的情况下消息发送会采取Round Robin轮询方式把消息发送到不同的queue(分区队列)；而消费消息的时候从多个queue上拉取消息，这种情况发送和消费是不能保证顺序。

但是如果控制发送的顺序消息只依次发送到同一个queue中，消费的时候只从这个queue上依次拉取，则就保证了顺序。

当发送和消费参与的queue只有一个，则是全局有序；

<img src="_media/middleware/RocketMq/全局顺序消息.png"/>

如果多个queue参与，则为分区有序，将部分消息给到相应的标识，即相对每个queue，消息都是有序的。

<img src="_media/middleware/RocketMq/部分顺序消息.png"/>

```java
/**
 * 部分顺序消息生产
 */
public class ProducerInOrder {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("OrderProducer");
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();
        // 订单列表
        List<Order> orderList = new ProducerInOrder().buildOrders();
        for (int i = 0; i < orderList.size(); i++) {
            String body = orderList.get(i).toString();
            Message msg = new Message("PartOrder", null, "KEY" + i, body.getBytes());
            // MessageQueueSelector 消息队列选择器，根据消息特征选择
            SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                @Override
                public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                    Long id = (Long) arg;  //根据订单id选择发送queue
                    long index = id % mqs.size();
                    return mqs.get((int) index);
                }
            }, orderList.get(i).getOrderId());//订单id
            System.out.println(String.format("SendResult status:%s, queueId:%d, body:%s",
                    sendResult.getSendStatus(),
                    sendResult.getMessageQueue().getQueueId(),
                    body));
        }
        producer.shutdown();
    }

    /**
     * 订单
     */
    private static class Order {
        private long orderId;
        private String desc;

        public long getOrderId() {
            return orderId;
        }

        public void setOrderId(long orderId) {
            this.orderId = orderId;
        }

        public String getDesc() {
            return desc;
        }

        public void setDesc(String desc) {
            this.desc = desc;
        }

        @Override
        public String toString() {
            return "Order{" +
                    "orderId=" + orderId +
                    ", desc='" + desc + '\'' +
                    '}';
        }
    }

    /**
     * 生成模拟订单数据  3个订单   每个订单4个状态
     * 每个订单 创建->付款->推送->完成
     */
    private List<Order> buildOrders() {
        List<Order> orderList = new ArrayList<Order>();
        Order orderDemo = new Order();
        orderDemo.setOrderId(001);
        orderDemo.setDesc("创建");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(002);
        orderDemo.setDesc("创建");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(001);
        orderDemo.setDesc("付款");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(003);
        orderDemo.setDesc("创建");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(002);
        orderDemo.setDesc("付款");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(003);
        orderDemo.setDesc("付款");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(002);
        orderDemo.setDesc("推送");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(003);
        orderDemo.setDesc("推送");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(002);
        orderDemo.setDesc("完成");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(001);
        orderDemo.setDesc("推送");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(003);
        orderDemo.setDesc("完成");
        orderList.add(orderDemo);

        orderDemo = new Order();
        orderDemo.setOrderId(001);
        orderDemo.setDesc("完成");
        orderList.add(orderDemo);
        return orderList;
    }
}
```

```java
/**
 * 部分顺序消息消费
 */
public class ConsumerInOrder {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("OrderConsumer2");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_LAST_OFFSET);
        consumer.subscribe("PartOrder", "*");
        // MessageListenerOrderly 顺序消费监听
        consumer.registerMessageListener(new MessageListenerOrderly() {
            Random random = new Random();
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, 
                                                       ConsumeOrderlyContext context) {
                context.setAutoCommit(true);
                for (MessageExt msg : msgs) {
                    // 可以看到每个queue有唯一的consume线程来消费, 订单对每个queue(分区)有序
                    System.out.println("consumeThread=" + Thread.currentThread().getName()
                            + ",queueId=" + msg.getQueueId() + ", content:"
                            + new String(msg.getBody()));
                }
                try {
                    //模拟业务逻辑处理中...
                    TimeUnit.MILLISECONDS.sleep(random.nextInt(300));
                } catch (Exception e) {
                    e.printStackTrace();
                    //这个点要注意：意思是先等一会，一会儿再处理这批消息，而不是放到重试队列里
                    return ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });
        consumer.start();
        System.out.println("Consumer Started.");
    }
}
```

#### 【2】延时消息的生产与消费  

Producer 将消息发送到消息队列 RocketMQ 服务端，但并不期望这条消息立马投递，而是延迟一定时间后才投递到 Consumer 进行消费，该消息即延时消息。

比如在电商交易中超时未支付关闭订单的场景，在订单创建时会发送一条延时消息。这条消息将会在 30 分钟以后投递给消费者，消费者收到此消息后需要判断对应的订单是否已完成支付。 如支付未完成，则关闭订单。如已完成支付则忽略。

<img src="_media/middleware/RocketMq/延时消息图示.png" style="zoom:50%;" />

```java
/**
 * 延时消息-生产者
 */
public class ScheduledMessageProducer {
    public static void main(String[] args) throws Exception {
        // 实例化一个生产者来产生延时消息
        DefaultMQProducer producer = new DefaultMQProducer("ScheduledProducer");
        // 设置NameServer的地址
        producer.setNamesrvAddr("127.0.0.1:9876");
        // 启动Producer实例
        producer.start();
        int totalMessagesToSend = 10;
        for (int i = 0; i < totalMessagesToSend; i++) {
            Message message = new Message("ScheduledTopic", 
                    ("Hello scheduled message " + i).getBytes());
            // 设置延时等级4,这个消息将在30s之后投递给消费者(详看delayTimeLevel)
            // delayTimeLevel：(1~18个等级)"1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h"
            message.setDelayTimeLevel(4);
            // 发送消息
            producer.send(message);
        }
        // 关闭生产者
        producer.shutdown();
    }
}
```

```java
public class ScheduledMessageConsumer {
    public static void main(String[] args) throws Exception {
        // 实例化消费者
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ScheduledConsumer");
        // 指定Namesrv地址信息.
        consumer.setNamesrvAddr("127.0.0.1:9876");
        // 订阅Topics
        consumer.subscribe("ScheduledTopic", "*");
        // 注册消息监听者
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> messages, ConsumeConcurrentlyContext context) {
                for (MessageExt message : messages) {
                    // 打印消息等待了多久
                    System.out.println("Receive message[msgId=" + message.getMsgId() + "] "
                            + (message.getStoreTimestamp()-message.getBornTimestamp()) + "ms later");
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        // 启动消费者
        consumer.start();
    }
}
```

#### 【3】批量消息的生产与消费

原来发送消息是逐条发送，如果量大的情况下会有性能瓶颈，批量发送消息可以将一部分消息打包发送，能显著提高传递小消息的性能。限制是这些批量消息应该有相同的topic，相同的waitStoreMsgOK（集群时会细讲），而且不能是延时消息。此外，这一批消息的总大小不应超过4MB。

<img src="_media/middleware/RocketMq/批量消息图示.png" style="zoom:50%;" />

```java
/**
 * 批量消息-生产者  list不要超过4m
 */
public class BatchProducer {

    public static void main(String[] args) throws Exception {
        // 实例化消息生产者Producer
        DefaultMQProducer producer = new DefaultMQProducer("BatchProducer");
        // 设置NameServer的地址
        producer.setNamesrvAddr("127.0.0.1:9876");
        // 启动Producer实例
        producer.start();

        String topic = "BatchTest";
        List<Message> messages = new ArrayList<>();
        messages.add(new Message(topic, "Tag", "OrderID001", "Hello world 1".getBytes()));
        messages.add(new Message(topic, "Tag", "OrderID002", "Hello world 2".getBytes()));
        messages.add(new Message(topic, "Tag", "OrderID003", "Hello world 3".getBytes()));
        messages.add(new Message(topic, "Tag", "OrderID004", "Hello world 4".getBytes()));
        messages.add(new Message(topic, "Tag", "OrderID005", "Hello world 5".getBytes()));
        messages.add(new Message(topic, "Tag", "OrderID006", "Hello world 6".getBytes()));
        try {
            producer.send(messages);
        } catch (Exception e) {
            producer.shutdown();
            e.printStackTrace();
        }
        // 如果不再发送消息，关闭Producer实例。
        producer.shutdown();
    }
}
```

```java
/**
 * 批量消息-消费者
 */
public class BatchComuser {
    public static void main(String[] args) throws Exception {
        // 实例化消息生产者,指定组名
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("BatchComsuer");
        // 指定Namesrv地址信息.
        consumer.setNamesrvAddr("127.0.0.1:9876");
        // 订阅Topic
        consumer.subscribe("BatchTest", "*");
        //负载均衡模式消费
        consumer.setMessageModel(MessageModel.CLUSTERING);
        // 注册回调函数，处理消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                            ConsumeConcurrentlyContext context) {
                System.out.printf("%s Receive New Messages: %s %n",
                        Thread.currentThread().getName(), msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //启动消息者
        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}
```

批量切分：当消息总量超过4M甚至更大时，可以将这些消息拆分为小的包进行发送

```java
/**
 * 批量消息-超过4m-生产者
 */
public class SplitBatchProducer {

    public static void main(String[] args) throws Exception {
        // 实例化消息生产者Producer
        DefaultMQProducer producer = new DefaultMQProducer("BatchProducer");
        // 设置NameServer的地址
        producer.setNamesrvAddr("127.0.0.1:9876");
        // 启动Producer实例
        producer.start();
        String topic = "BatchTest";
        //使用List组装
        List<Message> messages = new ArrayList<>(100 * 1000);
        //10万元素的数组
        for (int i = 0; i < 100 * 1000; i++) {
            messages.add(new Message(topic, "Tag", "OrderID" + i, ("Hello world " + i).getBytes()));
        }

        //把大的消息分裂成若干个小的消息（1M左右）
        ListSplitter splitter = new ListSplitter(messages);
        while (splitter.hasNext()) {
            List<Message> listItem = splitter.next();
            producer.send(listItem);
            Thread.sleep(100);
        }
        // 如果不再发送消息，关闭Producer实例。
        producer.shutdown();
        System.out.printf("Consumer Started.%n");
    }

}

class ListSplitter implements Iterator<List<Message>> {
    private int sizeLimit = 1000 * 1000;//1M
    private final List<Message> messages;
    private int currIndex;
    public ListSplitter(List<Message> messages) { this.messages = messages; }
    @Override
    public boolean hasNext() { return currIndex < messages.size(); }
    @Override
    public List<Message> next() {
        int nextIndex = currIndex;
        int totalSize = 0;
        for (; nextIndex < messages.size(); nextIndex++) {
            Message message = messages.get(nextIndex);
            int tmpSize = message.getTopic().length() + message.getBody().length;
            Map<String, String> properties = message.getProperties();
            for (Map.Entry<String, String> entry : properties.entrySet()) {
                tmpSize += entry.getKey().length() + entry.getValue().length();
            }
            tmpSize = tmpSize + 20; // 增加日志的开销20字节
            if (tmpSize > sizeLimit) {
                if (nextIndex - currIndex == 0) {//单个消息超过了最大的限制（1M），否则会阻塞进程
                    nextIndex++; //假如下一个子列表没有元素,则添加这个子列表然后退出循环,否则退出循环
                }
                break;
            }
            if (tmpSize + totalSize > sizeLimit) { break; }
            else { totalSize += tmpSize; }
        }
        List<Message> subList = messages.subList(currIndex, nextIndex);
        currIndex = nextIndex;
        return subList;
    }
}
```

#### 【4】过滤消息的生产与消费

在消息生产时可能会带有一些标签或属性，可以根据这些标签或属性进行消费，称之为在消费端过滤，分为Tag过滤和SQL过滤。

**Tag过滤**：在消息生产的时候加入二级标签tags，在大多数情况下，TAG是一个简单而有用的设计，其可以来选择您想要的消息。

```java
/**
 * tag过滤-生产者
 */
public class TagFilterProducer {

    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("TagFilterProducer");

        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();
        //todo 设定三种标签
        String[] tags = new String[] {"TagA", "TagB", "TagC"};

        for (int i = 0; i < 3; i++) {
            Message msg = new Message("TagFilterTest",
                tags[i % tags.length],
                "Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        producer.shutdown();
    }
}
```

```java
/**
 * tag过滤-消费者
 */
public class TagFilterConsumer {

    public static void main(String[] args) throws InterruptedException, MQClientException, IOException {

        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("TagFilterComsumer");
        //todo 指定Namesrv地址信息.
        consumer.setNamesrvAddr("127.0.0.1:9876");
        //只消费TagA || TagB的消息
        consumer.subscribe("TagFilterTest", "TagA || TagB");
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                            ConsumeConcurrentlyContext context) {
                try {
                    for(MessageExt msg : msgs) {
                        String topic = msg.getTopic();
                        String msgBody = new String(msg.getBody(), "utf-8");
                        String msgPro = msg.getProperty("a");
                        String tags = msg.getTags();
                        System.out.println("收到消息：" + " topic :" + topic 
                                + " ,tags : " + tags +  " ,a : " + msgPro +" ,msg : " + msgBody);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}
```

**SQL过滤**：RocketMQ定义了一些基本语法来支持这个特性。你也可以很容易地扩展它。只有使用push模式的消费者才能用使用SQL92标准的sql语句

```
数值比较：比如：>，>=，<，<=，BETWEEN，=；
字符比较：比如：=，<>，IN；
IS NULL 或者 IS NOT NULL；
逻辑符号：AND，OR，NOT；
常量支持类型为：
数值，比如：123，3.1415；
字符，比如：'abc'，必须用单引号包裹起来；
NULL，特殊的常量
布尔值，TRUE 或 FALSE
```

```java
/**
 * sql过滤 -消息生产者（加入消息属性)
 */
public class SqlFilterProducer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("SqlFilterProducer");
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();
        String[] tags = new String[] {"TagA", "TagB", "TagC"};
        for (int i = 0; i < 10; i++) {
            Message msg = new Message("SqlFilterTest",
                tags[i % tags.length],
                ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET)
            );
            // 设置SQL过滤的属性
            msg.putUserProperty("a", String.valueOf(i));
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        producer.shutdown();
    }
}
```

```java
/**
 * sql过滤-消费者
 */
public class SqlFilterConsumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("SqlFilterConsumer");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.subscribe("SqlFilterTest",
            MessageSelector.bySql("(TAGS is not null and TAGS in ('TagA', 'TagB'))" +
                "and (a is not null and a between 0 and 3)"));
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                ConsumeConcurrentlyContext context) {
                try {
                    for(MessageExt msg : msgs) {
                        String topic = msg.getTopic();
                        String msgBody = new String(msg.getBody(), "utf-8");
                        String msgPro = msg.getProperty("a");
                        String tags = msg.getTags();
                        System.out.println("收到消息：" + " topic :" + topic + " ,tags : " + tags +  " ,a : " + msgPro +" ,msg : " + msgBody);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}
```

### 4、消息发送/消费的方法与属性

#### 【1】消息发送 - 重要属性

org.apache.rocketmq.example.details. ProducerDetails 类中

<img src="_media/middleware/RocketMq/消息发送的重要属性.png"/>

<img src="_media/middleware/RocketMq/消息发送的重要方法.png"/>

#### 【2】消息发送 - 重要方法（单向）

<img src="_media/middleware/RocketMq/单向发送重要方法.png"/>

#### 【3】消息发送 - 重要方法（同步）

<img src="_media/middleware/RocketMq/同步发送重要方法.png"/>

#### 【4】消息发送 - 重要方法（异步）

<img src="_media/middleware/RocketMq/异步发送重要方法1.png"/>

<img src="_media/middleware/RocketMq/异步发送重要方法2.png"/>

<img src="_media/middleware/RocketMq/异步发送重要方法3.png"/>

#### 【5】消息消费 - 重要属性

org.apache.rocketmq.example.details. ComuserDetails 类中

<img src="_media/middleware/RocketMq/消息消费重要属性.png"/>

#### 【6】消息消费 - 重要方法

<img src="_media/middleware/RocketMq/消息消费重要方法.png"/>

并发消息事件监听器

<img src="_media/middleware/RocketMq/注册并发消息事件监听器.png"/>

顺序消息事件监听器

<img src="_media/middleware/RocketMq/注册顺序消息事件监听器.png"/>

#### 【7】消息消费 - 消费确认（ACK）

1. 业务实现消费回调的时候，当且仅当此回调函数返回`ConsumeConcurrentlyStatus.CONSUME_SUCCESS`，RocketMQ才会认为这批消息（默认是1条）是消费完成的中途断电，抛出异常等都不会认为成功——即都会重新投递。
2. 返回`ConsumeConcurrentlyStatus.RECONSUME_LATER`，RocketMQ就会认为这批消息消费失败了。
3. 如果业务的回调没有处理好而抛出异常，会认为是消费失败`ConsumeConcurrentlyStatus.RECONSUME_LATER`处理。
4. 为了保证消息是肯定被至少消费成功一次，RocketMQ会把这批消息重发回Broker（topic不是原topic而是这个消费组的RETRY topic），在延迟的某个时间点（默认是10秒，业务可设置）后，再次投递到这个ConsumerGroup。而如果一直这样重复消费都持续失败到一定次数（默认16次），就会投递到DLQ死信队列。应用可以监控死信队列来做人工干预。
5. 另外如果使用顺序消费的回调MessageListenerOrderly时，由于顺序消费是要前者消费成功才能继续消费，所以没有`RECONSUME_LATER`的这个状态，只有`SUSPEND_CURRENT_QUEUE_A_MOMENT`来暂停队列的其余消费，直到原消息不断重试成功为止才能继续消费

### 5、集群部署

#### 【1】集群部署模式

1. 单master模式：只有一个 master 节点，称不上是集群，一旦这个 master 节点宕机，那么整个服务就不可用。

   ```
   rocketmq-all-... > distribution > conf 目录
    - 2m-2s-async
    - 2m-2s-sync
    - 2m-noslave
   ```

2. 多master模式：多个 master 节点组成集群，单个 master 节点宕机或者重启对应用没有影响。

   1. 优点：所有模式中性能最高（一个Topic的可以分布在不同的master，进行横向拓展）在多主的架构体系下，无论使用客户端还是管理界面创建主题，一个主题都会创建多份队列在多主中（默认是4个的话，双主就会有8个队列，每台主4个队列，所以双主可以提高性能，一个Topic的分布在不同的master，方便进行横向拓展。
   2. 缺点：单个 master 节点宕机期间，未被消费的消息在节点恢复之前不可用，消息的实时性就受到影响。

   ```properties
   ※ 多主节点配置，在 2m-noslave 目录下，分别代表两个节点的配置文件
    - broker-a.properties 
    - broker-b.properties
   ========================== broker-a.properties ==========================
   brokerClusterName=DefaultCluster # 集群名称需要一致
   brokerName=broker-a
   brokerId=0 # brokerId为0表示都为主节点
   deleteWhen=04
   fileReservedTime=48
   brokerRole=ASYNC_MASTER
   flushDiskType=ASYNC_FLUSH
   ```

3. 多master多slave模式（同步）：当消息发送到同步节点上时，需要等到从节点复制完成后再返回给生产者确认。

   ```properties
   ※ 多master多slave模式（同步）配置，在 master2m-2s-sync 目录下
    - broker-a.properties
    - broker-a-s.properties
    - broker-b.properties
    - broker-b-s.properties
   ========================== broker-a.properties ==========================
   brokerClusterName=DefaultCluster
   brokerName=broker-a
   brokerId=0
   deleteWhen=04
   fileReservedTime=48
   brokerRole=SYNC_MASTER
   flushDiskType=ASYNC_FLUSH
   ========================== broker-a-s.properties ==========================
   brokerClusterName=DefaultCluster
   brokerName=broker-a
   brokerId=1
   deleteWhen=04
   fileReservedTime=48
   brokerRole=SLAVE
   flushDiskType=ASYNC_FLUSH
   ========================== broker-b.properties ==========================
   brokerClusterName=DefaultCluster
   brokerName=broker-b
   brokerId=0
   deleteWhen=04
   fileReservedTime=48
   brokerRole=SYNC_MASTER
   flushDiskType=ASYNC_FLUSH
   ========================== broker-b.properties ==========================
   brokerClusterName=DefaultCluster
   brokerName=broker-b
   brokerId=1
   deleteWhen=04
   fileReservedTime=48
   brokerRole=SLAVE
   flushDiskType=ASYNC_FLUSH
   ```

4. 多master多slave模式（异步）：从节点（Slave）就是复制主节点的数据，对于生产者完全感知不到，对于消费者正常情况下也感知不到。（只有当Master不可用或者繁忙的时候，Consumer会被自动切换到从Slave 读。）

   1. 在多 master 模式的基础上，每个 master 节点都有至少一个对应的 slave。master节点可读可写，但是 slave只能读不能写，类似于 mysql 的主备模式。
   2. 优点： 一般情况下都是master消费，在 master 宕机或超过负载时，消费者可以从 slave 读取消息，消息的实时性不会受影响，性能几乎和多 master 一样。
   3. 缺点：使用异步复制的同步方式有可能会有消息丢失的问题。（Master宕机后，生产者发送的消息没有消费完，同时到Slave节点的数据也没有同步完）
