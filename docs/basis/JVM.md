

<img src=" _media/basis/JVM/jvm模型.png" alt="image-20201219204901406" />

<img src=" _media/basis/JVM/jvm详细模型.png" alt="image-20201219205042714" />

### 1、java从编码到执行的整个过程？

x.java -> 执行 javac -> 变成 x.class

当我们调用 Java 命令的时候 class 会被装载到内存中 —— classloader

一般情况下我们写类文件的时候也会用到 java 的类库，所以它要把 java 类库相关类也装载到内存里，

装载完成后会调用字节码解释器或者是即时编译器来进行解释或编译，

编译之后由执行引擎开始执行，执行到下面，面对的就是操作系统的硬件了，以上内容叫 JVM。

Java 编译好了之后变成 class，class 会被加载到内存，与此同时像 String、Object 这些 class 也会加载到内存

<img src=" _media/basis/JVM/Java编码到执行过程.png" />

### 2、Java是解释执行的还是编译执行的？

- 解释和编译是可以混合执行的（HotSpot）
- Java 代码要想放到 JVM 里去运行，首先需要经过 Javac 的编译，将 Java 代码编译为字节码 Class 文件。
  Class 文件反汇编后就是一条条 JVM 指令了，这些指令通过解释或编译后能够交给执行引擎执行。
- 解释执行：将 JVM 指令逐行翻译为本地机器码，逐行解释，逐行执行。
  - 优点：程序启动速度很快。
  - 缺点：程序执行速度很慢。
- 将 Class 文件直接编译成本地机器码并缓存下来，CPU可以直接执行。
  - 优点：执行时省去了解释的过程，执行速度很快。
  - 缺点：编译过程比较耗时，程序启动速度很慢。
- 特别常用的一些代码，代码用到的次数特别多，这时候它会做成一个本地的编译，就像 C 语言在 Windows 上执行的时候把它编译成exe文件一样，那么下次再执行这段代码的时候就不需要解释器来一句一句的解释来执行，执行引擎可以直接交给操作系统来调用，这个效率会高很多
- 不是所有的代码都会被JIT进行及时编译的。如果这样的话，Java就不能跨平台了，有一些特定的，执行次数很多次的时候，会进行及时编译器的编译

### 3、什么是JVM

JVM现在可以称之为一个跨语言的平台，Java叫跨平台的语言大家都了解，作为JVM虚拟机来讲，目前能够在JVM上跑的语言特别多，除了Java外，还有Scala、kotlin、groovy、clojure、jython、jruby等一百多种语言可以直接跑在虚拟机上，所谓的JVM虚拟机本身也是一种规范，Linux上游Linux的实现，Unix有Unix的实现，JVM帮你屏蔽了操作系统的这些底层

<img src=" _media/basis/JVM/从跨平台的语言到跨语言的平台.png" />

JVM与Java无关，

<img src=" _media/basis/JVM/JVM与Java无关.png"/>

其能够做到这么多语言都在上面跑，关键的原因就是class这个东西，任何语言只要能编译成class，复合class文件的规范，就可以在Java虚拟机上执行

JVM是一种规范，他就定义了Java虚拟机应该能够执行什么等等...Java虚拟机应该具备哪些模块，遇到什么样的指令应该做什么样的东西，可以到Oracle网站上查
- java virtual machine specifications
- https://docs.oracle.com/en/java/javase/13/
- https://docs.oracle.com/javase/specs/index.html

JVM是虚构出来的一台计算机，既然它是一个虚拟的计算机，你就可以想象成一层单独的机器，它有自己的CPU、指令集、汇编语言，相当于自己是一个操作系统。

Javac的过程

- <img src=" _media/basis/JVM/Javac的过程.png" alt="image-20201213193148104"  />

### 4、常见的JVM的实现

<img src=" _media/basis/JVM/常见的JVM实现.png"/>

HotSpot：目前用的最多的java虚拟机，java -version会输出你现在用的虚拟机的名字、执行模式、版本等信息，64位的Server版用的执行模式是mixed mnode

Jrockit：曾经号称世界上最快的JVM，后被Oracle收购，合并于HotSpot

J9 -IBM：IBM自己的实现

Microsoft VM：微软自己的实现

TaobaoVM：淘宝自己的实现，实际上是HotSpot的定制版

LiquidVM：一个直接针对硬件的虚拟机VM，它下面是没有操作系统的，下面直接就是硬件，运行效率更高

azul zing：很贵，特点是快，尤其垃圾回收在1毫秒之内，是业内表盖，它的一个垃圾回收算法后来被HotSpot洗嗽才有了现在的ZGC

### 5、JDK、JRE、JVM

JVM叫Java的虚拟机，只是用来执行的，就是你所有东西都弄好了之后它来执行

JRE叫运行时环境，Java要想在操作系统上运行，除了虚拟机外，java的那些核心类库是必需的

JDK叫Java开发工具包，包含JRE和JVM

<img src=" _media/basis/JVM/JDK、JRE、JVM.png"/>

### 6、class文件是什么？

整个 Java 虚拟机就是以 class 文件为核心，其格式就是一个二进制的字节流，这个二进制字节流是由 Java 虚拟机来解释的

参考程序

```java
package jvmtest;
public class T0100_ByteCode01 {
}
```

编译完成后，再看反编译版本，它会自动的帮你加了构造方法，默认的无参构造

<img src=" _media/basis/JVM/编译后的class文件.png" alt="image-20201213201553847" style="zoom: 67%;" />

这个 class 文件如果用16进制的编译器打开后的内容如下，工具为sublime

<img src=" _media/basis/JVM/class文件.png" alt="image-20201213203414022" style="zoom: 33%;" />

整个class文件的格式就是个二进制的字节流，这个二进制字节流由Java虚拟机来解释

<img src=" _media/basis/JVM/ClassFileFormat2.png"  />

参考分析，下图

<img src=" _media/basis/JVM/分析图1.png" alt="image-20201213222743287" />

上图可清晰的看出每个部分有多长，一般 u1、u2、u3...u8 ，其中 u 指的是无符号整数，u1 就是一个字节，u2 就是两个字节... ，一个小格就是一个字节，对于 16 进制来讲就是 4 位，两个 16 进制就一个字节 8 位。

前面 4 个字节，指的是这个文件的统一标识符，当我们看到这个文件的统一标识符的时候，就知道它是 class文件，很多文件都是这样标识的，它们都有自己的头，用 16 进制打开，前面几位一般都一样。看到 CA FE BA BE 这就是 Java 编译完的 class 文件，这部分叫 magic number ， 魔术

00 00 是minor version， minor version 表示的是整个版本的小标识符；00 34 是major version， 我们看到的class文件的时候，这8个字节代表的就是这个意思（16进制）

00 10 是constant_pool_count，表示常量池里面有多少个内容，常量池是class文件里最复杂的内容，而且常量池里会互相引用，以及常量池会被其他的各方面引用，所以常量池是及其复杂的。能够存储constant_pool_count - 1个常量，因为常量池是从1开始的，数组是从0开始的，所以要减去1,。原因是：它有一个0存在那里，将来可能会有些引用指向，表示不指向任何常量池的一项，那时候就可以用0来表示，保留了一个可能性。

<img src=" _media/basis/JVM/分析图2.png" alt="image-20201213224046987" />

### 7、观察ByteCode的方法

javap  Java自带的叫 javap 的命令

JBE 它除了可以观察二进制码之外还可以直接修改

JclassLib  IDEA 插件（常用）

### 8、分析ByteCode

General Information 通用的一些信息，前面图解也有说明这些信息的代表意思

<img src=" _media/basis/JVM/文件分析.png" alt="image-20201214003433400" />

ConstantPool常量池里面的常量类型特别多，常量池是一个class里面最复杂的东西

<img src=" _media/basis/JVM/CONSTANT_Methodref_info.png"/>

<img src=" _media/basis/JVM/CONSTANT_Class_info.png"/>

<img src=" _media/basis/JVM/CONSTANT_NameAndType_info.png"/>

源文件分析

<img src=" _media/basis/JVM/源文件分析2.png" />

- 举例：第一行最后的两位07，去上图查第tag = 7的是CONSTANT_Class_info，再去看Constant Pool中的2号是一个CONSTANT_Class_info，然后它又指向了当前自己类名14号，接下来的000e就是14号

Interface：接口目前并未实现

Fields：由于我们这个Class没有任何属性，所以Fields中也没有任何东西

Methods：它会给我们构造方法加一个无参的构造方法
- 一个方法的汇编实现翻译完了之后就是下图，这些就是Java的汇编，当做为一个Java虚拟机来讲，当它读到一个Class文件里面内容的时候，就要去查表里代表的是哪条指令
- 方法还有其他的附加属性，最主要的还是Code，就是这个方法的代码是怎么实现的
- <img src=" _media/basis/JVM/methods方法汇编.png" alt="image-20201214011736050" />
- 在Java虚拟机规范中的第七节，整个二进制码分别代表的什么内容，这个方法的具体实现翻译成Class之后的内容，这些就是Java的汇编，所以当Java虚拟机读到一个Class文件里面的内容的时候，就是找到这个方法的一个实现的时候，例如读到了30，它就要去表里查30代表哪一条指令，然后将这条指令翻译过来，上图中我们的第一条指令是 -> aload_0，然后在去查aload_0代表什么意思
- <img src=" _media/basis/JVM/aload_0.png" alt="image-20201214012419907" />
- <img src=" _media/basis/JVM/aload_0的表示.png" alt="image-20201214012724564" />

### 9、总结回顾

整个class文件有：
- Magic Number
- Minor version（小号）/ Major Version （主号）
- constant_pool_count 有多少个常量池
- 常量池 - 1 具体的内容有很多种类型
- access_flags 指的是整个class你前面写的是public还是privote等
- this_class 当前类是哪个
- super_class 父类是谁
- Interfaces_count 实现了哪些接口
- Interface 接口的索引
- fields 有哪些属性
  - access_flags：是不是public的，是不是static的，是不是final的
  - name_index：名称的索引
  - descriptor_index：描述符，到底是什么类型的
  - attributes：另外它所附加的一些属性
- methods 方法的各种结构，到底是怎样进行标识的，名字、描述符的索引、附加属性

attributes 附加属性里面最重要的一项是方法表，也就是这个方法编译完成之后的字节码指令，JVM看到这个指令的时候首先会读一些进来，然后根据指令去查自己的指令表，找到执行之后再来看这个指令是什么意思，拿aload_0来说，在方法的局部变量表里面的第0项，只要不是静态方法，它永远都是this，放到栈里之后进行第二条指令invokespecial特殊调用this的构造方法，然后巨魔在根据指令一条一条执行，当我们看到code的时候，最后一条指令是return（见8、2图）

aload_0表示2a，然后是b7，b7对应的是invokespecial，invokespecial是有两个参数的00 01，它们指向常量池第一号的，所以它们调用了java long的构造方法。下一条指令是b1，是return，说明方法结束了，因此构造方法里面的5个字节是这个文件的构造方法具体实现。任何一个class，不写任何构造方法的时候，它自己会生成一个构造方法，而生成的这个构造方法不做任何调用时，会调用父类的构造方法Object

<img src=" _media/basis/JVM/构造方法的具体实现.png" alt="image-20201214013327009"  />

JVM的有8个指令是原子的

> lock 锁定
>
> read 读取
>
> load 载入
>
> use 使用
>
> assign 赋值
>
> store 存储
>
> write 写入
>
> unlock 解锁

## 二、详解Class加载过程

Class Loading Linking Initializing

### 1、Class文件从硬盘到内存的过程

<img src=" _media/basis/JVM/从硬盘到内存的过程.png" alt="image-20201214102519596" style="zoom: 50%;" />

首先分为三个大步骤，以下逐一解释
- Loading
  - Loading是把一个Class文件load（装载）到内存里去
- Linking
  - Verification
    - 校验装载进来的Class文件是不是复合Class文件的标准，如果装载进来的不是CA FE BA BE，在这个步骤就被拒掉了
  - Preparation
    - 是把Class文件中静态变量赋默认值，不是初始值，比如static int i = 8，注意在这个步骤是将i先赋值为默认值0
  - Resolution
    - 是把Class文件常量池里面用到的符号引用，要给它转为直接内存地址，直接可以访问到的内容
- Initializing
  - 将静态变量这个时候赋值为初始值

### 2、类加载器

<img src=" _media/basis/JVM/类加载器.png" alt="image-20201214110912706"  />

首先JVM本身有一个类加载器的层次，这个类加载器本身就是一个普通的class，每个层次分别加载不同的Class，JVM所有的class都是被类加载器加载到内存的，那么这个类加载器可以叫做ClassLoader

每一个class都是被ClassLoader装载到内存的，那么这个Classloader就有一个顶级父类，这个父类就叫Classloader，它是一个abstract抽象类，相当于这个类被谁load到内存里了，它一定是classloader这个类的子类

一个class文件被load到内存之后，创建了两块内容，一块是将.class二进制文件扔到内存中，另一块是生成了一个class类的对象，这一块对象指向了第一块内容

如果用过反射，应该知道我们通过class这个对象去拿他的有些方法，甚至可以调用，分析一下执行反射的过程，它一定是将那些方法的信息存在这个对象里，然后要执行方法的时候，它会去找那个class文件里面的二进制码，翻译成Java指令来执行

内存分区：在存储常量或class的各种信息的时候，实际上这块内容逻辑上叫Method Area方法区，1.8之前这个方法区实现落地叫PermGen永久代，1.8之后叫MetaSpace，所以这两块说的都是方法区

<img src=" _media/basis/JVM/classloader2.png" alt="image-20201214120940406"  />

类加载器的层次：

类加载器的加载过程是分成不同层次来加载的，不同的类加载器来加载不同的class

- 第一层：最顶层的是BootStrap，它是来加载lib里jdk最核心的内容，比如说 rt.jar、charset.jar等核心类，所以当我调用getClassLoader()拿到这个类加载器的结果是一个空值的时候就代表已经达到了最顶层的加载器
- 第二层：Extension加载器扩展类，加载扩展包里的各种文件，这些扩展包在jdk安装目录jar/lib/ext下的jar
- 第三层：这个就是平时用的加载器application，它用于加载classpath指定的内容
- 第四层：这个就是自定义加载器CustomClassloader，加载自己定义的加载器

CustomClassloader父类加载器是 -> application福类加载器是 -> Extension父类加载器是 -> Bootstrap

注意：不是继承关系

<img src=" _media/basis/JVM/语法关系继承.png" alt="image-20201214121246619" style="zoom: 50%;" />

### 3、双亲委派机制

<img src=" _media/basis/JVM/双亲委派问题.png" alt="image-20201214123351089"  />

一个class文件需要被load到内存的过程
- 任何一个class，如果是自定义的Classloader，这时候就先尝试去自定义里面找，它的内部维护者一个缓存，查找缓存看是否已经被加载进来，如果加载进来了就不需要再加载第二遍，如果没有加载进来就马上进行加载

- 如果在自己自定义的缓存中没有找到，它并不是直接加载这块内存，它会去它的父加载器application中维护的缓存里去找，如果存在就直接返回，如果不存在就继续委托给它的父加载器Extension，如果有就直接返回，没有就继续委托它的父加载器Bootstrap，如果有就直接返回，没有就往回委托

- 如果Bootstrap维护的缓存中并没有加载，那么就会委托给Extension，而Extension只负责加载扩展jar包部分，所以会继续向下委托给application，而application只负责加载classpath指定的内容，其他无法加载，所以会继续向下委托给自定义Classloader，当我们能够把这个类加载进来的时候叫做成功，如果加载不进来就会抛异常ClassNotFound，以上叫做双亲委派

  <img src=" _media/basis/JVM/双亲委派过程.png"/>

为什么要做双亲委派？（面试题）
- 主要是为了安全
- 假设自定义的任何一个class都可以直接load到内存，那么就可以自定义java.lang.string，将这个string直接load到内存中，打包给客户，然后将密码存储为string类型对象，这样就不安全了
- 双亲委派就不会出现这样的情况，自定义Classloader加载一个java.lang.string时就产生了警惕，它先去上面查有没有加载过，上面有加载过就直接返回给你，不给你重新加载

### 4、Launcher

<img src=" _media/basis/JVM/举例程序.png" alt="image-20201214150328228"  />

- 类加载器的范围来自Launcher源码，当打印一个Class类文件的toString方法时，它默认显示是类的名字加上hashcode码，上图sun.misc.Launcher$ExtClassLoader指的是sun.misc包下面的Launcher类，这个类下面有一个内部类叫ExtClassloader
- Launcher就是ClassLoader一个包装类启动类，这个类里面规定了Bootstrap等类加载器加载的路径
  - <img src=" _media/basis/JVM/类加载范围.png" alt="image-20201214150953930"  />
  - <img src=" _media/basis/JVM/Bootstrap.png" alt="image-20201214151320127" />
  - <img src=" _media/basis/JVM/AppClassloader.png" alt="image-20201214151734745" style="zoom: 50%;" />
  - <img src=" _media/basis/JVM/ExtClassloader.png" alt="image-20201214151904473"  />
- 验证程序
  - <img src=" _media/basis/JVM/验证程序.png" alt="image-20201214152114876" style="zoom: 67%;" />
  - 上图程序可以看出Launcher的加载路径，首先pathBoot拿到属性，将字符串中的分号替换为换行符，输出
  - <img src=" _media/basis/JVM/Bootstrap加载的.png" alt="image-20201214152318878"  />
  - <img src=" _media/basis/JVM/ext加载的.png" alt="image-20201214152419622"  />
  - <img src=" _media/basis/JVM/app加载的.png" alt="image-20201214152509120"  />

### 5、自定义类加载器及源码解析

<img src=" _media/basis/JVM/类加载验证程序.png" alt="image-20201214153921690" />

- loadClass这个方法的过程：
  
  - 如果需要加载一个类，那么只需要调用classLoad的loadclass方法就能够把这个类加载到内存中，加载到内存后找到硬盘上找到这个类的class文件源码，把它load到内存，与此同时生成一个class类的对象返回
- 什么时候需要自己去加载一个类？
  - Tomcat在load自己的那部分，是要将自定义的那些class加载进去
  - JRebel热部署，需要一个classloader手动的load到内存去
  - Spring动态代理，它是一个新的class，当你要用的时候就会把新的load到内存里
- 每一个classloader内部的parent都是定义的classloader类型，且是final值，改不了
- 加载过程：
  - <img src=" _media/basis/JVM/加载过程源码.png" alt="image-20201214155031623" style="zoom: 67%;" />
  - findLoadedClass是否已经加载进来了，如果加载进来就直接返回，否则进入向父的加载过程，如果父类没有加载到就只能自己加载了，当只能自己加载时它调用了findClass这个类，这个类是protected受保护的，只能在子类里访问，见下图
  - <img src=" _media/basis/JVM/loadClass源码解析.png" alt="image-20201214155437356" />
  - <img src=" _media/basis/JVM/findClass源码.png" alt="image-20201214155702787" />
  - 当我自定义Classloader的时候，只用重写findClass，这个设计模式就是钩子函数模板方法。
- <img src=" _media/basis/JVM/classloader源码解析.png" alt="image-20201214175216817" style="zoom: 50%;" />
  
  - ClassLoader源码： findInCache - > parent.loadClass - > findClass()
- 自定义类加载器
  - 自定义类加载器有很多种方式，其中最简单的一种就是从ClassLoader继承，重写它的findClass方法，findClass方法的过程会用到它的一个辅助方法defindClass，以下是重写过程
  
  - File创建了一个文件位于C盘的test目录，之后传过去类名com.jvm.test.Hello，然后将名字中间的 ' . ' 替换成 ' / '，这样就找到了它的全路径，最后加上它的Class文件名。
  
  - 之后就是将它load到内存中，使用FileInputStream将其转换为InputStream，再用ByteArrayOutputStream转换成一个二进制字节数组，从文件中读出来写进字节数组里，再toByteArray()转换成一个二进制字节数组，再用defineClass来变成一个class类对象
  
  - ```java
    public class T006_MSBClassLoader extends ClassLoader {
    
        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
            File f = new File("c:/test/", name.replace(".", "/").concat(".class"));
            try {
                FileInputStream fis = new FileInputStream(f);
                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                int b = 0;
    
                while ((b=fis.read()) !=0) {
                    baos.write(b);
                }
    
                byte[] bytes = baos.toByteArray();
                baos.close();
                fis.close();//可以写的更加严谨
    
                return defineClass(name, bytes, 0, bytes.length);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return super.findClass(name); //throws ClassNotFoundException
        }
    
        public static void main(String[] args) throws Exception {
            ClassLoader l = new T006_MSBClassLoader();
            Class clazz = l.loadClass("com.jvm.Hello");
            Class clazz1 = l.loadClass("com.jvm.Hello");
    
            System.out.println(clazz == clazz1);
    
            Hello h = (Hello)clazz.newInstance();
            h.m();
    
            System.out.println(l.getClass().getClassLoader());
            System.out.println(l.getParent());
    
            System.out.println(getSystemClassLoader());
        }
    }
    ```
  
  - 一般class就是一个二进制流，自己编译好后，可以手动加密，一下采用异或加密，将文件里所有内容读出来，然后进行异或，异或后再写回去
  
  - ```java
    public class T007_MSBClassLoaderWithEncription extends ClassLoader {
    
        public static int seed = 0B10110110;
    
        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
            File f = new File("c:/test/", name.replace('.', '/').concat(".msbclass"));
    
            try {
                FileInputStream fis = new FileInputStream(f);
                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                int b = 0;
    
                while ((b=fis.read()) !=0) {
                    baos.write(b ^ seed);
                }
    
                byte[] bytes = baos.toByteArray();
                baos.close();
                fis.close();//可以写的更加严谨
    
                return defineClass(name, bytes, 0, bytes.length);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return super.findClass(name); //throws ClassNotFoundException
        }
    
        public static void main(String[] args) throws Exception {
    
            encFile("com.jvm.hello");
    
            ClassLoader l = new T007_MSBClassLoaderWithEncription();
            Class clazz = l.loadClass("com.jvm.Hello");
            Hello h = (Hello)clazz.newInstance();
            h.m();
    
            System.out.println(l.getClass().getClassLoader());
            System.out.println(l.getParent());
        }
    
        private static void encFile(String name) throws Exception {
            File f = new File("c:/test/", name.replace('.', '/').concat(".class"));
            FileInputStream fis = new FileInputStream(f);
            FileOutputStream fos = new FileOutputStream(new File("c:/test/", name.replaceAll(".", "/").concat(".msbclass")));
            int b = 0;
    
            while((b = fis.read()) != -1) {
                fos.write(b ^ seed);
            }
    
            fis.close();
            fos.close();
        }
    }
    ```

