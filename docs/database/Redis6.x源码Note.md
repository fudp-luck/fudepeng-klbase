## 一、Redis 6.x新老特性

### 1、前置

1. Redis 源码下载地址：https://download.redis.io/releases/redis-6.2.6.tar.gz
2. 源码工具：Source Inight `projec -> new project -> [select redis src] -> add tree -> close`

### 2、源码前置铺垫

<img src="_media/database/Redis6.x源码-Redisson/源码前置铺垫.png"/>

1. 首先 Redis 的入口是 server.c
2. 在启动了之后定义了 `redisCommandTable[]` 数组，这里面包含了 redis 所有的命令，数组中包含了一堆 redisCommand ，redisCommand 中包含了处理的方法 redisCommandProc
3. 此外还需要关注的内容：
   1. `ae [EventLoop / A simple event-driven]` 是流程中最核心的内容
   2. `anet `定义了 TCP 相关内容
   3. `networking` ，io_threads 相关内容
   4. `zmalloc `，内存分配的包装（申请内存空间），要注意在 C 语言中有指针，在 Java 中是 new 的，不用自己去处理 gc，在 C 中要自己维护内存，可以理解为 Java 中的 new 和 gc 就是围绕 zmalloc 的调用
   5. `afterSleep / beforeSleep` ，睡前做什么、睡后做什么，sleep 阻塞：在等待 IO 事件时会阻塞 `epoll_wait(time)`

### 3、server.c 中的 main 方法

#### 【1】initServer(); 初始化服务

<img src="_media/database/Redis6.x源码-Redisson/initServer(); 初始化服务.png"/>

1. 在 server.c 的 main 方法中，第一个需要关注的是 initServer(); 初始化服务，在这个函数中创建了以下内容
2. 创建了 EventLoop 事件循环器：`aeCreateEventLoop(server.maxclients+CONFIG_FDSET_INCR);`
3. 定长创建 redis 的 16 个库，每个库也是一个结构体：`zmalloc(sizeof(redisDb)*server.dbnum);`
4. anet.c 中建立 TCP 连接，创建了6379端口，能够监听了：`listenToPort(server.port,&server.ipfd) == C_ERR)`
5. 创建时间事件：`aeCreateTimeEvent(server.el, 1, serverCron, NULL, NULL)`
6. 接受客户端的 handler：`createSocketAcceptHandler(&server.ipfd, acceptTcpHandler)`
7. `aeCreateFileEvent`：EventLoop 就是在循环处理这些事件：FileEvent \ TimeEvent
8. `aeSetBeforeSleepProc(server.el,beforeSleep);` / ` aeSetAfterSleepProc(server.el,afterSleep);`

#### 【2】acceptTcpHandler 方法

<img src="_media/database/Redis6.x源码-Redisson/acceptTcpHandler .png"/>

- acceptTcpHandler 方法创建了客户端的连接，并将这个函数封装成 client 对象

#### 【3】createSocketAcceptHandler 方法

<img src="_media/database/Redis6.x源码-Redisson/createSocketAcceptHandler .png"/>

1. 首先`createSocketAcceptHandler(&server.ipfd, acceptTcpHandler)`方法参数中的 acceptTcpHandler 就是上述封装好的 client 对象

2. 之后创建 IO 事件`aeCreateFileEvent(server.el, sfd->fd[j], AE_READABLE, accept_handler,NULL)`

   - 在单线程中 EventLoop 事件循环器是核心，创建事件最终都会和 EventLoop 关联起来，此处就是创建了一个监听的 IO 事件，并绑定到 EventLoop 上

     ```
     参数：
     server.el：EventLoop
     sfd->fd[j]：监听6379的fd
     AE_READABLE：IO 事件的类型 
     event_typeaccept_handler：封装好的 Client 对象
     ```

3. createFileEvent 创建的就是将监听的 event 封装成 FileEvent 结构体，得到 mask 类型后处理回调，和 handle 绑定起来

   1. `aeFileEvent \*fe = &eventLoop->events[fd];`

   2. aeFileEvent 是一个结构体，FileEvent 可以理解为 IOEvent 当前创建出的 IOEvent 就是 6379 的监听 socket

   3. 当传进来的 mask 是 AE_READABLE 只读的，就可以通过 rfileProc 调用传进来 Client 对象方法

      ```c
      typedef struct aeFileEvent {    
          int mask; /* one of AE_(READABLE|WRITABLE|BARRIER) */    
          aeFileProc *rfileProc;    
          aeFileProc *wfileProc;    
          void *clientData;
      } aeFileEvent;
      ```

4. 之后将系统文件描述符加到 EventLoop 中`aeApiAddEvent(eventLoop, fd, mask)`

   - 这一步有四种实现方式，表示在哪个系统中优先使用哪种多路复用器

     ```c
     aeApiAddEvent - Function in ae_select.c at line 62 (win)
     aeApiAddEvent - Function in ae_epoll.c at line 74 (Linux)
     aeApiAddEvent - Function in ae_kqueue.c at line 102 (unix)
     aeApiAddEvent - Function in ae_evport.c at line 155
     ```

5. 最后将上面对 Listen 的封装注册到 epoll 多路复用器中 `epoll_ctl(state->epfd,op,fd,&ee)`

6. 此时 EventLoop 上已经绑定了 Listen 的 FileEvent，还有 beforeSleep，afterSleep 睡前睡后要处理的函数

7. 总结出 `initServer();` 做了事件循环器，将监听Listen绑定到事件循环器，并把beforeSleep，afterSleep注册到事件循环器上

