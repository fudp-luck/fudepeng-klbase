## 一、Redis介绍及NIO原理

### 1、概论

在刚有计算机的时候，数据可以存在文件里

Linux中有 `grep`、`awk` 等命令，还可以用Java语言等语言写个程序，然后做一个基于这个文件的`I/O` 流读取查找。

但随着文件变大，读写的速度会越来越慢，硬盘的`I/O`称为瓶颈

### 2、常识性问题

在计算机当中，数据是存在磁盘中的，那么以磁盘的维度来讲，有两个指标：第一个是寻址，是毫秒（ms）级的；第二个是带宽，也就是说单位时间可以有多少个字节流过去，多大的数据量流过去，基本上是G或M级别的，比如说有几百兆或1G的带宽速度

内存：内存中有寻址，是纳秒级（ns）的
- 千分之一秒是毫秒；千分之一毫秒是微秒；千分之一微秒是纳秒
- 也就是说在磁盘中获取数据比内存在寻址上慢了10万倍，因为内存数据直接可以放到CPU总线，所以各种数据只要在内存中，在速度上是优于磁盘的

IO Buffer（I/O缓冲区）的成本问题：
- 磁盘中存在磁道和扇区，一个扇区是512个字节，在访问硬盘时，都是以最小粒度，一个扇区512个字节进行寻找，那么一块硬盘动辄1T、2T，其中就会存在很多块512字节，那所需的数据的存放位置就是一个问题
- 当一个区域足够小，那它带来的索引成本就会变大，如果1T硬盘中都是512字节的格子，那么在上层操作系统中就要准备一个索引，这个索引就不是4个字节了，可能需要8个字节或更大才能表示一个很大的区间，来索引出这么多的小格子，所以成本会变大。
- 在格式化磁盘时，存在4k对齐的概念，也就是说在真正使用硬件时，并不是按照512个字节为一次读写量，比如你读1k的数据，硬盘会直接返回4k。在一般磁盘都是默认格式化4k操作系统，无论读多少都是最少4k从磁盘读取

### 3、数据库

#### 【1】数据库的问题

如果数据放在文件里，使用相同的命令查询，随着文件的变大，其查询速度就会变慢，这时访问硬盘就会收到硬盘`I/O`瓶颈的影响，这时目前计算机不可逾越的问题。

#### 【2】数据库做得事情

首先定义了一个 `data page` 的概念，其大小为4k
- 如果有一张表在数据库中，里面有很多行，存到磁盘的时候用了很多4k的小格子，这个4k刚好与磁盘中最少4k读取相对应，当需要读取4k中的某个数据时正好符合磁盘的一次`I/O`，没有浪费
- 这样其实定义成8k或16k也是可以的，定义小了会浪费，定义大了没有关系
- 那么如果之前这个文件中已经存有几万行数据散落在这些4k的小格子中，这时只有这些小格子的话，查询数据的成本复杂度和之前是相同的，因为还是要从第一个4k格子开始读，在读到内存，挨个去找，走的还是全量`I/O`，所以一定会很慢

数据库中的索引，如果数据库只建表没有索引，那么依然没有提速。
- 其实索引也是使用的4k这种存储模型，就是4k格子前面放的是一行行的数据，后面放的是那行面向的某一个列
- 比如身份证这一列，就将身份证这列的数据放到4k前面，每一个身份证号指向的是哪一个data page。所以当数据量变大时，索引肯定也会很多。
- 总之就是数据是用4k格子去存，没有使用索引的话，查询速度还是很慢，如果想提升这个速度，就要建一套索引系统，索引系统变相来说也是一笔数据。

在建关系型数据库时，必须给出schema，就是说必须给出这个表一共有多少列，每列的类型约束。
- 每个列的类型就是字节宽度，比如第一列是`varchar(20)`，那么第一列未来一定会开辟20个字节，这种情况下表里面每一行的数据宽度就定死了，
- 如果向这张表插入一行数据，有10个字段，只给出第1个和第7个，那么剩下的就会用0去开辟，来补充那些字节，
- 这么做的好处在于表中更倾向于行级存储，以行为单位存，这样占位在未来增删改时，不用移动数据，直接用新数据在原有位置复写就可以了

数据和索引都是存在硬盘中的，而内存是速度最快的地方，在真正查询的时候需要在内存中准备一个B+Tree，其树干在内存中
- B+Tree所有的叶子就是这些4k小格子。当查询时只要命中索引，那么这个查询就会走B+Tree树干，最终找到某一个叶子，内存中的树干只存一些区间
- 这样可以充分利用各自的能力，磁盘能存很多东西，内存速度快，这样的一种数据结构可以加速遍历查找的速度，而数据库又是分而治之的存储，所以这时候获取数据速度极快，最终的目的在于减少`I/O`的流量。

### 4、数据库表变大，速度降低问题

随着数据量的变大，表中不止这么几个data page了，表的本身涨到几百万，上亿行，数据量达到1T了，这时检索速度、性能一定会变低

在做增删改操作时，如果表中有索引，就需要修改索引或者调整其位置，也就是说维护索引会使增删改变慢，但是查询速度是否会变慢，分为两个方面：
1. 假设表中数据由100T，硬盘最大容量100T，内存中也刚好存下所有树干，没有溢出问题，当有一条简单查询且where条件能命中索引时，速度依然很快，因为where条件走的还是内存B+Tree。
2. 当并发情况下，很多条查询到达，或者一个复杂查询到达，这时的查询就不是要获取一个data page到内存了，因为数量变大，数据量越大，其能够被很多查询命中的几率就很大，所以这个时候会受到硬盘带宽的影响，速度变慢。假设有一万个查询，每个查询查一个4k，每条查询的条件都不同，刚好又散落到不同的4k上，当这些查询进到服务器时，每个4k都是挨个向内存走的，这时就会有一部分等待前面的那些4k运行结束后才能轮到自己

### 5、解决上述问题

1. 首先存在一个最快的关系型数据库 HANA，是一个内存级别的关系型数据库，但是很贵
2. 其次数据在磁盘和内存中的体积不一样，在磁盘当中是不存在指针的概念的，而且在内存中会启动一些压缩优化的策略
3. 当正常的关系型数据库出现变慢的问题，而HANA的价格太高时，出现了折中的方案，就是缓存

### 6、数据库引擎的排名

- https://db-engines.com/en/ranking
- 数据库的前十名

  ![](F:\Java\images.assets\Redis.assets\数据库排名.png)
- key-value数据库的排名

  ![](F:\Java\images.assets\Redis.assets\keyvalue数据库排名.png)
- 在System中会列出全部的数据库

  ![](F:\Java\images.assets\Redis.assets\全部数据库.png)
- 进入到某个数据库后，会对其有多维度的统计

  ![](F:\Java\images.assets\Redis.assets\redis数据库.png)

### 7、Redis为什么会取代Memcach

1. Memcach中的value没有类型的概念，但redis的value具有类型的概念了，这是两者最本质的区别
2. 当使用没有类型的value时，可以使用json来表达复杂的内容，但当客户端要通过缓存系统取回value中的某一个元素，也就是memcach中存了一个数组，redis中存一个list，其取回元素的成不就会不同。
3. memcach需要返回value中所有数据到客户端，那么当很多客户端都要访问的话，网卡`I/O`就是最大的瓶颈，再者客户端要有自己实现的代码对数据进行解码，并且找到所需元素；
4. 如果使用redis的话，redis server 对其每种类型都有自己的方法，相当于index () 指出了下标索引，可以直接获取到所需元素，这样就规避了memcach的问题

### 8、Redis基于Linux安装

#### 【1】官网地址

- 官网地址：https://redis.`I/O`/
- 建议先查看README.md文件

#### 【2】安装步骤

````shell
 yum install wget 
 cd ~
 mkdir soft
 cd soft
 wget https://download.redis.`I/O`/releases/redis-6.0.9.tar.gz 
 tar xf redis-6.0.9.tar.gz 
 yum install gcc
 make
 make install PREFIX=/opt/soft/redis6  
 vi /etc/profile
	 export REDIS_HOME=/opt/soft/redis6
	 export PATH=$PATH:$REDIS_HOME/bin
 source /etc/profile
 cd /redis-6.0.9/utils
 ./install_server.sh # 可以执行一次或多次
 ps -fe | grep redis
````

1. 解释`make`：make是编译命令相当于javac，它必须要找到make file文件，这个文件是一个编译脚本，其相当于一个跳板，会跳到某一目录下，之后才会真正执行make命令以及带的参数，真正的执行文件在src目录下的makefile

   <img src="F:\Java\images.assets\Redis.assets\Makefile1.png" alt="image-20210106164501090" />

2. 解释`./install_server.sh`：

   1. 一个物理机中可以有多个redis实例，通过port区分

   2. 可执行程序就一份在目录，但内存中未来的多个实例需要各自的配置文件，持久化目录等资源

   3. 脚本会帮助你自动启动

      <img src="F:\Java\images.assets\Redis.assets\redis脚本.png" alt="image-20210106163535739" />

#### （三）安装异常一

- 如果出现了异常，首先要清除 make distclean

<img src="F:\Java\images.assets\Redis.assets\异常1.png" alt="image-20210106164703761" />

```shell
# 解决办法是：升级 gcc
 yum -y install centos-release-scl
 yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
 scl enable devtoolset-9 bash
# 以上方式是一次性的 ，永久性的还要执行一个命令，echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile 
```

#### （四）安装异常二

- 进行到./install_server.sh，出现异常


```shell
This systems seems to use systemd.
Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!
```

```shell
# 解决办法 
vi ./install_server.sh
  
#bail if this system is managed by systemd
#_pid_1_exe="$(readlink -f /proc/1/exe)"
#if [ "${_pid_1_exe##*/}" = systemd ]
#then
#       echo "This systems seems to use systemd."
#       echo "Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!"
#       exit 1
#fi
 
// 然后重新运行 
```

### 9、Redis的效率

- redis是单进程单实例的，当并发来的时候，是如何变的很快的？
- <img src="F:\Java\images.assets\Redis.assets\redis模型.png" alt="image-20210106201015894" />
- 多个客户端访问时通过TCP首先到达kernel（内核），而redis和内核之间使用的是epoll（由内核提供的非阻塞多路复用的系统调用），它可以通过系统调用来遍历哪个客户端链接有数据，从而进行处理
- 因为客户端`I/O`慢，内存数据快，当数据过来时通过epoll放到mmap区，之后redis拿出来进行处理，所以慢慢的epoll中的数据是有顺序的，指的是这里面的数据会一笔一笔的处理，这时就需要在客户端前面进行负载控制，如果要对一个key保证事务的话，就要让这些操作让一个线程发出来

### 10、BIO—— NIO

计算机中有内核，内核可以接得住很多的客户端链接（一个链接就是一个文件描述符(fd)），所有的链接首先到达内核

#### 【1】BIO

- BIO时期只有一个read命令可以读文件描述符，当socket产生的这些文件描述符数据包没到的时候，read命令不能返回，是阻塞状态的，所以CPU并没有时刻处理那些及时到达的数据，会造成资源浪费，线程更多的话，切换线程也是有成本的；BIO时期CPU没有得到充分的利用，所以内核发生了变化

  <img src="F:\Java\images.assets\Redis.assets\BIO.png" alt="image-20210106202709299" />

#### 【2】同步非阻塞NIO

同步非阻塞NIO时期，这时的文件描述符是noneblock非阻塞的，那么就可以用一个线程，其中用while死循环去read，看是否有数据，会返回一个boolen值，如果没有数据就遍历下一个，有数据就直接处理，这个过程是轮询，发生在用户空间；如果有1000个fd，代表着用户进程轮询调用1000次，成本很高，需要解决系统调用问题，内核需要发展

<img src="F:\Java\images.assets\Redis.assets\同步非阻塞NIO.png" alt="image-20210106204202630" />

#### （三）多路复用的NIO

多路复用的NIO，这个时期将发生在用户空间的轮询，放到了内核里面，内核里面多了个系统调用早期称之为select，这时用户空间的线程/进程调的是新的系统调用select，如果这时有1000个fd，就需要把这些fd作为参数都传给select（下图中的readfds）并返回，之后由内核去监控，看这1000个哪些有数据，之后再拿着这些有数据的文件描述符去调read，传参传的是有数据的文件描述符，这个时期相比于之前在系统调用上做的更加精准了，为了减少用户态和内核态的切换

<img src="F:\Java\images.assets\Redis.assets\多路复用NIO.png" alt="image-20210106210346140" />

<img src="F:\Java\images.assets\Redis.assets\select.png" alt="image-20210106205137859" />

- 这里存在的问题是，每次需要先传很多的fd，之后返回，返回后还要找那些fd有数据，再调read，相对复杂
- 在计算机中有内核态和用户态，进程有一块自己的内存空间，里面放着1000个文件描述符，调用内核的时候需要将这些fd传过去，这个过程叫拷贝，当然在传输的时候都希望是零拷贝，而在上述情况中，为了判断fd有没有数据而不断拷贝，所以在决策的环节中文件描述符这样的数据就称为累赘了

#### （四）伪A`I/O`

- 伪A`I/O`，这个时期为了解决上述问题，出现了一个调用 mmap ,使用map做映射，就是内核态和用户态共同开辟出来的共享空间，在传输数据时不用拷贝，直接存到共享空间，双方都可以访问，这个共享空间是通过内核来实现的（mmap）在这个空间中存在红黑树和链表的结构，与上面的区别就是1000个文件描述符可以直接放到共享空间红黑树中，由内核空间查哪些是数据包到达的，之后将这些有数据的fd放到链表中，用户空间就可以直接用链表中的数据调用read

  <img src="F:\Java\images.assets\Redis.assets\NIO2.png" alt="image-20210106220003507" />

  <img src="F:\Java\images.assets\Redis.assets\mmap.png" alt="image-20210106211106767" />

#### （五）零拷贝

- 零拷贝指的是内核空间多了个sendfile的系统调用，sendfile中有输入和输出两个参数，在内核发展出sendfile之前就存在read和write两个系统调用，读取数据的过程就是先读到内核，在通过read读给用户态，再通过write写回来，有了sendfile之后，就只调用这一个

  <img src="F:\Java\images.assets\Redis.assets\零拷贝.png" alt="image-20210106212724891" />

  <img src="F:\Java\images.assets\Redis.assets\sendfile.png" alt="image-20210106212251654" />

- 当mmap和sendfile结合之后就是kafka，从网卡读来数据到内核，之后走到kafka，kafka中会使用mmap，mmap可以挂在到文件，又因为mmap的空间是共享的，所以fafka看到文件数据后会直接触发内核，减少系统调用，减少拷贝，当消费者来拿数据时走到就是sendfile零拷贝，sendfile的输入来自文件，输出来自消费者
  
  - <img src="F:\Java\images.assets\Redis.assets\kafka.png" alt="image-20210106213024888" />
  
