## 一、Caffeine Cache 基础

### 1、Caffeine Cache 介绍

Caffeine Cache 是一个 Java 缓存库，具备高性能和可扩展性等特点，被称为「 **本地缓存之王** 」。

Spring Boot 1.x 版本中的默认本地缓存是 Guava Cache。

但在 Spring5 （SpringBoot 2.x）后，Spring 官方放弃了 Guava Cache 作为缓存机制，而是使用性能更优秀的 Caffeine 作为默认缓存组件。

:point_right:    <a href="https://github.com/ben-manes/caffeine/wiki/Benchmarks-zh-CN" target="_blank">Caffeine Cache 官方测试报告</a>

### 2、Caffeine Cache 特点

1. 自动将数据加载到缓存中，同时也可以采用异步的方式加载。

2. 具备内存淘汰策略，可以基于频次、基于最近访问、最大容量。

3. 可以根据上一次的缓存访问或者上一次的数据写入两种方式决定缓存的过期的设置。

4. 当一条缓存数据过期了，可以自动清理，清理的时候也是异步线程来做。

5. 同时考虑到了JVM的内存管理机制，内部加入了弱引用、软引用 。

6. 当缓存数据被清理后，会收到相关的通知信息。

7. 缓存数据的写入可以传播到外部的存储。

8. 具备统计功能：被访问次数，命中，清理的个数，加载个数。

## 二、Caffeine Cache 入门

### 1、Caffeine Cache 引入

:point_right:    <a href="https://github.com/ben-manes/caffeine" target="_blank">Caffeine Cache 官网地址</a>

Maven：

```xml
   <!-- Spring boot Cache-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <!--for caffeine cache-->
        <dependency>
            <groupId>com.github.ben-manes.caffeine</groupId>
            <artifactId>caffeine</artifactId>
            <version>2.7.0</version>
        </dependency>
```

Cache是一个核心的接口，里面定义了很多方法，一般通过 Cache 的子类来使用缓存我们要使用缓存，根据官方的方法，可以通过 caffeine 这个类来获得实现Cache的类。

### 2、Caffeine Cache 简单使用

Cache是一个核心的接口，里面定义了很多方法，我们要使用缓存一般是使用Cache的的子类；

根据官方的方法，我们通过caffeine这个类来获得实现Cache的类。

```java
    public static void Cache() throws Exception {
        //构建一个新的Caffeine实例
        Cache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(100)//设置缓存中保存的最大数量
                .expireAfterAccess(3L, TimeUnit.SECONDS)//如无访问则3秒后失效
                .build();//构建Cache接口实例

        cache.put("mca", "www.mashibing.com");//设置缓存项
        cache.put("baidu", "www.baidu.com");//设置缓存项
        cache.put("spring", "www.spring.io");//设置缓存项

        //获取数据
        log.info("获取缓存[getIfPresent]:mca={}", cache.getIfPresent("mca"));
        TimeUnit.SECONDS.sleep(5);//休眠5秒

        //获取数据
        log.info("获取缓存[getIfPresent]:mca={}", cache.getIfPresent("mca"));
    }
```

最普通的一种缓存，无需指定加载方式，需要手动调用 `put();`进行加载。需要注意的是，`put();`方法对于已存在的 key 将进行覆盖。如果这个值不存在，调用 `getIfPresent();`，则会立即返回 null，不会被阻塞。

### 3、Caffeine 配置说明

|             配置项             | 说明                                                       |
| :----------------------------: | ---------------------------------------------------------- |
|  `initialCapacity=[integer]`   | 初始的缓存空间大小                                         |
|      `maximumSize=[long]`      | 缓存的最大条数                                             |
|     `maximumWeight=[long]`     | 缓存的最大权重                                             |
| `expireAfterAccess=[duration]` | 最后一次写入或访问后经过固定时间过期                       |
| `expireAfterWrite=[duration]`  | 最后一次写入后经过固定时间过期                             |
| `refreshAfterWrite=[duration]` | 创建缓存或者最近一次更新缓存后经过固定的时间间隔，刷新缓存 |
|           `weakKeys`           | 打开 key 的弱引用                                          |
|          `weakValues`          | 打开 value 的弱引用                                        |
|          `softValues`          | 打开 value 的软引用                                        |
|         `recordStats`          | 开发统计功能                                               |

### 4、过期数据的同步加载

#### 【1】cache.get();

```java
public static void CacheExpire() throws Exception {
        //构建一个新的Caffeine实例
        Cache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(100)//设置缓存中保存的最大数量
                .expireAfterAccess(3L, TimeUnit.SECONDS)//如无访问则3秒后失效
                .build();//构建Cache接口实例

        cache.put("mca", "www.mashibing.com");//设置缓存项
        cache.put("baidu", "www.baidu.com");//设置缓存项
        cache.put("spring", "www.spring.io");//设置缓存项

        TimeUnit.SECONDS.sleep(5);//休眠5秒
        //获取数据
        log.info("获取缓存[getIfPresent]:baidu={}", cache.getIfPresent("baidu"));

        // 失效处理
        log.info("获取缓存[get]获取缓存:baidu={}", cache.get("baidu", (key) -> {
            log.info("进入[失效处理]函数");
            try {
                TimeUnit.SECONDS.sleep(3);//休眠3秒
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.info("[失效处理]:mca={}", cache.getIfPresent("mca"));
            return key.toUpperCase();
        }));
        log.info("经过失效处理之后[getIfPresent]:mca={}", cache.getIfPresent("mca"));
        log.info("经过失效处理之后[getIfPresent]:baidu={}", cache.getIfPresent("baidu"));
    }
```

显示效果：

<img src="_media/database/caffeinecache/过期数据同步加载运行结果.png"/>

如果数据已经过期，然后调用了 `get();`里面的函数式接口之后，会自动的给缓存重新赋值，赋的值就是 return 返回的值。同时看打印的时间，就知道这里的重新赋值是同步的，会阻塞的。

#### 【2】Cacheloader 接口

Caffeine还提供有一个较为特殊的 Cacheloader 接口，这个接口的触发机制有些不太一样，它所采用的依然是同步的加载处理。

处理流程是：

1. 首先在`builder();`的时候写上一个函数式接口（编写重新加载数据的流程）
2. 获取数据的时候，通过`getAll();`方法触发builder中的函数式接口流程，进行重新加载数据。

