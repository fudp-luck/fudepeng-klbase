## 一、介绍Hippo4j

线程池的本质就是减少创建和销毁线程带来的一些资源损耗，在工作中通常需要设置一个合理的线程池参数。

如果线程数设置过多，CPU在调度线程时的上下文切换成本太高；如果设置过少，没有充分发挥CPU的资源性能，浪费硬件资源。

一些高并发的处理，以及一个比较大的批处理，离不开线程池的操作。

#### 【1】使用线程池存在的一些痛点

* 线程池不能随便定义，线程数多了任务慢，少了资源利用不到位。
* 线程池参数的评估成本太高，测试过程麻烦，而且如果出现变动，还需要重新部署。
* 线程池中的线程在执行任务，如果超时了，无法感知，无法快速处理这个问题。
* 如果出现了流量激增，可能大量的任务会堆积到阻塞队列，或者触发拒绝策略，影响业务。

#### 【2】痛点解决

Hippo4j监控工具就是解决上述这些问题的，Hippo4j基于对原生的线程池ThreadPoolExecutor做了一个增强，扩展了很多的功能。

* 所有的线程池都可以交给Hippo4j管理。
* Hippo4j可以动态的修改线程池的核心参数，以更好的适配硬件资源，提升处理速度。
* Hippo4j还提供了报警的功能，快速发现问题，快速解决问题。
* 还可以实时的监控到线程池运行时的一些数据信息。
* Hippo4j还针对一些框架的线程次，也提供监控功能
  * Web容器的线程池……
  * Hystrix的线程池……
  * RabbitMQ或者是RocketMQ的线程池……

Hippo4j的Gitee地址：https://gitee.com/opengoofy/hippo4j

可以直接基于提供的演示地址去查看效果：http://console.hippo4j.cn/index.html

## 二、Hippo4j快速入门

Hippo4j 中有两个角色，一个Sever，一个Client

Server就是Hippo4j的图形化界面

Client就是自己的工程，自己的项目，需要正常的引入Hippo4j的依赖，将构建线程池的方式更改为Hippo4j的要求的形式，再编写好各种配置文件即可。

### 1、安装Hippo4j-Server

Hippo4j-Server提供很多中安装方式，其中以Docker为核心去安装。

#### 【1】Docker方式安装

docker启动的方式是最easy的，镜像中已经集成好了几乎所有环境

提供一个 `docker-compose.yml` 文件即可。

```yml
version: '3.1'
services:
  hippo4j:
    image: hippo4j/hippo4j-server
    container_name: hippo4j
    ports:
      - 6691:6691
```

`docker-compose up -d` 启动即可

访问http://host:6691/index.html，默认用户名和密码为admin/123456
<img src="_media/tools/Hippo4j/Hippo4j控制台.png"/>

#### 【2】 源码的方式安装

会直接从Gitee上下载源码，下载源码时需要注意版本。

正常的解压，并且编译。

准备好需要的环境，直接运行起来即可

##### 下载源码<img src="_media/tools/Hippo4j/下载源码.png"/>

##### 编译源码

如果依赖下载失败，主要看自己的Maven配置。

##### 启动前准备环境

Hippo4j-Server提供了两种数据源，可以采用MySQL，也可以采用H2。

这边咱们直接采用MySQL作为数据源。

找到指定的工程，准备环境

<img src="_media/tools/Hippo4j/准备环境.png"/>

在指定的地址的MySQL中，提前构建出hippo4j_manager库

<img src="_media/tools/Hippo4j/指定MySQL.png"/>

回到启动类，启动项目即可。

<img src="_media/tools/Hippo4j/启动项目.png"/>

启动成功后，会自动根据resources中提供的SQL脚本生成库表

<img src="_media/tools/Hippo4j/生成SQL脚本.png"/>

##### 访问具体的Web界面

直接访问本地的6691端口即可。默认的用户名和密码依然是admin/123456

<img src="_media/tools/Hippo4j/访问控制台.png"/>

### 2、实现Hippo4j-Client

#### 【1】本地Linux安装Hippo4j-Server

因为Hippo4j-Client和Hippo4j-Server之间需要相互通讯。