- yum install man man-pages  帮助程序，可以查看8类文档
  - man ls
  
    <img src="F:\Java\images.assets\Redis.assets\manls.png" alt="image-20210106204437361" />
  - man 2 read
  
    <img src="F:\Java\images.assets\Redis.assets\man2read.png" alt="image-20210106204644306" />
  - cd /proc/进程号/fd   ll
  
    <img src="F:\Java\images.assets\Redis.assets\帮助程序redis.png" alt="image-20210106204340911" />
  
- JVM中一个线程的成本是1M，如果线程多了，那么会造成调度成本CPU浪费，，并且还存在着内存成本

### 11、单线程处理用户请求的好处

<img src="F:\Java\images.assets\Redis.assets\单进程处理模型.png" alt="image-20210114184219884" style="zoom:80%;" />

- 这是一个顺序性的问题，当在分布式情况下，保证数据一致性就很重要，而顺序性指的就是每个连接内的命令是顺序到达、顺序处理的。
- 例如：
  - 如果多个客户端对server中的key = a进行操作，那么很难保证那个客户端的命令先到达
  - 如果是一个客户端，那么它内部是线性的，在没有使用多线程的情况下，是线程安全的，那么如果对key = a 做创建和删除操作，一定是创建命令会先到达先处理，key的数据可以保证
  - 如果客户端中是多线程，且线程不安全的情况下，就有可能先将删除命令发出去，再发送创建命令。

### 12、Redis启动验证

- 通过 redis-cli 命令直接启动，默认连接6379

- 通过 redis-cli -h 查看帮助信息

  - ```sh
    Usage: redis-cli [OPT`I/O`NS] [cmd [arg [arg ...]]]
      -h <hostname>      Server hostname (default: 127.0.0.1).
      -p <port>          Server port (default: 6379).
      -s <socket>        Server socket (overrides hostname and port).
      -a <password>      Password to use when connecting to the server.
                         You can also use the REDISCLI_AUTH environment
                         variable to pass this password more safely
                         (if both are used, this argument takes precedence).
      --user <username>  Used to send ACL style 'AUTH username pass'. Needs -a.
      --pass <password>  Alias of -a for consistency with the new --user opt`I/O`n.
      --askpass          Force user to input password with mask from STDIN.
                         If this argument is used, '-a' and REDISCLI_AUTH
                         environment variable will be ignored.
      -u <uri>           Server URI.
      -r <repeat>        Execute specified command N times.
      -i <interval>      When -r is used, waits <interval> seconds per command.
                         It is possible to specify sub-second times like -i 0.1.
      -n <db>            Database number.
      -3                 Start sess`I/O`n in RESP3 protocol mode.
      -x                 Read last argument from STDIN.
      -d <delimiter>     Delimiter between response bulks for raw formatting (default: \n).
      -D <delimiter>     Delimiter between responses for raw formatting (default: \n).
      -c                 Enable cluster mode (follow -ASK and -MOVED redirect`I/O`ns).
      --raw              Use raw formatting for replies (default when STDOUT is
                         not a tty).
      --no-raw           Force formatted output even when STDOUT is not a tty.
      --csv              Output in CSV format.
      --stat             Print rolling stats about server: mem, clients, ...
      --latency          Enter a special mode continuously sampling latency.
                         If you use this mode in an interactive sess`I/O`n it runs
                         forever displaying real-time stats. Otherwise if --raw or
                         --csv is specified, or if you redirect the output to a non
                         TTY, it samples the latency for 1 second (you can use
                         -i to change the interval), then produces a single output
                         and exits.
      --latency-history  Like --latency but tracking latency changes over time.
                         Default time interval is 15 sec. Change it using -i.
      --latency-dist     Shows latency as a spectrum, requires xterm 256 colors.
                         Default time interval is 1 sec. Change it using -i.
      --lru-test <keys>  Simulate a cache workload with an 80-20 distribut`I/O`n.
      --replica          Simulate a replica showing commands received from the master.
      --rdb <filename>   Transfer an RDB dump from remote server to local file.
      --pipe             Transfer raw Redis protocol from stdin to server.
      --pipe-timeout <n> In --pipe mode, abort with error if after sending all data.
                         no reply is received within <n> seconds.
                         Default timeout: 30. Use 0 to wait forever.
      --bigkeys          Sample Redis keys looking for keys with many elements (complexity).
      --memkeys          Sample Redis keys looking for keys consuming a lot of memory.
      --memkeys-samples <n> Sample Redis keys looking for keys consuming a lot of memory.
                         And define number of key elements to sample
      --hotkeys          Sample Redis keys looking for hot keys.
                         only works when maxmemory-policy is *lfu.
      --scan             List all keys using the SCAN command.
      --pattern <pat>    Keys pattern when using the --scan, --bigkeys or --hotkeys
                         opt`I/O`ns (default: *).
      --intrinsic-latency <sec> Run a test to measure intrinsic system latency.
                         The test will run for the specified amount of seconds.
      --eval <file>      Send an EVAL command using the Lua script at <file>.
      --ldb              Used with --eval enable the Redis Lua debugger.
      --ldb-sync-mode    Like --ldb but uses the synchronous Lua debugger, in
                         this mode the server is blocked and script changes are
                         not rolled back from the server memory.
      --cluster <command> [args...] [opts...]
                         Cluster Manager command and arguments (see below).
      --verbose          Verbose mode.
      --no-auth-warning  Don't show warning message when using password on command
                         line interface.
      --help             Output this help and exit.
      --vers`I/O`n          Output vers`I/O`n and exit.
    
    Cluster Manager Commands:
      Use --cluster help to list all available cluster manager commands.
    
    Examples:
      cat /etc/passwd | redis-cli -x set mypasswd
      redis-cli get mypasswd
      redis-cli -r 100 lpush mylist x
      redis-cli -r 100 -i 1 info | grep used_memory_human:
      redis-cli --eval myscript.lua key1 key2 , arg1 arg2 arg3
      redis-cli --scan --pattern '*:12345*'
    
      (Note: when using --eval the comma separates KEYS[] from ARGV[] items)
    
    When no command is given, redis-cli starts in interactive mode.
    Type "help" in interactive mode for informat`I/O`n on available commands
    and settings.
    ```

- 在Redis中默认准备了16个库（0 - 15），在连接的时候可以通过 -n 来指定连接哪一个库，也可以在内部通过 select 8 来指定进入到 8 号库。redis内部进程分为了16个独立的区域，每个区域是隔离的，也就是说在0号库的key，在其他库是看不到的，以下是验证：

  <img src="F:\Java\images.assets\Redis.assets\16号库验证.png" alt="image-20210114194153006" style="zoom:80%;" />

- 在客户端中可以通过 help 命令来查看如何使用

  - help command ，会告诉你命令如何使用

  - help @group ，redis中将命令分组，这条命令可以查看其他命令如何使用，比如 help @string

  - 可以使用Tab键，后续命令会补全

    ```shell
    redis-cli 6.0.9
    To get help about Redis commands type:
          "help @<group>" to get a list of commands in <group>
          "help <command>" for help on <command>
          "help <tab>" to get a list of possible help topics
          "quit" to exit
    
    To set redis-cli preferences:
          ":set hints" enable online hints
          ":set nohints" disable online hints
    Set your preferences in ~/.redisclirc
    
    127.0.0.1:6379> help COMMAND
    
      COMMAND -
      summary: Get array of Redis command details
      since: 2.8.13
      group: server
    ```
  

## 二、Redis的数据类型 —— String

![image-20210114201454138](F:\Java\images.assets\Redis.assets\string模型.png)

- string类型其实要想象成byte，其中有关于字符串、字符数组、数值、BitMap等命令操作，可以通过 help @string查看

### 1、有关String类型的命令

| 命令                                                         | summary总结                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| APPEND key value                                             | 如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值的末尾 |
| BITCOUNT key [start end]                                     | 在字符串中计数1（bit）出现的次数                             |
| BITFIELD key  [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP\|SAT\|FAIL] | 对字符串执行任意位域整数操作                                 |
| BITOP operat`I/O`n destkey key [key ...]                     | 触发二进制位的逻辑操作，与、或、非、异或                     |
| BITPOS key bit [start] [end]                                 | 寻找给定二进制位在字节中出现的位置                           |
| DECR key                                                     | 将 key 中储存的数字值减一                                    |
| DECRBY key decrement                                         | key 所储存的值减去给定的值（decrement）                      |
| GET key                                                      | 获取指定 key 的值                                            |
| GETBIT key offset                                            | 对字符串值，获取指定偏移量上的位(bit)                        |
| GETRANGE key start end                                       | 返回 key 中字符串值的子字符                                  |
| GETSET key value                                             | 将给定 key 的值设为 value ，并返回 key 的旧值                |
| INCR key                                                     | 将 key 中储存的数字值增一                                    |
| INCRBY key increment                                         | 将 key 所储存的值加上给定的增量值                            |
| INCRBYFLOAT key increment                                    | 将 key 所储存的值加上给定的浮点增量值                        |
| MGET key [key ...]                                           | 获取所有(一个或多个)给定 key 的值                            |
| MSET key value [key value ...]                               | 同时设置一个或多个 key-value 对                              |
| MSETNX key value [key value ...]                             | 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在 |
| PSETEX key milliseconds value                                | 类似SETEX ，以毫秒为单位设置 key 的生存时间                  |
| SET key value [EX seconds\|PX milliseconds\|KEEPTTL] [NX\|XX] | 设置指定 key 的值，nx只能设置不存在的k，xx只能设置存在的key  |
| SETBIT key offset value                                      | 对字符串值，设置或清除指定偏移量上的位                       |
| SETEX key seconds value                                      | 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位) |
| SETNX key value                                              | 只有在 key 不存在时设置 key 的值                             |
| SETRANGE key offset value                                    | 用 value 参数覆写给指定 key 所储存的字符串值，从偏移量 offset 开始 |
| STRALGO LCS algo-specific-argument [algo-specific-argument ...] | 对字符串运行算法(当前的LCS)                                  |
| STRLEN key                                                   | 返回 key 所储存的字符串值的长度                              |

### 2、演示与解析 —— 字符串与数值

#### 【1】set [nx] [xx]  、mset 、 get 、 mget

<img src="F:\Java\images.assets\Redis.assets\演示1.png" alt="image-20210115211451563" style="zoom: 67%;" />

- set key value [nx] 可以用于分布式锁的时候，当很多个客户端，有很多个链接都要对一个单线程的redis发起 set k1 nx ，那么哪个链接成功了，哪个客户端就拿到了锁，其它的全部返回失败，因为这个链接首先创建了 k1

#### 【2】append 、 getrange 、 setrange 、 strlen

<img src="F:\Java\images.assets\Redis.assets\演示2.png" alt="image-20210115214057382" />

- value值中有正反向索引的概念，所以既可以正想 0 - 5取值，也可以返向 -1- -5取值，也可取0 - -1

  <img src="F:\Java\images.assets\Redis.assets\正反向索引.png" alt="image-20210115214714019" />

- setrange中要给出offset偏移量，比如上述案例给出的是6，那么就会从下标为6，第7个位置开始覆盖为新传入的value值，超出原有value值长度的部分也会覆盖进去，做一个扩充

#### （三）incr 、decr 、incrby 、decrby

<img src="F:\Java\images.assets\Redis.assets\演示3.png" alt="image-20210115221417407" />

#### （四）getset

<img src="F:\Java\images.assets\Redis.assets\演示6.png" alt="image-20210116123347143" />

- getset 命令，在设置新值的时候，把旧值返回
- 如果没有这个命令的情况下，需要先get k1，再set k1，这时要考虑成本问题，这两个操作等于在通信时发了两个包，两次的I/O请求，
- 如果是getset的话，等于在通信时发送了一个命令，减少了一次I/O过程

### 3、二进制安全演示

#### 【1】type key命令

1. `type key`命令，描述key对应的value的类型，每种value类型都会有响应的方法，这些方法和类型是绑定的。

2. 如果从客户端发出非使用类型的方法，想操作这个类型key的话，在其内部不必发生实际的操作，只需要用客户端发出方法对应的类型进行匹配，若果不匹配直接返回错误

   <img src="F:\Java\images.assets\Redis.assets\演示4.png" alt="image-20210115223442135" style="zoom: 67%;" />

#### 【2】object encoding key命令

1. `object encoding key`命令，可以通过 help object 查看，encoding是object后面的子命令，描述value值的编码。

2. 比如`set k1 9 `中k1的编码是 int ，因为value的string类型中存在数值操作。

   <img src="F:\Java\images.assets\Redis.assets\演示5.png" alt="image-20210115223514358" />

#### （三）二进制安全演示

- 当 k1 = hello ，k1 的长度为5

  <img src="F:\Java\images.assets\Redis.assets\二进制安全演示1.png" alt="image-20210116113713881" />
- 当 k2 = 9 ，k2 的长度为1，此时 k2 的编码为 int

  <img src="F:\Java\images.assets\Redis.assets\二进制安全演示2.png" alt="image-20210116114048020" />
- 当向k2追加 ‘999’，k2 的长度为4，编码为raw，此时依然可以对 k2 做 incr 数值操作，操作后，k2 的编码为 int

  <img src="F:\Java\images.assets\Redis.assets\二进制安全演示3.png" alt="image-20210116114305773"/>
- 当 k3 = a，长度为1，当向 k3 追加 ‘中’ 字，其长度变为 4

  <img src="F:\Java\images.assets\Redis.assets\二进制安全演示4.png" alt="image-20210116114448221"  />
- int 类型用字节表示范围是 -127 - 127，且在不同语言中的宽度不同，比如在Java中 int 类型需要开辟 4 个字节

### 4、二进制安全

1. 当客户端访问 redis 时，从socket中得到的是字节流，并非字符流，也就是说未来双方只要有统一的编解码，数据就不会被破坏，所以称为二进制安全
  - 如果有不同的客户端，且每个客户端的语言不同，那么它们对于 int 类型的宽度理解不同，有可能发生节段溢出的错误，就像在多语言环境开发时，更倾向于使用 json 或 xml 这种文本表示数据的方式来交互，而不适用序列化，这个时候就需要自己加一个编码器和解码器
  - 如果编码器和解码器不一样的情况，redis 中定义 int 为4个字节，客户端认为 int 为 2 个字节，客户端取出数据之后就会溢出，所以这时 redis 作为核心的中间者，只取字节流，保证二进制安全