```java
public static void LoadingCache() throws Exception {
        LoadingCache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(100)//设置缓存中保存的最大数量
                .expireAfterAccess(3L, TimeUnit.SECONDS)//如无访问则3秒后失效
                .build(new CacheLoader<String, String>() {
                    @Override
                    public String load(String key) throws Exception {
                        log.info("正在重新加载数据...");
                        TimeUnit.SECONDS.sleep(1);
                        return key.toUpperCase();
                    }

                });

        cache.put("mca", "www.mashibing.com");//设置缓存项
        cache.put("baidu", "www.baidu.com");//设置缓存项
        cache.put("spring", "www.spring.io");//设置缓存项

        TimeUnit.SECONDS.sleep(5);

        //创建key的列表，通过cache.getAll()拿到所有key对应的值
        ArrayList<String> keys = new ArrayList<>();
        keys.add("mca");
        keys.add("baidu");
        keys.add("spring");
        //拿到keys对应缓存的值
        Map<String, String> map = cache.getAll(keys);
        for (Map.Entry<String, String> entry : map.entrySet()) {
            //获取数据
            log.info("缓存的键:{}、缓存值：{}", entry.getKey(), entry.getValue());
        }
        log.info("LoadingCache 方法结束");

    }
```

运行效果：

<img src="_media/database/caffeinecache/过期数据同步加载运行结果-Cacheloader接口.png"/>

与之前的 `get();` 的同步加载操作不同的是，这里使用了专属的功能接口完成了数据的加载，从实现的结构上来说的更加的标准化，符合于 Caffeine 自己的设计要求。

第一种方式是针对于临时的一种使用方法，第二种更加的统一，同时有模板效应

### 5、过期数据的异步加载

假如你在拿去缓存数据的时候，如果有3个值都过期了，你使用的同步的方式得依次加载，这样阻塞等待的时间较长，所以这里可以使用异步的方式，就能同时进行加载。

```java
public static void AsyncLoadingCache() throws Exception {
        AsyncLoadingCache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(100)//设置缓存中保存的最大数量
                .expireAfterAccess(3L, TimeUnit.SECONDS)
                .buildAsync(new CacheLoader<String, String>() {
                    @Override
                    public String load(String key) throws Exception {
                        log.info("正在重新加载数据...");
                        TimeUnit.SECONDS.sleep(1);
                        return key.toUpperCase();
                    }
                });
        //使用了异步的缓存之后，缓存的值都是被CompletableFuture给包裹起来的
        //所以在追加缓存和得到缓存的时候要通过操作CompletableFuture来进行
        cache.put("mca", CompletableFuture.completedFuture("www.mashibing.com"));//设置缓存项
        cache.put("baidu", CompletableFuture.completedFuture("www.baidu.com"));//设置缓存项
        cache.put("spring", CompletableFuture.completedFuture("www.spring.io"));//设置缓存项

        TimeUnit.SECONDS.sleep(5);

        //创建key的列表，通过cache.getAll()拿到所有key对应的值
        ArrayList<String> keys = new ArrayList<>();
        keys.add("mca");
        keys.add("baidu");
        keys.add("spring");
        //拿到keys对应缓存的值
        Map<String, String> map = cache.getAll(keys).get();
        for (Map.Entry<String, String> entry : map.entrySet()) {
            log.info("缓存的键:{}、缓存值：{}", entry.getKey(), entry.getValue());//获取数据
        }
        log.info("AsyncLoadingCache 方法结束");
    }
```

显示效果：

<img src="_media/database/caffeinecache/过期数据异步加载运行结果.png"/>

AsyncLoadingCache的父接口是AsyncCache，而AsycnCache和Cache接口是同级的。

<img src="_media/database/caffeinecache/AsyncCache.png"/>

## 三、缓存淘汰机制

缓存之中的数据内容不可能一直被保留，因为只要时间一到，缓存就应该将数据进行驱逐，但是除了时间之外还需要考虑到个问题，缓存数据满了之后呢?是不是也应该进行一些无用数据的驱逐处理呢?

Caffeine提供三类驱逐策略：

1. 基于大小（size-based）
2. 基于时间（time-based）
3. 基于引用（reference-based）

### 1、基于大小的驱逐策略

最大容量 和 最大权重 只能二选一作为缓存空间的限制

#### 【1】最大容量

最大容量，如果缓存中的数据量超过这个数值，Caffeine 会有一个异步线程来专门负责清除缓存，按照指定的清除策略来清除掉多余的缓存。

```java
/**
 * 最大容量清理策略
 *
 * @throws Exception
 */
public static void ExpireMaxType() throws Exception {
    //Caffeine 会有一个异步线程来专门负责清除缓存
    Cache<String, String> cache = Caffeine.newBuilder()
            // 将最大数量设置为一
            .maximumSize(1)
            .expireAfterAccess(3L, TimeUnit.SECONDS)
            .build();
    cache.put("name", "张三");
    cache.put("age", "18");
    // 此时刚刚开启异步线程进行清理，还没有清理掉，所以立马获取是能够拿到值的
    System.out.println(cache.getIfPresent("name"));
    TimeUnit.MILLISECONDS.sleep(100);
    System.out.println(cache.getIfPresent("name"));
    System.out.println(cache.getIfPresent("age"));

}
```

<img src="_media/database/caffeinecache/最大容量运行结果.png"/>

可以看到，"name"的数据已经被清除了

#### 【2】最大权重

```java
/**
 * 最大权重清理策略
 */
public static void ExpireWeigherType() throws Exception {
    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumWeight(100) // 总共设置100权重容量
            .weigher(((key, value) -> {
                System.out.println("权重处理，key=" + key + " value=" + value);
                //这里直接返回一个固定的权重，真实开发会有一些业务的运算
                if (key.equals("age")) {
                    return 30;
                }
                return 50;
            }))
            .expireAfterAccess(3L, TimeUnit.SECONDS)
            .build();
    cache.put("name", "张三");
    cache.put("age", "18");
    cache.put("sex", "男");
    TimeUnit.MILLISECONDS.sleep(100);
    System.out.println(cache.getIfPresent("name"));
    System.out.println(cache.getIfPresent("age"));
    System.out.println(cache.getIfPresent("sex"));
}
```

<img src="_media/database/caffeinecache/最大权重运行结果.png"/>

运行结果：第一个数据被清除了，因为第三个进来权重大于100，导致被清理。

### 2、基于时间的驱逐策略

#### 【1】最后一次读

距离最后一次读的时间超过了n/s后驱逐

```java
/**
 * 时间驱逐策略 - 最后一次读
 *
 * @throws Exception
 */
public static void ExpireAfterAccess() throws Exception {
    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(100)
            .expireAfterAccess(1L, TimeUnit.SECONDS)
            .build();
    cache.put("name", "张三");
    for (int i = 0; i < 10; i++) {
        System.out.println("第" + i + "次读：" + cache.getIfPresent("name"));
        TimeUnit.SECONDS.sleep(2);
    }
}
```

#### 【2】最后一次写

距离最后一次写的时间超过了n/s后驱逐