所以之前我的Hippo4j-Server的Docker安装方式，是基于云服务安装的，用着麻烦。

在本地构建一个虚拟机，在本的Linux上再次安装一份Hippo4j-Server

同时基于Docker的方式，指定好使用的MySQL，方便一会Client注意以及修改线程池信息时，查看数据库的数据变化

```yml
version: '3.1'
services:
  hippo4j:
    image: hippo4j/hippo4j-server
    container_name: hippo4j
    ports:
      - 6691:6691
    environment:
      - DATASOURCE_MODE=mysql
      - DATASOURCE_HOST=192.168.11.1
      - DATASOURCE_PORT=3306
      - DATASOURCE_DB=hippo4j_manager
      - DATASOURCE_USERNAME=root
      - DATASOURCE_PASSWORD=root
```

#### 【2】构建Hippo4j-Client（无中间件依赖）

首先需要准备一个SpringBoot工程。

导入Hippo4j的client依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>cn.hippo4j</groupId>
        <artifactId>hippo4j-spring-boot-starter</artifactId>
        <version>1.5.0</version>
    </dependency>
</dependencies>
```

启动类追加注解

```java
@EnableDynamicThreadPool
```

编写配置文件

```java
spring:
  application:
    name: hippo4j-client
  dynamic:
    thread-pool:
      server-addr: 192.168.11.88:6691
      username: admin
      password: 123456
      namespace: mashibing   # 租户名称
      item-id: ${spring.application.name}   # 项目名称，需要与与服务名称保持一致。
```

配置线程池信息

```java
/**
 * 交给Hippo4j-Server管理的线程池
 * @author zjw
 * @description
 */
@Configuration
public class ThreadPoolConfig {

    @Bean
    @DynamicThreadPool
    public ThreadPoolExecutor testThreadPool(){
        //1、 采用线程池Builder去构建
        ThreadPoolExecutor testThreadPool = ThreadPoolBuilder.builder()
                .corePoolSize(10)
                .maximumPoolSize(10)
                .keepAliveTime(10)
                .timeUnit(TimeUnit.SECONDS)
                .workQueue(BlockingQueueTypeEnum.RESIZABLE_LINKED_BLOCKING_QUEUE)
                .threadFactory("test")
                .rejected(new ThreadPoolExecutor.AbortPolicy())
                .threadPoolId("test")
                .dynamicPool()
                .build();
        return testThreadPool;
    }
}
```

需要在Hippo4j的图形化界面上构建租户&项目

<img src="_media/tools/Hippo4j/构建租户&项目.png"/>

<img src="_media/tools/Hippo4j/构建租户&项目2.png"/>

动态线程池

* **线程池管理：** 这边可以将新注册上来的Hippo4j-Client中的线程池信息读取并且保存到数据库中，后期出现同样的线程池的标识的，直接按照当前数据库保存的配置信息走。 他无法动态修改。<img src="_media/tools/Hippo4j/动态线程池管理.png"/>
* **线程池实例：** 这个才是动态修改线程池配置信息的选项。<img src="_media/tools/Hippo4j/线程池实例.png"/>

#### 【3】构建Hippo4j-Client（依赖配置中心）

依赖配置中心的方式和上面玩的无中间件依赖是两个套路。

其实区别在于：

* 依赖配置中心方式，可以不安装Hippo4j-Server，只需要在项目中导入好依赖，就可以实现线程池的动态配置刷新效果。 如果需要实现监控界面报表等内容，需要接入Prometheus和Grafana，成本也不低。
* 无中间件依赖，这种方式只需要额外安装部署Hippo4j-Server即可，功能相对依赖配置中心的更丰富。

<img src="_media/tools/Hippo4j/依赖配置中心.png"/>

### 3、Web容器线程池监控

tomcat这种Web容器的线程池，Hippo4j也可以实现监控，而且不需要额外的导入任何的依赖

为了更好的查看Hippo4j的动态修改效果，额外追加一个Controller查看效果

```java
@GetMapping("/web")
public String web(){
    System.out.println(Thread.currentThread().getName());
    return "ok";
}
```

访问接口时，可以查看到处理Controller业务的线程是Tomcat的线程池中的线程信息

<img src="_media/tools/Hippo4j/查看Tomcat线程池中的线程信息.png"/>

不需要额外的配置，在项目启动后，可以直接在Hippo4j的Server界面查看到监控的容器信息<img src="_media/tools/Hippo4j/Hippo4j的Server.png"/>

同时Hippo4j也支持针对web容器的线程池的设置的动态修改

<img src="_media/tools/Hippo4j/动态修改线程池.png"/>

### 4、框架线程池监控

Hippo4j支持多种框架的线程池监控功能

其中包括，Dubbo、Hystrix、RabbitMQ、RocketMQ同时还支持SpringCloud中的Stream组件。

以RabbitMQ为测试的效果，来进行监控和修改。 **监控的是消费者消费消息时的线程池信息。**

#### 【1】准备RabbitMQ环境

安装RabbitMQ服务，开启图形化界面，准备一个Queue

1、安装RabbitMQ服务

```yml
version: '3.1'
services:
  rabbitmq:
    image: rabbitmq:3.8.5
    container_name: rabbitmq
    ports:
      - 5672:5672
      - 15672:15672