2. redis中首先会对encoding编码状态做判断和预存，判断是 raw 、embstr 、int
3. 在redis数值计算中，首先要将这个字节从内存中取出，先转换成数值，之后更新 key 上的encoding编码0
  - 这一系列的操作，方便于下一次如果是数值操作的情况，可以不用重新判定其encoding，且可以规避报错
  - 编码并不会影响数据的存储，且增加速度

### 5、客户端编码集

- ‘中’ 字在redis中的长度为3，是因为客户端的编码集为 UTF-8

  <img src="F:\Java\images.assets\Redis.assets\编码集演示1.png" alt="image-20210116122246891" />
- 当客户端编码集为 GBK 时， ‘中’ 字的长度为2

  ![image-20210116122626171](F:\Java\images.assets\Redis.assets\编码集演示2.png)
- 当调用 redis-cli --raw 命令，会触发格式化，如果不使用 --raw ，redis只会识别ASCII码，直接按照16进制显示；redis在UTF-8下的 ‘中’ 字在GBK编码下是 ‘涓’，就是所谓的乱码

  <img src="F:\Java\images.assets\Redis.assets\编码集演示3.png" alt="image-20210116123145835" />

### 6、演示与解析 —— BitMap（位图）

#### 【1】位图原理

![image-20210116172611852](F:\Java\images.assets\Redis.assets\位图模型.png)

- 当使用位命令时，二进制位也有索引，它的索引是从左到右的，0 - 7 表示第一个字节的索引， 8 - 15 表示第二个字节的索引，虽然每个字节是割裂开的，但对于redis里面二进制位来说，它的索引是一长串。
- 二进制的value，要么是 0 ， 要么是 1。
- 字符集的标准是 ASCII 码 ，其他一概称为扩展字符集，就是其他字符集对 ASCII 码重编码
-  ASCII 码的字节的第一位必须是 0 （0xxxxxxx），后面可以从全 0 到全 1，代表不同的东西

#### 【2】setbit offset key vlaue 命令

<img src="F:\Java\images.assets\Redis.assets\位图演示1.png" alt="image-20210116172217252" />

- 当 `setbit k1 1 1` ，表示的是对下标为 1 的位的值改为 1 ，位显示为 01000000 ，其结果在 ASCII 码中表示 ‘@’，k1 此时的长度为 1，因为有 1 个字节，
- 当 `setbit k1 7 1` ，表示的是对下标为 7 的位的值改为 1，位显示为 01000001 ，其结果在ASCII 码中表示 ‘A’，k1 此时的长度依然为1，因为偏移量为 7 并没有超过第一个字节的范围
- 当` setbit k1 9 1` ，表示的是对下标为 9 的位的值改为1 ，位显示为 01000001 01000000 ，其结果在 ASCII 码中表示 ‘A@’ ，k1此时的长度为 2 ，因为在偏移量为 9 设置值，已经超过了第一个字节的范围，扩到了第二个字节

#### （三）bitpos key bit start end 命令

<img src="F:\Java\images.assets\Redis.assets\位图演示3.png" alt="image-20210116185903253" />

- bitpos 命令寻找指定二进制位在字符串中存在的第一个位置
- start end ， 指的是字节索引区间

#### （四）bitcount key start end 命令

<img src="F:\Java\images.assets\Redis.assets\位图演示4.png" alt="image-20210116190414105" />



- bitcount 命令统计字节区间内 1 出现的次数

#### （五）bitop operat`I/O`n destkey key ... 命令

<img src="F:\Java\images.assets\Redis.assets\位图演示5.png" alt="image-20210116191335871" />

![image-20210116192042356](F:\Java\images.assets\Redis.assets\位图演示6.png)

- 按位与操作：有 0 则 0 ，全 1 为 1
- 按位或操作：有 1 则 1 ，全 0 为 0

### 7、BItMap应用场景

#### 【1】场景一及方案

1. 假设在用户系统中，统计未来用户的的登录天数，且窗口随即，比如在电商网站中，需要统计 9月1日前一周到9月1日后一周，这14天的用户登录天数，或者双11的前后进行统计
2. 场景一解决方案一，mysql：
   1. 可以使用 MySQL 创建一张用户登录表，用户的每一笔登录操作都在里面插入一条记录，并记录用户ID以及登录时间
   2. 成本复杂度：关系型数据库中，用户每笔登录插入库中的记录中，日期需要准备 4 个字节，ID 需要准备 4 个字节，那么此时每笔记录就要消耗 8 个字节，如果每个人一年登录 200 天，且有 100w 或 1000w 用户时，想要查询随机窗口数据
3. 场景一解决方案二，redis bitmap：
   1. 计算出每次登录的日期为今年的第几天，依次使用 setbit 存到位图相应位置，一个位代表一天，这样一个用户全年只占50个字节，1000w个用户只占477M
   2. 若按照月份存入，比如 2 月份 28 天，占用 28 位，4 个字节，多出来的位数占 0
   3. 如果取随即窗口，可以按天所在的位取出相应的字节，由程序干预，进行取值

#### 【2】场景二及方案

1. 统计网站活跃用户，比如 618 当天有多少人，或者 1号 到 3号 有多少人
5. 场景二解决方案：
   - 可以使所有用户的 ID 映射在二进制位上，每个用户占用一位
   - 比如小红代表下标为 1，小明下标为 2，计算 2021 年 1 月 1 日 到 2 日的活跃用户，则 setbit 20210101 1 1、 setbit 20210102 2 1，也就是说以日期作为 key
   - 统计这两天的活跃用户，bittop or destkey 20210101 20210102  ，bitcount destkey 0 -1

## 三、Redis的数据类型 —— List

![image-20210117183821401](F:\Java\images.assets\Redis.assets\list类型模型.png)

- redis的key中存在 head 头指针和 tail 尾指针，
- head 头指针指向 value 链表的第一个元素，tail 尾指针指向 value 链表的最后一个元素，操作时可以通过这两个指针快速访问第一个和最后一个元素

### 1、有关List类型的命令

| 命令                                                         | summary总结                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| BLPOP key [key ...] timeout                                  | 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| BRPOP key [key ...] timeout                                  | 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| BRPOPLPUSH source destinat`I/O`n timeout                     | 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| LINDEX key index                                             | 通过索引获取列表中的元素                                     |
| LINSERT key BEFORE\|AFTER pivot element                      | 在列表的元素前或者后插入元素                                 |
| LLEN key                                                     | 获取列表长度                                                 |
| LPOP key                                                     | 移出并获取列表的第一个元素                                   |
| LPOS key element [RANK rank] [COUNT num-matches] [MAXLEN len] | 返回列表中匹配元素的索引                                     |
| LPUSH key element [element ...]                              | 将一个或多个值插入到列表头部                                 |
| LPUSHX key element [element ...]                             | 将一个值插入到已存在的列表头部                               |
| LRANGE key start stop                                        | 获取列表指定范围内的元素                                     |
| LREM key count element                                       | 移除列表元素                                                 |
| LSET key index element                                       | 通过索引设置列表元素的值                                     |
| LTRIM key start stop                                         | 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。 |
| RPOP key                                                     | 移除列表的最后一个元素，返回值为移除的元素。                 |
| RPOPLPUSH source destinat`I/O`n                              | 移除列表的最后一个元素，并将该元素添加到另一个列表并返回     |
| RPUSH key element [element ...]                              | 在列表中添加一个或多个值                                     |
| RPUSHX key element [element ...]                             | 为已存在的列表添加值                                         |

### 2、演示与解析

#### 【1】Lpush 、Rpush 、Lpop 、Rpop

<img src="F:\Java\images.assets\Redis.assets\list演示1.png" alt="image-20210119194158376" />

![](F:\Java\images.assets\Redis.assets\list解析图1.png)

- Lpush中插入元素，从左插入 a 后，再插入 b，key 的 head 指针就可以找到 a，并在其左侧插入 b，之后 head 指向 b，b 指向 a ，Rpush 与其相反
- Lpop 从左侧依次弹出元素，Rpop 从右侧依次弹出元素
- 以上，使用同向命令时（Lpush & Lpop，Rpush & Rpop），等同于对 list 做栈的操作，先进后出，后进先出
- 以上，使用返向命令时（Lpush & Rpop，Rpush & Lpop），等于于对 list 做队列的操作，先进先出

#### 【2】Lrange key start stop

<img src="F:\Java\images.assets\Redis.assets\list演示2.png" alt="image-20210119201745455" />

- 在 list 中有正反向索引的概念，所以 Lrange 取索引指定范围内元素时，可以使用 Lrange 0 -1

#### （三）Lindex key index

<img src="F:\Java\images.assets\Redis.assets\list演示3.png" alt="image-20210119201941705" />

- Lindex 可以取出指定索引的元素

#### （四）Lset key index element

<img src="F:\Java\images.assets\Redis.assets\list演示4.png" alt="image-20210119202137530" />

- Lset 可以通过索引设置列表元素的值
- 此时的 list 相当于数组，因为数组是频繁使用下标的

#### （五）Lrem key count element

<img src="F:\Java\images.assets\Redis.assets\list演示5.png" alt="image-20210119204049536" />

- 在 list 类型中，链表元素时不去重的
- count 分为正数、负数、0 三种情况：
  - 为正数时，在链表中从左 head 头，向右依次找次数个
  - 为负数时，在链表中从右 tail 尾，向左依次找次数个
  - 为 0 时 ，移除列表中所有指定元素

#### （六）Linsert key before|after pivot element  |  llen

<img src="F:\Java\images.assets\Redis.assets\list演示6.png" alt="image-20210120100919224" />

- pivot 表示的是确切的元素，而非索引。
- linsert k1 after a 1 ，表示在 a 前面插入 1
- llen 表示 list 的长度

#### （七）blpop 、brpop 

<img src="F:\Java\images.assets\Redis.assets\list演示7.png" />

<img src="F:\Java\images.assets\Redis.assets\list演示8.png" />

- blpop 、brpop ，表示阻塞弹出元素，timeout 表示超时时间，超时时间为 0 时表示一直阻塞
- 上述案例中，当两个客户端阻塞时，第三个客户端发送消息，只能一个一个接收数据
- 综上，list 还支持一个简单的阻塞队列，阻塞的单播队列

#### （八）Ltrim key start end

<img src="F:\Java\images.assets\Redis.assets\list演示9.png" alt="image-20210120102738213" />

- 会对给出的 start end 的两端删除

## 四、Redis的数据类型 —— Hash

![image-20210120104612252](F:\Java\images.assets\Redis.assets\hash类型模型.png)

### 1、有关Hash类型的命令

| 命令                                           | summary总结                                           |
| ---------------------------------------------- | ----------------------------------------------------- |
| HDEL key field [field ...]                     | 删除一个或多个哈希表字段                              |
| HEXISTS key field                              | 查看哈希表 key 中，指定的字段是否存在。               |
| HGET key field                                 | 获取存储在哈希表中指定字段的值。                      |
| HGETALL key                                    | 获取在哈希表中指定 key 的所有字段和值                 |
| HINCRBY key field increment                    | 为哈希表 key 中的指定字段的整数值加上增量 increment   |
| HINCRBYFLOAT key field increment               | 为哈希表 key 中的指定字段的浮点数值加上增量 increment |
| HKEYS key                                      | 获取所有哈希表中的字段                                |
| HLEN key                                       | 获取哈希表中字段的数量                                |
| HMGET key field [field ...]                    | 获取所有给定字段的值                                  |
| HMSET key field value [field value ...]        | 同时将多个 field-value (域-值)对设置到哈希表 key 中。 |
| HSCAN key cursor [MATCH pattern] [COUNT count] | 迭代哈希表中的键值对。                                |
| HSET key field value [field value ...]         | 将哈希表 key 中的字段 field 的值设为 value 。         |
| HSETNX key field value                         | 只有在字段 field 不存在时，设置哈希表字段的值。       |
| HSTRLEN key field                              | 获取哈希字段值的长度                                  |
| HVALS key                                      | 获取哈希表中所有值。                                  |

### 2、演示与解析

#### 【1】场景

- 当需求面向一个用户，有姓名，年龄等一些属性，如果使用 redis 存储应该如何存储？
- 可以是用 string 类型：set zhangsan::name zhangsan ，set zhangsan::age 18 ,取值时使用 keys zhangsan *，就可以得到 zhangsan 相关的所有 key，客户端就可以循环遍历。但是这种办法成本很高，如果这个用户有一百个字段，那么在取值时可以使用 key * 或者是 more get 多次读取 key ，这其中存在多次对 redis 的通信，且如果使用 keys * 级别的查找，成本是很高的
- 可以使用 hash 来解决问题，hash 中的命令可以理解为是在 string 命令的前面加一个 h，针对上述场景，可以使用 hset zhangsan name zhangsan ，hset zhangsan age 18

#### 【2】hset 、hmset

<img src="F:\Java\images.assets\Redis.assets\hash演示1.png" alt="image-20210120112207010" />

#### （三）hget 、hmget

<img src="F:\Java\images.assets\Redis.assets\hash演示2.png" alt="image-20210120112258735" />

#### （四）hkeys 、hvas 、hgetall

<img src="F:\Java\images.assets\Redis.assets\hash演示3.png" alt="image-20210120112434491" />

#### （五）hincrby 、hincrbufloat

<img src="F:\Java\images.assets\Redis.assets\hash演示4.png" alt="image-20210120113206053" />

- 数值操作可以针对值进行覆盖，也可使用数值操作命令

### 3、应用场景

1. 整合访问总量，降低调用次数：比如客户端打开一个页面，里面关于商品信息有很多字段，接口拿到每一个数据都要访问一次 redis，使用 hash 就可以拿出它所有的 values 返回给客户端
2. 数据实时变化：比如微博中关于个人的关注，点赞，或者商品详情页的浏览次数，被收藏的次数，加入购物车的次数，数据既要查询也要发生计算，这时可以使用 hash 支持数值计算以及统一取回面向一个对象的一批数据

## 五、Redis的数据类型 —— Set

![](F:\Java\images.assets\Redis.assets\set结构图.png)

- list 是有序，可重复的
- set 是无序，不可重复的

### 1、有关Set类型的命令