```java
/**
 * 时间驱逐策略 - 最后一次写
 *
 * @throws Exception
 */
public static void ExpireAfterWrite() throws InterruptedException {
    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(100)
            .expireAfterWrite(1L, TimeUnit.SECONDS)
            .build();
    cache.put("name", "张三");
    for (int i = 0; i < 10; i++) {
        System.out.println("第" + i + "次读：" + cache.getIfPresent("name"));
        TimeUnit.SECONDS.sleep(1);
    }
}
```

### 3、自定义失效策略

可以根据业务需求，对不同的key定义不同的驱逐策略

```java
/**
 * 时间驱逐策略 - 自定义
 *
 * @throws InterruptedException
 */
public static void MyExpire() throws InterruptedException {
    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(100)
            .expireAfter(new MyExpire())
            .build();
    cache.put("name", "张三");
    for (int i = 0; i < 10; i++) {
        System.out.println("第" + i + "次读：" + cache.getIfPresent("name"));
        TimeUnit.SECONDS.sleep(1);
    }
}
```

```java
/**
 * 自定义驱逐策略，比较灵活
 */
public class MyExpire implements Expiry<String, String> {

    /**
     * 创建后(多久失效)
     * @param key
     * @param value
     * @param currentTime
     * @return
     */
    @Override
    public long expireAfterCreate(String key, String value, long currentTime) {
        //创建后
        System.out.println("创建后,失效计算 -- " + key + ": " + value);
        //将两秒转换为纳秒，并返回；代表创建后两秒失效
        return TimeUnit.NANOSECONDS.convert(2, TimeUnit.SECONDS);
    }

    /**
     * 更新后(多久失效)
     * @param key
     * @param value
     * @param currentTime
     * @param currentDuration
     * @return
     */
    @Override
    public long expireAfterUpdate(String key, String value, long currentTime, long currentDuration) {
        //更新后
        System.out.println("更新后,失效计算 -- " + key + ": " + value);
        return TimeUnit.NANOSECONDS.convert(5, TimeUnit.SECONDS);
    }

    /**
     * 读取后(多久失效)
     * @param key
     * @param value
     * @param currentTime
     * @param currentDuration
     * @return
     */
    @Override
    public long expireAfterRead(String key, String value, long currentTime, long currentDuration) {
        //读取后
        System.out.println("读取后,失效计算 -- " + key + ": " + value);
        return TimeUnit.NANOSECONDS.convert(100, TimeUnit.SECONDS);
    }
}
```

<img src="_media/database/caffeinecache/自定义失效策略运行结果.png"/>

### 4、基于引用驱逐策略

AysncLoadingCache 不支持弱引用和软引用。

#### 【1】软引用

当程序内存比较充足时，软引用不会被回收，当内存不足时，会被优先GC。

```java
/**
 * 时间驱逐策略 - 软引用:-Xms20m -Xmx20m
 * @throws InterruptedException
 */
public static void ExpireSoft() throws InterruptedException {
    Cache<String, Object> cache = Caffeine.newBuilder()
            .maximumSize(100)
            .softValues()
            .build();

    cache.put("name", new SoftReference<>("张三"));
    System.out.println("第1次读：" + cache.getIfPresent("name"));
    List<byte[]> list = new LinkedList<>();
    try {
        for (int i = 0; i < 100; i++) {
            list.add(new byte[1024 * 1024 * 1]); //1M的对象
        }
    } catch (Throwable e) {
        //抛出了OOM异常时
        TimeUnit.SECONDS.sleep(1);
        System.out.println("OOM时读：" + cache.getIfPresent("name"));
        System.out.println("Exception*************" + e.toString());
    }
}
```

<img src="_media/database/caffeinecache/软引用运行结果.png"/>

#### 【2】弱引用

程序出发GC就会被回收

```java
/**
 * 时间驱逐策略 - 弱引用
 * @throws InterruptedException
 */
public static void ExpireWeak() throws InterruptedException {
    Cache<String, Object> cache = Caffeine.newBuilder()
            .maximumSize(100)
            .weakValues()
            .build();
    cache.put("name",new WeakReference<>("张三"));

    System.out.println("第1次读："+cache.getIfPresent("name"));
    System.gc();//进行一次GC垃圾回收
    System.out.println("GC后读："+cache.getIfPresent("name"));
}
```

<img src="_media/database/caffeinecache/弱引用运行结果.png"/>

## 四、状态收集和异步监听

### 1、状态收集器

Caffeine 自带有数据的统计功能，在build之前，添加 `recordStats();`来开启数据统计功能。

例如：

1. 缓存查询次数；
2. 查询准确（指定数据的 KEY 存在并且可以返回最终的数据）的次数；
3. 查询失败的次数。默认情况下是没有开启此数据统计信息；

#### 【1】统计信息

```java
/**
 * 状态收集器 - 收集统计信息
 *
 * @throws Exception
 */
public static void CacheStats() throws Exception {
    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(2)
            .recordStats() //开启统计功能
            .expireAfterAccess(200L, TimeUnit.SECONDS)
            .build();
    cache.put("name", "张三");
    cache.put("sex", "男");
    cache.put("age", "18");
    //设置的key有些是不存在的,通过这些不存在的进行非命中操作
    String[] keys = new String[]{"name", "age", "sex", "phone", "school"};
    for (int i = 0; i < 1000; i++) {
        cache.getIfPresent(keys[new Random().nextInt(keys.length)]);
    }
    CacheStats stats = cache.stats();
    System.out.println("用户请求查询总次数：" + stats.requestCount());
    System.out.println("命中个数：" + stats.hitCount());
    System.out.println("命中率：" + stats.hitRate());
    System.out.println("未命中次数：" + stats.missCount());
    System.out.println("未命中率：" + stats.missRate());

    System.out.println("加载次数：" + stats.loadCount());
    System.out.println("总共加载时间：" + stats.totalLoadTime());
    System.out.println("平均加载时间（单位-纳秒）：" + stats.averageLoadPenalty());
    
    //加载失败率，= 总共加载失败次数 / 总共加载次数
    System.out.println("加载失败率：" + stats.loadFailureRate()); 
    System.out.println("加载失败次数：" + stats.loadFailureCount());
    System.out.println("加载成功次数：" + stats.loadSuccessCount());

    System.out.println("被淘汰出缓存的数据总个数：" + stats.evictionCount());
    System.out.println("被淘汰出缓存的那些数据的总权重：" + stats.evictionWeight());
}
```

#### 【2】自定义统计信息