### 6、lazyloading

- 懒初始化又叫懒加载，JVM规范并没有规定何时加载，JVM虚拟机的实现都是用的懒加载，就是什么时候需要这个类再去加载，并不是说一个jar文件中有2000多个类，但我只用到了其中一个，就把2000多个全部加载进来。如下图

- <img src=" _media/basis/JVM/lazyloading.png" alt="image-20201214182643957" style="zoom: 50%;" />

- 验证：

- ```java
  public class T008_LazyLoading { //严格讲应该叫lazy initialzing，因为java虚拟机规范并没有严格规定什么时候必须loading,但严格规定了什么时候initialzing
      public static void main(String[] args) throws Exception {
          //P p; //没有new ， 没有访问不会被加载
          //X x = new X();  // new了  会被加载
          //System.out.println(P.i);  // 打印final值不需要加载整个类
          //System.out.println(P.j);  // 非final 会加载
          // 会被加载
          Class.forName("com.mashibing.jvm.c2_classloader.T008_LazyLoading$P");
  
      }
  
      public static class P {
          final static int i = 8;
          static int j = 9;
          static {
              /*
              
              */
              System.out.println("P");
          }
      }
  
      public static class X extends P {
          static { System.out.println("X"); }
      }
  }
  ```


### 7、混合模式

- Java是解释执行的，一个class文件load到内存之后通过java的解释器IntePreter来执行。
- Java中其实有一个JIT的编译器，指的是有某些代码需要把它编译成本地代码，相当于exe。
- 如果有人问Java是编译语言还是解释语言，就回答想解释就解释，想编译就编译，因为默认情况下是混合模式（见下图），混合使用解释器+热点代码编译。
- 什么叫热点代码编译？
  - 就是我们写了一段代码，这个代码里有一个循环或者一个方法在执行的时候刚开始使用解释器执行，结果发现在整个执行代码过程中有某段方法或某循环执行的频率特别高，就会把这段代码编译成本地代码，那么将来再执行这段代码时就执行本地的，效率提升
  - 检测热点代码：-XX: CompileThreshold = 10000
- <img src=" _media/basis/JVM/混合模式.png" alt="image-20201214184255486"  />
- 怎么检测热点代码呢?
  - 就是用一个计数器，每个方法上都有一个方法计数器，循环有循环技术器，结果发现某个方法一秒执行了超过某个值十万次，就要对它进行编译
- 为什么不都编译成本地代码，执行率更高？
  - 因为Java解释器现在效率也已经非常高了，在一些简单的代码执行上不属于编译器，其次如果一段代码执行文件特别多，各种类库有好几十个class很正常，直接编译，编译过程会特别长，所以现在是混合模式，但可以用参数来指定到底使用哪种模式
- 以下是验证程序
  - <img src=" _media/basis/JVM/混合模式验证程序.png" alt="image-20201214184830141"  />

### 8、ClassLoader的parent是如何指定的？

- 任何一个classloader，都有一个parent，我们自定义classloader并没有指定当我的parent是谁，我new自己自定义classloader时。它会调用父亲默认的参数为空的classloader，所以当它调用父亲默认参数为空的classloader时，调用的时下图
- <img src=" _media/basis/JVM/默认指定.png" alt="image-20201214194602237" />
- 方法里面又调用了自己另外一个构造方法，这个构造方法里参数已经指定了parent
- <img src=" _media/basis/JVM/指定父类默认调用.png" alt="image-20201214194235839" style="zoom: 67%;" />
- 用这个构造方法可以指定它的父亲是谁，protected受保护的只能是子类调用，
- 以下是自定义指定父亲的classloader，用自己定义的ClassLoader从ClassLoader里继承，构造方法里面调用super指定parent，要是想没有指定情况下看它默认是谁，用this.getParent()。
- <img src=" _media/basis/JVM/自定义指定parent.png" alt="image-20201214194425986" />

### 9、打破双亲委派机制

在JDK中是有过两次破坏过双亲委派机制

这个双亲委派是在classloader的loadClass源码里面已经设好的

- 如何打破： 重写loadClass()
- 什么时候打破过这种机制在周志明写的《深入理解Java虚拟机》里面写过以下三种情况
  - JDK1.2之前，自定义ClassLoader都必须重写loadClass
  - ThreadContextClassLoader可以实现基础类调用实现类代码，通过thread.setContextClassLoader指定
- 模块化的时候使用热启动，热部署
  
  - osgi、Tomcat都有自己的模块指定classloader（可以加在同一类库的不同版本），两个classloader可以load进同名的类，这是完全没问题的，这里就打破了双亲委派机制
- 代码示例：
  - 以下程序因为只是重写了findClass，所以是不能打破双亲委派机制的
  
  - ```java
    public class T011_ClassReloading1 {
        public static void main(String[] args) throws Exception {
            T006_MSBClassLoader msbClassLoader = new T006_MSBClassLoader();
            Class clazz = msbClassLoader.loadClass("com.mashibing.jvm.Hello");
    
            msbClassLoader = null;
            System.out.println(clazz.hashCode());
                    // 第一个classloader设为空
            msbClassLoader = null;
                    // 在new一个classloader
            msbClassLoader = new T006_MSBClassLoader();
            Class clazz1 = msbClassLoader.loadClass("com.mashibing.jvm.Hello");
            System.out.println(clazz1.hashCode());
                    //两个class是否相等？，true是因为双亲委派，
            // 不管怎样操作，都会让父亲去找，找到了就不重新加载
            System.out.println(clazz == clazz1);
        }
    }
    ```
  
  - 在这定义了一个新的classloader，只是重写了findClass是不能打破的，如果打破见下图
  
  - ```java
    public class T012_ClassReloading2 {
        // 定义自己新的classloader，从classloader继承
        private static class MyLoader extends ClassLoader {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                // 首先去找你要的load class文件，如果没找到让父亲去load，如果找到了自己就load，
                // 我们把是不是已经加载过了这段逻辑去掉了，如果加载同一个class是覆盖不了的，
                // 但直接把classloader整体干掉就行了
                File f = new File("C:/work/ijprojects/JVM/out/production/JVM/"
                        + name.replace(".", "/").concat(".class"));
    
                if(!f.exists()) return super.loadClass(name);
    
                try {
    
                    InputStream is = new FileInputStream(f);
    
                    byte[] b = new byte[is.available()];
                    is.read(b);
                    return defineClass(name, b, 0, b.length);
                } catch (IOException e) {
                    e.printStackTrace();
                }
    
                return super.loadClass(name);
            }
        }
        // 所以Tomcat这么干掉，它直接webapplication的整个classloader全部干掉，重新加载一边
        public static void main(String[] args) throws Exception {
            MyLoader m = new MyLoader();
            Class clazz = m.loadClass("com.jvm.Hello");
    
            m = new MyLoader();
            Class clazzNew = m.loadClass("com.jvm.Hello");
    
            System.out.println(clazz == clazzNew);
        }
    }
    ```

### 10、详解Linking分成三步

- Verification：就是验证文件是否符合JVM规定，格式不对就不会在进行下一步CA FE BA BE
- Preparation：给静态成员变量赋默认值，在第三步里给这些值初始化
- Resolution：叫解析，将类、方法、属性等符号引用解析为直接引用，将常量池中的各种符号引用解析为指针、偏移量等内存地址的直接引用

### 11、详解Initializing

- 调用类初始化代码，给静态成员变量赋初始值

- ```java
  public class T001_ClassLoadingProcedure {
      public static void main(String[] args) {
          /*
              1、count在上面是3，当调用T先把T load到内存进行verification校验，
                  然后进行preparation赋默认值，这时候T是null空值，继续进行resolution
                  在进行Initializing初始化给赋初始值，这时候就是count = 2，然后会调用
                  new T count ++ 所以就是3
           */
          System.out.println(T.count);
      }
  }
  
  class T {
      public static T t = new T(); // null
      public static int count = 2; //0
  
      //private int m = 8;
  
      private T() {
          count ++;
          //System.out.println("--" + count);
      }
  }
  ```

- 静态变量问题：
  - new对象的过程也分为两步
    - 第一步是给这个对象申请内存的过程
    - 申请完内存之后它里面的变量先赋默认值
    - 之后是初始化，赋初始值
    - 将这块内存里的内容赋给引用变量
  - 见上图，假设有一个成员变量private int m = 8，如果用起来需要new对象，首先new出来T的内存，这时候里面的成员变量是默认值0，也就是 m = 0，下一步它才会调用构造方法，之后才会赋初始值m = 8
  
- 小总结：
  - load - 默认值 - 初始值
  - new - 申请内存 - 默认值 - 初始值
  
- 单例模式（双重检查）（面试题）
  - 做成线程安全的，上来先检查INSTANCE == null， 如果等于空的话就是还没有任何线程给它进行初始化，这时候上锁来给它初始化，下面又对它进行了一次检查，主要是第一次检查到加锁期间可能被别的线程给初始化了，所以进行第二次检查，如果依然为空，这个时候就可以安心的初始化了
  - 问题是：这种单例模式代码需不需要加volatile？为什么？
    - 是需要加volatile的，主要是重排的问题
    - 如果不加的话会发生这样一种情况：第一个线程检查完了没有其他线程初始化，然后我加锁且对INSTANCE进行初始化，可不幸的是初始化过程中（初始化一半的时候，就是new了这个对象并且还申请了内存，之后还给里面的成员变量，假设有一个值还给它赋了一个初始值为0），这种情况下，这个时候的INSTANCE就已经指向内存了，所以这个INSTANCE已经不等于空了，如果另外一个线程来了，之后它会首先执行if(INSTANCE == NULL)，这时候的INSTANCE已经不是空值了，处于半初始状态，那我第二个线程就直接开始使用这个初始值了，而不是用那个默认值，解决问题的关键就是加上volatile
    - <img src=" _media/basis/JVM/单例双重检查.png" alt="image-20201214205700751"  />

## 三、Java内存模型（JMM）

在这种高并发下，Java内存模型到底是怎么提供支持的？一个对象出来之后再内存里到底是怎么布局的？

首先看硬件层的并发优化的基础知识，之后是Java内存模型在硬件的基础上是怎样实现的，在之后是指令重排和八大原子指令。推荐《深入理解计算机系统第三版》

### 1、存储器的层次结构

现在的 CPU 硬件和原来的不太一样，它增加了很多结构，这个结构是通过缓存来进行的，CPU 的速度特别快，

CPU 比内存的速度大概要快 100 个数量级，比硬盘快 100w 个数量级，更慢的是远程文件存储，

现在的存储结构是金字塔形的，如果文件特别大，存取速度要求不高，可以存在远程或磁盘上，但对速度要求比较快的时候，就需要将磁盘上的某些文件放在内存里，如果要求更快，内存中的内容就会放到告诉缓存里称之为 L3，这个高速缓存是在主板上的，所以是被所有的 CPU 共享的，但是即便高速缓存对于 CPU 来讲，速度还是慢得多，这时候在 CPU 内部还有两层缓存，叫 L2 和 L1，但实际上最快的是寄存器，是 CPU 内部最核心的存储单元，只存储那么几个数，

所以 CPU 要想读数据的时候：假设有一个数据要被读到 CPU里去执行，首先它被 load 到内存，CPU 要读的时候首先是去高速缓存（L1/L2）去找，如果这个数在里面就直接拿来用，如果没有就会去下一层找，如果有就把它 load 到我最近一级再交给  CPU，CPU下次再访问这个数的时候就比较快了。

所以这个结果就是离 CPU 越近容量越小也更快

<img src=" _media/basis/JVM/存储器的层次结构.png" />

<img src="_media/basis/JVM/对比表.png"/>

### 2、CPU数据不一致问题

- 问题：
  - 首先计算单元与寄存器是最快的，假如有一个数在内存里，这个数它会被 load 到 L3 缓存的上，L2 和 L1 是在CPU 的内部的，这时候会产生一个情况，L3 或值主存里面的这个数会被 load 到不同的 CPU 内部，这个时候加入把CPU1 的 x 值编程 1，CPU2 的 x 值编程2，就会产生数据不一致的问题
  - <img src=" _media/basis/JVM/cache line.png" alt="image-20201214233245401" />
- 解决问题：
  - bus lock 总线锁，每个 CPU 都是通过总线访问 L3，在总线上加锁，意思就是我第一个 CPU 访问数据时另一个不允许访问，是老 CPU 会用，问题就是它把总线锁上了，另一个 CPU 去访问也得等着，效率较低
    - <img src=" _media/basis/JVM/多线程一致性硬件支持总线锁.png" alt="image-20201214233631069" style="zoom: 50%;" />
  - 新的 CPU 用各种各样的一致性协议，一般来说是 MESI，其实和 MESI 相关的协议非常多，比如MSI、MESI、MOSI、Synaps Firefly Dragon这些都是数据一致性协议。因为Intel用MESI协议，大多数人都用Intel的CPU，可以参考：https://www.cnblogs.com/z00377750/p/9180644.html
    - <img src=" _media/basis/JVM/MESI.png" alt="image-20201214235001622"  />
    - 其实就是它给每一个缓存的内容作了一个标记，比如我这个CPU中读了一个数x，这个时候给缓存作了一个标记，这个x和主存的内容相比有没有更改过
      - 更改过标记成M，叫Modified
      - 如果是独享，标记为Exlusive
      - 如果这个内容我读的时候别人也在读就标记成Shared
      - 如果这个内容在我读的时候被别的CPU改过就标记成Invalid，这时候说明我读的数据已经是无效的
    - 通过MESI协议来让各个CPU之间的缓存保持一致性，比如：我观察到我这个数是Invalid，但是我马上要对这个数进行计算的时候，我在去从内存中读一遍它就已经变成有效的了
    - 现代CPU的数据一致性实现 = 缓存锁（MESI等待各种协议） + 总线锁
    - 这几种状态什么情况下进行处理，是根据这个协议来的，也是根据我的指令来的

### 3、缓存行

还是这张图：

<img src=" _media/basis/JVM/cache line.png" alt="image-20201214233245401"  />

- 当我们要把内存中的一些数据放到CPU自己的缓存里的时候，并不是只把这一个数据放进去，例如：用到了int类型 12，它只有4个字节，但读缓存的时候不会只把这4个字节读到缓存中，而是将它和它后面的内容全部读进去，这叫缓存行，读取缓存以cache line为基本单位，目前是64bytes
- 问题：位于同一缓存行的两个不同的数据x、y，被两个不同CPU锁定，产生互相影响的伪共享问题。一个CPU只用x，但读的时候会将x、y一起读进来；另一个CPU只用y，但读的时候x、y也会一起读进来。如果其中一个CPU发生改变，就会通知另一个CPU更新数据。在硬件上的设计很多都是按块来执行的，并不是按字节一位一位来执行的。
- 对比程序：
  - 处于同一缓存行，不同CPU
    - <img src=" _media/basis/JVM/缓存行不对齐.png" alt="image-20201215002558119" />
  - 缓存行对齐
    - <img src=" _media/basis/JVM/缓存行对齐.png" alt="image-20201215003211178" />
- 伪共享，使用缓存行的对齐能提高效率
- disruptor：号称单机效率最高的队列。它里面有一个环，这个环有一个指针，这个指针用的非常频繁，而且在多线程下用，如果被多个线程多个CPU访问，互相之间缓存行有影响的话效率就变低了。这个指针前面放了7个long，后面又放了7个long，就保证了不管是跟前面还是后面对齐，永远不会出现同一缓存行的情况
  - <img src="_media/basis/JVM/disruptor.png"/>

### 4、乱序问题

- CPU为了提高指令执行效率，会在一条指令执行过程中（比如：去内存读数据（慢100倍）），去同时执行另一条指令，前提是两条指令没有依赖关系
- 比如：CPU一次性从里面读了5条指令，读进来之后它这条指令进行计算，算第一条指令时需要去内存里面某个位置读一个数据，CPU比内存快100倍内存起步，也就是我在执行第一条指令时，需要等着这个数读回来后才能读下一条指令，这个时候CPU就要等100倍的时间，所以现在CPU要做的是，这个指令去读数据的这段时间，我就会去分析下一段指令，如果下面一段指令跟我们上面指令没有直接依赖关系，我下面指令先运行，也就是说我等你回来的这段期间，我就可以直接隐形下一段指令，等什么时候你读回来的我再接着运行
- 例如：洗水壶、烧开水、洗茶壶、洗茶杯、拿茶叶、泡茶。它真正执行的时候是乱序的，烧开水的过程中就可以洗茶壶、洗茶杯，类似内部多线程
- 就是我从内存里去读数据，因为他比较慢，所以我同时执行其他的指令
- <img src=" _media/basis/JVM/乱序执行.png" alt="image-20201215004545243" />
- 写指令也有可能出现乱序的问题，CPU在L1和CPU中间还有一个缓存叫WCBuffer合并写，写操作也可以进行合并

### 5、CPU合并写技术

- 当CPU要给某个数做计算，然后把这个结果先写入L1中，假如 L1 里面没有这个值，缓存没命中，就会写到L2里，但写L2的过程中，由于L2的速度慢，所以在写的过程中，假如这个数后续的一些指令也改变了这个值，它就会把这些执行合并到一起，放到合并缓存中，最终计算扔到L2里，其原因还是因为太慢了，我这算了好几步你还没访问到，所以这是一个合并写机制。
- 那写合并是个什么概念呢？
  - 这个合并写会有个细节，这个CPU会把其中的一些个指令写到一个合并写缓存里面叫WCBuffer这个缓存可以说他比L1还要快。但这个buffer非常贵，一般这个buffer只有4个位置。 
  - 假如说我们同时写东西的时候只要说比这四个位置要小，可以同时把这四个位置全写好一次性扔到2里， 这是一种方式；还有一种方式就是一次性我要写六七个，我要分两次才能够把这个合并的内容写到L2里，所以说合并写的这个buffer位置非常少，非常宝贵，当我们往里头更新一些内容的时候假如我一次性写不超过四个， 只写三个、两个、一个。 这个时候合并写的内容满了就会扔到我们的L2里，还有一种就是分成两下，第一下写满过去，再来下写满写过去。 所以这就产生了一个好玩儿的问题，假如我们在程序里头要同时修改6个位置，是把它分成3个组快还是说6个位置同时写快，分成3个组这就保障这三个同时修改的时候这个 buffer满了之后马上写下去，但是我要六个合并到起的话它就要内部进行一些个并发性的东西进行控制，这三个满了才能写必须等着另外三个等等。
  - 我们来看这个实验，这小实验很好玩儿做了两个循环：第一个循环改了六个的位置，第二个里面分了两个循环，第一个循环修改其中三个位置，第二个改其中另外3个位置。如果说这几个位置都一样的情况下那个效率会更高。
    - <img src=" _media/basis/JVM/合并写6个一起存.png" alt="image-20201215010718443"  />
    - <img src=" _media/basis/JVM/演示结果.png" alt="image-20201215010925698" />
    - 因为这种写法充分利用了合并写
- 证明乱序执行
  - <img src="_media/basis/JVM/证明乱序执行.png"/>
  - 只要又一次出现了0,0这种情况就说明一定发生了重排
  - <img src="_media/basis/JVM/乱序执行结果.png"/>

### 6、如何保证特定情况下不乱序——volatile

volatile保证了有序性

<img src=" _media/basis/JVM/有序性保障.png" alt="image-20201215012133170" />

- 硬件层面：
  - 在你保证有序性的同时硬件层面指令，汇编指令做了这样两件事，第一个是加锁，但是为了提高效率在CPU指令级别，很多CPU做了同一件事就是加内存屏障，这是CPU级别的内存屏障，不同CPU它的内存屏障指令是不一样的，有的设计非常复杂，有的也非常简单，拿Intel的来说，只有三条指令，见上图
    - sfence屏障（栅栏的意思）存屏障
    - ifence读屏障
    - mfence是两个加起来
- 内存屏障到底什么意思？
  - 有两个指令不可以重排，中间加个屏障sfence，不能把上面的挪到下面去，也不可以把下面的挪到上面，这是写指令
  - 读指令，我们要对一个内存里的指令，有一条指令读了它，另外一条也读了他，这两条指令不可以重排                                                                                                             
  - 读写指令，上面读写操作都得完成，下面才能继续运行
- 同一个CPU指令来说，读第一个和读第二个有可能会产生前后关系，volatile一个内存，在Java实现volatile内存的时候，实际上前后都加了内存屏障，不能混排
- <img src=" _media/basis/JVM/lock汇编.png" alt="image-20201215013747603"  />
- jvm级别的内存屏障或者叫jvm有序性，它的硬件级别实现并不一定依赖于硬件级别的内存屏障，还可以依赖于硬件级别的lock指令，所以它具体的jvm级别的东西和硬件级别的东西不是一回事

## 四、内存屏障与JVM指令

乱序之后产生的新问题：即使在某些情况下不能乱序存在，如果乱序就会产生数据不一致，如果这样怎么保证不乱序？

### 1、硬件层面CPU处理