| 命令                                           | summary总结                                            |
| ---------------------------------------------- | ------------------------------------------------------ |
| SADD key member [member ...]                   | 向集合添加一个或多个成员                               |
| SCARD key                                      | 获取集合的成员数                                       |
| SDIFF key [key ...]                            | 返回第一个集合与其他集合之间的差异                     |
| SDIFFSTORE destinat`I/O`n key [key ...]        | 返回给定所有集合的差集并存储在 destinat`I/O`n 中       |
| SINTER key [key ...]                           | 返回给定所有集合的交集                                 |
| SINTERSTORE destinat`I/O`n key [key ...]       | 返回给定所有集合的交集并存储在 destinat`I/O`n 中       |
| SISMEMBER key member                           | 判断 member 元素是否是集合 key 的成员                  |
| SMEMBERS key                                   | 返回集合中的所有成员                                   |
| SMOVE source destinat`I/O`n member             | 将 member 元素从 source 集合移动到 destinat`I/O`n 集合 |
| SPOP key [count]                               | 移除并返回集合中的一个随机元素                         |
| SRANDMEMBER key [count]                        | 返回集合中一个或多个随机数                             |
| SREM key member [member ...]                   | 移除集合中一个或多个成员                               |
| SSCAN key cursor [MATCH pattern] [COUNT count] | 迭代集合中的元素                                       |
| SUNION key [key ...]                           | 返回所有给定集合的并集                                 |
| SUNIONSTORE destinat`I/O`n key [key ...]       | 所有给定集合的并集存储在 destinat`I/O`n 集合中         |

### 2、演示与解析

#### 【1】sadd 、smembers 、srem

<img src="F:\Java\images.assets\Redis.assets\Set演示1.png" alt="image-20210120134242272" />

#### 【2】交集 sinter 、sinterstore

<img src="F:\Java\images.assets\Redis.assets\Set演示2.png" alt="image-20210120134710676" />

- sinterstore 的出现是为了节省 I/O，当想让 redis 中既有 k1、k2 的全量集，又有交集，可以使用

#### （三）并集 suNIOn 、suNIOnstore

<img src="F:\Java\images.assets\Redis.assets\Set演示3.png" alt="image-20210120135030580" />

#### （四）差集 sdiff 、sdiffstore

<img src="F:\Java\images.assets\Redis.assets\Set演示4.png" alt="image-20210120135352549" />

- 差集分为左差集和右差集，redis 中没有提供方向的概念，所以只能认为定义哪个 key 在前

#### （五）随即事件 srandmember

<img src="F:\Java\images.assets\Redis.assets\Set演示5.png" alt="image-20210120135934467" />

- 当 set 中有 5 个元素时，随即事件中分为以上几种情况
- 当 count 为正数时，会取出一个去重的结果集，数量不能超过已有集
- 当 count 为负数时，会取出一个带有重复元素的结果集，会满足指定的数量
- 如果为 0 时，不返回

### 3、随即事件的应用场景

1. 可以用于抽奖场景，比如现在有 10 个奖品，用户有两种情况，一种是小于奖品数量，一种是大于奖品数量

2. 假设微博粉丝抽奖，准备 3 件礼物，需要准备一个数据集，集中放的是粉丝，如果抽的数为 +3 ，那么就一定会返回 3 个不重复的人，也就是一个人最短只能中一件礼物，如果是 -3 就有可能一个人抽走了两份礼物
3. 假设公司内部抽奖，7 个人分 20 份，那么只要把 count 放大就可以了，SRANDMEMBER key -20，这样就会返回一个重复的数据集
4. 假设公司年会，一般会有一二三等奖，奖品一定是小于人数的，而且在奖池中一次只能抽一个人，且只能交替抽奖，因为环节中会有领导讲话，抽奖后，这个奖品就不会在放回去，所以这时可以用 spop 命令，每次弹出一个，比较符合年会过程

## 六、Redis的数据类型 —— Sorted_Set 

![image-20210120143000551](F:\Java\images.assets\Redis.assets\zset结构图.png)

- sorted set 在使用时有以下几个维度
  - sorted 分值维度，可以根据给定的分值进行排序
  - 当分值的值都为 1 时，则按照元素名称字简序排序
  - 其正序和倒叙，就是索引，每个元素都有自己的正反向索引
- sorted set 中有一些命令是关注元素的，可以给出参数的元素，给出排名，分值；或者参数是分值，给出分值区间，可以取出哪些元素；或者给出排名或索引，可以根据索引给出一些元素，且能带着元素的同时还带着分值。

### 1、有关Sorted_Set的命令

| 命令                                                         | summary总结                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| BZPOPMAX key [key ...] timeout                               | 从一个或多个已排序的集合中移除并返回得分最高的成员，或阻塞直到有一个可用 |
| BZPOPMIN key [key ...] timeout                               | 从一个或多个已排序的集合中移除并返回得分最低的成员，或阻塞直到有一个可用 |
| ZADD key [NX\|XX] [CH] [INCR] score member [score member ...] | 向有序集合添加一个或多个成员，或者更新已存在成员的分数       |
| ZCARD key                                                    | 获取有序集合的成员数                                         |
| ZCOUNT key min max                                           | 计算在有序集合中指定区间分数的成员数                         |
| ZINCRBY key increment member                                 | 有序集合中对指定成员的分数加上增量 increment                 |
| ZINTERSTORE destinat`I/O`n numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM\|MIN\|MAX] | 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 destinat`I/O`n 中 |
| ZLEXCOUNT key min max                                        | 在有序集合中计算指定字典区间内成员数量                       |
| ZPOPMAX key [count]                                          | 移除并返回排序集中得分最高的成员                             |
| ZPOPMIN key [count]                                          | 移除并返回排序集中得分最低的成员                             |
| ZRANGE key start stop [WITHSCORES]                           | 通过索引区间返回有序集合指定区间内的成员                     |
| ZRANGEBYLEX key min max [LIMIT offset count]                 | 通过字典区间返回有序集合的成员                               |
| ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]  | 通过分数返回有序集合指定区间内的成员                         |
| ZRANK key member                                             | 返回有序集合中指定成员的索引                                 |
| ZREM key member [member ...]                                 | 移除有序集合中的一个或多个成员                               |
| ZREMRANGEBYLEX key min max                                   | 移除有序集合中给定的字典区间的所有成员                       |
| ZREMRANGEBYRANK key start stop                               | 移除有序集合中给定的排名区间的所有成员                       |
| ZREMRANGEBYSCORE key min max                                 | 移除有序集合中给定的分数区间的所有成员                       |
| ZREVRANGE key start stop [WITHSCORES]                        | 返回有序集中指定区间内的成员，通过索引，分数从高到低         |
| ZREVRANGEBYLEX key max min [LIMIT offset count]              | 按索引返回排序集中的成员范围，分数从高到低排列               |
| ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count] | 返回有序集中指定分数区间内的成员，分数从高到低排序           |
| ZREVRANK key member                                          | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| ZSCAN key cursor [MATCH pattern] [COUNT count]               | 迭代有序集合中的元素（包括元素成员和元素分值）               |
| ZSCORE key member                                            | 返回有序集中，成员的分数值                                   |
| ZUNIONSTORE destinat`I/O`n numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM\|MIN\|MAX] | 计算给定的一个或多个有序集的并集，并存储在新的 key 中        |

### 2、演示与解析

#### 【1】zadd 、zcount 、zrange 、zrevrange

<img src="F:\Java\images.assets\Redis.assets\ZSet演示1.png" alt="image-20210121141738249" />

- zadd 可以向 set 中添加带有分值的元素
- zcount  key min max ，统计分值最小到最大区间内有多少元素
- zrange key start stop withscores ，按照下标展示 set 中元素，可选是否带有分值，可以取正序前几名的元素
- zrevrange key start stop withscores ，与 zrange 相反，取倒序前几名的元素

#### 【2】zscore 、zrank 、zincrby

<img src="F:\Java\images.assets\Redis.assets\ZSet演示2.png" alt="image-20210121142547215" />

- zscore key member ，取出指定元素的分值
- zrank key member ，取出指定元素的排名
- zincrby key increment member ，可以增加或减少指定元素的分值
- sorted set 会实时根据给定的分值维护元素的顺序，且支持增删改查的操作，可见以下场景

### 3、使用场景

- 实现歌曲排行榜，所有歌曲的分值可以按照下载量、点击量、播放量进行排序，且倒序zrevrange排列取榜单前十名
- 第一天所有歌曲的分值都为 0，当某个人对一个元素播放 1 次，就可以使用 zincrby 对分值加 1，当很多人都对榜单进行点击增加，那当我想看榜单时，redis 会用最快最实时的速度告诉你前十是哪些

### 4、实现原理

#### 【1】权重

- 首先作为 set ，一定会有交并差集的操作，当两个 key 中有相同的元素时，其分值依照哪个为准，取大、取小、取和，这里面还有一个权重的概念

- 在不指定权重和方式时，默认将两个分值相加取和

  <img src="F:\Java\images.assets\Redis.assets\ZSet演示3.png" alt="image-20210121145545356" />
- 在指定权重时（k1 k2 1 0.5），表示将 k2 的分值除以二与 k1 的分值相加

  <img src="F:\Java\images.assets\Redis.assets\ZSet演示4.png" alt="image-20210121145710543" />
- 可以指定处理方式为 max ，取分值中的最大值

  <img src="F:\Java\images.assets\Redis.assets\ZSet演示5.png" alt="image-20210121145808129" />

#### 【2】排序是如何实现的？为什么这么快？

- 其原理为跳表、平衡树

  ![](F:\Java\images.assets\Redis.assets\跳表1.png)

  ![](F:\Java\images.assets\Redis.assets\跳表2.png)

- 跳表也叫平衡树，就是查找任何元素的复杂度都比较平均，那么以上到底增删改查的速度是慢还是快？
  - 当要查找的数值比前几行都要大时，在表中就要查找很多次，还不如直接线性遍历
  - 但在并发发生时，数据量比较多，元素比较多，表比较宽，增删改查操作比较多时，也就是说它的平均值是相对最优的
  - 需要从增删改查 4 个方面综合评定它的效率，不能只看一个，当增删改查操作都发生一边之后，它的平均速度是最稳定的

## 七、Redis的进阶操作

### 1、redis 管道（Pipeline）

#### 【1】管道含义

- 当客户端对服务器中的 redis 进程发送很多命令时：
  - 正常情况每个命令都需要走一次数据的传输，执行后返回再执行第二条
  - 当客户端是我们自己，针对与这一台服务器，就可以把多次请求的这个过程压缩成一笔请求，类似于啤酒理论（假设有 24 瓶酒要拿回家，是一瓶一瓶拿，还是装箱一起拿走）
- 这就是在计算机编程时，使用的 buff 机制，就是为了减少这种没必要的调用，减少系统间的调用，减少网络 I/O

#### 【2】管道操作演示：

<img src="F:\Java\images.assets\Redis.assets\管道演示.png" alt="image-20210121165956300" />

- Linux 管道 ：`yum install nc`
- 组合管道：`echo ；`  \n 表示换行符 ； $2 表示 10 的宽度为 2

#### （三）冷启动：

- 正常情况下 redis 进程启动时是空的，但有时期望它启动后预加载一些数据（可以将进程启动后，写一个程序，抽取库里面哪些热数据；或者请求到达时，从库中拿到再放到缓存里；或者可以直接跑一个程序，将数据写成文件，一次放到进程中），所以这里面会有大量插入数据，或者从文件中批量插入数据。
- 这时，从文件中批量插入数据的话，这里面就会间接的用到管道的功能
- 除了 nc 以外，还开启了 redis-cli-pipe 功能，其实本质就是上述 nc 那种 socket 连接，只不过它要求这个文件需要一个命令，不能直接 \n ，需要 unix2DOS 命令
- 因为 Linux 和 Windows 对换行符的理解不同，Linux 是 \n，Windows 是 \r\n
- 一般冷加载操作是运维方面实操，此处只做了解

### 2、发布订阅

#### 【1】命令

| 命令                                        | summary总结                        |
| ------------------------------------------- | ---------------------------------- |
| PSUBSCRIBE pattern [pattern ...]            | 订阅一个或多个符合给定模式的频道。 |
| PUBLISH channel message                     | 将信息发送到指定的频道。           |
| PUBSUB subcommand [argument [argument ...]] | 查看订阅与发布系统状态。           |
| PUNSUBSCRIBE [pattern [pattern ...]]        | 退订所有给定模式的频道。           |
| SUBSCRIBE channel [channel ...]             | 订阅给定的一个或多个频道的信息。   |
| UNSUBSCRIBE [channel [channel ...]]         | 指退订给定的频道。                 |

- 类似于 list 类型中 blpop、brpop，可以实现阻塞的单波队列

#### 【2】场景一

1. 比如直播的场景，直播的聊天窗口，一个人发消息，所有人都看得到

2. 实现例子：subscribe 只有在监听 k1 通道时，才能收到其他客户端推送的消息

   <img src="F:\Java\images.assets\Redis.assets\发布订阅场景一演示.png" alt="image-20210121182737346" />

#### （三）场景二

1. 比如微信或QQ或腾讯课堂时，除了进入聊天室后能看到新的消息，在向上滑动时，也能看到历史消息
2. 分析及举例：
   - 如果将历史数据放到 mysql 中，那么数据全量可以保证，但当很多人查看时，需要支撑特别多的群，那么多人查询以及翻页功能时，成本就比较高
   - ![](F:\Java\images.assets\Redis.assets\场景2架构模型1.png)
   - 见上图，在程序设计中，依据用户习惯分为实时接受消息，三天内历史记录，以及更老的一些聊天记录。
   - 其中实时消息可以通过发布订阅来实现；
   - 三天内的数据可以使用 sorted_set ，其中有 zremrangebyrank 和 zremrangebyscore 两个操作，前者是根据所有排名给出一个范围，删除多少，那么保留三天数据，其实就是将日期靠前的移除（比如，1-5 号，要保留5、4、3，三天的数据），之后将时间日期作为分值，将消息作为元素存到 redis ，这时放入 sorted set 的数据就已经排好序了，我们只需要在 sorted set 中维护一个维度，要么就保留多少条记录，要么就按照时间保留。
   - 更老的数据全量一定是在数据库中，这时候可能触发更老的数据的需求会降低
   - 在写入数据时，客户端可以单调发布订阅，实现实时消息
   - 同时可以单调一次 sorted set 将三天内的数据写入
   - 并且将更久远的数据通过kafka，逐一写入数据库，防止数据过多对数据库压力过大
   - 以上架构方案容易产生的问题：当单调发布订阅，推送完消息后，写数据的客户端挂机了，这时历史数据没有写入，会导致刷新不到历史数据
   - ![](F:\Java\images.assets\Redis.assets\场景2架构模型2.png)
   - 见上图，可以使用两个 redis 进程，一个做发布订阅，实时通信，另一个订阅第一个进程的消息，并写入 sorted set 以及通过 kafka 写入数据库
3. 总结以上：
   - redis 本身就是内存级别的，无非就是对它连接数的控制，有多个 socket 连接，其实成本并不是很大，因为它并不需要进行密集的 CPU 计算，数据到了直接向订阅的所有 socket 广播出去就可以了