```java
/**
 * 自定义状态收集器
 * @throws Exception
 */
public static void MyStatsCounter() throws Exception {
    Cache<String, String> cache = Caffeine.newBuilder()
            .maximumSize(2)
            .recordStats(MyStatsCounter::new) //开启统计功能，指向自定义收集器
            .expireAfterAccess(200L, TimeUnit.SECONDS)
            .build();
    cache.put("name", "张三");
    System.out.println(cache.getIfPresent("name"));
}
```

```java
/**
 * 自定义统计收集器
 */
public class MyStatsCounter implements StatsCounter {
    @Override
    public void recordHits(int count) {
        System.out.println("命中之后执行的操作");
    }

    @Override
    public void recordMisses(int count) {
    }

    @Override
    public void recordLoadSuccess(long loadTime) {
    }

    @Override
    public void recordLoadFailure(long loadTime) {
    }

    @Override
    public void recordEviction() {
    }

    @Override
    public CacheStats snapshot() {
        return null;
    }
}
```

### 2、清除、更新异步监听

```java
/**
     * 清除、更新一步监听
     *
     * @throws Exception
     */
    public static void cacheListener() throws Exception {
        Cache<String, String> cache = Caffeine.newBuilder()
                .maximumSize(2)
                .removalListener(((key, value, cause) ->
                        System.out.println(
                                " 键：" + key +
                                " 值：" + value +
                                " 清除原因：" + cause)))
                .expireAfterAccess(1, TimeUnit.SECONDS)
                .build();
        cache.put("name", "张三");
        cache.put("sex", "男");
        cache.put("age", "18");
        TimeUnit.SECONDS.sleep(2);
        cache.put("name2", "张三");
        cache.put("age2", "18");
        cache.invalidate("age2");
        TimeUnit.SECONDS.sleep(10);
    }
```

<img src="_media/database/caffeinecache/清除、更新异步监听运行结果.png"/>

## 五、Caffeine Cache + Redis 两级缓存架构

### 1、两级缓存架构设计

在有高性能需求的项目服务中，可以通过将热点数据存储到 Redis 等缓存中间件中来提高访问速度，同时降低数据库压力。

但中间件的使用必然会带来 IO 方面的问题，在某些需要极致性能的场景下，就需要本地缓存的支援，从而再次提升程序的响应速度与服务性能，所以出现了本地缓存 + 分布式缓存的两级缓存架构。

两级缓存架构中采用 Caffeine 作为一级缓存，Redis 作为二级缓存。

<img src="_media/database/caffeinecache/两级缓存架构流程.png"/>

### 2、两级缓存架构优缺点

#### 【1】优点

1. 一级缓存（Caffeine）基于应用的内存，访问速度非常快，对于一些变更频率低、实时性要求低的数据，可以放在本地缓存中，提升访问速度；
2. 使用一级缓存能够减少和 Redis 的二级缓存的远程数据交互，减少网络 I/O 开销，降低这一过程中在网络通信上的耗时。

#### 【2】缺点

1. 数据一致性问题：两级缓存与数据库的数据要保持一致，一旦数据发生了修改，在修改数据库的同时，一级缓存、二级缓存应该同步更新。
2. 分布式多应用情况下：一级缓存之间也会存在一致性问题，当一个节点下的本地缓存修改后，需要通知其他节点也刷新本地一级缓存中的数据，否则会出现读取到过期数据的情况。
3. 缓存的过期时间、过期策略以及多线程的问题。

### 3、两级缓存代码实现

#### 【1】准备表结构和数据

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `name` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '姓名',
  `age` int NOT NULL COMMENT '年龄',
  `email` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '邮箱',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 6 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;
```

```sql
INSERT INTO `user` VALUES (1, 'Jone', 18, 'test1@baomidou.com');
INSERT INTO `user` VALUES (2, 'Jack', 20, 'test2@baomidou.com');
INSERT INTO `user` VALUES (3, 'Tom', 28, 'test3@baomidou.com');
INSERT INTO `user` VALUES (4, 'Sandy', 21, 'test4@baomidou.com');
INSERT INTO `user` VALUES (5, 'Billie', 24, 'test5@baomidou.com');
```

#### 【2】创建项目

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.6.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>2.6.6</version>
        </dependency>
        <!--for caffeine cache-->
        <dependency>
            <groupId>com.github.ben-manes.caffeine</groupId>
            <artifactId>caffeine</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.1</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.29</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.14</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
    </dependencies>
```

#### 【3】配置信息

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.255.133:3306/caffeine_architecture?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource
  redis:
    host: 192.168.255.133
    port: 6379
    database: 0
    connect-timeout: 5000
```

#### 【4】添加User实体

```java
@ToString
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

#### 【5】创建Mapper接口

```java
/**
 * MyBatisPlus中的Mapper接口继承自BaseMapper
 */
public interface UserMapper extends BaseMapper<User> {
}
```

#### 【6】测试操作

```java
@SpringBootTest
class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    void queryUser() {
        List<User> users = userMapper.selectList(null);
        for (User user : users) {
            System.out.println(user);
        }
    }
}
```

#### 【7】日志输出

```yaml
mybatis-plus:
  configuration:
    # 日志
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

#### 【8】MyBatisPlusConfig

```java
import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan("com.msb.caffeine.mapper")
public class MyBatisPlusConfig {