- 保障有序性有特定的指令，在不同的CPU有不同的有序性保证的指令
- Intel CPU主要有三条指令，第一种叫sfence屏障save；第二种是Ifence屏障load；第三种是mfence，来保障指令有序性
- 有序性指的是在特定位置插进去内存屏障
- 内存屏障，可以认为就是一个屏障，两方越不过来，屏障两端的顺序不可能发生改变，屏障有各种各样的，在硬件层面上（X86）就如下三种
  - sfence:  store| 在sfence指令前的写操作当必须在sfence指令后的写操作前完成。
  - lfence：load | 在lfence指令前的读操作当必须在lfence指令后的读操作前完成。
  - mfence：modify/mix | 在mfence指令前的读写操作当必须在mfence指令后的读写操作前完成。
- 原子指令
  - 如x86上的”lock …” 指令是一个Full Barrier，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。Software Locks通常使用了内存屏障或原子指令来实现变量可见性和保持程序顺序
  - 例如：在CPU层面实际上还有这种操作lock...add...， lock是一条指令，lock后面跟的是另外一条指令。这句话的意思是当这条指令完成之前你对某个内存实际上是锁定的，如果你前面加了一个lock的话说明在执行这条指令的过程当中这个内存不能被其他人所改变。所以，从CPU的层面上来讲这几种方式保证了不乱序

### 2、JVM级别如何规范

JVM本身是软件层级，JVM本身是跑在操作系统上的软件，JVM只是做了一些规范，以下读到的屏障只是JVM规范，它的实现是JVM自己实现的，一定是依赖于硬件的实现，所以当看到以下的东西时要知道这是个虚的东西，当看到sfence、Ifence、mfence时就是实际的东西

- LoadLoad屏障
  - 对于这样的语句Load1；LoadLoad；Load2
  - 在Load2及后续要读取的数据被访问钱，保证Load1要读取的数据已经读取完毕
- StoreStore屏障
  - 对于这样的语句Store1；StoreStore；Store2
  - 在Store2及后续写入操作执行前，保证Store1的写入操作对其他处理器可见
- LoadStore屏障
  - 对于这样的语句Load1；LoadStore；Store2
  - 在Store2及后续写入操作执行前，保证Load1要读取的数据已经读取完毕
- StoreLoad全能屏障
  - 对于这样的语句Store1；StoreLoad；Load2
  - 在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见

### 3、volatile实现细节

现在有一个编程趋势，大家认为volatile没必要用了，因为synchronize经过优化之后，它的效率现在也很高，并不比volatile差多少，所以不是在那种效率追求到极致的情况下，volatile干脆也不用了，因为硬件内存屏障指的是X86的，各种各样的内存屏障的实现在不同CPU上是不同的，所以volatile在不同的CPU上肯定也是不一样的。

- 字节码层面
  - ACC_VOLATILE
  - 我们在源码中写volatile，之后会编译为字节码bytecod，如图
  - <img src=" _media/basis/JVM/volatile源码.png" alt="image-20201215131850992" />
  - <img src=" _media/basis/JVM/修饰符.png" alt="image-20201215132441883" />
  - <img src=" _media/basis/JVM/volatile字节码.png" alt="image-20201215132650398" />
- JVM层面
  - JVM的实现层级实际上对所有volatile的写操作前面加了StoreStoreBarrier，后面加了一个StoreLoadBarrier；在所有volatile读操作前面加了LoadLoadBarrier，后面加了一个LoadStoreBarrier
  - 想一下，如果前面加了LoadLoad，上面的L1和下面的读操作一定不能重排序，下面加了一个LoadStore，上面这个读操作和下面任何的这种修改的操作肯定不能重排序。所以就保证了在读的过程中不会有任何不一致的情况，同样的写操作也是一样
  - volatile的实现细节就是：volatile内存区的读写都加屏障
  - <img src=" _media/basis/JVM/volatile实现细节.png" alt="image-20201215153138291"  />
- OS和硬件层面
  - 硬件层面需要用hsdis -HotSpot Dis Assembler这个工具，叫做HotSpot的反汇编，不是Java的反汇编，是虚拟机的反汇编，是把虚拟机编译好的字节码再进行反汇编。用来观察虚拟机编译好的那些字节码在CPU级别是用什么样的汇编指令来完成的，见下图，讲了在Windows上的实现，在Linux上有可能是另外的一个实现。Windows是lock指令实现的
  - <img src=" _media/basis/JVM/JVM反汇编.png" alt="image-20201215153603954" />
- 所以volatile的实现细节要分不同的层次，它首先写源码的时候把它编译完成后在字节码上加了ACC_VOLATILE这么一个标志。当虚拟机读到这个标志的时候它就会在内存区读写之前都加屏障。加完之后虚拟机和操作系统去执行这个虚拟机程序后读到这个屏障的时候，在硬件层面Windows上是用lock指令实现的，但在Linux上实现是上面一个屏障，下面一个屏障，最后一条lock指令，中间才是volatile的区域

### 4、synchronize实现细节

- 字节码层面
  - 加上了ACC_SUNCHRONIZED修饰符
  - 实际上在字节码层面就是monitorenter和monitorexit指令，这两个就可以实现synchronize
  - <img src=" _media/basis/JVM/synchronize验证.png" alt="image-20201215154320479"  />
  - 看下图的指令，加了两条指令monitorenter（监视器开始），monitorexit（监视器退出），有两条monitorexit的原因是，发现异常之后它会自动退出的意思。同步快synchronize产生一个monitorenter，但产生异常后会自动退出，所以有两条monitorexit指令
  - <img src=" _media/basis/JVM/synchronize字节码.png" alt="image-20201215154422074" style="zoom: 67%;" />
- JVM层面
  - JVM层面实际上就是C和C++调用了操作系统提供的同步机制
  - 当牵扯得到某个具体系统相关的内容，在Linux和Windows上的具体实现是不一样的，Linux和Windows内核都会给大家提供同步机制，所以这里是C和C++调用了操作系统提供的同步机制
- OS和硬件层面
  - X86：lock cmpxchg xxx
  - 硬件层面实际上是lock一条指令
- synchronize在字节码层面如果你是方法的话就直接加了一个ACC_SUNCHRONIZED修饰符，如果是同步语句块的话就是用monitorenter和monitorexit，在JVM层面当它看到了字节码指令之后，对应的是C和C++调用了操作系统提供的同步机制，同步机制是要依赖于硬件CPU的，CPU级别是使用lock指令来实现的
- 比如，我们要在synchronize某一块内存上加个数，把i的值从0变成1，从0变成1的这个过程如果放在CPU去执行，可能有好几条指令或者不能同步，所以用lock指令。cmpxchg前面如果加了一个lock的话，指的是后面的指令执行的过程中将这块内存锁定，只有我这条指令能改，其他指令是改不了的。

### 5、原子操作、内存模型、重排规则

- Java8大原子操作，
  - 《深入理解虚拟机》的第364页说了这个问题，其实这个模型已经被Java给放弃了，因此了解就行，JMM没有变化，只是描述的方式发生了变化
  - <img src=" _media/basis/JVM/8大原子操作.png" alt="image-20201215174916193" />
- Java并发内存模型本身是没有任何变化的，这个Java面对线程模型层级的一个抽象，具体的这块怎么去实现要看JVM的具体实现、操作系统的实现、硬件层级的实现
- <img src=" _media/basis/JVM/Java内存模型.png" alt="image-20201215175010550" />
- 这个内容很可能读到hanppens-before原则，这是显现Java语言的一个规范，有些指令不允许进行重排，这也是有你的Java虚拟机去实现的。这个问题的优先级不高，这只是Java语言上的一个要求，那么具体的JVM实现，本质上的实现前后不能错顺序
- <img src=" _media/basis/JVM/JVM排序规则.png" alt="image-20201215175129468" />
- 另外一个说法as if serial（好像是序列化执行的）说的不管如何重排序，单线程执行结果不会改变。
  - 本来正常代码应该是a = b、b = c、c = d这么一步一步的，但是你真正CPU执行的时候是乱序的，只要保证最后的结果不变，那as if serial就OK了

### 6、对象相关问题 —— 对象的创建过程

问题：

- <img src=" _media/basis/JVM/关于对象的问题.png" alt="image-20201215160221914" />

- <img src=" _media/basis/JVM/对象的创建过程.png" alt="image-20201215175316262" />
- 创建对象 ——> 然后需要申请内存 ——> 成员变量赋默认值 ——> 然后调用构造方法 ——> 在字节码层面，调用构造方法时，把成员变量设为初始值 ——> 接下来调用构造方法语句super调用父类
- 观察虚拟机的配置： java -XX: +PrintCommandLineFlags -version
  - <img src="_media/basis/JVM/虚拟机配置.png"/>

### 7、对象相关问题 ——对象的内存布局

- 作为对象的内存布局来讲分为两种，

  - 第一种叫普通对象，
  - 第二种叫数组对象

- 普通对象：

  - 第一是对象头，在HotSpot里面称为markWord，长度是8个字节
  - 第二个是ClassPointer指针：-XX:+UseCompressedClassPointers（压缩指针），长度为4个字节，不开启压缩是8个字节
  - 第三个是实例数据，引用类型：-XX:+UseCompressedOops（压缩普通对象直针），长度为4个字节，不开启压缩为8个字节，Oops Ordinary Object Pointers
  - 第四个是Padding（对齐），这个对齐是8的倍数
  - 解读：普通对象，首先由有一个对象头markWord8个字节，第二个这个对象它是属于哪个class的，它有一个指针是ClassPointer，这个指针指向你要的class对象，接下来实例数据也就是成员变量还有引用类型，第四个就是Padding对齐，主要是因为算出来正好15个字节，但是作为64位机器，它是按块来读，不是按字节来读，所有有一个对齐是8的倍数

- 数组对象

  - 对象头：markWord 8字节
  - ClassPointer指针
  - 数组长度：4字节
  - 数组数据
  - 对齐 8的倍数
  - 解读：数组对象多了一项，第一个对象头一样，第二个ClassPointer表示这个对象数组里装的是哪种类型的东西，第三个多了一个数组长度4字节，第四个是数组数据，第五个对齐8的倍数

- 观察Object大小的实验：Java不像C和C++有sizeOf，它本身有一个agent机制

  1. 新建项目ObjectSize（1.8）

  2. 创建文件ObjectSizeAgent

     - Agent的机制：假如有一个JVM虚拟机，还有一个class要load内存，在load内存中可以有一个Agent代理，可以截获这些class文件001 010 等等，可以对其任意修改，当然我就可以在这里读出来整个object的大小

     -  

       ```java
       package com.mashibing.jvm.agent;
       
       import java.lang.instrument.Instrumentation;
       
       public class ObjectSizeAgent {
           private static Instrumentation inst;
       
           public static void premain(String agentArgs, Instrumentation _inst) {
               inst = _inst;
           }
       
           public static long sizeOf(Object o) {
               return inst.getObjectSize(o);
           }
       }
       ```

  3. src目录下创建META-INF/MANIFEST.MF

     - ```java
       Manifest-Version: 1.0
       Created-By: mashibing.com
       Premain-Class: com.mashibing.jvm.agent.ObjectSizeAgent
       ```

     - 注意Premain-Class这行必须是新的一行（回车+换行），确认idea不能有任何错误提示

  4. 打包jar文件（拿到另一个项目中用）

  5. 在需要使用该Agent Jar的项目中引入该jar包

     - project structure —— project settings —— library添加该jar包

  6. 运行时需要该Agent Jar的类，加入参数

     - ```java
       -javaagent:C:\work\ijprojects\ObjectSize\out\artifacts\ObjectSize_jar\ObjectSize.jar
       ```

  7. 如何使用该类：

     - ~~~java
       ```java
          package com.mashibing.jvm.c3_jmm;
          
          import com.mashibing.jvm.agent.ObjectSizeAgent;
          
          public class T03_SizeOfAnObject {
              public static void main(String[] args) {
                  System.out.println(ObjectSizeAgent.sizeOf(new Object()));
                  System.out.println(ObjectSizeAgent.sizeOf(new int[] {}));
                  System.out.println(ObjectSizeAgent.sizeOf(new P()));
              }
          
              private static class P {
                                  //8 _markword
                                  //4 _oop指针
                  int id;         //4
                  String name;    //4
                  int age;        //4
          
                  byte b1;        //1
                  byte b2;        //1
         
                  Object o;       //4
                  byte b3;        //1
          
              }
          }
       ~~~

     - 看下图的运行结果：Object大小是16个字节，int类型数组16个字节，P对象32个字节

     - <img src="_media/basis/JVM/object运行结果.png"/>

     - 分析：Object是16个字节，因为一个markWord是8个字节，下面是一个class指针，我们运行Java虚拟机的时候默认是把压缩指针打开的，就会把本来8个字节的指针压缩成4个字节，8+4=12，Java中的内存是以8的倍数来分配的，所以分配的内存是16个字节

     - 分析：int数组16个字节，头8个字节，class指针压缩后4个字节，数组长度4个字节，一共16字节

     - 分析：P对象，上述代码中有7个成员变量，头8字节+class指针4字节+int4字节+string是引用类型应该占8个字节，但是UseCompressedOops参数开启了压缩，是4个字节+int+byte是1个字节+byte+object+byte+对齐  = 32字节

     - Java Agent有很多用处，阿里的调试GC的工具就是拿这个来完成的

- Hotspot开启内存压缩的规则（64位机）

  - 4G以下，直接砍掉高32位
  - 4G - 32G，默认开启内存压缩 ClassPointers Oops
  - 32G，压缩无效，使用64位
    内存并不是越大越好！

3. 对象头包括什么

   - 对象头非常复杂，每个版本的实现不同，目前看的是1.8的实现。如果具体了解需要看Hotspot源码，见下图
   - <img src="_media/basis/JVM/Hotspot源码.png"/>
   - markWord在不同的对象状态时里面记得内容时是不同的
   - <img src="_media/basis/JVM/markWord64位.png"/>
   - markWord里面装的是什么？
     - 第一：锁定信息，2位，代表对象有没有被锁定，严格来讲是3位，还有1位来表示是否是偏向锁；
     - 第二：GC的标记，它被回收多少次了，分带年龄
     - 普通的状态下，无锁的状态，锁标志位是01，表示没有synchronize，无锁时对象的hashcode也不是记录在这里的，而是被调用的时候才会被记录，这个hashcode有两种情况，第一种情况hashcode这个方法被重写了，第二个是没有被重写过，它是默认根据对象后面内存布局来计算一个值，25位，称之为identityHashCode
     - 由于synchronize升级的过程中有一个偏向锁，存在一个锁升级的过程，就是说我这里被某一个线程占用了，它只偏向于这个线程，这个线程下次再来就不需要加锁，所有严格来讲它头上有3位来代表锁，其中有1位代表是否偏向锁，epoch是保持变量状态的时间戳
     - 总而言之，不同的状态面前这个markWord每一位表示的是不同的内容，这点比较复杂。
     - 为什么GC年龄默认是15？ 因为头上只有4位，4位代表的最大范围是0~15
     - 问题：当Java处在偏向锁、重量级锁状态，hashcode值存在哪？
       - 简单答案：当一个对象计算过identityHashCode之后，不能进入偏向锁状态
       - https://cloud.tencent.com/developer/article/1480590
       - https://cloud.tencent.com/developer/article/1484167
       - https://cloud.tencent.com/developer/article/1485795
       - https://cloud.tencent.com/developer/article/1482500

4. 对象定位的方式

   - 这个是参考资料: https://blog.csdn.net/clover_lily/article/details/80095580
   - 对象定位也有两种: 1、句柄池，2、直接指针
   - 这两种说法在《深入了解Java虚拟机》那本书上也讲了，其实概念就是当我们new出来个对像T t = new T();，这个小t是怎么找到这个对象的，有两种方式：
     - 第一种是通过句柄池， 通过间接指针，间接指针的意思是：它第一步把小 t  指向两个指针，这两个指针其中一个指向对象， 另外一个指向t.class. 这个就是中间隔了一下
       - <img src=" _media/basis/JVM/句柄池.png" alt="image-20201215195705166" />
     - 第二种直接指向对象然后在指向T.class,他俩没有优劣之分，有的虚拟机实现用第一种有的用第二种，Hotspot用的是第二种，第二种效率比较高直接找到对象，第一种他要找一个指针再找下一 个，但是第一种GC的时候效率比较高，因为GC后面牵扯到算法我们讲GC的时候再说。
       - <img src=" _media/basis/JVM/直接指针.png" alt="image-20201215200109812" />

5. 对象应如何分配

   - 实际上对象的分配过程非常复杂，观察下图:首先new一个对象的时候先往栈上分配，栈上如果能分配下就分配在栈上，然后栈弹出对象就没了，如果栈上分配不下，特别大直接分配到堆内存，老年代。如果不大，首先会进行线程本地分配，线程本地分配能分配下就分配，分配不下找伊甸区然后进行GC的过程，GC过程年龄到了就直接到老年代了，如果年龄不到的话反复GC一直到年龄到了为止。
   - <img src="_media/basis/JVM/对象分配过程.png"/>
   - 所谓的对象分配过程指的是这个过程，这个过程里有一些详细的细节，比如栈上分配到底是怎么分配的，关于这件事等我讲CG的时候再来详细的讲这个问题。

## 五、Java运行时数据区和常用指令

JVM Runtime Data Area and JVM Instructions

<img src="_media/basis/JVM/指令实例程序.png"/>

一般来说一个class 经过load、link、Initializing后到虚拟机的运行时引擎，开始运行，这时内存中是什么情况？就是以下内容 run-time data area

推荐参考资料，最严谨的资料是 Java Virtual Machine Specification。所有的资料都要以这个为准，关于Java语言的应该看Java Language Specification

### 1、运行时数据区

<img src=" _media/basis/JVM/运行时数据区.png" alt="image-20201215211813778" />

- ProgramCounter简称PC，指的是存放下一条指令位置的这个么一个内存区域

  - <img src=" _media/basis/JVM/PC计数器.png" alt="image-20201215215424560" />

  - PC程序计数器：存放指令位置

    ```
虚拟机的运行，类似于这样的循环：
    while( not end ) {
	取PC中的位置，找到对应位置的指令；
    	执行该指令；
	PC ++;
    }
    ```
```
  
- Heap，堆，在GC 的时候详细介绍
  - 在线程间共享
  - <img src="_media/database/JVM/Heap.png" alt="image-20201215215941451" />

- 栈，在内存中有两块
  - （<span style='color:red'>重点</span>）JVM Stacks：Java运行的时候写这个方法，每一个线程对一个一个栈，这是java内部的jvm虚拟机里面管理的
    - JVM Stacks的栈是归线程独有的，线程栈中装的是栈帧
    - <img src="_media/database/JVM/JVMStacks.png" alt="image-20201215215748860" />
    - <img src="_media/database/JVM/Java栈.png" alt="image-20201219205255434" />
  - Native Method Stacks：指的是本地方法，也就是C和C++，Java调用了JNI，等同于Java自身的这个栈，一般不管它
- Direct Memory，直接内存，在Java1.4之后增加了一个内容，属于NIO的内容，一般情况下我们所有的内存都归Java虚拟机直接管理，为了增加IO的效率，增加了这个直接内存，就是从Java虚拟机内部可以直接访问操作系统管理的那部分内存的，可以理解为用户空间直接去访问内核空间的一些内存，Direct Memory不归JVM管理，归操作系统管理
  - 在原来的IO里面，比如网络要访问一个数据，访问的过程如下：网络传过来一个数据，这个数据首先在内核空间放到一块内存里，JVM要使用时，直接将这块内存拷贝到JVM空间，有一个内存拷贝的过程，如果使用这种模型，那么每传一次数据都要拷贝一份，效率比较低
  - 而Java1.4之后增加了NIO，NIO使用的是直接内存，直接内存就省下了拷贝的过程，可以直接访问内核空间中网络传过来的这块网络缓存区上的内容，叫做zeroCopy 零拷贝