### 3、事务

| 命令                | summary总结                                                  |
| ------------------- | ------------------------------------------------------------ |
| DISCARD             | 取消事务，放弃执行事务块内的所有命令。                       |
| EXEC                | 执行所有事务块内的命令。                                     |
| MULTI               | 标记一个事务块的开始。                                       |
| UNWATCH             | 取消 WATCH 命令对所有 key 的监视。                           |
| WATCH key [key ...] | 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。 |

- 以上述发布订阅来说，有一些单调的操作，那么其实可以把一些单调、中间可能会断开的、希望是源于不可分割的事情，作为事务

- redis 中的事务不向 MySQL 中那么完整，因其本身要追求速度，所以它并没有真正事务锁具备的回滚操作

  ![](F:\Java\images.assets\Redis.assets\事务结构图.png)

  <img src="F:\Java\images.assets\Redis.assets\事务演示1.png" />

- 见上图，client0 先执行事务并发命令 `get k1 & keys *` ，这时 client1 执行事务并发命令 `del k1 & set k2 world`，当   client1 首先执行了 exec 时，client1 发出的命令被先执行，也就是先将 k1 删除后设置了 k2，之后 client0 执行，获取 k1 时返回了 nil ，keys * 返回的是 k2

- watch 可以通过乐观锁实现 CAS 操作，其表达的意思是监控，也就是说在后续事务执行的时候，发现了 watch 所监控的 key 发生了改变，就会将事务直接撤销，至于后续要如何处理，要由客户端自己捕获，要么重新提交，要么回滚等

  <img src="F:\Java\images.assets\Redis.assets\事务演示2.png" alt="image-20210122100856549" />

- redis 事务不支持回滚的优点：

  - Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
  - 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

### 4、布隆过滤器

- 源码地址：https://github.com/RedisBloom/RedisBloom

  ![image-20210122114821134](F:\Java\images.assets\Redis.assets\bloom原理结构图.png)

- 根据上图分析：什么是 bloom 过滤器，就是用小的空间解决大量数据匹配的过程，根本上是 BitMap 的思路

- 如果网站有 10W 种商品，每一种商品的名称是 4 个字节，一共是 40w 个字节，但是如果每种商品只用几个二进制位来表示的话，那么它整体的体积就会变的很小

- 当网站上有一个商品元素，这个元素会经历 n 个哈希函数或映射函数（上图举例为 3 个），那么当这个元素分别算了 n 次映射函数之后，就将 BitMap 中的某个位置标记为 1，另外的商品元素重复刚才的过程，但是这其中可能会出现元素虽然不同，但两个元素计算的某一个函数得出的位置可能会重叠

- 当用户请求的元素访问时，如果这件商品系统中存在，那么就一定会算出为 1 的位置上，如果缓存不存在，就会让请求穿透到数据库上。但是会出现一种情况，就是请求过来的元素在系统中不存在，但通过计算刚好覆盖到为 1 的位置上，那么也会被放行

- 综上所述：bloom 过滤器是拿稍微的一点复杂度或者一点损耗来换取时间成本的，是用概率解决问题，不能够 100% 阻挡用户的恶意请求，但是会让这类请求的穿透概率降低

- 架构思想设计 bloom 有三种方式：

  <img src="F:\Java\images.assets\Redis.assets\bloom三种实现方式.png" alt="image-20210122120919474" />

  - 选用哪一种取决于系统需要的性能和成本
  - 如果所有东西都压在 redis 上，因为它是内存级别的，且对 CPU 的损耗并不大，这样的话可以对客户端更轻量一些，也更符合微服务架构的概念，就是将所有东西都迁出去，自己本身无状态，只放业务代码

- bloom 安装过程
  ```shell
  # 访问 redis.`I/O`  ->  modules ->访问其 git 地址 -> 复制 zip 下载链接
   wget *.zip
   yum install unzip
   unzip *.zip
   make  -> bloom.so
   cp bloom.so /opt/soft/redis6
   redis-server --loadmodule /opt/soft/redis6/bloom.so
   redis-cli
   bf.add ooxx abc ; bf.exits abc ; bf.exits xxxx
   cf.add 布谷鸟过滤器  作为额外了解
  ```

- 什么时候使用，以及注意的要点：
  - 当系统存在穿透，且元素不存在的情况使用
  - 客户端中要增加在 redis 中的 key、value 标记
  - 数据库增加了元素
  - 完成元素对 bloom 的添加

### 5、redis 作为数据库和缓存的区别

- 首先是数据的完整性，缓存数据其实是非全量的数据，全量的数据都存放在数据库中，且缓存应该随着访问变化
- 其次加缓存的目的是为了减轻后端访问的压力，缓存中存放的更多应该是前面请求的内容，所谓的热数据
- 当 redis 用作缓存时，需要尽量降低后端数据库的压力，而且因为内存大小是有限的，是一个瓶颈，如果内存可以无限大，就可以直接用内存当数据库了，无非要考虑持久化，那么 redis 的数据如何根据业务的变化，只保留热数据？

### 6、缓存LRU —— key 的有效期

#### 【1】key 的有效期

- key 的有效期是由业务逻辑来推动的
- 内存有限是有业务运转决定的

#### 【2】场景

1. 场景一：
   1. 如果数据库中有一些数据，需要以天为单位变化，每天 12 点一过，就把昨天的历史交易额写到数据库里，这类数据不会变化，但第二天往往关注的是前一天的数据，而不会关注更久远的数据，所以这些数据在未来请求的时候是有一天有效期的，过了有效期后，这些数据不应该存在内存中
   2. 我们期望这类数据到期就会把它清掉，因为会有新数据，只要发现它不存在了，那么再次请求时就会拿到新的数据来更新，这是由业务逻辑推到的
2. 场景二：因为内存有限，随着访问的变化，应该淘汰掉冷数据，就是有些数据访问了它一次，放到内存里了，但发现未来可能 3 个小时或 10 个小时都不再访问了，应该被清理掉

#### （三）清理方式

- 一类是人为规定的过期时间；
- 一类是人无能为力，因为人也不知道未来运行的时候，谁更应该被清理掉，或者说内存要满了，就要清理掉或者选出几个进行清理

#### （四）内存多大，有没有限制

- 配置文件中可以通过 maxmemory 来设置内存的大小，为 0 时代表无内存限制，单位为字节；
- 当maxmemory限制达到的时候Redis会使用的行为由 Redis的maxmemory-policy配置指令来进行配置。

#### （五）回收策略

1. **noevict`I/O`n**：返回错误当内存限制达到并且客户端尝试执行会让更多内存被使用的命令（大部分的写入指令，但DEL和几个例外）

2. **allkeys-lru**：尝试回收最少使用的键（LRU），使得新添加的数据有空间存放。

3. **volatile-lru**：尝试回收最少使用的键（LRU），但仅限于在过期集合的键,使得新添加的数据有空间存放。

4. **allkeys-random**：回收随机的键使得新添加的数据有空间存放。

5. **volatile-random**：回收随机的键使得新添加的数据有空间存放，但仅限于在过期集合的键。

6. **volatile-ttl**：回收在过期集合的键，并且优先回收存活时间（TTL）较短的键,使得新添加的数据有空间存放。

   ```shell
   # allkeys- 表示所有的 key
   # volatile- 表示那些要过期的 key
   # LRU 算法表示更久没有使用过，强调的是多久没有碰它
   # LFU 算法表示最少使用，碰了多少次
   # 第一个 noevict`I/O`n 不能作为缓存使用，作为数据库时一定要使用，因为数据是不能丢失的
   # 作为缓存时 random 级别的过于随意了，ttl 的时间成本，复杂度太高了，所以一般在 lru 中选择一个
   # 挑选那种 lru 主要看占比，如果这个缓存边大量的做过期、到期时优先使用 volatile-lru；
   # 如果缓存没怎么做过期操作，就是随意的靠着 redis 自己淘汰空出缓存空间，就要选择 allkeys-lru，因为这时淘汰的 key 并不多，所以它释放的量不能达到很多。
   ```

#### （六）演示：

<img src="F:\Java\images.assets\Redis.assets\lru演示1.png" alt="image-20210122215107517" />

- 有效期不会随着访问而延长

  <img src="F:\Java\images.assets\Redis.assets\lru演示2.png" alt="image-20210122215448469" />
- 如果发生写操作，会剔除过期时间
- Expireat 命令用于以 UNIX 时间戳(unix timestamp)格式设置 key 的过期时间。key 过期后将不再可用。

### 7、缓存LRU —— 过期原理

- Redis keys过期有两种方式：被动和主动方式。
- 当一些客户端尝试访问它时，key会被发现并主动的过期。
- 当然，这样是不够的，因为有些过期的keys，永远不会访问他们。 无论如何，这些keys应该过期，所以定时随机测试设置keys的过期时间。所有这些过期的keys将会从密钥空间删除。
- 具体就是Redis每秒10次做的事情：
  1. 测试随机的20个keys进行相关过期检测。
  2. 删除所有已经过期的keys。
  3. 如果有多于25%的keys过期，重复步骤1。
- 这是一个平凡的概率算法，基本上的假设是，我们的样本是这个密钥控件，并且我们不断重复过期检测，直到过期的keys的百分百低于25%,这意味着，在任何给定的时刻，最多会清除1/4的过期keys。

## 八、Redis的持久化

### 1、概述：

#### 【1】为什么需要持久化

- 一般使用 redis + MySQL 的场景，一般会将数据进行双写，使用异步的方式通过队列向 mysql 中写，当 redis 作为缓存时，所追求的是速度快，所以这时的数据可可以丢失的，因为后面还有数据库做全量，无非需要解决缓存挂掉，重启时数据恢复如何做，是否需要恢复的过程，需不需要冷启动和热启动

#### 【2】持久化的两种方式

1. 单机持久化问题的提出是因为掉电易失的特征
2. 只要是存数据的技术，无论是 mysql 或 redis 都会有两类方式，这是一个通用的知识点：一个是快照，或者称为副本；另一个是日志
3. 快照：
  1. mysql 的数据本身就是存在磁盘中，不会丢失，但运维方面每天会将这块硬盘的数据拷贝出来，放到其他的硬盘中或者别的主机中，做异地的可靠性存储，并将数据按照时间拉链，当某一个库发生致命性错误时，可以进行回滚，这种形式称为快照
  2. redis 的数据是存在内存中的，这时可以把数据从内存移到磁盘上，一个小时挪出来一份，那么在磁盘上就有了数据的副本、快照，之后也可以把磁盘上的数据挪到别的主机上去，那么即使 redis 进程被炸成灰了，也可以拿快照恢复出一个慢一个时点的数据
4. 日志：在用户发生增删改时，对着服务发生增删改的操作，每一个操作都会向一个文件中去记录，最终日志文件比较完整，数据文件上面的副本全都没有了，最终只要把日志里面的命令一条一条执行，执行后就会出现曾经执行过的那些数据，也可以做一个恢复
5. redis 中的持久化：RDB 就是上面说的快照、副本；AOF 就是日志，向文件中追加写操作

### 2、RDB 概述

#### 【1】概述

- 首先向快照、副本这一类的数据持久化，都是时点性的
- 时点性：比如 redis 中每小时落一次文件，每个文件有几个G，不可能一瞬间落成，会延续几十秒，比如 8 点落成的文件，8 点 10 分结束，从内存中将数据写到磁盘，那么磁盘中的文件应该是 8 点的时候的状态，而非 8 点 10 分或 8 点到 8 点 10 分之间变化的状态。

#### 【2】落实时点性方案一及问题推论：阻塞

1. 方案：当代码需要做快照了，那么整个 redis 进程就不对外提供服务了。这时内存把所有的键值对全部写到磁盘中，即使是从 8 点写到 8 点半，这期间没有发生任何变化，磁盘中的每一笔记录还是 8 点时候的样子，这个文件隶属于 8 点

2. 方案产生的问题以及推论：

   - 方案一中最为致命的问题是阻塞，一旦阻塞就代表着服务不可用，在企业当中服务不可用是一件很重要的问题，所以不能够采纳方案一

   - 应该使其不阻塞，在 redis 对外提供服务的同时 ，还能满足数据落地，持久化

     ![](F:\Java\images.assets\Redis.assets\时点混乱.png)

   - 上图阐述中，a =  3 写入后就不能对 a 重新写入，所以 a = 8 的状态就会丢失，见以下解决问题

### 3、RDB 知识点补充 —— Linux 管道

1. Linux 中的管道就是<span style='color:orange'>前面输出作为后面的输入</span>，例如：ls -l /etc | more ，可以分页显示命令内容
2. <span style='color:orange'>管道会触发子进程</span>

   <img src="F:\Java\images.assets\Redis.assets\Linux管道演示1.png" alt="image-20210127215949574" />

   - 如上图，当前执行的程序是一个进程，遇到管道之后，左边先启动一个进程，右边再启动一个进程，这时在内存中会有三个进程

     <img src="F:\Java\images.assets\Redis.assets\Linux管道演示2.png" alt="image-20210128163805903" />
   - 如上图，是存在父子进程不同的 ID 号的，$$ 命令取进程号到优先级高于管道，$BASHPID 命令取进程号优先级低于管道

     <img src="F:\Java\images.assets\Redis.assets\Linux管道演示3.png" alt="image-20210128165759630" />
   - 如上图，常规思想中，进程是数据隔离的，子进程无法看到父进程的内容

     <img src="F:\Java\images.assets\Redis.assets\Linux管道演示4.png" alt="image-20210128170007149" />
   - 通过 export 环境变量，父进程是可以让子进程看到数据的

     <img src="F:\Java\images.assets\Redis.assets\Linux管道演示5.png" alt="image-20210128170635957" />

     <img src="F:\Java\images.assets\Redis.assets\Linux管道演示6.png" alt="image-20210128170858008" />
   - 如上验证，Linux  export 环境变量，子进程的修改不会破坏父进程

     <img src="F:\Java\images.assets\Redis.assets\Linux管道演示7.png" alt="image-20210128171205381" />
   - 父进程的修改也不会破坏子进程
3. 父子进程的思考及问题
   1. 思考：
      - 通过以上验证，理论上可以理解为子进程里面会创建一个数据副本，因此父子进程双方都可见 num，却互不影响
      - 基于这一点，如果父进程是 redis，其内存数据为 10g ，那么要达到非阻塞的情况下，redis 父进程可以在 8 点的时候创建出一个 redis 的子进程，这相当于这个时候 redis 的数据拷贝过去了一个副本，父进程继续接受响应修改数据，子进程数据存成文件就可以了
   2. 问题：
      1. 速度问题，创建子进程相当于拷贝数据的速度的快慢；
      2. 内存大小问题，如果父进程 10g，创建一个子进程也是 10g，那么内存是否有 20g 的空间，是否还有空余的 10g 来创建子进程
   3. 为了解决以上问题，在开发 Linux 系统时，提出了一个系统调用，fork()