    /**
     * 新的分页插件,一缓和二缓遵循mybatis的规则,
     * 需要设置 MybatisConfiguration#useDeprecatedExecutor = false 避免缓存出现问题(该属性会在旧插件移除后一同移除)
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }

}
```

### 4、手动两级缓存实现

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.TimeUnit;

@Configuration
public class CaffeineConfig {

    @Bean
    public Cache<String, String> caffeineCache() {

        // 创建一个Caffeine缓存实例
        return Caffeine.newBuilder()
                .initialCapacity(128) // 初始容量
                .maximumSize(1024) // 最大容量
                .expireAfterWrite(15, TimeUnit.SECONDS) // 写入后10分钟过期
                .build();
    }
}
```

```java
//只查数据库
    public User query1_1(long userId) {
        User user = userMapper.selectById(userId);
        log.info("query1_1 方法结束");
        return user;
    }

    //Caffeine+Redis两级缓存查询
    public User query1_2(long userId) {
        String key = "user-" + userId;
        User user = (User) cache.get(key,
                k -> {
                    //先查询 Redis  （2级缓存）
                    Object obj = redisTemplate.opsForValue().get(key);
                    if (Objects.nonNull(obj)) {
                        log.info("get data from redis:" + key);
                        return obj;
                    }
                    // Redis没有则查询 DB（MySQL）
                    User user2 = userMapper.selectById(userId);
                    log.info("get data from database:" + userId);
                    redisTemplate.opsForValue().set(key, 
                            user2, 30, TimeUnit.SECONDS);
                    return user2;
                });
        return user;
    }
```

在 Cache 的 `get();` 中，会先从 Caffeine 缓存中进行查找，如果找到缓存的值那么直接返回。

没有的话查找 Redis，Redis 再不命中则查询数据库，最后都同步到Caffeine的缓存中。

修改或删除时需要同时删除 Redis 和 Caffeine，否则会出现数据不一致问题。

```java
public void update1_1(User order) {
        log.info("update User data");
        String key = "user-" + order.getId();
        userMapper.updateById(order);
        //修改 Redis
        redisTemplate.opsForValue().set(key, order, 120, TimeUnit.SECONDS);
        // 修改本地缓存
        cache.put(key, order);
    }
 
public void delete1_1(long userId) {
        log.info("delete User");
        userMapper.deleteById(userId);
        String key = "user-" + userId;
        redisTemplate.delete(key);
        cache.invalidate(key);
    }
```

但是这种形式对代码的侵入性较强，所以需要其他形式的方式。

### 5、注解方式两级缓存实现

在 Spring中，提供了 CacheManager 接口和对应的注解

1. `@Cacheable`：根据键从缓存中取值，如果缓存存在，那么获取缓存成功之后，直接返回这个缓存的结果。如果缓存不存在，那么执行方法，并将结果放入缓存中。
2. `@CachePut`：不管之前的键对应的缓存是否存在，都执行方法，并将结果强制放入缓存。
3. `@CacheEvict`：执行完方法后，会移除掉缓存中的数据。

使用注解，就需要配置 Spring 中的 CacheManager  ，在这个CaffeineConfig 中

```java
//使用CaffeineCacheManager来管理缓存
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
                .initialCapacity(128)
                .maximumSize(1024)
                .expireAfterWrite(15, TimeUnit.SECONDS));
        return cacheManager;
    }
```

#### 【1】@EnableCaching

在启动类上再添加上 `@EnableCaching` 注解

```java
@SpringBootApplication
@MapperScan("com.fudepeng.caffeine.mapper")
@EnableCaching
public class CaffeineApp {
    public static void main(String[] args) {
        SpringApplication.run(CaffeineApp.class);
    }
}
```

#### 【2】@Cacheable

在 `UserService` 类对应的方法上添加 `@Cacheable` 注解，因为 `Controller` 层没有被动态代理

```java
    /**
     * Caffeine+Redis两级缓存查询-- 使用注解
     * `@Cacheable`:根据键从缓存中取值，如果缓存存在，那么获取缓存成功之后，直接返回这个缓存的结果。如果缓存不存在，那么执行方法，并将结果放入缓存中。
     * @param userId
     * @return
     */
    @Cacheable(value = "user", key = "#userId")
    public User query2_2(long userId) {
        String key = "user-" + userId;
        //先查询 Redis  （2级缓存）
        Object obj = redisTemplate.opsForValue().get(key);
        if (Objects.nonNull(obj)) {
            log.info("get data from redis:" + key);
            return (User) obj;
        }
        // Redis没有则查询 DB（MySQL）
        User user = userMapper.selectById(userId);
        log.info("get data from database:" + userId);
        redisTemplate.opsForValue().set(key, user, 30, TimeUnit.SECONDS);

        return user;
    }
```

#### 【3】@Cacheable 注解的属性

| 参数        | 解释                                                         | col3                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| key         | 缓存的key，可以为空，如果指定要按照SpEL表达式编写，如不指定，则按照方法所有参数组合 | `@Cacheable(value=”testcache”, ey=”#userName”)`              |
| value       | 缓存的名称，在 spring 配置文件中定义，必须指定至少一个       | 例如：`@Cacheable(value=”mycache”)`                          |
| condition   | 缓存的条件，可以为空，使用 SpEL 编写，返回 true 或者 false，只有为 true 才进行缓存 | `@Cacheable(  value=”testcache”,         condition=”#userName.length()>2” )` |
| methodName  | 当前方法名                                                   | `#root.methodName`                                           |
| method      | 当前方法                                                     | `#root.method.name`                                          |
| target      | 当前被调用的对象                                             | `#root.target`                                               |
| targetClass | 当前被调用的对象的class                                      | `#root.targetClass`                                          |
| args        | 当前方法参数组成的数组                                       | `#root.args[0]`                                              |
| caches      | 当前被调用的方法使用的Cache                                  | `#root.caches[0].name`                                       |

**condition属性指定发生的条件**，示例表示只有当userId为偶数时才会进行缓存

```java
/**
 * 只有当userId为偶数时才会进行缓存
 * @param userId
 * @return
 */
@Cacheable(value = "user", key = "#userId", condition = "#userId%2==0")
public User query2_3(long userId) {
    String key = "user-" + userId;
    //先查询 Redis  （2级缓存）
    Object obj = redisTemplate.opsForValue().get(key);
    if (Objects.nonNull(obj)) {
        log.info("get data from redis:" + key);
        return (User) obj;
    }
    // Redis没有则查询 DB（MySQL）
    User user = userMapper.selectById(userId);
    log.info("get data from database:" + userId);
    redisTemplate.opsForValue().set(key, user, 30, TimeUnit.SECONDS);

    return user;
}
```

#### 【4】@CacheEvict

`@CacheEvict` 是用来标注在需要清除缓存元素的方法或类上的，当标记在一个类上时表示其中所有的方法的执行都会触发缓存的清除操作。

`@CacheEvict` 可以指定的属性有 `value`、`key`、`condition`、`allEntries`和 `beforeInvocation`。

1. 其中 `value`、`key` 和 `condition` 的语义与 `@Cacheable` 对应的属性类似。即 `value` 表示清除操作是发生在哪些 Cache 上的（对应Cache的名称）；
2. `key` 表示需要清除的是哪个`key`，如未指定则会使用默认策略生成的`key`；
3. `condition`表示清除操作发生的条件。

```java
/**
 * 清除缓存(所有的元素)
 * @param userId
 */
@CacheEvict(value = "user", key = "#userId", allEntries = true)
public void deleteAll(long userId) {
    System.out.println(userId);
}

/**
 * beforeInvocation=true：在调用该方法之前清除缓存中的指定元素
 * @param userId
 */
@CacheEvict(value = "user", key = "#userId", beforeInvocation = true)
public void delete(long userId) {
    System.out.println(userId);
}
```

#### 【5】@CachePut

```java
/**
 * 不管之前的键对应的缓存是否存在，都执行方法，并将结果强制放入缓存。
 * @param userId
 */
@CachePut(value = "user", key = "#userId")
public void CachePut(long userId) {
    System.out.println(userId);
}
```

### 6、自定义注解实现两级缓存架构

#### 【1】自定义注解

用于添加在需要操作的缓存方法上

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DoubleCache {
    String cacheName();

    String key(); // 支持springEl表达式

    long l2TimeOut() default 120; // 缓存过期时间

    CacheType type() default CacheType.FULL; // 操作缓存的类型
}
```

#### 【2】CacheType

CacheType   是一个枚举类型的变量，表示操作缓存的类型

```java
public enum CacheType {
    FULL,   //存取
    PUT,    //只存
    DELETE  //删除
}
```

#### 【3】SpringEl表达式解析器

从前面我们知道，key要支持 springEl 表达式，写一个ElParser的方法，使用表达式解析器解析参数：

```java
import com.fudepeng.caffeine.bean.User;
import org.springframework.expression.EvaluationContext;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.common.TemplateParserContext;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;

import java.util.TreeMap;

/**
 * EL表达式解析器
 */
public class ElParser {
    public static String parse(String elString, TreeMap<String,Object> map){
        elString=String.format("#{%s}",elString);
        //创建表达式解析器
        ExpressionParser parser = new SpelExpressionParser();
        //通过evaluationContext.setVariable可以在上下文中设定变量。
        EvaluationContext context = new StandardEvaluationContext();
        map.entrySet().forEach(entry->
                context.setVariable(entry.getKey(),entry.getValue())
        );

        //解析表达式
        Expression expression = parser.parseExpression(elString, new TemplateParserContext());
        //使用Expression.getValue()获取表达式的值，这里传入了Evaluation上下文
        String value = expression.getValue(context, String.class);
        return value;
    }

    public static void main(String[] args) {
        String elString="#user.name";
        String elString2="#oder";
        String elString3="#p0";

        TreeMap<String,Object> map=new TreeMap<>();
        User user = new User();
        user.setId(111L);
        user.setName("lijin");
        map.put("user",user);
        map.put("oder","oder-8888");

        String val = parse(elString, map);
        String val2 = parse(elString2, map);
        String val3 = parse(elString3, map);
        System.out.println(val);
        System.out.println(val2);
        System.out.println(val3);

    }
}
```

#### 【4】面向切面的处理

```java
import com.github.benmanes.caffeine.cache.Cache;
import lombok.AllArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;
import java.util.Objects;
import java.util.TreeMap;
import java.util.concurrent.TimeUnit;

/**
 * 切面类
 */
@Slf4j
@Component
@Aspect // 切面的注解
@AllArgsConstructor
public class CacheAspect {
    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private Cache cache;


    /**
     * 切点注解要切入的地方
     */
    @Pointcut("@annotation(com.fudepeng.caffeine.cache.DoubleCache)")
    public void cacheAspect() {
    }

    /**
     * 环绕通知
     * @param point
     * @return
     * @throws Throwable
     */
    @Around("cacheAspect()")
    public Object doAround(ProceedingJoinPoint point) throws Throwable {
        //获取方法的签名及其他信息
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();

        //拼接解析springEl表达式的map
        String[] paramNames = signature.getParameterNames();
        Object[] args = point.getArgs();
        TreeMap<String, Object> treeMap = new TreeMap<>();
        for (int i = 0; i < paramNames.length; i++) {
            treeMap.put(paramNames[i], args[i]);
        }

        DoubleCache annotation = method.getAnnotation(DoubleCache.class);
        String elResult = ElParser.parse(annotation.key(), treeMap);
        String realKey = annotation.cacheName() + ":" + elResult;

        //强制更新
        if (annotation.type() == CacheType.PUT) {
            Object object = point.proceed();
            redisTemplate.opsForValue().set(realKey, object, annotation.l2TimeOut(), TimeUnit.SECONDS);
            cache.put(realKey, object);
            // 发送消息到Redis频道(让其他应用把Caffeine缓存失效掉)
            redisTemplate.convertAndSend("cacheUpdateChannel", realKey);
            log.info("cacheUpdateChannel:" + realKey);
            return object;
        }
        //删除
        else if (annotation.type() == CacheType.DELETE) {
            redisTemplate.delete(realKey);
            cache.invalidate(realKey);
            // 发送消息到Redis频道(让其他应用把Caffeine缓存失效掉)
            redisTemplate.convertAndSend("cacheUpdateChannel", realKey);
            log.info("cacheUpdateChannel:" + realKey);
            return point.proceed();
        }

        //读写，查询Caffeine
        Object caffeineCache = cache.getIfPresent(realKey);
        if (Objects.nonNull(caffeineCache)) {
            log.info("get data from caffeine");
            return caffeineCache;
        }

        //查询Redis
        Object redisCache = redisTemplate.opsForValue().get(realKey);
        if (Objects.nonNull(redisCache)) {
            log.info("get data from redis");
            cache.put(realKey, redisCache);
            return redisCache;
        }

        log.info("get data from database");
        Object object = point.proceed();
        if (Objects.nonNull(object)) {
            //写入Redis
            redisTemplate.opsForValue().set(realKey, object, annotation.l2TimeOut(), TimeUnit.SECONDS);
            //写入Caffeine
            cache.put(realKey, object);
            // 发送消息到Redis频道(让其他应用把Caffeine缓存失效掉)
            redisTemplate.convertAndSend("cacheUpdateChannel", realKey);
            log.info("cacheUpdateChannel:" + realKey);
        }
        return object;
    }
}
```

切面中主要做了下面几件工作：

* 通过方法的参数，解析注解中 key 的 springEl 表达式，组装真正缓存的 key。
* 根据操作缓存的类型，分别处理存取、只存、删除缓存操作。
* 删除和强制更新缓存的操作，都需要执行原方法，并进行相应的缓存删除或更新操作。
* 存取操作前，先检查缓存中是否有数据，如果有则直接返回，没有则执行原方法，并将结果存入缓存。

然后使用的话就非常方便了，代码中只保留原有业务代码，再添加上我们自定义的注解就可以了：

```java
	/**
     * 存入
     * @param userId
     * @return
     */
    @DoubleCache(cacheName = "user", key = "#userId", type = CacheType.PUT)
    public User query3(Long userId) {
        User user = userMapper.selectById(userId);
        return user;
    }

    /**
     * 覆盖
     * @param user
     * @return
     */
    @DoubleCache(cacheName = "user", key = "#user.userId", type = CacheType.PUT)
    public int update3(User user) {
        return userMapper.updateById(user);
    }

    /**
     * 删除
     * @param user
     * @return
     */
    @DoubleCache(cacheName = "user", key = "#user.userId", type = CacheType.DELETE)
    public void deleteOrder(User user) {
        userMapper.deleteById(user);
    }
```

### 7、缓存一致性问题

就是如果一个应用修改了缓存，另外一个应用的 caffeine 缓存是没有办法感知的，所以这里就会有缓存的一致性问题

<img src="_media/database/caffeinecache/缓存一致性问题.png"/>

解决方案：在 Redis 中做一个发布和订阅。

遇到修改缓存的处理，需要向对应的频道发布一条消息，然后应用同步监听这条消息，有消息则需要删除本地的Caffeine缓存。

```java
// 发布的具体实现在定义切面类中 >> CacheAspect.Class
// 发送消息到Redis频道(让其他应用把Caffeine缓存失效掉)
redisTemplate.convertAndSend("cacheUpdateChannel", realKey);
```

RedisConfig

```java
import com.msb.caffeine.service.RedisMessageListener;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.listener.ChannelTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(); // 需要设置主机名，端口，密码等参数
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        // 使用Jackson2JsonRedisSerializer来序列化和反序列化对象
        GenericJackson2JsonRedisSerializer jackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        // 设置键序列化器
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());