```

`docker-compose up -d` 启动即可

<img src="_media/tools/Hippo4j/docker启动.png"/>

2、开启图形化界面

进入到容器内部，enable自带的一个插件

```sh
# 进入到RabbitMQ容器内部
docker exec -it rabbitmq bash
# 直接执行
rabbitmq-plugins enable rabbitmq_management
```

在本地环境访问：http://你的IP:15672/

默认用户名和密码：guest/guest

<img src="_media/tools/Hippo4j/RabbitMq控制台.png"/>

3、准备一个Queue，测试消费者的线程池监控

<img src="_media/tools/Hippo4j/消费者线程池监控.png"/>

#### 【2】在hippo4j-client追加RabbitMQ配置

1、导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

2、编写配置文件

```yml
spring:
  rabbitmq:
    host: 192.168.11.88
    port: 5672
    username: guest
    password: guest
    virtual-host: /
```

3、编写配置类

```java
@Configuration
public class RabbitMQThreadPoolConfig {

    /**
     * 构建消费者要用到的线程
     * @return
     */
    @Bean
    public ThreadPoolTaskExecutor rabbitThreadPool(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("rabbit-");
        return executor;
    }

    /**
     * 构建容器工厂，将线程池设置进入
     * @param connectionFactory
     * @return
     */
    @Bean
    public AbstractRabbitListenerContainerFactory<?> defaultContainerFactory(ThreadPoolTaskExecutor rabbitThreadPool,AbstractConnectionFactory connectionFactory){
        DirectRabbitListenerContainerFactory factory = new DirectRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        connectionFactory.setExecutor(rabbitThreadPool);
        return factory;
    }
}
```

4、构建消费者

```java
@Component
public class TestListener {

    @RabbitListener(queues = "hippo4j",containerFactory = "defaultContainerFactory")
    public void consume(String message){
        System.out.println(Thread.currentThread().getName() + "：消费消息 --> " + message);
    }
}
```

5、查看消费消息的效果

<img src="_media/tools/Hippo4j/查看消息消费效果.png"/>

#### 【3】实现hippo4j监控

方式很简单，只需要导入指定的依赖即可。

根据官方文档中的提升，可以单独引入具体的三方框架依赖的支持，也可以直接引入全家桶。

单独引入方式

```xml
<dependency>
    <groupId>cn.hippo4j</groupId>
    <!-- Dubbo -->
    <artifactId>hippo4j-spring-boot-starter-adapter-dubbo</artifactId>
    <!-- Alibaba Dubbo -->
    <artifactId>hippo4j-spring-boot-starter-adapter-alibaba-dubbo</artifactId>
    <!-- Hystrix -->
    <artifactId>hippo4j-spring-boot-starter-adapter-hystrix</artifactId>
    <!-- RabbitMQ -->
    <artifactId>hippo4j-spring-boot-starter-adapter-rabbitmq</artifactId>
    <!-- RocketMQ -->
    <artifactId>hippo4j-spring-boot-starter-adapter-rocketmq</artifactId>
    <!-- SpringCloud Stream RocketMQ -->
    <artifactId>hippo4j-spring-boot-starter-adapter-spring-cloud-stream-rocketmq</artifactId>
    <!-- SpringCloud Stream RabbitMQ -->
    <artifactId>hippo4j-spring-boot-starter-adapter-spring-cloud-stream-rabbitmq</artifactId>
    <version>1.5.0</version>