### 4、RDB 知识点补充 —— Linux  fork()

1. fork()，本身是指针的一个引用，它可以达到的的效果是创建速度特别快，且对空间要求不是特别多
   1. 速度相对快
   2. 空间相对小，只会占用很小的空间
2. fork() 的实现

   ![](F:\Java\images.assets\Redis.assets\fork的实现1.png)

   - 如上图，在计算机中有物理内存，类似于线性的地址空间字节数组，之后会有程序，比如 redis，程序默认整个内存都是自己使用的，所以会有一个虚拟地址空间，是一个虚拟地址到物理地址的映射
   - man 2 fork

     ![](F:\Java\images.assets\Redis.assets\fork的实现2.png)

### 5、RDB 具体实现

1. 实现理论

   <img src="F:\Java\images.assets\Redis.assets\rdb的具体实现.png" alt="image-20210128183035631" style="zoom: 80%;" />

   - 在 redis 中父进程调了一个 fork ，创建出一个子进程，且速度相对快，不会涨出内存，因为如果父进程中 10g，物理内存中就存了 10g，fork 方式创建出子进程只是做了一些映射关系，并没有把 10g 的数据再拷贝一份，根据以上，子进程写成文件，以一种异步后台的形式写成文件
2. 实现方式
   1. 命令的方式：
      1. save 命令：触发前台阻塞，在目的明确的时候，比如关机维护，那么就必须发一个 save，将数据做一个全量，相对少见
      2. bgsave 命令：后端异步的非阻塞方式，会触发 fork 创建子进程，在正常运行的情况，不做任何停服的时候使用
   2. 在配置文件中，编写 bgsave 规则
      - 配置文件中用的是 save 的标识，但触发的是 bgsave
      - `vi /etc/redis/6379.conf  --> SNAPSHOTTING`
        - save 900 1    ;    save 300 10     ;      save 60 10000
        - （save 时间 操作数），当时间达到 60 秒时，操作数没有达到 10000 就会进入 300 秒的判断，操作数大于 10 ，就会触发，900 秒时即使有 1 条也需要存一下，时间不宜过长
        - 关闭：save " "
        - redis 中默认开启 RDB
        - rdbchecksum ：在 RDB 文件最后写一个交易位
        - dbfilename ：dump.rdb ，文件名
        - dir /var/lib/redis/6379 ：文件存储的目录

### 6、RDB 优点及弊端

1. 优点：类似于 Java 序列化，就是把内存中的字节数组直接放到磁盘中，恢复的时候也是直接恢复，速度相对快
2. 弊端
   1. 不支持拉链，只有一个 dump.rdb ，所以需要每天定时将文件拷到别的磁盘
   2. 丢数据相对多，它不是实时在记录数据的变化，因为它是时点性的，时点与时点之间窗口数据容易丢失
   3. 如果在两个时点之间，比如 8 点落成了 rdb ，9 点刚要落 rdb，这时挂机了，就丢了 1 小时的数据

### 7、AOF 概述

1. Append Only File ，向文件中追加，追加的内容就是服务器发生的写操作
2. 伴随着每一个写，都会有一个相应的到达，丢失的数据相对少
3. redis 中 RDB 和 AOF 可以同时开启，但只会用 AOF 做恢复，因为其数据完整性好一些
4. AOF 中记录的是日志，等于重新执行里面的命令，速度相对 RDB 慢
5. 4.0 版本以后，AOF 中包含 RDB 全量，增加记录新的写操作，这样在恢复数据时，可以先将 rdb 二进制恢复，再把 AOF 文件之后新增的内容一条一条执行，这样新增数据相对少一些，速度就相对快一些
6. 案例：如果 redis 运行了 10 年，持久化方案为 AOF，在第十年时，redis 挂了，这时 AOF 文件可能 10t，恢复需要 5 年，那么这时恢复的话，内存是不会溢出的，因为 AOF 中的命令都是溢出前写进去的
7. 优缺点：
   1. 优点：体量可以无限变大
   2. 缺点：因其体量大，所以恢复慢
8. 保证日志足够小的方案
   1. 4.0 之前：重写机制，就是抵消和整合命令，然后合并重复的，归属到一个 key 里面，比如 10 年中 AOF 只有创建 key 删除 key 两个操作，如果最后是创建 key，删除 key 没有发生，那么创建之前的操作都是没有意义的，可以互相抵消掉。
   2. 4.0 之后：一旦发生了重写，会将老的数据 RDB 到 AOF 文件中，之后将增量以指令的方式 append 到 AOF，也可以得出结论，AOF 当中是包含 RDB 和增量日志的。
9. I/O问题：当发生增删改时，会触发 I/O 行为向磁盘记录，且拖慢 redis 内存速度，所以在 AOF 中有三个几倍可调，no / always / everysec

### 8、AOF 配置解析

1. 命令配置：`vi /etc/redis/6379.conf   -->    APPEND ONLY MODE`
   1. `appendonly no    -->    appendonly yes`    默认是关闭 AOF ，需要打开
   2. `appendfilename "appendonly.aof"`    文件名
   3. `appendfsync always  /  appendfsync no  /  appendfsync everysec` 这三个配置项会影响到 redis 的性能速度以及数据的完整性，默认是 everysec 每秒级
   4. `no-appendfsync-on-rewrite no`  在 redis 子进程做 rdb 重写时，父进程要不要对磁盘刷新，no 认为自己的进程正在对磁盘进行疯狂的写，这时自己不能参与写来争抢 I/O ，但容易丢数据，这里需不需要开要看对数据的敏感性
   5. `aof-use-rdb-preamble yes`  开启 RDB & AOF 混合模式
2. 补充知识 -- flush，解释上述 appendfsync
   1. <img src="F:\Java\images.assets\Redis.assets\flush.png" alt="image-20210129132432905" />
   2. Java 编程时，如果打开一个文件向里面写东西，写完之后，在 close 之前需要 flush：首先在计算机中，所有对磁盘硬件的 I/O 操作都必须操作内核，在内核中，比如 Java 程序想调用文件描述符 8 ，会给它开一个 buffer，之后 Java 在写的时候，是先写到 buffer，buffer满了之后，内核缓冲区会向磁盘刷写，之后再关闭
   3. redis 中，比如有增删改操作 4 笔，那么它每笔会向 buffer 中写
      1. no 表示为 redis 不调 flush，内核什么时候满了，什么时候向磁盘刷写，这时可能会丢失一个 buffer 的数据
      2. always 表示为，redis 只要写了一个文件描述符，就调用一个 flush 把这个文件描述符刷写进去，即使 buffer 没有满，也会往磁盘中写，所以 always 一定是最可靠的，最多就在调下来的瞬间，没调成功，丢失一条数据
      3. everysec 表示为每秒调用一次 flush，是 redis 默认的级别，有可能丢接近一个 buffer 的数据，但概率不高，是处于速度最慢的 always 和速度最快的 no 之间的一种，相对丢的数据量比较少

### 9、RDB & AOF 混合使用

#### 【1】演示非混合

- 环境：
  - daemonize yes    -->    daemonize yes 日志在前台展示
  - logfile /var/log/redis_6379.log    注释掉，否则会将日志输为文件
  - aof-use-rdb-preamble yes    --> aof-use-rdb-preamble no    关掉使用 rdb
  - auto-aof-rewrite-percentage 100    &    auto-aof-rewrite-min-size 64mb    触发重写，当达到 64m、100%时触发重写，并且 redis 会记录最后一次重写完的体积大小，比如重写之后降低为 32m，那么下一次只有再达到一个 100% 64m 时，才会再次触发重写
- 启动：redis-server /etc/redis/6379.conf
- AOF 文件：

  ![](F:\Java\images.assets\Redis.assets\aof文件1.png)
- 开启 bgsave 后：使用 redis-check-rdb dump.rdb 检查 rdb 文件

  ![](F:\Java\images.assets\Redis.assets\aof文件2.png)

  - ![image-20210129165151178](F:\Java\images.assets\Redis.assets\rdb文件.png)
- AOF 文件重写  BGREWRITEAOF

  <img src="F:\Java\images.assets\Redis.assets\aof文件重写1.png" alt="image-20210129165558119" />

  <img src="F:\Java\images.assets\Redis.assets\aof文件重写2.png" alt="image-20210129165642164" />

#### 【2】演示混合

- 环境：aof-use-rdb-preamble yes
- 启动：redis-server /etc/redis/6379.conf
- AOF 文件：

  ![image-20210129170223732](F:\Java\images.assets\Redis.assets\aof文件3.png)
- AOF 文件增量日志

  ![](F:\Java\images.assets\Redis.assets\aof文件4.png)

## 九、Redis的集群 

### 1、单点问题 & CAP 原则

1. 单点问题
   1. 存在单点故障问题，进程一挂，数据就没有了，即使存到磁盘上，放到其他机器，也要等待很久才继续可用
   2. 容量有限，内存是有限的，当数据过多时会有存放不下的可能
   3. 压力，连接数压力，I/O 压力，CPU 运营操作的压力，都比较大
2. CAP 原则，最多只能实现两点，不能三者兼顾：一致性、可用性、分区容错性

### 2、AKF 原则技术拆解

- 解决上述单机问题，会有不同的解决方案，以及组合方案，所以需要 AKF 以 xyz 轴线的方式对技术进行拆解划分
- AKF 是微服务拆分的四大原则之一，且不仅只限定微服务，数据库、redis 等都可以用 AKF 进行拆分

#### 【1】X 轴

- 将 redis 实例做副本，一主多备的模型，并且可以进行读写分离

  ![](F:\Java\images.assets\Redis.assets\AKF的X轴.png)
- X 轴方案是全量镜像，能够解决单点故障问题，不能 100% 解决压力问题以及容量有限的问题
- redis 的主如果有 10g 数据，那么备机也会有 10g 数据，每一个拿出来都代表所有基于内存的数据，所以它不能解决容量问题
- 一般情况 redis 的主可以提供读写，备机只能提供读取，因为除了有主备模型，还有主主模型，就是多主，多主多活的成本更高，再者这类模型的数据一致性问题更严重，所以压力也不是 100% 解决，只解决了读取的方向

#### 【2】Y 轴

- 将 redis 作为多主，按照不同业务进行划分，比如订单数据存一台 redis、财务数据存一台等

  ![](F:\Java\images.assets\Redis.assets\AKF的Y轴.png)
- Y 轴相当于按照业务的多台 redis 都是单点的，但其又可以和 X 轴一起使用，对每个业务 redis 做备机

#### （三）Z 轴

- 当 redis 中容量不足时，可以通过一定规则算法将数据分散到另外一台 redis 中，比如可以将 0-5w 的用户放到 redis1 中，5w-10w 的用户放到 redis2 中

  ![](F:\Java\images.assets\Redis.assets\AKF的Z轴.png)

### 3、AKF 一变多的问题一

- 在企业环境中，需要关注数据一致性问题，为了解决这个问题，有两种方案，强一致性和最终一致性

#### 【1】强一致性：

- 当客户端对主节点进行写操作后，主节点阻塞不返回，而去通知备节点进行写操作，当备节点全部写成功后，再返回给客户端

- 强一致性的方案一直是企业想要追寻的，但成本极高，可能是不可能达到的效果
- 强一致性会带来服务不可用的问题，比如备机在写操作时程序异常退出了，或者执行缓慢、延迟等，那么一般在通信里会有超时的概念，可能三到五秒写不成功，就会认为写失败，这就代表着服务不可用
- 也就是说，强一致性会破坏可用性，但上述主备方案就是为了解决可用性的问题，所以强一致性方案是不可用的

  <img src="F:\Java\images.assets\Redis.assets\强一致性.png" alt="image-20210203162117558" />

#### 【2】弱一致性

- 通过上面的问题，只能将强一致性降级，需要容忍一部分数据的丢失
- 这个方案中，当客户端写入主节点成功后，就立刻返回，备机以异步的方式处理这笔写操作
- 但可能会造成，主节点返回成功后，备节点都返回失败了，这时主节点也挂掉了，重启之后就丢了一笔刚才的写操作，k1

  <img src="F:\Java\images.assets\Redis.assets\弱一致性.png" alt="image-20210203162627591" />

#### （三）最终一致性

- 最终一致性也归属于弱一致性
- 此方案就是在主节点写操作后立刻返回成功，并将这笔操作写入到 kafka（或类似的技术），通过 kafka 向备机进行写操作
- kafka 必须是可靠的，不能是单机，一定是集群的，也就是主节点写进去的东西不能丢，且响应速度还足够快，数据还能够持久化等
- 以上环节与 redis 无关，redis 并没有采取这种方式
- 最终一致性会出现数据不一致的问题，如果消息队列环节还未处理备机写操作，这时有客户端读取备机数据，那么这时的数据是不一致的
- 一般像 redis 、Zookeeper 都是最终一致性的，可以强调成强一致性，可以先对其进行数据更新

  <img src="F:\Java\images.assets\Redis.assets\最终一致性.png" alt="image-20210203163528498" />

### 4、AKF 一变多的问题二 ：

- 在主从或主备模式下，主机本身就是一个单点，如果主机挂掉了，需要将备机或从机切换为主，来维持主机的高可用，并且需要一个机制来判定主机已经挂了，需要切换主机

1. 主机监控推论
   - 监控程序应该是一个 一变多的集群，比如说有三台监控程序来监控一台主机，也可以理解为三台监控程序来决策主机是否已经挂掉了
   - 首先，如果三台监控程序一同决策，都必须给出主机挂掉的消息，这样无疑是最精确的，这时就是强一致性。但如果其中一台程序出现了网络延迟，就会造成监控不可用，破坏了可用性
   - 其次，如果三台监控程序只有一台来决策，一台给出主机挂掉的消息，就将主机踢掉，换另外一个，这时其他两台依然连接正常，会造成统计不准确。也就是说一台决策的情况是势力范围或权重不够，比如一个客户端从服务中取 a = 1，另一个客户端从服务中取 a = 3，这时整个服务对外暴露的就是两种状态，这样就会产生网络分区问题，就是在服务当中，从外部网络访问的时候可以拿到两个不同的版本
   - 以上推论就是为什么竞争需要过半，如果不过半就会产生网络分区问题（脑裂），但脑裂并非绝对的不好，它会有一个分区容忍性的概念