        // 设置值序列化器
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        template.afterPropertiesSet();
        return template;
    }

    @Bean
    RedisMessageListenerContainer container(RedisConnectionFactory factory, RedisMessageListener listener) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(factory);
        container.addMessageListener(listener, new ChannelTopic("cacheUpdateChannel"));
        return container;
    }
}
```

监听消息

```java
import com.github.benmanes.caffeine.cache.Cache;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.Message;
import org.springframework.data.redis.connection.MessageListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class RedisMessageListener implements MessageListener {

    @Autowired
    private Cache cache;

    //这里就是应用接收到了（要删除缓存的策略）： 这里就强制删除Caffeine中的缓存数据
    @Override
    public void onMessage(Message message, byte[] pattern) {
        String cacheKey = new String(message.getBody());
        if (!cacheKey.equals("")) {
            log.info("invalidate:" + cacheKey);
            cache.invalidate(cacheKey);
        }
    }
}

```

## 六、Caffeine Cache 原理与源码

### 1、Caffeine Cache的宏观结构

缓存的内部结构与 HashMap 类似，是`key-value`结构

因为Caffeine Cache是多线程并发的，为了保证线程安全，其内部是 ConcurrentHashMap 来实现的，缓存的所有数据几乎都在ConcurrentHashMap中

> final ConcurrentHashMap<Object, Node<K, V>> data;

其次因为需要定时清理的功能，所以其内部具备一个定时器（Schedule），用来定期清理数据，属于内部机制

再者在很多情况下需要异步处理，所以内部实现了异步线程池，通过Executor来实现异步线程机制，默认使用的是 `ForkJoinPool.commonPoll();`

### 2、缓存淘汰算法

#### 【1】概述

SpringBoot 2.0 开始默认使用了 Caffeine 作为默认本地缓存，之前使用的是 Guava，其中很大的原因就是因为 Caffeine 具备的缓存淘汰算法最优

其缓存淘汰算法分为三类：

1. FIFO：先进先出（用的最少）
2. LRU：最近最少使用
3. LFU：最不经常使用淘汰算法（使用的频次）

LRU算法：

1. 如果一个数据最少最近被访问到，那么被认为未来被访问的概率也是最低的，优先淘汰。
2. 它在内部维护一个链表，会将最近被访问的放到表头，最少被访问的在表尾，当触发淘汰时，会将表尾的数据淘汰掉。
3. 缺点：但可能会出现缓存污染问题，就是被访问的数据被加入到链表中后，有一部分数据再之后将不会被访问，对偶发性和周期性的数据没有特别好的处理机制
4. 优点：针对突发性的流量效果较好，适合做局部的流量爆发，比如秒杀场景。

LFU算法：

1. 如果一个数据在一定时间内被访问的次数很低，被认为未来被访问的概率也是最低的。
2. 其内部维护一个容器或队列，通过给数据做访问计数，当容器满了的时候淘汰计数最小的。
3. 优点：适合局部周期性的流量，从缓存命中率上来说要比LRU高。
4. 缺点：需要额外的空间来存储访问次数，且每次访问的时候都要更新这个访问次数，增加开销，对局部突发性流量是不适应的。

Caffeine 中使用的是 TinyLFU，针对LFU进行的优化。

#### 【2】Caffeine - TinyLFU

TinyLFU 的思想和JVM中分代垃圾回收类似

<img src="_media/database/caffeinecache/TinyLFU.png"/>

TinyLFU 整体流程：

1. 所有新写入的数据，都写入 WindowCache；
2. 当 WindowCache 满了，开始淘汰（LRU）进入考察区；
3. 如果考察区未满，直接放入，如果满了就要在考察区中进行PK规则，失败直接被淘汰；
4. 如果考察区中的缓存项的访问频次达到了阈值，这个数据就可以进入保护区；
5. 如果保护区满了，就根据tiny-LFU 进行淘汰。

TinyLFU PK 机制：WindowCache 进来的缓存项（访问频次W） VS 最老的缓存项（访问频次O）

1. `W > O 淘汰 O`
2. `W <= O && W < 5` 淘汰W
3. `W <= O &&  W > 5` 随机淘汰一个
4. 在考察区中访问频次到达15后，进入保护区

### 3、源码分析

#### 【1】数据结构

`build();` >> `Caffe.java` >> `BoundedLocalCache.java`

```java
// Caffe.java
public <K1 extends K, V1 extends V> @NonNull Cache<K1, V1> build() {
        this.requireWeightWithWeigher(); // 权重 
        this.requireNonLoadingCache(); // 加载缓存
        return (Cache)(this.isBounded() ? 
                       new BoundedLocalCache.BoundedLocalManualCache(this) : 
                       new UnboundedLocalCache.UnboundedLocalManualCache(this));
}
```

```java
// BoundedLocalCache.java 构造
BoundedLocalManualCache(Caffeine<K, V> builder, @Nullable CacheLoader<? super K, V> loader) {
            this.cache = LocalCacheFactory.newBoundedLocalCache(builder, loader, false);
            this.isWeighted = builder.isWeighted();
        }