#### 【4】InitServerLast 方法 - IO Thread

<img src="_media/database/Redis6.x源码-Redisson/InitServerLast .png"/>

- IOThreadMain

<img src="_media/database/Redis6.x源码-Redisson/IOThreadMain.png"/>

1. IO 线程就是盯着自身队列中的客户端

2. 在 Redis 中主线程是工作线程，而开启一个 IO Thread 就只负责读写行为，读取客户端发的内容

3. 整个流程就是由多个 IO 线程并行读进来，之后有工作线程处理，最后由 IO 线程发出去

   <img src="_media/database/Redis6.x源码-Redisson/Redis6多线程模型.png"/>

4. IO Thread 做得事情：开辟了一堆 IO 线程，每个线程将来去处理自己的若干个客户端连接数

#### 【5】IO Thread 配置文件

```shell
# 配置文件
# 配置 IO 线程数且不能大于128，主要关注于写线程
io-threads 4   
# 开启读线程，默认是关闭的，官方注释（Usually threading reads doesn't  help much 如果开启读线程没有任何帮助）
io-threads-do-reads yes  
# 如果要做redis性能压测的话使用且要带着写线程配置数
redis-benchmark --threads 4  
```

1. 网络行为是非对称的，就是请求 request 体积很小，响应 response 体积很大
2. Redis 读的过程是要将值发送到内核中，当体积比较大的时候开启多线程读是有益的，但体积小的时候开启多个线程多个 CPU 去处理的优化效果不大
3. 是否开启读线程，取决于写场景多不多，如果只是只读型的就没有必要开启，当吞吐压力比较大，set value 数据比较大能够享受到多线程优势时开启
4. 类似于 redis 这类中间件尽量不要去动配置，有时不改动配置是最优的

#### 【6】reactor 响应器模型

1. 单线程响应器模型：6.x 之前的 Redis

   - 一个线程有有一个多路复用器，所做的事情连带着接收客户端、读写、task、其他 event 事件处理，都在一个线程中处理完了

     <img src="_media/database/Redis6.x源码-Redisson/单线程响应器模型.png"/>

2. 多线程线程响应器模型：Netty

   - Netty 中多线程响应器模型，每个线程都有多路复用器，比较有代表性的：Kafka、RocketMq

     <img src="_media/database/Redis6.x源码-Redisson/多线程响应器模型.png"/>

3. 单线程响应器模型：Redis6.x

   - Redis 其实还是单线程响应模型，只有主线程有多路复用器，用一些 thread 线程持有了一批 client，读线程可以选择性开启

     <img src="_media/database/Redis6.x源码-Redisson/单线程响应redis6.png"/>

#### 【7】aeMain(server.el);

<img src="_media/database/Redis6.x源码-Redisson/aeMain.png"/>

- aeProcessEvents 流程

<img src="_media/database/Redis6.x源码-Redisson/aeMain内部流程.png"/>  ，-

- `aeMain(server.el); `就是开始执行 main 方法
- Redis 的底层绕不开 EventLoop

## 二、Redis 基本类型源码

### 1、aeProcessEvents 整体流程

<img src="_media/database/Redis6.x源码-Redisson/aeProcessEvents .png"/>

- aeProcessEvents 代码对照

<img src="_media/database/Redis6.x源码-Redisson/aeProcessEvents代码对照.png"/>



### 2、beforSleep 方法

<img src="_media/database/Redis6.x源码-Redisson/beforesleep.png"/>

1. 首先在 beforesleep 函数中有两个很重要的方法，这两个方法在 networking.c 中

   1. 客户端写事件需要使用的一些线程 `handleClientsWithPendingWritesUsingThreads();`
   2. 客户端读事件需要使用的一些线程 `handleClientsWithPendingReadsUsingThreads();`

2. 其次，在 networking.c 中有一个很重要的函数 IOThreadMain

   ```c
       while(1) {        
           /* Wait for start */        
           for (int j = 0; j < 1000000; j++) {            
               if (getIOPendingCount(id) != 0) break;        
           }        
           /* Give the main thread a chance to stop this thread. */        
           if (getIOPendingCount(id) == 0) {            
               pthread_mutex_lock(&io_threads_mutex[id]);            
               pthread_mutex_unlock(&io_threads_mutex[id]);            
               continue;        
           }    
           ......    
       }
   ```

   - 源码中表示：当 getIOPendingCount(id) == 0 时会一直循环，也就是说这个线程是没有让出 CPU 的，等于这个 IO 线程会在 CPU 上空转，但不会不好，因为 Redis 的响应堆在主线程，所有的处理也在主线程，它一颗 CPU 就够了，如果一台机器是四核，那么其他三颗 CPU 是空着的，一直跑这个循环也没有什么问题，它会一直霸占这个 CPU 一旦有一些读取的事件使判断大于零，那么就可以往下走了，发现的比较及时，不需要等待

3. 也就是说 `handleClientsWithPendingWritesUsingThreads();` 和 `handleClientsWithPendingReadsUsingThreads();`会将想写想读的客户端添加到 IO Thread List 中，同时会向 IOPendingCount 中记录客户端数量

4. 而每个线程通过线程 ID 从 IO Thread List 取到分配给自己的客户端，每个 IO Thread 都在等待 getIOPendingCount(id) 值不等零，之后进入下面的逻辑

### 3、aeApiPoll 方法

<img src="_media/database/Redis6.x源码-Redisson/aeApiPoll.png"/>

- 服务端启动后，拿到相应的 Listen 事件，更新 mask 的值为只读，之后进行赋值，用于后序处理