2. 网络分区 & 分区容忍性
   1. 网络分区就是通过网络去访问服务时，不同的客户端拿到了不同的结果
   2. 就像是微服务中的服务注册中心，当公司里有一个购物车的服务，可能有 10 台机器做负载，注册中心也是集群模式， 这 10 台都要去注册中心集群里去注册，这时前面通过网关和其他服务调用时，不同的服务拿到集群中不同的节点，有的服务看到后面有 10 台可用，有的能看到 5 台可用，但其实它们只需要其中一台能通过就可以了，或者负载的时候，负载多少台没有太大差异，反正可以跑通就可以了，以上叫分区容忍性
3. 主机监控程序需要部分决策
   1. 当三台监控程序中，有两台进行决策，那么就代表了它们的势力范围是 2 ，也就是说它们两台的消息要达成一致，这两台之间要互相通信，互相决策，这时即使另外一台持相反消息，外界也不会响应，因为其势力范围为 1已经没有决策权了
   2. 这样做的好处在于，两台结成势力范围，另外一台不算数，这时这两台认为主机挂了就是挂了，没挂就是没挂，不会出现模棱两可的状态
   3. 如果这时监控的集群是 4 个节点，那么两台的势力范围就会出现脑裂的问题，所以在这种情况下，就需要结成 3 台节点的势力范围
   4. 以上推论出的公式：（n / 2）+ 1 ， 也就是通俗说的过半
4. 主机监控程序一般使用奇数台
   - 因为要控制承担的成本和风险，就是可以允许几台出现故障
   - 三台中即使有一台挂掉了，那么还有两台可以结成势力范围，可以允许有一台挂掉；四台中要满足过半，就是三台决策，那么它也只能允许一台出现故障，再多挂一台就结不成势力范围了
   - 以上，其所能承担的风险是一致的，但成本不同

### 5、主从复制原理及配置

1. 主从复制的原理：Redis 使用默认的异步复制，其特点是低延迟和高性能，是绝大多数 Redis 用例的自然复制模式。但是，从 Redis 服务器会异步地确认其从主 Redis 服务器周期接收到的数据量；Redis 使用的是弱一致性，而非强一致性
2. 命令方式设置主从
   1. 5.0 以前使用 slaveof
   2. 5.0 以后使用 replicaof  --> replicaof 127.0.0.1 6379
   3. 客户端启动时追随：redis-server ./6380.conf --replicaof 127.0.0.1 6379
3. 配置解读
   - 6379启动配置

     ![](F:\Java\images.assets\Redis.assets\6379启动配置.png)
   - 6381启动配置

     ![](F:\Java\images.assets\Redis.assets\6381启动配置.png)
   - 以上 6379 和 6381 就配置为主从了
   - 默认情况下 6381 从机是禁止写入的
   - 同步数据的过程：
     - 首先 6379 中落成一个磁盘文件 RDB 
     - 其次 6381 中清除了自己的原有数据
     - 之后 6381 中将 RDB 同步到自己的内存中，完成数据同步
   - 如果 6381 挂掉后又很快的启动了，那么这个过程采用的是增量同步数据的方案，而非重新拉取 RDB
   - 如果开启了 appendonly yes ，就是开启了全量同步的方案
   - 当主机 6379 挂掉之后，需要将从切换为主，比如将 6380 切换为主，并让 6381 重新追随
     - 6380 --> replicaof no one
     - 6381 --> replicaof 127.0.0.1 6380
4. 配置文件方式配置主从  REPLICAT`I/O`N
   1. `replicaof <masterip> <masterport> ` 表示追随谁
   2. `masterauth <master-password> ` master 的密码
   3. `replica-serve-stale-data yes ` 当主备复制时，如果数据较多，需要传输一段时间的数据，那么在这段时间中，备机中原有的数据需不需要支持查询，yes 表示支持查询原有数据，no 表示必须同步完成后才能支持查询
   4. `replica-read-only yes  `表示开启它的只读模式
   5. `repl-backlog-size 1mb  `在 redis 中维护的一个队列，队列的大小由自己设置，在备机挂掉又恢复做增量复制时，拿着 RDB 里面的信息，给出一个偏移量，比如上次取得位置是 8 ，现在的位置是 20 ，那么就从 8 到 20 传过去就可以了，但如果这段时间的数据将队列写满了，里面的数据就会被挤出，取不到数据时，就会触发全量复制
   6. `min-replicas-to-write 3  ` & `  min-replicas-max-lag 10  ` 最小写成功，需要根据业务需求判断是否开启，一旦开启就会向强一致性靠拢

### 6、sentinel 哨兵使用

1. http://www.redis.cn/topics/sentinel.html

2. Redis 的 Sentinel 系统用于管理多个 Redis 服务器（instance）， 该系统执行以下三个任务：

   - **监控（Monitoring**）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
   - **提醒（Notificat`I/O`n）**： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
   - **自动故障迁移（Automatic failover）**： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

3. 启动 sentinel：`redis-server ./26379.conf --sentinel`

4. 配置文件：

   - `port 26379`  sentinel 端口号

   - `sentinel monitor mymaster 127.0.0.1 6379 2`  表示监控的是 6379 ，两台 sentinel 权重

   - 其源码中包含了 sentinel 配置文件的示例

     ```
     sentinel monitor mymaster 127.0.0.1 6379 2
     sentinel down-after-milliseconds mymaster 60000
     sentinel failover-timeout mymaster 180000
     sentinel parallel-syncs mymaster 1
     
     sentinel monitor resque 192.168.1.3 6380 4
     sentinel down-after-milliseconds resque 10000
     sentinel failover-timeout resque 180000
     sentinel parallel-syncs resque 5
     ```

   - sentinel 在选举之后会最终修改自己的配置文件的，比如修改监控的主机端口

5. sentinel 互相发现

   - 哨兵之间是通过发布订阅来做到互相发现的
   - psubscribe * 表示订阅方，订阅所有消息，会发现每个哨兵都在发布消息

### 7、sharding 分片引入

![image-20210208204514795](F:\Java\images.assets\Redis.assets\sharding分片.png)

1. 方式1：按照业务逻辑对数据进行拆分，存在不同的 redis 实例中
2. 方式2（sharding）：当数据不能拆分时，可以采用 hash + 取模，按照 redis 实例数进行取模，但当再新增一台实例时，原有的数据就要拿出来重新取模放到新的位置上，影响分布式下的扩展性
3. 方式3（sharding）：
   - 当数据不能拆分时，可以采用 random 随即的方式，当客户端有一个数据，会随即的放到其中一台实例中
   - 但当需要取出这些数据时，并不知道这些数据具体放在哪里了，需要特定的场景
   - 当 k1 为 list 类型时，两台实例中都会有 lpush k1 的随即部分数据，另一台客户端只要连上这两个实例做 rpop 消费就可以了
   - 这时 k1 就相当于 topic，每一个 redis 就相当于 partit`I/O`n ，类似于 kafka
4. 方式4（sharding）：
   - 当数据不能拆分时，可以采用一致性哈希算法 kemata
   - kemata 中不做取模运算，它抽象为一个环形结构，实例节点和数据都要参与 hash 计算
   - 实例节点计算出的位置是物理节点，其他的都是虚拟节点，数据计算出的虚拟节点会找到数值比它大的物理节点，并存进去
   - 当新增节点时，不影响节点后面的区间数据的查找映射关系，但前面的数据会出现数据不命中的情况，可以在查找映射关系时，查找前两个节点的数据，当然这是成本问题，需要根据业务需求来确定
   - 比如每台实例是以其 IP 地址计算 hash 的，可以使每台实例的 IP 后面依次拼 10 个数字，这样每一个实例就会出现在多个物理节点上，这时某一个 key 找到自己位置后，找到离它最近的那个点，就会解决数据倾斜的问题

### 8、Redis 中代理的使用

1. 上述方案产生的问题：
   - 搭建起一个项目的时候，对 redis 不能只开一个连接，会有一个连接池
   - 每一次连接，无论是长连接或是短连接，都会造成损耗，也就是说 redis 的连接成本很高，是对 server 端造成的
   
     <img src="F:\Java\images.assets\Redis.assets\redis连接成本.png" alt="image-20210209091745470" />
2. 以代理方式解决上述问题【1】

   <img src="F:\Java\images.assets\Redis.assets\代理实现1.png" alt="image-20210209092359345" style="zoom:80%;" />

   - 可以通过代理层来缓解 server 端的压力，且只需关注代理层的性能
   - 当前面客户端过多时，可以对代理层做集群
3. 以代理方式解决上述问题【2】

   <img src="F:\Java\images.assets\Redis.assets\代理实现2.png" alt="image-20210209093232748" style="zoom: 67%;" />

   - 当代理也出现问题了，在代理前面可以加一个 LVS ，LVS 不处理连接握手，数据包过来了就发过去，只要数据中心能够让那么多流量进来，那么相对应的 LVS 服务器就能让这些流量流过去
   - 代理层在这个时候没有必要做高可用，因为 LVS 不能只是一台，LVS 如果挂掉了，那么后面的就都无法访问了，所以 LVS 会做一个主备
   - LVS 的主备是靠 keepalived 来管理的，它除了可以完成主备间切换，还可以间接的监控两个 Proxy 的健康状态，当发现其中一台挂了，就会触发脚本，在前面剔除它，走另一个
   - 无论企业中技术多复杂，对于客户端是透明的，加入 LVS 后，无论 redis 有多少台 IP，代理层有多少 IP，整个后端有多复杂，前面客户端无论有多少个项目组，都只会记录一个地址，所有客户端代码以及 API 都会变得极其简单

### 9、预分区 & Cluster 引入

#### 【1】分片及代理方案的弊端

- 上述方案无论是传统的还是代理的，都没有离开 modula、random、kemata 三种模式
- 以上的缺点是只能用来做缓存，不能做分布式数据库
- 为了解决上述问题，提出了预分区的概念

#### 【2】预分区

<img src="F:\Java\images.assets\Redis.assets\预分区.png" alt="image-20210209231002797" />

- 加入目前有两台实例，但不代表未来只有两台实例，可能会新增一个节点，那么当新增节点出现后，就会对架构中的代理或客户端，也是对算法的一个挑战，如果这里使用的是客户端模式，或者代理层模式，算法使用的是一致性哈希或者取模方式，当新增节点时，会造成 rehash 重新计算数据的位置，重新分配，也会造成其中有一部分数据被拦截，只能从新节点取数据，但是取不到
- 那么解决上述问题的关键在于预分区，就是说可能未来会从两个节点加到三个节点，那么这时就直接取模取 10 个节点，模数值的范围是 0~9 ，之后中间只需要加一层 mapping ，比如第一个节点领到的是 0,1,2,3,4 ，第二个节点领到的是 5,6,7,8,9 ，各领到了 5 个槽位或 5 个分片
- 当未来新增一个节点时，只需要让 mapping 让出几个槽位，比如从第一个节点中拿到 3,4 ，第二个节点中拿到 8,9 ，那么相应的它不需要将全部数据都 rehash 一遍，只需要在这个 redis 当中找到 3,4 槽位的数据直接传输给它，只要数据迁移结束，新的映射关系让客户端知道后，之后的任何的 key 进来都可以去到正确的位置去取了，不影响新的 rehash ，也不会出现曾经有的数据找不到
- 在上述理论中，数据迁移是必须存在的，之前的一致性哈希的迁移程度很高，要对被影响的点重新做一遍 rehash，往里面加的时候，需要先从有序表里取出离我最近的点，把它所有的 key 取出来全算一遍，之后属于我这个区间的放到里面，其余的删掉，才能加进去，整个服务还要下线
- 现在只要曾经的数据模一个相对大的值，槽位足够大，那么之后新增节点，只需要将槽位让出去就=可以了

#### （三）redis 集群 —— Cluster

<img src="F:\Java\images.assets\Redis.assets\cluster结构.png" alt="image-20210209234807505" />

- 在 redis 集群中，是无主模型，如上图三台实例分领了公司的数据，各分 1/3 ，在客户端取的时候，客户端想连接哪个就连哪个，客户端中不存在任何逻辑，如上比如 k1 属于 4 号槽位，在 redis3 的实例上
- 客户端 get k1 直接发给其中一台 redis，每个 redis 只需要将曾经取模计算的功能放到每个 redis 自己身上，hash%10 那么当 get k1 到 redis2 上时， redis2 只需要先拿 key 取模，然后算出槽位，和本机的 mapping 进行比对，如果有就直接返回，如果没有的话就要去到别的实例上去找，所以每台实例上还应该有其他实例的映射关系
- 在上述两个条件的保证下，在这个集群中的每个实例都可以当家做主，当上述案例在本机中没有，且计算槽位后是 4 ，然后看槽位在哪个实例中，之后根据这台的标识，就会给客户端返回，客户端就会重定向到 redis3 中去，在 redis3 中重复上述操作

#### （四）hash tag 数据分治下的事物问题

- 无论是上述那种分治情况，当数据分开出现不同节点时，那么必然会带来聚合操作难以实现的问题
- 比如一个操作需要几个 key，但这几个 key 不再一个节点上，或者对两个 set 取交集等也很难实现，其实这时给 redis 发两个 set 的命令是可以实现的，但 redis 并没有实现，而是直接屏蔽掉了，因为这里面会有一个数据移动的过程，其作者一直是将影响性能的东西全部抹杀掉，所以一般会让计算向数据移动，而非移动数据，但作者想了另外的方式，hash tag
- 数据一旦分开，就很难被整合使用，那么数据不分开，就一定能发生事物，如果 key 定义为 {oo}k1 和 {oo}k2 ，那么它们拿着 {oo} 取模，而不是不一样的字符串取模，那么一定会落在一个节点上

#### （五）redis 集群槽位

- Redis 集群有 16384 个哈希槽，每个 key 通过 CRC16 校验后对 16384 取模来决定放置哪个槽，集群的每个节点负责一部分hash槽，举个例子，比如当前集群有3个节点，那么
  - 节点 A 包含 0 到 5500号哈希槽；
  - 节点 B 包含5501 到 11000 号哈希槽；
  - 节点 C 包含11001 到 16384号哈希槽；
- 这种结构很容易添加或者删除节点.，比如如果我想新添加个节点 D,，我需要从节点 A, B, C 中得部分槽到 D 上，如果我想移除节点 A，需要将 A 中的槽移到 B 和 C 节点上，然后将没有任何槽的 A 节点从集群中移除即可， 由于从一个节点将哈希槽移动到另一个节点并不会停止服务，所以无论添加删除或者改变某个节点的哈希槽的数量都不会造成集群不可用的状态

#### （六）redis 集群中的主从复制模型