```

```java
// BoundedLocalCache.java 数据结构
final ConcurrentHashMap<Object, Node<K, V>> data;
// >>  Node.java  > Node<K, V>
```

#### 【2】W-TinyLFU

淘汰方法的入口，针对 Window和 Main 的淘汰

```java
// BoundedLocalCache.java 淘汰的主方法（入口）
@GuardedBy("evictionLock")
    void evictEntries() {
        if (this.evicts()) {
            // 针对Windows淘汰
            int candidates = this.evictFromWindow();
            // 针对Main淘汰
            this.evictFromMain(candidates);
        }
    }
```

Window淘汰：

```java
@GuardedBy("evictionLock")
    int evictFromWindow() {
        int candidates = 0;

        Node next;
        for(Node<K, V> node = (Node)this.accessOrderWindowDeque().peek(); // 按照排序的方式得到头部节点
            // 判断Window的大小是否超过了最大限制，同时需要满足node节点不为null
            this.windowWeightedSize() > this.windowMaximum() && node != null; 
            // 找到下一个节点
            node = next) {
            
            next = node.getNextInAccessOrder();
            if (node.getWeight() != 0) { // 判断node的权重不为0
                node.makeMainProbation(); // 进入考察区
                this.accessOrderWindowDeque().remove(node); // 从WindowDeque中删除
                this.accessOrderProbationDeque().add(node); // 添加到考察区的Deque
                ++candidates;
                // 设置权重
                this.setWindowWeightedSize(this.windowWeightedSize() - (long)node.getPolicyWeight());
            }
        }

        return candidates;
    }