- Method Area方法区，这个方法区里面装的是各种各样的class和常量池的内容叫 run-time constant pool
  - 归所有线程共享的，方法区里面装的是每一个class的结构
  - ![](_media/database/JVM/methodarea.png"/>
  - 注意：方法区是一个逻辑上的概念。在1.8之前和之后是对它不同的实现
    - 在1.8之前称它为 Perm Space 永久代
      字符串常量位于PermSpace
      FGC不会清理
      大小启动的时候指定，不能变
    - 在1.8之后称它为Meta Space 元数据区
      字符串常量位于堆
      会触发FGC清理
      不设定的话，最大就是物理内存
- run-time constant pool 运行时数据区 常量池
  - class文件格式中有一项叫常量池，常量池的内容在运行时就放在了run-time constant pool里面
  - ![](_media/database/JVM/运行时数据区常量池.png"/>
- 总结以上内容：
  - 每一个线程都有自己的PC，都有自己的JVM Stacks（里面装的是栈帧），还有自己的Native Method Stacks。但它们共享的是堆和Method Area
  - <img src="_media/database/JVM/运行时数据区总结.png" alt="image-20201215221553951" />
  - 问题：为什么每个线程都有自己的PC，共有一个PC不行吗？
    -  一个线程执行完了，接下来CPU切换到另外一个线程去执行， 另外一个线程执行完了之后又切回来的时候这个线程执行到哪里了CPU是不知道的，所以需要有一个位置记录着说个线程执行到那条指令，所以每一个线程都得有自己的PC。

### 2、（重点）栈帧 —— JVM Stacks

<img src="_media/database/JVM/栈帧详细.png" alt="image-20201215223447504" />

![](_media/database/JVM/栈帧.png"/>

- 栈帧Frame一个JVM Stacks栈里面是一个个栈帧， 方法启动就会有，一个方法对应一个栈帧， 每个栈帧里面都有自己的内容

  - 第一个是Local Variables局部变量表
  - 第二个叫Operand Stacks操作数栈，每个局部变量都有自己的Operand Stacks
  - 第三个叫Dynamic Linking动态链接
    - https://blog.csdn.net/qq_41813060/article/details/88379473 
    - jvms的2.6.3这一 节是专门讲的Dynamic Linking，指的是一个线程有自己的线程栈，每一个线程栈里面装着一 个个的栈帧，每个栈帧里面有自己的操作数栈，还有一个叫Dynamic Linking的东西，Dynamic Linking是指到我们运行时常量池（class文件里面常量池里面的那个符号链接），这个方法叫什么名，方法的类型是什么，找到这个符号链接，看看它有没有解析，如果没有解析就进行动态解析，如果已经解析了就拿过来使用，所以Dynamic Linking就是指的这个东西。
    - 比如：a方法调用了b方法，那么b方法在哪？就需要到常量池里面去找，这个过程link就叫Dynamic Linking
  - 第四个叫Return Address返回值地址
    - 有一个方法a调用了b，如果有返回值的情况，你这b执行完了之后把返回值放哪，执行结束后应该回到那个地址上去继续执行，这个叫return address。
  - 整个方法执行中会跟Heap堆打交道，放到堆中，会跟Method Area方法区打交道，对应哪些常量，Local去方法区找。

- 结合程序解释局部变量表

  - 通过Jclasslibrary来解析他二进制码，之后点到main方法打开来。他有两张表，这个方法里面其实记录这两张表，第一张表呢是LineNumberTable记录的是行号，第二个就是LocalVariabletable局部变量表。概念就是这个方法内部使用到的变量。
  - 那我这个方法使用到了几个内部的局部变量？第一个是args、第二个是i。所以局部变量表，指的就是我们当前这个方法，这个栈帧里面用到的那些局部变量，栈帧弹出就没了。
  - ![](_media/database/JVM/局部变量表指令.png"/>

- 题目解析，在上述程序执行过程是什么样的？

  - <img src="_media/database/JVM/面试题解析.png" alt="image-20201216001631732"  />
  - ![](_media/database/JVM/程序解析.png"/>
    - 0 bipush 8 ：将8作为byte类型压栈，byte值会扩展成int类型
    - 2 istore_1 ：将栈顶的值8出栈，放到局部变量表为1的位置上，至此 int i = 8 就完成了
    - 3 iload_1  ：将局部变量表位置为1的值拿出来，压栈，此时栈中的数依然是8
    - 4 iinc 1 by 1 ：把局部变量表为1的位置上的数加1，此时局部变量表中i的值为9，栈中依然为8，i++已经在局部变量表中完成
    - 7 istore_1 ：将栈顶的值8出栈，放到局部变量表为1的位置上，赋值回来，局部变量表中i的值从9又变为8
  - ![](_media/database/JVM/问题解析2.png"/>
    - 与之前的不同的是 iinc 1 by 1 在 iload_1 之前执行了

- 对于一个机器的设计来说到目前有两种方式，设计它的指令集

  - 基于寄存器的指令集：我们的JVM就是基于栈的指令集
    - 比较简单，压栈出栈
  - 基于栈的指令集：汇编语言就是各种各样的寄存器
    - 根据寄存器不同的数量、大小，相对复杂，但速度快
  - Hotspot中的Local Variable Table 就类似于寄存器临时的存放世数据然后在我运算的完成之后再把数据存回来。

- 单条指令都有可能不是原子性的， 更不要说多条指令了。

- 我们把刚才讲的概念梳理一下：

  - Jvm Stacks：面装的是一个个的栈帧， 这个栈帧方法启动就会有，所以大家记住一个方法对应一个栈帧，每个栈帧里面都有自己的局部变量表、都有自己操作数栈，在你整个执行的方法中它会和堆打交道，把new出来的东西扔到堆里，它会和方法区打交道。

  - 案例一：

    - <img src="_media/database/JVM/例子1.png" alt="image-20201216005236777" style="zoom: 50%;" />

  - 案例二：

    - <img src="_media/database/JVM/例子2.png" alt="image-20201216005853623"  />

  - 案例三：

    - 下图的这条指令：Hello_02 = new Hello_02(); h.m1(); ，m1是一个方法，方法中有一个int i = 200，main方法中new了一个对象h，调用了m1方法。
    - 指令执行的过程：首先m1对应的指令sipush -> istore_1 -> return，这时由于main方法调用了m1，所以整个线程栈里有两个方法栈帧，第一个是main栈帧，第二个是m1栈帧，当调用m1时，main先暂停，产生新的m1栈帧，等m1执行结束后就弹出，继续下一个栈帧执行。
    - <img src="_media/database/JVM/案例三.png" alt="image-20201216201122749"  />
    - new指令：new出来hello_02对象
    - dup指令：在我们堆内存里面new了一个对象，之后这个对象的地址会压栈。此时为对象赋值叫做默认值，将对象放在对应的栈空间之后，dup指令会在栈中在复制一个对象的内存地址
    - invokespacial指令：执行特殊的方法，意思是执行默认的构造方法，执行构造方法的过程会把dup复制的内存地址弹出做运算，因为执行构造方法时，需要告诉它对哪个对象执行，所以这时对象中的值就会赋初始值且调用完构造方法了，此时初始化完成。
    - astore_1指令：将栈中的对象地址弹出到局部变量表为1的位置，就是h。以上就是new出来对象复制给h的过程
    - aload_1指令：将局部变量表中的h压栈
    - invokespacial指令：调用了m1方法，这时栈中的h需要出栈，之后执行的就是下一个栈帧
    - sipush -> istore_1 ->return 回到invokespacial的位置继续执行，之后return
    - 注意：DCL使用volatile的原因，是因为new 到 invokespacial 之间容易指令重排

    - aload_ 1 h压栈调用m1 -> invokespacial 搞定。那第二个如果有一个
      return的情况，它的返回值是int如果有一个return怎么办， 这个就相对绕一些，大家看着思考- -下， 其
      实和前面一样多了一个pop, 为什么会多一个pop, 是因为m1方法在返回的时候往main栈帧的那个栈
      顶上放了一个100，main会调用m1, m1的返回值是100，这个100会放到main方法的栈顶上，然后在
      main方法调用完成之后由于这个栈顶上有一个返回值，我先把它弹出来，然后在return. 理解完这个意
      思之后你再看这边第三个，分析- -下m1执行完成之后， 它会放到我们main方法的栈顶，所以main方法
      这些东西我就不说了，第一句话就完成了，接下调用m1方法，m1方法完成之后会往栈顶放一个100，
      但是由于这个100被我们赋值给i了，所以istore_ 2直接弹出来扔到我们局部变量表第二一个位置也就是
      i。第0个位置是args/第1个位置是h/第2个位置就是i。返回值放到了 栈帧的顶部

  - 案例四：

    -![image-20201216205119323](_media/database/JVM/案例四.png"/>

  - 案例五：递归

    - 递归方法，方法都叫m，只不过参数是不一样的
    - ![](_media/database/JVM/递归.png"/>
    - iload_1指令：局部变量表中第0个位置是this，第1个位置是n，n的值是3，所以就是把3拿出来压栈
    - iconst_1指令：是把产量值1压栈
    - if_icmpne 7：指令：if判断语句，icmp将两个值比较，ne notequ不等，将3和1两个值弹出比较，如果不等的话就跳到第7条
    - isub指令：将3和1弹出做计算，计算结果压栈

### 3、invoke指令

- InvokeStatic
  - <img src="_media/database/JVM/InvokeStatic.png" alt="image-20201216211719035" style="zoom: 67%;" />
  - main方法中调用了m方法，m方法是静态方法，所以调用m方法的指令是InvokeStatic
  - 调用静态方法会使用InvokeStatic
- InvokeVirtual
  - <img src="_media/database/JVM/InvokeVirtual.png" alt="image-20201216212201938" />
  - InvokeVirtual自带多态，就是new了一个对象去调用m方法，在栈中压入的是哪个对象，就调用哪个对象的m方法
- InvokeInterface
  - <img src="_media/database/JVM/InvokeInterface.png" alt="image-20201216212725141" />
  - 作为一个Interface调用具体方法
- InvokeSpecial
  - <img src="_media/database/JVM/InvokeSpecial.png" alt="image-20201216212512074" />
  - 多数方法都是调用的InvokeSpecial ，代表的是可以直接定位的，不需要多态的方法
  - private方法 和 构造方法
- InvokeDynamic（JVM最难指令）
  - <img src="_media/database/JVM/InvokeDynamic2.png" alt="image-20201216213519545" />
  - InvokeDynamic，在1.7之前是没有的，后来加进来的指令。当看到lambda表达式的时候这条指令开始起作用，因为Java开始支持动态语言了，动态语言就会产生很多的class。所谓的动态class指的是在运行时产生的
  - lambda表达式或者反射或者其他动态语言scala、kotlin, 或者CGLib ASM,动态产生=的class,会用到的指令。
  -![image-20201216213754255](_media/database/JVM/运行结果.png"/>
  - 一旦有任何一个类有Lambda表达式的时候，它一定会产生一个内部类，这个内部类叫Lambda，而这个Lambda它自己的内部第一个是1747.....这是内部类的内部类
  - 注意：for(;;){I i = C::n;}，在循环中写Lambda，会产生很多很多的class，这些class会放在Method Area <1.8 时 Perm Space 产生FGC 不回收

## 六、JVM调优必备理论知识

GC Collector —— 三色标记

### 1、什么叫垃圾？

- <img src="_media/database/JVM/garbage.png" alt="image-20201217124110413" />
- 没有引用指向的任何对象都叫垃圾，比如：丢线球，中途没有线拉着的都叫垃圾
- Java和C++对垃圾处理的区别
  - Java：只管扔垃圾就行，有人帮忙处理
    - GC处理垃圾
    - 开发效高，执行效率低
  - C++比较精确，立马回收

### 2、怎么找到这个垃圾？

一般有两种算法

- 第一种：reference count 引用计数
  - 有一个引用指向一个对象，在它头上写了一个数字，有多少引用指向它数字就写几，当数字变成0的时候，就是没有任何引用指向，就是垃圾
  - <img src="_media/database/JVM/找垃圾1.png" alt="image-20201217124241492" />
  - 不能解决的问题：例如循环使用 A -> B - > C -> A，大家都是1，对于RC来说大家都不是垃圾，可是没有其他引用指向这一团，这几个都是垃圾，可以采用另一种方式 Root Searching 根可达算法
  - <img src="_media/database/JVM/产生循环问题.png" alt="image-20201217124352023" />
- 第二种：Root Searching 根可达算法
  - 从根上开始搜，Java程序一个main方法开始运行，一个main方法会启一个线程，这个线程里面会有线程栈，里面会有main栈帧。 
    - 从这个main里面开始的这些对象都是我们的根对象，当然这个main方法调用了别的方法，那别方法也是我们会引用到的，都是有用的对象。
    - 另外一个叫静态变量，一个class它有一个静态的变量。 load到内存之后马上就得对静态变量进行初始化，所以静态变量访问到的对象叫做根对象。
    - 还有常量池，指的是如果你这个class会用到其他的class的那些个类的对象，这些是根对象。
    - JNI，指的是如果你调用了C和C++写的那些本地方法所用到的那些个类或者对象。
  - <img src="_media/database/JVM/根可达.png" alt="image-20201217124706513" />
  - 总而言之，当个程序起来之后马上需要的那些个对象就叫做跟对象，Root Searching是首先找到根对象，然后根据这根线一直往外找到那些有用的。
  - 名词解释:
    - 根可达算法：是从根上对象开始搜索
    - 线程栈变量：一个main方法开始运行， main线程栈中的变量调用了其他方法，main栈中的方法访问到的对象叫根对象
    - 静态交量：T.class对静态变量初始化，能够访问到的对象叫做根对象
    - 常量池：如果一个class能够用到其他的class的对象叫做根对象
    - JNI指针：如果调用了本地方法运用到本地的对象叫做根对象

### 3、常用的垃圾回收算法

GC Algorithms(常见的垃圾回收算法)，找到这个垃圾之后怎么进行清除的算法。GC常用的算法有三种如下:

1. Mark-Sweep（标记清除）
   - 首先找到那些有用的，没用标记出来后清掉
   - <img src="_media/database/JVM/标记清除.png" alt="image-20201217125333457" />
   - 标记清除算法产生的问题。见下图，我们从GC的根找到那些不可回收的，绿色都是不可回收，黄色都是可以回收的，我们把它回收之后就变成空闲状态了。在存活对象较多的情况下效率比较高。需要经过两遍扫描， 第一遍扫描是找到那些有用的，第二遍扫描是把那些没用的找出来清理掉。执行效率上偏低一些， 容易产生碎片。
   - <img src="_media/database/JVM/标记清除2.png" alt="image-20201217125537725" />
2. Copying（拷贝）
   - 将内存一分为二，之后把有用的拷贝到下满绿色区域，完成后把上面全部清掉，如图
   - <img src="_media/database/JVM/拷贝1.png" alt="image-20201217125707394" />
   - 适用于存活对象较少的情况，只扫描一次，效率提高且没有碎片，空间浪费，但移动复制对象，需要调整对象引用
   - <img src="_media/database/JVM/拷贝2.png" alt="image-20201217125850168" />
3. Mark-Compat（标记压缩）
   - 把所有存活对象压缩到前面去，即使有未使用的空间，也先压进去，之后剩下的地方就全部清理空间又是连续的，还没有碎片。
   - <img src="_media/database/JVM/标记压缩1.png" alt="image-20201217130035579" />
   - 存在的问题，首先是通过GC Roots找到那些不可回收的，然后把不可回收的往前挪，就变成肯定要扫描两次还需要移动对象，第一遍扫描先找出有用的来，第二遍才能进行移动。移动的话如果是多线程你还要进行同步，所以效率是要低很多。好处在于不会产生碎片，方便对象分配不会产生内存减半。
   - <img src="_media/database/JVM/标记压缩.png" alt="image-20201217130620693" />
4. 总结：举例，如果这是一个房间， 在这个房间里面有两个小人，在这里不停的产生垃圾，不停的往外扔线团，或者再扔或者在剪断，与此同时，第一种算法叫Mark Sweep，标记为垃圾了就给它清理掉，但是别的空间都是不动的，它的效率还可以，但是容易产生碎片。第二种是把房间一分为二，你们只能在这半边随便产生垃圾，等这边玩不动了(垃圾太多了)。直接把这边有用的拷贝过来，剩下的清理就直接整个内存清理搞定，效率非常高。第三种叫标记压缩，压缩的意思就是把这些对象凑到起，把这些垃圾全部给它清走，接下来剩下这个空间还都是连续的，你想分配任何内容的时候往里分配就行了，这叫做标记压缩，效率相对慢多了。

### 4、堆内存的逻辑分区（不适合不分代的垃圾处理器）

- 分代算法和垃圾回收器是有关系的，分代是存在于ZGC之前的所有垃圾回收器都是分带算法，目前用到的垃圾回收器至少在逻辑上是分代的，其实除了G1之外的其他垃圾回收器不仅在逻辑上，在物理上也是分代的。
- 部分垃圾回收器使用的模型
  - 除了Epsilon、ZGC、Shenandoah之外的GC都是使用逻辑分代模型
  - G1是逻辑分代，物理不分代
  - 除此之外不仅逻银分代， 而且物理分代
- 逻辑分代的概念是给内存做一些概念上的分区，物理分代是真正的物理内存。
- 把内存分成如下几个内容
  - 新生代（young），刚new出来的叫新生代，新生代存活对象特别少，死去对象特别多，所使用的算法是Copying。新生代中又分为如下：
    - eden（伊甸区）：默认比例是8，是我们刚刚new出来对象存在的区域；
    - survivor（S0）：默认比例是1，是回收一次之后到这个区域，这里由于装的对选哪个不同，采取的算法也不同
    - survivor（S1）：默认比例是1
  - 老年代（old），是垃圾回收很多次都没有回收掉的内容，老年代中存活对象特别多，适用于Mark Compact或Mark Sweep 算法
  - <img src="_media/database/JVM/堆内存逻辑分区.png" alt="image-20201217152850032" />
  - 一个对象从出生到消亡的过程：一个对象产生之后首先进行栈上分配，栈上如果分配不下就会进入伊甸区，伊甸区经过一次垃圾回收之后进入survivor区，survivor区在经过一 次垃圾回收之后又进入另外一个survivor,，与此同时伊甸区的某些对象也跟着进入另外一个sunvivor，什么时候年龄够了会进入到old区，这是整个的对象的一个逻辑上的移动过程。 见下图
  - s0-s1之间的复制年龄超过限制时，进入old区，通过参数: -XX:MaxTenuringThreshold配置。
  - ![](_media/database/JVM/分配过程.png"/>

### 5、GC的概念

<img src="_media/database/JVM/GC的一些概念.png" alt="image-20201217153836980" />

- -Xms-Xmx
  - Xmn
    - eden
    - s0
    - s1
- -Xms -Xmx ：
  - X是非标参数，
  - m是memory，
  - s是最小值，
  - x是最大值，
  - n是new
- MinorGC / YGC：年轻代空间耗尽时触发
- MajorGC / FulIGC：在老年代无法继续分配空间时触发，新生代老年代同时进行回收。

### 6、栈上分配问题

- 栈上分配，什么样的对象分配在栈上：
  - 线程私有小对象：小对象，线程是私有的
  - 无逃逸：就在某段代码中使用，除了这段代码就没有人认识它了
  - 支持标量替换：意思是用普通的属性、类型代替对象就叫标量替换
  - 无需调整
- 栈上分配要比堆上分配要快，在栈上分配不下了，它会优先进行本地分配
- 线程本地分配TLAB (Thread Local Allocation Buffer)：
  - 问题：很多个线程都会在Eden区分配对象，会对Eden去进行一定的空间征用，那么当很多线程都在征用时，谁抢到算谁的。多线程的同步，效率就会降低，所以设计了这么一个机制叫做TLAB
  - 解决问题TLAB
    - 占用eden区, 默认占用1%，在Eden区取用1%的空间， 这块空间叫做这个线程独有。分配对象的时候首先往我线程独有的这块空间里进行分配。
    - 多线程的时候不用竞争eden就可以申请空间，提高效率
    - 小对象
    - 无需调整
    - 老年代
      - 大对象
      - eden
  - 分析以下程序：User类中有一个int类型的id和String类型的name，有一个方法叫alloc，当你调用一下alloc，就new-一个对象出来。这个new User出来之后没有任何引用指向它，所以没逃逸。那如果有外部引用，在外面写一个User u, 方法里面u= new User这个就是有逃逸，因为这个对象被外面的给引用了。
    - <img src="_media/database/JVM/程序分析.png" alt="image-20201217155049098" />
    - 验证程序的参数：
      - ‘ - ’ 代表去掉属性
      - -XX:-DoEscapeAnalysis 去掉逃逸分析
      - -XX:-EliminateAllocations 去掉标用替换
      - -XX:-UseTLAB 去掉TLAB

### 7、对象何时进入老年代？

- 超过 -XX:MaxTenuringThreshold指定次数（YGC），来指定对象何时进入老年代，如果你不指定的话默认如下：
  - Parallel Scavenge ->15
  - CMS -> 6
  - G1 -> 15
- GC的age是4位， 4位的范围是0~15。这个值是不可以调大的。CMS是6， 除了它之外别的都是15
- 第二种叫动态年龄（了解即可）：s1 -> s2 超过50%。两个s之间相互拷贝，只要超过50%的时候就会把年龄最大的直接就放入到old区，也就是不一定非得到15岁，不一定非得到6岁
  - 如图， s1里面在加上伊甸区里面，整个对象一下子拷贝到s2里，经过一次垃圾回收(YGC)之后，s2中对象已经超过s2的一半了， 将这里面年龄最大的些对象直接进入old区，这叫做动态年龄的一个判断。
  - ![image-20201217155607267](_media/database/JVM/动态年龄.png"/>

### 8、以上5 - 7 节总结

- 对象分配过程：start会new一个新的对象，首先在栈上分配，如果能分配就会分配到栈上，弹出来之后结束。如果分配不下，就会判断大不大(用一个参数来指定的)，如果特别大，直接进入old区(FGC才会结束)，如果不够大，会进去TLAB，到伊甸区(E) ，进行GC清除，如果清完了结束，如果没有清完进s1, s1在进行GC的清除，如果年龄够了进入old区，如果不够进s2。

- ![](_media/database/JVM/分配流程.png"/>

- ```java
  java // 回车 查看Java参数
```

- ‘ - ’ ：横杠开头都是标志参数

- -X：是非标参数

- -XX：是不稳定参数

- -Xms：起始的Java堆大小

- -Xmx：最大的java堆大小

- ```java
  java -XX:PrintFlagsFinal -version //打印所有参数代码，会打印七八百条
  java -XX:PrintFlagsFinal -verston | grep Tenure //精确查询
  ```

- JVM 的误区：虚拟机并不是永远地要求对象的年龄必须达到了 MaxTenuringThreshold 才能晋升老年代，如果在
  Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到MaxTenuringThreshold中要求的年龄这里讲的是动态年龄的判定。对于动态的判定的条件就是相同年龄所有对象大小的总和大于Survivor空间的一半，然后算出的年龄要和MarTeruringThreshold的值进行比较，以此保证MaxTenuringThreshold设置太大 (默认15)，导致对象无法晋开。

  - 间题提出场景：如果说非得相同年龄所有对象大小总和大于Survivor空间的一半才能晋升，分析以下场景：
  - MaxTenuringThreshold为15；年龄1的对象占用了33%；年龄2的对象占用33%；年龄3的对象占用34%。
  - 开始推论：按照晋升的标准。首先年龄不满足MaxTenuringThreshold，会晋升，每个年龄的对象都不满足50%。不会晋升。
  - 得到假设结论: Survivor都占用了 100%了，但是对象就不晋升。导致老年代明明有空间，但是对象就停留在年轻代。但这个结论似乎与JVM的表现不符合，只要老年代有空间，最后还会晋升的。
  - <img src="_media/basis/JVM/晋升年龄代码.png"/>
  - 以上是晋升年龄计算的代码，代码中有TargetSurvivorRatio的值。-XX:TargetSurvivorRatio目标存活率，默认为50%
  - 通过这个比率来计算一个期望值， desired_survivor_size ，然后用一个toalt数器， 累加每个年龄段对象大小的总和。当total大于desired_survivor_size停止，然后用当前age和MaxTenuringThreshold对比找出最小值作为结果。
  - 总体表征就是，年龄从小到大进行累加，当加入某个年龄段后，累加和超过survivor区域TargetSurvivorRatio的时候，就从这个年龄段往上的年龄的对象进行晋升。
  - 再次推演：还是上面的场景。年龄1的占用了33%， 年龄2的占用了33%，累加和超过默认的TargetSurvivorRatio (50%)，年龄2和年龄3的对象都要晋升。
  - 小结：动态对象年龄判断，主要是被TargetSurvivorRatio这个参数来控制。而且算的是年龄从小到大的累加和，而不是某个年龄段对象的大小，先记住这个参数TargetSurvivorRatio 虽然你以后基本不会调整他。
  - 分配担保YGC期间survivor区空间不够了，通过空间担保直接进入老年代。

### 9、常见的垃圾回收器

<img src="_media/basis/JVM/常用的垃圾回收器.png"/>

以上是常见的垃圾回收器。前面这几个加上G1在逻辑上都是分代的。ZGC和Shenandoah它们在逻辑上连代都不分。Epsilon是DEBUG用的。

JDK诞生之后第一个垃圾回收器就是Serial和Serial old.。为了提高效率，诞生了PS，为了配合CMS.，诞生了PN。CMS是1.4版本后期引入的，CMS是里程碑式的GC，它开启了并发回收的过程但是CMS毛病较多，因此目前没有任何一个JDK版本默认是CMS。Serial指的是单线程；Parallel Scavenge指的是多线程。

常见的垃圾回收器组合最常用的是有三种(Serial+Serial Old)、(Parallel Scavenge+Parallel Old)、(ParNew+CMS) 上图中，但凡是能连接在一起的都可以组合。 前面几种不仅都是在逻辑上分年轻代和老年代，在物理上也是分年轻代和老年代的。G1只是在逻辑上分年轻代老年代，在物理上他就分成一块块的。

1. Serial（几十兆）

   - 当工作的时候所有工作线程全部停止，断开的线程则是垃圾，如果突然加入Serial则停止，进行垃圾清理。
   - safe point = 线程停止（要找到一个安全点上，线程停止），因为停顿时间较长所有Serial现在用的较少
   - <img src=" _media/basis/JVM/Serial.png" alt="image-20201217160957504" />

2. Serial Old

   - 这个用在老年代，他用的是Mark Sweep的算法，用的也是单线程，
   - <img src=" _media/basis/JVM/Serial Old.png" alt="image-20201217161220737" />

3. Parallel Scavenge（上百兆 - 几个G）

   - 如果JVM没有做任何调优的话，默认的就是Parallel Scavenge和Parallel Old简称PS+PO。
   - Parallel Scavenge 和Serial区别：Parallel Scavenge是多线程清理垃圾。
   - 举例：你和你媳妇儿正在这里剪线团扔线团制造垃圾，然后你爸妈进来了一人给你们一个大耳巴子，让你们一遍站着去，然后开始他们俩人一块儿清理垃圾。清理完后你们继续。所以是多线程清理垃圾的意思。其它的可以用这两个为基础扩展开来，读的时候也就容易理解了。后面首先以这两种为主PS+PO
   - 图

4. Parallel Old

   - a compacting collector that uses multiple GC threads.
   - 整理算法
   - <img src=" _media/basis/JVM/Parallel Old.png" alt="image-20201217161453866" />

5. ParNew

   - 其实就是Parallel New的意思，和Parallel Scavenge没什么区别，就是它的新版本做了一些增强以便能让它和CMS配合使用， CMS在某一特定阶段的时候ParNew会同时运行。 所以这个才是第三个诞生解决的问题：我工作的时候，其余线程不能工作，必须等GC回收器结束才可以。

   - <img src=" _media/basis/JVM/ParNew.png" alt="image-20201217161611311" />
   - ParNew 和Parallel Scavenge的区别
     - PN响应时间优先，配合CMS
     - PS吞吐量优先，两个要配合使用的。
     - ▪[https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html#GUID-3D0BB91E-9BFF-4EBB-B523-14493A860E73](https://docs.oracle.com/en/java/javase/13/gctuning/ergonomics.html)

6. CMS（20G）

   - 理解：在一个房间有很多人在清理垃圾，现在服务器越来越大，无论多少人来清理都需要很长时间，初始标记根上对象，并发标记中80%时间都在并发，因为这里最浪费时间，一边产生垃圾，一边标记垃圾，这个过程很难完成，就有了重新标记，最后并发清理

   - CMS非常重要，因为它诞生了一个重程建式的东西，原来所有的垃圾回收就是，垃圾回收器来了，其他所有工作的线程都得停止，等着我回收，结束后才能继续工作，CMS的诞生就消除了这个问题，但CMS本身问题非常多，以至于没有任何jdk版本默认都不是CMS

   - CMS叫做：Concurrent Mark Sweep （Concurrent并发），意思就是工作线程制造垃圾的同事，CMS可以同时回收垃圾

   - 从线程的角度理解：当不管用几个钱程进行垃极回收时，这个过程都太长了。在内存比较小的情况下，没有问题，速度很快。但是现在的服务器内存越来越大，大到什么程度，原来是一个房间，现在可以看成一个天安门广场。作为一个这么大的内存无论你多少个线程来清理一遍也得需要特别长的时间。以前大概有10G内存的时候它用PS+PO停顿时间清理一次， 大概需要11秒钟。有人用CMS， 这个最后也会产生碎片，之后产生FGC, FGC默认的STW最长的到10几个小时。

   - <img src=" _media/basis/JVM/CMS.png" alt="image-20201217162021534" />

   - CMS的四个阶段

     - CMS Initial Mark（初始初始标记阶段）
       - 直接找到最根上的对象，其他的对象不标记，直接标记最根上的
       - 初始标记STW（停顿）开始的标记
     - CMS Concurrent Mark（并发标记阶段）
       - 据统计80%的GC的时间是浪费在这里，因此它把这块最浪费时间的和我们的应用程序同时运行，对客户来说感觉可能是慢了一些， 但至少你还有反应。就是一边产生垃圾， 一边跟着标记。
       - 并发标记和应用程序同时运行
     - CMS Remark（重新标记阶段）
       - 因为上一个过程很难完成，所以最后又有一个CMS remark(新标记)，这又是个STW， 在并发标记过程中产生的那些新的垃圾，在重新标记里头给它标记一下，这个时候需要停顿， 时间不长。
       - 重新标记又是个STW,在并发标记中产生的新垃圾在重新标记中标记
     - CMS Concurrent Sweep（并发清理阶段）
       - 并发清理也有它的问题，并发清理过程也会产生新的垃圾啊，这个时候的垃圾叫做浮动垃圾，浮动垃圾就得等着下一-次CMS再 次运行的过程把它给清理掉。

   - 什么条件赖发CMS呢?

     - 老年代分配不下了，处理不下会触发CMS，初始标记是单线程重新标记是多线程。

   - CMS的缺点：CMS出现问题时，会调用Serial Old老年代出来使用单线程进行标记压缩

   - CMS的两大问题

     - Memory Fragmentation内存的碎片，是比较严重的问题。如果你的内存超级大。CMS设计出来就只能对付几百兆内存，到上G内存（32G就会出问题）。现在有很多人拿它来应付很大的内存就会出问题。因为一旦老年代产生了很多碎片的时候，然后从年轻代过来的这些对象已经找不到空间了，这叫PromotionFailed找不到空间了。这时候它把Serial Old老奶奶请出来，让它用一个线程在这里面做标记压缩。

       - ```java
         -XX:+UseCMSCompactAtFullCollection
         -XX:CMSFulIGCsBeforeCompaction  // 默认为0，指的是经过多少次FGC才进行压缩
         ```

     - Floating Garbage浮动的垃圾，当出现Concurrent Mode Failure和PromotionFailed时说明碎片较多，里面内存分配不下，会调用Serial Old，并不是说当你使用CMS之后这个东西没有办法避免，可以降低触发CMS的阈值。

       - <img src="_media/basis/JVM/问题及解决.png"/>

   - FGC的概念：整体内存需要回收叫做FGC，内存不够会触发，默认会PS + PO 执行，FGC清理old区，降低阈值的话就设置上图最后一条参数

7. G1（上百G）：后面会做详细分析

8. ZGC（4T - 16T）：JDK13中的垃圾回收器，此处不做重点

### 10、CMS Concurrent阶段的算法

- 怎么样才能进行并发标记？
  - CMS用的是三色标记外加Incremental Update算法。而G1用的是三色算法加SATB算法，主要配合它的Rset来进行。 ZGC用的是颜色指针。他们之间就是数量级的一个提升
  - <img src=" _media/basis/JVM/三色标记.png" alt="image-20201217163559860" />
  - 三色扫描算法 —— 白灰黑
  - 在并发按标记时，引用可能产生变化，白色对象有可能被错误回收
  - 解决方案
    - SATB
      - snapshot at the beginning
      - 在起始的时候做一个快照
      - 当B -> D消失时，要把这个引用推到GC的堆栈，保证D还能被GC扫描
    - Incremental Update
      - 当一个白色对象被一个黑色对像引用
      - 将黑色对象重标记为灰色，让collector重新扫描

### 11、常见垃圾回收器组合参数设定（1.8）

垃圾回收有一些指定的命令，这些命令，jdk给的特别不直观。这做了一个总结。 这个总结是在oracle的网站上找到有人写的一个博客， 可以作为参考文档。

- -XX:+UseSerialGC=serial New(DefNew)+SerialOld ，当你设定这个参数之后就相当于年轻代的serial加上老年代这两种的组合，可以分别指定，也可以用个参数来指定它。小型程序，默认情况下不会使这种选项，HotSpot会根据计算及配和JDK版本自动选择收集器
- -XX:+UseParNewGC=ParNew+SerialOld, 这个组合已经很少用(在某些版本中已经废弃)
  - https://stackoverflow.com/questions/34962257/wby-remove-suport-for-parnewserialold-anddefnewcms-in-the-future
- -XX:UseConc(urrent)MarkSweepGC= ParNew+CMS+Serlal Old
- -XX:UseParllGC(默认)=Parallel Scavenge+Parallel Old(1.8默认) [PS+SerialOld]
- -XX:UseParallOldGC=Parallel Scavenge+Parallel Old
- -XX:UseG1GC=G1
- Linux中没找到默认GC的查看方法而windows中会打印UserParallelGC
  - java+XX:+PrintCommandLineFlags -version
  - 通过GC的日志来分辨
- Linux下1.8版本默认的垃圾回收器到底是什么？
  - 1.8.0 181默认(看不出来)Copy MarkCompact
  - 1.8.0 _222默认PS+PO
- MethodArea的发展：
  - 这个分代的模型之前讲过新生代和老年代，而永久代和元数据区，这个两个东西都叫MethodArea。方法区指的是：在1.8以前叫永久代Permanent Generation，然后1 .8叫做元数据区。
  - 怎么来理解呢? MethodArea存好多数据，除了存class的信息外，还存方法编译完的信息和代码编译完的信息以及字节码等等，这些东西全部都保留在永久代(MethodArea里面)。这里面有可能也会产生溢出，后面给大家举例子。在1.8之前永久代这个东西是无限大，首先你的物理内存这是不行的。它是有大小的，你必须指定大小，而且指定完了不能改。不像别的还有弹性空间，堆的话你可以设置一个最小和最大，不够使了弹来弹去。所以1.7以前发生了内存溢出是解决不掉的，就是永久代溢出，你就得把永久代设的特别大。由于永久代里面装的是各种各样的class的信息。所以在1.8之后它改了一个名字MethodArea叫做元空间，这个空间是没有大小限制的，可以不设置，无上限(受限于物理内存) 。 1.7和1.8还有个重要的区别字符串常t，原来咱们讲字符串常量是在MethodArea里面，但是在1.8之后就放到了堆里面。字符串常量这件事也取决于不同的虚拟机的一个发展。

## 七、PS+PO调优实战

### 1、JVM调优第一步，了解JVM常用命令参数

- JVM的命令行参数参考: https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

- HotSpot参数分类

  - 标准：- 开头所有的HotSpot都支持
  - 非标准：-X开头，特定版本HotSpot支持特定命令
  - 不稳定:：-XX开头，下个版本可能取消
  - java -version
    - <img src=" _media/basis/JVM/java-version.png" alt="image-20201218105551418" />
  - java -X
    - <img src=" _media/basis/JVM/java-x.png" alt="image-20201218105638887" />

- 见以下小程序：它有一个List是个LinkedList，是一个链表，后面一个循环byte[] b = new byte[1024*1024]，这是1M大小，list add开始循环进入内存，内存迟早会被占完。这个小程序运行起来存一定会产生相应的溢出， 以下根据这个小程序来看JVM命令

  - ```java
    import java.util.List;
    import java.util.LinkedList;
    public class HelloGC {
      public static void main(String[] args) {
        System.out.println("HelloGC!");
        List list = new LinkedList();
        for(;;) {
          byte[] b = new byte[1024*1024];
          list.add(b);
        }
      }
    }
    ```

  - 内存有两种方式对虚拟机产生影响

    - 内存泄漏（memory leak）：有一块内存无人占用，也无法回收就是内存泄漏，只要内存足够大，不会产生内存溢出
    - 内存溢出（out of memory）：不断产生新对象，致使内存空间不足

  - 怎么指定一个应用程序的参数？

    - <img src="_media/basis/JVM/idea指定参数.png"/>

- 命令1：java -XX:+PrintCommandLineFlags HelloGC ， 以下是命令执行过程

  - InitialHeepSize —— 起始的堆大小，根据你的内存算出来
  - MaxHeepSize —— 最大堆大小
  - UseCompressedClassPointers —— 默认开启对象头指针压缩
  - UseComperssedOops —— 默认开启普通指针压缩
  - <img src="_media/basis/JVM/命令1.png"/>

- 命令2：java -Xmn10M -Xms40M -Xmx60M -XX:PrintCommandLineFlags -XX:+PrintGC HelloGC PrintGCDetails PrintGCTimeStamps PrintGCCauses ， 以下是命令执行过程

  - -Xmn10M —— 新生代大小
  - -Xms40M —— 最小堆大小
  - -Xmx60M —— 最大堆大小，默认开始最小是40，最大是60，最好不要让他产生弹性扩大和压缩，这样会浪费系统的计算资源，对系统性能造成影响，一般情况设定一样的大小
  - -XX:+PrintGC HelloGC ——  打印GC回收信息
  - PrintGCDetails PrintGCTimeStamps —— 打印详细GC信息
  - PrintGCCauses —— 打印产生GC的原因
  - <img src="_media/basis/JVM/命令2.png"/>
  - GC（产生原因）Allocation Failure（分配失败）--> 整个堆经过一次GC从多大到多大 --> 花了多少时间。
    - 由于每次的GC会后不了新产生的字节，因为它都装进一个list里面了，会直接丢到old区，old区越来越大会产生FULL GC ... 最后FGC也不行了，就会内存溢出

- 命令3：java -XX:+UseConcMarkSweepGC -XX+PrintCommandLineFlags HelleGC ， 以下是命令执行过程

  - 启用CMS垃圾回收器
  - UseConcMarkSweepGC命令会很频繁，CMS和PS+PO基本上差不多，唯一不同的是标记的详细阶段过程
  - <img src="_media/basis/JVM/命令3.png"/>

- 命令4：java -XX:+PrintFlagsInitial 默认参数值

- 命令5：java -XX:+PrintFlagsFinal 最终参数值

- 命令6：java -XX:+PrintFlagsFinal | grep xxx 找到对应的参数

- 命令7：java -XX:+PrintFlagsFinal -version |grep GC

### 2、PS GC日志详解

<img src="_media/basis/JVM/PSPo日志格式.png"/>

- 使用命令打印日志的详细信息PrintGC后面加Detalis

- 上图[ ]中详解写：中间的括号是GC产生的原因，当你看到GC时，指的是YGC，如果你想看到FulIGC，就是全称的显示，这时候他才是FllGC。

- DefNew：产生的年代，这个是年轻代，4544-259k是回收之前年轻代的大小是4544k，回收之后是259K。6144指的是真个年轻代的总额大小，后面的这个数是整个GC所产生的时间。

- 4544-4356：指的是整个堆的大小，19840K是整个堆的空间， 后面是整体的时间。

- 产生GC的类型 - 产生的原因 - [年轻代 - 回收前大小 - 回收后大小 - 整个堆大小 - 整体时间

  - <img src="_media/basis/JVM/日志命令.png"/>

- Times：

  - 当执行time ls命令时，下图表示总共占用多长时间，用户time是多长时间，内核time是多长时间
  - <img src="_media/basis/JVM/timels.png"/>

- heap dump部分，一旦产生内存溢出，会把整个堆都heap dump出来

  - <img src="_media/basis/JVM/heapdump1.png"/>

  - ```java
    eden space 5632K, 94% used [0x00000000ff980000,0x00000000ffeb3e28,0x00000000fff00000)
                              //  后面的内存地址指的是，起始地址，使用空间结束地址，整体空间结束地址
                              //  内存的起始地址到整体空间技术地址是5632k，占用了94%
    ```

  - Metaspace used（真正使用多少），class space：专门给class信息做存储的内容

    - capacity（Metaspace容量是多少），committed（虚拟占用），reserved(内存保留)。
    - 整个内存保留出来的空间叫reserved，因为整个内存其中分成一块一块的， 连续占用了三块内容叫committed，在这三块里面其实你只使用了其中一部分作为它的整体容量，这个容量是可以弹性扩大的capacity，然后在这个容量里面你只占用了其中一部分。

  - the space和total是一样的： 97%给used这个意思。total 年轻代= eden + 1个survivor。

- 年轻代大小加起来不相等?

  - 原因：除了你new的对象外，整个年轻代外还有其他信息。观察日志主要是看它的一个变化，比如说前面占了百分之一， 后面突然占了百分之百。

### 3、GC调优前两个重要基础概念

- 基础概念：
  1. 吞吐量：用户代码时间 / (用户代码执行时间+垃圾回收时间)
  2. 响应时间：STW越短，响应时间越好

- 所谓调优，首先确定，追求的是什么？ 吞吐量优先，还是响应时间优先？ 还是在满足一定响应时间的情况下，要求达到多大的吞吐量..  每一个项目具体的要求都不一样， 达到性能要求就可以，达不到好好调，实在是调不了加内存、加CPU。
- 例如：科学计算和数据挖掘要保证吞吐量(throughput) ；吞吐量优先的一般很简单：先选定垃圾回收器(PS+PO) ；如果说响应时间：比如说网站、带界面的程序(GUI) 、对外提供的相应的API (1.8，垃圾回收选G1)，这类的服务响应时间要优先。

### 4、什么是调优？

- 调优是非常粗糙的一个概念， 简单说就是把整个系统进行优化，整个系统优化能干的事儿多了，都可以叫调优。GC的调优指的是什么呢?把它分成三大类:
  1. 根据需求进行JVM规划和预调优
  2.  优化运行VM运行环境(慢，卡顿)
  3. 解决JVM运行过程中出现的各种问题(OOM)
- OOM 就只是调优其中的一部分，问题出现了就得解决。并不是全部，作为调优来讲，前面两个也很重要，第一种是JVM的规划和预调优，第=种是优化VM运行环境。这两个在我们真正的调优，至少有一半以上是做reboot的，在游戏服务器中，reboot会经常断一下，出现问题首先重启-下，调优从规划开始，根据自己本身的业务开始调整。

### 5、调优，从规划开始

（100w并发，似transation)说每秒钟进行百万次的页面浏览这是能达到的。一般我们来聊百万并发的时候指的是百万个tensation业务逻辑的处理。比如聊电商网站指的是100W个订单一秒钟下来。据说淘宝一年最高的是54w并发，据说比淘宝更牛的是12306，号称并发上百万。

- 调优，从业务场景开始，没有业务场景的调优都是耍流氓

- 无监控（指的是压力测试，能看到结果），不调优

- 步骤：

  1. 熟悉业务场景（选定垃圾回收器，没有最好的垃圾回收器，只有最合适的垃圾回收器）

     1. 响应时间、停顿时间[CMS G1 2GC]（需要给用户作响应）
     2. 吞吐量=用户时间（用户时间+ GC时间） [PS]

  2. 选择回收器组合

  3. 计算内存需求：内存需求弹性需求大，内存小，回收快也能承受，所以内存大小没有一定的规定。

  4. 选定CPU（越高越好，按照预算来）

  5. 设定年代大小、升级年龄

  6. 设定日志参数

     - ```java
       -Xloggc:/opt/xxx/logs/xxx-xxx-gc-%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause
       //在生产环境中日志文件,后面日志名字,按照系统时间产生,循环产生,日志个数5个,每个大小20m,这样的好处在于整体大小100M。我能控制整体文件大小。
       ```

  7. 注：一般记录日志的时候，记录日志文件如果只有一个日志文件，肯定不行，有些服务器可能一天产生的日志文件就上T，连着30天就是30T，产生量大，如果查找问题，查找不到问题所在。其实这个工作是运维干的活儿。

### 6、预调优案例

1. 案例一：垂直电商，最高每日百万订单，处理订单系统需要什么样的服务器配置?
   - 这个问题比较业余，因为很多不同的服务器配置都能支撑(1.5G可以、16G也可以)，1小时360000集中时间段，100个订单/秒，（找一小时内的高峰期，1000订单秒）例：每天一百万订单，每个小时不会产生很高的并发量，我们寻找高峰时间，做一个假设100w有72w订单在高峰期（5点-7点）产生，比如一小时平均36w订单，所以我们内存选择大小是按照巅峰时间选择的，也可能1000个订单/s。
   - 好多时候就是经验值拿过来做压力测试，实在不行拿过来加CPU加内存。
   - 非要计算：一个订单产生需要多少内存？512K*1000 ，500M内存一秒钟丢进来250订单，我只要不到一秒时间处理完成，这里垃圾回收就ok了，所以这一块难于估计
   - 专业一点儿问法：要求响应时间100ms，需要做压测
2. 案例二：12306遭遇春节大规模抢票应该如何支撑？(这个类似架构设计,跟预调优不是很有关系)
   - 订单信息每天固定,可以丢到缓存中，不同的业务逻辑有不同的业务设计，
   - 12306应该是中国并发量最大的秒杀网站：号称并发量100W最高
   - CDN > LVS > NGINX >业务系统>每台机器1W并发（10K问题，单机10K问题已经被解决，redis是其中的关键） 100台机器装100个redis就搞定了
   - 考虑问题：普通电商订单 -> 下单 -> 订单系统（I0）减库存 -> 等待用户付款，这个事务如果非要同步方式完成，这个TPS撑不了多少时间
   - 12306的一种可能的模型：下单 -> 减库存和订单(redis kafka)同时异步进行 -> 等付款。异步是当你下完订单之后，它一个线程去减库存，另外一个线程直接把你下单的信息扔到kafka或者redis里直接返回ok，你下单成功等待你付款，什么时候你付款完了后面那些个订单处理线程就会去里面拿数据，这个处理完了就会持久化到Hbase或者是mysql
   - 如：车票信息,存在库存中,如果需要哪个去库存取；减库存最后还会把压力压到一台服务器，可以做分布式本地库存+单独服务器做库存均衡
   - 大流量的处理方法；分而治之，每台机器放一定数量,每个机器只减自己机器上的内存
   - 假设有100w机器,每台机器1w并发，在计算机界叫做10k问题，单机10k问题，redis是其中关键，redis可以解决，更多的问题,需要具体问题具体分析,看具体的业务逻辑
   - 怎么得到一个事务会消耗多少内存？
     - 弄台机器，看能承受多少TPS？是不是达到目标？扩容或调优，让它达到
     - 用压测来确定
3. 理论补充
   - TPS：Transactions Per Second（每秒传输的事物处理个数），即服务器每秒处理的事务数。TPS包括一条消息入和一条消息出，加上一次用户数据库访问。（业务TPS = CAPS × 每个呼叫平均TPS）
     - TPS是软件测试结果的测量单位。一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数。
     - 一般的，评价系统性能均以每秒钟完成的技术交易的数量来衡量。系统整体处理能力取决于处理能力最低模块的TPS值。
   - QPS：每秒查询率
     - QPS是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准，在因特网上，作为域名系统服务器的机器的性能经常用每秒查询率来衡量。对应fetches/sec，即每秒的响应请求数，也即是最大吞吐能力。

### 7、优化环境

1. 有一个50万PV（访问量）的资料类网站（从磁盘提取文档到内存）原服务器32位，1.5G 的堆，用户反馈网站比较缓慢，因此公司决定升级，新的服务器为64位，16G 的堆内存，结果用户反馈卡顿十分严重，反而比以前效率更低了
   - 为什么原网站慢？
     - 很多用户浏览数据，很多数据load到内存，内存不足，频繁GC，STW长，响应时间变慢。
   - 为什么会更卡顿？
     - 内存越大，FGC时间越长。
   - 解决方案：
     -  PS 或 PN + CMS 或者 G1。PS+PO无论怎么设置，卡顿还是会出现。
   - java对象看产生的大小，回收完还会回收到老年代文档对象本身可以走代理服务器。
2. 系统CPU经常100%，如何调优？(面试高频) CPU100%那么一定有线程在占用系统资源
   1. 找出哪个进程cpu高（top）
   2. 该进程中的哪个线程cpu高（top -Hp）
   3. 导出该线程的堆栈(jstack)
   4. 查找哪个方法（栈帧）消耗时间（jstack）
   5. 工作线程占比高还是垃圾回收线程占比高
3. 系统内存飙高，如何查找问题？（面试高频)
   1. 堆栈比较多，导出堆内存（map）
   2. 分析(jhat、jvisualvm、mat、jprofiler ...)
4. 如何监控JVM？
   - jstat、jvisualvm、jprofiler、arthas top.像这类的工具我会给大家讲些典型的。

## 八、JVM调优实战

### 1、代码案例分析

- 承接上一课，我们继续看如何监控JVM，通过一个实例了来了解一些实际当中的操作，见下面的代码。

- ```java
  package com.jvm.test;
  
  import java.math.BigDecimal;
  import java.util.ArrayList;
  import java.util.Date;
  import java.util.List;
  import java.util.concurrent.ScheduledThreadPoolExecutor;
  import java.util.concurrent.ThreadPoolExecutor;
  import java.util.concurrent.TimeUnit;
  
  /**
   * 从数据库中读取信用数据，套用模型，并把结果进行记录和传输
   */
  
  public class T15_FullGC_Problem01 {
  
      private static class CardInfo {
          BigDecimal price = new BigDecimal(0.0);
          String name = "张三";
          int age = 5;
          Date birthdate = new Date();
  
          public void m() {}
      }
  
      private static ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(50,
              new ThreadPoolExecutor.DiscardOldestPolicy());
  
      public static void main(String[] args) throws Exception {
          executor.setMaximumPoolSize(50);
  
          for (;;){
              modelFit();
              Thread.sleep(100);
          }
      }
  
      private static void modelFit(){
          List<CardInfo> taskList = getAllCardInfo();
          taskList.forEach(info -> {
              // do something
              executor.scheduleWithFixedDelay(() -> {
                  //do sth with info
                  info.m();
  
              }, 2, 3, TimeUnit.SECONDS);
          });
      }
  
      private static List<CardInfo> getAllCardInfo(){
          List<CardInfo> taskList = new ArrayList<>();
  
          for (int i = 0; i < 100; i++) {
              CardInfo ci = new CardInfo();
              taskList.add(ci);
          }
  
          return taskList;
      }
  }
  ```

- 这个代码是模拟了一个情形，一个风控的场景，风控就是风险控制，一般是银行或者第三方公司在向一个人发放贷款的时候，但是你要控制风险。需要知道这个人以前的信用记录到底好不好，我需要进行风险控制。风险控制模型有很多，肯定需要从数据库里面调出相应的信息看看风险等级是多少，给出评级。

- 程序设计的很简单，其中拿了一个信用卡的例子，CardInfo信用卡类。把这个人信用卡的记录都调出来，之后做一些自己相应的业务处理方法来对它进行处理和计算。信用卡记录这个东西呢它会从数据库里面每次扔出100个卡来，然后用这100个信息来进行一些风险模型的计算，来看看我这个模型是不是符合modelFit。当然。这里面都是模拟的代码，那具体怎么做呢，这个应用程序里头呢有一个类叫CardInfo，有一个方法叫getAllcardInfo,每次都是拿100个出来。拿100个出来之后用一个线程池做计算，线程池用的是ScheduledThreadPoolExecutor（定时任务）。new 出来线程池，50个线程然后做出它的拒绝策略。接下来开始做业务处理，业务处理的时候会调用 modelFit()方法，调用完后睡个100毫秒来模拟业务的停顿。在modelFit里面做了一个什么事情呢，它调用getAllcardInfo(拿到这100个list，拿到之后在其中拿出每一个来调用schedulewithFixedDelay，隔一段时间调用一次这样的方法，最开始往后推2毫秒，以后每3秒钟执行一次。ok,这是整个程序的大体逻辑。当然这个程序肯定是有问题。

- 扩展：当你有一个云。有一堆机器放到你的云集群上面。其实机器特别多的话是很难管控的，会有专门的网管团队来监控这些机器。它会发现你的JVM的问题，像作为阿里非常专业专门管理JVM的这种网管，他对VM的理解会深刻的多。这个时候对于JVM的监控很多是有自己特定的网管软件，即便是很普通的那种中小型的企业来说也会有自己网管软件如Ansible专业软件，至少他会监控机器内存是不是饱了，CPU是不是标高了。总而言之，每台机器如果它真的有什么问题至少在CPU层级和内存层级会进行报警。定型机器问题，然后去解决问题。

### 2、问题定位

1. ```java
   java -Xms200M -Xmx200M -XX:+PrintGC com.jvm.gc.T15_FullGC_Problemo1
       // 在Linx上输入上代码,回车启动。
   ```

2. 一般是运维团队首先受到报警信息（CPU Memory)

3. top命令观察到问题：内存不断增长 CPU占用率居高不下

   - 收到报警信息，拿top命令去查，top后你会看到它的PID（1364），它占比比较高。然后top还有一个常见的用法：top -Hp 1364。这个时候它会把这个进程里面的所有的线程全都列出来，这些都是Java这个进程里面内部的一些线程。如下图，你会看到每个线程的占比都差不多，偶尔有某一个线程比较高。因为这个小例子，某些线程占得比较高的时候，这个小例子最终会是垃圾回收的线程占得比较高，因为它垃圾回收不过来了，所以它来回不断的使劲儿回收。每次都回收一点点。
   - <img src=" _media/basis/JVM/top命令.png" alt="image-20201219231338030" />
   - 实际这种例子里面非常有可能是你的业务逻辑线程，那一根业务逻辑线程占比非常高，就需要另外的命令jstack

4. top -Hp 观察进程中的线程，哪个线程CPU和内存占比高

   - <img src=" _media/basis/JVM/tophp命令.png" alt="image-20201219232122187" />

5. jps命令定位具体java的ps进程，jps只会列java的进程。

6. jstack命令

   - 实际中飙高的可能是业务逻辑线程，这时就要用jstack，比如我接上1374这个线程号，1364是我们java的整个进程。我们要定位某一个线程cpu的占比要比其他cpu高很多，那我要定位这个线程里面到底是什么样的问题的时候，你需要把这个线程号给记下来（1374）。
   - jstack用到里面的这个线程号是16进制的，所以你要把1374的十进制转换成16进制才行。jstack会有它自己的一些功能，就是把这些个1591这个进程里面下面这些线程全部都给你列出来。在每个线程里面都会给你打印出来线程的一些状态这是比较重要的。这个线程也有自己线程号码。这里面比较重要的是线程的状态，是你这个线程到底有没有阻塞，如果你这线程长时间的wait、长时间的block说明这个线程还有问题。jstack 1374
   - <img src="_media/basis/JVM/jstack.png"/>
   - jstack定位线程状况，重点关注：WAITING BLOCKED。
   - 比如说：waiting on <Dx0000000088ca3310>（a java.lang.Object)正在等待这把锁的释放。
   - <img src="_media/basis/JVM/jpsjstack.png"/>
   - 假如有一个进程中100个线程，很多线程都在waiting on 某一把锁，然后线程不该阻塞的被阻塞了，该结束的没结束掉，怎么搜索？一定要找到是哪个线程持有这把锁，怎么找？搜索jstack dump的信息，找<0X..>，看哪个线程持有这把锁，一般的这个线程状态是RUNNABLE。表示这个线程正在运行但是我一直持有这把锁不释放，那就会导致整个线程的死锁。
   - 定义锁，如果看到很多信息，都在同一个id号时，说明都在等一个锁，这个时候定义这个锁就可以了
   - 可以写一个死锁程序，用jstack观察 ；写一个程序，一个线程持有锁不释放，其他线程等待

7. jmap -histo 4655 | head -20，查找有多少对象产生，列出前20个

   - <img src="_media/basis/JVM/jmap.png"/>

8. jmap -dump:format=b,file=xxx pid :手动导出堆转储文件

   - 我们的系统执行一个map命令时间确实不长，如果线上系统，内存特别大，jmap执行期间会对进程产生很大影响，甚至卡顿（电商一定不适合）。如下说法：

     1：设定了参数HeapDump，OOM的时候会自动产生堆转储文件

     - java -Xms20M -Xmx20M -XX:+UseParallelGC -XX:+HeapDumpOnOutOfMemoryError com.jvm.test.T15_FullGC_Problem01

     2：很多服务器备份（高可用），停掉这台服务器对其他服务器不影响

     3：在线定位，arthas

9. 使用MAT / jihat进行dump文件分析 https://www.cnblogs.com/baihuitestsoftware/articles/6406271.html

   - jhat -J-mx512M xxx.dump http://192.168.60.31:7000 拉到最后：找到对应连接，可以使用OQL查找特定问题对象

10. 为什么阿里规范里规定，线程的名称（尤其是线程池）都要写有意义的名称。怎么样自定义线程池里的线程名称？（自定义ThreadFactory）

11. 找到代码的问题

### 3、OOM问题

OOM问题，关于这个问题有很多命令可以用，每个命令它的重点都不太一样，比如:

- jinfo pid 这个命令就是把这个进程的一些详细信息给你列出来看看，关系不大。
- jstat -gc 就是把gc的信息打印出来。动态观察gc情况/阅读GC日志发现频繁GC / arthas观察 / jconsole / jvisualVM / Jprofiler（最好用）， jstat -gc 4655 500：每隔500毫秒打印GC的情况。gc的信息包括各种Eden区各种内存的大小还有gc了多少次。这个信息看起来不是很好看不直观，用的也不多。
- jconsole是 jdk自带的，比较直观。以下会演示。
- 里面存在的一些问题：你要远程去跟踪一个进程，你是从Windows上去跟Linux上的一个进程，是从本机去跟踪远程服务器上的一个进程，而作为inux的服务器来说，很少有人会装图形界面，你可以在线的跟踪，后面给你讲阿里的arthas 。 当然你要想远程的访问服务器，服务器肯定对你做一些操作。
- Java有一个标准的访问远程服务的这样一个协议叫JMX

### 4、jconsole远程连接

1. 程序启动加入参数

   ```shell
   java -Djava.rmi.server.hostname=192.168.60.143 -Dcom.sun.management.jmxremote -
   Dcom.sun.management.jmxremote.port=11111 -
   Dcom.sun.management.jmxremote.authenticate=false -
   Dcom.sun.management.jmxremote.ss1=false XXX
   ```

2. 如果遭遇Local host name unknown：XXX的错误，修改/etc/hosts文件，把XXX加进去

3. 关闭Linux防火墙（实战中应该打开对应端口）

4. Windows上打开jconsole远程连接 Linux的地址:端口

5. jconsole现在用的也少了，如果需要的话可以直接用jvisualvm，以下

### 5、jvisualvm远程连接

- https://www.cnblogs.com/liugh/p/7620336.html（简单做法）了解即可，因为它有限制，不如JMX支持的细，所以如果想远程监控，直接起JMX就行了
- 监视：新建一个远程连接，添加JMX连接，连接上你会看到一些相关信息，可以对它进行监视，总共装了多少类进来，这个线程包括多少个线程，这个线程总运行多长的时间，具体有哪些线程
- <img src="_media/basis/JVM/jvisualvm1.png"/>
- 下图，看到这步之后就知道怎么定位问题的所在了，刚刚我们观察到会有很多的类，很多内存被占用了。你只要用这种jvisualvm，是这种图形化看起来就方便多了。你点击了之后它会对远程的那台机器正在跑着的java进程内存，里面的信息给您导出来。你能看到我这个进程里面有那些个类，占了多少个字节另外有多少个数量。BigDecimal这个类占了好多，多少个实例。看过这些信息之后你还不能定位是由于那个类产生了OOM吗。是不是可以知道那个类占用的特别多。
- <img src="_media/basis/JVM/jvisualvm2.png"/>
- 面试官问你是怎么定位OOM问题的？ 如果你回答用图形界面（错误）。为什么这么说，作为一个服务器来讲它在哪儿不断的运行。刚你也看到了，当你开一个JMX服务的话，会影响原来的运行效率。
  - 已经上线的系统不用圈形界面用什么？（cmdline athas） 
  - 图形界面到底用在什么地方？测试！测试的时候进行监控。
- jprofiler (收费)这个是号称是图形化最好用的工具，没有给大家说是因为要收费。

### 7、阿里Arthas，用于检测JVM运行情况

- 首先使用jvisualvm远程连接，将程序启动

  - java -Xms20M -Xmx20M -XX:+PrintGC com.jvm.test.T15_FullGC_Problem01
  - <img src=" _media/basis/JVM/arthas.png" alt="image-20201220003532218" />

- 下载后解压，ls本地目录

  - <img src="_media/basis/JVM/arthas1.png"/>

- 到Arthas目录下，cd a*bin进入目录

  - <img src="_media/basis/JVM/arthas2.png"/>

- 启动arthas-boot.jar，as.sh也可以启动，需要设置环境变量 java -jar arthas-boot.jar

  - <img src="_media/basis/JVM/arthas3.png"/>

- 然后会看到一个进程号为1442，后面T15_FullGC_Problem01是我们运行的java程序，它的进程号在arthas 1，所有敲进程号1，它会把自己挂到这个进程上

  - <img src=" _media/basis/JVM/arthas4.png" alt="image-20201220003828522" />

- 之后就可以用arthas命令来观察这个内容，help可以查看到常用的命令，如果敲jvm命令，它会把详细的配置情况列出来

  - <img src="_media/basis/JVM/arthas5.png"/>

- 使用jvm命令（相当于jinfo）后显示的界面，最主要的就是GARBAGE-COLLECTORS，这个是能观察到的，用这个可以看到用的是哪种垃圾回收器，Copy是Eden区的；MarkSweepCompact是old区的

  - <img src=" _media/basis/JVM/garbagecollectors.png" alt="image-20201220004106778" />

-  MEMORY-MANGERS就是内存是怎么管理的、OPERATING-SYSTEM是哪个操作系统。想看有多少线程的命令是thread，想看某条线程具体的情况你就可以敲thread 1或者其他编号，

  - <img src=" _media/basis/JVM/memorymangers.png" alt="image-20201220004207394" />
  - 他有这个功能叫做dashboard（类似于top）他会用一个命令行让你来观察这个图形化界面
  - <img src=" _media/basis/JVM/dashboard.png" alt="image-20201220004425968" />

- 还有一个常用命令是heapdump导出堆内存情况，相当于 jmap -dump:format=b,file=xxx pid ，他有默认路径还可以自己设置路径。比如说我指定到root/20200104.hprof这里，文件名称你随便起，只要格式正确。这个运行对主进程也有很大的影响。所以我们能不导堆就尽量的不去导堆，能在线定位就在线定位。

  - <img src="_media/basis/JVM/heapdump.png"/>

- 然后我们来分析这个文件，可以用这个命令jhat 导出文件名称，来分析文件，还可以指定他多少内存，执行jhat的时候内存溢出，因为它这个文件太大了：（jhet -J-mx512M 文件名称）能让它利用的内存更大一些，当你指定一个小内存用这个小内存慢慢的导，这取决于你后面整个堆文件到底有多大，它有多少个G没有关系，你要指定它的大小就行了。把CPU停掉也没有关系，因为这个东西已经被离线导出来，就算关掉原来的，也可以观察。创建一个文件然后去检测，启动一个http server 端口7000

  - <img src="_media/basis/JVM/jhet.png"/>

- 然后我们就可以通过端口去访问它了，里面有它的paclage和other拉倒最底下去找instance点进去，它就相当于jmap那个界面，能够分析出来哪个类包含的对象最多，给你分析出来哪个类产生哪个对象

  - <img src=" _media/basis/JVM/jhet2.png" alt="image-20201220005012215" />

- 最强大的功能是：Execute Object Query Language(OQL) query。可以显示有哪些对象，对象里面有多少字节和引用，可以观察到那个对象产生了问题。下图，显示所有String对象

  - <img src=" _media/basis/JVM/oql.png" alt="image-20201220005221174" />

- 搜索点进去之后还能看到这个对象到底占用了多少个字节，他里面的data members到底是什么，References to this object：有多少个引用指向了这个Object，还可以通过语法进行过滤where...等等。

  - <img src=" _media/basis/JVM/jhet3" alt="image-20201220005300984" />

- 如果服务器没有安装arthas用什么分析？可以用上说的命令行方式进行分析，jmap也可以，但是arthas有jmap干不了的事儿。刚才讲heapdump加jhnat分析，但是有他们有干不了的事儿。分析dump文件刚刚看到了可以使用jhat，除此之外还可以使用MAT。多数人分析dump文件都是使用的MAT。除了MAT之外还有比它好用的一个叫jvisualvm。在你服务器产生了一个dump文件之后，你把它下载下来，之后完全可以通过jvisualivm来做分析。

- arthas还有更牛的地方，他能进行 jad反编译和热替换，比如说某个类可能会有问题，我们就可以将它在内存里反编译出来。我们自己有源文件反编译有什么用吗？ 

  - 动态代理生成类的问题定位 
  - 第三方的类（观察代码）
  - 版本问题（确定自己最新提交的版本是不是被使用）。
  - 如果没有这个的话是不是把整个项目停掉重新装上去，用这样的方式才能实现这样的问题。一旦我发现反编译出来的版本不是我想要的版本那就和下面的命令相配合，热替换。

- redefine 热替换目前有些限制条件：只能改方法实现（方法已经运行完成），不能改方法名，不能改属性 mQ>mm0)，即便是这样这个redefine也很有用，不然的话你改这一处别得地方都要改，不过后面版本可以实现。假如我在运行一个tomcat，上面有一个class确认他已经有问题了，但是他已经load内存了，如果我想替换这个class，普通情况下我只能把服务器停掉或者整个作用域停掉，我再重新编译正确的东西装进去，让Tomcat在重新装载一遍，不过在生产环境中这么弄是有问题的，在普通的小规模公司这样可以。但是在大规模公司淘宝京东这样的不能停的，这个流程非常复杂，消费的时间很多。对于阿里来说这样不可行，以下是演示。

- 演示两个小程序T.java和TT.java。写一个for循环，模拟Tomcat现在做一件事儿，做了阻塞 -> new T 这个对象的m()方法。这个小程序意思就是我有个TT里面有个方法，我的主程序不断去调用这个TT m这个方法，我们想在线把这个方法改掉，总而言之你可以想象一下我这个T它会调用某个类，某个类它里面出了bug，我们用arthas 到进程上反编译，然后直接改为输出2，然后redefine /rootTT.dass,然后回去调用这个程序他就输出2了，这就是redefine的含义。

  - ```java
    public class T{
        public static void main(String[] args) throws Exception{
            for(;;){
                SyStem.in.read();
                new TT().m();
            }
        }
    }
    ```

  - ```java
    public class TT{
        public void m(){
            System.out.println(1);
        }
    }
    ```

- dassloader里面有一个方法叫redefine，arthas里面还有其他的一些好用的功能，比如search class，watch是观察方法的执行结果，可以看文档。没有包含的功能是：jmap。所以现在arthas还不能完全替换JVM。

- 问题，为什么需要在线程排查？

  - 在生产线上我们经常会碰到一些不好排查的问题，例如线程安全问题，用最简单的threaddump或者heapdump，不好排查到问题原因。为了排查这些原因，有时候我们会临时加一些日志，比如在一些关键的函数里打印出入参，然后重新打包发布，如果打了日志还是没有找到问题，继续加日志，重新打包发布。对于上线流程复杂而且审核比较严的公司，从改代码到上线需要层层的流转，会大大影响问题排查的进度。

## 九、JVM实战调优——案例汇总

### 1、系统升级反而卡顿

- （上节讲过），给大家回顾一下，有一个在线文件阅读一个场景。他硬件原来是1.5G内存，虽然运行慢，但是他不会出问题，但是后来给他升级内存之后就变卡了，就是说内存变大了，原来的垃圾回收器想回收整个内存的时间反而更长。所以说不是内存变大你的系统就一定有性能上的提升，你还得选择特定的垃圾回收器才可以。

### 2、线程池不当运用产生OOM

- 不断的向list里加对象

### 3、不停的FGC

- 实际生产有一个JVM的问题，就是他们的系统反应特别的慢，很卡顿，他们的系统是（jira)，这个是做测试的提交报告用的。根据上面的命令进行定位，打开日志后发现问题，就是系统在不停的FGC。下图是实际参数。
- <img src="_media/basis/JVM/smile.png"/>
- 这个例子中的线上定位没有直接定位出来，所以下载了堆文件，但这个堆有10G，不能执行jmap，如果实在生产环境中执行，堆过大，jmap会对系统造成较大的冲击。不过虽然系统很慢，但是能用，实在太慢就重启一下
- <img src="_media/basis/JVM/smilegc.png"/>

### 4、Tomcat http-header-size过大问题

- 以下是实验环境
- <img src=" _media/basis/JVM/实验环境.png" alt="image-20201220095606332" />
- 使用jmeter模拟并发，模拟100个线程调用程序，程序中还设置了很大的内存，每调用一次就加一点，很快就满了
- <img src="_media/basis/JVM/jmeter模拟并发.png"/>
- 能把这个Http 11 outputbuffer 名字记下来就很好，问题是它占了特别大的内存，已调查是header-size设置太大
- <img src=" _media/basis/JVM/Tomcat.png" alt="image-20201220095928139" />
- <img src=" _media/basis/JVM/http11.png" alt="image-20201220100647339" />
- 结论：请求过多，设置堆内存太小，会导致oom问题

### 5、lambda表达式导致方法区溢出

- lambda表达式导致方法区溢出问题(MethodArea / Perm Metaspace) LambdaGC.java

- -XX:MaxMetaspaceSize=9M -XX:+PrintGCDetails，下面小程序叫LambdaGC，这个是实验性质的，在大家去面试的时候千万不要去聊这个东西。lambda表达式他是一个比较特殊的东西，他会不断地产生内部类，而且内部类会产生内部类的对象，就是说每个lambda表达式就是一个内部类

- <img src=" _media/basis/JVM/LambdaGC.png" alt="image-20201219145456735" />

- 设置完方法区后，运行出现溢出，如下是运行时打印情况

- ```java
  "C:\Program Files\Java\jdk1.8.0_181\bin\java.exe" -XX:MaxMetaspaceSize=9M -XX:+PrintGCDetails "-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.1\lib\idea_rt.jar=49316:C:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.1\bin" -Dfile.encoding=UTF-8 -classpath "C:\Program Files\Java\jdk1.8.0_181\jre\lib\charsets.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\deploy.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\access-bridge-64.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\cldrdata.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\dnsns.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\jaccess.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\jfxrt.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\localedata.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\nashorn.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\sunec.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\sunjce_provider.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\sunmscapi.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\sunpkcs11.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\zipfs.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\javaws.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\jce.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\jfr.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\jfxswt.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\jsse.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\management-agent.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\plugin.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\resources.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\rt.jar;C:\work\ijprojects\JVM\out\production\JVM;C:\work\ijprojects\ObjectSize\out\artifacts\ObjectSize_jar\ObjectSize.jar" com.mashibing.jvm.gc.LambdaGC
  [GC (Metadata GC Threshold) [PSYoungGen: 11341K->1880K(38400K)] 11341K->1888K(125952K), 0.0022190 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
  [Full GC (Metadata GC Threshold) [PSYoungGen: 1880K->0K(38400K)] [ParOldGen: 8K->1777K(35328K)] 1888K->1777K(73728K), [Metaspace: 8164K->8164K(1056768K)], 0.0100681 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
  [GC (Last ditch collection) [PSYoungGen: 0K->0K(38400K)] 1777K->1777K(73728K), 0.0005698 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
  [Full GC (Last ditch collection) [PSYoungGen: 0K->0K(38400K)] [ParOldGen: 1777K->1629K(67584K)] 1777K->1629K(105984K), [Metaspace: 8164K->8156K(1056768K)], 0.0124299 secs] [Times: user=0.06 sys=0.00, real=0.01 secs] 
  java.lang.reflect.InvocationTargetException
  	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  	at java.lang.reflect.Method.invoke(Method.java:498)
  	at sun.instrument.InstrumentationImpl.loadClassAndStartAgent(InstrumentationImpl.java:388)
  	at sun.instrument.InstrumentationImpl.loadClassAndCallAgentmain(InstrumentationImpl.java:411)
  Caused by: java.lang.OutOfMemoryError: Compressed class space
  	at sun.misc.Unsafe.defineClass(Native Method)
  	at sun.reflect.ClassDefiner.defineClass(ClassDefiner.java:63)
  	at sun.reflect.MethodAccessorGenerator$1.run(MethodAccessorGenerator.java:399)
  	at sun.reflect.MethodAccessorGenerator$1.run(MethodAccessorGenerator.java:394)
  	at java.security.AccessController.doPrivileged(Native Method)
  	at sun.reflect.MethodAccessorGenerator.generate(MethodAccessorGenerator.java:393)
  	at sun.reflect.MethodAccessorGenerator.generateSerializationConstructor(MethodAccessorGenerator.java:112)
  	at sun.reflect.ReflectionFactory.generateConstructor(ReflectionFactory.java:398)
  	at sun.reflect.ReflectionFactory.newConstructorForSerialization(ReflectionFactory.java:360)
  	at java.io.ObjectStreamClass.getSerializableConstructor(ObjectStreamClass.java:1574)
  	at java.io.ObjectStreamClass.access$1500(ObjectStreamClass.java:79)
  	at java.io.ObjectStreamClass$3.run(ObjectStreamClass.java:519)
  	at java.io.ObjectStreamClass$3.run(ObjectStreamClass.java:494)
  	at java.security.AccessController.doPrivileged(Native Method)
  	at java.io.ObjectStreamClass.<init>(ObjectStreamClass.java:494)
  	at java.io.ObjectStreamClass.lookup(ObjectStreamClass.java:391)
  	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1134)
  	at java.io.ObjectOutputStream.defaultWriteFields(ObjectOutputStream.java:1548)
  	at java.io.ObjectOutputStream.writeSerialData(ObjectOutputStream.java:1509)
  	at java.io.ObjectOutputStream.writeOrdinaryObject(ObjectOutputStream.java:1432)
  	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1178)
  	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
  	at javax.management.remote.rmi.RMIConnectorServer.encodeJRMPStub(RMIConnectorServer.java:727)
  	at javax.management.remote.rmi.RMIConnectorServer.encodeStub(RMIConnectorServer.java:719)
  	at javax.management.remote.rmi.RMIConnectorServer.encodeStubInAddress(RMIConnectorServer.java:690)
  	at javax.management.remote.rmi.RMIConnectorServer.start(RMIConnectorServer.java:439)
  	at sun.management.jmxremote.ConnectorBootstrap.startLocalConnectorServer(ConnectorBootstrap.java:550)
  	at sun.management.Agent.startLocalManagementAgent(Agent.java:137)
  ```

- 整个方法没有结束它就不会清理，关于方法区的清理每个垃圾回收器都不一样，有的不会清理，有的会清，有的还特别苛刻，如果面试官没有问题你，尽量不要提，很少发生这事儿，如果你想显得特别牛X的时候可以用这个案例lambda会产生方法区的溢出。总而言之，这个案例告诉大家一点，方法区也会溢出的。

### 6、直接内存溢出

大家去查：《深入理解Java虚拟机》P59，是非常少见的，自己去使用Unsafe分配直接内存，或者使用NIO的问题。

### 7、栈溢出问题

- 栈溢出问题 ：
  - -Xss设定太小，这个也不要和面试官说，自己知道怎么回事就好了，
  - 一个方法不断得用另一个方法，栈肯定就溢出了，一直调用导出栈溢出。
- 这个是写法上的内容，比较一下下面两段程序的异同，分析哪个是更优写法
  - <img src=" _media/basis/JVM/obj写法1.png" alt="image-20201219152026586" />
  - <img src=" _media/basis/JVM/obj写法2.png" alt="image-20201219152054206" />
  - 第一种只会有一个O，而且创建一个object他就会把原来的丢弃回收不指向原来的引用，
  - 第二种每一次调用就会创建一个，他都会指向引用
  - 对于垃圾回收来讲，如果只有一个对象，new出来一个Object，在第二次循环的时候就指向了第二个Object，那么原来的Object没有任何指向，垃圾回收器一来就回收了。如果循环没有结束这里对象会不断产生，而且还有一个引用指向它。

### 8、重写finalize引发频繁GC

- 这是小米云真实案例，HBase同步系统，系统通过nginx访问超时报警，原因是系统cpu飙高，频繁GC，不断产生新对象，原因是因为C++程序员重写finalize引发频繁GC问题，为什么C++程序员会重写finalize？
- 因为C++是手动回收内存的，手动回收内存的话怎么写？ new，new完了之后自己要定义析构函数，定义完后要delete这个对象，然后又理所当然的认为Java里面是不是也有类似的C++的析构函数，然后他就发现了有一个方法叫finalize估计和那个析构函数很类似所以他就重写了。因为finalize耗时比较高，如果好多对象同时诞生了，但是你回收这个对象的时候默认会调用finelize，每个都会耗时长你回收不完引发频繁GC。（new delete）finalize耗时比较长（200ms）
- 这个问题：如果有一个系统，内存一直消耗不超过10%，但是观察GC日志，发现FGC总是频繁产生，会是什么引起的？
  - 有人调用了System.gc(）别跟面试官说这个，太low了。

### 9、Distuptor内存溢出

- Distuptor有个可以设置链的长度，如果过大，然后对象大，消费完不主动释放，会溢出

## 十、垃圾回收算法串讲

<img src="_media/basis/JVM/常用的垃圾回收器.png"/>

分代的这种是通过组合用的（Serial和Serial Old组合）、（Parallel Scavenge和ParallelOld并行回收）、（ParNew和CMS）。 G1回收器是在逻辑上有分代在物理上没有的，到了ZGC再也不分代了，可以说VM调优这件事儿以后会越来越容易，Shenandoah他就一个分代了。

在CMS里面一个关键点是一个井发的回收，回顾一下以前讲的东西Serial和Serialold回收，就是我们工作线程玩一会，分到我垃圾回收线程玩一会，工作线程再继续，单线程的叫Serial，多线程的就叫Parallel，这种的都叫做并行的回收或者单线程的回收，但是你不能叫并发回收，到cms这个级别就是并发回收，意思就是我的垃圾回收线程和我的工作线程是同时工作的，相当于一边拉一边产，一边产生垃圾一边儿收拾，对这样应用程序的效率响应时间就友好很多了，在这个里面其实有一条线，是内存越扩越大的过程，产生不同的垃圾回收器。

原来一个垃圾回收器就足够了，内存小，后面变大了之后还用Serial停顿时间变得特别长，Parallel多线程开始上了，一个人不行两个人上了，但是内存越变越大，原来一个小房间现在变成大广场了，你就算有好多个人收拾需要的时间也太长了，后面就诞生了cms，你边产生我边回收，不过内存越变越大变成了天安门大广场，只要我有一次垃圾回收扫描过程，这时间就太长了，因为内存太大了，无论你怎么调优它的停顿时间都太长了，你任何一个算法都离不开这整个这个代的完整过程，他天生的毛病，这时候G1就诞生了，他把内存分成一小块一小块的，你在这个小块玩，我去回收另一小块，分而治之。到了ZGC的时候又变的比较灵活，有小的有大的。

### 1、CMS

- 的诞生过程和核心算法：
  - 他的核心算法和G1没有太大区别，cms它在整个垃圾回收比重比不是那么高，不过他是承上启下的过程，正是因为他的出现才有了G1，ZGC，cms他的诞生是一波三折，原来人们从来没有想过垃圾回收线程可以和生产线程同时工作，非常的厉害，有一个哥们儿他是发明垃圾回收算法的，从他写了那篇论文之后用了10年以上才产生了cms这个垃圾回收器，它的产生还是挺困难的，后来人们在应用cms过程之中发现了这个效果啊怎么用都不是特别好，因为cms原来讲过他有两大毛病一个是碎片化，一个是浮动垃圾，尤其是产生了碎片化严重了之后，你后面单线程的FGC效率简直就让人无法忍受，所以cms这个垃圾回收器当你用它的时候你的调优目标就是尽量的不要有FGC，但是他在本质上你是避免不了的，所以CMS就比较尴尬，你像上面每一个回收器他都是默认的，CMS就都不是，不过依然不可否定他的存在，他是一个承上启下的过程。原来我们都是我干完了你接着干，现在呢是我们两个可以一块儿干。具体怎么实现的呢我们往下看，回顾一下过程。
- 图
- CMS从大的方面来分它分为4个阶段：
  - 第一个阶段叫initial mark初始标记
    - 意思就是他先通过我们GCroots我们的根找到根对象，这第一个阶段就完了，但是要注意的是他是STW，不过即便他是STW，因为我们找到的根对象特别少，所以它STW的时间会非常的短。
  - 第二个阶段叫concurrent mark并发标记
    - 并发标记会发生很多次，与此同时我们在并发的时候，我们其他的工作线程也在不断改变的这些引用的它的指向，这时候就非常容易出错，他是最耗时间的阶段，并发执行了，这时候它不产生STW，他就对用户的响应就比较及时，他到底怎么做的？
    - 找到根上对象之后，这个阶段叫并发标记也是最耗时的一个阶段，它把最耗时的一个阶段并发执行了，和我们工作线程一块儿执行，这时候不产生STW，对于用户的响应就比较及时，怎么做的呢？
    - 他会从根对象继续往下找，但是找的过程之中很有可能会发生一件事，就是原来那个垃圾被我加了一个引用他就不是垃圾了，我就不会把他给回收掉，如果说是在我并发标记过程之中它变成不是垃圾的，这个时候就会进入remark阶段重新标记，标记那些在我上一个阶段改的过程中改的这些对象，因为应用改的不是特别多，remark也是个STW的过程，不过他的时间也不长，所以他就可以有效的控制暂停时间。
  - 第三个阶段叫remark重新标记
    - 从垃圾变成不是垃圾的，把漏标的重新标记，虽然他是STW但时间也不长
  - 第四个阶段叫concurrent sweep并发的回收
    - 把不用的垃圾回收，回收的过程中产生的新垃圾也就是浮动垃圾，会在下一轮进行回收
- 这个里面有两个并发的阶段，你只要在JVM里面看到 concurrent ，说明是垃圾回收线程和工作线程在一块工作，他和我们普通的并发不太一样。当然你做的特别细的话是有6个阶段，加了两个小的准备阶段，不过一般4个也差不多，一般跟面试官聊4个问题也不大就搞定了。
- 从线程的角度重新来理解cms，最开始的时候是我们的工作线程在这里使劲儿的工作，你可以理解为两个小孩儿在房间里玩儿，不断的产生垃圾。垃圾到达一定程度的时候会触发垃圾回收过程，垃圾回收过程首先做了一个初始标记，初始标记找根儿上的对象，由于它不多，虽然是STW的，但STW的时间并不长。最耗时的时间在于并发标记上，因为我把最耗时间的这部分和我们正常的工作线程放在一块儿来做，所以保证了我的应用程序的响应时间，这是并发标记。很不幸的是在我应用程序的执行过程当中有一个引用又指向它了，这时候它又从垃圾变成不是垃圾了，所以我才会有一个重新标记。重新标记的过程中是找到我这些漏标的，重新标记好，这时候一定要STW，不然我重新标记的过程中你产生新的了。虽然是STW但实际的时间并不长。最后我们进入并发清理，可能我们好多个线程一块儿来进行清理。并发清理的过程中也会产生新的垃圾，这些垃圾叫做浮动垃圾，这些浮动垃圾会在下一次GC回收过程之中回收
- <img src=" _media/basis/JVM/CMS.png" alt="image-20201217162021534" />

### 2、G1

之前讲到的这些垃圾回收器，它们的内存都是连续的，分大块的，块一旦特别大的时候，无论什么调优都没戏了，所以G1带来的就是整个内存模型的改变

- 大家如果做对G1的一个入门的话可以参考这篇文章https://www.orale.com/technical-resources/articles/java/g1gc.html
- severstyle：主要运行在服务器端的一个垃圾回收器，主要用于Hospot上面。它的目标是通过并行和并发的手段，能够达到暂停时间特别短，维持不错的吞吐量，据研究他的吞吐量比PS降低了10%-15%，他的停顿时间达到200毫秒。就看你的自己的一个预期了，要是想让你的应用程序不管怎么样200毫秒之内都有响应用G1。这就是它的由来。
- <img src=" _media/basis/JVM/g1.png" alt="image-20201220105912911" />
- 这是G1原理性的模型，原来在物理上都是两块连续的内存，一块是新生代，一块是老年代。每次对他们操作都是在这么连续的一块操作，空间太大了，无论怎么样你的效率都已经提升不了太多了。所以，这时候我们就要改变自己的内存布局
- 在软件架构设计有两大思想极其重要，一个是分而治之，一个是分层，现在分而治之用的越来越多，他把内存分为Region，从G1开始他的内存变成一小块一小块的，从1M2M最大到32M。每一份Region在逻辑上依然属于某一个分代，这个分代分为四种：
  - 第一种Old区都是放老对象的、
  - Survivor区放存活对象的、
  - Eden区放新生对象的、
  - Humongous区大对象区域，对象特别大有可能会跨两个Region。所以G1的内存模型已经完完全全的和以前的分代模型不一样了。再回去理解上图中的garbage first就是来自这里。
- ![image-20201220110445677]( _media/basis/JVM/g1模型.png"/>
- 它到底有一些什么样的特点：
  - 1、并发收集；
  - 2、压缩空闲空间不会延长GC的暂停时间；
  - 3、更易预测的GC暂停时间；
  - 4、适用不需要现实很高的吞吐量的场景。
- CMS也是并发收集，他们两个算法在本质上没有区别，但是ZGC和Shenandoah的算法就有本质上的区别了，那两个是颜色指针（Colorpointer），这个词的用处就是“打”面试官用。简单说一下他们区别（记下这两个名字）
  - 三色标记：把对象分为三个不同额颜色，每个不同的颜是标志他到底有没有标记过，还是标记一半了，还是没有标记。
  - 颜色指针：一个指针猜我们内存中没有进行压缩他是64位，他会拿出来3位，来在其中做一下标记，来标记变化过得指针。
- 每个分区都可能是年轻代也可能是老年代，但是在同一时刻只能属于某个代。年轻代、幸存代、老年代这些概念还存在，成为逻辑上的概念，这样方便复用之前分代框架的逻辑。在物理上不需要连续，则带来了额外的好处
- 有的分区内垃圾对象特别多，有的分区内垃圾对象很少，G1会优先回收垃圾对象特别多的分区，这样可以花费较少的时间来回收这些分区的垃圾，这也就是G1名字的由来，即首先收集垃圾最多的分区。新生代其实并不是使用与这种算法的，依然是在新生代满了的时候，对整个新生代进行回收
- 整个新生代中的对象，要么被回收。要么晋升，至于新生代也采取分区机制的原因，则是因为这样跟老年代的策略统一，方便调整代的大小。G1还是一种带压缩的收集器，在回收老年代的分区时，是将存活的对象从一个分区拷贝到另外一个可用分区，这个拷贝的过程就实现了局部的压缩。每个分区的大小从1M到32M不等，但是都是2的幂次方。
- G1的内存区域不是固定的E或者O，比如说：在某一个时间段这一块区域是E区，但是我进行一次YGC回收之后我把这个E区擦除了，下一次回收时候我可能就把他当做O区来用了，所以说他比较灵活。
- <img src=" _media/basis/JVM/g1区域.png" alt="image-20201220111549437" />
- G1中的三个概念：
  - CSet：有哪些card（下面cardtable解释）需要被回收，会收集到CSet表格里
    - <img src=" _media/basis/JVM/cset.png" alt="image-20201220111801403" />
  - Rset：每一个Region里面都有一个表格，是它的hash表，都记录了其他的Region中的对象到本Region的引用
    - <img src=" _media/basis/JVM/RSet.png" alt="image-20201220113347823" />
  - cardtable：主要用于分代模型里帮助我们垃圾回收比较快，存在Old区，因为Y区每次都要全部扫描
    - <img src="_media/basis/JVM/cardtable.png"/>
  - 比如：在我的年轻代有好多对象，在我的老年代里也有好多对象，那你怎么确定他活着呢？通过根对象找的话他很有可能到了old区了，然后o区他可能还指向了年轻代里面，这样的话你要去遍历整个老年代，效率得多低啊，所以他这把内部分为一个一个card，所以这些对象存在不同的card里面，如果有一个card里面对象指向到年轻代，他会把这个标记为Dirty，会有一个bitmap来代表
    - Card Table 由于做YGC时，需要扫描整个OLD区，效率非常低，所以JVM设计了CardTable，如果个OLD区CardTable中有对象指向Y区，就将它设为Dirty，下次扫描时，只需要扫描DirtCard，在结构上，Card Table用BitMap来实现。
    - Card Table是一个表格，里面记录了每一个card的信息
- 整个JVM来说它的一些前沿性的东西
  - 第一个是，阿里的多租户JVM，他们在研究自己的JVM，他会把他分为好几个小块分给租户用，整体是一个完整的JVM，他们都够在同一个JVM里共享一些东西，会更方便，比独立的JVM更方便。
  - 第二个是，很多java用的webapp，他有一个特点，一个访问来了他就会产生一个对象，访问走了这些对象马上就没有，这些对象我能不能标记为这个访问来了我标记为sessionGC，用完直接回收，本质上也是分而治之，这个效率就高多了。
  - 每个Region有多大，可以通过参数来指定，我们到参数总结的时候会把这个总结在一起。
  - G1还有一个特点，就是它新老年代的一个比例，原来默认是1：2在原来分代模型里头。但是这个比例也是可以指定的，不过你一旦指定之后是不能变的。作为G1来说它的新老年代比例是动态的，一般不用手工指定。因为这个G1预测停顿时间的基准。就是G1会跟踪每一次停顿，每次STW，假如你的Y区用时比较长，他就会把Y区调小一点，我就不用手动调，我是动态调的。
- humongous object：就是超过单个region的百分之50称之为大对象，当然也有跨越多个region的这种
- GC什么时候触发？
  - Eden空间不足；
  - Old空间不足或者System.gc()。
  - G1的GC分为两种一个是YGC一种是FGC，第一种呢不难。伊甸区空间不足了，这里G1什么时间会产生FGC，多线程并行执行，就是最后对象分配不开了。
- G1如果产生FGC，你应该怎么办？
  - 扩内存
  - 提高CPU性能（回收的快，业务逻辑产生对象的速度固定，垃圾回收越快，内存空间越大）
  - 降低MixedGC触发的阈值，让MixedGC提早发生（默认是45%）这个应该是面试官想问的，要把这个回答出来。
- MixedGC
  - 就比如YGC不行了，对象产生特别多到45%，占堆内存的空间超过45%，默认就启动MixedGC，这个值是可以自己定的，MixedGC相当于一套完整的CMS。
  - MixedGC的过程？
    - 初始标记STW
    - 并发标记
      - <img src=" _media/basis/JVM/mixedgc并发标记.png" alt="image-20201220113921158" />
    - 最终标记STW（重新标记）：处理漏标
      - <img src=" _media/basis/JVM/mixedgc重新标记.png" alt="image-20201220114003538" />
    - 筛选回收STW（并行）
      - <img src=" _media/basis/JVM/mixedgc筛选回收.png" alt="image-20201220114135297" />
      - 混合回收，不分Y、O
      - 那个region满了，就回收哪个
      - 回收最具有价值的garbage
- 并发标记的算法
  - 接下来讲到的算法，CMS、G1，最核心的在于并发标记，就是一边干活，一边标记垃圾，难点在于：标记对象的过程中，对象引用关系发生改变
  - <img src=" _media/basis/JVM/并发标记算法.png" alt="image-20201220114335927" style="zoom: 50%;" />

### 3、三色标记算法

- 在CMS和G1里面用的都是同一个算法，这个算法叫三色标记，把对象在逻辑上分成三种颜色，
  - 第一种颜色叫黑色，它自己是不是垃圾已经被标记完了，而且成员变量会牵扯到它引用的一些对象也已经标记完了，这时候我们称之为这个对象是黑色；
  - 第二种是灰色，本身标记完了，但是还没有标记到它所引用的那些对象，引用的那些对象还是白色没有标记到的，所以这个时候它叫做灰色；
  - 第三种叫白色就是没被标记到的对象，这些叫白色。
  - <img src=" _media/basis/JVM/三色标记算法.png" alt="image-20201220114415918" />
- 看下图
  - 比如所有一个对象A，它所引用的对象B和C都已经标记完了，那它自己也标记完了—— 黑色，黑色不会重新标记再扫描
  - B和C里头，看B还有一个引用指向白色对象没标记到，那这个B就是灰色的。自己遍历到，孩子还没有遍历到就是灰色。
  - D就是白色没有遍历到的节点，自己遍历到孩子还没有遍历到就是灰色。
  - 漏标：什么情况下会产生漏标，黑色指向白色，指向白色对象其他引用没了，这样就会产生漏标，你就遍历不到了。必须两个条件同时具备：
    - 黑色指向白色，
    - 灰色指向白色的没了。
  - 我们CMS的和G1的核心就是在于并发标记的线程和我们工作的线程同时进行，只有这个阶段会发生漏标。
  - <img src="_media/basis/JVM/漏标.png"/>
- 漏标的两种解决方案
  - 第一种：跟踪A指向D的增加（incremental update）；
    - 比如说A指向D的时候我跟踪这个引用，产生了这个引用之后我把A重新标记为灰色，原来是黑的不会进行扫描，但是我重新给你标记成灰色，当我下一次扫描的时候我发现这个一个灰色对象那我就重新扫描它的孩子们，所以这个D又被我找到了，这种的叫做incremental update（增量跟新），产生新的标记之后，关注引用的增加。CMS就是用的这个算法。
  - 第二种是跟踪B指向D的消失（SATB snapshot at the beginning）。
    - 关注引用的删除，SATB，刚开始做一个快照，当B和D消失的时候要把这个引用推到GC的堆栈，保证D还能被GC扫描到，最重要的是要把这个引用推到GC的堆栈，是灰色对象指向白色的引用，如果一旦某一个引用消失掉了，我会把他放到栈，我其实还是能找到它的，我下回直接扫描他就行了，那样白色就不会漏标，这个是G1使用的。
  - 问题：G1他为什么使用SATB？不用incremental update？
    - 因为变成灰色成员还要重新扫，重新再来一遍，效率太低了。
    - 灰色 -> 白色引用消失时，如果没有黑色指向白色引用会被push到堆栈，下次扫描（重新标记阶段）时拿到这个引用，由于RSet的存在，不需要扫描整个堆去查找指向白色的引用，效率比较高。SATB配合RSet，浑然天成。
-  RSet与赋值的效率
  - 由于RSet的存在，那么每次给对象赋值引用的时候，就得做一些额外的操作。
  - 指的是在RSet中做一些额外的记录（在GC中被称为写屏障）。这个写屏障不等于内存屏障，这个写屏障是GC专有的写屏障。
  - No Silver Bullet

## 十一、JVM常见参数总结

### 1、CMS日志分析

- 首先启动Linux，执行命令：

  - ```java
    java -Xms20M -Xmx20M -XX:+PrintGCDetails -XX:+UseConcMarkSweepGC com.jvm.test.T15_FullGC_Problem01
        // -Xms 堆大小
        // -XX:+PrintGCDetails 打印GC的详细信息
        // -XX:+UseConcMarkSweepGC 启动CMS
    ```

- 下图是年轻代回收的日志信息，刚开始是YGC，是ParNew垃圾回收器做的，Allocation Failure（分配失败）所以ParNew开始执行的对象越来越多，年轻代已经满了，启动年轻代垃圾回收器

  - 图
  - ParNew：年轻代垃圾回收器
  - 6144 -> 640：收集前后对比
  - （6144）：整个年轻代的大小
  - 6585 -> 2770：整个堆的情况
  - （19840）：整个堆的大小

- 下图是日志代码+注释：

- ```java
  [GC (CMS Initial Mark) [1 CMS-initial-mark: 8511K(13696K)] 9866K(19840K), 0.0040321 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
  	//8511 (13696) : 老年代使用（最大）
  	//9866 (19840) : 整个堆使用（最大）
  //一般来说PN+CMS+SO，单线程对于old区markcompact，机器会完全卡死，新生代升不到老念叨，如果频繁发生SerialOld卡顿，应该调小，调小代表着原来到了68%会发生CMS，调小到34%就会发生CMS，那我这时候CMS已经在运行了，不断的开始回收了，哪怕这时候对象不断往里回收，后面还有预留的空间，这时候就不容易发生SerialOld，当然调小之后会频繁CMS回收
  // XX:+CMSInitiatingOccupancyFraction 使用多少比例的老年代后开始CMS手机，默认是68%（近似值）如果频繁发生SerialOld卡顿，应该调小
  [CMS-concurrent-mark-start] // 并发标记
  [CMS-concurrent-mark: 0.018/0.018 secs] [Times: user=0.01 sys=0.00, real=0.02 secs] 
  	//并发时间，这里的时间意义不大，因为是并发执行
  [CMS-concurrent-preclean-start]//清理之前的阶段，它把某些card标记为dirty，gc去标记的代记引用
  [CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
  	//标记Card为Dirty，也称为Card Marking
  [GC (CMS Final Remark) [YG occupancy: 1597 K (6144 K)][Rescan (parallel) , 0.0008396 secs][weak refs processing, 0.0000138 secs][class unloading, 0.0005404 secs][scrub symbol table, 0.0006169 secs][scrub string table, 0.0004903 secs][1 CMS-remark: 8511K(13696K)] 10108K(19840K), 0.0039567 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
  	//STW阶段，YG occupancy:年轻代占用及容量
  	//[Rescan (parallel)：STW下的存活对象标记
  	//weak refs processing: 弱引用处理
  	//class unloading: 卸载用不到的class
  	//scrub symbol(string) table: 
  		//cleaning up symbol and string tables which hold class-level metadata and 
  		//internalized string respectively
  	//CMS-remark: 8511K(13696K): 阶段过后的老年代占用及容量
  	//10108K(19840K): 阶段过后的堆占用及容量
  
  [CMS-concurrent-sweep-start]
  [CMS-concurrent-sweep: 0.005/0.005 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
  	//标记已经完成，进行并发清理
  [CMS-concurrent-reset-start]
  [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
  	//重置内部结构，为下次GC做准备
  ```

### 2、G1日志分析

- G1推荐使用1.8就可以了，1.6没戏，1.7不成熟。

- G1首先由YGC是STW的，不管是PS+PO还是PN+CMS都需要指定Y区，Xmn不指定会有默认比例，G1不推荐制定比例，因为G1会动态调整，调整的依据就是YGC暂停时间

- 启动G1

- 下面的日志是把G1的三个阶段都复制下来了，这三个阶段不是顺序执行的，而是在一定条件之后会触发

- ```java
  [GC pause (G1 Evacuation Pause) (young) 
   (initial-mark), 0.0015790 secs]
  //pause 暂停 STW
  //young -> 年轻代 Evacuation-> 复制存活对象  发生在young区
  //initial-mark 混合回收的阶段，这里是YGC混合老年代回收，看到initial-mark的时候，Mixedgc已经开始了，伴随着YGC同时进行，YGC混合着老年代回收
     [Parallel Time: 1.5 ms, GC Workers: 1] //一个GC线程，workers启动了多少线程
        [GC Worker Start (ms):  92635.7]//时间
        [Ext Root Scanning (ms):  1.1]//从跟对象开始搜索
        [Update RS (ms):  0.0]//更新了多少
           [Processed Buffers:  1]
        [Scan RS (ms):  0.0]
        [Code Root Scanning (ms):  0.0]//又开始
        [Object Copy (ms):  0.1]//复制时间
        [Termination (ms):  0.0]
           [Termination Attempts:  1]
        [GC Worker Other (ms):  0.0]
        [GC Worker Total (ms):  1.2]
        [GC Worker End (ms):  92636.9]
     [Code Root Fixup: 0.0 ms]
     [Code Root Purge: 0.0 ms]
     [Clear CT: 0.0 ms]//开始清理cardtable
     [Other: 0.1 ms]//其他的又执行了多少毫秒
        [Choose CSet: 0.0 ms]
        [Ref Proc: 0.0 ms]
        [Ref Enq: 0.0 ms]
        [Redirty Cards: 0.0 ms]
        [Humongous Register: 0.0 ms]
        [Humongous Reclaim: 0.0 ms]
        [Free CSet: 0.0 ms]
     [Eden: 0.0B(1024.0K)->0.0B(1024.0K) Survivors: 0.0B->0.0B Heap: 18.8M(20.0M)->18.8M(20.0M)]
   [Times: user=0.00 sys=0.00, real=0.00 secs] //三个区从多少会受到多少
  //以下是混合回收其他阶段
  [GC concurrent-root-region-scan-start]
  [GC concurrent-root-region-scan-end, 0.0000078 secs]
  [GC concurrent-mark-start]
  //无法evacuation，进行FGC
  [Full GC (Allocation Failure)  18M->18M(20M), 0.0719656 secs]
     [Eden: 0.0B(1024.0K)->0.0B(1024.0K) Survivors: 0.0B->0.0B Heap: 18.8M(20.0M)->18.8M(20.0M)], [Metaspace: 38
  76K->3876K(1056768K)] [Times: user=0.07 sys=0.00, real=0.07 secs]
  ```

- 如果不知道应用程序使用的是哪种垃圾回收器的时候，除了之前讲到的方法以外，还可以使用查看日志格式的方法。

### 3、GC常用参数 

```java
* -Xmn -Xms -Xmx -Xss ： // 年轻代 最小堆 最大堆 栈空间
* -XX:+UseTLAB ： // 使用TLAB，默认打开，一般不用动
* -XX:+PrintTLAB ：// 打印TLAB的使用情况，一般不用动
* -XX:TLABSize ：// 设置TLAB大小，一般不用动
* -XX:+DisableExplictGC ：// System.gc()不管用 ，FGC（System.gc()是程序中的gc建议，打开参数后使其不管用）
* -XX:+PrintGC ：// 打印GC的信息
* -XX:+PrintGCDetails ：// GC的详细信息
* -XX:+PrintHeapAtGC ：// GC打印堆栈的情况
* -XX:+PrintGCTimeStamps ：// 发生GC的时间
* -XX:+PrintGCApplicationConcurrentTime (重要性低) ：// 打印应用程序时间
* -XX:+PrintGCApplicationStoppedTime （重要性低） ：// 打印暂停时长
* -XX:+PrintReferenceGC （重要性低） ：// 记录回收了多少种不同引用类型的引用
* -verbose:class ：// 类加载详细过程
* -XX:+PrintVMOptions ：// 打印JVM的参数
* -XX:+PrintFlagsFinal  -XX:+PrintFlagsInitial ：// 最终的 -初始化默认的参数，须会用
    java -XX:+PrintFlagsFinal -version | grep G1 // 打印G1相关的命令参数
* -Xloggc:opt/log/gc.log ：// 记录GC日志
* -XX:MaxTenuringThreshold ：// 升代年龄，最大值15，默认值为15，CMS是6
* -XX:PreBlockSpin ：// 锁自旋次数，不建议设置
* -XX:CompileThreshold ：// 热点代码检测参数, 逃逸分析 标量替换 ...   这些不建议设置
```

### 4、Parallel常用参数 PS+PO

```java
* -XX:SurvivorRatio ：// E：S0：S1 比例 8:1:1，可以调
* -XX:PreTenureSizeThreshold ：// 大对象到底多大，通过这个参数指定，有些大对象会直接分到old区
* -XX:MaxTenuringThreshold ：// 升代年龄，最大值15，默认值为15，CMS是6
* -XX:+ParallelGCThreads ：// 并行收集器的线程数，同样适用于CMS，一般设为和CPU核数相同
* -XX:+UseAdaptiveSizePolicy ：// 自动选择各区大小比例
```

### 5、CMS常用参数

```java
* -XX:+UseConcMarkSweepGC ：// CMS启动
* -XX:ParallelCMSThreads ：// CMS线程数量，默认是核的一半，因为CMS是O区，不能将所有线程都占用了，那样就相当	于是STW，需要给应用程序一些CPU
* -XX:CMSInitiatingOccupancyFraction ：// 使用多少比例的老年代后开始CMS收集，默认是68%(近似值)，如果频繁发生SerialOld卡顿，应该调小，（频繁CMS回收）
* -XX:+UseCMSCompactAtFullCollection ：//在FGC时进行压缩，当然回收过程中耗时特别长，会损失用户响应时间，是解决CMS碎片化的参数
* -XX:CMSFullGCsBeforeCompaction ：// 多少次FGC之后进行压缩，严格来讲，在CMS时进行压缩，CMS是不压缩的，可以通过参数指定，比如设置为5，那就是5次之后再压缩，是解决CMS碎片化的参数
* -XX:+CMSClassUnloadingEnabled ：// 回收永久代，回收方法区不用的class，1.8之前的
* -XX:CMSInitiatingPermOccupancyFraction ：// 达到什么比例时进行Perm（放class信息的）回收，1.8之前的
* GCTimeRatio ：// 设置GC时间占用程序运行时间的百分比
* -XX:MaxGCPauseMillis ：// 停顿时间，是一个建议时间，GC会尝试用各种手段达到这个时间，比如减小年轻代
```

### 6、G1常用参数

```java
* -XX:+UseG1GC ：// 启动G1
* -XX:MaxGCPauseMillis ：// 建议值，G1会尝试调整Young区的块数来达到这个值(10ms/100ms)
* -XX:GCPauseIntervalMillis ：// GC的间隔时间
* -XX:+G1HeapRegionSize ：// 分区大小，建议逐渐增大该值，1 2 4 8 16 32。随着size增加，垃圾的存活时间更长，GC间隔更长，但每次GC的时间也会更长，ZGC中做了改进（动态区块大小）
* G1NewSizePercent ：// 新生代最小比例，默认为5%， 年轻代的动态调整
* G1MaxNewSizePercent ：// 新生代最大比例，默认为60%， 年轻代的动态调整
* GCTimeRatio ：// GC时间建议比例，G1会根据这个值调整堆空间，和上述第二条设置其中一个即可，一般设置上面的
* ConcGCThreads ：// 线程数量
* InitiatingHeapOccupancyPercent ：// 启动G1的堆空间占用比例
```

### 7、程序、进程、线程、纤程（多线程高并发的内容）

- <img src=" _media/basis/JVM/纤程.png" alt="image-20201220120040664" />

- 纤程还有个名字叫协程，这里面我们区分四个概念程序、进程、线程、纤程。给大家的这个概念不是特别学术化的概念，而是通俗易懂的概念，非常学术的话它需要你有一定的底层知识你才能理解学术上的的进程到底是什么意思，大家可以去看《深入理解计算机系统》那本书，他实际上是操作系统出来之后用它来做一些资源的调度，所以是资源调度的单位，后来发现进程的切换实在是太消耗资源所以才有了线程和后来的纤程，其实是一层一层的优化，这也是调优，操作系统层面的调优。一个exe程序放在界面这叫一个程序，一个exe程序跑起来就叫做进程，一个程序可以有多个进程，可以启很多个QQ，不过有好多个程序是不允许启多个进程的。一个进程里面不同的执行路径就有很多个线程，在一个线程里面还可以分不同的路径的叫做纤程。

- 程序和进程都比较简单，现在来讲线程和纤程，他的区别是什么，本质上原理是一样的，操作系统怎么进行线程的切换，CPU要切换另一个线程，线程栈里面的内容保存好，把另外一个线程拿进来，所以他是通过栈来做的。纤程也是通过栈来完成。

- 线程和纤程的本质区别是一个通过内核空间一个不通过内核空间

- linux操作系统它分为两个不同的级别，一个是用户级一个是系统级，JVM是跑在用户空间里的，他要进行系统调用时候他要通过内核空间来进行调用的，这其中就包括了启动一个个线程，当启动一个线程时候需要内核调用，这个过程比较复杂，消耗资源比较多，正是因为如此这种线程级的并发就比较重量级。所以人们就探索，那干脆把线程挪到用户空间去，一个个线程不就是栈来回切换的记录吗，所以每一个纤程和每一个纤程对应的也是一个纤程栈，我自己执行时候对应第一个栈数据，其他纤程对应是另外的纤程数据，我整个运行时候运行到这里了可以让他暂停，运行到下一个了可以让他暂停，他们本质上看上去不也是CPU执行，他们之间的切换时通过用户空间的，不通过系统空间的，所以他们之间切换资源消耗比较低，切换就比较快。
- 线程间的切换发生在内核空间，纤程间的切换发生在用户空间，所以纤程更轻，在做计算时效率更高

- 到目前JDK13都没见官方支持纤程的影子，java要像想支持纤程，需要用开源第三方的纤程库。

- 启动一个纤程要占到1M内存，你的操作系统是支撑不了多少个线程，如果你的线程到了1W了，那你的机子就慢死了，大部分时间他都会花到切换上，那么纤程不一样，他可以有几万个。纤程的管理和执行现在在JDK上不成熟，不能直接支持，需要用下面这个类库

- <img src=" _media/basis/JVM/纤程依赖.png" alt="image-20201220122014246" />

  目前不建议在工作中使用，以下是使用过程，直接用maven引入类库

- ```java
  public class HelloFibeer {
      public static void main(String[] args) {
          long start = System.currentTimeMillis();
          //启动了一万个纤程
          for (int k = 0; k < 10000; k++) {
              //calc();
              //Fiber在类库代表一个纤程
              //void：纤程的返回值是void
              Fiber<Void> fiber = new Fiber(new SuspendableRunnable(){
                  public void run() throws SuspendExecution,InterruptedException{
                      calc();
                  }
              });
              fiber.start();
          }
      }
      //做计算
      static void calc(){
          int result = 0;
          for (int m = 0; m < 10000; m++) {
              for (int i = 0; i < 200; i++) {
                  result+=i;
              }
          }
      }
  }
  ```

- 它的源码是用什么写的？到底怎么执行的？如果你要用quasar的话他就是java写的，你要执行的时候只需要设定JVM options。本来它不支持纤程的，不可能支持在用户空间里头某一个纤程过来就给你分配一个小栈。怎么做到的？instrumentation这个类库做了一个agent代理，它把你cass做了内部改动，自动帮你生成对应的栈，然后帮你管理栈之间的运行。有纤程是为了更短时间的切换。