- 为了使在部分节点失败或者大部分节点无法通信的情况下集群仍然可用，所以集群使用了主从复制模型，每个节点都会有 N-1 个复制品
- 在我们例子中具有 A,B,C 三个节点的集群，在没有复制模型的情况下，如果节点 B 失败了，那么整个集群就会以为缺少 5501-11000 这个范围的槽而不可用
- 然而如果在集群创建的时候（或者过一段时间）我们为每个节点添加一个从节点 A1，B1，C1，那么整个集群便有三个 master 节点和三个 slave 节点组成，这样在节点 B 失败后，集群便会选举 B1 为新的主节点继续服务，整个集群便不会因为槽找不到而不可用了
- 不过当 B 和 B1 都失败后，集群是不可用的

### 10、redis 中的代理

1. Twitter TwemProxy：https://github.com/twitter/twemproxy/
2. predixy：https://blog.csdn.net/rebaic/article/details/76384028 、https://github.com/joyieldinc/predixy

### 11、代理分片机制：twemProxy

1. build

   ```shell
    cd soft -> mkdir twemproxy
    git clone git@github.com:twitter/twemproxy.git
    (error) yum update nss
    cd twemproxy/twemproxy
    yum install automake libtool -y
    autoreconf -fvi
    (error Autoconf vers`I/O`n 2.64 or higher is required) 
      	 yum search autoconf
     	 cd /etc/yum.repos.d/
     	 wget -0 /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
    	 yum clean all
    	 cd ~/soft/twemproxy/twemproxy
    	 yum search autoconf
    	 yum install autoconf268
     	 autoreconf268 -fvi
    ./configur
    make
    cd src -> nutcracker 可执行文件
    源码 vi scripts/nutcracker.init
   ```

   - 源码 vi scripts/nutcracker.init

     <img src="F:\Java\images.assets\Redis.assets\twemproxy期望目录.png" alt="image-20210210110705262" />

   ```shell
    cp /scripts/nutcracker.init  etc/init.d/twemproxy
    cd  etc/init.d/ -> chmod +x twemproxy
    mkdir /etc/nutcracker
    cd ~/soft/twemproxy/twemproxy/conf
    cp ./*  /etc/nutcracker/
    cd ~/soft/twemproxy/twemproxy/src
    cp nutcracker  /usr/bin  # 可以在系统任何位置使用可执行程序命令
    cd /etc/nutcracker/
    cp nutcracker.yml nutcracker.yml.bak
    vi nutcracker.yml
   ```

   -  vi nutcracker.yml

     <img src="F:\Java\images.assets\Redis.assets\twem修改配置文件.png" alt="image-20210210111600818" />

2. start
   ```shell
    mkdir data -> mkdir 6379 ->mkdir 6380   在当前目录启动，就会以当前目录做为持久化目录
    cd 6379 -> redis-server --port 6379
    cd 6380 -> redis-server --port 6380
    service twemproxy start
    redis-cli -p 22121
   ```

   <img src="F:\Java\images.assets\Redis.assets\twem代理.png" alt="image-20210210115246528" />

### 12、代理分片机制：predixy

1. build ~

2. start

   ```shell
    mkdir predixy
    cd predixy
    wget https://github.com/joyieldInc/predixy/releases/download/1.0.5/predixy-1.0.5-bin-amd64-linux.tar.gz
    tar xf predixy-1.0.5-bin-amd64-linux.tar.gz 
    cd predixy-1.0.5
    cd conf -> vi predixy.conf
   	 Bind 127.0.0.1:7617
     	 #Include cluster.conf     Include sentinel.conf
   ```

   - `vi sentinel.conf `

     - `:.,$y ` 表示光标位置到最后一行复制  -> np 粘贴

     - `:.,$s/#// `表示将 # 号替换为空

       ```shell
       SentinelServerPool {
           Databases 16
           Hash crc16
           HashTag "{}"
           Distribut`I/O`n modula
           MasterReadPr`I/O`rity 60
           StaticSlaveReadPr`I/O`rity 50
           DynamicSlaveReadPr`I/O`rity 50
           RefreshInterval 1
           ServerTimeout 1
           ServerFailureLimit 10
           ServerRetryTimeout 1
           KeepAlive 120
           Sentinels {
               + 127.0.0.1:26379
               + 127.0.0.1:26380
               + 127.0.0.1:26381
           }
           Group ooxx {
           }
           Group xxoo {
           }
       }    
       ```

   - `cd ../bin/  `等待启动

   - `cd test` 另起客户端 ，启动哨兵 ：`vi 26379.conf  ->  vi 26380.conf  ->  vi 26381.conf`

     ```shell
     port 26379 // 26379 的哨兵监控 36779 和 46379 的 redis 主机
     sentinel monitor ooxx 127.0.0.1 36379 2  
     sentinel monitor xxoo 127.0.0.1 46379 2
     
     port 26380
     sentinel monitor ooxx 127.0.0.1 36379 2
     sentinel monitor xxoo 127.0.0.1 46379 2
     
     port 26381
     sentinel monitor ooxx 127.0.0.1 36379 2
     sentinel monitor xxoo 127.0.0.1 46379 2
     ```

     ```shell
      redis-server 26379.conf --sentinel
      redis-server 26380.conf --sentinel
      redis-server 26381.conf --sentinel
      mkdir 36379 ....    mkdir 46379 ....  # 新建持久化目录，并在目录下启动 redis
      cd 36379  ->  redis-server --port 36379  .....
      cd 36380  ->  redis-server --port 36380 --replicaof  127.0.0.1 36379   .....
     ```

   - `./predixy ../conf/predixy.conf ` 启动 predixy

   - `redis-cli -p 7617`

     - `set {oo}k1 a   set {oo}k2 b`
     - 可以保证两个 key 在同一个节点上

   - 代理两套主从后，不支持事物，配置文件中删除一套主从后可以支持事物

     ```shell
     SentinelServerPool {
         Databases 16
         Hash crc16
         HashTag "{}"
         Distribut`I/O`n modula
         MasterReadPr`I/O`rity 60
         StaticSlaveReadPr`I/O`rity 50
         DynamicSlaveReadPr`I/O`rity 50
         RefreshInterval 1
         ServerTimeout 1
         ServerFailureLimit 10
         ServerRetryTimeout 1
         KeepAlive 120
         Sentinels {
             + 127.0.0.1:26379
             + 127.0.0.1:26380
             + 127.0.0.1:26381
         }
         Group =ooxx {
         }
     }    
     ```

     - `./predixy ../conf/predixy.conf ` 启动 predixy

### 13、cluster 使用

```shell
 cd soft/redis-6.0.9
 cd utils/create-cluster/
 ./create-cluster start  官方脚本启动
```

<img src="F:\Java\images.assets\Redis.assets\cluster脚本.png" alt="image-20210217182453938" />

<img src="F:\Java\images.assets\Redis.assets\cluster槽位分配.png" alt="image-20210217182949776" />

```shell
 redis-cli -c -p 3001
 ./create-cluster stop   ./create-cluster clean
 redis-cli --cluster help  可以查看相关命令
# 不通过脚本，手动启动
 redis-cli --cluster create 127.0.0.1:3001 127.0.0.1:3002 127.0.0.1:3003 127.0.0.1:3004 127.0.0.1:3005 127.0.0.1:3006 --cluster-replicas 1
# 移动数据，解决数据倾斜，给出挪动多少个槽位数，给出移动到哪个节点的节点 ID，给出从哪些节点移动的节点 ID
 redis-cli --cluster reshard 127.0.0.1:3001
 redis-cli --cluster info  127.0.0.1:3001 # 查看信息
 redis-cli --cluster check 127.0.0.1:3001 # 查看详细信息
```

## 十、Redis开发及API演示

### 1、击穿

1. 在高并发的前提下，当 redis 中的某数据过期，恰巧此时并发请求这一热点数据，由于 key 的过期，造成并发直接访问了数据库

   <img src="F:\Java\images.assets\Redis.assets\击穿情况.png" alt="image-20210217233921834" />
2. 解决问题

   <img src="F:\Java\images.assets\Redis.assets\击穿解决.png" alt="image-20210217235301201" style="zoom: 67%;" />

   - 因为 redis 是单进程单线程的，无论多少并发访问 redis 都需要排队访问，所以在 redis 中可以通过 setnx() （在没有数据时才能添加），使第一个请求去 DB 请求数据（相当于为请求上锁），其他请求随即睡眠一段时间，等待数据返回正常访问
   - 可以通过设置锁的过期时间的方法，防止第一个请求在访问 DB 时挂掉，造成死锁
   - 可以使用多线程方案，解决锁超时的问题
   - 在分布式情况下，由 redis 自行调度会很麻烦，所以需要后面的 Zookeeper

### 2、穿透

1. 用户发起请求查询的数据是系统中不存在的，数据库中没有所以缓存中也没有，当大量这样的请求直接发到数据库，对数据库也会产生不小的压力

   <img src="F:\Java\images.assets\Redis.assets\穿透问题.png" alt="image-20210218071826976" />
2. 解决问题

   <img src="F:\Java\images.assets\Redis.assets\穿透解决.png" alt="image-20210218071735334" />

   - 可以使用布隆过滤器来解决问题，具体实现参考上述布隆过滤器
   - 布隆过滤器的弊端在于，只能添加，不能删除，当业务中需求时，可以更换为布谷鸟过滤器

### 3、雪崩

1. 当 redis 中大量的 key 在同一时间失效，那么大量访问这类 key 的请求将直接到达数据库，比如业务中的场景，在零点时必须要更新新的数据，需要更新缓存中的热点数据

   <img src="F:\Java\images.assets\Redis.assets\雪崩情况.png" alt="image-20210218073406692" />
2. 问题解决

   <img src="F:\Java\images.assets\Redis.assets\雪崩解决.png" alt="image-20210218074108111" />

   - 如果业务需求与时点性无关，那么可以将 key 的过期时间随即分散
   - 如果业务需求需要零点更新数据，那么可以在业务层使服务睡眠几十毫秒，使请求数降低，后强依赖于击穿方案

### 4、分布式锁

1. 使用 setnx 实现
2. 对 setnx 加入过期时间防止请求挂掉
3. 加入多线程（守护线程），用来延长过期时间，防止锁超时
4. 可以使用 Redisson 实现 redis 分布式锁
5. 可以使用 Zookeeper 做分布式锁，也是最为容易的方案
   - 当使用分布式锁时，是可以容忍在速度上没有那么快的
   - Zookeeper 要求准确度和一致性是最强的

### 6、redis API

1. https://docs.spring.`I/O`/spring-data/redis/docs/2.4.3/reference/html/#reference
2. https://github.com/redis/jedis
3. https://github.com/lettuce-`I/O`/lettuce-core

### 7、高低阶 API 实现

```java
// redis-cli
CONFIG GET *
CONFIG SET "protected-mode" no
systemctl status firewalld.service // 防火墙
systemctl stop firewalld.service // 关闭防火墙
    
// 依赖
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
    
// applicat`I/O`n.yml
spring:
  redis:
    host: 192.168.60.142
    port: 6379
    
 // RedisTestApplicat`I/O`n
@SpringBootApplicat`I/O`n
public class RedisTestApplicat`I/O`n {

    public static void main(String[] args) {
        ConfigurableApplicat`I/O`nContext run = SpringApplicat`I/O`n.run(RedisTestApplicat`I/O`n.class, args);
        RedisTest redis = run.getBean(RedisTest.class);
        redis.getRedis();
    }
}

// RedisTest
@Component
public class RedisTest {
    @Autowired
    private RedisTemplate redisTemplate;
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    public void getRedis(){
        // high-level
        redisTemplate.opsForValue().set("hello","world");
        System.out.println(redisTemplate.opsForValue().get("hello"));

        // high-level
        stringRedisTemplate.opsForValue().set("hello01","china");
        System.out.println(stringRedisTemplate.opsForValue().get("hello01"));

        // low-level
        RedisConnect`I/O`n conn = redisTemplate.getConnect`I/O`nFactory().getConnect`I/O`n();
        conn.set("hello02".getBytes(),"person".getBytes());
        System.out.println(new String(conn.get("hello02".getBytes())));
    }
}
```

<img src="F:\Java\images.assets\Redis.assets\高低阶演示.png" alt="image-20210218152058119" />

### 8、hash 代码实现

```java
// 依赖
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-json</artifactId>
</dependency>
    
// Person
public class Person {
    private String name;
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

// RedisTest
@Component
public class RedisTest {
    @Autowired
    private RedisTemplate redisTemplate; // 会破坏key的序列化
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Autowired
    private ObjectMapper objectMapper;

    public void getRedis(){

        HashOperat`I/O`ns<String, Object, Object> hash = stringRedisTemplate.opsForHash();
        hash.put("aka","name","xiaoming");
        hash.put("aka","age","22");
        System.out.println(hash.entries("aka"));

        Person person = new Person();
        person.setName("xiaohong");
        person.setAge(23);

        // 使stringRedisTemplate 的 value 按照 json 序列化，避免对象中属性类型不一致
        stringRedisTemplate.setValueSerializer
            (new Jackson2JsonRedisSerializer<Object>(Object.class));
        Jackson2HashMapper jm = new Jackson2HashMapper(objectMapper,false);
//        redisTemplate.opsForHash().putAll("bab",jm.toHash(person)); 
        stringRedisTemplate.opsForHash().putAll("bab",jm.toHash(person));
//        Map bab = redisTemplate.opsForHash().entries("bab");
        Map bab = stringRedisTemplate.opsForHash().entries("bab");
        Person per = objectMapper.convertValue(bab, Person.class);
        System.out.println(per.getName());
    }
}
```

### 9、自定义 template 代码实现

```java
// MyTemplate
@Configurat`I/O`n
public class MyTemplate {
    @Bean
    public StringRedisTemplate ooxx(RedisConnect`I/O`nFactory fc){
        StringRedisTemplate tp = new StringRedisTemplate(fc);
        tp.setHashValueSerializer(new Jackson2JsonRedisSerializer<Object>(Object.class));
        return tp;
    }
}

// RedisTest
    @Autowired
    @Qualifier("ooxx")
    private StringRedisTemplate stringRedisTemplate;
```

### 10、发布订阅代码实现

```java
public void getRedis() throws InterruptedExcept`I/O`n {

        // 发布
        stringRedisTemplate.convertAndSend("ooxx","hello");

        // 持续订阅
        RedisConnect`I/O`n connect`I/O`n = stringRedisTemplate.getConnect`I/O`nFactory().getConnect`I/O`n();
        connect`I/O`n.subscribe((message,bytes)->{
            byte[] body = message.getBody();
            System.out.println(new String(body));
        },"ooxx".getBytes());
        while (true){
            // 模拟自己发消息
            stringRedisTemplate.convertAndSend("ooxx","from main");
            Thread.sleep(3000);
        }
    }
}
```