```

Main 淘汰：

```java
 @GuardedBy("evictionLock")
    void evictFromMain(int candidates) {
        int victimQueue = 1;
        // 拿到考察区最老的缓存项
        Node<K, V> victim = (Node)this.accessOrderProbationDeque().peekFirst();
        // 从Window进来的最新的缓存项
        Node<K, V> candidate = (Node)this.accessOrderProbationDeque().peekLast();

        while(this.weightedSize() > this.maximum()) {
            if (candidates == 0) {
                candidate = null;
            }

            if (candidate == null && victim == null) {
                if (victimQueue == 1) {
                    victim = (Node)this.accessOrderProtectedDeque().peekFirst();
                    victimQueue = 2;
                } else {
                    if (victimQueue != 2) {
                        break;
                    }

                    victim = (Node)this.accessOrderWindowDeque().peekFirst();
                    victimQueue = 0;
                }
            } else if (victim != null && victim.getPolicyWeight() == 0) {
                victim = victim.getNextInAccessOrder();
            } else if (candidate != null && candidate.getPolicyWeight() == 0) {
                candidate = candidate.getPreviousInAccessOrder();
                --candidates;
            } else {
                Node evict;
                if (victim == null) {
                    evict = candidate.getPreviousInAccessOrder();
                    Node<K, V> evict = candidate;
                    candidate = evict;
                    --candidates;
                    this.evictEntry(evict, RemovalCause.SIZE, 0L);
                } else if (candidate == null) {
                    evict = victim;
                    victim = victim.getNextInAccessOrder();
                    this.evictEntry(evict, RemovalCause.SIZE, 0L);
                } else {
                    K victimKey = victim.getKey();
                    K candidateKey = candidate.getKey();
                    Node evict;
                    if (victimKey == null) {
                        evict = victim;
                        victim = victim.getNextInAccessOrder();
                        this.evictEntry(evict, RemovalCause.COLLECTED, 0L);
                    } else if (candidateKey == null) {
                        --candidates;
                        evict = candidate;
                        candidate = candidate.getPreviousInAccessOrder();
                        this.evictEntry(evict, RemovalCause.COLLECTED, 0L);
                    } else if ((long)candidate.getPolicyWeight() > this.maximum()) {
                        --candidates;
                        evict = candidate;
                        candidate = candidate.getPreviousInAccessOrder();
                        this.evictEntry(evict, RemovalCause.SIZE, 0L);
                    } else {
                        --candidates;
                        // Tiny-LFU算法核心
                        if (this.admit(candidateKey, victimKey)) {
                            evict = victim;
                            victim = victim.getNextInAccessOrder();
                            this.evictEntry(evict, RemovalCause.SIZE, 0L);
                            candidate = candidate.getPreviousInAccessOrder();
                        } else {
                            evict = candidate;
                            candidate = candidate.getPreviousInAccessOrder();
                            this.evictEntry(evict, RemovalCause.SIZE, 0L);
                        }
                    }
                }
            }
        }

    }
```

```java
@GuardedBy("evictionLock")
    boolean admit(K candidateKey, K victimKey) {
        // 得到最老缓存项和最新缓存项的访问频次
        int victimFreq = this.frequencySketch().frequency(victimKey);
        int candidateFreq = this.frequencySketch().frequency(candidateKey);

        if (candidateFreq > victimFreq) {
            return true;
        } else if (candidateFreq <= 5) { // 最新缓存项访问频次小于5
            return false;
        } else {
            int random = ThreadLocalRandom.current().nextInt();
            return (random & 127) == 0;
        }
    }
```

#### 【3】Caffeine 空间优化

DATA-LFU 算法需要记录元素的访问频次，一般是使用long类型存储，这样就会有很多的空间浪费，所以它参考了布隆过滤器的方式。

准备一个bitmap二进制数组来存储频次，初始值都为0，当有一个元素进来后通过哈希等方式计算它的位置，将这个位置改为1。

但bitmap二进制只有0和1，不过 在Caffeine中频次最高只有16次，所以可以占据4个bit位来记录。

至于hash冲突的问题，可以使用4个hash函数，相当于存储频次就需要存放4个值，其中取最小的即可。

long类型占64位，实际可以存储16个4位，每个key占用4个位置，代表一个long可以存储4个key的频次，所以实际上long的数组大小是实际容量的4倍。

源码：`FrequencySketch.java`

```java
long[] table;

/**
 * 添加方法
 */ 
public void increment(@NonNull E e) {
        if (!this.isNotInitialized()) {
            // 得到hash值
            int hash = this.spread(e.hashCode());
            int start = (hash & 3) << 2;
            
            // 4个hash函数
            int index0 = this.indexOf(hash, 0);
            int index1 = this.indexOf(hash, 1);
            int index2 = this.indexOf(hash, 2);
            int index3 = this.indexOf(hash, 3);
            
            boolean added = this.incrementAt(index0, start);
            added |= this.incrementAt(index1, start + 1);
            added |= this.incrementAt(index2, start + 2);
            added |= this.incrementAt(index3, start + 3);
            if (added && ++this.size == this.sampleSize) {
                this.reset();
            }

        }
    }

boolean incrementAt(int i, int j) {
        int offset = j << 2;
        long mask = 15L << offset;
        if ((this.table[i] & mask) != mask) {
            long[] var10000 = this.table;
            var10000[i] += 1L << offset;
            return true;
        } else {
            return false;
        }
    }
```

```java
/**
 * 获取访问频次
 * 其中设计 count-Min sketch 算法
 */ 
public int frequency(@NonNull E e) {
        if (this.isNotInitialized()) {
            return 0;
        } else {
            int hash = this.spread(e.hashCode());
            int start = (hash & 3) << 2;
            int frequency = Integer.MAX_VALUE;

            // 循环四次，取最小值
            for(int i = 0; i < 4; ++i) {
                int index = this.indexOf(hash, i);
                int count = (int)(this.table[index] >>> (start + i << 2) & 15L);
                frequency = Math.min(frequency, count);
            }

            return frequency;
        }
    }
```

