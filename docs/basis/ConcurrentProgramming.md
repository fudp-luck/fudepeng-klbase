## 一、线程的基本概念

### 1、基本概念

#### 【1】进程、线程、纤程

进程：硬盘上有一个程序 QQ.exe，这个程序是一个静态的概念，当点开之后运行登录进去了，这时叫一个进程。进程相对于程序来说是一个动态的概念

线程：是进程里面最小的执行单元，是一个程序里不同的执行路径

纤程：用户态不经过内核态的线程

#### 【2】示例

程序的运行结果：T1 和 main 交替输出

这就是程序中有两条不同的执行路径在交叉进行，这就是直观概念上的线程

```java
public class T01_WhatIsThread {
    private static class T1 extends Thread {
        @Override
        public void run() {
           for(int i=0; i<10; i++) {
               try {
                   TimeUnit.MICROSECONDS.sleep(1);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               System.out.println("T1");
           }
        }
    }

    public static void main(String[] args) {
        //new T1().run();  // 方法调用，执行结果为 T1 T1 T1 main main main
        new T1().start(); // 结果为交替输出 T1 main T1 main T1 main
        for(int i=0; i<10; i++) {
            try {
                TimeUnit.MICROSECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("main");
        }
    }
}
```

### 2、创建线程的方式

#### 【1】创建线程的方式

1. `new Thread().start();`
2. `new Thread(Runnable).start();`
3. 可以说是 `Executors.newCachedThreadPool()`，线程池也是用前面两种之一；
4. 也可以说是`FutureTask + Callable`

#### 【2】代码示例

```java
public class T02_HowToCreateThread {
    static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("Hello MyThread!");
        }
    }

    static class MyRun implements Runnable {
        @Override
        public void run() {
            System.out.println("Hello MyRun!");
        }
    }

    static class MyCall implements Callable<String>{

        @Override
        public String call() throws Exception {
            System.out.println("Hello MyCall");
            return "success";
        }
    }
    // 启动线程的5种方式
    public static void main(String[] args) {
        new MyThread().start();
        new Thread(new MyRun()).start();
        new Thread(()->{
            System.out.println("Hello Lambda!");
        }).start();
        Thread t = new Thread(new FutureTask<String>(new MyCall()));
        t.start();

        ExecutorService service = Executors.newCachedThreadPool();
        service.execute(()->{
            System.out.println("Hello TreadPool");
        });
        service.shutdown();
    }
}
```

### 3、线程的方法

#### 【1】方法

1. `sleep();`：睡眠，当前线程暂停一段时间，让给别的线程去执行； sleep 的复活是由睡眠时间而定，等睡眠到规定的时间自动复活
2. `yield();`：在当前线程正在执行的时候停止下来进入等待队列，回到等待队列中系统调度算法里还是依然有可能将这个刚回去的线程拿回来继续执行，更大的可能性是把原来等待的那些线程拿出来一个执行， 所以yield 的意思是当前执行的线程让出一下 CPU，后面的能不能抢到不管
3. `join();`：在当前线程加入调用 join 的线程，本线程等待，等调用的线程执行完毕，再回去执行， 比如：t1 和 t2 两个线程，在 t1 的某个点上调用了 t2.join() ，它会跑到 t2 去运行，t1 等待 t2 运行完毕后再运行，自己join 自己，没有意义

#### 【2】代码示例

```java
public class T03_Sleep_Yield_Join {
    public static void main(String[] args) {
//        testSleep();
//        testYield();
        testJoin();
    }
    /* 
    Sleep，意思就是睡眠，当前线程暂停一段时间，让给别的线程去执行。
    Sleep是怎么复活的？ 由你的睡眠时间而定，等睡眠到规定的时间自动复活
     */
    static void testSleep() {
        new Thread(()->{
            for(int i=0; i<100; i++) {
                System.out.println("A" + i);
                try {
                    Thread.sleep(500);
                    //TimeUnit.Milliseconds.sleep(500)
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
    /*
    Yield，就是当前线程正在执行的时候停止下来进入等待队列，
    回到等待队列里在系统调度算法里还是依然有可能将这个刚回去的线程拿回来继续执行，
    当然，更大的可能性是把原来等待的那些线程拿出来一个执行，
    所以yield的意思是当前执行的线程让出以下CPU，后面的能不能抢到不管
     */
    static void testYield() {
        new Thread(()->{
            for(int i=0; i<100; i++) {
                System.out.println("A" + i);
                if(i%10 == 0) Thread.yield();


            }
        }).start();

        new Thread(()->{
            for(int i=0; i<100; i++) {
                System.out.println("------------B" + i);
                if(i%10 == 0) Thread.yield();
            }
        }).start();
    }
    /*
    join，意思就是在自己当前线程加入调用Join的线程，本线程等待。等调用的线程执行完毕，再回去执行。
    比如，t1和t2两个线程，在t1的某个点上调用了t2.join()，它会跑到t2去运行，t1等待t2运行完毕后再运行
    自己join自己，没有意义
     */
    static void testJoin() {
        Thread t1 = new Thread(()->{
            for(int i=0; i<100; i++) {
                System.out.println("A" + i);
                try {
                    Thread.sleep(500);
                    //TimeUnit.Milliseconds.sleep(500)
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread t2 = new Thread(()->{

            try {
                t1.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            for(int i=0; i<100; i++) {
                System.out.println("A" + i);
                try {
                    Thread.sleep(500);
                    //TimeUnit.Milliseconds.sleep(500)
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

### 4、线程的状态

#### 【1】Java中线程状态的迁移图

<img src="_media/basis/多线程与高并发/java中线程状态的迁移图.png"/>

#### 【2】常见的线程状态有六种

当我们 new 一个线程，还没有调用 start() 方法时，该线程处于新建状态

线程对象调用 start() 方法时，会被线程调度器来执行（交给操作系统来执行），此时的状态叫 Runnable
1. Runnable 内部有两个状态：Ready 就绪状态；Running 运行状态。
2. 就绪状态：就是放到 CPU 的等待队列里，排队等待 CPU 运行
3. Running 运行状态：等真正到 CPU 上去运行的时候才叫（调用 yiled() 时会从 Running 状态切换到 Ready 状态，线程配调度器选中执行的时候又从 Ready 状态切换到 Running 状态）

如果当前线程顺利的执行结束，就会切换到 Teminated 结束状态，（需要注意 Teminated 之后不可以回到 new 状态再调用 start()）

在 Runnable 这个状态里头还有其他一些状态的变迁
1. TimedWaiting 等待
2.  Waiting 等特
3. Blocked 阻塞：在同步代码块中没得到锁就会阻塞状态，获得锁的时候是就绪状态运行。

在运行的时候如果调用了 `o.wait()、t.join()、LockSupport.park()` 就会进入Waiting 状态

调用 `o.notify()、o.notifyAll()、LockSupport.unpark()` 就又回到 Running 状态

TimedWaiting 按照时间等待，等时间结束后就会切换回原先的状态

`Thread.sleep(time)、o.wait(time)、t.jion(time)、LockSupport,parkNanos()、LockSupport,parkUntil()` 这些都是关于时间等待的方法。

#### 【3】线程状态相关问题

问题1：哪些状态是 JVM 管理的？哪些是操作系统管理的？

  - 上面这些状态全是由JVM管理的
  - 因为 JVM 管理的时候也要通过操作系统，所以哪个是操作系统和哪个是 JVM 是无法区分的，JVM 是跑在操作系统上的一个普通程序。

问题2：线程什么状态时候会被挂起？挂起是否也是一个状态？

  - Running 的时候
  - 在一个 CPU 上会跑很多个线程，CPU 会隔一段时间执行一次这个线程，在隔一段时间执行一次另一个线程，这个是CPU内部的调度，线程在 Running 状态时，切换出该状态就叫线程挂起，由CPU控制

问题3：如何关闭线程？

  - 关闭线程的代码是 stop
  - 不要去关闭线程，stop 易造成线程状态不一致，要让线程正常结束，stop 方法已被废除

问题4：interrupt问题

  - 表示被打断，需要 catch 异常，做异常处理
  - 只有在一些框架中为了保证程序的健壮性而使用，业务逻辑中一般不会使用，了解即可

#### 【4】代码

```java
public class T04_ThreadState {
    static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println(this.getState());

            for(int i=0; i<10; i++) {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(i);
            }
        }
    }

    public static void main(String[] args) {
        Thread t = new MyThread();
        // 怎样得到这个线程的状态？ 通过t.getState()这个方法
        System.out.println(t.getState());
        t.start(); // 到这start完成后使Runnable状态
        try {
            t.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //然后join之后，结束了，是Terminated状态
        System.out.println(t.getState());
    }
}
```

### 5、synchronized关键字

#### 【1】为什么要上锁

访问某一段代码或者某临界资源的时候需要有锁

比如：
- 两个程序同时对一个数字做递增，如果两个线程共同访问的时候，第一个线程读到的是 0，然后加 1，在自己线程内部内存里面计算，在还没有写回去的时候，第二个线程读到了它还是 0，加 1 在写回去，本来加了两次，但还是 1
- 那么在对这个数字递增的过程当中加锁，也就是说第一个线程对这个数字访问的时候是独占的，不允许别的线程来访问，不允许别的线程来对它计算，必须加 1 后释放锁，其他线程才能对它继续加

实质上，这把锁并不是对数字进行锁定的，可以任意指定，想锁谁就锁谁

#### 【2】代码示例

如果需求是，上了把锁之后才能对 count-- 访问，可以 new 一个 Object，所以这里锁定就是 o，当拿到这把锁的时候才能执行这段代码，是锁定的某一个对象

```java
public class T {
    private int count = 10;
    private Object o = new Object();

    public void m() {
        synchronized(o) { //任何线程要执行下面的代码，必须先拿到o的锁
            count--;
            System.out.println(Thread.currentThread().getName() + " count = " + count);
        }
    }
}
```

如果每定义一个锁的对象 Object o，把它 new 出来，那加锁的时候每次都要 new 一个新的对象出来，所以有一个简单的方式 —— `synchronized(this)`

```java
class T2{
    private int count = 10;

    public void m() {
        synchronized(this) { //任何线程要执行下面的代码，必须先拿到this的锁
            count--;
            System.out.println(Thread.currentThread().getName() + " count = " + count);
        }
    }
}
```

如果要是锁定当前对象，也可以写如下方法，synchronized 方法和 synchronized(this) 在执行这段代码时是等值的

```java
class T3{
    private int count = 10;

    public synchronized void m() { //等同于在方法的代码执行时要synchronized(this)
        count--;
        System.out.println(Thread.currentThread().getName() + " count = " + count);
    }
}
```

静态方法 static 是没有 this 对象的，不需要 new 对象就能执行这个方法，但如果方法加上了synchronized 的话就代表 synchronized(T.class) ；

synchronized(T.class) 锁定就是 T 类的对象（当 T.class 加载到内存时会创建两个内容，一个是将 class 文件放到内存，另一个是创建了 .class 对象指向 class 文件）

```java
class T4{
    private static int count = 10;

    public synchronized static void m() { //这里等同于synchronized(T4.class)
        count--;
        System.out.println(Thread.currentThread().getName() + " count = " + count);
    }

    public static void mm() {
        synchronized(T4.class) { //考虑一下这里写synchronized(this)是否可以？
            count --;
        }
    }
}
```

T.class 是单例的么：一个 class 加载到内存一般情况是单例的，如果是同一个 Classloader 空间那它一定是单例的。如果不是同一个类加载器，不同类加载器互相之间也不能访问，那就是单例的

#### 【3】程序分析一

一下程序的问题：可能读不到其它线程修改过的内容，除此之外 count-- 之后下面的 count 输出和减完的结果不对，因为如果有一个线程把它从 10 减到 9 了，之后又有一个线程在上一个线程还没输出的时候进来把 9 减到8，继续输出的 8 而不是 9

解决办法1：在上面加上 volatile，保证线程间的可见性

解决办法2：在方法上加 synchronized，加了 synchronized 后就没必要加 volatile 了，因为 synchronized 既能保证原子性，又能保证可见性

```java
class T5 implements Runnable{
    private /*volatile*/ int count = 100;

    public /*synchronized*/ void run() {
        count--;
        System.out.println(Thread.currentThread().getName() + " count = " + count);
    }

    public static void main(String[] args) {
        T5 t = new T5();
        for(int i=0; i<100; i++) {
            new Thread(t, "THREAD" + i).start();
        }
    }
}
```

#### 【4】程序分析二

同步方法和非同步方法是否可以同时调用：当有一个 synchronized 的 m1 方法，调用 m1 的时候能不能调用m2，是可以的

```java
class T6{
    public synchronized void m1() {
        System.out.println(Thread.currentThread().getName() + " m1 start...");
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " m1 end");
    }

    public void m2() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " m2 ");
    }

    public static void main(String[] args) {
        T6 t = new T6();

		/*new Thread(()->t.m1(), "t1").start();
		new Thread(()->t.m2(), "t2").start();*/

        new Thread(t::m1, "t1").start();
        new Thread(t::m2, "t2").start();

		/*
		//1.8之前的写法
		new Thread(new Runnable() {

			@Override
			public void run() {
				t.m1();
			}
		});
		*/
    }
}
```

#### 【5】分析程序三-模拟银行账户

首先定义一个账户，有名称、余额

写方法用来给哪个用户设置它多少余额，读方法通过这个名字得到余额

如果给写方法加锁，给读方法不加锁，容易产生脏读问题，

业务中如果中间读到了一些不太好的数据也没关系，如果不允许客户读到中间不好的数据那这个就有问题。

比如说：给张三设置 100 块钱，睡了 1 毫秒之后去读它的值，然后再睡 2 秒再去读它的值，此时会看到读到的值有问题
- 原因是在设定的过程中 this.name 在中间睡了一下，这个过程当中模拟了一个线程来读，这个时候调用的是 getBalance() 方法，而调用这个方法的时候是不用加锁的
- 所以其他线程不需要等整个过程执行结束，就可以读到中间结果产生的内存，这个现象就叫做脏读
- 这问题的产生就是 synchronized 方法和非 synchronized 方法是同时运行的。
- 解决就是把 getBalance() 加上 synchronized 就可以了，如果你的业务允许脏读，就可以不用加锁，加锁之后的效率低下。

```java
/**
 * 面试题：模拟银行账户
 * 对业务写方法加锁
 * 对业务读方法不加锁
 * 这样行不行？
 *
 * 容易产生脏读问题（dirtyRead）
 */
public class Account {
    String name;
    double balance;

    public synchronized void set(String name, double balance) {
        this.name = name;
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.balance = balance;
    }

    public /*synchronized*/ double getBalance(String name) {
        return this.balance;
    }

    public static void main(String[] args) {
        Account a = new Account();
        new Thread(()->a.set("zhangsan", 100.0)).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(a.getBalance("zhangsan"));

        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(a.getBalance("zhangsan"));
    }
}
```

### 6、synchronized属性：可重入

#### 【1】m1() -> m2()

如果一个同步方法调用另外一个同步方法，其中一个方法加了锁，另外一个方法也需要加锁，加的是同一把锁也是同一个线程，那这个时候申请仍然会得到该对象的锁。

比如：m1() 和 m2() 方法是 synchronized， 此时 m1 开始的时候该线程得到了这把锁，然后在 m1 中调用m2
- 如果此时不允许任何线程再来拿这把锁，就会死锁
- 但此时调 m2 它发现是同一个线程，因为 m2 也需要申请这把锁，它发现是同一个线程申请的这把锁，就可以得到，这就叫可重入锁

```java
class T7{
    synchronized void m1() {
        System.out.println("m1 start");
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        m2();
        System.out.println("m1 end");
    }

    synchronized void m2() {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("m2");
    }

    public static void main(String[] args) {
        new T7().m1();
    }
```

#### 【2】模拟父类子类

父类 synchronized，子类调用 `super.m();` 的时候必须要可重入，否则就会出问题（调用父类是同一把锁）

所谓的重入锁就是你拿到这把锁之后不停的加锁，加好几层，但锁定的还是同一个对象，去掉一层就减1.

```java
public class FatherT {
    synchronized void m() {
        System.out.println("m start");
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("m end");
    }

    public static void main(String[] args) {
        new TT().m();
    }
}

class TT extends FatherT {
    @Override
    synchronized void m() {
        System.out.println("child m start");
        super.m();
        System.out.println("child m end");
    }
}
```

### 7、synchronized异常锁

m() 加锁，while(true) 不断执行，此时线程启动，count++ 如果等于 5 的时候认为的产生异常。

这时候如果产生任何异常，就会被准备抢锁的线程乱冲进来，程序乱入，这是异常的概念。

```java
/**
 * 程序在执行过程中，如果出现异常，默认情况锁会被释放
 * 所以，在并发处理的过程中，有异常要多加小心，不然可能会发生不一致的情况。
 * 比如，在一个web app处理过程中，多个servlet线程共同访问同一个资源，这时如果异常处理不合适，
 * 在第一个线程中抛出异常，其他线程就会进入同步代码区，有可能会访问到异常产生时的数据。
 * 因此要非常小心的处理同步业务逻辑中的异常
 * @author mashibing
 */
class T8{
    int count = 0;
    synchronized void m() {
        System.out.println(Thread.currentThread().getName() + " start");
        while(true) {
            count ++;
            System.out.println(Thread.currentThread().getName() + " count = " + count);
            try {
                TimeUnit.SECONDS.sleep(1);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            if(count == 5) {
                //此处抛出异常，锁将被释放，要想不被释放，可以在这里进行catch，然后让循环继续
                int i = 1/0; 
                System.out.println(i);
            }
        }
    }

    public static void main(String[] args) {
        T8 t = new T8();
        Runnable r = new Runnable() {

            @Override
            public void run() {
                t.m();
            }
        };
        new Thread(r, "t1").start();

        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(r, "t2").start();
    }
}
```

### 8、synchronized底层实现

#### 【1】历史问题

JDK 早期的时候，synchronized 的底层实现是重量级的，synchronized 每次都是要到操作系统内核态申请这把锁，这就会造成 synchronized 效率非常低

Java 后来开始处理高并发的程序的时候，很多程序员都不满意，认为synchrionized 锁太重了，所以改进，后来的改进才有了锁升级的概念关于这个锁升级的概念

#### 【2】锁升级

锁升级的过程：
1. 先去访问某把锁的线程比如 sync(Object) ，来了之后先在这个 Object 的头上面 markword 记录这个线程。（第一个线程访问的时候实际上是没有给这个 Object 加锁的，在内部实现的时候，只是记录这个线程的ID（偏向锁））。
2. 偏向锁如果有线程争用的话，就升级为自旋锁，概念就是（有一个哥们儿在蹲马桶，另外来了一个哥们，他就在旁边儿等着，他不会跑到 CPU 的就绪队列里去，而就在这等着占用 CPU ，通过 while 循环在这儿转圈玩儿，很多圈之后不行的话就再一次进行升级）。
3. 自旋锁转圈十次之后，升级为重量级锁，重量级锁就是去操作系统那里去申请资源。这是一个锁升级的过程。

<a href="https:/blog.csdn.net/baidu 38083619/article/details/82527461/" target="_blank">参考</a>

需要注意并不是 CAS 的效率就一定比系统锁要高，这个要区分实际情况：执行时间短（加锁代码），线程数少，用自旋执行时间长，线程数多，用系统锁

## 二、volatile与CAS

### 1、volatile 程序

m(); 方法中首先定义了一个变量布尔类型等于 true，这里模拟的是一个服务器的操作，值为 true，程序就会不间断的运行，什么时候为 false 什么时候停止

测试：new Thread，启动一个线程，调用 m 方法，睡了一秒，最后 running 等于 false。

此时运行方法是不会停止的，如果加上 volatile ，那么结果就是启动程序一秒之后他就会方法就会停止（volatile 就是不停的追踪这个值，时刻看什么时候发生了变化）

```java
/**
 * volatile 关键字，使一个变量在多个线程间可见
 * A B线程都用到一个变量，java默认是A线程中保留一份copy，这样如果B线程修改了该变量，则A线程未必知道
 * 使用volatile关键字，会让所有线程都会读到变量的修改值
 *
 * 在下面的代码中，running是存在于堆内存的t对象中
 * 当线程t1开始运行的时候，会把running值从内存中读到t1线程的工作区，在运行过程中直接使用这个copy，并不会每次都去
 * 读取堆内存，这样，当主线程修改running的值之后，t1线程感知不到，所以不会停止运行
 *
 * 使用volatile，将会强制所有线程都去堆内存中读取running的值
 *
 * 可以阅读这篇文章进行更深入的理解
 * http://www.cnblogs.com/nexiyi/p/java_memory_model_and_thread.html
 *
 * volatile并不能保证多个线程共同修改running变量时所带来的不一致问题，也就是说volatile不能替代synchronized
 * @author mashibing
 */
public class T01_HelloVolatile {
    /*volatile*/ boolean running = true; //对比一下有无volatile的情况下，整个程序运行结果的区别
    void m() {
        System.out.println("m start");
        while(running) {
        }
        System.out.println("m end!");
    }

    public static void main(String[] args) {
        T01_HelloVolatile t = new T01_HelloVolatile();
        new Thread(t::m, "t1").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        t.running = false;
    }
}
```

### 2、volatile作用

#### 【1】保证线程的可见性

Java 中的堆内存是所有线程共享的内存，除了共享的内存之外，每个线程都有自己的专属的区域（工作内存）

如果在共享内存中有一个值，当各个线程都要去访问这个值的时，会将这个值 copy 一份，copy 到自己的这个工作空间里面，然后对这个值进行任何改变，首先是在自己的空间里进行改变，改完之后会马上写回去，但不好控制什么时候去检查这个是否被更新

在这个线程里面发生的改变，并没有及时的反应到另外一个线程里面，这就是线程之间的不可见，对这个变量值加了 volatile 之后就能够保证一个线程的改变，另外一个线程马上就能看到。

从 CPU 层级的角度看，如果两个线程运行在不同的 CPU，那么上述的操作就是从共享内存（主存）中 copy 到 CPU 高速缓存中，操作结束后又会写回主存，如果一个变量在多个 CPU 中都存有缓存，就会出现缓存不一致，就是线程之间的不可见

解决这个问题有两种方法，都是在硬件层级实现的：

  - lock 总线锁：因为 CPU 和其他部件进行通信都是通过总线来进行的，如果对总线加 lock 锁的话，也就是说阻塞了其他 CPU 对其他部件访问（如内存），从而使得只能有一个 CPU 能使用这个变量的内存，但是效率很低
  - MESI 高速缓存一致性协议：详见 JVM

#### 【2】禁止指令重新排序

CPU 原来执行一条指令的时候它是一步一步的顺序的执行，但是现在的 CPU 为了提高效率，它会把指令并发的来执行，第一个指令执行到一半的时候第二个指令可能就已经开始执行了，这叫做流水线式的执行。在这种新的架构的设计基础之上想充分的利用这一点，那么就要求你的编译器把你的源码编译完的指令之后呢可能进行一个指令的重新排序。这个是通过实际工程验证了，不仅提高了，而且提高了很多。

如何禁止详见 JVM 中 volatile 具体实现

### 3、DCL单例

单例：保证在 JVM 的内存中永远只有某一个类的一个实例

在我们工程当中有一些类没有必要 new 好多个对象，比如权限管理者。

#### 【1】饿汉式

在 Mgr01 类中定义了这个类的对象，同时将 Mgr01 的构造方法设置成 private，不允许被其他程序 new

理论上来说我就只有自己一个实例了，通过 getInstance(); 访问这个实例，所以无论调用多少次，其本质上只有这一个对象，由 JVM 来保证永远只有这一个实例

```java
public class Mgr01 {
    private static final Mgr01 INSTANCE = new Mgr01();
    private Mgr01(){}
    public static Mgr01 getInstance(){return INSTANCE;}
    public void m(){
        System.out.println("m");
    }

    public static void main(String[] args) {
        Mgr01 m1 = Mgr01.getInstance();
        Mgr01 m2 = Mgr01.getInstance();
        System.out.println(m1 == m2);
    }
}
```

#### 【2】调用方法的时候再初始化

```java
public class Mgr02 {
    private static final Mgr02 INSTANCE;
    static {
        INSTANCE = new Mgr02();
    }
    private Mgr02(){}
    public static Mgr02 getInstance(){return INSTANCE;}
}
```

#### 【3】懒汉式

要求线程安全，但在多线程下会存在问题 

```java
public class Mgr03 {
    private static Mgr03 INSTANCE;