</dependency>
```

全家桶

```xml
<dependency>
    <groupId>cn.hippo4j</groupId>
    <artifactId>hippo4j-spring-boot-starter-adapter-all</artifactId>
    <version>1.5.0</version>
</dependency>
```

引入依赖ok后，直接重新启动项目即可。<img src="_media/tools/Hippo4j/启动项目-查看控制台.png"/>

动态修改的效果依然在：

<img src="_media/tools/Hippo4j/动态修改结果.png"/>

## 三、通知报警

Hippo4j为咱们提供了各种通知的功能。

通知又分为两类：

* 如果线程池配置改变，会通知
* 另外就是报警：
  * 任务执行超时报警
  * 活跃度报警，线程的活跃度
  * 容量报警，工作队列中的任务达到多少就报警
  * 线程池执行了拒绝策略，也会报警

在线程池管理的位置，配置上报警信息

<img src="_media/tools/Hippo4j/配置报警信息.png"/>

## 四、压测调整线程池参数

### 1、环境准备

要准备两套业务，一套CPU密集，一套IO密集。

为了压测还需要准备jmeter。

一套CPU密集的业务，两个线程，每个线程做CAS对Atomic自增10W次

在准备业务前，先准备两套线程池做不同业务的处理

```java
@Bean
@DynamicThreadPool
public ThreadPoolExecutor ioThreadPool() {
    //1、 采用线程池Builder去构建
    ThreadPoolExecutor testThreadPool = ThreadPoolBuilder.builder()
            .corePoolSize(10)
            .maximumPoolSize(10)
            .keepAliveTime(10)
            .timeUnit(TimeUnit.SECONDS)
            .workQueue(BlockingQueueTypeEnum.RESIZABLE_LINKED_BLOCKING_QUEUE)
            .threadFactory("io")
            .rejected(new ThreadPoolExecutor.AbortPolicy())
            .threadPoolId("io")
            .dynamicPool()
            .build();
    return testThreadPool;
}