    private Mgr03(){}
    public static /*synchronized*/ Mgr03 getInstance() {
        if (INSTANCE == null) {
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            INSTANCE = new Mgr03();
        }
        return INSTANCE;
    }
}
```

#### 【4】懒汉式变型

通过减小同步代码块的方式来提高效率，不可行

```java
public class Mgr05 {
    private static  Mgr05 INSTANCE;
    private Mgr05(){}
    public static Mgr05 getInstance(){
        if (INSTANCE == null){
            // 妄图通过减小同步代码块的方式提高效率，不可行
            synchronized (Mgr05.class){
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            INSTANCE = new Mgr05();
        }
        return INSTANCE;
    }
}
```

#### 【5】DCL

第一个线程进来后判断，确实是空值，然后进行下面的初始化过程

假设第一个线程把这个 INSTANCE 已经初始化了，此时第二个线程进入

第一个线程检查等于空的时候，第二个线程检查也等于空，所以第二个线程在 `if(INSTANCE == null)` 这句话的时候停住了

暂停之后，第一个线程已经把它初始化完了释放锁，第二个线程继续往下运行，往下运行的时候它会尝试拿这把锁

此时第一个线程已经释放了，它是可以拿到这把锁的，注意，拿到这把锁之后他还会进行一次检查，由于第一个线程已经把 INSTANCE 初始化了所以这个检查通过了，它不会在重新 new 一遍

因次，双重检查是能够保证线程安全的

```java
public class Mgr06 {
    private static volatile Mgr06 INSTANCE;
    private Mgr06(){}
    public static Mgr06 getInstance(){
        if (INSTANCE == null){
            synchronized (Mgr06.class){
                if (INSTANCE == null){
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    INSTANCE = new Mgr06();
                }
            }
        }
        return INSTANCE;
    }
}
```

#### 【6】要不要加volatile（面试题）

需要加 volatile，不加 volatile 就会出现指令重排序

第一个线程 `INSTANCE = new Mgr06();` 经过编译器编译之后的指令被分成三步 
1. 给指令申请内存 
2. 给成员变量初始化 
3. 将这块内存的内容赋值给 INSTANCE 

这个过程经过了 4 条指令，可能会发生指令重排，在初始化未完成的时候，直接将内存内容赋值给 INSTANCE ，其他线程访问的时候，发现 INSTANCE 不为孔，直接用了这个错的值做操作

加了 volatile 之后，对这个对象上的指令重排序是不允许存在的，所以在这个时候一定是保证初始化完成之后才会赋值给你这个变量

#### 【7】验证volatile不能替代synchronized

分析程序：如果不加 volatile 是一定会有问题的，结果是到不了 10 万的

原因很简单，count 值改变之后只是被别的线程所看见，但是光看见没用，count++ 本身它不是一个原子性的操作，所以说 volatile 保证线程的可见性，并不能替代 synchronized，保证不了原子性。

要想解决这个问题，加上 synchronized

```java
class T10{
    volatile int count = 0;
    /*synchronized*/ void m(){
        for (int i = 0; i < 100000; i++) {
            count++;
        }
    }
    public static void main(String[] args) {
        T10 t = new T10();
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
           threads.add(new Thread(t::m,"thread-"+i));
        }
        threads.forEach((o)->{o.start();});
        threads.forEach((o)->{
            try {
                o.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println(t.count);
    }
}
```

### 4、锁优化

锁优化的内容非常多，这些优化分布在不同的步骤上，大体分为两种：把锁粒度变细、把锁粒度变粗

#### 【1】锁优化

把锁粒度变细：
- 在 synchronized 锁征用不是很剧烈的前提下，可以将锁粒度变小
- 见下面程序，如果 m1 方法前面有一堆业务逻辑，后面有一堆业务逻辑，这个业务逻辑用 sleep 来模拟它，中间是需要加锁的代码，此时不应该把锁加在整个方法上，只应该加在 count++ 上（参见m2），这很简单就叫做锁的细化

```java
/**
 * synchronized优化
 * 同步代码块中的语句越少越好
 * 比较m1和m2
 * @author mashibing
 */
public class FineCoarseLock {

    int count = 0;

    synchronized void m1() {
        //do sth need not sync
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //业务逻辑中只有下面这句需要sync，这时不应该给整个方法上锁
        count ++;

        //do sth need not sync
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    void m2() {
        //do sth need not sync
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //业务逻辑中只有下面这句需要sync，这时不应该给整个方法上锁
        //采用细粒度的锁，可以使线程争用时间变短，从而提高效率
        synchronized(this) {
            count ++;
        }
        //do sth need not sync
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

把锁粒度变粗：当锁征用特别频繁，由于锁的粒度越变越细，好多小的细锁跑在程序上，方法或者某一段业务逻辑中，不如变成一把大锁，征用反而就没有那么频繁了，程序写的好，不会发生死锁

#### 【2】特殊情况

下面有一个小概念，在某一种特定的不小心的情况下，把o变成了别的对象了，这个时候线程的并发就会出问题。锁是在对象的头上的两位来作为代表的，你这线程本来大家都去访问这两位了，结果突然把这把锁变成别的对象，去访问别的对象的两位了，原对象和锁之间就没有任何关系了。因此，以对象作为锁的时候不让它发生改变，加final。

```java
/**
 * 锁定某对象o，如果o的属性发生改变，不影响锁的使用
 * 但是如果o变成另外一个对象，则锁定的对象发生改变
 * 应该避免将锁定对象的引用变成另外的对象
 * @author mashibing
 */
public class SyncSameObject {
    /*final*/ Object o = new Object();
    void m() {
        synchronized(o) {
            while(true) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName());
            }
        }
    }

    public static void main(String[] args) {
        SyncSameObject t = new SyncSameObject();
        //启动第一个线程
        new Thread(t::m, "t1").start();
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //创建第二个线程
        Thread t2 = new Thread(t::m, "t2");
        //锁对象发生改变，所以t2线程得以执行，如果注释掉这句话，线程2将永远得不到执行机会
        t.o = new Object(); 
        t2.start();
    }
}
```

### 5、CAS

CAS号称是无锁优化，或者叫自旋。

我们通过Atomic类（原子的）。由于某一些特别常见的操作，老是来回的加锁，加锁的情况特别多，所以java就提供了这些常见的操作这么一些个类，这些类的内部就自动带了锁，当然这些锁的实现并不是synchronized重量级锁，而是CAS的操作来实现的（号称无锁）。

凡是以Atomic开头的都是用CAS这种操作来保证线程安全的这么一些个类。AtomicnIteger的意思就是里面包了一个Int类型，这个int类型的自增count++是线程安全的，由于我们在工作开发中经常性的有那种需求，一个值所有的线程共同访问它往上递增，所以JDK专门提供了这样的一些类。使用方法AtomicInteger如下代码

```java
/**
 * 解决同样的问题的更高效的方法，使用AtomXXX类
 * AtomXXX类本身方法都是原子性的，但不能保证多个方法连续调用是原子性的
 *
 * @author mashibing
 */
public class T01_AtomicInteger {
    /*volatile*/ //int count1 = 0;
    AtomicInteger count = new AtomicInteger(0);
    /*synchronized*/ void m() {
        for (int i = 0; i < 10000; i++)
            //if count1.get() < 1000
            count.incrementAndGet(); //count1++
    }

    public static void main(String[] args) {
        T01_AtomicInteger t = new T01_AtomicInteger();
        List<Thread> threads = new ArrayList<Thread>();
        for (int i = 0; i < 10; i++) {
            threads.add(new Thread(t::m, "thread-" + i));
        }
        threads.forEach((o) -> o.start());
        threads.forEach((o) -> {
            try {
                o.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println(t.count);
    }
}
```

### 6、CAS原理

内部实现的原理叫CAS操作，incrementAndGet()调用了getAndAddInt

```java
/**
     * Atomically increments by one the current value.
     *
     * @return the updated value
     */
    public final int incrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
```

当然这个也是一个CompareAndSetint操作

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```

AtomicInteger它的内部是调用了Unsafe这个类里面的方法CompareAndSetl(CAS），字面意思是比较并且设定。我原来想改变某一个值0，我想把它变成1，但是其中我想做到线程安全，就只能加锁synchronized，不然线程就不安全。我现在可以用另外一种操作来替代这把锁，就是cas操作，你可以把它想象成一个方法，这个方法有三个参数，cas （V，Expected，NewValue）。

```c
cas(V,Expected, NewValue)
    - if V == E
      V = New
      otherwise try again or fail
    - CPU 原语支持
```

第一个参数是要改的那个值；Expected第二个参数是期望当前的这个值会是几；NewValue要设定的新值。

比如原来这个值是3，我这个线程想改这个值的时候我一定期望你现在是3，是3我才改，如果你在我该的过程中变成4了，那你跟我的期望值就对不上了，说明有另外一个线程改了这个值了，那我这个cas就重新在试一下，再试的时候我希望你这个值是4，在修改的时候期望值是4，没有其他的线程修改这个值，那好，我给你改成5，这就是cas操作。

Expected如果对的上期望值，NewValue才会去对其修改，进行新的值设定的时候，这个过程之中来了一个线程把你的值改变了怎么办，我就可以再试一遍，或者失败，这个是cas操作。

当你判断的时候，发现是我期望的值，还没有进行新值设定的时候值发生了改变怎么办，Cas是cpu的原语支持，也就是说cas操作是cpu指令级别上的支持，中间不能被打断。

### 7、ABA问题（面试）

假如有一个值，我拿到这个值是1，想把它变成2，我拿到1用cas操作，期望值是1，准备变成2，这个对象Object，在这个过程中，没有一个线程改过我肯定是可以更改的，但是如果有一个线程先把这个1变成了2后来又变回1，中间值更改过，它不会影响我这个cas下面操作，这就是ABA问题。

解决：如果是int类型的，最终值是你期望的，这种没关系可以不去管这个问题。如果你确实想管这个问题可以加版本号，做任何一个值的修改，修改完之后加一，后面检查的时候连带版本号一起检查。
- 如果是基础类型：无所谓。不影响结果值；
- 如果是引用类型：可能会影响到对象中的内部逻辑

### 8、Unsafe

#### 【1】Unsafe 类操作

Unsafe 类可以直接操作虚拟机中的内存：`allocateMemory； putXX； freeMemory； pageSize`

可以生成类实例：`allocateInstance`

可以直接操作类或实例变量：`objectFieldOffset； getInt； getObject`

CAS 相关操作：`compareAndSwapObject Int Long`

#### 【2】CAS 为什么不需要加锁

原因是使用了Unsafe这个类，关于这个类，了解即可，这个类里面的方法非常非常多，而且这个类除了用反射使用之外，其他不能直接使用，不能直接使用的原因，和ClassLoader是有关系的。

所有的Atomic操作内部下面都是CompareAndSet这样的操作，那个CompareAndSet就是在Unsafe这个类里面完成的。

```java
public final class Unsafe {
    private static native void registerNatives();
    static {
        registerNatives();
    }
    private Unsafe() {}
    private static final Unsafe theUnsafe = new Unsafe();
}
```

#### （三）Unsafe 类的特点

Unsafe的构造方法是private的，单例的

Unsafe在新版本中使用的是弱引用weakCompareAndSetObject Int Long，好处是垃圾回收时的效率更高

## 三、Atomic类和线程同步新机制

### 1、Atomic

原来写m++在多线程访问的情况下需要加锁，那现在可以用AtomicInteger了，它内部就已经帮我们实现了原子操作，直接写count.incrementAndGet(); // count1++ 这个就相当于count++。

见如下小程序：模拟计数，所有的线程都要共同访问这个数count值，当所有线程都要访问这个数的时候，如果每个线程都往上加了1000，这时是需要加锁的，不加锁会出问题。但是，把它改成AomicInteger之后就不用在做加锁的操作了，因为incrementAndGet内部用了CAS操作，直接无锁的操作往上递增，原因是无欸、锁的操作效率会更高。

```java
/**
 * 解决同样的问题的更高效的方法，使用AtomXXX类
 * AtomXXX类本身方法都是原子性的，但不能保证多个方法连续调用是原子性的
 */
public class T01_AtomicInteger {
	/*volatile*/ //int count1 = 0;
	
	AtomicInteger count = new AtomicInteger(0); 

	/*synchronized*/ void m() { 
		for (int i = 0; i < 10000; i++)
			//if count1.get() < 1000
			count.incrementAndGet(); //count1++
	}
    
	public static void main(String[] args) {
		T01_AtomicInteger t = new T01_AtomicInteger();
		List<Thread> threads = new ArrayList<Thread>();
		for (int i = 0; i < 10; i++) {
			threads.add(new Thread(t::m, "thread-" + i));
		}
		threads.forEach((o) -> o.start());
		threads.forEach((o) -> {
			try {
				o.join();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		});
		System.out.println(t.count);
	}
}
```

见如下程序：模拟很多个线程对一个数进行递增。使用到以下三种方法：

- 第一种：一个long类型的数，递增的时候加锁；
- 第二种：使用AtomicLong可以让它不断的往上递增；
- 第三种：LongAdder;

由于很多线程对一个数进行递增这个事儿，工作当中经常会碰上，比如说在秒杀的时候。那么这三种哪个效率更高一些呢？很多测试来看AtomicLong还比不上第一种，但是在像现在这种测试的条件下AtomicLong他的效率还是比synchronized效率要高。程序中，count1、conut2、count3分别是以不同的方式进行实现递增，一上来启动了1000个线程，比较多算是，因为少的话模拟不了那么高的并发。将全部线程new出来，之后每一个线程都做了十万次递增，

- 第一种方式，打印起始时间 -> 线程开始 -> 所有线程结束 -> 打印结束时间 -> 计算最后花了多少时间；
- 第二种方式，是用synchronized，Object lock = new Object();，然后new Runnable()，在递增的时候使用的是synchronized (lock)，这里是替代了AtomicLong，之后计算时间；
-  第三种方式，用的是LongAdder，这个LongAdder里面直接就是count3.increment()；

- 我们跑起來对比LongAdder是效率最高的，如果线程数变小了LongAdder来必有优势，循环数量少了LongAdder也未必有优势，所以，实际当中用哪种你要考虑一下你的并发有多高


```java
public class T02_AtomicVsSyncVsLongAdder {
    static long count2 = 0L;
    static AtomicLong count1 = new AtomicLong(0L);
    static LongAdder count3 = new LongAdder();

    public static void main(String[] args) throws Exception {
        Thread[] threads = new Thread[1000];

        for(int i=0; i<threads.length; i++) {
            threads[i] =
                    new Thread(()-> {
                        for(int k=0; k<100000; k++) count1.incrementAndGet();
                    });
        }

        long start = System.currentTimeMillis();
        for(Thread t : threads ) t.start();
        for (Thread t : threads) t.join();
        long end = System.currentTimeMillis();
        //TimeUnit.SECONDS.sleep(10);
        System.out.println("Atomic: " + count1.get() + " time " + (end-start));
        //-----------------------------------------------------------
        Object lock = new Object();
        for(int i=0; i<threads.length; i++) {
            threads[i] =
                new Thread(new Runnable() {
                    @Override
                    public void run() {

                        for (int k = 0; k < 100000; k++)
                            synchronized (lock) {
                                count2++;
                            }
                    }
                });
        }
        start = System.currentTimeMillis();
        for(Thread t : threads ) t.start();
        for (Thread t : threads) t.join();
        end = System.currentTimeMillis();
        System.out.println("Sync: " + count2 + " time " + (end-start));
        //----------------------------------
        for(int i=0; i<threads.length; i++) {
            threads[i] =
                    new Thread(()-> {
                        for(int k=0; k<100000; k++) count3.increment();
                    });
        }
        start = System.currentTimeMillis();
        for(Thread t : threads ) t.start();
        for (Thread t : threads) t.join();
        end = System.currentTimeMillis();
        //TimeUnit.SECONDS.sleep(10);
        System.out.println("LongAdder: " + count1.longValue() + " time " + (end-start));
    }
    static void microSleep(int m) {
        try {
            TimeUnit.MICROSECONDS.sleep(m);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

为什么Atomic要比Sync快？

- 因为Atomic不加锁，synchronized是要加锁的，有可能要去操作系统申请重量级锁，所以synchronized效率偏低（在这种情景下）

LongAdder为什么比Atomic效率高？
- <img src="_media/basis/多线程与高并发/LongAdder.png"/>
- 因为LongAdder的内部做了一个分段锁。在它的内部会把一个值放到一个数组里，比如说数组长度是4，最开始是0，1000个线程将250个线程锁在第一个数租元素里，以此类推，每一个都往上递增算出来结果在加到一起。

### 2、ReentrantLock

ReentranLlock是基于CAS的可重入锁，synchronized本身就是可重入锁的一种，意思就是锁了一次之后还可以对同样这把锁再锁一次，synchronized必须是可重入的，不然子类调用父类是没法实现的，、

看如下小程序：synchronized m1方法里面做了一个循环每次睡1秒钟，每隔一秒种打印一次。当满足条件时调用synchronized m2方法。在这个执行过程中，一个线程对synchronized(this)这把锁申请了两次，并没有发生死锁。

```java
public class T01_ReentrantLock1 {
	synchronized void m1() {
		for(int i=0; i<10; i++) {
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(i);
			if(i == 2) m2();
		}
		
	}
	synchronized void m2() {
		System.out.println("m2 ...");
	}
	public static void main(String[] args) {
		T01_ReentrantLock1 rl = new T01_ReentrantLock1();
		new Thread(rl::m1).start();
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

子类和父类如果是synchronized(this)就是同一把锁，同一个this当然是同一把锁。

RentantLock是可以替代synchronized的，见如下代码：原来写synchronized的地方换写lock.lock()，加完锁之后需要注意的是记得lock.unlock()解锁，由于synchromized是自动解锁的，大括号执行完就结束了。lock就不行，lock必须得手动解锁，手动解锁一定要写在try.finaly里面保证最好一定要解锁，不然的话上锁之后中间执行的过程有问题了，死在那了，别人就永远也拿不到这把锁了。

```java
/**
 * reentrantlock用于替代synchronized
 * 需要注意的是，必须要必须要必须要手动释放锁（重要的事情说三遍）
 * 使用syn锁定的话如果遇到异常，jvm会自动释放锁，但是lock必须手动释放锁，因此经常在finally中进行锁的释放
 */
public class T02_ReentrantLock2 {
	Lock lock = new ReentrantLock();

	void m1() {
		try {
			lock.lock(); //synchronized(this)
			for (int i = 0; i < 10; i++) {
				TimeUnit.SECONDS.sleep(1);
				System.out.println(i);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}

	void m2() {
		try {
			lock.lock();
			System.out.println("m2 ...");
		} finally {
			lock.unlock();
		}
	}

	public static void main(String[] args) {
		T02_ReentrantLock2 rl = new T02_ReentrantLock2();
		new Thread(rl::m1).start();
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		new Thread(rl::m2).start();
	}
}
```

RentrantLock相比于synchronized，有一些功能还是比较强大的

可以使用 tryLock进行尝试锁定，不管锁定与否，方法都将继续执行，Synchronized如果搞不定的话他肯定就阻塞了，但是用ReentrantLock你自己就可以决定你到底要不要wait。下面程序：如果第线程m1，5秒钟把程序执行完则线程m2可能得到这把锁。由于我的第一个钱程跑了10秒钟，所以第二个线程里申请5秒肯定是那不到的，把循环次数减少就可以能拿到了。

```java
public class T03_ReentrantLock3 {
	Lock lock = new ReentrantLock();

	void m1() {
		try {
			lock.lock();
			for (int i = 0; i < 3; i++) {
				TimeUnit.SECONDS.sleep(1);
				System.out.println(i);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}

	/**
	 * 使用tryLock进行尝试锁定，不管锁定与否，方法都将继续执行
	 * 可以根据tryLock的返回值来判定是否锁定
	 * 也可以指定tryLock的时间，由于tryLock(time)抛出异常，所以要注意unclock的处理，必须放到finally中
	 */
	void m2() {
		boolean locked = false;
		try {
			locked = lock.tryLock(5, TimeUnit.SECONDS);
			System.out.println("m2 ..." + locked);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			if(locked) lock.unlock();
		}
	}

	public static void main(String[] args) {
		T03_ReentrantLock3 rl = new T03_ReentrantLock3();
		new Thread(rl::m1).start();
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		new Thread(rl::m2).start();
	}
}
```

ReentrantLock还可以用lock.lockInteruptibly() 这个类，对interrupt()方法做出响应，可以被打断的加锁。以这种方式加锁就可以调用一个t2.interrupt()打断线程2的等待。下面程序：线程1上来加锁，加锁之后开始睡，睡的没完没了的，这时线程2想拿到这把锁不太可能，拿不到锁线程2就会在那等着，如果我们使用原来的这种lock.lock()是打断不了线程2让其继续执行的，那么我们就可以用另外一种方式lock.lockInteruptibly() ，想停止线程2的时候就可以用interrupt()放法。

```java
public class T04_ReentrantLock4 {		
	public static void main(String[] args) {
		Lock lock = new ReentrantLock();
		Thread t1 = new Thread(()->{
			try {
				lock.lock();
				System.out.println("t1 start");
				TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
				System.out.println("t1 end");
			} catch (InterruptedException e) {
				System.out.println("interrupted!");
			} finally {
				lock.unlock();
			}
		});
		t1.start();		
		Thread t2 = new Thread(()->{
			try {
				//lock.lock();
				lock.lockInterruptibly(); //可以对interrupt()方法做出响应
				System.out.println("t2 start");
				TimeUnit.SECONDS.sleep(5);
				System.out.println("t2 end");
			} catch (InterruptedException e) {
				System.out.println("interrupted!");
			} finally {
				lock.unlock();
			}
		});
		t2.start();		
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.interrupt(); //打断线程2的等待
	}
}
```

ReentantLock还可以指定为公平锁，当我们new一个ReentrantLock时，可以传一个参数为true，这个true表示公平锁，公平锁的意思是谁等在前面就先让谁执行，而不是说谁后来了之后就马上让谁执行。如果说这个锁不公平，来了一个线程上来就抢，它是有可能抢到的，如果说这个锁是个公平锁，这个线程上来会先检查队列里有没有原来等着的，如果有的话他就先进队列里等着别人先运行。ReentrantLock默认是非公平锁。

注：公平锁并非完全公平，仍然会出现一个线程连续获得锁的现象，是因为线程调用 start() 之后，并不是立刻开始执行 run() 方法，它需要等待其他系统资源准备就绪

例如：CPU 资源，t2 最开始可能没有获取到 CPU 时间，t1 却获取到并跑了几次

<img src="_media/basis/多线程与高并发/公平锁解释.png"/>

```java
public class T05_ReentrantLock5 extends Thread {
		//参数为true表示为公平锁，请对比输出结果
	private static ReentrantLock lock=new ReentrantLock(true); 
    public void run() {
        for(int i=0; i<100; i++) {
            lock.lock();
            try{
                System.out.println(Thread.currentThread().getName()+"获得锁");
            }finally{
                lock.unlock();
            }
        }
    }
    public static void main(String[] args) {
        T05_ReentrantLock5 rl=new T05_ReentrantLock5();
        Thread th1=new Thread(rl);
        Thread th2=new Thread(rl);
        th1.start();
        th2.start();
    }
}
```

回顾一下: Reentrantlock vs synchronized
- ReentrantLock可以替代synchronized这是没问题的，他也可以重入，可以锁定的。本身的底层是cas
- trylock：自己来控制，锁不住该怎么办
- lockinterruptibly：这个类，中间你还可以被打断
- 还可以公平和非公平的切换
- 现在除了synchronized之外，多数内部都是用的cas。当我们聊这个AQS的时候实际上它内部用的是park和unpark，也不是全都用的cas，他还是做了一个锁升级的概念，只不过这个锁升级做的比较隐秘，在你等待这个队列的时候如果你拿不到还是进入一个阻塞的状态，前面至少有一个cas的状态，不像原先就直接进入阻塞状态了。

Reentrantlock和synchronized不同：

- synchronized：系统自带、系统自动加锁，自动解锁、不可以出现多个不同的等待队列、默认进行四种领状态的升级
- ReentrantLock：需要手动枷锁，手动解锁、可以出现多个不同的等待队列、CIS的实现

### 3、CountDownLatch

CoumtDown叫倒数，Latch是门栓的意思（倒数的一个门栓，5、4、3、 2、1数到了，我这个门栓就开了

看下面的小程序：new了100个线程，且定义了100个数量的CoumtDownLatch，就是说门栓Latch上记了个数threads.length是100，每一个线程结束的时候我让 latch.countDown()，然后所有线程start，再latch.awaiy()，最后结束。latch.await()的意思是阻塞，每个线程执行到 latch.await() 的时候这个门栓就在这里等着，并且记了个数是100，每一个线程结束的时候都会往下CountDown，CountDown是在原来的基础上减1，一直到这个数字变成0的时候门栓就会被打开，这就是它的概念，它是用来等着线程结束的。

对比于join来说，用join实际上不太好控制，必须要你线程结束了才能控制，但是如果是一个门栓的话，在线程里不停的CoumtDown，在一个线程里就可以控制这个门栓什么时候往前走，用join我只能是当前线程结束了你才能自动往前走，当然用join可以，但是CountDown比它要灵活。

```java
public class T06_TestCountDownLatch {
    public static void main(String[] args) {
        usingJoin();
        usingCountDownLatch();
    }

    private static void usingCountDownLatch() {
        Thread[] threads = new Thread[100];
        CountDownLatch latch = new CountDownLatch(threads.length);
        for(int i=0; i<threads.length; i++) {
            threads[i] = new Thread(()->{
                int result = 0;
                for(int j=0; j<10000; j++) result += j;
                latch.countDown();
            });
        }
        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }
        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end latch");
    }

    private static void usingJoin() {
        Thread[] threads = new Thread[100];
        for(int i=0; i<threads.length; i++) {
            threads[i] = new Thread(()->{
                int result = 0;
                for(int j=0; j<10000; j++) result += j;
            });
        }
        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }
        for (int i = 0; i < threads.length; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("end join");
    }
}
```

### 4、CyclicBarrier

CyclicBarrier，意思是循环栅栏，想象有一个栅栏，什么时候人满了就把栅栏推倒，都放出去，出去之后栅栏又重新起来，再来人，满了，推倒之后又继续。

下面程序：两个参数，第二个参数不传也是可以的，就是满了之后不做任何事情。第一个参数是20，满20之后帮我调用第二个参数指定的动作，我们这个指定的动作就是一个Runnable对象，打印（满人，发车）。下面第一种写法是满了之后我什么也不做，第二种写法是用Labda表达式的写法。这个意思就是线程堆满了，我们才能往下继续执行。

```java
public class T07_TestCyclicBarrier {
    public static void main(String[] args) {
        //CyclicBarrier barrier = new CyclicBarrier(20);

        CyclicBarrier barrier = new CyclicBarrier(20, () -> System.out.println("满人，发车"));

        /*CyclicBarrier barrier = new CyclicBarrier(20, new Runnable() {
            @Override
            public void run() {
                System.out.println("满人，发车");
            }
        });*/

        for(int i=0; i<100; i++) {
                new Thread(()->{
                    try {
                        barrier.await();

                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } catch (BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                }).start();          
        }
    }
}
```

场景：
- 可以进行复杂操作，包括数据库、网络、文件
- 可以并发执行，对线程的操作
- 比如业务需求要从数据库、网络、硬盘取文件，一种方法是顺序执行；另一种是并发执行，由不同的线程去执行不同的操作，但必须等这几个操作都结束后，才能往下继续

### 5、Phaser（JDK1.7之后）

Phaser它就更像是结合了CountDownLatch和CyclicBarrier，Phaser翻译过来叫阶段。

Phaser是按照不同的阶段来对线程进行执行，就是它本身是维护着一个阶段这样的一个成员变量，当前我是执行到那个阶段，是第0个，还是第1个阶段等等，每个阶段不同的时候这个线程都可以往前走，有的线程走到某个阶段就停了，有的线程一直会走到结束。你的程序中如果说用到分好几个阶段执行，而且有的人必须得几个人共同参与的一种情形的情况下可能会用到这个Phaser。

有种情形很可能用到，如果你写的是遗传算法，遗传算法是计算机来模拟达尔文的进化策略所发明的一种算法，当你去解决这个问题的时候这个Phaser是有可能用的上的。这个东西更像是CydicBarrier，原来是一个循环的栅栏，循环使用，但是这个栅栏是一个栅栏一个栅栏的。

模拟结婚场景：结婚是有好多人要参加的，因此，定义了一个类Person实现Runnable，模拟我们每个人要做一些操作，有4个方法，arrive（到达）、eat（吃）、leavel（离开）、hug（拥抱）。作为一个婚礼来说它会分成好几个阶段，第一阶段大家都得到齐了，第二个阶段大家开始吃饭，三阶段大家离开，第四个阶段新郎新娘人洞房，每个人都有这几个方法，在方法的实现里头我就简单的睡了1000个毫秒

在看主程序，一共有五个人参加婚礼，接下来新郎，新娘参加婚礼，一共七个人。start之后就调用 run() 方法，它会顺序调用每一个阶段的方法。我们在每一个阶段需要控制人数，第一个阶段得要人到期了才能开始，二阶段所有人都吃饭，三阶段所有人都离开，但是，到了第四阶段进入洞房的时候就只能新郎和新娘做。所以，要模拟一个程序就要把整个过程分好几个阶段，而且每个阶段必须要等这些线程完成了才能进入下一个阶段。

模拟的过程：定义了一个MarriagePhaser从Phaser这个类继承，重写onAdvance（前进）方法，线程抵达这个栅栏的时候，所有的线程都满足了这个第一个栅栏的条件了onAdvance会被自动调用，目前我们有好几个阶段，这个阶段是被写死的，必须是数字0开始，onAdvance会传来两个参数phase是第几个阶段，registeredParties是目前这个阶段有几个人参加，每一个阶段都有一个打印，返回值false，一直到最后一个阶段返回true，所有线程结束，整个Phaser栅栏组就结束了。

怎么才能让线程在一个栅栏面前给停住？ 就是调用phaser.arriveAndAwaitAdvance()这个方法，这个方法的意思是到达等待继续往前走，直到新郎新娘入洞房，其他人不在参与，调用phaser.arriveAndDeregister()这个方法。还有可以调用方法phaser.register()往上加，不仅可以控制栅栏上的个数还可以控制栅栏上的等待数量，这个就叫做phaser。

```java
public class T09_TestPhaser2 {
    static Random r = new Random();
    static MarriagePhaser phaser = new MarriagePhaser();
    static void milliSleep(int milli) {
        try {
            TimeUnit.MILLISECONDS.sleep(milli);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        phaser.bulkRegister(7);
        for(int i=0; i<5; i++) {
            new Thread(new Person("p" + i)).start();
        }
        new Thread(new Person("新郎")).start();
        new Thread(new Person("新娘")).start();
    }

    static class MarriagePhaser extends Phaser {
        @Override
        protected boolean onAdvance(int phase, int registeredParties) {

            switch (phase) {
                case 0:
                    System.out.println("所有人到齐了！" + registeredParties);
                    System.out.println();
                    return false;
                case 1:
                    System.out.println("所有人吃完了！" + registeredParties);
                    System.out.println();
                    return false;
                case 2:
                    System.out.println("所有人离开了！" + registeredParties);
                    System.out.println();
                    return false;
                case 3:
                    System.out.println("婚礼结束！新郎新娘抱抱！" + registeredParties);
                    return true;
                default:
                    return true;
            }
        }
    }

    static class Person implements Runnable {
        String name;

        public Person(String name) {
            this.name = name;
        }

        public void arrive() {
            // 自定义milliSleep方法 ，封装trycatch
            milliSleep(r.nextInt(1000));
            System.out.printf("%s 到达现场！\n", name);
            phaser.arriveAndAwaitAdvance();
        }

        public void eat() {
            milliSleep(r.nextInt(1000));
            System.out.printf("%s 吃完!\n", name);
            phaser.arriveAndAwaitAdvance();
        }

        public void leave() {
            milliSleep(r.nextInt(1000));
            System.out.printf("%s 离开！\n", name);


            phaser.arriveAndAwaitAdvance();
        }

        private void hug() {
            if(name.equals("新郎") || name.equals("新娘")) {
                milliSleep(r.nextInt(1000));
                System.out.printf("%s 洞房！\n", name);
                phaser.arriveAndAwaitAdvance();
            } else {
                phaser.arriveAndDeregister();
                //phaser.register()
            }
        }

        @Override
        public void run() {
            arrive();
            eat();
            leave();
            hug();
        }
    }
}
```

### 6、ReadWriteLock

ReadWriteLock是读写锁。读写锁的概念其实就是共享锁和排他锁，读锁就是共享锁，写锁就是排他锁。首先要理解，读写有很多种情况，比如说你数据库里的某条数据你放在内存里读的时候特别多，而改的时候并不多。

举一个例子：公司的组织结构，我们要想显示这组织结构下有哪些人在网页上访问，所以这个组织结构被访问到会读，但是很少更改，读的时候多写的时候就并不多，这个时候好多线程来共同访问这个结构的话，有的是读线程有的是写线程，要求不产生数据不一致的情况下采用最简单的方式就是加锁，读的时候只能自己读，写的时候只能自己写，但是这种情况下效率会非常的底，尤其是读线程非常多的时候，那就可以做读写锁，当读线程上来的时候加一把锁是允许其他读线程可以读，写线程来了不给它，你先别写，等我读完你在写。读线程进来的时候我们大家一块读，因为你不改原来的内容，写线程上来把整个线程全锁定，你先不要读，等我写完你在读。

读写锁的使用：定义两个方法，read()读一个数据，write()写一个数据。read这个数据的时候我需要你往里头传一把锁，这个传那把锁你自己定，我们可以传自己定义的全都是排他锁，也可以传读写锁里面的读锁或写锁。write的时候也需要往里面传把锁，同时需要你传一个新值，在这里值里面传一个内容。我们模拟这个操作，读的是一个int类型的值，读的时候先上锁，设置一秒钟，之后read over，最后解锁unlock。再下面写锁，锁定后睡1000毫秒，然后把新值给value，write over后解锁。

我们现在的问题是往里传这个lock有两种传法，第一种直接new ReentrantLock()传进去，分析下这种方法，主程序定义了一个Runnable对象，第一个是调用read()方法，第二个是调用write()方法同时往里头扔一个随机的值，然后起了18个读线程，起了两个写线程，这个两个我要想执行完的话，我现在传的是一个ReentrantLock，这把锁上了之后没有其他任何人可以拿到这把锁，而这里面每一个线程执行都需要1秒钟，在这种情况下你必须得等20秒才能结束；

第二种，我们换了锁 new ReentrantReadWriteLock()是ReadWriteLock的一种实现，在这种实现里头我又分出两把锁来，一把叫readLock，一把叫writeLock，通过他的方法readWriteLock.readLock()来拿到readLock对象，读锁我就拿到了。通过readWriteLock.writeLock()拿到writeLock对象。这两把锁在我读的时候扔进去，因此，读线程是可以一起读的，也就是说这18个线程可以一秒钟完成工作结束。所以使用读写锁效率会大大的提升。

```java
public class T10_TestReadWriteLock {
    static Lock lock = new ReentrantLock();
    private static int value;

    static ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    static Lock readLock = readWriteLock.readLock();
    static Lock writeLock = readWriteLock.writeLock();

    public static void read(Lock lock) {
        try {
            lock.lock();
            Thread.sleep(1000);
            System.out.println("read over!");
            //模拟读取操作
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void write(Lock lock, int v) {
        try {
            lock.lock();
            Thread.sleep(1000);
            value = v;
            System.out.println("write over!");
            //模拟写操作
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public static void main(String[] args) {
        //Runnable readR = ()-> read(lock);
        Runnable readR = ()-> read(readLock);
        //Runnable writeR = ()->write(lock, new Random().nextInt());
        Runnable writeR = ()->write(writeLock, new Random().nextInt());
        for(int i=0; i<18; i++) new Thread(readR).start();
        for(int i=0; i<2; i++) new Thread(writeR).start();
    }
}
```

### 7、新锁和synchronized比重及分布式锁

以后还写不写synchronized？分布式锁咋实现？

以后一般不用这些新的锁，多数都用synchronized。只有特别特别追求效率的时候才会用到这些新的锁。现在用分布式锁很多，分布式锁也是面试必问的，它也不难比较简单，redis有两种方法可以实现分布式锁，还有 ZooKeeper也可以实现分布式锁，还有数据库也可以实现，但数据库实现的效率就比较低了。

例子：比如秒杀。在开始秒杀之前它会从数据库里面读某一个数据，比如所一个电视500台，只能最多售卖500台，完成这件事得是前面的线程访问同一个数最开始是0一直涨到500就结束，需要加锁，从0递增，如果是单机LongAdder或Atomicinteger搞定。分布式的就必须得用分布式锁，对一个数进行上锁。

### 8、Semaphore

Semaphore信号灯。可以往里面传一个数，permits是允许的数量，可以想着有几盏信号灯，一个灯里面闪着数字表示到底允许几个来参考我这个信号灯。

s.acquire()这个方法叫阻塞方法，阻塞方法的意思是说acquire不到的话就停在这，acquire的意思就是得到。如果Semaphore s = new Semaphore(1)写的是1，我取一下，acquire一下就变成0，当变成0之后别人是acquire不到的，然后继续执行，线程结束之后注意要s.release()，执行完该执行的就把他release掉，release又把0变回去1，还原化。

Semaphore的含义就是限流，比如说你在买票，Semaphore写5就是只能有5个人可以同时买票。acquire的意思叫获得这把锁，线程如果想继续往下执行，必须得从Semaphore里面获得一个许可，一共有5个许可用到0了就得等着。

例如，有一个八条车道的机动车道，这里只有两个收费站，到这儿，谁acquire得到其中某一个谁执行

默认Ssemaphore是非公平的，new Senaphore(2,true)第二个值传true设置公平。公平是有一堆队列在那等，大家过来排队。用这个车道和收费站来举例子，就是我们有四辆车都在等着进一个车道，当后面在来一辆新的时候，它不会超到前面去，要在后面排着这叫公平。所以说内部是有队列的

不仅内部是有队列的，以上reentrantlock、CountDownlath, Cyclitarier. Phaser, ReatdWitelock. Semaphore还有后面的Exchanger都是用同一个队列，同一个类来实现的，这个类叫AQS。

```java
public class T11_TestSemaphore {
    public static void main(String[] args) {
        //Semaphore s = new Semaphore(2);
        Semaphore s = new Semaphore(2, true);
        //允许一个线程同时执行
        //Semaphore s = new Semaphore(1);

        new Thread(()->{
            try {
                s.acquire();

                System.out.println("T1 running...");
                Thread.sleep(200);
                System.out.println("T1 running...");

            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                s.release();
            }
        }).start();

        new Thread(()->{
            try {
                s.acquire();

                System.out.println("T2 running...");
                Thread.sleep(200);
                System.out.println("T2 running...");

                s.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

### 9、Exchanger

看以下程序，定义了一个Exchanger，Exchanger叫做交换器，俩人之间互相交换个数据用的。

第一个线程有一个成员变量s，然后exchanger.exchangel(s)，第二个也是这样，t1线程名字叫T1，第二个线程名字叫T2。到最后，打印出来你会发现他们俩交换了一下。线程间通信的方式非常多，这只是其中一种，就是线程之间交换数据用的。

exchanger可以把它想象成一个容器，这个容器有两个值，两个线程，有两个格的位置，第一个线程执行到exchanger.exchange的时候，阻塞，但是要注意调用exchange方法的时候是往里面扔了一个值，可以认为把T1扔到第一个格子里，然后第二个线程开始执行，也执行到这句话了，exchange，他把自己的这个值T2扔到第二个格子里。接下来这两个值交换一下，T1扔给T2，T2扔给T1，两个线程继续往前跑。exchange只能是两个线程之间，交换这个东西只能两两进行。

<img src="_media/basis/多线程与高并发/exchanger.png"/>

```java
public class T12_TestExchanger {
    static Exchanger<String> exchanger = new Exchanger<>();

    public static void main(String[] args) {
        new Thread(()->{
            String s = "T1";
            try {
                s = exchanger.exchange(s);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " " + s);

        }, "t1").start();
        new Thread(()->{
            String s = "T2";
            try {
                s = exchanger.exchange(s);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " " + s);
        }, "t2").start();
    }
}
```

### 10、LockSupport

在以前我们要阻塞和唤醒某一个具体的线程有很多限制比如：
- 因为waitQ方法需要释放锁，所以必须在synchronized中使用，否则会抛出异常IlegalMonitorStateException
- notity()方法也必须在synchronized中使用，并且应该指定对象
- synchronized()、wait()、notify()对象必须一致，一个synchronized()代码块中只能有一个线程调用wait()或notify()

在JDK1.6中的java.util.concurrent的子包locks中引了LockSupport这个API，LockSupport是一个比较底层的工具类，用来创建锁和其他同步工具类的基本线程阻塞原语。

Java锁和同步器框架的核心 AQS：AbstractQueuedSynchronizer，就是通过调用 LockSupport .park()和LockSupport.unpark() 的方法来实现线程的阻塞和唤醒的：

```java
public class T13_TestLockSupport {
    public static void main(String[] args) {
        Thread t = new Thread(()->{
            for (int i = 0; i < 10; i++) {
                System.out.println(i);
                if(i == 5) {
                    LockSupport.park();
                }
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        t.start();
    }
}
```

从以上的小程序中，我们不难看出LockSuppont使用起来的是比较灵灵活的，没有了所谓的限制。我们来分析一下代码的执行过程，首先我们使用lonbda表达式创建了线程对象"t”，然后通过“t”对象调用线程的启动方法start()，然后我们再看线程的内容，在for循环中，当i的值等于5的时候，我们调用了LockSuppot的park方法使当前线程阻塞，注意看方法并没有加锁，就默认使当前线程阻塞了，由此可以看出LockSupprt.park()方法并没有加锁的限制。

```java
public class T13_TestLockSupport {
    public static void main(String[] args) {
        Thread t = new Thread(()->{
            for (int i = 0; i < 10; i++) {
                System.out.println(i);
                if(i == 5) {
                    LockSupport.park();
                }
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        t.start();
        LockSupport.unpark(t);
    }
}
```

分析以上程序：只需要在第一个小程序的主线程中，调用LockSuppor的unpark()方法，就可以唤醒某个具体的线程，这里我们指定了线程“t”，代码运行以后，线程并没有被胆塞，成功唤醒了线程＂t”，在这里还有一点，就是在主线程中线程't"调用了Start方法以后，因为紧接着执行了LockSuppor的unpark方法，所以也就是说，在线程't"还没有执行还没有被阻塞的时候，已经调用了Locksuppor的unparK方法来唤醒线程”t”，之后线程"t"才调用了LockSupport的park来使线程”t”阻塞，但是线程“t”并没有被阻塞，由此可以看出，LockSuppont的unpark方法可以先于park()方法执行。

```java
public class T13_TestLockSupport {
    public static void main(String[] args) {
        Thread t = new Thread(()->{
            for (int i = 0; i < 10; i++) {
                System.out.println(i);
                if(i == 5) {
                    LockSupport.park();
                }
                if(i == 8) {
                    LockSupport.park();
                }
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        t.start();
        LockSupport.unpark(t);
    }
```

分析以上小程序：在第二个小程序的基础上又添加了一个if 判断，在等于8的时候再次调用LockSupport的park()方法来使线程"t”阻塞，我们可以看到线程被阻塞了，原因是LockSupport的unpark()方法就像是获得了一个"令牌”，而LockSupport的park()方法就像是在识别"令牌”，当主线程调用了LockSupport.unpark(t)方法也就说明线程"t”已经获得了"令牌"，当线程"t"再调用LockSupport的park方法时，线程“t”已经有令牌了，这样他就会马上再继续运行，也就不会被阻塞了，但是当等于8的时候线程“t”再次调用了LockSupport的park()方法使线程再次进入阻塞状态，这个时候”令牌”已经被使用作废掉了，也就无法阻塞线程"t"了，而且如果主线程处于等待"令牌”状态时，线程“t”再次调用了LockSupport的park()方法，那么线程"t"就会永远阻塞下去，即使调用unpark()方法也无法唤醒了。

由以上三个小程序我们可以总结得出以下几点；

- LockSupport不需要synchornized加锁就可以实现线程的阻塞和唤醒
- LockSupport.unpartk()可以先于LockSupport,park()执行，并且线程不会阻塞
- 如果一个线程处于等待状态，连续调用了两次park()方法，就会使该线程永远无法被唤醒

LockSupport中park()和unpark()方法的实现原理：

- park()和unpark()方法的实现是由Unsefa类提供的，而Unsefa类是由C和C++语言完成的，它主要通过一个变量作为一个标识，变量值在0，1之间来回切换，当这个变量大于0的时候线程就获得了“令牌”，从这一点我们不难知道，其实park(和unpark()方法就是在改变这个变量的值，来达到线程的阻塞和唤醒的。

- ```java
  public static void park() {
          UNSAFE.park(false, 0L);
      }
  public static void unpark(Thread thread) {
          if (thread != null)
              UNSAFE.unpark(thread);
      }
  ```

## 四、淘宝面试题与源码阅读方法论

### 1、面试题1：

实现一个容器，提供两个方法add、size，写两个线程：线程1，添加10个元素到容器中，线程2，实时监控元素个数，当个数到5个时，线程2给出提示并结束

#### 【1】程序1

小程序1的执行流程：new一个ArrayList，在自定义的add方法直接调用list的add方法，在自定义的size方法直接调用list的size方法，想法很简单，首先小程序化了这个容器，接下来启动了t1线程，t1线程中做了一个循环，每次循环就添加一个对象，加一个就打印显示一下到第几个了，然后给了1秒的间隔，在t2线程中写了了一个while循环，实时监控着集合中对象数量的变化，如果数量达到5就结束t2线程。

小程序1的执行结果：方法并没有按预期的执行，我们注意看t2线程中c.size()这个方法，当对象添加以后，ArraylList的size(）方肯定是要更新的，我们分析一下，当t1线程中的size()方法要更新的时候，还没有更新t2线程就读了，这个时候t2线程读到的值就与实际当中加入的值不一致了，所以得出两结论，第一这个方案没有加同步，第二while(true)中的c.size()方法永远没有检测到，没有检测到的原因是线程与线程之间是不可见的。


```java
public class T01_WithoutVolatile {
	List lists = new ArrayList();
	public void add(Object o) {
		lists.add(o);
	}
	public int size() {
		return lists.size();
	}
	public static void main(String[] args) {
		T01_WithoutVolatile c = new T01_WithoutVolatile();

		new Thread(() -> {
			for(int i=0; i<10; i++) {
				c.add(new Object());
				System.out.println("add " + i);
				try {
					TimeUnit.SECONDS.sleep(1);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, "t1").start();
		
		new Thread(() -> {
			while(true) {
				if(c.size() == 5) {
					break;
				}
			}
			System.out.println("t2 结束");
		}, "t2").start();
	}
}
```

#### 【2】程序2

小程序2是在小程序1的基础上做了一些改动，用voblatle修饰了一下List集合，实现线程间信息的传递，但是还是有不足之处，程序还是无法运行成功，而且我们还得出，volatile—定要尽量去修饰普通的值，不要去修饰引用值，因为volate修饰引用类型，这个引用对象指向的是另外一个new出来的对象对象，如果这个对象里边的成员变量的值改变了，是无法观察到的，所以小程序2也是不理想的。


```java
	//添加volatile，使t2能够得到通知
	//volatile List lists = new LinkedList();
	volatile List lists = Collections.synchronizedList(new LinkedList<>());

	public void add(Object o) {
		lists.add(o);
	}

	public int size() {
		return lists.size();
	}

	public static void main(String[] args) {

		T02_WithVolatile c = new T02_WithVolatile();
		new Thread(() -> {
			for(int i=0; i<10; i++) {
				c.add(new Object());
				System.out.println("add " + i);				
			}
		}, "t1").start();
		
		new Thread(() -> {
			while(true) {
				if(c.size() == 5) {
					break;
				}
			}
			System.out.println("t2 结束");
		}, "t2").start();
	}
}
```

#### 【3】程序3

小程序3的执行流程：小程序3用了锁的方式（利用wait和notifjy），通过给object对象枷锁然后调用wait和notfy实现，首先List集合实现add和size方法，之后main方法里创建了object对象，然后写了两个线程t1和t2，t1用来增加对象，t2用来监控list集合添加的对象个数，在t2线程我们给object对象加锁，然后判断list集合对象的个数为5的时候，就调用wait方法阻塞t2线程，并给出相应提示，t1线程里给objec对象加锁，通过for循环来给list集合添加对象，当对象添加到5个的时候，唤醒t2线程来完成对象个数的监控，这里我们需要保证先启动的是第二个线程，让它直接进入监控状态，以完成实时监控。

小程序3的执行结果：当试过了小程序3，我们会发现，这种写法也是行不通的，原因是notfy方法不释放锁，当t1线程调用了notfiy方法后，并没有释放当前的锁，所以t1还是会执行下去，待到t1执行完毕，t2线程才会被唤醒接着执行，这个时候对象已经不只有5个了，所以这个方案也是行不通的。


```java
public class T03_NotifyHoldingLock { //wait notify

	//添加volatile，使t2能够得到通知
	volatile List lists = new ArrayList();

	public void add(Object o) {
		lists.add(o);
	}

	public int size() {
		return lists.size();
	}
	
	public static void main(String[] args) {
		T03_NotifyHoldingLock c = new T03_NotifyHoldingLock();
		
		final Object lock = new Object();
		
		new Thread(() -> {
			synchronized(lock) {
				System.out.println("t2启动");
				if(c.size() != 5) {
					try {
						lock.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
				System.out.println("t2 结束");
			}
			
		}, "t2").start();
		
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		}

		new Thread(() -> {
			System.out.println("t1启动");
			synchronized(lock) {
				for(int i=0; i<10; i++) {
					c.add(new Object());
					System.out.println("add " + i);					
					if(c.size() == 5) {
						lock.notify();
					}					
					try {
						TimeUnit.SECONDS.sleep(1);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
		}, "t1").start();		
	}
}
```

#### 【4】程序4（成功）

小程序4是在小程序3的基础上做了一些小改动，分析一下执行流程，首先2线程执行，判断到lis集合里的对象数量没有5个，t2线程被阻塞了，接下来1线程开始执行，当循环添加了5个对象后，唤醒了t2线程，重点在于notiy方法是不会是释放锁的，所以在notify以后，又紧接着调用了wait方法阻塞了t1线程，实现了t2线程的实时监控，t2线程执行结束，打印出相应提示，最后调用notfy方法唤醒t1线程，让t1线程完成执行。

执行结果，发现示例4完成了面试题的功能成功运行。

```java
public class T04_NotifyFreeLock {

	//添加volatile，使t2能够得到通知
	volatile List lists = new ArrayList();

	public void add(Object o) {
		lists.add(o);
	}

	public int size() {
		return lists.size();
	}
	
	public static void main(String[] args) {
		T04_NotifyFreeLock c = new T04_NotifyFreeLock();
		
		final Object lock = new Object();
		
		new Thread(() -> {
			synchronized(lock) {
				System.out.println("t2启动");
				if(c.size() != 5) {
					try {
						lock.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
				System.out.println("t2 结束");
				//通知t1继续执行
				lock.notify();
			}
			
		}, "t2").start();
		
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		}
		new Thread(() -> {
			System.out.println("t1启动");
			synchronized(lock) {
				for(int i=0; i<10; i++) {
					c.add(new Object());
					System.out.println("add " + i);
					
					if(c.size() == 5) {
						lock.notify();
						//释放锁，让t2得以执行
						try {
							lock.wait();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
					
					try {
						TimeUnit.SECONDS.sleep(1);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
		}, "t1").start();		
	}
}
```

#### 【5】程序5

小程序5使用CountDownlatch(门闩)来完成这一题的需求，分析代码的执行流程，首先我们不难看出和小程序4的写法大同小异，同样是list集合实现add和size方法，两个线程比1和t2，t1线程里是循环添加对象，t2里是实时监控，不同点在于没有了锁，采用了await()方法替换了t2线程和t1线程中的wait()方法，执行流程是创建门闩对象latch，t2线程开始启动，判断到对象不等于5，调用awat(方法阻塞t2线程，t1线程开始执行添加对象，当对象增加到5个时，打开门闩让t2继续执行。

执行结果看似没什么大问题，但是当我们把休眠1秒这段带代码，从t1线程里注释掉以后，会发现出错了，原因是在T线程里，对象增加到5个时，t2线程的门闩确实被打开了，但是t线程马上又会接着执行，之前是t会休眠1秒，给2线程执行时间，但当注释掉休眠1秒这段带代码，切就没有机会去实时监控了，所以这种方案来使用门闩是不可行的。但是如果我们非得使用门闩，还要求在对象数量为5的时候把t2线程打印出来，如何实现呢？（程序6）

```java
public class T05_CountDownLatch {

	// 添加volatile，使t2能够得到通知
	volatile List lists = new ArrayList();

	public void add(Object o) {
		lists.add(o);
	}

	public int size() {
		return lists.size();
	}

	public static void main(String[] args) {
		T05_CountDownLatch c = new T05_CountDownLatch();
		CountDownLatch latch = new CountDownLatch(1);
		new Thread(() -> {
			System.out.println("t2启动");
			if (c.size() != 5) {
				try {
					latch.await();					
					//也可以指定等待时间
					//latch.await(5000, TimeUnit.MILLISECONDS);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			System.out.println("t2 结束");
		}, "t2").start();

		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		}

		new Thread(() -> {
			System.out.println("t1启动");
			for (int i = 0; i < 10; i++) {
				c.add(new Object());
				System.out.println("add " + i);
				if (c.size() == 5) {
					// 打开门闩，让t2得以执行
					latch.countDown();
				}
				/*try {
					TimeUnit.SECONDS.sleep(1);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}*/
			}
		}, "t1").start();
	}
}
```

#### 【6】程序6（成功）

小程序6很容易理解，我们只需要在t1线程打开R2线程门闩的时候，让他再给自己加一个门闩就可以

```java
public class T05_CountDownLatch {

	// 添加volatile，使t2能够得到通知
	volatile List lists = new ArrayList();

	public void add(Object o) {
		lists.add(o);
	}

	public int size() {
		return lists.size();
	}

	public static void main(String[] args) {
		T05_CountDownLatch c = new T05_CountDownLatch();
		CountDownLatch latch = new CountDownLatch(1);
		new Thread(() -> {
			System.out.println("t2启动");
			if (c.size() != 5) {
				try {
					latch.await();					
					//也可以指定等待时间
					//latch.await(5000, TimeUnit.MILLISECONDS);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			System.out.println("t2 结束");
		}, "t2").start();

		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		}

		new Thread(() -> {
			System.out.println("t1启动");
			for (int i = 0; i < 10; i++) {
				c.add(new Object());
				System.out.println("add " + i);
				if (c.size() == 5) {
					// 打开门闩，让t2得以执行
					latch.countDown();
                    try {
					latch.await();					
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				}
				/*try {
					TimeUnit.SECONDS.sleep(1);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}*/
			}
		}, "t1").start();
	}
}
```

#### 【7】程序7

小程序7采用LockSuppor实现的，与之前同的只是改变了线程阻塞和唤醒所使用的方法，但是小程序7中其实也是有不足的，当注释掉t1线程中休眠1秒方法的时候，程序就出错了，原因是在t1线程调用unpark()方法唤醒t2线程的时候，t1线程并没有停止，就会造成t2线程无法及时的打印出提示信息，怎么解决呢？（程序8）

```java
public class T06_LockSupport {

	// 添加volatile，使t2能够得到通知
	volatile List lists = new ArrayList();

	public void add(Object o) {
		lists.add(o);
	}

	public int size() {
		return lists.size();
	}

	public static void main(String[] args) {
		T06_LockSupport c = new T06_LockSupport();

		CountDownLatch latch = new CountDownLatch(1);

		Thread t2 = new Thread(() -> {
			System.out.println("t2启动");
			if (c.size() != 5) {

				LockSupport.park();

			}
			System.out.println("t2 结束");
		}, "t2");
		t2.start();
		try {
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e1) {
			e1.printStackTrace();
		}
		new Thread(() -> {
			System.out.println("t1启动");
			for (int i = 0; i < 10; i++) {
				c.add(new Object());
				System.out.println("add " + i);

				if (c.size() == 5) {
					LockSupport.unpark(t2);
				}
				/*try {
					TimeUnit.SECONDS.sleep(1);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}*/
			}
		}, "t1").start();
	}
}
```

【8】程序8（成功）

在类的成员变量里定义了静态的线程对象tt和t2，然后在main方法里创建了t1线程和12线程，t2线程中判断了list集合中对象的数量，然后2线程阻塞，t线程开始执行添加对象，对象达到5个时，打开t2线程阻塞t1线程，至此程序结束，运行成功。


```java
public class T07_LockSupport_WithoutSleep {

	// 添加volatile，使t2能够得到通知
	volatile List lists = new ArrayList();

	public void add(Object o) {
		lists.add(o);
	}

	public int size() {
		return lists.size();
	}

	static Thread t1 = null, t2 = null;

	public static void main(String[] args) {
		T07_LockSupport_WithoutSleep c = new T07_LockSupport_WithoutSleep();

		t1 = new Thread(() -> {
			System.out.println("t1启动");
			for (int i = 0; i < 10; i++) {
				c.add(new Object());
				System.out.println("add " + i);

				if (c.size() == 5) {
					LockSupport.unpark(t2);
					LockSupport.park();
				}
			}
		}, "t1");

		t2 = new Thread(() -> {
			LockSupport.park();
			System.out.println("t2 结束");
			LockSupport.unpark(t1);
		}, "t2");

		t2.start();
		t1.start();
    }
}
```

#### 【9】程序9（成功）

通过Semaphore实现的，大体的执行流程大体是这样的，创建一个Semaphore对象，设置只能有1一个线程可以运行，首先线程1开始启动，调用acquire方法限制其他线程运行，在for循环添加了4个对象以后，调用S.releasel表示其他线程可以运行，这个时候1线程启动12线程，调用joing把CPU的控制权交给12线程，2线程打印出提示信息，并继续输出后来的对象添加信息，当然了这个方案看起来很牵强，但实现了这个效果，可以用做参考。

```java
public class T08_Semaphore {
    // 添加volatile，使t2能够得到通知
    volatile List lists = new ArrayList();

    public void add(Object o) {
        lists.add(o);
    }

    public int size() {
        return lists.size();
    }

    static Thread t1 = null, t2 = null;

    public static void main(String[] args) {
        T08_Semaphore c = new T08_Semaphore();
        Semaphore s = new Semaphore(1);

        t1 = new Thread(() -> {
            try {
                s.acquire();
                for (int i = 0; i < 5; i++) {
                    c.add(new Object());
                    System.out.println("add " + i);
                }
                s.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            try {
                t2.start();
                t2.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            try {
                s.acquire();
                for (int i = 5; i < 10; i++) {
                    System.out.println(i);
                }
                s.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }, "t1");

        t2 = new Thread(() -> {
            try {
                s.acquire();
                System.out.println("t2 结束");
                s.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "t2");
        t1.start();
    }
}
```

面试题1总结：面试题1中的9个小程序9种方案，5种技术分别是volatile、wait()和notify()、Semaphore、CountDownLatch、LockSupport，其中wait和notity，这个方案建议牢牢掌握，其它的可以用作巩固技术。

### 2、面试题2：

写一个固定容量的同步容器，拥有put和get方法，以及getCount方法，能够支持2个生产者线程以及10个消费者线程的阻塞调用。

#### 【1】程序1

程序的执行流程：创建了一个LinkedList集合用于保存“馒头”，定义了MAX变量来限制馒头的总数，定义了count变量用来判断生产了几个“馒头”和消费了几个“馒头”，在put方法中，首先判断 LinkedList集合中“馒头”是否是MAX变量的值，如果是启动所有消费者线程，反之开始生产“馒头”，在get方法中，首先判断是否还有“馒头”，也就是MAX的值是否为0，如果为0通知所有生产者线程开始生产“馒头”，反之不为0“馒头”数就继续减少，需要注意的点是，我们为什么要加synchronized，因为我们++count我们生产了3个“馒头”，当还没来得及加的时候，count值为2的时候，另外一个线程读到的值很可能是2，并不是3，所以不加锁就会出问题，main方法中通过for循环分别创建了2个生产者线程生产分别生产25“馒头”，也就是50个馒头，10个消费者线程每个消费者消费5个“馒头”，也就是50个“馒头”，首先启动消费者线程，然后启动生产者线程。

小程序分析：为什么用while而不是用if？因为当LinkedList集合中”“馒头”数等于最大值的时候，if在判断了集合的大小等于MAX的时候，调用了wait()方法以后，它不会再去判断一次，方法会继续往下运行，假如在你wait(以后，另一个方法又添加了一个”馒头”，你没有再次判断，就又添加了一次，造成数据错误，就会出问题，因此必须用while。我们用的是notifyAII()来唤醒线程的，notifyAlI方法会叫醒等待队列的所有方法，那么我们都知道，用了锁以后就只有一个线程在运行，其他线程都得wait()，不管你有多少个线程，这个时候被叫醒的线程有消费者线程和生产者线程，比如说我们是生产者线程，生产满了之后叫醒消费者线程，可它同样的也会叫醒另一个生产者线程，假如这个生产者线程拿到了刚才第一个生产者释放的这把锁，又wait一遍，wait完以后，又叫醒全部的线程，然后又开始争枪这把锁，其实从这个意义上来讲，生产者的线程wait的是没有必要去叫醒别的生产者的，我们能不能只叫醒消费者线程，就是生产者线程只叫醒消费者线程，消费者线程只负责叫醒生产者线程，如果想达到这样一个程度的话用另外一个小程序。（程序2）


```java
public class MyContainer1<T> {
	final private LinkedList<T> lists = new LinkedList<>();
	final private int MAX = 10; //最多10个元素
	private int count = 0;

	public synchronized void put(T t) {
		while(lists.size() == MAX) { //想想为什么用while而不是用if？
			try {
				this.wait(); //effective java
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}		
		lists.add(t);
		++count;
		this.notifyAll(); //通知消费者线程进行消费
	}
	
	public synchronized T get() {
		T t = null;
		while(lists.size() == 0) {
			try {
				this.wait();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		t = lists.removeFirst();
		count --;
		this.notifyAll(); //通知生产者进行生产
		return t;
	}
	
	public static void main(String[] args) {
		MyContainer1<String> c = new MyContainer1<>();
		//启动消费者线程
		for(int i=0; i<10; i++) {
			new Thread(()->{
				for(int j=0; j<5; j++) System.out.println(c.get());
			}, "c" + i).start();
		}
		try {
			TimeUnit.SECONDS.sleep(2);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		//启动生产者线程
		for(int i=0; i<2; i++) {
			new Thread(()->{
				for(int j=0; j<25; j++) c.put(Thread.currentThread().getName() + " " + j);
			}, "p" + i).start();
		}
	}
}
```

#### 【2】程序2

使用了ReentrantLock，它与synchronized最大区别其实在这个面试题里已经体现出来了，ReentrantLock它可以有两种Condition条件，在put方法里是生产者线程，一但MAX达到峰值的时候是producer.await()，最后是consumer.signalAII

就是说在producer的情况下阻塞的，叫醒的时候只叫醒consumer，在get();方法里是我们的消费者线程，一但集合的size空了，我是consumer.await()，然后我只叫醒producer，这就是ReentrantLock的含义，它能够精确的指定哪些线程被叫醒，注意是哪些不是哪个

Lock和Condition的本质是在synchronized里调用wait和notify的时候，它只有一个等待队列，如果lock.newnewCondition的时候，就变成了多个等待队列，Condition的本质就是等待队列个数，以前只有一个等待队列，现在我new了两个Condition，一个叫producer一个等待队列出来了，另一个叫consumer第二个的等待队列出来了，当我们使用producer.await 的时候，指的是当前线程进入producer的等待队列，使用producer.signalAll指的是唤醒producer这个等待队列的线程，consumner也是如此，所以小程序就很容易理解了，我在生产者线程里叫醒consumer等待队列的线程也就是消费者线程，在消费者线程里叫醒producer待队列的线程也就是生产者线程。

```java
public class MyContainer2<T> {
	final private LinkedList<T> lists = new LinkedList<>();
	final private int MAX = 10; //最多10个元素
	private int count = 0;
	
	private Lock lock = new ReentrantLock();
	private Condition producer = lock.newCondition();
	private Condition consumer = lock.newCondition();
	
	public void put(T t) {
		try {
			lock.lock();
			while(lists.size() == MAX) { //想想为什么用while而不是用if？
				producer.await();
			}
			
			lists.add(t);
			++count;
			consumer.signalAll(); //通知消费者线程进行消费
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public T get() {
		T t = null;
		try {
			lock.lock();
			while(lists.size() == 0) {
				consumer.await();
			}
			t = lists.removeFirst();
			count --;
			producer.signalAll(); //通知生产者进行生产
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
		return t;
	}
	
	public static void main(String[] args) {
		MyContainer2<String> c = new MyContainer2<>();
		//启动消费者线程
		for(int i=0; i<10; i++) {
			new Thread(()->{
				for(int j=0; j<5; j++) System.out.println(c.get());
			}, "c" + i).start();
		}
		
		try {
			TimeUnit.SECONDS.sleep(2);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		//启动生产者线程
		for(int i=0; i<2; i++) {
			new Thread(()->{
				for(int j=0; j<25; j++) c.put(Thread.currentThread().getName() + " " + j);
			}, "p" + i).start();
		}
	}
}
```

### 3、阅读源码的原则：

1. 跑不起来的不读
   - 跑不起来的源码不要读，看也看不懂，很难看懂，事倍功半，读起来还费劲，什么时候这个源码必须得跑起来，跑起来有什么好处就是，你可以用debug一条线跟进去，举个例子，比如ReentrantLock的lock0)方法，没有跑起来的时候，静态的来读源码你会怎么读，按ctrl鼠标单击lock()方法，进入这个方法，会看到这个方法调用了别的方法，你又会按ctrl鼠标单击进入它调用的这个方法，一层层往下，你会发现没法读了，所以如果这个东西能跑起来就不一样了，你会发现与之前鼠标单击跟进的结果不一样了，原因是因为多态的实现，如果一个方法有很多子类的实现，但是你不知道跟着这条线，它会去实现那个方法，所以你就得全部看一遍。
2. 解决问题就好一目的性
   - 在实际中解决问题就好，读源码一方面可以解决问题，一方面可以应对面试，什么意思呢，如果你接手了一个别人改过6手的代码，现在你的老板说这个代码有些问题，你往里边加一些功能或者修改一些bug，你解决问题就好，你不要从头到尾去读去改这个代码，你读你能累死你，目的性要强，解决问题就好。
3. 一条线索到底
   - 读源码的时候要一条线索到底，不要只读表面，我们知道一个程序跑起来以后，可能这个程序非常大，一个main方法有很多的put、get、size各种各样其他的方法，每一个方法你调进去，这个方法很有可能又去调别的方法，你不要每个方法先看一遍表面，让后再去里边找，要一条线索到底，就读一个方法，由浅到深看一遍。
4. 无关细节略过
   - 有那些边界性的东西，在你读第一边的时候，没必要的时候，你可以先把它略过。

## 五、AQS源码阅读与强软弱虚4种引用及ThreadLocal原理与源码

### 1、AQS源码——画图及概念

<img src="_media/basis/多线程与高并发/aqs源码泳道图.png"/>

上图叫泳道图，是UML图的一种叫甬道，这个图诠释了哪个类里的哪个方法又去调用了哪个类里的的哪个方法，从泳道图可以看出，ReentrantLock调用它 locK() 方法的时候，它会调用 Sync 的 acquire(1) 方法

NonfairSync 的父类是 Sync，因为你看到NonfairSync子类里方法的时候，它有可能用到父类的方法，所以要去父类里读才可以，所以AQS是所有锁的核心

lock -> Nonfairsync 的 acquire(1) (->父类sync -> 父类AQS) -> AQS 的 acquire(1) -> NonfairSync 重写的 AQS里的tryAcquire(1)        ->nonfairTryAcquire(acquires)

继续看nonfairTryAcquire(acquires)方法实现，开始它得到了当前线程，这时出现了一个方法getState()，跟进这个方法会发现，这个方法又进到了AQS类里，这个getStatel方法返回了一个state，这个state就是一个volatile修饰的int类型的数，这时就牵扯到AQS的结构了

### 2、AQS源码——解析

<img src="_media/basis/多线程与高并发/AQS.png"/>

AQS队列又可以称为CLH队列，AQS的核心就是这个state，这个state所代表的意思随子类来定，目前举例是ReentrantLock，刚才state的值是0，当你获得了之后它会变成1，就表示当前线程得到了这把锁，什么时候释放完了，state又会从1变回0，说明当前线程释放了这把锁，所以这个state 0和1就代表了加锁和解锁

这个state值的下面跟着一个队列，这个队列是AQS自己内部所维护的队列，这个队列里边每一个所维护的都是node节点，他在AQS这个类里属于AQS的内部类，在这个node里最重要的一项是他里面保留了一个Thread一个线程，所以这个队列是个线程队列，而且还有两个prev和next分别是前置节点和后置节点

所以AQS里边的队列是一个一个的node，node里装的是线程Thread，这个node它可以指向前面的这一个，也可以指向后面的这一个，所以叫双向链表，所以AQS的核心是一个state，以及监控这个state的双向链表，每个列表里面有个节点，这个节点里边装的是线程，那么那个线程得到了state这把锁，那个线程要等待，都要进入这个队列里边，当我们其中一个node得到了state这把锁，就说明这个node里得线程持有这把锁，所以当我们acquire(1）上来以后看到这个state的值是0，那我就直接拿到state这个把锁，现在是非公平上来就抢，抢不着就进队列里acquireQueued()

怎么是抢到呢？先得到当前线程，然后获取state的值，如果state的值等于0，用compareAndSetState(0,acquire)方法尝试把state的值改为1，假如改成了setExclusiveOwnerThread(把当前线程设置为独占statie这个把锁的状态，说明我已经得到这个把锁，而且这个把锁是互斥的，我得到以后，别人是得不到的，因为别人再来的时候这个state的值已经变成1了，如果说当前线程已经是独占state这把锁了，就往后加个1就表示可重入了。

什么叫模板方法？

- 模板方法就是父类里有一个方法，就是这个tryAcquire(1)，它调用了子类的一些方法，这些子类的方法没有实现，我调用自己的方法事先写好，但是由于这些方法就是给子类用来去重写实现的，所以我就像一个模板一样，我要打造一辆汽车，我要造地盘，造发动机，造车身，最后组装，造地盘这件事子类去实现，哪个车厂去实现是哪个车厂的事，奥迪是奥迪的事，奥托是奥托的事，车身也是发动机也是，最后反正这个流程是一样的，就像一个模板一样，我的模板是固定的，里边的方法是由具体的子类去实现的，当我们开始造车的时候，就会调用具体子类去实现的函数，所以叫为钩子函数，我勾着谁呢？勾着子类的实现，这个就叫模板方法。

### 3、AQS源码——解读

首先在通过在lock()方法处打断点，debug运行，观察AQS源码

```java
public class TestReentrantLock {
    private static volatile int i = 0;
    public static void main (String[] args) {
        ReentranLock lock = new ReentrantLock();
        lock.lock();
        i++;
        lock.unlock();
    }
}
```

在lock()方法里面，读到了他调用了sync.acquite(1)

<img src="_media/basis/多线程与高并发/aqs源码第二步.png"/>

再跟进到acquire(1)里，可以看到acquire(1)里又调用了子类实现的 tryAcquire(arg)，

<img src="_media/basis/多线程与高并发/aqs源码第三步.png"/>

跟进到tryAcquire(arg)里又调用了nonfairTrytAcquire(acquires)

<img src="_media/basis/多线程与高并发/aqs源码第四步.png"/>

nonfairTrytAcquire(acquires) 读进去会发现里面就调用到了state这个值，首先拿到当前线程，拿到state的值，然后进行i判断，如果state的值为0，说明没人上锁，就给自己上锁，当前线程就拿到这把锁，拿到这个把锁的操作用到了CAS(compareAndSetState)的操作，从0让他变成1，state的值设置为1以后，设置当前线程是独一无二的拥有这把锁的线程，否则如果当前线程已经占有这把锁了，就在原来的基础上加1就可以了，视为重入锁

<img src="_media/basis/多线程与高并发/aqs源码第五步.png"/>

我们跟进到tryAcquirel(arg)是拿到了这把锁以后的操作，如果拿不到锁，实际上是调用了acquireQueued()方法，acquireQueued()方法里又调用了addWaiter(Node.EXCLUSWE)然后后面写-—个arg(数值1)，

方法结构是这样的acquireQueued(addWaiter(Node.EXCLUSIVE),arg)，如果得到这把锁了，后面的acquireQueued是不用运行的，如果没有得到这把锁，后面的acquireQueued()才需要运行，进入到队列中排队，排队的时候需要传递两个参数，第一个参数是某个方法的返回值addWaiter(Node.EXCLUSIVE)，意思就是把当线程作为排他形式扔到队列里边。

<img src="_media/basis/多线程与高并发/aqs源码第六步.png"/>

我们来说一下这个adbWaiter 方法，这个方法意思是说在添加等待者的时候，使用的是什么类型，如果这个线程是Node,.EXCLUSVE那么就是排他锁，Node.SHARED就是共享锁，首先是获得当前要加进等待者队列的线程的节点，然后是一个死循环，其中做的事情就是用CAS方式将新加入的节点设置成最后一个

<img src="_media/basis/多线程与高并发/aqs源码addWaiter.png"/>

这个增加线程节点操作，如果没有成功，那么就会不断的试，一直试到我们的这个node节点被加到线程队列末端为止，意思就是说，其它的节点也加到线程队列末端了，我无非就是等着你其它的线程都加到末端了，我加最后一个，不管怎么样我都要加到线程末端去为止。

源码读这里我们可以总结得出，AQS(AbstractQueuedSynchronizer的核心就是用CAS(compareAndSet)去操作head和tail，就是说用CAS操作代替了锁整条双向链表的操作

通过AQS是如何设置链表尾巴的来理解AQS为什么效率这么高

### 4、AQS源码——效率高的原因

假如要往一个链表上添加尾巴，尤其是好多线程都要往链表上添加尾巴，用普通的方法，第一点要加锁，因为多线程，你要保证线程安全，一般的情况下，我们会锁定整个链表(Sync)，我们的新线程来了以后，要加到尾巴上，这样很正常，但是我们锁定整个链表的话，锁的太多太大了，现在呢它用的并不是锁定整个链表的方法，而是只观测tail这一个节点就可以了，怎么做到的呢？

compareAndAetTail(oldTailnode)，中oldTai是它的预期值，假如说我们想把当前线程设置为整个链表尾巴的过程中，另外一个线程来了，它插入了一个节点，那么Node oldTailtail的整个oldTai就不等于整个新的Tail，那么既然不等于了，说明中间有线程被其它线程打断了，那如果说确实还是等于原来的oldTail，这个时候就说明没有线程被打断，那我们就接着设置尾巴，只要设置成功了，compareAndAetTal(oidTailinode)方法中的参数node就做为新的Tail了，所以用了CAS操作就不需要把原来的整个链表上锁，这也是AQS在效率上比较高的核心。

### 5、AQS源码——为什么是双向链表

其实你要添加一个线程节点的时候，需要看一下前面这个节点的状态，如果前面的节点是持有线程的过程中，这个时候你就得在后面等着，如果说前面这个节点已经取消掉了，那你就应该越过这个节点，不去考虑它的状态，所以你需要看前面节点状态的时候，就必须是双向的。

接下来解读acquireQueued()这个方法，这个方法的意思是，在队列里尝试去获得锁，在队列里排队获得锁，那么它是怎么做到的呢？

我们先大致走一遍这个方法，首先在for循环里获得了Node节点的前置节点，然后判断如果前置节点是头节点，并且调用tryAcquire(arg) 方法尝试一下去得到这把锁，获得了头节点以后，你设置的节点就是第二个，你这个节点要去和前置节点争这把锁，这个时候前置节点释放了，如果你设置的节点拿到了这把锁，拿到以后你设置的当前节点就被设置为前置节点，如果没有拿到这把锁，当前节点就会阻塞等着前置节点叫醒你，所以它上来之后是竞争，如果你是最后节点，你就下别说了，你就老老实实等着，如果你的前面已经是头节点了，说明快轮到我了，那我就跑一下，试试看能不能拿到这把锁，说不定前置节点这会儿已经释放这把锁了，如果拿不着阻塞，等着前置节点释放这把锁以后，叫醒队列里的线程。

- <img src="_media/basis/多线程与高并发/aqs源码acquireQueued.png"/>

### 6、AQS源码——VarHandle

在addWaiter 这个方法里边有一个node.setPrevRelaved(oldTail)，这个方法的意思是把当前节点的前置节点写成tail，进入这个方法你会看到PREV.set(this,p)，PREV有这么一个东西叫VarHandle，

这个VarHandle是什么呢？这个东西实在JDK1.9之后才有的，Var叫变量(variable)，Handle叫句柄，打个比方，比如我们写了一句话叫Object o= new Object()，我们new了一个Object，这个时候内存里有一个小的引用“O”，指向一段大的内存这个内存里是new的那个Object对象，那么这个VarHandle指什么呢？指的是这个“引用”，我们思考一下，如果VarHandle代表“引用”，那么VarHandle所代表的这个值PREV也指这个“引用”。这个时候我们会生出一个疑问，本来已经有一个“O”指向这个Object对象了，为什么还要用另外一个引用也指向这个对象，这是为什么?

<img src="_media/basis/多线程与高并发/aqs源码VarHandle.png"/>

见以下小程序：VarHandle除了可以完成普通属性的原子操作，还可以完成原子性的线程安全的操作，这也是VarHandle的含义。

在IDK1.9之前要操作类里边的成员变量的属性，只能通过反射完成，用反射和用VarHandle的区别在于，VarHandle的效率要高的多，反射每次用之前要检查，VarHandle不需要，VarHandle可以理解为直接操纵二进制码，所以VarHandle反射高的多

```java
public class T01_HelloVarHandle {

    int x = 8;

    private static VarHandle handle;

    static {
        try {
            handle = MethodHandles.lookup().findVarHandle(T01_HelloVarHandle.class, "x", int.class);
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        T01_HelloVarHandle t = new T01_HelloVarHandle();

        //plain read / write
        System.out.println((int)handle.get(t));
        handle.set(t,9);
        System.out.println(t.x);

        handle.compareAndSet(t, 9, 10);
        System.out.println(t.x);

        handle.getAndAdd(t, 10);
        System.out.println(t.x);

    }
}
```

### 7、AQS源码——JDK1.8

JDK1.8中与上述的区别1：线程过来后首先会进行非公平的抢占，如果抢占失败才会进入到上述的流程，再一次尝试获取，获取失败则进入队列

<img src="_media/basis/多线程与高并发/aqs源码java8实现1.png"/>

addWaiter 方法中并未使用死循环

<img src="_media/basis/多线程与高并发/aqs源码java8实现2.png"/>

VarHandl方法只在1.9中才有，这里就是使用node.prev = pred 直接设置前置节点

<img src="_media/basis/多线程与高并发/aqs源码java8实现3.png"/>

### 8、ThreadLocal

Threadlocal的含义是线程本地

看以下程序，定义Person类，类里面定义了一个String类型的变量name，name的值为"zhangsan”，在ThreadLocali这个类里，我们实例化了这个Person类，然后在main方法里我们创建了两个线程，第一个线程打印了p.name，第二个线程把p.name的值改为了 “lisi”，两个线程访问了同一个对象，最后的结果是打印出了“lisr而不是“zhangsan”，因为原来的值虽然是“zhangsan”，但是有一个线程1秒之后把它变成lisr了，另一个线程两秒钟之后才打印出来，那它一定是变成”liis”了。

```java
public class ThreadLocal1 {
	volatile static Person p = new Person();
	
	public static void main(String[] args) {
				
		new Thread(()->{
			try {
				TimeUnit.SECONDS.sleep(2);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			
			System.out.println(p.name);
		}).start();
		
		new Thread(()->{
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			p.name = "lisi";
		}).start();
	}
}

class Person {
	String name = "zhangsan";
}
```

但是有的时候我们想让这个对象每个线程里都做到自己独有的一份，见以下程序，这个小程序中，用到了ThreadLocal，main方法中第二个线程在1秒终之后往t对象中设置了一个Person对象，虽然我们访问的仍然是这个t对象，第一个线程在两秒钟之后回去get获取t对象里面的值，第二个线程是1秒钟之后往t对象里set了一个值，从多线程普通的角度来讲，既然我一个线程往里边set()了一个值，另外一个线程去get这个值的时候应该是能get到才对，但是很不幸的是，我们1秒钟的时候set了一个值，两秒钟的时候去拿这个值是拿不到的，这个小程序证明了这一点，这是为什么呢？原因是如果我们用ThreadLocal的时候，里边设置的这个值是线程独有的，线程独有的是什么意思呢？就是说这个线程里用到这个ThreadLocal的时候，只有自己去往里设置，设置的是只有自己线程里才能访问到的Person，而另外一个线程要访问的时候，设置也是自己线程才能访问到的Person，这就是ThreadLocal的含义

```java
public class ThreadLocal2 {
    static ThreadLocal<Person> tl = new ThreadLocal<>();
    public static void main(String[] args) {

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(tl.get());
        }).start();

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            tl.set(new Person());
            System.out.println(tl.get().name);
        }).start();
    }

    static class Person {
        String name = "zhangsan";
    }
}
```

### 9、ThreadLocal源码

ThreadLocal源码中的set方法，ThreadLocal设置值的时候，首先拿到当前线程，这时会发现，这个set方法里多了一个容器ThreadLocalMap，这个容器是一个map，是一个key、value对，这个值是设置到了map里面，而且这个map中，key设置的是this，value设置的是我们想要的那个值，这个this就是当前对象ThreadLocal，value就是Person类，这么理解就行了，如果map不等于空的情况下就设置进去就行了，如果等于空，就创建一个map

<img src="_media/basis/多线程与高并发/ThreadLocal源码set.png"/>

ThreadLocalMap map=getMap(t)，点击到了getMap这个方法看到，它的返回值是 t.threadLocals

<img src="_media/basis/多线程与高并发/ThreadLocal源码getMap.png"/>

进入t.threadLocals，发现ThreadLocalMap是在Thread这个类里，所以说这个map是在Thred类里的

<img src="_media/basis/多线程与高并发/ThreadLocal源码map.png"/>

set：Thread.currentThread.map(ThreadLocal, person) -- Thread 类里面的当前线程的 map

所以Person类被set到了当前线程里的某一个map里面去了，所以t1线程set了一个Person对象到自己的map里，t2线程访问t1的map就访问不到

为什么要用ThreadLocal? 我们根据Spirng的声明式事务来解析，为什么要用ThreadLocal
- 声明式事务一般来讲我们是要通过数据库的，但是我们知道Spring结合Mybatis，我们是可以把整个事务写在配置文件中的，而这个配置文件里的事务，它实际上是管理了一系列的方法，方法1、方法2、方法3.....而这些方法里面可能都写了去配置文件里拿到数据库连接Connection，然后声明式事务可以把这几个方法合在一起，视为一个完整的事务，如果说在这些方法里，每一个方法在连接池中拿的连接，都不是同一个对象，就不能形成一个完整的事务的，那么怎么保证这么多Connection之间保证是同一个Connection呢？
- 把这个Connection放到这个线程的本地对象里ThreadLocal里面，以后再拿的时候，实际上我是从ThreadLocal里拿的，第1个方法拿的时候就把Connection放到ThreadLocal里面，后面的方法要拿的时候，从ThreadLocal里直接拿，不从线程池拿。

### 10、Java中4种引用 —— 强引用

首先明白什么是一个引用？

- Object o = new Object() 这就是一个引用了，一个变量指向new出来的对象，这就叫以个引用。

见以下程序，理解强引用：
- 定义一个M类，类中重写一个方法叫finalize()，这个方法是已经被废弃的方法，主要想说明一下在垃圾回收的过程中，各种引用它不同的表现，垃圾回收的时候，它是会调用finalize()这个方法的

- 当我们new出来一个象，在java语言里是不需要手动回收的，C和C++是需要的，在这种情况下，java的垃圾回收机制会自动的帮你回收这个对象，但是它回收对象的时候它会调用finalizeh()这个方法，我们重写这个方法之后我们能观察出来，它什么时候被垃圾回收了，什么时候被调用了，我在这里重写这个方法的含义是为了以后面试的时候方便你们造火箭，让你们观察结果用的，并不说以后在什么情况下需要重写这个方法，这个方法永远都不需要重写，而且也不应该被重写。

  ```java
  public class M {
      @Override
      pritected void finallize() throws Throwable {
          System.out.println("finalize");
      }
  }
  ```

普通引用NormalReference，也就是默认的引用，意思就是只要有一个引用指向这个对象，那么垃圾回收器就不会回收它，也叫强引用，只有没有引用指向创建的那个对象的时候才会回收。

见以下程序，new了一个m出来，然后调用了System.gc()，显式的来调用一下垃圾回收，让垃圾回收尝试一下，看能不能回收这个m，需要注意的是，要在最后阻塞住当前线程，为什么?因为System.gc()是跑在别的线程里边的，如果main线程直接退出了，那整个程序就退出了，那gc不gc就没有什么意义了，所以你要阻塞当前线程，在这里调用了System.in.read()阻塞方法，它没有什么含义，只是阻塞当前线程的意思。

阻塞当前线程就是让当前整个程序不会停止，程序运行起来你会发现，程序永远不会输出，为什么呢？我们想一下，这个M是有一个小引用m指向它的，那有引用指向它，它肯定不是垃圾，不是垃圾的话一定不会被回收。

```java
public class T01_NormalReference {
    public static void main(String[] args) throws IOException {
        M m = new M();
        m = null; // todo 当m不再指向M对象时，M对象就会被回收
        System.gc();
        System.in.read();
    }
}
```

### 11、Java中4种引用 —— 软引用

软引用的含义，当有一个对象(字节数组)被一个软引用所指向的时候，只有系统内存不够用的时候，才会回收它(字节数组)

我们来分析SoftReference<byte[]>m = new SoftReference<>(new byte[1024*1024*10])，首先栈内存里有一个m，指向堆内存里的SoftReference软引用对象，注意这个软引用对象里边又有一个对象，可以想象一下软引用里边一个引用指向了一个10MB大小的字节数组，然后通过m.get()拿到这个字节数组然后输出，它会输出HashCode值，然后调用System.gc()，让垃圾回收去运行，那么如果说，这个时候如果gc运行完以后，字节数组被回收了，你再次打印m.get()的时候，它应该是个null值了，然后在后面又分配了一个15MB大小的数组，最后再打印m.get()，注意还是第一个m的get，如果这个时候被回收了它应该打印null值，没有被回收的话，应该打印一个HashCode值

运行以下程序，设置一下堆内存最大为20MB，这时运行程序会发现，第三次调用m.get()输出的时候，输出的值为null，我们来分析一下，第一次我们的堆内存这个时候最多只能放20MB，第一次创建字节数组的时候分配了10MB，这个时候堆内存是能分配下的，这个时候我调用了gc来做回收是无法回收的，因为堆内存够用，第二次创建字节数组的时候分配了15MB，这个时候对内存的内存还够15MB吗？肯定是不够的，不够了怎么办？清理，清理的时候既然内存不够用，就会把你这个软引用给干掉，然后15MB内存分配进去，所以这个时候你再去get第一个字节数组的时候它是一个nul值，这是就是软引用的含义

```java
public class T02_SoftReference {
    // 软引用非常适合缓存使用
    public static void main(String[] args) throws IOException {
        SoftReference<byte[]> m = new SoftReference<>(new byte[1024*1024*10]);
        System.out.println(m.get());
        System.gc();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(m.get());
        byte[] b = new byte[1024*1024*15];
        System.out.println(m.get());
    }
}
```

软引用的使用场景：做缓存用

举个例子：从内存里边读一个大图片，然你用完了之后就没什么用了，你可以放在内存里边缓存在那里，要用的时候直接从内存里边拿，但是由于这个大图片占的空间比较大，如果不用的话，那别人也要用这块空间，那就把它干掉，这个时候就用到了软引用

再举个例子，从数据库里读一大堆的数据出来，这个数据有可能比如说你按一下back，我还可以访问到这些数据，如果内存里边有的话，我就不用从数据库里拿了，这个时候我也可以用软应用，需要新的空间你可以把我干掉，没问题我下次去数据库取就行了，但是新空间还够用的时候，我下次就不用从数据库取，直接从内存里拿就行了


### 12、Java中4种引用 —— 弱引用

弱引用的意思是，只要遭遇到gc就会回收。

见以下程序，WeakReference m =new WeakReference<>(new M())，new了一个对象m指向的是一个弱引用又指向了new出来的另外一个M对象，然后通过m.get()来打印这个M对象，接下来gc调用垃圾回收，如果他它没有被回收，你接下来get还能拿到，反之则不能

```java
public class T03_WeakReference {
    public static void main(String[] args) {
        WeakReference<M> m = new WeakReference<>(new M());
        System.out.println(m.get());
        System.gc();
        System.out.println(m.get());
        ThreadLocal<M> tl = new ThreadLocal<>();
        tl.set(new M());
        tl.remove();
    }
}
```

运行后，第一次打印出来了，第二次打印之前调用了gc，所以第二次打印出了null值，那么把它创建出来有什么用呢？

这个东西作用在于，如果有另外一个强引用指向了这个弱引用之后，只要这个强引用消失掉，这个弱引用就应该被回收，

这个东西一般用在容器里，比如ThreadLocal

之后创建了一个对象叫 tl，这个 tl 对象的引用指向ThreadLocal对象，ThreadLocal对象里又指向了一个M对象

![ThreadLocal](_media/basis/多线程与高并发/ThreadLocal.png"/>

<img src="_media/basis/多线程与高并发/ThreadLocal2.png"/>

ThreadLocal内部执行了怎样的操作？参考上图
- 首先主线程中有一个线程的局部变量tl，tl指向new出来的ThreadLocal对象，这是一个强引用；
- 之后向ThreadLocal中放了一个对象，实际上是放到了当前线程的threadLocals变量里面，这个变量指向了一个map，map.set(this, value)，this是当前线程，value是M对象；
- 当向map中set的时候，放进去了一个Entry，这个Entry是从弱引用WeakReference继承出来的；
- 这个弱引用中装的是ThreadLocal对象，在Entry构造时调用了super(k)，这个k就指的是THreadLocal对象，那WeakReference就相当于 new WeakReference key。
- tl强引用指向ThreadLocal对象，是一个局部变量，当方法结束就会回收，如果此时ThreadLocal对象还被一个强引用的key指向的话，那么ThreadLocal对象将不会被回收；
- 由于很多线程是长期存在的，比如服务器线程会不间断运行，那么tl以及tl中的map和key都会长期存在，那么这个ThreadLocal对象就永远不会被回收，会产生内存泄漏的情况，所以使用弱引用；
- 但是当tl消失时，key的指向也被回收了，threadLocals中的map是一直存在的，相当于key是null，value指向上图中10M的字节码，一直访问不到，当map越积攒越多，就会出现内存泄漏，所以要remove掉。见上述程序

### 13、Java中4种引用 —— 虚引用

- 虚引用用来管理堆外内存的，虚引用的构造方法至少都是两个参数的，第二个参数还必须是一个队列，这个虚引用基本没用，就是说不是给你用的，是给写JVM(虚拟机)的人用的
- 见以下程序，程序里创建了一个List集合用于模拟内存溢出，还创建了一个ReferenceQueue(引用队列)，在main方法里创建一个phantomReference对象指向了一个new出来的PhantomReference对象，这个对像里面可以访问两个内容，第一个内容是它又通过一个虚引用指向了我们new出来的一个M对象，第二个内容它关联了一个Queue(队列)，这个时候一但虚引用被回收，这个虚引用会装到这个队列里，就是垃圾回收的时候，一但把这个虚引用给回收的时候，会装到这个队列里，让你接收到一个通知，什么时候你检测到这个队列里面如果有一个引用存在了，说明这个虚引用被回收了，这个虚引用指向的任何一个对象，垃圾回收上来就把这个M对象给干掉，当M对象被干掉的时候，你会收到一个通知，通知你的方式就是往这个Queue(队列)里放进一个值
- 在小程序启动前先设置好了堆内存的最大值，然后看第一个线程启动以后，它会不停的往List集合里分配对象，什么时候内存占满了，触发垃圾回收的时候，另外一个线程就不断的监测这个队列里边的变动，如果有就说明这个虚引用被放进去了，就说明被回收了
- 在第一个线程启动后会看到，无论怎么get这个phantomReference里面的值，它输出的都是空值，虚引用和弱引用的区别就在于，弱引用里边有值你get的时候还是get的到的，但是虚引用你get里边的值你是get不到的
- <img src="_media/basis/多线程与高并发/虚引用程序.png" alt="image-20210104115426634" style="zoom:67%;" />
- 作用是什么？
  - 有一种情况，NIO里边有一个比较新的DirectByteBulffer(直接内存)，直接内存是不被JVM(虚拟机)直接管理的内存，是被操作系统管理，又叫做堆外内存，这个DirectByteBuffer是可以指向堆外内存的。
  - 如果这个DirectByteBuffer设为null，垃圾回收器不能回收DirectByteBuffer，它指向内存都没在堆里，无法回收，所以当这个DirectByteBuffer被设为nul的时候，可以用虚引用，当我们检测到这个虚引用被垃圾回收器回收的时候，你做出相应处理去回收堆外内存
- <img src="_media/basis/多线程与高并发/虚引用.png" alt="image-20201223225622557" style="zoom:67%;" />
- 如果自己写了一个Nety，然后再Nety里边分配内存的时候，用的是堆外内存，那么堆外内存又想做到自动的垃圾回收，不能让使用API的人去回收？所以你这个时候可以检测虚引用里的Queue，什么时候Queue检测到DirectByteBuffen(直接内存被回收了，这个时候你就去清理堆外内存，堆外内存怎么回收呢？你如果是C和C++语言写的虚拟机的话，当然是del和free这个两个函数，它们也是C和C++提供的，java里面现在也提供了，堆外内存回收，这个回收的类叫Unsafe，这个类在IDK1.8的时候可以用java的反射机制来用它，但是IDK1.9以后它被加到包里了，普通人是用不了的，但JUC的一些底层有很多都用到了这个类，这个Unsafe类里面有两个方法，allocateMemory方法直接分配内存也就是分配堆外内存，freeMemory方法回收内存也就是手动回收内存，这和C、C++里边一样你直接分配内存，必须得手动回收

## 六、并发容器

### 1、组织结构

- <img src="_media/basis/多线程与高并发/容器脑图.png"/>
- 第一大类是Collection叫集合
  - 意思是不管这个容器是什么结构，你可以把一个元素一个元素的往里面扔；
- 第二大类是Map
  - 是一对一对的往里扔。其实Map来说可以看出是Collection一个特殊的变种，你可以把一对对象看成一个entry对象，所以这也是一个整个对象。容器就是装一个一个对象的，这么一些个集合。
- 严格来讲数组也属于容器。从数据结构角度来讲在物理上的这种存储的数据结构其实只有两种，一种是连续存储的数组Array，另一种就是非连续存储的一个指向另外一个的链表。在逻辑结构那就非常非常多了。
- 容器从JAVA接口区分的比较明确，两大接口：Map是一对一对的。Collection是一个一个的，在它里面又分三大类。Queue是后来加入的接口，而这个接口专门就是为高并发准备的，所以这个接口对高并发来说才是最重要的。
- 面试时可以说：容器分两大类Collection、Map，Collection又分三大类List、Set、Queue队列，队列就是一对一队的，往这个队列里取数据的时候它和这个List、Set都不一样。大家知道List取的时候如果是Array的话还可以取到其中一个的。Set主要和其他的区别就是中间是唯一的，不会有重复元素，这个它最主要的区别。
- Queue就是一个队列，有进有出
  - 那么在这个基础之上它实现了很多的多线程的访问方法（比如说put阻塞式的放、take阻塞式的取），这个是在其他的List、Set里面都是没有的。
  - 队列最主要的原因是为了实现任务的装载的这种取和装这里面最重要的就是是叫做阻塞队列，它的实现的初衷就是为了线程池、高井发做准备的。
  - 最开始java1.0容器里只有两个，
    - 第一个叫Vector可以单独的往里扔，还有一个是Hashtable是可以一对一对往里扔的。Vector相对于实现了List接口，Hashtable实现了Map接口。
    - 但是这个两个容器在1.0设计的时候稍微有点问题，这两个容器设计成了所有方法默认都是加synchronized的，这是它最早设计不太合理的地方。多数的时候我们多数的程序只有一个线程在工作，所以在这种情况下你完全是没有必要加synchronized ，因此最开始的时候设计的性能比较差。
    - 后来它意识到了这一点，在Hashtable之后又添加了HashMap，HashMap就是完全的没有加锁，一个是二话没说就加锁，一个是完全没有加锁。那这两个除了这个加锁区别之外还有其他的一些源码上的区别，
    - 所以Sun在那个时候就在这个HashMap的基础之上又添加了一个，说你用的这个新的HashMap比原来Hashtable好用，但是HashMap没有那些锁的东西，那么怎么才可以让这个HashMap既可以用于这些不需要锁的环境，又可以用于需要锁的环境呢？所以它又添加了一个方法叫做Collections相当于这个容器的工具类，这个工具类里有一个方法叫synchronizedMap，这个方法会把它变成加锁的版本。所以，HashMap有两个版体。
- Vector和 Hashtable 自带锁，基本不用

### 2、HashTable ~ CHM —— HashTable

- 见以下小程序：Hashtable里面装的是KeyValue对。这个Key是常量，这两个常量在Constants类里：100万对UUID的内容要装到容器里，定义了100个线程，访问的时候会有100个线程来模拟。

- ```java
  public class Constants {
      public static final int COUNT = 1000000;
      public static final int THREAD_COUNT = 100;
  }
  ```

- 这两个值的变化会引起效率上的变化，永远要记住这一点，哪个效率高那个效率低不要想当然，务必要写程序来测试，见以下程序，我先new出100万个Key和100万个Value来，然后把这些东西装到数组里面去，for循环(inti=O;i<count; i++)。

  - 为什么先把这些UUID对准备好而不是装的时候现生成？原因是我们写这个测试用例的时候前后用的是一样的，向Hashtable里头装的时候也得是这100万对，同样的内容，但要是每次都生成是不一样的内容,在这种情况下你测试就会有一些干扰因素。后面，写了一个线程类MyThread，从Thread继承，start，gap是每个线程负责往里面装多少。线程开始往里面扔。在看后面主程序代码，记录起始时间，new出来一个线程数组，这个线程数据总共有100个线程，给它做初始化。由于你需要指定这个start值，所以这个MyThread启动的时候 i 乘以count() ，总而言之就是这个起始的值不一样，第一个线程是从0开始，第二个是从100000个开始。然后让每一个线程启动，等待每一个线程结束，最后计算这个线程时间。
  - 重新解释一下，现在有一个Hashtable，里面装的是一对一对的内容，现在我们起了100个线程，这个100个线程去Key，Value取数据，一个线程取1万个数据，一共100万个数据，100个线程，每个线程取1万个数据往里插。

- ```java
  public class T01_TestHashtable {
      static Hashtable<UUID, UUID> m = new Hashtable<>();
      static int count = Constants.COUNT;
      static UUID[] keys = new UUID[count];
      static UUID[] values = new UUID[count];
      static final int THREAD_COUNT = Constants.THREAD_COUNT;
      static {
          for (int i = 0; i < count; i++) {
              keys[i] = UUID.randomUUID();
              values[i] = UUID.randomUUID();
          }
      }
  
      static class MyThread extends Thread {
          int start;
          int gap = count/THREAD_COUNT;
  
          public MyThread(int start) {
              this.start = start;
          }
  
          @Override
          public void run() {
              for(int i=start; i<start+gap; i++) {
                  m.put(keys[i], values[i]);
              }
          }
      }
      public static void main(String[] args) {
          long start = System.currentTimeMillis();
          Thread[] threads = new Thread[THREAD_COUNT];
          for(int i=0; i<threads.length; i++) {
              threads[i] =
              new MyThread(i * (count/THREAD_COUNT));
          }
          for(Thread t : threads) { t.start(); }
          for(Thread t : threads) { try { t.join(); } catch (InterruptedException e)  e.printStackTrace(); } }
          long end = System.currentTimeMillis();
          System.out.println(end - start);
          System.out.println(m.size());
  
          //-----------------------------------
  
          start = System.currentTimeMillis();
          for (int i = 0; i < threads.length; i++) {
              threads[i] = new Thread(()->{
                  for (int j = 0; j < 10000000; j++) {
                      m.get(keys[10]);
                  }
              });
          }
          for(Thread t : threads) { t.start(); }
          for(Thread t : threads) { try { t.join(); } catch (InterruptedException e)  e.printStackTrace(); } }
          end = System.currentTimeMillis();
          System.out.println(end - start);
      }
  }
  ```

### 3、HashTable ~ CHM —— HashMap

- 虽然速度比较快，但是数据会出问题，还各种各样的报异常。主要是因为它内部会把这个变成TreeNode，总而言之HashMap往里扔的时候，由于它内部没有锁，所以多线程访问的时候会出问题。

- ```java
  public class T02_TestHashMap {
      static HashMap<UUID, UUID> m = new HashMap<>();
      static int count = Constants.COUNT;
      static UUID[] keys = new UUID[count];
      static UUID[] values = new UUID[count];
      static final int THREAD_COUNT = Constants.THREAD_COUNT;
      static {
          for (int i = 0; i < count; i++) {
              keys[i] = UUID.randomUUID();
              values[i] = UUID.randomUUID();
          }
      }
      static class MyThread extends Thread {
          int start;
          int gap = count/THREAD_COUNT;
  
          public MyThread(int start) {
              this.start = start;
          }
  
          @Override
          public void run() {
              for(int i=start; i<start+gap; i++) {
                  m.put(keys[i], values[i]);
              }
          }
      }
  
      public static void main(String[] args) {
          long start = System.currentTimeMillis();
          Thread[] threads = new Thread[THREAD_COUNT];
          for(int i=0; i<threads.length; i++) {
              threads[i] =
              new MyThread(i * (count/THREAD_COUNT));
          }
          for(Thread t : threads) { t.start(); }
          for(Thread t : threads) { try { t.join(); } catch (InterruptedException e)  e.printStackTrace(); } }
  
          long end = System.currentTimeMillis();
          System.out.println(end - start);
          System.out.println(m.size());
      }
  }
  ```

### 4、HashTable ~ CHM —— SynchronizedHashMap

- SynchronizedMap这个方法，给HashMap手动加锁，它的源码自己做了一个Object，然后每次都是Synchronized(Object),严格来讲他和那个Hashtable效率上区别不大。

- ```java
  public class T03_TestSynchronizedHashMap {
      static Map<UUID, UUID> m = Collections.synchronizedMap(new HashMap<UUID, UUID>());
      static int count = Constants.COUNT;
      static UUID[] keys = new UUID[count];
      static UUID[] values = new UUID[count];
      static final int THREAD_COUNT = Constants.THREAD_COUNT;
      static {
          for (int i = 0; i < count; i++) {
              keys[i] = UUID.randomUUID();
              values[i] = UUID.randomUUID();
          }
      }
      static class MyThread extends Thread {
          int start;
          int gap = count/THREAD_COUNT;
  
          public MyThread(int start) {
              this.start = start;
          }
  
          @Override
          public void run() {
              for(int i=start; i<start+gap; i++) {
                  m.put(keys[i], values[i]);
              }
          }
      }
  
      public static void main(String[] args) {
  
          long start = System.currentTimeMillis();
  
          Thread[] threads = new Thread[THREAD_COUNT];
  
          for(int i=0; i<threads.length; i++) {
              threads[i] =
              new MyThread(i * (count/THREAD_COUNT));
          }
          for(Thread t : threads) { t.start(); }
          for(Thread t : threads) { try { t.join(); } catch (InterruptedException e)  e.printStackTrace(); } }
          long end = System.currentTimeMillis();
          System.out.println(end - start);
          System.out.println(m.size());
  
          //-----------------------------------
  
          start = System.currentTimeMillis();
          for (int i = 0; i < threads.length; i++) {
              threads[i] = new Thread(()->{
                  for (int j = 0; j < 10000000; j++) {
                      m.get(keys[10]);
                  }
              });
          }
          for(Thread t : threads) { t.start(); }
          for(Thread t : threads) { try { t.join(); } catch (InterruptedException e)  e.printStackTrace(); } }
  
          end = System.currentTimeMillis();
          System.out.println(end - start);
      }
  }
  ```

### 5、HashTable ~ CHM —— ConcurrentHashMap

- ConcurrentHashMap是多线程里面真正用的，并发的。

- ConcurrentHashMap提高效率主要提高在读上面，由于它往里插的时候内部又做了各种各样的判断，本来是链表的，到8之后又变成了红黑树，然后里面又做了各种各样的cas的判断，所以他往里插的数据是要更低一些的。HashMap和Hashtable虽然说读的效率会稍微低一些，但是它往里插的时候检查的东西特别的少，就加个锁然后往里一插。所以，关于效率，还是看你实际当中的需求。

- ```java
  public class T04_TestConcurrentHashMap {
      static Map<UUID, UUID> m = new ConcurrentHashMap<>();
      static int count = Constants.COUNT;
      static UUID[] keys = new UUID[count];
      static UUID[] values = new UUID[count];
      static final int THREAD_COUNT = Constants.THREAD_COUNT;
      static {
          for (int i = 0; i < count; i++) {
              keys[i] = UUID.randomUUID();
              values[i] = UUID.randomUUID();
          }
      }
      static class MyThread extends Thread {
          int start;
          int gap = count/THREAD_COUNT;
  
          public MyThread(int start) {
              this.start = start;
          }
  
          @Override
          public void run() {
              for(int i=start; i<start+gap; i++) {
                  m.put(keys[i], values[i]);
              }
          }
      }
  
      public static void main(String[] args) {
          long start = System.currentTimeMillis();
          Thread[] threads = new Thread[THREAD_COUNT];
          for(int i=0; i<threads.length; i++) {
              threads[i] =
              new MyThread(i * (count/THREAD_COUNT));
          }
          for(Thread t : threads) { t.start(); }
          for(Thread t : threads) { try { t.join(); } catch (InterruptedException e)  e.printStackTrace(); } }
  
          long end = System.currentTimeMillis();
          System.out.println(end - start);
  
          System.out.println(m.size());
  
          //-----------------------------------
  
          start = System.currentTimeMillis();
          for (int i = 0; i < threads.length; i++) {
              threads[i] = new Thread(()->{
                  for (int j = 0; j < 10000000; j++) {
                      m.get(keys[10]);
                  }
              });
          }
          for(Thread t : threads) { t.start(); }
          for(Thread t : threads) { try { t.join(); } catch (InterruptedException e)  e.printStackTrace(); } }
  
          end = System.currentTimeMillis();
          System.out.println(end - start);
      }
  }
  ```

### 6、Vector ~ Queue —— ArrayList

- 认识一下Vector到Queue的发展历程，下面程序TicketSeller模拟售票，写法比较简单，用一个List把这些票全装进去，往里面装一万张票，然后10个线程也就是10个窗口对外销售，只要size大于零，只要还有剩余的票时我就往外卖，取一张往外卖remove。如果到最后一张票的时候，好几个线程执行到这里所以线程都发现了size大于零，所有线程都往外买了一张票，那么会发生什么情形，只有一个线程拿到了这张票，其他的拿到的都是空值，就是超卖的现象。没有加锁，线程不安全。

- ```java
  public class TicketSeller1 {
  	static List<String> tickets = new ArrayList<>();
  	static { for(int i=0; i<10000; i++) tickets.add("票编号：" + i); }	
  	public static void main(String[] args) {
  		for(int i=0; i<10; i++) {
  			new Thread(()->{
  				while(tickets.size() > 0) {
  					System.out.println("销售了--" + tickets.remove(0));
  				}
  			}).start();
  		}
  	}
  }
  ```

### 7、Vector ~ Queue —— Vector

- 最早的容器Vector，内部是自带锁的，源码中有很多方法都是synchronized，所以Vector一定是线程安全的。

- 100张票，强哥窗口，为了线程安全在调用size、remove方法时加锁了，但这两个的中间过程没有加锁，那么好多个线程还是会判断size>0，还是存在超卖的问题

- ```java
  public class TicketSeller2 {
  	static Vector<String> tickets = new Vector<>();
  	static { for(int i=0; i<10000; i++) tickets.add("票编号：" + i); }	
  	public static void main(String[] args) {		
  		for(int i=0; i<10; i++) {
  			new Thread(()->{
  				while(tickets.size() > 0) {					
  					try {
  						TimeUnit.MILLISECONDS.sleep(10);
  					} catch (InterruptedException e) {
  						e.printStackTrace();
  					}					
  					System.out.println("销售了--" + tickets.remove(0));
  				}
  			}).start();
  		}
  	}
  }
  ```

### 8、Vector ~ Queue —— LinkedList

- 虽然用了这个加锁的容器了，由于在你调用这个并发容器的时候，你是调用了其中的两个原子方法，所以在外层还得在加一把锁synchronized(tckets)，继续判断Size，售出去不断的remove，这种方案就没有问题了，但不是效率最高的方案

- ```java
  public class TicketSeller3 {
  	static List<String> tickets = new LinkedList<>();
  	static { for(int i=0; i<10000; i++) tickets.add("票编号：" + i); }	
  	public static void main(String[] args) {		
  		for(int i=0; i<10; i++) {
  			new Thread(()->{
  				while(true) {
  					synchronized(tickets) {
  						if(tickets.size() <= 0) break;						
  						try {
  							TimeUnit.MILLISECONDS.sleep(10);
  						} catch (InterruptedException e) {
  							e.printStackTrace();
  						}						
  						System.out.println("销售了--" + tickets.remove(0));
  					}
  				}
  			}).start();
  		}
  	}
  }
  ```

### 9、Vector ~ Queue —— Queue

- Queue的效率是最高到，这是最新的一个接口，他的主要目标就是为了高并发用的，就是为了多线程用的。所以，以后考虑多线程这种单个元素的时候多考虑Queue。

- 这个使用的是ConcurrentLinkedQueue，然后里面并没有加锁，直接调用了一个方法叫poll，pol的意思就是我从tickets去取值，这个值什么时候取空了就说明里面的值已经没了，所以这个while(true)不断的往外销售，一直到他突然取票的时候这里面没了，那我这个窗口就可以关了不用买票了。poll加了很多对于多线程访问的时候比较友好的一些方法，它的源码，得到并且去除掉这里面这个值，如果这个已经是空就返回null

- ```java
  public class TicketSeller4 {
  	static Queue<String> tickets = new ConcurrentLinkedQueue<>();
  	static { for(int i=0; i<1000; i++) tickets.add("票 编号：" + i); }
  	public static void main(String[] args) {
  		for(int i=0; i<10; i++) {
  			new Thread(()->{
  				while(true) {
  					String s = tickets.poll();
  					if(s == null) break;
  					else System.out.println("销售了--" + s);
  				}
  			}).start();
  		}
  	}
  }
  ```

### 10、以上总结，面向接口编程

- 从Map这个角度来讲最早是从Hashtable，二话不说先加锁到HashMap去除掉锁，再到synchronizedHashMap加一个带锁的版本，到ConcurrentlashMep多线程时候专用。注意，不是替代关系，这个归根结底还是会归到，到底cas操作就一定会比synchronized效率要高吗，不一定，要看你并发量的高低，要看你锁定之后代码执行的时间，任何时候在你实际情况下都需要通过测试，压测来决定用哪种容器。
- 设计上方法有一种叫面向接口编程，为什么要面向接口编程，如果说你在工作之中设计一个程序，这个程序你应该设计一个接口，这个接口里面只包括业务逻辑，取出学生列表，放好，但是这里列表具体的实现是放到Hashtable里面还是放到HashMap里面还是放到ConcurrentHashMap里面，你可以写好几个不同的实现，在不同的并发情况下采用不同的实现你的程序会更灵活，在这里你们能不能理解就是这种面向接口的编程和面向接口的设计它的微妙之所在。

### 11、多线程下使用的容器 —— ConcurrentMap

- Map经常用的有这么几个

  - ConcurrentHashMap用hash表实现的这样一个高并发容器；

  - 既然有了ConcurrentHashMap正常情况下就应该有ConcurentTreeMap，但是并没有，原因就是ConcurrentHashMap里面用的是cas操作，这个cas操作它用在tree的时候，用在树这个节点上的时候实现起来太复杂了，所以就没有这个ConcurrentTreeMap，但是有时间也需要这样一个排好序的Map，那就有了ConcurrentSkipListMap跳表结构就出现了。

  - ConcurrentSkipListMap通过跳表来实现的高并发容器并且这个Map是有排序的；

  - 跳表底层本身存储的元素一个链表，它是排好顺序的，对于单链表来说，即使数据是已经排好序的，想要查询其中的一个数据，只能从头开始遍历链表，这样效率很低，时间复杂度很高，是 O(n)。我们可以为链表建立一个“索引”，这样查找起来就会更快，如下图所示，我们在原始链表的基础上，每两个结点提取一个结点建立索引，我们把抽取出来的结点叫做索引层或者索引，down 表示指向原始链表结点的指针

  - 他们两个的区别一个是有序的一个是无序的，同时都支持并发的操作。

  - <img src="_media/basis/多线程与高并发/跳表.png"/>

  - ```java
    public class T01_ConcurrentMap {
    	public static void main(String[] args) {
    		Map<String, String> map = new ConcurrentHashMap<>();
    		//Map<String, String> map = new ConcurrentSkipListMap<>(); //高并发并且排序		
    		//Map<String, String> map = new Hashtable<>();
    		//Map<String, String> map = new HashMap<>(); //Collections.synchronizedXXX
    		//TreeMap
    		Random r = new Random();
    		Thread[] ths = new Thread[100];
    		CountDownLatch latch = new CountDownLatch(ths.length);
    		long start = System.currentTimeMillis();
    		for(int i=0; i<ths.length; i++) {
    			ths[i] = new Thread(()->{
    				for(int j=0; j<10000; j++) map.put("a" + r.nextInt(100000), "a" + r.nextInt(100000));
    				latch.countDown();
    			});
    		}		
    		Arrays.asList(ths).forEach(t->t.start());
    		try { latch.await(); } catch (InterruptedException e) { e.printStackTrace(); }
    		long end = System.currentTimeMillis();
    		System.out.println(end - start);
    		System.out.println(map.size());
    	}
    }
    ```

### 12、多线程下使用的容器 —— CopyOnWrite

- CopyOnWrite其中有CopyOnWriteList、CopyOnWriteSet两个。

- CopyOnWrite的意思叫写时复制。见以下程序，用了一个容器List，一个一个元素往里装，往里装的时候，装的一堆的数组，一堆的字符串，每100个线程往里面装1000个，

- 各种各样的实现，可以用ArrayList、Vector，但是ArrayList会出并发问题，因为多线程访问没有锁，可以用CopyOnWriteArrayList。

- 通过名字进行分析一下，当Write的时候我们要进行复制，写时复制。这个原理非常简单，当我们需要往里面加元素的时候你把里面的元素得复制出来。在很多情况下，写的时候特别少，读的时候很多。在这个时候就可以考虑CopyOnWrite这种方式来提高效率。

- CopyOnWrite为什么会提高效率呢，是因为读的时候不加锁，Vector是写的时候加锁，读的时候也加锁。那么用CopyOnWriteList读的时候不加锁，写的时候会在原来的基础上拷贝一个，拷贝的时候扩展出一个新元素来，然后把你新添加的这个扔到这个元素扔到最后这个位置上。与此同时把指向老的容器的一个引用指向新的，这个写法就是写时复制。

- 这个写时复制，写的效率比较低，因为每次写都要复制。在读比较多写比较少的情况下使用CopyOnWrite

- ```java
  public class T02_CopyOnWriteList {
  	public static void main(String[] args) {
  		List<String> lists = 
  				//new ArrayList<>(); //这个会出并发问题！
  				//new Vector();
  				new CopyOnWriteArrayList<>();
  		Random r = new Random();
  		Thread[] ths = new Thread[100];		
  		for(int i=0; i<ths.length; i++) {
  			Runnable task = new Runnable() {	
  				@Override
  				public void run() {
  					for(int i=0; i<1000; i++) lists.add("a" + r.nextInt(10000));
  				}				
  			};
  			ths[i] = new Thread(task);
  		}
  		runAndComputeTime(ths);
  		System.out.println(lists.size());
  	}
  	
  	static void runAndComputeTime(Thread[] ths) {
  		long s1 = System.currentTimeMillis();
  		Arrays.asList(ths).forEach(t->t.start());
  		Arrays.asList(ths).forEach(t->{
  			try { t.join(); } catch (InterruptedException e) { e.printStackTrace(); }
  		});
  		long s2 = System.currentTimeMillis();
  		System.out.println(s2 - s1);		
  	}
  }
  ```

- CopyOnWrite对比的是synchronizedList

- ```java
  public class T03_SynchronizedList {
  	public static void main(String[] args) {
  		List<String> strs = new ArrayList<>();
  		List<String> strsSync = Collections.synchronizedList(strs);
  	}
  }
  ```

### 13、多线程下使用的容器 —— BlockingQueue

- BlockingQueue，是线程池需要用到的这方面的内容。

- BlockingQueue的概念重点是在Blocking上，Blocking阻塞，Queue队列，是阻塞队列。他提供了一系列的方法，我们可以在这些方法的基础之上做到让线程实现自动的阻塞。

- 他提供了一些比较友好的接口：

  - 第一个就是offer对应的是原来的add
    - offer是往里头添加，加进去没加进去它会给你一个布尔类型的返回值，
    - 和原来的add的区别是，add如果加不进去了是会抛异常的。所以一般的情况下我们用的最多的Queue里面都用offer，它会给你一个返回值
  - 提供了poll取数据，然后提供了peek拿出来这个数据。
    - peek的概念是去取并不是让你remove掉，
    - poll是取并且remove掉，而且这几个对于BlockingQueue来说也确实是线程安全的一个操作。
  
- ```java
  public class T04_ConcurrentQueue {
  	public static void main(String[] args) {
  		Queue<String> strs = new ConcurrentLinkedQueue<>();
  		for(int i=0; i<10; i++) {
  			strs.offer("a" + i);  //add
  		}		
  		System.out.println(strs);		
  		System.out.println(strs.size());
  		
  		System.out.println(strs.poll());
  		System.out.println(strs.size());
  		
  		System.out.println(strs.peek());
  		System.out.println(strs.size());		
  		//双端队列Deque
  	}
  }
  ```

### 14、多线程下使用的容器 —— LinkedBlockingQueue

- LnkedBlockingQueue，体现Concurremt的这个点在哪里呢，我们来看这个LinkedBlockingQueue，用链表实现的BlockingQueve，是一个特别大的有界队列，相当于无界队列。就是它可以一直装到 Integer.MAX_ _VALUE，一直添加。

- 见以下程序，这么一些线程，第一个线程是往里头加内容，put。BlockingQueue在Queve的基础上又添加了两个方法，这两个方法一个叫put，一个叫take。这两个方法是真真正正的实现了阻塞。

  - put往里装如果满了的话我这个线程会阻塞住，take往外取如果空了的话线程会阻塞住。

- 所以这个BlockingQueve就实现了生产者消费者里面的那个容器。这个小程序是往里面装了100个字符串，a开头结尾，每装一个的时候睡1秒钟。然后，后面又启动了5个线程不断的从里面take，空了就等着，什么时候新加了就马上取出来。这是BlockingQueue和Queue的一个基本的概念。

- ```java
  public class T05_LinkedBlockingQueue {
  	static BlockingQueue<String> strs = new LinkedBlockingQueue<>();
  	static Random r = new Random();
  	public static void main(String[] args) {
  		new Thread(() -> {
  			for (int i = 0; i < 100; i++) {
  				try {
  					strs.put("a" + i); //如果满了，就会等待
  					TimeUnit.MILLISECONDS.sleep(r.nextInt(1000));
  				} catch (InterruptedException e) {
  					e.printStackTrace();
  				}
  			}
  		}, "p1").start();
  
  		for (int i = 0; i < 5; i++) {
  			new Thread(() -> {
  				for (;;) {
  					try {
                          //如果空了，就会等待
  						System.out.println(Thread.currentThread().getName() + " take -" + strs.take()); 
  					} catch (InterruptedException e) {
  						e.printStackTrace();
  					}
  				}
  			}, "c" + i).start();
  		}
  	}
  }
  ```

### 15、多线程下使用的容器 —— ArrayBlockingQueue

- ArrayBlockingQueue是有界的，可以指定它一个固定的值10，它容器就是10，那么当你往里面扔容器的时候，一旦他满了这个put方法就会阻塞住。然后你可以看看用add方法满了之后他会报异常。

- offer用返回值来判断到底加没加成功，offer还有另外一个写法你可以指定一个时间尝试着往里面加1秒钟，1秒钟之后如果加不进去它就返回了。

- Queue和List的区别到底在哪里，主要就在这里，添加了offer、peek. poll. put、take这些个对线程友好的或者阻塞，或者等待方法。

- ```java
  public class T06_ArrayBlockingQueue {
  	static BlockingQueue<String> strs = new ArrayBlockingQueue<>(10);
  	static Random r = new Random();
  	public static void main(String[] args) throws InterruptedException {
  		for (int i = 0; i < 10; i++) {
  			strs.put("a" + i);
  		}		
  		//strs.put("aaa"); //满了就会等待，程序阻塞
  		//strs.add("aaa");
  		//strs.offer("aaa");
  		strs.offer("aaa", 1, TimeUnit.SECONDS);		
  		System.out.println(strs);
  	}
  }
  ```

### 16、多线程下使用的容器 —— DelayQueue

- DelayQueue可以实现在时间上的排序，按照在里面等待的时间来进行排序。

- 见以下程序：new了一个DelayQueue，他是BlockingQueue的一种也是用于阻塞的队列，这个阻塞队列装任务的时候要求你必须实现Delayed接口，Delayed往后拖延推迟，Delayed需要做一个比较compareTo，等待时间越短的就会优先得到运行，比较的时候可以按照时间来排序。

- 总而言之，要实现Comparable接口重写 compareTo方法来确定你这个任务之间是怎么排序的。getDelay去拿到你Delay多长时间了。往里头装任务的时候首先拿到当前时间，在当前时间的基础之上指定在多长时间之后这个任务要运行，添加顺序参考代码。

- 一般的队列是先进先出。这个队列是不一样的，按时间进行排序（按紧迫）。 DelayQueue就是按照时间进行任务调度。

- ```java
  public class T07_DelayQueue {
  	static BlockingQueue<MyTask> tasks = new DelayQueue<>();
  	static Random r = new Random();	
  	static class MyTask implements Delayed {
  		String name;
  		long runningTime;
  		
  		MyTask(String name, long rt) {
  			this.name = name;
  			this.runningTime = rt;
  		}
  		@Override
  		public int compareTo(Delayed o) {
  			if(this.getDelay(TimeUnit.MILLISECONDS) < o.getDelay(TimeUnit.MILLISECONDS))
  				return -1;
  			else if(this.getDelay(TimeUnit.MILLISECONDS) > o.getDelay(TimeUnit.MILLISECONDS)) 
  				return 1;
  			else 
  				return 0;
  		}
  		@Override
  		public long getDelay(TimeUnit unit) {			
  			return unit.convert(runningTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
  		}			
  		@Override
  		public String toString() { return name + " " + runningTime; }
  	}
  
  	public static void main(String[] args) throws InterruptedException {
  		long now = System.currentTimeMillis();
  		MyTask t1 = new MyTask("t1", now + 1000);
  		MyTask t2 = new MyTask("t2", now + 2000);
  		MyTask t3 = new MyTask("t3", now + 1500);
  		MyTask t4 = new MyTask("t4", now + 2500);
  		MyTask t5 = new MyTask("t5", now + 500);		
  		tasks.put(t1);
  		tasks.put(t2);
  		tasks.put(t3);
  		tasks.put(t4);
  		tasks.put(t5);		
  		System.out.println(tasks);		
  		for(int i=0; i<5; i++) {
  			System.out.println(tasks.take());
  		}
  	}
  }
  ```

### 17、多线程下使用的容器 —— PriorityQueue

- DelayQueue本质上用的是一个PriorityQueue， PriorityQueue是从AbstractQueue继承的。

- PriorityQueue特点是它内部你往里装的时候并不是按顺序往里装的，而是内部进行了一个排序。按照优先级，最小的优先。它内部实现的结构是一个二叉树，这个二叉树可以认为是堆排序里面的那个最小堆值排在最上面。

- ```java
  public class T07_01_PriorityQueque {
      public static void main(String[] args) {
          PriorityQueue<String> q = new PriorityQueue<>();
          q.add("c");
          q.add("e");
          q.add("a");
          q.add("d");
          q.add("z");
          for (int i = 0; i < 5; i++) {
              System.out.println(q.poll());
          }
      }
  }
  ```

### 18、多线程下使用的容器 —— SynchronizeQueue

- SynthionousQueue容量为0，不是用来装内容的，SynchronousQueue是专门用来两个线程之间传内容的，给线程下达任务的，本质上这个容器的概念和Exchanger是一样的。

- 看下面代码，有一个线程起来等着take，里面没有值一定是take不到的，然后就等着。然后当put的时候能取出来，take到了之后能打印出来，最后打印这个容器的size一定是0，打印出aaa。那当把线程注释掉，在运行一下程序就会在这阻塞，永远等着。如果add方法直接就报错，原因是满了，这个容器为0，你不可以往里面扔东西

- 这个Queue和其他的很重要的区别就是不能往里头装东西，只能用来阻塞式的put调用，要求是前面得有人等着拿这个东西的时候你才可以往里装，但容量为0，其实说白了就是我要递到另外一个的手里才可以。这个SynchronousQueue看似没有用，其实不然，SynchronousQueue在线程池里用处特别大，很多的线程取任务，互相之间进行任务的一个调度的时候用的都是它。

- ```java
  public class T08_SynchronusQueue { //容量为0
  	public static void main(String[] args) throws InterruptedException {
  		BlockingQueue<String> strs = new SynchronousQueue<>();		
  		new Thread(()->{
  			try {
  				System.out.println(strs.take());
  			} catch (InterruptedException e) {
  				e.printStackTrace();
  			}
  		}).start();
  		strs.put("aaa"); //阻塞等待消费者消费
  		//strs.put("bbb");
  		//strs.add("aaa");
  		System.out.println(strs.size());
  	}
  }
  ```

### 19、多线程下使用的容器 —— TransferQueue

- TansterQueue传递，实际上是前面这各种各样Queue的一个组合，它可以给线程来传递任务，与此同时不像是SynchronousQueue只能传递一个，TransterQueue做成列表可以传好多个。

- 比较厉害的是它添加了一个方法叫transfer，如果我们用put就相当于一个线程来了往里—装它就走了。transfer就是装完在这等着，阻塞等有人把它取走我这个线程才回去干我自己的事情。

- 一般使用场景：是我做了一件事情，我这个事情要求有一个结果，有了这个结果之后我可以继续进行我下面的这个事情的时候，比方说我付了钱，这个订单我付账完成了，但是我一直要等这个付账的结果完成才可以给客户反馈。

- ```java
  public class T09_TransferQueue {
  	public static void main(String[] args) throws InterruptedException {
  		LinkedTransferQueue<String> strs = new LinkedTransferQueue<>();		
  		new Thread(() -> {
  			try {
  				System.out.println(strs.take());
  			} catch (InterruptedException e) {
  				e.printStackTrace();
  			}
  		}).start();		
  		strs.transfer("aaa");
  		//strs.put("aaa");
  
  		/*new Thread(() -> {
  			try {
  				System.out.println(strs.take());
  			} catch (InterruptedException e) {
  				e.printStackTrace();
  			}
  		}).start();*/
  
  	}
  }
  ```

## 七、多线程交替输出解决方案

问题：两个线程，第一个线程是从1到26，第二个线程是从A到一直到Z，然后要让这两个线程做到同时运行，交替输出，顺序打印。

### 1、LockSupport 实现

- 用LockSupport其实是最简单的。让一个线程输出完了之后停止，然后让另外一个线程继续运行就完了。我们定义了两个数组，两个线程，第一个线程拿出数组里面的每一个数字来，然后打印，打印完叫醒t2，然后让自己阻塞。另外一个线程上来之后自己先park，打印完叫醒线程t1。两个线程就这么交替来交替去，就搞定了。

- ```java
  //Locksupport park 当前线程阻塞（停止）
  //unpark(Thread t)
  public class T02_00_LockSupport {
      static Thread t1 = null, t2 = null;
      public static void main(String[] args) throws Exception {
          char[] aI = "1234567".toCharArray();
          char[] aC = "ABCDEFG".toCharArray();
          t1 = new Thread(() -> {
                  for(char c : aI) {
                      System.out.print(c);
                      LockSupport.unpark(t2); //叫醒T2
                      LockSupport.park(); //T1阻塞
                  }
          }, "t1");
  
          t2 = new Thread(() -> {
              for(char c : aC) {
                  LockSupport.park(); //t2阻塞
                  System.out.print(c);
                  LockSupport.unpark(t1); //叫醒t1
              }
          }, "t2");
          t1.start();
          t2.start();
      }
  }
  ```

### 2、wait、notify、notifyAll 实现

- 首先调用wait、 notty的时候，walit线程阻塞，notify叫醒其他线程，调用这个两个方法的时候必须要进行synchronized锁定的，如果没有synchronized这个线程是锁定不了的，因此我们定义一个锁的对象new Object()，两个数组，第一线程上来先锁定Objec对象o，锁定完对象之后，开始输出，输出第一个数字，输出完之后叫醒第二个，然后自己walt。还是这个思路，其实这个就和LookSuppont的park、unpark是非常类似的，

- 最容易出错的就是把整个数姐都打印完了要记得nottfy，因为这两个线程里面终归有一个线程walt的，是阻塞在这停止不动的。

- ```java
  public class T06_00_sync_wait_notify {
      public static void main(String[] args) {
          final Object o = new Object();
          char[] aI = "1234567".toCharArray();
          char[] aC = "ABCDEFG".toCharArray();
          new Thread(()->{
              synchronized (o) {
                  for(char c : aI) {
                      System.out.print(c);
                      try {
                          o.notify();
                          o.wait(); //让出锁
                      } catch (InterruptedException e) { e.printStackTrace(); }
                  }
                  o.notify(); //必须，否则无法停止程序
              }
  
          }, "t1").start();
  
          new Thread(()->{
              synchronized (o) {
                  for(char c : aC) {
                      System.out.print(c);
                      try {
                          o.notify();
                          o.wait();
                      } catch (InterruptedException e) { e.printStackTrace(); }
                  }
                  o.notify();
              }
          }, "t2").start();
      }
  }  //如果我想保证t2在t1之前打印，也就是说保证首先输出的是A而不是1，这个时候该如何做？
  ```

### 3、保证一个线程先执行 wait、notify、notifyAll 实现

- 保证第一个线程先运行，办法也是非常的多的，看以下程序，使用自旋的方式，设置一个boolean类型的变量，t2刚开始不是static。如果说2没有static的话，我这个t1线程就wait，要求t2必须先static才能执行业务逻辑。

- 还有一种写法就是t2先wait，然后t1先输出，输出完了之后notify；

- 还有一种写法用CountDownLatch也可以；

- ```java
  public class T07_00_sync_wait_notify {
      private static volatile boolean t2Started = false;
      //private static CountDownLatch latch = new C(1);
      public static void main(String[] args) {
          final Object o = new Object();
          char[] aI = "1234567".toCharArray();
          char[] aC = "ABCDEFG".toCharArray();
          new Thread(()->{
              //latch.await();
              synchronized (o) {
                  while(!t2Started) {
                      try {
                          o.wait();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
                  for(char c : aI) {
                      System.out.print(c);
                      try {
                          o.notify();
                          o.wait();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
                  o.notify();
              }
          }, "t1").start();
  
          new Thread(()->{
              synchronized (o) {
                  for(char c : aC) {
                      System.out.print(c);
                      //latch.countDown()
                      t2Started = true;
                      try {
                          o.notify();
                          o.wait();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
                  o.notify();
              }
          }, "t2").start();
      }
  }
  ```

- 以上两种最重要的方法，一个是LockSuppont，一个是synchronized、wait、notfy。这两种面试的时候你要是能写出来问题就不大，但是，你如果能用新的lock的接口，就不再用synchronized，用这种自旋的，也可以。严格来讲这个lock和synchromized本质是一样的。不过他也有好用的地方，下面我们来看看写法。


### 4、ReentrantLock 单 Condition 实现

- 用一个ReentrantLock，然后调用newCondition，上来之后先lock相当于synchronized了，打印完之后signal叫醒另一个当前的等待，最后condition.signal()相当于notiy()，之后另外一个也类似就完了，这种写法相当于synchronized的一个变种。

- ```java
  public class T08_00_lock_condition {
      public static void main(String[] args) {
          char[] aI = "1234567".toCharArray();
          char[] aC = "ABCDEFG".toCharArray();
          Lock lock = new ReentrantLock();
          Condition condition = lock.newCondition();
          new Thread(()->{
              try {
                  lock.lock();
                  for(char c : aI) {
                      System.out.print(c);
                      condition.signal();
                      condition.await();
                  }
                  condition.signal();
              } catch (Exception e) {
                  e.printStackTrace();
              } finally {
                  lock.unlock();
              }
          }, "t1").start();
  
          new Thread(()->{
              try {
                  lock.lock();
                  for(char c : aC) {
                      System.out.print(c);
                      condition.signal();
                      condition.await();
                  }
                  condition.signal();
              } catch (Exception e) {
                  e.printStackTrace();
              } finally {
                  lock.unlock();
              }
          }, "t2").start();
      }
  }
  ```

### 5、ReentrantLock 多 Condition 实现

- 如果使用两个Condition的情况就会好很多。一把锁的等待队列里有好多线程，假如我要notify的话，实际上要找出一个让它运行，如果说我要调用的是一个notifyAlI的话，是让所有线程都醒过来去争用这把锁看谁能抢的到，谁抢到了就让这个线程运行。

- 那么，在这里面不能要求哪一类或者哪一个线程醒过来，既然我们有两个线程，那完全可以模仿生产者和消费者我干脆来两种的Condition，Condition它本质上是一个等待队列，就是两个等待队列，其中一个线程在这个等待队列上，另一个线程在另外一个等待队列上。

- 所以如果说我用两个Condition的话就可以精确的指定哪个等待队列里的线程醒过来去执行任务。

- 相当于有两个等待队列，t1在这个等待队列里，t2在另一个等待队列里，在t1完成了之后呢叫醒t2是指定你这个队列的线程醒过来，所以永远都是t2。

- ```java
  public class T09_00_lock_condition {
      public static void main(String[] args) {
          char[] aI = "1234567".toCharArray();
          char[] aC = "ABCDEFG".toCharArray();
          Lock lock = new ReentrantLock();
          Condition conditionT1 = lock.newCondition();
          Condition conditionT2 = lock.newCondition();
          new Thread(()->{
              try {
                  lock.lock();
                  for(char c : aI) {
                      System.out.print(c);
                      conditionT2.signal();
                      conditionT1.await();
                  }
                  conditionT2.signal();
              } catch (Exception e) {
                  e.printStackTrace();
              } finally {
                  lock.unlock();
              }
          }, "t1").start();
  
          new Thread(()->{
              try {
                  lock.lock();
                  for(char c : aC) {
                      System.out.print(c);
                      conditionT1.signal();
                      conditionT2.await();
                  }
                  conditionT1.signal();
              } catch (Exception e) {
                  e.printStackTrace();
              } finally {
                  lock.unlock();
              }
          }, "t2").start();
      }
  }
  ```

### 6、volatile  CAS实现

- 用自旋式的写法，就是不使用锁，相当于自己写了一个自旋锁。CAS的写法，这个写法用了enum，到底哪个线程要运行他只能取两个值，T1和T2，然后定义了一个ReadyToRun的变量，刚开始的时候是T1，这个意思呢就相当于是我有一个信号灯，这个信号灯要么就是T1要么就是T2，只能取这个两个值，不能取别的，当一开始的时候我在这个信号灯上显示的是T1，T1你可以走一步。

- 第一上来判断是不是T1，如果不是就占用cpu在这循环等待，如果一看是T1就打印，然后把r值变成T2进行下一次循环，下一次循环上来之后这个r是不是T1，不是T1就有在这转圈玩儿，而第二个线程发现它变成T2了，变成T2了下面的线程就会打印A，打印完了之后有把这个变成了T1，就这么交替交替，写volatile是保证线程的可见性。用enum类型，就是防止它取别的值，用一个int类型或者布尔也都可以。

- ```java
  public class T03_00_cas {
      enum ReadyToRun {T1, T2}
      static volatile ReadyToRun r = ReadyToRun.T1; //思考为什么必须volatile
      public static void main(String[] args) {
          char[] aI = "1234567".toCharArray();
          char[] aC = "ABCDEFG".toCharArray();
          new Thread(() -> {
              for (char c : aI) {
                  while (r != ReadyToRun.T1) {}
                  System.out.print(c);
                  r = ReadyToRun.T2;
              }
          }, "t1").start();
  
          new Thread(() -> {
              for (char c : aC) {
                  while (r != ReadyToRun.T2) {}
                  System.out.print(c);
                  r = ReadyToRun.T1;
              }
          }, "t2").start();
      }
  }
  ```

### 7、BlockingQueue 实现

- BlockingQueue可以支持多线程的阻塞操作，他有两个操作一个是put，一个take。 put的时候满了他就会阻塞住，take的时候如果没有，他就会阻塞住在这儿等着，我们利用这个特点来了两个BlockingQueue，这两个BlockingQueue都是ArrayBlockingQueue数组实现的，但是数组的长度是1，相当于我用了两个容器，这两个容器里头放两个值，这两个值比如说我第一个线程打印出1来了我就在这边放一个，而另外一个线程盯着这个事，他take，这个take里面没有值的时候他是要在这里阻塞等待的，take不到的时候他就等着，等什么时候这边打印完了，take到了他就打印这个A，打印完了A之后他就往第二个里面放一个，第一个线程也去take第二个容器里面的，什么时候take到了他就接着往下打印。

- ```java
  public class T04_00_BlockingQueue {
      static BlockingQueue<String> q1 = new ArrayBlockingQueue(1);
      static BlockingQueue<String> q2 = new ArrayBlockingQueue(1);
      public static void main(String[] args) throws Exception {
          char[] aI = "1234567".toCharArray();
          char[] aC = "ABCDEFG".toCharArray();
          new Thread(() -> {
              for(char c : aI) {
                  System.out.print(c);
                  try {
                      q1.put("ok");
                      q2.take();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
  
          }, "t1").start();
          
          new Thread(() -> {
              for(char c : aC) {
                  try {
                      q1.take();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.print(c);
                  try {
                      q2.put("ok");
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }, "t2").start();
      }
  }
  ```

### 8、PipedInputStream、PipedOutputStream 实现

- 这个效率非常低，它里面有各种的同步，了解即可。这里要把两个线程连接起来要求的步骤比较多，要求建立一个PipedinputStream和一个PipedOutputStream。就相当于两个线程通信，第一个这边就得有一个OutputStream，对应第二个线程这边就得有一个InputStream，同样的第二个要往第一个写的话，第一个也得有一个InputStream，第二个也还得有一个OutputSiream。最后要求你的第一个线程的input1和你第二个线程的output2连接connec起来，互相之间的扔消息玩儿，这边搞定了告诉另一边儿，另一边儿搞定了告诉这边，回合制。

- ```java
  public class T10_00_PipedStream {
      public static void main(String[] args) throws Exception {
          char[] aI = "1234567".toCharArray();
          char[] aC = "ABCDEFG".toCharArray();
          PipedInputStream input1 = new PipedInputStream();
          PipedInputStream input2 = new PipedInputStream();
          PipedOutputStream output1 = new PipedOutputStream();
          PipedOutputStream output2 = new PipedOutputStream();
          input1.connect(output2);
          input2.connect(output1);
          String msg = "Your Turn";
          new Thread(() -> {
              byte[] buffer = new byte[9];
              try {
                  for(char c : aI) {
                      input1.read(buffer);
                      if(new String(buffer).equals(msg)) {
                          System.out.print(c);
                      }
                      output1.write(msg.getBytes());
                  }
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }, "t1").start();
  
          new Thread(() -> {
              byte[] buffer = new byte[9];
              try {
                  for(char c : aC) {
                      System.out.print(c);
                      output2.write(msg.getBytes());
                      input2.read(buffer);
                      if(new String(buffer).equals(msg)) {
                          continue;
                      }
                  }
  
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }, "t2").start();
      }
  }
  ```

### 9、TransferQueue实现

- TransferQueue，就是我一个线程往里头生产之后扔在这的时候，个线程是阻塞的不动的，什么时候有另外一个线程把这个拿走了，拿走了之后这个线程才返回继续运行。

- 以下用了一个TransferQueue，第一个线程先take，相当于第一个线程做了一个消费者，就在这个Queue等着，看看有没有人往里扔。第二个线程上来经过transfer，就把这个字母扔进去了，扔进去了一个A，第一个线程发现很好，来了一个，我就把这个拿出来打印，打印完之后我又进行transfer，进去了一个1。然后，第二个线程它去里面take，把这个1列里让对方去打印。take出来打印。相当于我们自己每个人都把自己的一个数字或者是字母交到一个队列里面，让对方去打印。

- ```java
  public class T13_TransferQueue {
      public static void main(String[] args) {
          char[] aI = "1234567".toCharArray();
          char[] aC = "ABCDEFG".toCharArray();
  
          TransferQueue<Character> queue = new LinkedTransferQueue<Character>();
          new Thread(()->{
              try {
                  for (char c : aI) {
                      System.out.print(queue.take());
                      queue.transfer(c);
                  }
  
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }, "t1").start();
  
          new Thread(()->{
              try {
                  for (char c : aC) {
                      queue.transfer(c);
                      System.out.print(queue.take());
                  }
  
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }, "t2").start();
      }
  }
  ```

## 八、线程池与源码阅读

### 1、概论

![image-20240507173345333](_media/basis/多线程与高并发/线程池继承.png"/>

线程池首先有几个接口先了解第一个是Executor，第二个是ExecutorService，在后面才是线程池的一个使用ThreadPoolExecutor

### 2、Executor

- Executor 执行者，所以他有一个方法叫执行，那么执行的东西是Runnable，所以这个Executor是一个接口可以有好多实现。
- 因此有了Executor之后，我们线程就是一个任务的定义，比如Runnable起了一个命令的意思，他的定义和运行就可以分开了，不像我们以前定义一个Thread，new一个Thread然后去重写它的Run方法，start才可以运行，或者以前写了一个Runnable也必须得new一个Thread出来，以前的这种定义和运行是固定的，是写死的就是你new一个Thread让他出来运行。
- 不用你亲自去指定每一个Thread，他的运行的为式你可以自己去定义，所以至于是怎么去定义的就看你怎么实现Executor的接口了，这里是定义和运行分开这么一个含义，所以这个接口体现的是这个意思，所以这个接口就比较简单，至于你是直接调用run还是new一个Thread那是你自己的事儿。

### 3、ExecutorService

- ExecutorService是从Executor继承，另外，他除了去实现Executor可以去执行一个任务之外，他还完善了整个任务执行器的一个生命周期，拿线程池来举例，一个线程池里面一堆的线程就是一堆的工人，执行完一个任务之后这个线程怎么结束，线程池定义了一些个方法：

- ```java
  void shutdown();//结束
  List<Runnable> shutdownNow();//马上结束
  boolean isshutdown();//是否结束了
  boolean isTerminated();//是不是整体都执行完了
  boolean awaitTermination(long timeout, Timeunit unit) throws InterruptedException;//等着结束，等多长时间，时间到了还不结束的话,就返回false
  ```

- 所以这里面实现了一些有关线程池生命周期的东西，扩展了Executor的接口，真正的线程池的现实是在ExecutorService的这个基础上来实现的。当我们看到这个ExecutorService的时候你会发现他除了Executor执行任务之外还有submit提交任务，执行任务是直接拿过来马上运行，而submit是扔给这个线程池，什么时候运行由这个线程池来决定，相当于是异步的，我只要往里面一扔就不管了。
- 那么，如果不管的话什么时候有结果，这里面就涉及了比较新的类：比如说Future、RunnableFuture、FutureTask。
- 之前线程的时候定义一个线程的任务只能去实现Runnable接口，那在1.5之后他就增加了Callable这个接口。

### 4、Callable

- 下面代码是Caliabe的文档，他说这个接口和java.lang.Runnable类似，所以这两个类设计出来都是想潜在的另外一个线程去运行他，
- 所以通过这点可以知道Callable和Runnable一样，也可以是一个线程来运行
- 为什么有了Runnable还要有Callable，因为Gallable有一个返回值，call这个方法相当与Runnable里面的run方法，而Runnable里的方法返回值是空值，而这里是可以有一个返回值的，给你一个计算的任务，最后你得给我一个结果啊。这个叫做Callable，
- 那么由于他可以返回一个结果，我就可以把这个结果给存储起来，等什么时候计算完了通知我就可以了，我就不需要像原来线程池里面我调用他的run在这等着了。
- Callable是什么，他类似于Runnable，不过Callable可以有返回值。

### 5、Future

- Future代表的是那个Callable被执行完了之后我怎么才能拿到那个结果？

  - 它会封装到一个Future里面。Future表示将来，未来。未来你执行完之后可以把这个结果放到这个未来有可能执行完的结果里头，所以Future代表的是未来执行完的一个结果。

- 由于Callable基本上就是为了线程池而设计的，所以你要是不用线程池的接口想去写Callable的一些个小程序还是比较麻烦，所以这里面是要用到一些线程池的直接的用法，比较简单。

- Future是怎么用的？

  - 在我们读这个ExecutorService的时候你会发现他里面有submit方法，这个submit是异步的提交任务，提交完了任务之后原线程该怎么运行怎么运行，运行完了之后他会出一个结果，这个结果出在哪儿，他的返回值是一个Future，所以你只能去提交一个Callable，必须有返回值，把Callable的任务扔给线程池，线程池执行完了，异步的，就是把任务交给线程池之后我主线程该干嘛干嘛，调用get方法直到有结果之后get会返回。Callable一般是配合线程池和Future来用的。

  - ```java
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    ```

- 更灵活的用法是FutureTask，即是一个Future同时又是一个Task，原来这Callable只能一个Task只能是一个任务，但是他不能作为一个Future来用。这个FutureTask相当于是我自己可以作为一个任务来用，同时这个任务完成之后的结果也存在于这个对象里，为什么他能做到这一点，因为FutureTask他实现了RunnableFuture，而RunnableFuture即实现了Runnable又实现了Future，所以他即是一个任务又是一个Future。所以这个FutureTask是更好用的一个类。大家记住这个类，后面还会有WorkStealingPool、ForkjoinPool这些个基本上是会用到FutureTask类的。

- ```java
  public class T06_00_Future {
  	public static void main(String[] args) throws InterruptedException, ExecutionException {		
  		FutureTask<Integer> task = new FutureTask<>(()->{
  			TimeUnit.MILLISECONDS.sleep(500);
  			return 1000;
  		}); //new Callable () { Integer call();}		
  		new Thread(task).start();		
  		System.out.println(task.get()); //阻塞
  	}
  }
  ```

- 我们拓展了几个类，需要将这几个小类理解一下：

  - Callable 类似与 Runnable，但是有返回值。
  - 了解了Future，是用来存储执行的将来才会产生的结果。
  - FutureTask，他是Future加上Runnable，既可以执行又可以存结果。
  - CompletableFuture，管理多个Future的结果。

### 6、CompletableFuture

- CompletableFuture 的底层特别复杂，但是用法特别灵活，CompletableFuture 的底层用的是ForkJoinPool。

- 先来看他的用法：

  - 这个CompletableFuture非常的灵活，它内部有好多关于各种结果的一个组合，这个CompletableFuture是可以组合各种各样的不同的任务，然后等这个任务执行完产生一个结果进行一个组合。
  - 假如写了一个网站，这个网站都卖格力空调，同一个类型，然后很多人买东西都会进行一个价格比较，而你提供的这个服务就是我到淘宝上去查到这个格力空调买多少钱，然后我另启动一个线程去京东上找格力空调卖多少钱，在启动一个线程去拼多多上找，最后，你汇总一下这三个地方各售卖多少钱，然后你自己再来选去哪里买。
  - 下面代码，模拟了一个去别的地方取价格的一个方法，首先你去别的地方访问会花好长时间，因此我写了一个delay()让他去随机的睡一段时间，表示我们要联网，我们要爬虫爬结果执行这个时间，然后打印了一下睡了多少时间之后才拿到结果的，如拿到天猫上的结果是1块钱，淘宝上结果是2块钱，京东上结果是3块钱，总而言之是经过网络爬虫爬过来的数据分析出来的多少钱。然后我们需要模拟一下怎么拿到怎么汇总。
  - 第一种写法就是我注释的这种写法，就是挨着牌的写，假设跑天猫跑了10秒，跑淘宝拍了10秒，跑京东跑了5秒，一共历时25秒才总出来。但是如果我用不同的线程呢，一个一个的线程他们是并行的执行他们计算的结果是只有10秒。
  - 但是用线程你写起来会有各种各样的麻烦事儿，比如说在去淘宝的过程中网络报错了该怎么办，你去京东的过程中正好赶上那天他活动，并发访问特别慢你又该怎么办，你必须得等所有的线程都拿到之后才能产生一个结果，如果想要做这件事儿的话与其是要你每一个都要写一个自己的线程，需要考虑到各种各样的延迟的问题，各种各样的异常的问题这个时候有一个简单的写法，
  - 用一个CompletableFuture，首先CompletableFuture是一个Future，所以他会存一个将来有可能产生的结果值，结果值是一个Double ，它会运行一个任务，这个任务最后产生一个结果、会存在CompletableFuture里面，结果的类型是Double。
  - 在这里定义了三个Future，分别代表了淘宝、京东、天猫，用了CompletableFuture的一个方法叫supplyAsync产生了一个异步的任务 ，这个异步的任务去天猫那边拉数据。你可以想象在一个线程池里面扔给他一个任务让他去执行，什么时候执行完了之后他的结果会返回到这个futureTM里面。但是总体的要求就是这些个所有的future都得结束才可以，才能展示我最后的结果。
  - 往下走还有这么一直写法，就是我把这三个future都可以扔给一个CompletableFuture让他去管理，他管理的时候可以调用allOf方法相当于这里面的所有的任务全部完成之后，最后join，你才能够继续往下运行。所以CompletableFuture除了提供了比较好用的对任务的管理之外，还提供了对于任务堆的管理，用于对一堆任务的管理。CompletableFuture还提供了很多的写法，比如下面Lambda表达式的写法。

- ```java
  public class T06_01_CompletableFuture {
      public static void main(String[] args) throws ExecutionException, InterruptedException {
          long start, end;
  
          /*start = System.currentTimeMillis();
  
          priceOfTM();
          priceOfTB();
          priceOfJD();
  
          end = System.currentTimeMillis();
          System.out.println("use serial method call! " + (end - start));*/
  
          start = System.currentTimeMillis();
          //写法一，定三个future
          CompletableFuture<Double> futureTM = CompletableFuture.supplyAsync(()->priceOfTM());
          CompletableFuture<Double> futureTB = CompletableFuture.supplyAsync(()->priceOfTB());
          CompletableFuture<Double> futureJD = CompletableFuture.supplyAsync(()->priceOfJD());
          //写法二，三个方法交给一个future管理，必须全部执行结束后返回
          CompletableFuture.allOf(futureTM, futureTB, futureJD).join();
          //写法三，lambda的写法
          CompletableFuture.supplyAsync(()->priceOfTM())
                  .thenApply(String::valueOf)
                  .thenApply(str-> "price " + str)
                  .thenAccept(System.out::println);
  
          end = System.currentTimeMillis();
          System.out.println("use completable future! " + (end - start));
  
          try {
              System.in.read();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
      private static double priceOfTM() {
          delay();
          return 1.00;
      }
  
      private static double priceOfTB() {
          delay();
          return 2.00;
      }
  
      private static double priceOfJD() {
          delay();
          return 3.00;
      }
  
      /*private static double priceOfAmazon() {
          delay();
          throw new RuntimeException("product not exist!");
      }*/
  
      private static void delay() {
          int time = new Random().nextInt(500);
          try {
              TimeUnit.MILLISECONDS.sleep(time);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          System.out.printf("After %s sleep!\n", time);
      }
  }
  ```

- CompletableFuture 是各种任务的一种管理类，总而言之CompletableFuture是一个更高级的类，它能够在很高的一个层面上来帮助你管理一些个你想要的各种各样的任务，比如说你可以对任务进行各种各样的组合，所有任务完成之后你要执行一个什么样的结果，以及任何一个任务完成之后你要执行一个什么样的结果，还有他可以提供一个链式的处理方式Lambda的一些写法，拿到任务之后结果进行一个怎样的处理。

### 7、ThreadPoolExecutor —— 自定义

- 线程池从目前JDK提供的有两种类型：

  - 第一种就是普通的线程池ThreadPoolExecutor，
  - 第二种是ForkJoinPool （分叉，分叉完再分叉），
  - 这两种是不同类型的线程池，能干的事儿不太一样，我们说线程池的时候一般是说的第一种线程池，严格来讲这两种是不一样的，以下先对ThreadPoolExecutor进行一个入门，后面再讲ForkJolnPool。

- ThreadPoolExecutor的父类是从AbstractExecutorService，而AbstractExecutorService的父类是ExecutorService，ExecutorService的父类是Executor，所以ThreadPoolExecutor就相当于线程池的执行器，就是大家伙儿可以向这个池子里面扔任务，让这个线程池去运行。另外在阿里巴巴的手册里面要求线程池是要自定义的，因为需要自定义标准的线程名，方便JVM调试。

- 线程池维护着两个集合，第一个是线程的集合，里面是一个一个的线程。第二个是任务的集合，里面是一个一个的任务，这叫一个完整的线程池。

- 怎么样手动定义一个线程池，七个参数：

  - 第一个参数corePoolSoze核心线程数，最开始的时候线程池里面存在的核心线程；

  - 第二个叫maximumPoolSize最大线程数，线程数不够了，能扩展到最大线程是多少；

  - 第三个keepAliveTime生存时间，意思是扩展出来的线程有很长时间没干活了请你把它归还给操作系统；

  - 第四个TimeUnit.SECONDS生存时间的单位，到底是毫秒纳秒还是秒自己去定义；

  - 第五个是任务队列，就是之前提到的BlockingQueue，各种各样的BlockingQueue都可以，以下用的是ArrayBlockingQueue，参数最多可以装四个任务；

  - 第六个是线程工厂defaultThreadFactory，返回的是new DefaultThreadFactory()，它要实现ThreadFactory的接口，这个接口只有一个方法叫newThread，可以通过这种方式产生自定义的线程，默认产生的是defaultThreadFactory，而defaultThreadFactory产生线程的时候有几个特点：new出来的时候指定了group制定了线程名字，然后指定的这个线程绝对不是守护线程，设定好你线程的优先级。自己可以定义产生的到底是什么样的线程，指定线程名叫什么（为什么要指定线程名称，有什么意义，就是可以方便出错是回溯）；

  - 第七个叫拒绝策略，指的是线程池满了，而且任务队列也满了，这种情况下我们就要执行各种各样的拒绝策略，jdk默认提供了四种拒绝策略，也是可以自定义的。

    - 1：Abort：抛异常

    - 2：Discard：扔掉，不拋异常

    - 3：DiscardOldest：扔掉排队时间最久的

    - 4：CallerRuns：调用者处理服务

    - 一般情况这四种我们会自定义策略，去实现这个拒绝策略的接口，处理的方式是一般我们的消息需要保存下来，要是订单的话那就更需要保存了，保存到kafka，保存到redis或者是存到数据库随便你然后做好日志。

    - ```java
      public class T14_MyRejectedHandler {
          public static void main(String[] args) {
              ExecutorService service = new ThreadPoolExecutor(4, 4,
                      0, TimeUnit.SECONDS, new ArrayBlockingQueue<>(6),
                      Executors.defaultThreadFactory(),
                      new MyHandler());
          }
      
          static class MyHandler implements RejectedExecutionHandler {
              @Override
              public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                  //log("r rejected")
                  //save r kafka mysql redis
                  //try 3 times
                  if(executor.getQueue().size() < 10000) {
                      //try put again();
                  }
              }
          }
      }
      ```

- 见以下代码，定义了一个任务Task，这个任务是实现Runnable接口，每一个任务里有一个编号i，然后打印这个编号，打印完后阻塞System.in.read()，每个任务都是阻塞的，定义一个线程池最长的有七个参数。

```java
public class T05_00_HelloThreadPool {

    static class Task implements Runnable {
        private int i;

        public Task(int i) { this.i = i; }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " Task " + i);
            try { System.in.read(); } catch (IOException e) { e.printStackTrace(); }
        }

        @Override
        public String toString() { return "Task{" + "i=" + i + '}'; }
    }

    public static void main(String[] args) {
        ThreadPoolExecutor tpe = new ThreadPoolExecutor(2, 4,
                60, TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(4),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.CallerRunsPolicy());

        for (int i = 0; i < 8; i++) {
            tpe.execute(new Task(i));
        }

        System.out.println(tpe.getQueue());

        System.out.println(tpe.getQueue()); // 队列中的任务数

        tpe.execute(new Task(100)); // 拒绝策略

        tpe.shutdown();
    }
}
```

### 8、ThreadPoolExecutor —— SingleThreadPool

- 见以下代码，第一个叫SingleThreadPool，这个线程池里面只有一个线程，一个线程的线程池可以保证扔进去的任务是顺序执行的。

- 为什么会有单线程的线程池？线程池是有任务队列的；生命周期管理线程池是能帮你提供的；

- ```java
  public class T07_SingleThreadPool {
  	public static void main(String[] args) {
  		ExecutorService service = Executors.newSingleThreadExecutor();
  		for(int i=0; i<5; i++) {
  			final int j = i;
  			service.execute(()->{				
  				System.out.println(j + " " + Thread.currentThread().getName());
  			});
  		}			
  	}
  }
  ```

### 9、ThreadPoolExecutor —— CachedPool

- 源码实际上是new了一个ThreadPoolExecutor，没有核心线程，最大线程可以有好多好多线程，然后60秒钟没有人理他，就回收了，他的任务队列用的是SynchronousQueue，没有指定他的线程工厂他是用的默认线程工厂的，也没有指定拒绝策略，他是默认拒绝策略的。

- CachedThreadPool的特点，就是你来一个任务我给你启动一个线程，当然前提是我的线程池里面有线程存在而且他还没有到达60秒钟的回收时间的时候，来一个任务，如果有线程存在我就用现有的线程池，但是在有新的任务来的时候，如果其他线程忙我就启动一个新的，

- 正常情况应该是来任务就扔到任务队列里面，可CachedThreadPool他用的任务队列是synchronousQueue，它是一个手递手容量为空的Queue，就是你来一个东西必须得有一个线程把他拿走，不然我提交任务的线程从这阻塞住了。

- synchronousQueue还可以扩展为多个线程的手递手，多个生产者多个消费者都需要手递手叫TransferQueue。这个CachedThreadPool就是这样一个线程池，来一个新的任务就必须马上执行，没有线程空着我就new一个线程。

- 那么阿里是不会推荐使用这种线程池的，原因是线程会启动的特别多，基本接近于没有上限的。

- 见以下程序，首先将这个service打印出来，最后在把service打印出来，我们的任务是睡500个毫秒，然后打印线程池，打印他的名字。运行一下，通过打印线程池的toString的输出能看到线程池的一些状态。

- ```java
  // 源码
  public static ExecutorService newCachedThreadPool() {
          return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                        60L, TimeUnit.SECONDS,
                                        new SynchronousQueue<Runnable>());
      }
  // 程序
  public class T08_CachedPool {
  	public static void main(String[] args) throws InterruptedException {
  		ExecutorService service = Executors.newCachedThreadPool();
  		System.out.println(service);
  		for (int i = 0; i < 2; i++) {
  			service.execute(() -> {
  				try {
  					TimeUnit.MILLISECONDS.sleep(500);
  				} catch (InterruptedException e) {
  					e.printStackTrace();
  				}
  				System.out.println(Thread.currentThread().getName());
  			});
  		}
  		System.out.println(service);
  		
  		TimeUnit.SECONDS.sleep(80);
  		
  		System.out.println(service);		
  	}
  }
  ```

### 10、ThreadPoolExecutor —— FixedThreadpool

- fixed是固定的含义，就是固定的一个线程数，FixedThreadPoo指定一个参数，到底有多少个线程，他的核心线程和最大线程都是固定的，因为他的最大线程和核心线程都是固定的就没有回收之说所以把他指定成0，这里用的是LinkedBlockingQueue（如果在阿里工作看到LinkedBlockingQueue一定要小心，他是不建议用的）

- 用一个固定的线程池有一个好处就是你可以进行并行的计算，

  - 并行和并发有什么区别concurrent vs paralle：并发是指任务提交，并行指任务执行；并行是并发的子集。并行是多个cpu可以同时进行处理，并发是多个任务同时过来。

- FixedThreadPool是确实可以让你的任务来并行处理的，那么并行处理的时候就可以真真正正的提高效率。

- isPrime 方法判断一个数是不是质数，然后写了另外一个getPrime方法，指定一个起始的位置，一个结束的位置将中间的质数拿出来一部分，主要是为了把任务给切分开。计算从1一直到200000这么一些数里面有多少个数是质数getPrime，计算了一下时间，只有一个main线程来运行，

- 不过我们既然学了多线程就完全可以这个任务切分成好多好多子任务让多线程来共同运行，我有多少cpu，这个取决你的机器数，在启动了一个固定大小的线程池，然后在分别来计算，分别把不同的阶段交给不同的任务，扔进去submit他是异步的，拿到get的时候才知道里面到底有多少个，全部get完了之后相当于所有的线程都知道结果了，最后我们计算一下时间，用这两种计算方式就能比较出来到底是并行的方式快还是串行的方式快。

- ```java
  // 源码
  public static ExecutorService newFixedThreadPool(int nThreads) {
          return new ThreadPoolExecutor(nThreads, nThreads,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>());
      }
  // 程序
  public class T09_FixedThreadPool {
  	public static void main(String[] args) throws InterruptedException, ExecutionException {
  		long start = System.currentTimeMillis();
  		getPrime(1, 200000); 
  		long end = System.currentTimeMillis();
  		System.out.println(end - start);
  		
  		final int cpuCoreNum = 4;
  		
  		ExecutorService service = Executors.newFixedThreadPool(cpuCoreNum);
  		
  		MyTask t1 = new MyTask(1, 80000); //1-5 5-10 10-15 15-20
  		MyTask t2 = new MyTask(80001, 130000);
  		MyTask t3 = new MyTask(130001, 170000);
  		MyTask t4 = new MyTask(170001, 200000);
  		
  		Future<List<Integer>> f1 = service.submit(t1);
  		Future<List<Integer>> f2 = service.submit(t2);
  		Future<List<Integer>> f3 = service.submit(t3);
  		Future<List<Integer>> f4 = service.submit(t4);
  		
  		start = System.currentTimeMillis();
  		f1.get();
  		f2.get();
  		f3.get();
  		f4.get();
  		end = System.currentTimeMillis();
  		System.out.println(end - start);
  	}
  	
  	static class MyTask implements Callable<List<Integer>> {
  		int startPos, endPos;
  		
  		MyTask(int s, int e) {
  			this.startPos = s;
  			this.endPos = e;
  		}
  		
  		@Override
  		public List<Integer> call() throws Exception {
  			List<Integer> r = getPrime(startPos, endPos);
  			return r;
  		}
  		
  	}
  	
  	static boolean isPrime(int num) {
  		for(int i=2; i<=num/2; i++) {
  			if(num % i == 0) return false;
  		}
  		return true;
  	}
  	
  	static List<Integer> getPrime(int start, int end) {
  		List<Integer> results = new ArrayList<>();
  		for(int i=start; i<=end; i++) {
  			if(isPrime(i)) results.add(i);
  		}		
  		return results;
  	}
  }
  ```

- **Cache vs Fixed**
  - 什么时候用Cache什么时候用Fixed，你得精确的控制你有多少个线程数，控制数量问题多数情况下你得预估并发量。如果线程池中的数量过多，最终他们会竞争稀缺的处理器和内存资源，浪费大量的时间在上下文切换上，反之，如果线程的数目过少，正如你的应用所面临的情况，处理器的一些核可能就无法充分利用。Brian Goetz建议，线程池大小与处理器的利用率之比可以使用公式来进行计算估算：线程池=你有多少个cpu 乘以 cpu期望利用率乘以（1+ W / C）。W除以C是等待时间与计算时间的比率。
  - 假如你这个任务并不确定他的量平稳与否，就像是任务来的时候他可能忽高忽低，但是我要保证这个任务来时有人做这个事儿，那么我们可以用Cache，当然你要保证这个任务不会堆积。那fixed的话就是这个任务来的比较平稳，我们大概的估算了一个值，就是这个值完全可以处理他，我就直接new这个值的线程来扔在这就ok了。（阿里是都不用，自己估算，进行精确定义）
- **合理分配线程池：**
  - CPU密集型
    - CPU密集的意思是该任务需要大量运算，而没有阻塞，CPU一直全速运行
    - CPU密集任务只有在真正多喝CPU上才可能得到加速（通过多线程）
    - 而在单线程无论你开几个模拟多线程该任务都不可能得到加速，因为CPU总的运算能力是有限的
    - CPU密集型任务配置尽可能少的线程数量
    - 公式：CPU核数+1个线程的线程池
  - IO密集型
    - O密集型，即该任务需要大量的I0，即大量的阻塞。在单线程上运行I0密集型的任会导致浪费大量的CPU运算能力浪费在等待。.所以在I0密集型任务中使用多线程可以大大的加速程序运行，即使在单核CPU上，这种加速主要就是利用了被浪费掉的阻塞时间。
    - 算法：
      - 由于IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如CPU核数*2
      - IO密集型时，大部分线程都阻塞，故需要多配置线程数
      - 参考公式：CPU核数/(1-阻塞系数)
      - 阻塞系数在0.8-0.9之间
      - 比如8核CPU：8/(1-0.9)=80个线程数

### 11、ThreadPoolExecutor —— ScheduledPool

- ScheduledPool定时任务线程池，就是定时器任务，隔一段时间之后这个任务会执行。这个就是我们专门用来执行定时任务的一个线程池。

- 源码中，我们newScheduledThreadPool的时候他返回的是ScheduledThreadPoolExecutor，然后在ScheduledThreadPoolExecutor里面他调用了super，他的super又是ThreadPoolExecutor，它本质上还是ThreadPoolExecutor，所以并不是别的，参数还是ThreadPool的七个参数。这是专门给定时任务用的这样的一个线程池，了解就可以了。

- 见以下程序，newScheduledThreadPool核心线程是4，其实他这里面有一些好用的方法比如是scheduleAtFixedRate间隔多长时间在一个固定的频率上来执行一次这个任务，可以通过这样的方式灵活的对于时间上的一个控制，第一个参数（Delay）第一个任务执行之前需要往后面推多长时间；第二个（period）间隔多长时间；第三个参数是时间单位;

- ```java
  public class T10_ScheduledPool {
  	public static void main(String[] args) {
  		ScheduledExecutorService service = Executors.newScheduledThreadPool(4);
  		service.scheduleAtFixedRate(()->{
  			try {
  				TimeUnit.MILLISECONDS.sleep(new Random().nextInt(1000));
  			} catch (InterruptedException e) {
  				e.printStackTrace();
  			}
  			System.out.println(Thread.currentThread().getName());
  		}, 0, 500, TimeUnit.MILLISECONDS);
  		
  	}
  }
  ```

- 分享一道阿里的面试题：假如提供一个闹钟服务，订阅这个服务的人特别多，10亿人，就意味着在每天早上七点钟的时候会有10亿的并发量涌向你这的服务器，问你怎么优化？
  
  - 思想是把这个定时的任务分发到很多很多的边缘的服务器上去，一台服务器不够啊，每一台服务器上有一个队列存着这些任务，然后线程去消费，也是要用到线程池的，大的结构上用分而治之的思想，主服务器把这些同步到边缘服务器，在每台服务器上用线程池加任务队列

### 12、ThreadPoolExecutor源码解析 

1. #### 常用变量的解释

   - control(ctl)，ctl代表两个意思，其中AtomicInteger是int类型，int类型是32位，高的三位代表线程池状态，低的29位代表目前线程池里有多少个线程数量。

   - 那为什么不用两个值，这里面肯定是自己进行了一些优化的，如果让我们自己写一定是两个值，我们线程池目前是什么状态，然后在这里面到底目前有多少个线程在运行，记录下来，只不过他把这两个值合二为一了，执行效率是会更高一些，因为这两个值都需要线程同步，所以他放在一个值里面，只要对一个线程进行线程同步就可以了，所以这里AtomicInteger在线程数量非常多，执行时间非常短的时候相对于synchronized效率会更高一些，在下面2、3是对ctl的一个计算。4是线程池的一些5种状态

   - RUNNING：正常运行的；

   - SHUTDOWN：调用了shutdown方法了进入了shutdown状态；

   - STOP：调用了shutdownnow马上让他停止；

   - TIDYING：调用了shutdown然后这个线程也执行完了，现在正在整理的这个过程叫TIDYING；

   - TERMINATED：整个线程全部结束；

   - 在下面就是对ctl的一些操作了，runStateO取他的状态，workerCountOfit算有多少个线程正在工作，还有第8和第9个  runStateLessThan   、 runStateAtLeast是帮助写代码的一些东西。

   - ```java
         // 1、ctl 可以看做是一个int类型的数字，高3位表示线程池状态，低29位表示worker数量
         private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
     	// 2、COUNT_BITS, Integer.SIZE 为32， 所以COUNT_BITS为29
         private static final int COUNT_BITS = Integer.SIZE - 3;
     	// 3、CAPACITY 线程池允许的最大线程数。 1左移29位，然后减1，即2^29-1
         private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
     
         // runState is stored in the high-order bits
     	// 4、线程池有5中状态，按大小排序如下： 
     	// RUNNING < SHUTDOWN < STOP < TIDYING < TERMINATED
         private static final int RUNNING    = -1 << COUNT_BITS;
         private static final int SHUTDOWN   =  0 << COUNT_BITS;
         private static final int STOP       =  1 << COUNT_BITS;
         private static final int TIDYING    =  2 << COUNT_BITS;
         private static final int TERMINATED =  3 << COUNT_BITS;
     
         // Packing and unpacking ctl
     	// 5、runStateOf() 获取线程池状态，通过按位与操作，低29位将全部变成0
         private static int runStateOf(int c)     { return c & ~CAPACITY; }
     	// 6、workerCountOf() 获取线程池worker数量，通过按位与操作，高3位将全部变成0
         private static int workerCountOf(int c)  { return c & CAPACITY; }
     	// 7、ctlOf() 根据线程池状态和线程池worker数量，生成ctl值
         private static int ctlOf(int rs, int wc) { return rs | wc; }
     
         /*
          * Bit field accessors that don't require unpacking ctl.
          * These depend on the bit layout and on workerCount being never negative.
          */
     	// 8、runStateLessThan() 线程池状态小于 xx
         private static boolean runStateLessThan(int c, int s) {
             return c < s;
         }
     	// 9、runStateAtLeast() 线程池状态大于等于 xx
         private static boolean runStateAtLeast(int c, int s) {
             return c >= s;
         }
     
         private static boolean isRunning(int c) {
             return c < SHUTDOWN;
         }
     ```

2. #### 构造方法

   ```java
   public ThreadPoolExecutor(int corePoolSize,
                                 int maximumPoolSize,
                                 long keepAliveTime,
                                 TimeUnit unit,
                                 BlockingQueue<Runnable> workQueue,
                                 ThreadFactory threadFactory,
                                 RejectedExecutionHandler handler) {
       	// 基本类型参数校验
           if (corePoolSize < 0 ||
               maximumPoolSize <= 0 ||
               maximumPoolSize < corePoolSize ||
               keepAliveTime < 0)
               throw new IllegalArgumentException();
       	// 空指针校验
           if (workQueue == null || threadFactory == null || handler == null)
               throw new NullPointerException();
           this.corePoolSize = corePoolSize;
           this.maximumPoolSize = maximumPoolSize;
           this.workQueue = workQueue;
       	// 根据传入参数 unit 和 keepAliveTime ，将存活时间转换为纳秒存到变量 keepAliveTime中
           this.keepAliveTime = unit.toNanos(keepAliveTime);
           this.threadFactory = threadFactory;
           this.handler = handler;
       }
   ```

3. #### 提交执行task的过程

   - execute执行任务的时候判断任务等于空，抛异常

   - 接下来就是拿状态值，拿到值之后计算这个值里面的线程数。活着的那些线程数是不是小于核心线程数，如果小于addWorker添加一个线程，addWorker是比较难的一个方法，他的第二个参数指的是，是不是核心线程，所以上来之后如果核心线程数不够先添加核心线程，再次检查这个值。

   - 这个线程里面上来之后刚开始为0，来一个任务启动一个核心线程，第二个就是核心线程数满了之后，放到队列里。最后核心线程满了，队列也满了，启动非核心线程。小于核心线程数就直接加，后面执行的逻辑就是超过核心线程数了直接往队列里扔，workQueue.offer就是把他扔进去队列里，再检查状态。

   - 在这中间可能会被改变状态值，因此需要双重检查，这个跟单例模式里面的DCL是一样的逻辑。isRunning，重新又拿这个状态，拿到这个状态之后这里是要进行一个状态切换的，如果不是Running状态说明执行过shutdown命令，会把这个Running转换成别的状态，其他情况下workerCount() 如果等于0说明线程池里面没有空闲线程了，线程池正常运行就添加非核心线程。这些步骤都是通过源码可以看出来的。如果添加work本身都不行就reject把他给拒绝掉。

   - ```java
     public void execute(Runnable command) {
             if (command == null)
                 throw new NullPointerException();
             /*
              * Proceed in 3 steps:
              *
              * 1. If fewer than corePoolSize threads are running, try to
              * start a new thread with the given command as its first
              * task.  The call to addWorker atomically checks runState and
              * workerCount, and so prevents false alarms that would add
              * threads when it shouldn't, by returning false.
              *
              * 2. If a task can be successfully queued, then we still need
              * to double-check whether we should have added a thread
              * (because existing ones died since last checking) or that
              * the pool shut down since entry into this method. So we
              * recheck state and if necessary roll back the enqueuing if
              * stopped, or start a new thread if there are none.
              *
              * 3. If we cannot queue task, then we try to add a new
              * thread.  If it fails, we know we are shut down or saturated
              * and so reject the task.
              */
             int c = ctl.get();
         	// worker 数量比核心线程数小，直接创建worker执行任务
             if (workerCountOf(c) < corePoolSize) {
                 if (addWorker(command, true))
                     return;
                 c = ctl.get();
             }
         	// worker 数量超过核心线程数，任务直接进入队列
             if (isRunning(c) && workQueue.offer(command)) {
                 int recheck = ctl.get();
                 // 线程池不是 RUNNING状态，说明执行过shutdown命令，需要对新加入的任务执行reject操作
                 // 需要recheck的原因是任务入队列前后，线程池的状态可能会发生变化
                 if (! isRunning(recheck) && remove(command))
                     reject(command);           
                 // 需要判断0值的原因主要是在线程池构造方法中，核心线程数允许为0
                 else if (workerCountOf(recheck) == 0)
                     addWorker(null, false);
             }
         	// 如果线程池不是运行状态，或者任务进入队列失败，则尝试创建worker执行任务
         	// 以下3点需要注意：
         	// 1、线程池不是运行状态时，addWorker内部会拍段线程池状态
         	// 2、addWorker 第2个参数表示是否创建核心线程数
         	// 3、addWorker返回false，则说明任务执行失败，需要执行reject操作
             else if (!addWorker(command, false))
                 reject(command);
         }
     ```

4. #### addWorker源码解析

   - addWorker就是添加线程，线程是要存到容器里，往里头添加线程的时候务必要知道可能有好多个线程都要往里头扔，所以一定要做同步，

   - 由于它要追求效率不会用synchronized，他会用lock或者是自旋也就增加了你代码更复杂的一个程度。

   - 整个addWorker源码做了两步，上面两个for循环只是做了第一步，把worker的数量加1，添加一个worker。

   - 数量在32位的那个29位里面，而且是在多线程的情况下加1，所以他进行了两个死循环干这个事儿，外层死循环套内层死循环，上来先拿状态值，然后进行了一堆的判断，如果状态值不符合的话就return false，这个状态值加不进去。

   - 什么时候这个状态值不符合？就是大于shutdown，说明你已经shutdown了，或者去除上面这些状态之外，所有的状态都可以往里加线程。

   - 加线程又是一个死循环，首先计算当前的wc线程数是不是超过容量了，超过容量就别加了，否则用cas的方式加，如果加成功了说明第一步完成了，就retry把整个全都boreak掉，外层循环内层循环一下全都跳出来了

   - 如果没加成功就get，get完了之后呢重新处理，continue retry，相当于前面在不断的试，一直试到我们把这个数加到1为止。然后，后面才是真真正正的启动这个work，

   - new一个work，这个work被new出来之后启动线程，这个work代表一个线程，其实这个work类里面有一个线程，加锁，是在一个容器里面，多线程的状态是一定要加锁的，锁定后查线程池的状态，因为中间可能被其他线程干掉过，看这个状态是不是shutdown了等等，如果满足往里加的条件，加进去，加完这个线程后启动开始运行，这是addWorker的一个大体逻辑。

   - ```java
     private boolean addWorker(Runnable firstTask, boolean core) {
             retry:
         	// 外层自旋
             for (;;) {
                 int c = ctl.get();
                 int rs = runStateOf(c);
     
                 // Check if queue empty only if necessary.
                 /*
                 这个条件写的比较难懂，以下是调整后的代码，与原来等价
                 (rs > SHUTDOWN) || 
                 (rs == SHUTDOWN && firstTask != null) || 
                 (rs == SHUTDOWN && workQueue.isEmpty())
                 1、线程池状态大于SHUTDOWN时，直接返回false
                 2、线程池状态等于SHUTDOWN，且firstTask不为null时，直接返回false
                 3、线程池状态等于SHUTDOWN，且队列为空，直接返回false
                 */
                 if (rs >= SHUTDOWN &&
                     ! (rs == SHUTDOWN &&
                        firstTask == null &&
                        ! workQueue.isEmpty()))
                     return false;
                 
     			// 内层自旋
                 for (;;) {
                     int wc = workerCountOf(c);
                     // worker数量超过容量，直接返回false
                     if (wc >= CAPACITY ||
                         wc >= (core ? corePoolSize : maximumPoolSize))
                         return false;
                     // 使用CAS的方式增加worker数量，若增加成功，则直接跳出外层循环进入到第二部分
                     if (compareAndIncrementWorkerCount(c))
                         break retry;
                     c = ctl.get();  // Re-read ctl
                     // 线程池状态发生变化，对外层循环进行自旋
                     if (runStateOf(c) != rs)
                         continue retry;
                     // else CAS failed due to workerCount change; retry inner loop
                 }
             }
     
             boolean workerStarted = false;
             boolean workerAdded = false;
             Worker w = null;
             try {
                 w = new Worker(firstTask);
                 final Thread t = w.thread;
                 if (t != null) {
                     final ReentrantLock mainLock = this.mainLock;
                     // worker的添加必须是串行的，因此需要加锁
                     mainLock.lock();
                     try {
                         // Recheck while holding lock.
                         // Back out on ThreadFactory failure or if
                         // shut down before lock acquired.
                         // 这需要重新检查线程池状态
                         int rs = runStateOf(ctl.get());
     
                         if (rs < SHUTDOWN ||
                             (rs == SHUTDOWN && firstTask == null)) {
                             // worker已经调用过了start()方法，则不再创建worker
                             if (t.isAlive()) // precheck that t is startable
                                 throw new IllegalThreadStateException();
                             // worker创建并添加到workers成功
                             workers.add(w);
                             // 更新 largestPoolSize 变量
                             int s = workers.size();
                             if (s > largestPoolSize)
                                 largestPoolSize = s;
                             workerAdded = true;
                         }
                     } finally {
                         mainLock.unlock();
                     }
                     // 启动worker线程
                     if (workerAdded) {
                         t.start();
                         workerStarted = true;
                     }
                 }
             } finally {
                 // worker线程启动失败，说明线程池状态发生了变化（关闭操作被执行），需要进行shutdown相关操作
                 if (! workerStarted)
                     addWorkerFailed(w);
             }
             return workerStarted;
         }
     ```

5. #### 线程池worker任务单元

   - 这后面是work类的一个简单的解释，他的里面包了一个线程，包了一个任务。然后记录着我这个work干过多少个任务了等等。

   - ```java
     private final class Worker
             extends AbstractQueuedSynchronizer
             implements Runnable
         {
             /**
              * This class will never be serialized, but we provide a
              * serialVersionUID to suppress a javac warning.
              */
             private static final long serialVersionUID = 6138294804551838833L;
     
             /** Thread this worker is running in.  Null if factory fails. */
             final Thread thread;
             /** Initial task to run.  Possibly null. */
             Runnable firstTask;
             /** Per-thread task counter */
             volatile long completedTasks;
     
             /**
              * Creates with given first task and thread from ThreadFactory.
              * @param firstTask the first task (null if none)
              */
             Worker(Runnable firstTask) {
                 setState(-1); // inhibit interrupts until runWorker
                 this.firstTask = firstTask;
                 // 这是worker的关键所在，使用了线程工厂创建了一个线程，传入的参数为当前worker
                 this.thread = getThreadFactory().newThread(this);
             }
     
             /** Delegates main run loop to outer runWorker  */
             public void run() {
                 runWorker(this);
             }
     
             // Lock methods
             //
             // The value 0 represents the unlocked state.
             // The value 1 represents the locked state.
     
             protected boolean isHeldExclusively() {
                 return getState() != 0;
             }
     
             protected boolean tryAcquire(int unused) {
                 if (compareAndSetState(0, 1)) {
                     setExclusiveOwnerThread(Thread.currentThread());
                     return true;
                 }
                 return false;
             }
     
             protected boolean tryRelease(int unused) {
                 setExclusiveOwnerThread(null);
                 setState(0);
                 return true;
             }
     
             public void lock()        { acquire(1); }
             public boolean tryLock()  { return tryAcquire(1); }
             public void unlock()      { release(1); }
             public boolean isLocked() { return isHeldExclusively(); }
     
             void interruptIfStarted() {
                 Thread t;
                 if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
                     try {
                         t.interrupt();
                     } catch (SecurityException ignore) {
                     }
                 }
             }
         }
     ```

6. #### 核心线程执行逻辑 —— runworker

   - runwork 是真正启动线程之后是怎么样去执行这个任务的，同样的，加锁。这个比较好玩的是这个work是从AbstractQueuedSynchronizer继承出来的同时实现了Runnable，说明work可以放在线程里运行，与此同时他本身就是一把锁，就可以做同步，

   - 另外，他是可以被线程执行的一个任务，为什么它本身就是一把锁啊，这个work可以认为是等着执行的一个工人，是好多个任务都可以往里面去扔内容的，也就是说会有多线程去访问这个对象的，多线程访问这个对象的时候他干脆就给自己做成了一把锁，就不要自己去定义一个lock了，所以你需要往这个work里面扔任务的时候，指定我这个线程就是你执行的这个钱程的时候，通过work自己去lock就可以了，完全没必要再去new别的lock，所以运行work的时候就先lock住，你要run他就得lock住才能执行，不然别的线程有可能把这个work给占了，下面又是一堆的执行，执行完了之后unlock出来，执行完了之后++。

   - ```java
     final void runWorker(Worker w) {
             Thread wt = Thread.currentThread();
             Runnable task = w.firstTask;
             w.firstTask = null;
         	// 调用unlock() 是为了让外部都可以中断
             w.unlock(); // allow interrupts
             boolean completedAbruptly = true;
             try {
                 /*
                 这是自旋
                 1、如果firstTask不为null，则执行firstTask
                 2、如果firstTask为null，则调用getTask() 从队列获取任务
                 3、阻塞队列的特性就是：当队列为空时，当前线程会被阻塞等待
                 */
                 while (task != null || (task = getTask()) != null) {
                     /*
                     这对worker进行加锁，是为了达到下面的目的：
                     1、降低锁范围，提升性能
                     2、保证每个worker执行的任务时串行的
                     */
                     w.lock();
                     // If pool is stopping, ensure thread is interrupted;
                     // if not, ensure thread is not interrupted.  This
                     // requires a recheck in second case to deal with
                     // shutdownNow race while clearing interrupt
                     // 如果线程池正在停止，则对当前线程进行终端操作
                     if ((runStateAtLeast(ctl.get(), STOP) ||
                          (Thread.interrupted() &&
                           runStateAtLeast(ctl.get(), STOP))) &&
                         !wt.isInterrupted())
                         wt.interrupt();
                     // 执行任务，且在执行前后通过beforeExecute()和afterExecute()来扩展其功能
                     // 这两个方法在当前类里面为空实现
                     try {
                         beforeExecute(wt, task);
                         Throwable thrown = null;
                         try {
                             task.run();
                         } catch (RuntimeException x) {
                             thrown = x; throw x;
                         } catch (Error x) {
                             thrown = x; throw x;
                         } catch (Throwable x) {
                             thrown = x; throw new Error(x);
                         } finally {
                             afterExecute(task, thrown);
                         }
                     } finally {
                         // 帮助gc
                         task = null;
                         // 已完成任务数加一
                         w.completedTasks++;
                         w.unlock();
                     }
                 }
                 completedAbruptly = false;
             } finally {
                 // 自旋操作被退出，说明线程池正在结束
                 processWorkerExit(w, completedAbruptly);
             }
         }
     ```

7. #### 总结

   - work类
     - 这个work他本身是Runnable同时又是AQS，关于AQS这块儿你可以先忽略无所谓，因为用别的方式也能实现。本身是一个Runnable你进来的任务他又用这个Runnable给你包装了一下，为什么又要包装呢，因为它里面有好多的状态需要记录，你原来这个任务里是没有的，另外这个东西必须得在线程里运行，所以呢他用Runnable又给你包装了一次。然后这个work类里面会记录着一个成员变量，这个成员变量是thread。是哪个thread正在执行我这个对象呢，很多个线程会抢，所以这个就是为什么要用AQS的原因。另外呢，在你整个执行的过程之中你也需要加锁，不然的话你别的线程进来，要求你这个work执行其他的任务也是很有可能的，这个时候也需要加锁，因此AQS是需要的。这是这个work类，简单的你就可以把它当成线程类，然后这个线程类执行的是你自己的任务就行了。
   - submit方法
   - 后面是execute方法，三步
     - 第一步：核心线程数不够，启动核心的；
     - 第二步：核心线程够了加队列；第三部：核心线程和队列这两个都满了，非核心线程；
   - addWorker做两件事
     - count先加1；
     - 真正的加进任务去并且start

### 13、WorkStealingPool

- WorkStealingPool是另外一种线程池，核心非常简单

- 原来我们讲的线程池，一个线程的集合然后去另外一个任务的队列里头取任务，取了执行。

- WorkStealing指和原来线程池的区别是：每一个线程都有自己单独队列，所以任务不断往里扔的时候它会在每一个线程的队列上不断的累积，让某一个线程执行完自己的任务之后就回去另外一个线程上面偷，你给我一个拿来我用，所以这个叫WorkStealing。

- 那到底这种这种线程池的方式和我们原来讲的共享同一个任务队列，他们之间有什么好的地方和不好的地方呢？

  - 就原来这种方式呢如果有某一个线程被占了好长好长时间，然后这个任务特别重，一个特别大的任务，其他线程只能空着，他没有办法帮到任务特别重的线程。
  - 但是这种就更加灵活一些，我要是任务特别重的时候，有另外一个任务要清的，没关系，我可以分一点儿任务给你，所以呢这个就是WorkStealing这种Pool。

- 看源码，实际上new了一个ForkoinPool，所以本质上他是一个ForkJoinPool，所以我们只要说清楚这个ForkJoinPool之后这个WorkStealing就大概知道什么意思了。

- ```java
  // 源码
  public static ExecutorService newWorkStealingPool() {
          return new ForkJoinPool
              (Runtime.getRuntime().availableProcessors(),
               ForkJoinPool.defaultForkJoinWorkerThreadFactory,
               null, true);
      }
  // 程序
  public class T11_WorkStealingPool {
  	public static void main(String[] args) throws IOException {
  		ExecutorService service = Executors.newWorkStealingPool();
  		System.out.println(Runtime.getRuntime().availableProcessors());
  
  		service.execute(new R(1000));
  		service.execute(new R(2000));
  		service.execute(new R(2000));
  		service.execute(new R(2000)); //daemon
  		service.execute(new R(2000));
  		
  		//由于产生的是精灵线程（守护线程、后台线程），主线程不阻塞的话，看不到输出
  		System.in.read(); 
  	}
  
  	static class R implements Runnable {
  		int time;
  		R(int t) { this.time = t; }
  		@Override
  		public void run() {			
  			try { TimeUnit.MILLISECONDS.sleep(time); } catch (InterruptedException e) { e.printStackTrace(); }
  			System.out.println(time  + " " + Thread.currentThread().getName());			
  		}
  	}
  }
  ```

### 14、ForkjoinPool

- 见以下程序，ForkJoinPool，它适合把大任务切分成一个一个的小任务去运行，小任务还是觉得比较大，再切，不一定是两个，也可以切成三个四个。切完这个任务执行完了要进行一个汇总，如下图所示，当然也有一些打印输出的任务不需要返回值的，只不过我们很多情况是需要进行一个结果的汇总，子任务汇总到父任务，父任务最终汇总到根任务，最后我们就得到了所有的结果，这个过程叫join，因此这个线程池就叫做ForkyoinPool。

- 那我们怎么样定义这个任务呢？

- 我们原来定义任务的时候是从Runnable来继承，

- 在这里我们一般实现ForkoinPool的时候需要定义成为特定的他的类型，这个类型呢是必须得能进行分叉的任务，所以他定义成是一种特殊类型的任务，这个叫ForkoinTask，

- 但是实际当中这个ForkjoinTask比较原始，我们可以用这个RecursiveAction，这里面有两种，

- 第一种叫RecursiveAction递归，为什么叫递归，是因为我们大任务可以切成小任务，小任务还可以切成小任务，一直可以切到满足我的条件为止，这其中隐含了一个递归的过程，因此叫RecursiveAction，是不带返回值的任务。

- 以下不带返回值的任务这个小程序，new了一个数组，长度为100万，装了很多通过Random来new出来的数，下面要对一堆数进行总和的计算，如果我用单线程来计算可以这样来计算：Arrays.stream(nums).sum()搞定，这是单线程，这个时间会比较长，我们可以进行多线程的计算，就像之前我们写过的FixedThreadPool，现在我们可以用ForkJoinPool来做计算，在计算的时候我要去最小的任务片这个数是不超过5万个数，你就不用在分了。 RecursiveAction是我们的任务，是用来做总和的，由于这里面是把数组进行了分片，所以定义了一个起始的位置和一个结束的位置，然后来进行compute。如果说我们这个数组里面的分片数量要比那个我们定义最小数量少了就是5万个数少了就直接进行计算就行，否则的话中间在砍掉一半，砍完了之后把当前任务在分成两个子任务，然后在让两个子任务进行分叉进行fork。这些任务有自己的一些特点，就是背后的后台线程，所以我需要通过一个阻塞操作让当前的main函数不退出，不然的话他一退出所有线程全退出了，ok，这个是叫做没有返回值的任务。

- 有返回值的任务你可以从RecursiveTask继承，看下面的AddTaskRet方法。

- ```java
  public class T12_ForkJoinPool {
  	static int[] nums = new int[1000000];
  	static final int MAX_NUM = 50000;
  	static Random r = new Random();
  	
  	static {
  		for(int i=0; i<nums.length; i++) {
  			nums[i] = r.nextInt(100);
  		}		
  		System.out.println("---" + Arrays.stream(nums).sum()); //stream api
  	}	
  
  	static class AddTask extends RecursiveAction {
  		int start, end;
  		AddTask(int s, int e) {
  			start = s;
  			end = e;
  		}
  		@Override
  		protected void compute() {
  			if(end-start <= MAX_NUM) {
  				long sum = 0L;
  				for(int i=start; i<end; i++) sum += nums[i];
  				System.out.println("from:" + start + " to:" + end + " = " + sum);
  			} else {
  				int middle = start + (end-start)/2;
  				AddTask subTask1 = new AddTask(start, middle);
  				AddTask subTask2 = new AddTask(middle, end);
  				subTask1.fork();
  				subTask2.fork();
  			}
  		}
  	}
  	
  	static class AddTaskRet extends RecursiveTask<Long> {		
  		private static final long serialVersionUID = 1L;
  		int start, end;		
  		AddTaskRet(int s, int e) {
  			start = s;
  			end = e;
  		}
  		@Override
  		protected Long compute() {			
  			if(end-start <= MAX_NUM) {
  				long sum = 0L;
  				for(int i=start; i<end; i++) sum += nums[i];
  				return sum;
  			} 			
  			int middle = start + (end-start)/2;			
  			AddTaskRet subTask1 = new AddTaskRet(start, middle);
  			AddTaskRet subTask2 = new AddTaskRet(middle, end);
  			subTask1.fork();
  			subTask2.fork();			
  			return subTask1.join() + subTask2.join();
  		}
  		
  	}
  	
  	public static void main(String[] args) throws IOException {
  		/*ForkJoinPool fjp = new ForkJoinPool();
  		AddTask task = new AddTask(0, nums.length);
  		fjp.execute(task);*/
  
  		T12_ForkJoinPool temp = new T12_ForkJoinPool();
  
  		ForkJoinPool fjp = new ForkJoinPool();
  		AddTaskRet task = new AddTaskRet(0, nums.length);
  		fjp.execute(task);
  		long result = task.join();
  		System.out.println(result);		
  		//System.in.read();		
  	}
  }
  ```

- 见以下小程序，底层也是用的ForkjoinPool实现的，也是ForkJoinPool的算法来实现的，就是流式API，本身不难，就是你把一个集合里的内容想象成一个河流一样，一个一个往外流，流到我们这的时候处理一下。流式处理的方式就是大家调各种的对集合里面迭代的需要处理每个元素的时候，这种时候处理起来更方便一些。

- 举例，我们new了一个ArrayList往里面装了10000个数，然后我让这个数进行计算，判断他是不是质数nums.forEach这个是lambda表达式同时也是一个流式处理，forEach就是拿出一个来计算看他是不是一个质数，然后计算一个时间，下面我用的是另外一种，上面是forEach在当前线程里面拿出每一个来，下面用的是paralleStream并行流，并行流的意思是它会把里面并行的来进行处理把这个任务切分成一个个子任务，这个时候里面也是用的ForkyoinPool，两个对比就会有时间差的一个差距，所以在互相之间这个线程不需要同步的时候，你可以用这种并行流来进行处理效率会更高一些，他底层的实现也是ForkjoinPool。

- ```java
  public class T13_ParallelStreamAPI {
  	public static void main(String[] args) {
  		List<Integer> nums = new ArrayList<>();
  		Random r = new Random();
  		for(int i=0; i<10000; i++) nums.add(1000000 + r.nextInt(1000000));
  		
  		//System.out.println(nums);
  		
  		long start = System.currentTimeMillis();
  		nums.forEach(v->isPrime(v));
  		long end = System.currentTimeMillis();
  		System.out.println(end - start);
  		
  		//使用parallel stream api
  		
  		start = System.currentTimeMillis();
  		nums.parallelStream().forEach(T13_ParallelStreamAPI::isPrime);
  		end = System.currentTimeMillis();		
  		System.out.println(end - start);
  	}
  	
  	static boolean isPrime(int num) {
  		for(int i=2; i<=num/2; i++) {
  			if(num % i == 0) return false;
  		}
  		return true;
  	}
  }
  ```

## 九、JMH与Disruptor

### 1、JMH

官网：http://openjdkjava.netprojects/code-tools/mh/

1. #### JMH使用 —— 创建JMH测试

   - jmh-core （jmh的核心）

   - Jmh-generator-annprocess（注解处理包)

   - ```
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <modelVersion>4.0.0</modelVersion>
     
         <properties>
             <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
             <encoding>UTF-8</encoding>
             <java.version>1.8</java.version>
             <maven.compiler.source>1.8</maven.compiler.source>
             <maven.compiler.target>1.8</maven.compiler.target>
         </properties>
     
         <groupId>mashibing.com</groupId>
         <artifactId>HelloJMH2</artifactId>
         <version>1.0-SNAPSHOT</version>
     
         <dependencies>
             <!-- https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-core -->
             <dependency>
                 <groupId>org.openjdk.jmh</groupId>
                 <artifactId>jmh-core</artifactId>
                 <version>1.21</version>
             </dependency>
     
             <!-- https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-generator-annprocess -->
             <dependency>
                 <groupId>org.openjdk.jmh</groupId>
                 <artifactId>jmh-generator-annprocess</artifactId>
                 <version>1.21</version>
                 <scope>test</scope>
             </dependency>
         </dependencies>
     
     
     </project>
     ```

2. #### JMH使用 —— idea安装JMH插件 JMH plugin

   - JMH运行，不应该影响正常程序的执行，最好的方式就是按照官网的说法是命令行的方式，比方说你要测试某一个包里面的类的话你应该把这个类和其他的依赖类打成一个jar包，然后单独的把这个jar包放到某一个机器上，在这个机器上对这个jar包进行微基准的测试，如果对他进行测试的比较好，那说明最后的结果还可以，如果说边开发边进行这种微基准的测试实际上他非常的不准，因为你的开发环境会对结果产生影响。
   - 只不过我们自己开发人员来说平时你要想进行一些微基准的测试的话，你要是每次打个包来进行正规一个从头到尾的测试，完了之后发现问题不对再去重新改，效率太低了。所以要在idea中安装插件。
   - idea安装MH插件：file->Settings->Plugins-~JMH-plugin。

3. #### JMH使用 —— 打开运行注解配置

   - 由于用到了注解，打开运行程序注解配置，注解需要写一个程序得解释，所以要允许JMH能够对注解进行处理：
   - compiler -> Annotation Processors -> Enable Annotation Processing

4. #### JMH使用 —— 定义需要测试的类

   - j见以下程序，并行处理流的一个程序，定义了一个list集合，然后往这个集合里扔了1000个数。写了一个方法来判断这个数到底是不是一个质数。写了两个方法，第一个是用forEach来判断我们这1000个数里到底有谁是质数；第二个是使用了并行处理流，这个forEach的方法就只有单线程里面执行，挨着牌从头拿到尾，从0拿到1000，但是并行处理的时候会有多个线程采用Forkjoin的方式来把里面的数分成好几份并行的进行处理。一种是串行处理，一种是并行处理，都可以对他们进行测试，但需要注意这个基准测试并不是对比测试的，你只是侧试一下你这方法写出这样的情况下他的吞吐量到底是多少，这是一个非常专业的测试的工具。严格的来讲这部分是测试开发专业的。

   - ```java
     package com.mashibing.jmh;
     
     import java.util.ArrayList;
     import java.util.List;
     import java.util.Random;
     
     public class PS {
     
     	static List<Integer> nums = new ArrayList<>();
     	static {
     		Random r = new Random();
     		for (int i = 0; i < 10000; i++) nums.add(1000000 + r.nextInt(1000000));
     	}
     
     	static void foreach() {
     		nums.forEach(v->isPrime(v));
     	}
     
     	static void parallel() {
     		nums.parallelStream().forEach(PS::isPrime);
     	}
     	
     	static boolean isPrime(int num) {
     		for(int i=2; i<=num/2; i++) {
     			if(num % i == 0) return false;
     		}
     		return true;
     	}
     }
     ```

5. #### JMH使用 —— 写单元测试

   - 这个测试类一定要在test package下面，对这个方法进行测试testforEach，很简单我就调用PS这个类的foreach就行了，对它测试最关键的是我加了这个注解@Benchmark，这个是JMH的注解，是要被JMH来解析处理的。

   - 这也是我们为么要把那个Annotation Processing给设置上的原因，非常简单，你只要加上注解就可以对这个方法进行微基准测试了，点击右键直接run。

   - ```jav
     package com.mashibing.jmh;
     
     import org.openjdk.jmh.annotations.Benchmark;
     
     import static org.junit.jupiter.api.Assertions.*;
     
     public class PSTest {
      @Benchmark
      public void testForEach() {
          PS.foreach();
      }
     }
     ```

6. #### 运行测试类报错问题

   - ```java
     ERROR: org.openjdk.jmh.runner.RunnerException: ERROR: Exception while trying to acquire the JMH lock (C:\WINDOWS\/jmh.lock): C:\WINDOWS\jmh.lock (拒绝访问。), exiting. Use -Djmh.ignoreLock=true to forcefully continue.
     	at org.openjdk.jmh.runner.Runner.run(Runner.java:216)
     	at org.openjdk.jmh.Main.main(Main.java:71)
     ```

   - 这个错误是因为JMH运行需要访问系统的TMP目录，解决办法是：
   - 打开RunConfiguration -> Environment Variables -> include system environment viables

7. #### JMH基本概念

   - Warmup：预热，由于JVM中对于特定代码会存在优化（本地化），预热对于测试结果很重要
   - Mesurement：总共执行多少次测试
   - Timeout
   - Threads：线程数，由fork指定
   - Benchmark mode：基准测试的模式
   - Benchmark：测试哪一段代码
   - 官方样例：
     http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/

### 2、Disruptor 介绍

- 按照英文翻译的话，Disruptor应该是分裂、瓦解。这个Disruptor是一个做金融的、做股票的这样一个公司交易所来开发的，为自己来开发的这么一个底层的框架，开发出来之后受到了很多的认可，开源之后，2011年获得Duke奖。如果你想把它用作MQ的话，单机最快的MQ。性能非常的高，主要是它里面用的全都是cas，另外把各种各样的性能开发到了极致，所以他单机支持很高的一个并发。Disruptor不是平时我们学的这个redis、不是平时我们所学的kafka，他可以跟他们一样有类似的用途，但他是单机，redis、kafka也可以用于集群。redis他有这种序列化的机制，就是你可以把它存储到硬盘上或数据库当中是可以的，kafka当然也有，Disruptor没有，Disruptor就是在内存里，Disruptor简单理解就是内存里用于存放元素的一个高效率的队列。
- 主页：http://lmax-exchange.github.io/disruptor/
- 源码：https://github.com/LMAX-Exchange/disruptor
- GettingStarted: https://github.com/LMAX-Exchange/disruptor/wiki/Getting-Started
- api: http://lmax-exchange.github.io/disruptor/docs/index.html
- maven: https://mvnrepository.com/artifact/com.lmax/disruptor
- Disruptor叫无锁、高并发、环形Buffer，直接覆盖（不用清除）旧的数据，降低GC频率，用于生产者消费者模式（如果说按照设计者角度来讲他就是观察者模式）。什么叫观察者模式，想象一下，我们在前面学各种各样的队列的时候，队列就是个容器，好多生产者往里头扔东西，好多消费者从里头往外拿东西。所谓的生产者消费者就是这个意思，为什么我们可以叫他观察者呢，因为这些消费者正在观察着里面有没有新东西，如果有的话我马上拿过来消费，所以他也是一种观察者模式。Disruptor实现的就是这个容器


### 3、Disruptor核心与特点

- Disruptor也是一个队列，和其他队列不一样的是，他是一个环形队列，环形的Buffer。一般情况下我们的容器是一个队列，不管你是用链表实现还是用数组实现的，它会是一个队列，那么这个队列生产者这边使劲往里塞，消费者这边使劲往外拿，但Disruptor的核心是一个环形的buffer。
  - 对比ConcurrentLinkedQueue : 链表实现
  - Disruptor是数组实现的
  - 无锁，高并发，使用环形Buffer，直接覆盖（不用清除）旧的数据，降低GC频率
  - 实现了基于事件的生产者消费者模式（观察者模式）
- 如果我们用ConcurrentLinkedQueue这里面就是一个一个链表，这个链表遍历起来肯定没有数组快，这个是一点。还有第二点就是这个链表要维护一个头指针和一个尾指针，我往头部加的时候要加锁，从尾部拿的时候也要加锁。另外链表本身效率就偏低，还要维护两个指针。关于环形的呢，环形本身就维护一个位置，这个位置称之为sequence序列，这个序列代表的是我下一个有效的元素指在什么位置上，就相当于他只有一个指针来回转。加在某个位置上怎么计算：直接用那个数除以我们整个的容量求余就可以
  - RingBuffer是一个环形队列
  - RingBuffer的序号，指向下一个可用的元素
  - 采用数组实现，没有首尾指针
  - 对比ConcurrentLinkedQueue，用数组实现的速度更快
  - 假如长度为8，当添加到第12个元素的时候在哪个序号上呢？用12%8决定
  - 当Buffer被填满的时候到底是覆盖还是等待，由Producer决定
  - 长度设为2的n次幂，利于二进制计算，例如：12%8 = 12 & (8 - 1)  pos = num & (size -1)
- 当生产者线程生产的特别多，消费者没来得及消费，那在往后覆盖的话怎么办？不会那么轻易的让你覆盖的，是有策略的，生产者生产满了，要在生产一个的话就马上覆盖这个位置上的数了。这时候是不能覆盖的，指定了一个策略叫等待策略，这里面有8中等待策略，分情况自己去用。最常见的是BlockingWait，满了我就在这等着，什么时候你空了消费者来唤醒一下就继续

### 4、Disruptor 开发步骤

- 1、定义Event - 队列中需要处理的元素

- 2、定义Event工厂，用于填充队列

  - 这里牵扯到效率问题：disruptor初始化的时候，会调用Event工厂，对ringBuffer进行内存的提前分配，GC产频率会降低

- 3、定义EventHandler（消费者），处理容器中的元素

- 见以下程序：这是官网的几个辅助程序：LongEvent这个事件里面或者说消息里面装的时一个long值，但这里面可以装任何值，任何类型的都可以往里装，这个long类型的值我们可以指定他set，官网上没有toSting方法，这里写了toString为了打印消息。

- ```java
  public class LongEvent {
      private long value;
  
      public long getValue() {
          return value;
      }
  
      public void setValue(long value) {
          this.value = value;
      }
  
      @Override
      public String toString() {
          return "LongEvent{" +
                  "value=" + value +
                  '}';
      }
  }
  ```

- 然后需要一个EventFactory，就是怎么产生这些个事件。LongEventFactory去实现EventFactiy的接口，重写它的newInstance方法直接new LongEvent。

  - 构建这个环的时候为什么要指定一个产生事件的工厂，直接new这个事件不可以吗？但是有的事件里面的构造方法不让new，产生事件工厂的话可以灵活的指定，这里面也是牵扯到效率的：这里牵扯效率问题，因为Disuptor初始化的时候会调用Event工厂，对ringBuffer进行内存的提高分配，GC频率会降低

- ```java
  public class LongEventFactory implements EventFactory<LongEvent> {
  
      public LongEvent newInstance() {
          return new LongEvent();
      }
  }
  ```

- 之后是 LongEventHandler，Handler就是我拿到这个事件之后该怎么样进行处理，所以这里是消息的消费者。在处理完这个消息之后就记一个数，总共记下来一共处理了多少消息，处理消息的时候默认调用的是onEvent方法，这个方法里面有三个参数，第一个是你要处理的那个消息，第二个是你处理的是哪个位置上的消息，第三个是整体的消息结束没结束，是不是处理完了。你可以判断他如果是true的话消费者就可以退出了，如果是false的话说明后面还有继续消费。

- ```java
  public class LongEventHandler implements EventHandler<LongEvent> {
      /**
       * 
       * @param longEvent
       * @param l RingBuffer的序号
       * @param b 是否为最后一个元素
       * @throws Exception
       */
      public static long count = 0;
      public void onEvent(LongEvent longEvent, long l, boolean b) throws Exception {
          count ++;
          System.out.println("["+Thread.currentThread().getName()+"]"+longEvent+"序号："+l);
      }
  }
  ```

- 所以我们定义了这三个类，关于这三个类在解释一下：我们现在有一个环，然后这个环上每一个位置装LongEvent，通过这个LongEventFactory的newInstance方法来产生LongEvent，当我拿到这个Event之后通过LongEventHandler进行处理。

- 定义好上述辅助类的情况下怎么才能比较有机的结合在一起，让他在Disruptor进行处理？见以下程序，首先把EvenFactory给他初始化了newLongfventFactony，定义这个环是2的N次方1024，然后new一个Disruptor出来，需要指定这么几个参数：factory产生消息的工厂；bufferSize是指定这个环大小到底是多少；defaultThreadFactory线程工厂，指的是当他要产生消费者的时候，当要调用这个消费者的时候他是在一个特定的线程里执行的，这个线程就是通过defaultThreadFactory来产生；

- 当拿到这个消息之后，就用这个LongEventHandler来处理。然后start，当start之后一个环起来了，每个环上指向的这个LongEvent也得初始化好，内存分配好了，整个就安安静静的等待着生产者的到来。

- 看生产者的代码，long sequence =ringBuffer.next()，通过next找到下一个可用的位置，最开始这个环是空的，下一个可用的位置是0这个位置，拿到这个位置之后直接去ringBuffer里面get(O)这个位置上的event。如果说你要是追求效率的极致，你应该是一次性全部初始化好，你get的时候就不用再去判断，如果你想做一个延迟，每次都要做判断是不是初始化了。get的时候就是拿到一个event，这个是我们new出来的默认的，但是我们可以改里面的event.set(值..)，填好数据之后ringBuffer.publish发布生产。

- ```java
  public class Main {
      public static void main(String[] args) {
          //Executor executor = Executors.newCachedThreadPool();
  
          LongEventFactory factory = new LongEventFactory();
  
          //must be power of 2
          int ringBufferSize = 1024;
  
          Disruptor<LongEvent> disruptor = new Disruptor<LongEvent>(factory, ringBufferSize, Executors.defaultThreadFactory());
  
          disruptor.handleEventsWith(new LongEventHandler());
  
          disruptor.start();
  
          RingBuffer<LongEvent> ringBuffer = disruptor.getRingBuffer();
  
          LongEventProducer producer = new LongEventProducer(ringBuffer);
  
          ByteBuffer bb = ByteBuffer.allocate(8);
  
          for(long l = 0; l<100; l++) {
              bb.putLong(0, l);
  
              producer.onData(bb);
  
              try {
                  Thread.sleep(100);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
  
          disruptor.shutdown();
      }
  }
  
  ```

- ```java
  // 官方案例 事件发布模板
  long sequence = ringBuffer.next();  // Grab the next sequence
  try {
      LongEvent event = ringBuffer.get(sequence); // Get the entry in the Disruptor
      // for the sequence
      event.set(8888L);  // Fill with data
  } finally {
      ringBuffer.publish(sequence);
  }
  ```

- Disruptor在后面提供了一些Lambda表达式的写法，为了支持这种写法对整个消息的构建过程做了改进。

- 读下面程序使用translator，就是怎么样构建这个消息，原来我们都是用消息的factory，但是下面这次我们用translator对他进行构建，就是把某一些数据翻译成消息。前面产生event工厂还是一样，然后bufferSize，后面再扔的是DaemonThreadFactory就是后台线程了，new LongEventHandler然后star拿到他的ringBuffer，前面都一样。只有一个地方叫EventTranslator不一样，我们在上述代码中是要写try catch然后把里面的值给设好，相当于把这个值转换成event对象。

- 相对简单的写法，它会把某些值转成一个LongEvent，通过EventTranslator。new出来后实现了translateTo方法，EventTranslator他本身是一个接口，所以你要new的时候你又要实现它里面没有实现的方法

- translateTo的意思是你给我一个Event，我会把这个Event给你填好。ringBuffer publishEvent(translator1)你只要把translator1交个ringBuffer就可以了。这个translator就是为了迎合Lambda表达式的写法（为java8的写法做准备）

- 另外transiator有很多种用法：

  - EventiranslatorOneArg只有带一个参数的EventTranslator。我带有一个参数，这个参数会通过我的transiateTo方法转换成一个LongEvent；
  - EventTranslatoVararg多了去了Nararg就是有好多个值，我把里值全都给你已来最后把结果set到event里面。

- ```java
  //===============================================================
          EventTranslator<LongEvent> translator1 = new EventTranslator<LongEvent>() {
              @Override
              public void translateTo(LongEvent event, long sequence) {
                  event.set(8888L);
              }
          };
  
          ringBuffer.publishEvent(translator1);
  
          //===============================================================
          EventTranslatorOneArg<LongEvent, Long> translator2 = new EventTranslatorOneArg<LongEvent, Long>() {
              @Override
              public void translateTo(LongEvent event, long sequence, Long l) {
                  event.set(l);
              }
          };
  
          ringBuffer.publishEvent(translator2, 7777L);
  
          //===============================================================
          EventTranslatorTwoArg<LongEvent, Long, Long> translator3 = new EventTranslatorTwoArg<LongEvent, Long, Long>() {
              @Override
              public void translateTo(LongEvent event, long sequence, Long l1, Long l2) {
                  event.set(l1 + l2);
              }
          };
  
          ringBuffer.publishEvent(translator3, 10000L, 10000L);
  
          //===============================================================
          EventTranslatorThreeArg<LongEvent, Long, Long, Long> translator4 = new EventTranslatorThreeArg<LongEvent, Long, Long, Long>() {
              @Override
              public void translateTo(LongEvent event, long sequence, Long l1, Long l2, Long l3) {
                  event.set(l1 + l2 + l3);
              }
          };
  
          ringBuffer.publishEvent(translator4, 10000L, 10000L, 1000L);
  
          //===============================================================
          EventTranslatorVararg<LongEvent> translator5 = new EventTranslatorVararg<LongEvent>() {
  
              @Override
              public void translateTo(LongEvent event, long sequence, Object... objects) {
                  long result = 0;
                  for(Object o : objects) {
                      long l = (Long)o;
                      result += l;
                  }
                  event.set(result);
              }
          };
  
          ringBuffer.publishEvent(translator5, 10000L, 10000L, 10000L, 10000L);
  ```

- Lambda的写法：

- ```java
  package com.mashibing.disruptor;
  
  import com.lmax.disruptor.RingBuffer;
  import com.lmax.disruptor.dsl.Disruptor;
  import com.lmax.disruptor.util.DaemonThreadFactory;
  
  public class Main03
  {
      public static void main(String[] args) throws Exception
      {
          // Specify the size of the ring buffer, must be power of 2.
          int bufferSize = 1024;
  
          // Construct the Disruptor
          Disruptor<LongEvent> disruptor = new Disruptor<>(LongEvent::new, bufferSize, DaemonThreadFactory.INSTANCE);
  
          // Connect the handler
          disruptor.handleEventsWith((event, sequence, endOfBatch) -> System.out.println("Event: " + event));
  
          // Start the Disruptor, starts all threads running
          disruptor.start();
  
          // Get the ring buffer from the Disruptor to be used for publishing.
          RingBuffer<LongEvent> ringBuffer = disruptor.getRingBuffer();
  
  
          ringBuffer.publishEvent((event, sequence) -> event.set(10000L));
  
          System.in.read();
      }
  }
  ```

- 生产者默认会有好多种生产方式，默认的是多线程生产者，但是假如你确定你整个程序里头只有一个生产者的话那你还能提高效率，就是在你指定Disruptor生产者的线程的方式是SINGLE，生产者的类型ProducerType。

### 5、Disruptor —— ProducerType生产者线程模式

- ProducerType有两种模式 Producer.MULTI和Producer.SINGLE

- 默认是MULTI，表示在多线程模式下产生sequence

- 如果确认是单线程生产者，那么可以指定SINGLE，效率会提升

- 如果是多个生产者（多线程），但模式指定为SINGLE，会出什么问题呢？

- 假如你的程序里头只有一个生产者还用ProducerMULTI的话，我们对序列来进行多线程访问的时候肯定是要加锁的，所以MULTI里面默认是有锁定处理的，但是假如你只有一个线程这个时候应该吧生产者指定为SINGLE，他的效率更高，因为它里面不加锁。

- 见以下小程序，这里指定的是Producer.SINGLE，但是生产的时候用的是一堆线程，当指定了Producer.SINGLE之后相当于内部对于序列的访问就没有锁了，它会把性能发挥到极致，它不会报错，它会把你的消息静悄悄的覆盖了，因此你要小心一点。这里有50 个线程然后每个线程生产100个数，最后结果正常的话应该是有5000个消费产生。

- ```java
  public class Main04_ProducerType {
      public static void main(String[] args) throws Exception {
          LongEvenFactory factory = new LongEventFactory();
          //'specify the of the ring buffer,must be power of 2.
          int buffersize = 1024;
          //construct the Disruptor
          // oisruptor<tongevent> disruptor = new Disruptor<>(factory,buffersize,Executors.defau1tThreadFactory();
          Disruptor<LongEvent> disruptor = new Disruptor<>(factory, buffersize,
                  Executors.defaultThreadFactory(), ProducerType.SINGLE, new Blockingwaitstrategy());
          //connect the handler
          disruptor.handleEventswith(new LongEventHandler());
          // disruptor.handleEventswith(new LongEventHandler()); //指定多个消费者
          // start the Disruptor, start all threads running
          disruptor.start();
          // get the ring buffer form the Disruptor to be used for publishing.
          RingBuffer<Longevent> ringBuffer = disruptor.getRingBuffer();
  
          // =======================================================================================
  
          final int threadcount = 50;
          CycliBarrier barrier = new CycliBarrier(threadcount);
          ExecutorService service = Executors.newCachedThreadPoo1();
          for (long i = 0; i < threadcount; i++) {
              final long threadNum = i;
              service.submit(O -> {
                  System.out.printf("Thread %s ready to start!\n", threadNum);
                  try {
                      fbarrier.await();
                  } catch (InterruptedException e) {
                      e.printstackTrace();
                  } catch (BrokenBarrierException e) {
                      e.printstackTrace();
                  }
  
                  for (int j = 0; j < 100; j++) {
                      ringBuffer.publishEvent((event, sequence) -> {
                          event.set(threadNum);
                          System.out.printin("生产了" + threadNum);
                      });
                  }
              });
          }
          service.shutdown();
          //disruptor.shutdown();
          TimeUnit.SECONDS.sleep(3);
          System.out.println(LongEventHandler.count);
      }
  }
  
  ```

### 6、Disruptor —— 等待策略 —— WaitStrategy

1. (常用）BlockingWaitStrategy：通过线程阻塞的方式，等待生产者唤醒，被唤醒后，再循环检查依赖的sequence是否已经消费。
2. BusySpinWaitStrategy：线程一直自旋等待，可能比较耗cpu
3. LiteBlockingWaitStrategy：线程阻塞等待生产者唤醒，与BlockingWaitStrategy相比，区别在signalNeeded.getAndSet,如果两个线程同时访问一个访问waitfor,一个访问signalAll时，可以减少lock加锁次数.
4. LiteTimeoutBlockingWaitStrategy：与LiteBlockingWaitStrategy相比，设置了阻塞时间，超过时间后抛异常。
5. PhasedBackoffWaitStrategy：根据时间参数和传入的等待策略来决定使用哪种等待策略
6. TimeoutBlockingWaitStrategy：相对于BlockingWaitStrategy来说，设置了等待时间，超过后抛异常
7. （常用）YieldingWaitStrategy：尝试100次，然后Thread.yield()让出cpu
8. （常用）SleepingWaitStrategy : sleep

### 7、Disruptor —— 消费者异常处理

- 默认：disruptor.setDefaultExceptionHandler()
- 覆盖：disruptor.handleExceptionFor().with()