@Bean
@DynamicThreadPool
public ThreadPoolExecutor cpuThreadPool() {
    //1、 采用线程池Builder去构建
    ThreadPoolExecutor testThreadPool = ThreadPoolBuilder.builder()
            .corePoolSize(10)
            .maximumPoolSize(10)
            .keepAliveTime(10)
            .timeUnit(TimeUnit.SECONDS)
            .workQueue(BlockingQueueTypeEnum.RESIZABLE_LINKED_BLOCKING_QUEUE)
            .threadFactory("cpu")
            .rejected(new ThreadPoolExecutor.AbortPolicy())
            .threadPoolId("cpu")
            .dynamicPool()
            .build();
    return testThreadPool;
```

CPU密集业务

```java
@Override
public Long doSomeCPUThing() throws InterruptedException {
    AtomicLong atomicLong = new AtomicLong(0);
    CountDownLatch latch = new CountDownLatch(2);
    cpuThreadPool.execute(() -> {
        incr(atomicLong);
        latch.countDown();
    });
    cpuThreadPool.execute(() -> {
        incr(atomicLong);
        latch.countDown();
    });
    latch.await();
    return atomicLong.get();
}
private void incr(AtomicLong atomicLong){
    for (int i = 0; i < 500000; i++) {
        atomicLong.incrementAndGet();
    }
}
```

另一套IO密集业务让业务做三次远程调用，做线程的sleep模拟效果

```java
@Override
public String doIOSomeThing() throws Exception{
    CountDownLatch latch = new CountDownLatch(3);
    Future<String> job1Result = ioThreadPool.submit(() -> {
        String result1 = job1();
        latch.countDown();
        return result1;
    });
    Future<String> job2Result = ioThreadPool.submit(() -> {
        String result2 = job2();
        latch.countDown();
        return result2;
    });
    Future<String> job3Result = ioThreadPool.submit(() -> {
        String result3 = job3();
        latch.countDown();
        return result3;
    });
    latch.await();
    return job1Result.get() + job1Result.get() + job1Result.get();
}
@SneakyThrows
private String job1() {
    Thread.sleep(100);
    return "RedisResult!";
}
@SneakyThrows
private String job2() {
    Thread.sleep(200);
    return "ServiceResult!";
}
@SneakyThrows
private String job3() {
    Thread.sleep(200);
    return "MySQLResult!";
}
```

---

准备Jmeter压测工具，直接官网的干活：https://jmeter.apache.org/
或者直接点这，下载：https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.5.zip

下载完毕后，确保有JAVA_HOME的环境变量后，直接打开即可。

<img src="_media/tools/Hippo4j/打开JMeter.png"/>

先添加线程组，准备测试

<img src="_media/tools/Hippo4j/添加线程组.png"/>

准备每次10个用户发请求，测试效果

<img src="_media/tools/Hippo4j/发送用户请求.png"/>

在线程组下面指定发送HTTP请求<img src="_media/tools/Hippo4j/指定发送HTTP请求.png"/>

指定好访问地址后，追加查看压测结果的内容

<img src="_media/tools/Hippo4j/追加查看压测结果.png"/>

<img src="_media/tools/Hippo4j/追加查看压测结果2.png"/>

### 2、 Windows环境下测试

测试前，先要清楚当前服务器的硬件条件，针对核心线程就是查看CPU内核数即可

<img src="_media/tools/Hippo4j/Windows环境下测试.png"/>

以12核为例，同时要请求，Windows操作系统，也不是service的专业服务器系统，他默认CPU资源被占用的就很多，测试不到特别真实的效果。过程是可以经历的。

压测CPU密集的任务，查看最终效果：

<img src="_media/tools/Hippo4j/CPU密集任务.png"/>

Windows测试是为了知道大概的测试流程，最终测试，扔到Linux上去跑！

### 3、 将服务部署到Linux中测试

#### 【1】将项目发布到Linux环境

准备好云服务器，以2C2G为例，内存无所谓，主要是看CPU内核数。

如果不清楚自己Linux服务器的内核数，可以查看

```shell
cat /proc/cpuinfo
```

需要将当前可以运行的服务，修改好环境配置，扔到Linux中运行起来

将rabbitMQ先关的都注释掉了。

首先给项目准备两个环境，将test环境连接Hippo4j的信息指定好

<img src="_media/tools/Hippo4j/准备两个环境.png"/>

打包前，还需要导入一个插件。在pom.xml文件中

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

直接打包

<img src="_media/tools/Hippo4j/打包.png"/>

将打好的jar扔到Linux环境中（Linux准备好开放具体端口，并且要提供Java的环境变量）

运行项目前，确保云服务器环境中的hippo4j中包含了租户名和项目名。

运行项目

```powershell
java -jar hippo4j-client-1.0-SNAPSHOT.jar --spring.profiles.active=test
```

启动成功

#### 【2】测试CPU密集

单独测试的效果

<img src="_media/tools/Hippo4j/测试CPU密集.png"/>

最终得到结论，当前2C系统，3个核心线程，相对来说，平均值是最快的，并且最大值也低一些。

#### 【3】测试IO密集

单独测试IO密集时，动态调整，得到120个核心线程最快。

不过我这边都是单独一次请求测试，最好多次请求达到峰值，得到一个结论会更好。

<img src="_media/tools/Hippo4j/测试IO密集.png"/>

#### 【4】合并测试两个接口

无非是将两个HTTP请求扔到一起做测试

<img src="_media/tools/Hippo4j/合并测试接口.png"/>

再查看最终结果即可

<img src="_media/tools/Hippo4j/查看最终结果.png"/><img src="https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1682628168089/471633fd252f4907b95f7b99b3ae7ac3.png"/>