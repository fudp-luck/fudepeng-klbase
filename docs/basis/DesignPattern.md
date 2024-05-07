

## 一、Singleton 单例模式

### 1、概述

只有单个实例，应用于只需要一个实例存在的时候，比如各种 menagerie 以及各种 Factory

单例模式的实现方式有 8 种

单例首先将构造方法设成私有的（private），使其它的类无法通过 new 创建对象

### 2、饿汉式（1）

类加载到内存后，就实例化一个单例，JVM 保证线程安全，JVM 保证每个 Class 只会 load 到内存一次，而 static 是在 Class 

简单实用，推荐使用

缺点：不管用到与否，类装载时就完成实例化（用不到的，为什么要加载呢？）

实现：

```java
public class Mgr01 {
    private static final Mgr01 INSTANCE = new Mgr01();
    private Mgr01(){}
    public static Mgr01 getInstance() {
        return INSTANCE;
    }
    public void M() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        Mgr01 m1 = Mgr01.getInstance();
        Mgr01 m2 = Mgr01.getInstance();
        System.out.println(m1 == m2);
    }
}
```

### 3、饿汉式（2）

实现：

```java
public class Mgr02 {
    private static final Mgr02 INSTANCE ;
    static {
        INSTANCE = new Mgr02();
    }
    private Mgr02(){}
    public static Mgr02 getInstance() {
        return INSTANCE;
    }
    public void M() {
        System.out.println("m");
    }
    public static void main(String[] args) {
        Mgr02 m1 = Mgr02.getInstance();
        Mgr02 m2 = Mgr02.getInstance();
        System.out.println(m1 == m2);
    }
}
```

### 4、懒汉式（1）

lazy loading 也称懒汉式，什么时候用到，什么时候初始化

虽然达到了按需初始化的目的，但却带来了线程不安全（多线程访问）的问题

实现：

```java
public class Mgr03 {
    private static Mgr03 INSTANCE ;
    private Mgr03(){}
    public static Mgr03 getInstance() {
        if (INSTANCE == null){
            // 测试线程时用的
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            INSTANCE = new Mgr03();
        }
        return INSTANCE;
    }
    public void M() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                System.out.println(Mgr03.getInstance().hashCode());
            }).start();
        }
    }
}
```

### 5、懒汉式（2）

上述懒汉式的线程不安全问题可以通过加锁 synchronized 来解决，但也带来了效率下降

实现：

```java
public class Mgr04 {
    private static Mgr04 INSTANCE ;
    private Mgr04(){}
    public static synchronized Mgr04 getInstance() {
        if (INSTANCE == null){
            // 测试线程时用的
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            INSTANCE = new Mgr04();
        }
        return INSTANCE;
    }
    public void M() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                System.out.println(Mgr04.getInstance().hashCode());
            }).start();
        }
    }
}
```

### 6、懒汉式（3）

妄图通过减小同步代码块来提高效率，不可行，不能在多线程下保证同一实例

实现：

```java
public class Mgr05 {
    private static Mgr05 INSTANCE ;
    private Mgr05(){}
    public static Mgr05 getInstance() {
        if (INSTANCE == null){
            // 妄图通过减小同步代码块来提高效率，不可行
            synchronized (Mgr05.class){
                // 测试线程时用的
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                INSTANCE = new Mgr05();
            }
        }
        return INSTANCE;
    }
    public void M() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                System.out.println(Mgr05.getInstance().hashCode());
            }).start();
        }
    }
}
```

### 7、双重检查（完美）

实现：

```java
public class Mgr06 {
    private static Mgr06 INSTANCE;
    //    private static volatile Mgr06 INSTANCE;  //防止指令重排

    private Mgr06() {
    }

    public static Mgr06 getInstance() {
        if (INSTANCE == null) {
            // 双重检查
            synchronized (Mgr06.class) {
                if (INSTANCE == null) {
                    // 测试线程时用的
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

    public void M() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                System.out.println(Mgr06.getInstance().hashCode());
            }).start();
        }
    }
}
```

### 8、静态内部类方式（完美）

JVM 保证单例

加载外部类时不会加载内部类，这样可以实现懒加载

实现：

```java
public class Mgr07 {
    private Mgr07() {
    }
    private static class Mgr07Holder{
        private static final Mgr07 INSTANCE = new Mgr07();
    }

    public static Mgr07 getInstance() {
        return Mgr07Holder.INSTANCE;
    }

    public void M() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                System.out.println(Mgr07.getInstance().hashCode());
            }).start();
        }
    }
}
```

### 9、枚举类 enum（完美）

不仅可以解决线程同步，还可以防止反序列化

前面的写法，都可以找到 .class 文件 通过反序列化 new 一个实例出来

因为 enum 没有构造方法，所以就算拿到 .class 文件也无法 new 一个实例

实现：

```java
public enum Mgr08 {
    INSTANCE;
    public void m(){}

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                System.out.println(Mgr06.getInstance().hashCode());
            }).start();
        }
    }
```

## 二、Strategy 策略模式

### 1、概述

策略模式封装的是做一件事不同的执行方式

注意事项和细节
1. 策略模式的关键是:分析项目中变化部分与不变部分
2. 策略模式的核心思想是：多用组合/聚合少用继承;用行为类组合，而不是行为的继承。更有弹性
3. 体现了“对修改关闭，对扩展开放”原则，客户端增加行为不用修改原有代码，只要添加一种策略（或者行为）即可，避免了使用多重转移语句(if.else if.else)
4. 提供了可以替换继承关系的办法：策略模式将算法封装在独立的 Strategy 类中使得你可以独立于其 Context 改变它，使它易于切换、易于理解、易于扩展
5. 需要注意的是:每添加一个策略就要增加-一个类， 当策略过多是会导致类数目庞

### 2、举例

自定义 Sorter 类，针对于 int 类型的数组进行排序

```java
public class Sorter {
    public void sort(int[] arr) {
        for(int i=0; i<arr.length - 1; i++) {
            int minPos = i;

            for(int j=i+1; j<arr.length; j++) {
                minPos = arr[j] < arr[minPos] ? j : minPos;
            }
            swap(arr, i, minPos);
        }
    }
    void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

当出现了其它类型，比如 dubbo、float 或者是对象（Cat）类型时，就需要再写一次 Sorter.sort，所以此时我们使用java.lang.Comparable，已达到更加灵活的方式

```java
public class Sorter {
    public void sort(Comparable[] arr) {
        for(int i=0; i<arr.length - 1; i++) {
            int minPos = i;

            for(int j=i+1; j<arr.length; j++) {
                minPos = arr[j] < arr[minPos] ? j : minPos;
            }
            swap(arr, i, minPos);
        }
    }
    void swap(Comparable[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

```java
public class Cat implements Comparable<Cat> {
    int weight, height;

    public Cat(int weight, int height) {
        this.weight = weight;
        this.height = height;
    }

    public int compareTo(Cat c) {

        if(this.weight < c.weight) return -1;
        else if(this.weight > c.weight) return 1;
        else return 0;
    }

    @Override
    public String toString() {
        return "Cat{" +
                "weight=" + weight +
                ", height=" + height +
                '}';
    }
}
```

```java
public class Dog implements Comparable<Dog> {
    int food;
    public Dog(int food) {
        this.food = food;
    }

    @Override
    public int compareTo(Dog d) {
        if(this.food < d.food) return -1;
        else if(this.food > d.food) return 1;
        else return 0;
    }

    @Override
    public String toString() {
        return "Dog{" +
                "food=" + food +
                '}';
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Cat[] a = {new Cat(3, 3), new Cat(5, 5), new Cat(1, 1)};
        //Dog[] a = {new Dog(3), new Dog(5), new Dog(1)};
        Sorter<Cat> sorter = new Sorter<>();
        sorter.sort(a);
        System.out.println(Arrays.toString(a));
    }
}
```

此时无论是 Cat 、Dog 对象的排序方式都在实体类中定义，那么可能后期会需求根据 Cat 的身高排序，那么此时上述实现就会不灵活，所以使用 java.util.Comparator 接口

```java
public class Sorter<T> {

    public void sort(T[] arr, Comparator<T> comparator) {
        for(int i=0; i<arr.length - 1; i++) {
            int minPos = i;

            for(int j=i+1; j<arr.length; j++) {
                minPos = comparator.compare(arr[j],arr[minPos])==-1 ? j : minPos;
            }
            swap(arr, i, minPos);
        }
    }
void swap(T[] arr, int i, int j) {
        T temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

```java
@FunctionalInterface // 当接口中只有一个方法时，注解代表函数式接口声明（可不加，作用是对方法数约束）
public interface Comparator<T> {
    int compare(T o1, T o2);

    // JDK 1.8 之后可以在接口中定义默认实现方法，为了向前兼容
    default void m() {
        System.out.println("m");
    }
}
```

```java
public class CatWeightComparator implements Comparator<Cat> {
    @Override
    public int compare(Cat o1, Cat o2) {
        if(o1.weight < o2.weight) return -1;
        else if (o1.weight > o2.weight) return 1;
        else return 0;
    }
}

public class CatHeightComparator implements Comparator<Cat> {
    @Override
    public int compare(Cat o1, Cat o2) {
        if(o1.height > o2.height) return -1;
        else if (o1.height < o2.height) return 1;
        else return 0;
    }
}

public class DogComparator implements Comparator<Dog> {
    @Override
    public int compare(Dog o1, Dog o2) {
        if(o1.food < o2.food) return -1;
        else if (o1.food > o2.food) return 1;
        else return 0;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        //int[] a = {9, 2, 3, 5, 7, 1, 4};
        Cat[] a = {new Cat(3, 3), new Cat(5, 5), new Cat(1, 1)};
        //Dog[] a = {new Dog(3), new Dog(5), new Dog(1)};
        Sorter<Cat> sorter = new Sorter<>();
        sorter.sort(a, (o1, o2)->{
            if(o1.weight < o2.weight) return -1;
            else if (o1.weight>o2.weight) return 1;
            else return 0;
        });
        System.out.println(Arrays.toString(a));
    }
}
```

## 三、工厂模式

### 1、概述

工厂模式的分为三种：简单工厂（静态工厂）、工厂方法、抽象工厂，在最初的《设计模式》中是没有简单工厂的，其系列只有后面的两种模式

任何产生对象的方法或类，都可以称之为工厂，单例也是一种工厂

#### 【1】为什么有了 new 之后还需要工厂？

灵活控制生产过程

对对象进行控制，日志，修饰，权限 .... 

#### 【2】举例场景：

比如任意定制交通工具

正常情况下可以定义 Car 、Plane 类，来定制交通工具

```java
public class Car {
    public void go() {
        System.out.println("car go ...");
    }
}
public class Plane {
    public void go(){
        System.out.println("plane fly");
    }
}
public class Main {
    public static void main(String[] args) {
        Car c = new Car();
        c.go();
        Plane p = new Plane();
        p.go();
    }
}
```

当上述情况下，每次换交通工具都需要重新 new 一个新的对象，所以可以通过接口来解决

```java
public interface Moveable {
    void go();
}

public class Car implements Moveable{
    public void go() {
        System.out.println("car go ...");
    }
}

public class Plane implements Moveable{
    public void go(){
        System.out.println("plane fly");
    }
}

public class Main {
    public static void main(String[] args) {
        Moveable m = new Car();
        m.go();
    }
}
```

如果此时需要定制交工工具的生产过程，或者说可能需要对对象进行权限的控制，那么就需要工厂模式了

### 2、简单工厂（静态工厂）

通过一个工厂类来创建对象，在创建对象前可以加入权限控制

```java
/**
 * 简单工厂的可扩展性不好
 */
public class SimpleVehicleFactory {
    public Car createCar() {
        //before processing
        return new Car();
    }

    public Broom createBroom() {
        return new Broom();
    }
}
```

如果需要新增一个交通工具时，就需要新增一个方法，可扩展行不好

### 3、工厂方法

对每一个交通工具都做一个工厂类

```java
public class CarFactory {
    public Moveable create() {
        System.out.println("a car created!");
        return new Car();
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Moveable m = new CarFactory().create();
        m.go();
    }
}
```

可以更严谨的实现任意定制生产过程

### 4、产品族带来的问题

场景：

- 一个现代人开着车，拿着AK47，吃着面包
- 一个魔法人骑扫帚，拿着魔法棒，吃毒蘑菇

场景实现：

```java
public class Car {
    public void go() {
        System.out.println("Car go wuwuwuwuw....");
    }
}
```

```java
public class AK47 extends Weapon{
    public void shoot() {
        System.out.println("tututututu....");
    }
}
```

```java
public class Bread extends Food{
    public void printName() {
        System.out.println("wdm");
    }
}
```

=============================== 

```java
public class Broom extends Vehicle{
    public void go() {
        System.out.println("Broom go wuwuwuwuw....");
    }
}
```

```java
public class MagicStick extends Weapon{
    public void shoot() {
        System.out.println("diandian....");
    }
}
```

```java
public class MushRoom extends Food{
    public void printName() {
        System.out.println("dmg");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Car c = new Car();
        c.go();
        AK47 w = new AK47();
        w.shoot();
        Bread b = new Bread();
        b.printName();
    }
}
```

问题：

- 以上场景中，每一组都是一组产品组，按照上述实现逻辑，每加入一个新的产品组，就要修改或增加 Main 中的代码
- 那么如何灵活的扩展产品组？

### 5、抽象工厂灵活扩展产品组

代码演示：

```java
public abstract class AbastractFactory {
    abstract Food createFood();
    abstract Vehicle createVehicle();
    abstract Weapon createWeapon();
}
```

```java
public class ModernFactory extends AbastractFactory {
    @Override
    Food createFood() {
        return new Bread();
    }

    @Override
    Vehicle createVehicle() {
        return new Car();
    }

    @Override
    Weapon createWeapon() {
        return new AK47();
    }
}
```

```java
public class MagicFactory extends AbastractFactory {
    @Override
    Food createFood() {
        return new MushRoom();
    }

    @Override
    Vehicle createVehicle() {
        return new Broom();
    }

    @Override
    Weapon createWeapon() {
        return new MagicStick();
    }
}
```

```java
public abstract class Food {
   abstract void printName();
}

public abstract class Vehicle { //interface
    abstract void go();
}

public abstract class Weapon {
    abstract void shoot();
}
```

```java
public class Car extends Vehicle{
    public void go() {
        System.out.println("Car go wuwuwuwuw....");
    }
}

public class AK47 extends Weapon{
    public void shoot() {
        System.out.println("tututututu....");
    }
}

public class Bread extends Food{
    public void printName() {
        System.out.println("wdm");
    }
}
```

```java
public class Broom extends Vehicle{
    public void go() {
        System.out.println("Broom go wuwuwuwuw....");
    }
}

public class MagicStick extends Weapon{
    public void shoot() {
        System.out.println("diandian....");
    }
}

public class MushRoom extends Food{
    public void printName() {
        System.out.println("dmg");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        AbastractFactory f = new ModernFactory();
//        AbastractFactory f = new MagicFactory();

        Vehicle c = f.createVehicle();
        c.go();
        Weapon w = f.createWeapon();
        w.shoot();
        Food b = f.createFood();
        b.printName();
    }
}
```

上述抽象工厂中，ModernFactory 和 MagicFactory 两个具体抽象方法类分别实现了各自的产品族，当再有新的产品族加入时，依照上述逻辑，在使用中只需要修改不同的工厂实现就可以实现不同的产品族（Main）

### 6、关于工厂方法的讨论

在抽象工厂中，使用的都是 abstract 抽象类，是否可以使用 interface ？
- 主要在于语义上，使用抽象类是因为实现的方法在现实中存在，比如面包，车等
- 也可以理解为形容词用接口，名词用抽象类

工厂方法也可以作为只有一种产品的抽象工厂

工厂方法与抽象工厂的对比
1. 工厂方法比较方便于产品单一维度下的扩展
2. 抽象工厂比较方便于在产品族维度下的扩展
3. 这两种都存在一定的局限性，比这两种更优良的是 Spring 的 bean 工厂

## 四、Facade 门面（外观模式）

### 1、概述

- 当外部访问系统时，此时系统内部有特别复杂的逻辑关系，那么就可以将这些复杂关系封装到一个类中，对外提供一个统一的接口
- 外观模式可以隐藏系统复杂性
- 可以和调停者使用同一个类进行管理

### 2、模型图

- 正常情况下的系统访问：
  - <img src="C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\4、门面&调停者\普通外部访问.png" alt="image-20210418144841837" style="zoom:80%;" />
- 改进 -> 外观模式
  - <img src="C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\4、门面&调停者\外观模式.png" alt="image-20210418145031656" style="zoom:80%;" />

## 五、Mediator调停者（中介模式）

### 1、概述

- 在系统中，各个模块之间存在互相调用关系，那么当有一个新的模块或系统加入进来与其他模块系统关联时，就会变得很复杂
- 这时可以抽出一个模块，让所有的模块和系统都与其关联，各个模块系统间不关联，由这个模块来统一管理，就是调停者，中介模式
- 中介模式最大的应用就是消息中间件（Mq），其他模块需要什么消息都去上面取，实现了解耦
- 可以和 Facade 门面使用同一个类管理

### 2、模型图

- 普通的系统间调用：
  - <img src="C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\4、门面&调停者\普通系统间调用.png" alt="image-20210418145817282" style="zoom:80%;" />
- 中介模式：
  - <img src="C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\4、门面&调停者\中介模式.png" alt="image-20210418145919378" style="zoom:80%;" />

## 六、Decorator 装饰器模式

### 1、概述

- 对现有对象添加新的功能，但不改变其结构，比如对一个方块添加颜色
- ![image-20210418153121793](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\5、装饰器模式\装饰器模式.png)

## 七、ChainOfResponsibility责任链

### 1、场景及推论

- 在论坛中发表文章

- 后台要经过信息处理（过滤敏感文字）才可以发表或者进入数据库

- 初步处理

  - ```java
    public class Main {
        public static void main(String[] args) {
            Msg msg = new Msg();
            msg.setMsg("大家好:)，<script>，欢迎访问，大家都是996 ");
    	    // 处理 msg
            String r = msg.getMsg();
            r = r.replace('<','[');
            r = r.replace('>',']');
            r = r.replaceAll("996","995");
            msg.setMsg(r);
            System.out.println(msg);
        }
    }
    
    class Msg {
        String name;
        String msg;
    
        public String getMsg() {
            return msg;
        }
    
        public void setMsg(String msg) {
            this.msg = msg;
        }
    
        @Override
        public String toString() {
            return "Msg{" +
                    "msg='" + msg + '\'' +
                    '}';
        }
    }
    ```

- 进一步优化，使用过滤

  - ```java
    public class Main {
        public static void main(String[] args) {
            Msg msg = new Msg();
            msg.setMsg("大家好:)，<script>，欢迎访问，大家都是996 ");
    	    // 处理 msg
            new HTMFilter.doFilter(msg);
            new SensitiveFilter.doFilter(msg);
            System.out.println(msg);
        }
    }
    
    class Msg {
        String name;
        String msg;
    
        public String getMsg() {
            return msg;
        }
    
        public void setMsg(String msg) {
            this.msg = msg;
        }
    
        @Override
        public String toString() {
            return "Msg{" +
                    "msg='" + msg + '\'' +
                    '}';
        }
    }
    class HTMLFilter implements Filter {
        @Override
        public boolean doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replace('<', '[');
            r = r.replace('>', ']');
            m.setMsg(r);
            return true;
        }
    }
    
    class SensitiveFilter implements Filter {
        @Override
        public boolean doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replaceAll("996", "955");
            m.setMsg(r);
            return false;
        }
    }
    ```

- 进一步优化，将所有的过滤器装到 list 中，如果再有其他的过滤器，就只需要往 list 中装即可

  - ```java
    public class Main {
        public static void main(String[] args) {
            Msg msg = new Msg();
            msg.setMsg("大家好:)，<script>，欢迎访问 mashibing.com ，大家都是996 ");
           // 处理 msg
            List<Filter> filters = new ArrayList<>();
            filters.add(new HTMFilter);
            filters.add(new SensitiveFilter);
            for(Filter f : filters){
                f.doFilter(msg);
            }
            System.out.println(msg);
        }
    }
    ```

- 进一步优化，将 list 封装成一个链条

  - <img src="C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\6、责任链模式\责任链模式.png" alt="image-20210418161329361" style="zoom:80%;" />

  - <img src="C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\6、责任链模式\责任链模式2.png" alt="image-20210418163133816" style="zoom: 67%;" />

  - 这种方式下 msg 需要经过每一 filter 的过滤，每一个 filter 都负责一个责任，连接起来像链条一样，就是责任链模式，且每一个链条间是可以连接的

  - ```java
    public class Main {
        public static void main(String[] args) {
            Msg msg = new Msg();
            msg.setMsg("大家好:)，<script>，欢迎访问 mashibing.com ，大家都是996 ");
    
            FilterChain fc = new FilterChain();
            fc.add(new HTMLFilter())
                    .add(new SensitiveFilter());
    
            FilterChain fc2 = new FilterChain();
            fc2.add(new FaceFilter()).add(new URLFilter());
    
            fc.add(fc2);
    
            fc.doFilter(msg);
            System.out.println(msg);
    
        }
    }
    
    class Msg {
        String name;
        String msg;
        public String getMsg() { return msg; }
        public void setMsg(String msg) { this.msg = msg; }
    
        @Override
        public String toString() {
            return "Msg{" +
                    "msg='" + msg + '\'' +
                    '}';
        }
    }
    
    interface Filter {
        void doFilter(Msg m);
    }
    
    class HTMLFilter implements Filter {
        @Override
        public void doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replace('<', '[');
            r = r.replace('>', ']');
            m.setMsg(r);
        }
    }
    
    class SensitiveFilter implements Filter {
        @Override
        public void doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replaceAll("996", "955");
            m.setMsg(r);
        }
    }
    
    class FaceFilter implements Filter {
        @Override
        public void doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replace(":)", "^V^");
            m.setMsg(r);
        }
    }
    
    class URLFilter implements Filter {
        @Override
        public void doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replace("mashibing.com", "http://www.mashibing.com");
            m.setMsg(r);
        }
    }
    
    class FilterChain implements Filter {
        private List<Filter> filters = new ArrayList<>();
        public FilterChain add(Filter f) {
            filters.add(f);
            return this;
        }
    
        public void doFilter(Msg m) {
            for(Filter f : filters) {
                !f.doFilter(m);
            }
        }
    }
    ```

- 进一步需求，使 filter 可以控制是否向下执行，可以将 Filter 接口的 doFilter 的返回值定义为 boolean，当返回 true 时继续向下执行，返回为 false 就停止

  - ```java
    package com.mashibing.dp.cor;
    
    import java.util.ArrayList;
    import java.util.List;
    
    public class Main {
        public static void main(String[] args) {
            Msg msg = new Msg();
            msg.setMsg("大家好:)，<script>，欢迎访问 mashibing.com ，大家都是996 ");
    
            FilterChain fc = new FilterChain();
            fc.add(new HTMLFilter())
                    .add(new SensitiveFilter());
    
            FilterChain fc2 = new FilterChain();
            fc2.add(new FaceFilter()).add(new URLFilter());
    
            fc.add(fc2);
    
            fc.doFilter(msg);
            System.out.println(msg);
    
        }
    }
    
    class Msg {
        String name;
        String msg;
        public String getMsg() { return msg; }
        public void setMsg(String msg) { this.msg = msg; }
    
        @Override
        public String toString() {
            return "Msg{" +
                    "msg='" + msg + '\'' +
                    '}';
        }
    }
    
    interface Filter {
        boolean doFilter(Msg m);
    }
    
    class HTMLFilter implements Filter {
        @Override
        public boolean doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replace('<', '[');
            r = r.replace('>', ']');
            m.setMsg(r);
            return true;
        }
    }
    
    class SensitiveFilter implements Filter {
        @Override
        public boolean doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replaceAll("996", "955");
            m.setMsg(r);
            return false;
        }
    }
    
    class FaceFilter implements Filter {
        @Override
        public boolean doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replace(":)", "^V^");
            m.setMsg(r);
            return true;
        }
    }
    
    class URLFilter implements Filter {
        @Override
        public boolean doFilter(Msg m) {
            String r = m.getMsg();
            r = r.replace("mashibing.com", "http://www.mashibing.com");
            m.setMsg(r);
            return true;
        }
    }
    
    class FilterChain implements Filter {
        private List<Filter> filters = new ArrayList<>();
        public FilterChain add(Filter f) {
            filters.add(f);
            return this;
        }
    
        public boolean doFilter(Msg m) {
            for(Filter f : filters) {
                if(!f.doFilter(m)) return false;
            }
            return true;
        }
    }
    ```

### 2、对 filter 执行顺序控制

- 场景

  - 在服务器接收 request 请求时，对其按照 filter1 filter2 filter3 进行过滤
  - 在服务器返回 response 时，对其按照 filter3 filter2 filter1 进行过滤

- 写法一

  - 首先调用 FilterChain（链条）的 doFilter，FilterChain 中的 index 用来记录执行到了第几个 filter，每调用一次 doFilter 就会加一，直到没有 filter 了返回

  - 其次当 index 为 0  时，就会取出链条中下标为 0 的 filter -> HTMLFilter 进行执行

  - 在 HTMLFilter 中执行过滤条件后，再次调用 FilterChain（链条）的 doFilter，index = 0 -> index = 1，取出链条中下标为 1 的 filter -> SensitiveFilter 进行执行

  - 在 SensitiveFilter 中执行过滤条件后，再次调用 FilterChain（链条）的 doFilter，index = 1 -> index = 2，这时 index 的值已经等于链条中 filter 的个数，证明之后没有其他 filter 后返回 false

  - 继续在 SensitiveFilter 中执行对 response 的过滤条件，执行后结束，返回到 HTMLFilter 中执行 response 的过滤条件后结束

  - ```java
    public class Servlet_Main {
        public static void main(String[] args) {
            Request request = new Request();
            request.str = "大家好:)，<script>，欢迎访问 ，大家都是996 ";
            Response response = new Response();
            response.str = "response";
    
            FilterChain chain = new FilterChain();
            chain.add(new HTMLFilter()).add(new SensitiveFilter());
            chain.doFilter(request, response, chain);
            System.out.println(request.str);
            System.out.println(response.str);
    
        }
    }
    
    interface Filter {
        boolean doFilter(Request request, Response response, FilterChain chain);
    }
    
    class HTMLFilter implements Filter {
        @Override
        public boolean doFilter(Request request, Response response, FilterChain chain) {
            request.str = request.str.replaceAll("<", "[").replaceAll(">", "]") + "HTMLFilter()";
            chain.doFilter(request, response, chain);
            response.str += "--HTMLFilter()";
            return true;
        }
    }
    
    class Request {
        String str;
    }
    
    class Response {
        String str;
    }
    
    class SensitiveFilter implements Filter {
        @Override
        public boolean doFilter(Request request, Response response, FilterChain chain) {
            request.str = request.str.replaceAll("996", "955") + " SensitiveFilter()";
            chain.doFilter(request, response, chain);
            response.str += "--SensitiveFilter()";
            return true;
        }
    }
    
    
    class FilterChain implements Filter {
        List<Filter> filters = new ArrayList<>();
        int index = 0;
    
        public FilterChain add(Filter f) {
            filters.add(f);
            return this;
        }
    
        public boolean doFilter(Request request, Response response, FilterChain chain) {
            if(index == filters.size()) return false;
            Filter f = filters.get(index);
            index ++;
    
            return f.doFilter(request, response, chain);
        }
    }
    ```

- 写法二，模拟 Serverlet 中的 FilterChain，Struts 的过滤器、SpringMVC 拦截器 是同样的原理

  - ```java
    public class Servlet_Main {
        public static void main(String[] args) {
            Request request = new Request();
            request.str = "大家好:)，<script>，欢迎访问 ，大家都是996 ";
            Response response = new Response();
            response.str = "response";
    
            FilterChain chain = new FilterChain();
            chain.add(new HTMLFilter()).add(new SensitiveFilter());
            chain.doFilter(request, response);
            System.out.println(request.str);
            System.out.println(response.str);
    
        }
    }
    
    interface Filter {
        void doFilter(Request request, Response response, FilterChain chain);
    }
    
    class HTMLFilter implements Filter {
        @Override
        public void doFilter(Request request, Response response, FilterChain chain) {
            request.str = request.str.replaceAll("<", "[").replaceAll(">", "]") + "HTMLFilter()";
            chain.doFilter(request, response);
            response.str += "--HTMLFilter()";
    
        }
    }
    
    class Request {
        String str;
    }
    
    class Response {
        String str;
    }
    
    class SensitiveFilter implements Filter {
        @Override
        public void doFilter(Request request, Response response, FilterChain chain) {
            request.str = request.str.replaceAll("996", "955") + " SensitiveFilter()";
            chain.doFilter(request, response);
            response.str += "--SensitiveFilter()";
    
        }
    }
    
    
    class FilterChain {
        List<Filter> filters = new ArrayList<>();
        int index = 0;
    
        public FilterChain add(Filter f) {
            filters.add(f);
            return this;
        }
    
        public void doFilter(Request request, Response response) {
            if(index == filters.size()) return;
            Filter f = filters.get(index);
            index ++;
    
            f.doFilter(request, response, this);
        }
    }
    ```

## 八、Observer观察者（优先）

### 1、概述

- Observer观察者模式，事件处理模型
- 在处理事件时，经常使用观察者模式 + 责任链模式

### 2、推论

- 场景：假设在孩子哭的时候，父母锁产生的动作

- 可以使用 while ，一直观察着孩子的状态，直到哭的时候做处理，需要另外一个线程的介入，使其达到哭的状态

  - ```java
    /**
     * 面向对象的傻等
     */
    class Child {
        private boolean cry = false;
    
        public boolean isCry() {
            return cry;
        }
        public void wakeUp() {
            System.out.println("Waked Up! Crying wuwuwuwu...");
            cry = true;
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Child child = new Child();
            while(!child.isCry()) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("observing...");
            }
        }
    }
    ```

- 加入观察者 Dad，当 wakeUp 方法中，孩子哭的时候，调用 Dad 中 feed 进行处理

  - ```java
    class Child {
        private boolean cry = false;
        private Dad d = new Dad();
    
        public boolean isCry() {
            return cry;
        }
    
        public void wakeUp() {
            cry = true;
            d.feed();
        }
    }
    
    class Dad {
        public void feed() {
            System.out.println("dad feeding...");
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Child c = new Child();
            //do sth
            c.wakeUp();
        }
    }
    ```

- 针对孩子哭这个事件，可以有很多个观察者

  - ```java
    class Child {
        private boolean cry = false;
        private Dad dad = new Dad();
        private Mum mum = new Mum();
        private Dog dog = new Dog();
    
        public boolean isCry() {
            return cry;
        }
    
        public void wakeUp() {
            cry = true;
            dad.feed();
            dog.wang();
            mum.hug();
        }
    }
    
    class Dad {
        public void feed() {
            System.out.println("dad feeding...");
        }
    }
    
    class Mum {
        public void hug() {
            System.out.println("mum hugging...");
        }
    }
    
    class Dog {
        public void wang() {
            System.out.println("dog wang...");
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Child c = new Child();
            //do sth
            c.wakeUp();
        }
    }
    ```

### 3、观察者模式

- 问题：在上述推论中，观察者与被观察者耦合度太高，通常观察者需要根据被观察者的具体事件来做出不同处理，比如针对孩子在哪哭，在什么时间哭，都需要做不同的处理，所以在观察者模式中 event 事件是不可少的

- ![image-20210420091021261](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\8、观察者模式\观察者模式.png)

- 分离观察者与被观察者，降低耦合度

  - ```java
    class Child {
        private boolean cry = false;
        private List<Observer> observers = new ArrayList<>();
    
        {
            observers.add(new Dad());
            observers.add(new Mum());
            observers.add(new Dog());
        }
    
    
        public boolean isCry() {
            return cry;
        }
    
        public void wakeUp() {
            cry = true;
            for(Observer o : observers) {
                o.actionOnWakeUp();
            }
        }
    }
    
    interface Observer {
        void actionOnWakeUp();
    }
    
    class Dad implements Observer {
        public void feed() {
            System.out.println("dad feeding...");
        }
    
        @Override
        public void actionOnWakeUp() {
            feed();
        }
    }
    
    class Mum implements Observer {
        public void hug() {
            System.out.println("mum hugging...");
        }
    
        @Override
        public void actionOnWakeUp() {
            hug();
        }
    }
    
    class Dog implements Observer {
        public void wang() {
            System.out.println("dog wang...");
        }
    
        @Override
        public void actionOnWakeUp() {
            wang();
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Child c = new Child();
            //do sth
            c.wakeUp();
        }
    }
    ```

- 有很多时候，观察者需要根据事件的具体情况来进行处理，source 原生对象可以产生很多 event 事件，也可以有很多观察者，观察者又会对不同事件采取不同措施

  - ```java
    class Child {
        private boolean cry = false;
        private List<Observer> observers = new ArrayList<>();
    
        {
            observers.add(new Dad());
            observers.add(new Mum());
            observers.add(new Dog());
        }
    
    
        public boolean isCry() {
            return cry;
        }
    
        public void wakeUp() {
            cry = true;
    
            wakeUpEvent event = new wakeUpEvent(System.currentTimeMillis(), "bed");
    
            for(Observer o : observers) {
                o.actionOnWakeUp(event);
            }
        }
    }
    
    //事件类 fire Event
    class wakeUpEvent{
        long timestamp;
        String loc;
    
        public wakeUpEvent(long timestamp, String loc) {
            this.timestamp = timestamp;
            this.loc = loc;
        }
    }
    
    interface Observer {
        void actionOnWakeUp(wakeUpEvent event);
    }
    
    class Dad implements Observer {
        public void feed() {
            System.out.println("dad feeding...");
        }
    
        @Override
        public void actionOnWakeUp(wakeUpEvent event) {
            feed();
        }
    }
    
    class Mum implements Observer {
        public void hug() {
            System.out.println("mum hugging...");
        }
    
        @Override
        public void actionOnWakeUp(wakeUpEvent event) {
            hug();
        }
    }
    
    class Dog implements Observer {
        public void wang() {
            System.out.println("dog wang...");
        }
    
        @Override
        public void actionOnWakeUp(wakeUpEvent event) {
            wang();
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Child c = new Child();
            //do sth
            c.wakeUp();
        }
    }
    ```

- 大多数时候，我们处理事件的时候，需要事件源对象，这时可以将事件原对象传入

  - ```java
    class Child {
        private boolean cry = false;
        private List<Observer> observers = new ArrayList<>();
    
        {
            observers.add(new Dad());
            observers.add(new Mum());
            observers.add(new Dog());
        }
    
    
        public boolean isCry() {
            return cry;
        }
    
        public void wakeUp() {
            cry = true;
    
            wakeUpEvent event = new wakeUpEvent(System.currentTimeMillis(), "bed", this);
    
            for(Observer o : observers) {
                o.actionOnWakeUp(event);
            }
        }
    }
    
    class wakeUpEvent{
        long timestamp;
        String loc;
        Child source;
    
        public wakeUpEvent(long timestamp, String loc, Child source) {
            this.timestamp = timestamp;
            this.loc = loc;
            this.source = source;
        }
    }
    
    interface Observer {
        void actionOnWakeUp(wakeUpEvent event);
    }
    
    class Dad implements Observer {
        public void feed() {
            System.out.println("dad feeding...");
        }
    
        @Override
        public void actionOnWakeUp(wakeUpEvent event) {
            feed();
        }
    }
    
    class Mum implements Observer {
        public void hug() {
            System.out.println("mum hugging...");
        }
    
        @Override
        public void actionOnWakeUp(wakeUpEvent event) {
            hug();
        }
    }
    
    class Dog implements Observer {
        public void wang() {
            System.out.println("dog wang...");
        }
    
        @Override
        public void actionOnWakeUp(wakeUpEvent event) {
            wang();
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Child c = new Child();
            //do sth
            c.wakeUp();
        }
    }
    ```

- 事件也可以形成继承体系

  - ```java
    class Child {
        private boolean cry = false;
        private List<Observer> observers = new ArrayList<>();
    
        {
            observers.add(new Dad());
            observers.add(new Mum());
            observers.add(new Dog());
            // JDK 1.8 中可以将方法传递过去，实现钩子函数
            observers.add((e)->{
                System.out.println("ppp");
            });
            //hook callback function 钩子函数，当事件发生时自动执行
        }
    
    
        public boolean isCry() {
            return cry;
        }
    
        public void wakeUp() {
            cry = true;
    
            wakeUpEvent event = new wakeUpEvent(System.currentTimeMillis(), "bed", this);
    
            for(Observer o : observers) {
                o.actionOnWakeUp(event);
            }
        }
    }
    
    abstract class Event<T> {
        abstract T getSource();
    }
    
    class wakeUpEvent extends Event<Child>{
        long timestamp;
        String loc;
        Child source;
    
        public wakeUpEvent(long timestamp, String loc, Child source) {
            this.timestamp = timestamp;
            this.loc = loc;
            this.source = source;
        }
    
        @Override
        Child getSource() {
            return source;
        }
    }
    
    interface Observer {
        void actionOnWakeUp(wakeUpEvent event);
    }
    
    class Dad implements Observer {
        public void feed() {
            System.out.println("dad feeding...");
        }
    
        @Override
        public void actionOnWakeUp(wakeUpEvent event) {
            feed();
        }
    }
    
    class Mum implements Observer {
        public void hug() {
            System.out.println("mum hugging...");
        }
    
        @Override
        public void actionOnWakeUp(wakeUpEvent event) {
            hug();
        }
    }
    
    class Dog implements Observer {
        public void wang() {
            System.out.println("dog wang...");
        }
    
        @Override
        public void actionOnWakeUp(wakeUpEvent event) {
            wang();
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Child c = new Child();
            //do sth
            c.wakeUp();
        }
    }
    ```

### 4、JDK 中关于 AWT 的观察者示例

- 点击按钮产生事件

  - ```java
    public class TestFrame extends Frame {
       public void launch() {
          Button b = new Button("press me");
          b.addActionListener(new MyActionListener());
          b.addActionListener(new MyActionListener2());
          this.add(b);
          this.pack();
          
          this.addWindowListener(new WindowAdapter(){
    
             @Override
             public void windowClosing(WindowEvent e) {
                System.exit(0);
             }
          });
          this.setLocation(400, 400);
          this.setVisible(true);
       }
       
       public static void main(String[] args) {
          new TestFrame().launch();
       }
       
       private class MyActionListener implements ActionListener { //Observer
    
          public void actionPerformed(ActionEvent e) {
             ((Button)e.getSource()).setLabel("press me again!");
             System.out.println("button pressed!");
          }    
       }
       
       private class MyActionListener2 implements ActionListener {
    
          public void actionPerformed(ActionEvent e) {
             System.out.println("button pressed 2!");
          }
       }
    }
    ```

- 通过代码自己实现 JDK 示例

  - ```java
    public class Test {
       public static void main(String[] args) {
          Button b = new Button();
          b.addActionListener(new MyActionListener());
          b.addActionListener(new MyActionListener2());
          b.buttonPressed();
       }
    }
    
    class Button {
       private List<ActionListener> actionListeners = new ArrayList<ActionListener>();
       
       public void buttonPressed() {
          ActionEvent e = new ActionEvent(System.currentTimeMillis(),this);
          for(int i=0; i<actionListeners.size(); i++) {
             ActionListener l = actionListeners.get(i);
             l.actionPerformed(e);
          }
       }
       
       public void addActionListener(ActionListener l) {
          actionListeners.add(l);
       }
    }
    
    interface ActionListener {
       public void actionPerformed(ActionEvent e);
    }
    
    class MyActionListener implements ActionListener {
    
       public void actionPerformed(ActionEvent e) {
          System.out.println("button pressed!");
       }
       
    }
    
    class MyActionListener2 implements ActionListener {
    
       public void actionPerformed(ActionEvent e) {
          System.out.println("button pressed 2!");
       }
    }
    
    class ActionEvent {
       long when;
       Object source;
       
       public ActionEvent(long when, Object source) {
          super();
          this.when = when;
          this.source = source;
       }
        
       public long getWhen() {
          return when;
       }
    
       public Object getSource() {
          return source;
       }
    }
    ```

### 5、总结

- Observer、Listener、Hook、CallBack 都是观察者模式
- 在很多系统中，Observer模式往往和责任链共同负责对于事件的处理，其中的某一个observer负责是否将事件进一步传递

## 九、Composite组合模式

### 1、概述

- 树状结构专用模式
- ![image-20210420100336735](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\9、组合模式\组合模式1.png)
- 一般树状结构是有目录和文件混合组成的
  - 目录拥有子节点，文件是没有子节点的

### 2、演示

- ```java
  package com.mashibing.dp.composite;
  
  import java.util.ArrayList;
  import java.util.List;
  
  abstract class Node {
      abstract public void p();
  }
  
  class LeafNode extends Node {
      String content;
      public LeafNode(String content) {this.content = content;}
  
      @Override
      public void p() {
          System.out.println(content);
      }
  }
  
  class BranchNode extends Node {
      List<Node> nodes = new ArrayList<>();
  
      String name;
      public BranchNode(String name) {this.name = name;}
  
      @Override
      public void p() {
          System.out.println(name);
      }
  
      public void add(Node n) {
          nodes.add(n);
      }
  }
  
  
  public class Main {
      public static void main(String[] args) {
  
          BranchNode root = new BranchNode("root");
          BranchNode chapter1 = new BranchNode("chapter1");
          BranchNode chapter2 = new BranchNode("chapter2");
          Node r1 = new LeafNode("r1");
          Node c11 = new LeafNode("c11");
          Node c12 = new LeafNode("c12");
          BranchNode b21 = new BranchNode("section21");
          Node c211 = new LeafNode("c211");
          Node c212 = new LeafNode("c212");
  
          root.add(chapter1);
          root.add(chapter2);
          root.add(r1);
          chapter1.add(c11);
          chapter1.add(c12);
          chapter2.add(b21);
          b21.add(c211);
          b21.add(c212);
  
          tree(root, 0);
  
      }
  
      static void tree(Node b, int depth) {
          for(int i=0; i<depth; i++) System.out.print("--");
          b.p();
  
          if(b instanceof BranchNode) {
              for (Node n : ((BranchNode)b).nodes) {
                  tree(n, depth + 1);
              }
          }
      }
  }
  ```

- ![image-20210420101229870](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\9、组合模式\演示结果.png)

## 十、Flyweight享元模式

### 1、概述

- 共享元数据，重复利用对象
- ![image-20210420102929400](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\10、享元模式\享元模式.png)
- 比如 word 文档中，写 A B C... ，如果每写一个 A 就 new 一个小对象出来的话，那么对象就会过多
- 使用享元模式，可以将这些字母装进一个大池子里面，用到的时候就拿出来，实现对象的复用
- 本质上都是池化思想，连接池，线程池

### 2、演示

- ```java
  package com.mashibing.dp.flyweight;
  
  import java.util.ArrayList;
  import java.util.List;
  import java.util.UUID;
  
  class Bullet{
      public UUID id = UUID.randomUUID();
      boolean living = true;
  
      @Override
      public String toString() {
          return "Bullet{" +
                  "id=" + id +
                  '}';
      }
  }
  
  public class BulletPool {
      List<Bullet> bullets = new ArrayList<>();
      {
          for(int i=0; i<5; i++) bullets.add(new Bullet());
      }
  
      public Bullet getBullet() {
          for(int i=0; i<bullets.size(); i++) {
              Bullet b = bullets.get(i);
              if(!b.living) return b;
          }
  
          return new Bullet();
      }
  
      public static void main(String[] args) {
          BulletPool bp = new BulletPool();
  
          for(int i=0; i<10; i++) {
              Bullet b = bp.getBullet();
              System.out.println(b);
          }
      }
  }
  ```

- 常量池 —— 享元模式

  - ```java
    public class TestString {
        public static void main(String[] args) {
            String s1 = "abc";
            String s2 = "abc";
            String s3 = new String("abc");
            String s4 = new String("abc");
    
            System.out.println(s1 == s2); //true
            System.out.println(s1 == s3); //false
            System.out.println(s3 == s4);
            System.out.println(s3.intern() == s1);// intern 表示s3指向常量池
            System.out.println(s3.intern() == s4.intern());
        }
    }
    ```

### 3、结合 Composite 的享元模式

- ![image-20210420103921222](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\10、享元模式\享元模式2.png)
- 存在一些组合对象，上图中的 ABABBA

## 十一、Proxy静态代理与动态代理

### 1、概述及推论

- ![image-20210420104149400](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\11、代理模式\代理模式.png)

- 场景：模拟方法运行的时间以及在方法前后加入日志

- 定义方法，模拟运行时间

  - ```java
    public class Tank implements Movable {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        @Override
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    
    interface Movable {
        void move();
    }
    ```

- 记录方法运行时间

  - ```java
    public class Tank implements Movable {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        @Override
        public void move() {
            long start = System.currentTimeMillis();
    
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
    
            long end = System.currentTimeMillis();
            System.out.println(end - start);
        }
    
        public static void main(String[] args) {
            new Tank().move();
        }
    }
    
    interface Movable {
        void move();
    }
    ```

- 如果被记录的方法无法改变，可以使用继承的方案

  - ```java
    public class Tank implements Movable {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        @Override
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        public static void main(String[] args) {
            new Tank2().move();
        }
    }
    
    class Tank2 extends Tank {
        @Override
        public void move() {
            long start = System.currentTimeMillis();
            super.move();
            long end = System.currentTimeMillis();
            System.out.println(end - start);
        }
    }
    
    interface Movable {
        void move();
    }
    ```

- 继承出现的问题是：当有其他代理业务，比如加入日志，那么就要再重新继承，耦合度高，所以使用代理

  - ```java
    public class Tank implements Movable {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        @Override
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        public static void main(String[] args) {
            new TankTimeProxy(new Tank()).move();
        }
    }
    
    class TankTimeProxy implements Movable {
        Tank tank;
        public TankTimeProxy(Tank tank) {
            this.tank = tank;
        }
    
        @Override
        public void move() {
            long start = System.currentTimeMillis();
            tank.move();
            long end = System.currentTimeMillis();
            System.out.println(end - start);
        }
    }
    
    interface Movable {
        void move();
    }
    ```

- 代理有各种类型，可以加入其他的代理，比如加入日志

  - ```java
    public class Tank implements Movable {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        @Override
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        public static void main(String[] args) {
            new TankTimeProxy().move();
        }
    }
    
    class TankTimeProxy implements Movable {
        Tank tank;
        @Override
        public void move() {
            long start = System.currentTimeMillis();
            tank.move();
            long end = System.currentTimeMillis();
            System.out.println(end - start);
        }
    }
    
    class TankLogProxy implements Movable {
        Tank tank;
        @Override
        public void move() {
            System.out.println("start moving...");
            tank.move();
            System.out.println("stopped!");
        }
    }
    
    interface Movable {
        void move();
    }
    ```

- 那么如何将多个代理进行组合  —— 静态代理

### 2、静态代理

- 演示：

  - 代理类与被代理类都实现同一个接口，就实现了多个代理组合

  - ```java
    public class Tank implements Movable {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        @Override
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        public static void main(String[] args) {
    
            Tank t = new Tank();
            TankTimeProxy ttp = new TankTimeProxy(t);
            TankLogProxy tlp = new TankLogProxy(ttp);
            tlp.move();
    
    //        new TankLogProxy(
    //                new TankTimeProxy(
    //                        new Tank()
    //                )
    //        ).move();
        }
    }
    
    class TankTimeProxy implements Movable {
        Movable m;
    
        public TankTimeProxy(Movable m) {
            this.m = m;
        }
    
        @Override
        public void move() {
            long start = System.currentTimeMillis();
            m.move();
            long end = System.currentTimeMillis();
            System.out.println(end - start);
        }
    }
    
    class TankLogProxy implements Movable {
        Movable m;
    
        public TankLogProxy(Movable m) {
            this.m = m;
        }
    
        @Override
        public void move() {
            System.out.println("start moving...");
            m.move();
            long end = System.currentTimeMillis();
            System.out.println("stopped!");
        }
    }
    
    interface Movable {
        void move();
    }
    ```

- 问题：如果让 TankLogProxy 可以重用，不仅可以代理 Tank，还可以代理其他任何能代理的类型

  - 在实际业务中，时间计算和日志记录是很多类都需要的东西
  - 所以实际上要做的就是分离代理行为和代理对象 —— 动态代理

### 3、动态代理 ——  JDK

- 动态代理的代理类不是自己手写的，JDK1.8 是动态生成的

- 演示一

  - ```java
    public class Tank implements Movable {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        @Override
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        public static void main(String[] args) {
            Tank tank = new Tank();
    
            //reflection 反射 通过二进制字节码分析类的属性和方法
    
            Movable m = (Movable)Proxy.newProxyInstance(Tank.class.getClassLoader(),
                    new Class[]{Movable.class}, //tank.class.getInterfaces()
                    new LogHander(tank)
            );
    
            m.move();
        }
    }
    
    class LogHander implements InvocationHandler {
    
        Tank tank;
    
        public LogHander(Tank tank) {
            this.tank = tank;
        }
        //getClass.getMethods[]
        // proxy 是生成的代理对象，method 是被代理对象的方法
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("method " + method.getName() + " start..");
            Object o = method.invoke(tank, args);
            System.out.println("method " + method.getName() + " end!");
            return o;
        }
    }
    
    interface Movable {
        void move();
    }
    ```

  - ![image-20210420153422473](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\11、代理模式\源码1.png)

  - ClassLoader loader ：表示要用哪一个 ClassLoader 把将来返回来的代理对象 load 到内存，使用被代理对象啊即可

  - Class<?>[] interfaces：表示被代理对象应该实现哪些接口

  - InvocationHandler h ：表示被代理对象方法被调用时怎么做处理

- 演示二，横切代码与业务代码分离

  - ```java
    public class Tank implements Movable {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        @Override
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        public static void main(String[] args) {
            Tank tank = new Tank();
    
            Movable m = (Movable)Proxy.newProxyInstance(Tank.class.getClassLoader(),
                    new Class[]{Movable.class}, //tank.class.getInterfaces()
                    new TimeProxy(tank)
            );
    
            m.move();
    
        }
    }
    
    class TimeProxy implements InvocationHandler {
        Movable m;
    
        public TimeProxy(Movable m) {
            this.m = m;
        }
    
        public void before() {
            System.out.println("method start..");
        }
    
        public void after() {
            System.out.println("method stop..");
        }
    
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            before();
            Object o = method.invoke(m, args);
            after();
            return o;
        }
    
    }
    
    interface Movable {
        void move();
    }
    ```

- 演示三，通过反射观察生成的代理对象，实现原理

  - JDK 反射生成代理必须面向接口，这是由 Proxy 内部实现决定的

  - ```java
    public class Tank implements Movable {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        @Override
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    
        public static void main(String[] args) {
            Tank tank = new Tank();
    // 将生成的代理类保存下来，这时会在项目目录中出现 com 目录
            System.getProperties().put("jdk.proxy.ProxyGenerator.saveGeneratedFiles","true");
    
            Movable m = (Movable)Proxy.newProxyInstance(Tank.class.getClassLoader(),
                    new Class[]{Movable.class}, //tank.class.getInterfaces()
                    new TimeProxy(tank)
            );
            m.move();
        }
    }
    
    class TimeProxy implements InvocationHandler {
        Movable m;
        public TimeProxy(Movable m) {
            this.m = m;
        }
    
        public void before() { System.out.println("method start.."); }
    
        public void after() { System.out.println("method stop.."); }
    
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            //Arrays.stream(proxy.getClass().getMethods()).map(Method::getName).forEach(System.out::println);
            before();
            Object o = method.invoke(m, args);
            after();
            return o;
        }
    }
    
    interface Movable { void move(); }
    ```

  - 反编译的字节码文件

    - <img src="C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\11、代理模式\原理.png" alt="image-20210420161423697" style="zoom:80%;" />
    - 其中 move() 方法中调用了 super.h.invoke，它的 super 是 Proxy
    - <img src="C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\11、代理模式\原理2.png" alt="image-20210420161545862" style="zoom:80%;" />
    - 当我们 new 代理类对象时，InvocationHandler 传进去，调用 super，所以 super 中的 h 就被指定为 InvocationHandler ，也就相当于演示一中 new LogHandler，所以在调用 move() 方法时调用的 invoke 就是 LogHandler 中的 invoke

  - 流程图：

    - ![image-20210420164939394](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\11、代理模式\流程图.png)
    - asm 是目前应用最广泛的二进制字节码的操作类库，它只有 30 几 k 的大小，可以直接修改字节码文件，使用 asm 后，Java才被称为动态语言，所谓动态语言就是在运行时直接改变这个类的属性或方法

### 4、动态代理 —— Instrument

- 本身是一个钩子函数、拦截器，可以在任何一个 Class load 到内存过程中拦截，对这个 class 进行定制，直接修改它的二进制码，只不过实现相对繁琐，需要了解二进制码中每一个 0/1 都是什么意思

### 5、动态代理 —— cglib

- cglib（code generate library），代码生成，比 JDK 的反射方式简单很多，不需要接口

- 演示：

  - ```xml
    <!-- https://mvnrepository.com/artifact/cglib/cglib -->
    <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>3.2.12</version>
    </dependency>
    ```

  - ```java
    public class Main {
        public static void main(String[] args) {
            // 增强器，将原来的类增强一下
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(Tank.class);
            // 设置回调，TimeMethodInterceptor 拦截器，相当于 InvocationHandler
            enhancer.setCallback(new TimeMethodInterceptor());
            // 生成动态代理对象
            Tank tank = (Tank)enhancer.create();
            tank.move();
        }
    }
    
    class TimeMethodInterceptor implements MethodInterceptor {
    
        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
    
            System.out.println(o.getClass().getSuperclass().getName());
            System.out.println("before");
            Object result = null;
            result = methodProxy.invokeSuper(o, objects);
            System.out.println("after");
            return result;
        }
    }
    
    class Tank {
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    ```

- cglib 不实现接口，如何知道代理的是哪个方法呢？

  - ```java
    class TimeMethodInterceptor implements MethodInterceptor {
    
        // o 是生成动态代理的对象
        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
    		// 打印动态代理类的父类，是Tank
            System.out.println(o.getClass().getSuperclass().getName());
            System.out.println("before");
            Object result = null;
            result = methodProxy.invokeSuper(o, objects);
            System.out.println("after");
            return result;
        }
    }
    ```

  - 在生成动态代理对象的时候，生成的是被代理对象 Tank 的子类

  - 当 Tank 类是 final 的，那么就无法使用 cglib 实现动态代理

  - 其底层实现也是 asm

### 6、Spring AOP

- 面向切面编程，可以在不改变原有业务逻辑代码的情况下，将新的逻辑直接切入进去，可以做权限控制，事物控制（在哪开始或结束），日志等

- 演示一：

  - ```java
    public class Main {
        public static void main(String[] args) {
            ApplicationContext context = new ClassPathXmlApplicationContext("app.xml");
            Tank t = (Tank)context.getBean("tank");
            t.move();
        }
    }
    
    public class Tank {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    
    public class TimeProxy {
    
        public void before() {
            System.out.println("method start.." + System.currentTimeMillis());
        }
    
        public void after() {
            System.out.println("method stop.." + System.currentTimeMillis());
        }
    
    }
    ```

  - ```xml
    <bean id="tank" class="com.dp.spring.v1.Tank"/>
    <bean id="timeProxy" class="com.dp.spring.v1.TimeProxy"/>
    
    <aop:config>
        <aop:aspect id="time" ref="timeProxy">
            <aop:pointcut id="onmove" expression="execution(void com.dp.spring.v1.Tank.move())"/>
            <aop:before method="before" pointcut-ref="onmove"/>
            <aop:after method="after" pointcut-ref="onmove"/>
        </aop:aspect>
    </aop:config>
    ```

- 演示二：

  - ```xml
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.4</version>
    </dependency>
    ```

  - ```java
    public class Main {
        public static void main(String[] args) {
            ApplicationContext context = new ClassPathXmlApplicationContext("app_auto.xml");
            Tank t = (Tank)context.getBean("tank");
            t.move();
        }
    }
    
    public class Tank {
    
        /**
         * 模拟坦克移动了一段儿时间
         */
        public void move() {
            System.out.println("Tank moving claclacla...");
            try {
                Thread.sleep(new Random().nextInt(10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    
    @Aspect
    public class TimeProxy {
    
        @Before("execution (void com.dp.spring.v2.Tank.move())")
        public void before() {
            System.out.println("method start.." + System.currentTimeMillis());
        }
    
        @After("execution (void com.dp.spring.v2.Tank.move())")
        public void after() {
            System.out.println("method stop.." + System.currentTimeMillis());
        }
    
    }
    ```

  - ```xml
    <aop:aspectj-autoproxy/>
    
    <bean id="tank" class="com.dp.spring.v2.Tank"/>
    <bean id="timeProxy" class="com.dp.spring.v2.TimeProxy"/>
    ```

## 十二、Iterator迭代器

### 1、概述

- 容器和容器的遍历
- 构建动态扩展的容器 List.add()，有数组和链表两种方式实现
- 在物理结构上只有数组和链表两种数据结构，类似于二叉树等属于逻辑结构
- ![](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\12、迭代器模式\迭代器.png)

### 2、构建一个容器，可以添加对象

- ```java
  public class Main {
      public static void main(String[] args) {
          ArrayList_ list = new ArrayList_();
          for(int i=0; i<15; i++) {
              list.add(new String("s" + i));
          }
          System.out.println(list.size());
      }
  }
  
  
  /**
   * 相比数组，这个容器不用考虑边界问题，可以动态扩展
   */
  class ArrayList_ {
      Object[] objects = new Object[10];
      //objects中下一个空的位置在哪儿,或者说，目前容器中有多少个元素
      private int index = 0;
      public void add(Object o) {
          // 当数组长度满了之后，动态扩展，jdk 源码中扩展的是50%
          if(index == objects.length) {
              // 创建一个新的数组，长度是原有的两倍
              Object[] newObjects = new Object[objects.length*2];
              // 将原有数组数据copy过来
              System.arraycopy(objects, 0, newObjects, 0, objects.length);
              objects = newObjects;
          }
  
          objects[index] = o;
          index ++;
      }
  
      public int size() {
          return index;
      }
  }
  ```

- ```java
  public class Main {
      public static void main(String[] args) {
          LinkedList_ list = new LinkedList_();
          for(int i=0; i<15; i++) {
              list.add(new String("s" + i));
          }
          System.out.println(list.size());
      }
  }
  
  
  /**
   * 相比数组，这个容器不用考虑边界问题，可以动态扩展
   */
  class LinkedList_ {
      Node head = null;
      Node tail = null;
      //目前容器中有多少个元素
      private int size = 0;
  
      public void add(Object o) {
          Node n = new Node(o);
          n.next = null;
  
          if(head == null) {
              head = n;
              tail = n;
          }
  
          tail.next = n;
          tail = n;
          size++;
      }
  
      private class Node {
          private Object o;
          Node next;
  
          public Node(Object o) {
              this.o = o;
          }
      }
  
      public int size() {
          return size;
      }
  }
  ```

### 3、添加容器共同接口，实现替换

- ```java
  public interface Collection_ {
      void add(Object o);
      int size();
  }
  ```

- ```java
  /**
   * 相比数组，这个容器不用考虑边界问题，可以动态扩展
   */
  class ArrayList_ implements Collection_ {
      Object[] objects = new Object[10];
      //objects中下一个空的位置在哪儿,或者说，目前容器中有多少个元素
      private int index = 0;
      public void add(Object o) {
          if(index == objects.length) {
              Object[] newObjects = new Object[objects.length*2];
              System.arraycopy(objects, 0, newObjects, 0, objects.length);
              objects = newObjects;
          }
  
          objects[index] = o;
          index ++;
      }
  
      public int size() {
          return index;
      }
  
  }
  ```

- ```java
  /**
   * 相比数组，这个容器不用考虑边界问题，可以动态扩展
   */
  class LinkedList_ implements Collection_ {
      Node head = null;
      Node tail = null;
      //目前容器中有多少个元素
      private int size = 0;
  
      public void add(Object o) {
          Node n = new Node(o);
          n.next = null;
  
          if(head == null) {
              head = n;
              tail = n;
          }
  
          tail.next = n;
          tail = n;
          size++;
      }
  
      private class Node {
          private Object o;
          Node next;
  
          public Node(Object o) {
              this.o = o;
          }
      }
  
      public int size() {
          return size;
      }
  }
  ```

- ```java
  public class Main {
      public static void main(String[] args) {
          Collection_ list = new ArrayList_();
          for(int i=0; i<15; i++) {
              list.add(new String("s" + i));
          }
          System.out.println(list.size());
  
  
          ArrayList_ al = (ArrayList_)list;
          for(int i=0; i<al.size(); i++) {
              //如果用这种遍历方式，就不能实现通用了
          }
      }
  }
  ```

### 4、遍历

- 每个容器都有自己的遍历方式，所以只能容器内部实现自己的遍历方法

- ```java
  public interface Collection_ {
      void add(Object o);
      int size();
  
      Iterator_ iterator();
  }
  ```

- ```java
  public interface Iterator_ {
      boolean hasNext();
  
      Object next();
  }
  ```

- ```java
  /**
   * 相比数组，这个容器不用考虑边界问题，可以动态扩展
   */
  class ArrayList_ implements Collection_ {
      Object[] objects = new Object[10];
      //objects中下一个空的位置在哪儿,或者说，目前容器中有多少个元素
      private int index = 0;
      public void add(Object o) {
          if(index == objects.length) {
              Object[] newObjects = new Object[objects.length*2];
              System.arraycopy(objects, 0, newObjects, 0, objects.length);
              objects = newObjects;
          }
  
          objects[index] = o;
          index ++;
      }
  
      public int size() {
          return index;
      }
  
      @Override
      public Iterator_ iterator() {
          return new ArrayListIterator();
      }
  
      private class ArrayListIterator implements Iterator_{
  
          private int currentIndex = 0;
  
          @Override
          public boolean hasNext() {
              if(currentIndex >= index) return false;
              return true;
          }
  
          @Override
          public Object next() {
              Object o = objects[currentIndex];
              currentIndex ++;
              return o;
          }
      }
  }
  ```

- ```java
  public class Main {
      public static void main(String[] args) {
          Collection_ list = new ArrayList_();
          for(int i=0; i<15; i++) {
              list.add(new String("s" + i));
          }
          System.out.println(list.size());
  
          //这个接口的调用方式：
          Iterator_ it = list.iterator();
          while(it.hasNext()) {
              Object o = it.next();
              System.out.println(o);
          }
      }
  }
  ```

### 6、JDK 的实现

- ```java
  public class Main {
      public static void main(String[] args) {
          Collection c = new ArrayList();
          for(int i=0; i<15; i++) {
              c.add(new String("s" + i));
          }
  
          Iterator it = c.iterator();
          while(it.hasNext()) {
              System.out.println(it.next());
          }
      }
  }
  ```

### 7、加入泛型模拟 JDK 的实现

- ```java
  public interface Collection_<E> {
      void add(E o);
      int size();
  
      Iterator_ iterator();
  }
  ```

- ```java
  public interface Iterator_<E> { //Element //Type //K //Value V Tank
      boolean hasNext();
  
      E next(); //Tank next() Iterator_<Tank> it = ... Tank t = it.next();
  }
  ```

- ```java
  /**
   * 相比数组，这个容器不用考虑边界问题，可以动态扩展
   */
  class ArrayList_<E> implements Collection_<E> {
      E[] objects = (E[])new Object[10];
      //objects中下一个空的位置在哪儿,或者说，目前容器中有多少个元素
      private int index = 0;
      public void add(E o) {
          if(index == objects.length) {
              E[] newObjects = (E[])new Object[objects.length*2];
              System.arraycopy(objects, 0, newObjects, 0, objects.length);
              objects = newObjects;
          }
  
          objects[index] = o;
          index ++;
      }
  
      public int size() {
          return index;
      }
  
      @Override
      public Iterator_<E> iterator() {
          return new ArrayListIterator();
      }
  
      private class ArrayListIterator<E> implements Iterator_<E> {
  
          private int currentIndex = 0;
  
          @Override
          public boolean hasNext() {
              if(currentIndex >= index) return false;
              return true;
          }
  
          @Override
          public E next() {
              E o = (E)objects[currentIndex];
              currentIndex ++;
              return o;
          }
      }
  
  
  }
  ```

- ```java
  public class Main {
      public static void main(String[] args) {
          Collection_<String> list = new ArrayList_<>();
          for(int i=0; i<15; i++) {
              list.add(new String("s" + i));
          }
          System.out.println(list.size());
  
          //这个接口的调用方式：
          Iterator_<String> it = list.iterator();
          while(it.hasNext()) {
              String o = it.next();
              System.out.println(o);
          }
      }
  }
  ```

## 十三、Visitor访问者

### 1、概述

- 在结构不变的情况下动态改变对于内部元素的动作
- 一般在做编译器和抽象语法树时会用到
- ![](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\13、访问者模式\访问者.png)

### 2、演示

- 场景：有一台电脑有三个部分组（CPU...），根据不同的购买着给到不同的折扣

- ```java
  public class Computer {
      ComputerPart cpu = new CPU();
      ComputerPart memory = new Memory();
      ComputerPart board = new Board();
  
      public void acccept(Visitor v) {
          this.cpu.accept(v);
          this.memory.accept(v);
          this.board.accept(v);
      }
  
      public static void main(String[] args) {
          PersonelVisitor p = new PersonelVisitor();
          new Computer().acccept(p);
          System.out.println(p.totalPrice);
      }
  }
  
  abstract class ComputerPart {
      abstract void accept(Visitor v);
      //some other operations eg:getName getBrand
      abstract double getPrice();
  }
  
  class CPU extends ComputerPart {
  
      @Override
      void accept(Visitor v) {
          v.visitCpu(this);
      }
  
      @Override
      double getPrice() {
          return 500;
      }
  }
  
  class Memory extends ComputerPart {
  
      @Override
      void accept(Visitor v) {
          v.visitMemory(this);
      }
  
      @Override
      double getPrice() {
          return 300;
      }
  }
  
  class Board extends ComputerPart {
  
      @Override
      void accept(Visitor v) {
          v.visitBoard(this);
      }
  
      @Override
      double getPrice() {
          return 200;
      }
  }
  
  interface Visitor {
      void visitCpu(CPU cpu);
      void visitMemory(Memory memory);
      void visitBoard(Board board);
  }
  
  class PersonelVisitor implements Visitor {
      double totalPrice = 0.0;
  
      @Override
      public void visitCpu(CPU cpu) {
          totalPrice += cpu.getPrice()*0.9;
      }
  
      @Override
      public void visitMemory(Memory memory) {
          totalPrice += memory.getPrice()*0.85;
      }
  
      @Override
      public void visitBoard(Board board) {
          totalPrice += board.getPrice()*0.95;
      }
  }
  
  class CorpVisitor implements Visitor {
      double totalPrice = 0.0;
  
      @Override
      public void visitCpu(CPU cpu) {
          totalPrice += cpu.getPrice()*0.6;
      }
  
      @Override
      public void visitMemory(Memory memory) {
          totalPrice += memory.getPrice()*0.75;
      }
  
      @Override
      public void visitBoard(Board board) {
          totalPrice += board.getPrice()*0.75;
      }
  }
  ```

### 3、ASM

- 官网：http://asm.aw2.io

## 十四、Builder构建器

### 1、概述

- 构建复杂对象
- 分离复杂对象的构建和表示
- 同样的构建过程可以创建不同的表示
- 无须记忆，自然使用
- ![image-20210423141726454](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\14、构建器\构建器.png)

### 2、演示一

- 场景：在游戏中的地形，有墙，暗堡，地雷等

- ```java
  public interface TerrainBuilder {
      TerrainBuilder buildWall();
      TerrainBuilder buildFort();
      TerrainBuilder buildMine();
      Terrain build();
  }
  ```

- ```java
  public class Terrain {
      Wall w;
      Fort f;
      Mine m;
  }
  
  class Wall {
      int x, y, w, h;
  
      public Wall(int x, int y, int w, int h) {
          this.x = x;
          this.y = y;
          this.w = w;
          this.h = h;
      }
  }
  
  class Fort {
      int x, y, w, h;
  
      public Fort(int x, int y, int w, int h) {
          this.x = x;
          this.y = y;
          this.w = w;
          this.h = h;
      }
  
  }
  
  class Mine {
      int x, y, w, h;
  
      public Mine(int x, int y, int w, int h) {
          this.x = x;
          this.y = y;
          this.w = w;
          this.h = h;
      }
  }
  ```

- ```java
  public class ComplexTerrainBuilder implements TerrainBuilder {
      Terrain terrain = new Terrain();
  
      @Override
      public TerrainBuilder buildWall() {
          terrain.w = new Wall(10, 10, 50, 50);
          return this;
      }
  
      @Override
      public TerrainBuilder buildFort() {
          terrain.f = new Fort(10, 10, 50, 50);
          return this;
      }
  
      @Override
      public TerrainBuilder buildMine() {
          terrain.m = new Mine(10, 10, 50, 50);
          return this;
      }
  
      @Override
      public Terrain build() {
          return terrain;
      }
  }
  ```

- ```java
  public class Main {
      public static void main(String[] args) {
          TerrainBuilder builder = new ComplexTerrainBuilder();
          Terrain t = builder.buildFort().buildMine().buildWall().build();
          //new Terrain(Wall w, Fort f, Mine m)
          //Effective Java
          // 如果一个类中的属性特别多，比如四十多个，那么有的时候不需要都传值使用
      }
  }
  ```

### 3、演示二

- ```java
  public class Person {
      int id;
      String name;
      int age;
      double weight;
      int score;
      Location loc;
  
      private Person() {}
  
      public static class PersonBuilder {
          Person p = new Person();
  
          public PersonBuilder basicInfo(int id, String name, int age) {
              p.id = id;
              p.name = name;
              p.age = age;
              return this;
          }
  
          public PersonBuilder weight(double weight) {
              p.weight = weight;
              return this;
          }
  
          public PersonBuilder score(int score) {
              p.score = score;
              return this;
          }
  
          public PersonBuilder loc(String street, String roomNo) {
              p.loc = new Location(street, roomNo);
              return this;
          }
  
          public Person build() {
              return p;
          }
      }
  }
  
  class Location {
      String street;
      String roomNo;
  
      public Location(String street, String roomNo) {
          this.street = street;
          this.roomNo = roomNo;
      }
  }
  ```

- ```java
  public class Main {
      public static void main(String[] args) {
          // 固定使用 id name age 其余都可以选择使用
          Person p = new Person.PersonBuilder()
                  .basicInfo(1, "zhangsan", 18)
                  //.score(20)
                  .weight(200)
                  //.loc("bj", "23")
                  .build();
      }
  }
  ```

## 十五、Adapter适配器

### 1、概述

- 接口转换器，当两个类不能直接访问时使用转换器
- 举例：比如电压转接头，一面接220V，一面接110V
- 比如字节流和字符流的转换，字节流读取文件后按行输出
- ![](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\15、适配器\适配器.png)

### 2、演示

- ```java
  public class Main {
      public static void main(String[] args) throws Exception {
          FileInputStream fis = new FileInputStream("c:/test.text");
          InputStreamReader isr = new InputStreamReader(fis);
          BufferedReader br = new BufferedReader(isr);
          String line = br.readLine();
          while (line != null && !line.equals("")) {
              System.out.println(line);
          }
          br.close();
      }
  }
  ```

### 3、误区

- 常见的 Adapter 类反而不是 Adapter，而是方便于编程
  - WindowAdapter，如果实现 WindowListener 就要实现里面所有的方法，如果实现的是 WindowAdapter，就只需要关注需要的方法
  - KeyAdapter

## 十六、Bridge桥接

### 1、概述

- 双维度扩展
- 分离抽象与具体
- 用聚合方式（桥）连接抽象与具体
- ![](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\16、桥接模式\桥接模式.png)

### 2、演示

- 场景：两个人约会送礼物，礼物有多重多样的（书、花），礼物的属性也多种多样（温暖的、狂野的），当礼物和礼物的属性多了以后，如果使用继承的方式，那么就会出现很多个类来实现

- 定义礼物类

- ```java
  public abstract class Gift {
      GiftImpl impl;
  }
  ```

- 定义书并继承礼物类

- ```java
  public class Book extends GiftImpl {
  }
  ```

- 定义花并继承礼物类

- ```java
  public class Flower extends GiftImpl {
  }
  ```

- 定义礼物实现属性类

- ```java
  public class GiftImpl {
  }
  ```

- 定义温暖的并继承礼物实现属性类

- ```java
  public class WarmGift extends Gift {
      public WarmGift(GiftImpl impl) {
          this.impl = impl;
      }
  }
  ```

- 定义狂野的并继承礼物实现属性类

- ```java
  public class WildGift extends Gift {
      public WildGift(GiftImpl impl) {
          this.impl = impl;
      }
  }
  ```

- ```java
  public class GG {
      public void chase(MM mm) {
          Gift g = new WarmGift(new Flower());// 温暖的花
          give(mm, g);
      }
  
      public void give(MM mm, Gift g) {
          System.out.println(g + "gived!");
      }
  }
  ```

- ```java
  public class MM {
      String name;
  }
  ```

## 十七、Command命令

### 1、概述

- 封装命令
- 结合 cor 责任链模式实现 undo（撤销，回到上一个命令）
- 别名：Action 动作 / Transaction 事物
- ![](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\17、命令模式\命令模式.png)
- command 与 组合模式实现宏命令（由很多命令组成的大命令）
- command 与 责任链模式实现多次 undo
- transaction 回滚

### 2、演示

- ```java
  public abstract class Command {
      public abstract void doit(); //exec run
      public abstract void undo();
  }
  ```

- ```java
  public class Content {
      String msg = "hello everybody ";
  }
  ```

- ```java
  public class CopyCommand extends Command {
      Content c;
      public CopyCommand(Content c) {
          this.c = c;
      }
  
      @Override
      public void doit() {
          c.msg = c.msg + c.msg;
      }
  
      @Override
      public void undo() {
          c.msg = c.msg.substring(0, c.msg.length()/2);
      }
  }
  ```

- ```java
  public class DeleteCommand extends Command {
      Content c;
      String deleted;
      public DeleteCommand(Content c) {
          this.c = c;
      }
  
      @Override
      public void doit() {
          deleted = c.msg.substring(0, 5);
          c.msg = c.msg.substring(5, c.msg.length());
      }
  
      @Override
      public void undo() {
          c.msg = deleted + c.msg;
      }
  }
  ```

- ```java
  public class InsertCommand extends Command {
      Content c;
      String strToInsert = "http://www.mashibing.com";
      public InsertCommand(Content c) {
          this.c = c;
      }
  
      @Override
      public void doit() {
          c.msg = c.msg + strToInsert;
      }
  
      @Override
      public void undo() {
          c.msg = c.msg.substring(0, c.msg.length()-strToInsert.length());
      }
  }
  ```

- ```java
  public class Main {
      public static void main(String[] args) {
          Content c = new Content();
  
          Command insertCommand = new InsertCommand(c);
          insertCommand.doit();
          insertCommand.undo();
  
          Command copyCommand = new CopyCommand(c);
          insertCommand.doit();
          insertCommand.undo();
  
          Command deleteCommand = new DeleteCommand(c);
          deleteCommand.doit();
          deleteCommand.undo();
  
          List<Command> commands = new ArrayList();
          commands.add(new InsertCommand(c));
          commands.add(new CopyCommand(c));
          commands.add(new DeleteCommand(c));
  
          for(Command comm : commands) {
              comm.doit();
          }
  
          System.out.println(c.msg);
  
          for(int i= commands.size()-1; i>=0; i--) {
              commands.get(i).undo();
          }
          System.out.println(c.msg);
      }
  }
  ```

## 十八、Prototype原型

### 1、概述

- Java 内部自带原型模式  Object.clone()
- ![image-20210423150642548](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\18、原型模式\原型模式.png)
- 实现原型模式需要实现标记型接口 Cloneable
- 一般会重写 clone() 方法
  - 如果只是重写 clone() 方法，而没有实现接口，调用时会报异常
- 一般用于一个对象的属性已经确定，需要产生很多相同对象的时候
- 需要区分深克隆和浅克隆

### 2、演示一，浅克隆

- ```java
  /**
   * 浅克隆
   */
  
  public class Test {
      public static void main(String[] args) throws Exception {
          Person p1 = new Person();
          Person p2 = (Person)p1.clone();
          System.out.println(p2.age + " " + p2.score);
          System.out.println(p2.loc);
  
          System.out.println(p1.loc == p2.loc); // true 两个对象都指向同一个 loc
          p1.loc.street = "sh"; // 所以当 p1 改动时 p2 也会改动
          System.out.println(p2.loc);
  
      }
  }
  
  class Person implements Cloneable {
      int age = 8;
      int score = 100;
  
      Location loc = new Location("bj", 22);
      @Override
      public Object clone() throws CloneNotSupportedException {
          return super.clone();
      }
  }
  
  class Location {
      String street;
      int roomNo;
  
      @Override
      public String toString() {
          return "Location{" +
                  "street='" + street + '\'' +
                  ", roomNo=" + roomNo +
                  '}';
      }
  
      public Location(String street, int roomNo) {
          this.street = street;
          this.roomNo = roomNo;
      }
  }
  ```

### 3、演示二，深克隆

- ```java
  /**
   * 深克隆的处理
   */
  public class Test {
      public static void main(String[] args) throws Exception {
          Person p1 = new Person();
          Person p2 = (Person)p1.clone();
          System.out.println(p2.age + " " + p2.score);
          System.out.println(p2.loc);
  
          System.out.println(p1.loc == p2.loc); // false
          p1.loc.street = "sh";
          System.out.println(p2.loc);
  
  
  
      }
  }
  
  class Person implements Cloneable {
      int age = 8;
      int score = 100;
  
      Location loc = new Location("bj", 22);
      @Override
      public Object clone() throws CloneNotSupportedException {
          Person p = (Person)super.clone();
          p.loc = (Location)loc.clone(); //将 loc 引用类型也克隆一次
          return p;
      }
  }
  
  class Location implements Cloneable {
      String street;
      int roomNo;
  
      @Override
      public String toString() {
          return "Location{" +
                  "street='" + street + '\'' +
                  ", roomNo=" + roomNo +
                  '}';
      }
  
      public Location(String street, int roomNo) {
          this.street = street;
          this.roomNo = roomNo;
      }
  
      @Override
      public Object clone() throws CloneNotSupportedException {
          return super.clone();
      }
  }
  ```

### 4、演示三，String不需要深克隆

- ```java
  /**
   * String也是引用类型，需要进一步深克隆吗？
   * 不需要，因为Sting类型本身指向了常量池，本身就是共用的
   * 比如常量池中有 北京 上海，首先 p1 p2都指向北京，当p1改为上海时，p1的引用会指向上海，    不会更改北京的值
   */
  public class Test {
      public static void main(String[] args) throws Exception {
          Person p1 = new Person();
          Person p2 = (Person)p1.clone();
          System.out.println(p2.age + " " + p2.score);
          System.out.println(p2.loc);
  
          System.out.println(p1.loc == p2.loc);
          p1.loc.street = "sh";
          System.out.println(p2.loc);
  
          p1.loc.street.replace("sh", "sz");
          System.out.println(p2.loc.street);
      }
  }
  
  class Person implements Cloneable {
      int age = 8;
      int score = 100;
  
      Location loc = new Location("bj", 22);
      @Override
      public Object clone() throws CloneNotSupportedException {
          Person p = (Person)super.clone();
          p.loc = (Location)loc.clone();
          return p;
      }
  }
  
  class Location implements Cloneable {
      String street;
      int roomNo;
  
      @Override
      public String toString() {
          return "Location{" +
                  "street='" + street + '\'' +
                  ", roomNo=" + roomNo +
                  '}';
      }
  
      public Location(String street, int roomNo) {
          this.street = street;
          this.roomNo = roomNo;
      }
  
      @Override
      public Object clone() throws CloneNotSupportedException {
          return super.clone();
      }
  }
  ```

### 5、演示四

- 当p1 p2 指向同一个对象，对象在指向常量池，那么当对象改变，p1 p2 的值也会改变

- ```java
  public class Test {
      public static void main(String[] args) throws Exception {
          Person p1 = new Person();
          Person p2 = (Person)p1.clone();
          System.out.println("p1.loc == p2.loc? " + (p1.loc == p2.loc));
  
          p1.loc.street.reverse();
          System.out.println(p2.loc.street);
      }
  }
  
  class Person implements Cloneable {
      int age = 8;
      int score = 100;
  
      Location loc = new Location(new StringBuilder("bj"), 22);
      @Override
      public Object clone() throws CloneNotSupportedException {
          Person p = (Person)super.clone();
          p.loc = (Location)loc.clone();
          return p;
      }
  }
  
  class Location implements Cloneable {
      StringBuilder street;
      int roomNo;
  
      @Override
      public String toString() {
          return "Location{" +
                  "street='" + street + '\'' +
                  ", roomNo=" + roomNo +
                  '}';
      }
  
      public Location(StringBuilder street, int roomNo) {
          this.street = street;
          this.roomNo = roomNo;
      }
  
      @Override
      public Object clone() throws CloneNotSupportedException {
          return super.clone();
      }
  }
  ```

## 十九、Memento备忘录

### 1、概述

- 记录状态，便于回滚
- ![image-20210423152915234](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\19、备忘录模式\备忘录.png)
- 经常被用在记录快照（瞬时状态）和存盘

## 二十、TemplateMethod模板方法

### 1、概述

- 钩子函数
- ![image-20210423154346055](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\20、模板方法\模板方法.png)
- 这个模式其实一直在用
  - paint(Graphics g)
  - WindowListener
    - windowClosing()
    - windowXXX
  - ASM
    - ClassVisitor
- 凡是我们要重写一个方法，系统帮我们自动调用的都叫模板方法

### 2、演示

- 在父类中定义 op1 op2 两个方法但不实现，同时定义 m 方法，方法中规定了两个方法的实现顺序，当使用的时候写一个子类去实现父类中的方法，调用时只调用父类的 m 方法，父类会自己调用子类的实现

- ```java
  public class Main {
      public static void main(String[] args) {
          F f = new C1();
          f.m();
      }
  }
  
  abstract class F {
      public void m() {
          op1();
          op2();
      }
  
      abstract void op1();
      abstract void op2();
  }
  
  class C1 extends F {
  
      @Override
      void op1() {
          System.out.println("op1");
      }
  
      @Override
      void op2() {
          System.out.println("op2");
      }
  }
  ```

## 二十一、State状态

### 1、概述

- 根据状态决定行为
- 当类需要根据状态去改变的时候，不如将状态抽象出来，根据状态进行实现，使用时直接调用对应的 state
- ![image-20210423160343659](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\21、状态模式\状态模式.png)

### 2、演示

- 场景：一个人有高兴和伤心的状态，每种状态的反应不同

- ```java
  /**
   * 当增加新的状态时非常不方便
   */
  
  public class MM {
      String name;
      private enum MMState {HAPPY, SAD}
      MMState state;
  
      public void smile() {
          //switch case
  
      }
  
      public void cry() {
          //switch case
      }
  
      public void say() {
          //switch case
      }
  }
  ```

- 如果上述方式，添加新状态时就非常不方便，所以将状态抽象出来

- ```java
  public class MM {
      String name;
      MMState state = new MMHappyState();
  
      public void smile() {
          state.smile();
      }
  
      public void cry() {
          state.cry();
      }
  
      public void say() {
          state.say();
      }
  
  }
  ```

- ```java
  public abstract class MMState {
      abstract void smile();
      abstract void cry();
      abstract void say();
  }
  ```

- ```java
  // 高兴时的状态
  public class MMHappyState extends MMState {
      @Override
      void smile() {
          System.out.println("happy smile");
      }
  
      @Override
      void cry() {
  
      }
      @Override
      void say() {
  
      }
  }
  ```

- ```java
  public class MMNervousState extends MMState {
      @Override
      void smile() { }
  
      @Override
      void cry() { }
  
      @Override
      void say() { }
  }
  ```

- ```java
  public class MMSadState extends MMState {
      @Override
      void smile() { }
  
      @Override
      void cry() { }
  
      @Override
      void say() { }
  }
  ```

### 3、有限状态机（FSM）

- 状态之间是如何进行迁移和变化的
- 线程状态迁移图
  - ![image-20210423161527957](C:\Users\18311\OneDrive\桌面\学习\笔记抓图\设计模式\21、状态模式\线程状态迁移图.png)
  - 当 new Theread 时，就处于 new 状态
  - 当调用 start 时，它就会被操作系统的线程调度器拿去执行，这时叫做执行态
  - 执行态有两种
    - 就绪，要被执行的时候首先会被扔到一个执行队列里，CPU 不会马上运行它
    - Running 态，什么时候 CPU 选中了它，什么时候执行，这时如果调用了 Tread.yield 就又会回到 Ready 态 就绪，或者被挂起也会会带 Ready 态
  - 整个线程执行完了就会到 Teminated
  - 如果在线程执行过程中调用了 sleep 方法，sleep 一段时间，或者同步线程 wait 等待一段时间，或者 join 方法合并的一段时间，或者 LockSupport.parkNanos() 或者 LockSupport.parkUntil()，以上这些方法时会进入 TimedWaiting（等待一段时间） 状态，这段时间结束后又会回到被线程调度器执行的状态
  - 如果在线程执行过程中调用了 sleep，join，park，没有指定时间，那么会无限制的等待下去，进入 Waiting 状态 ，什么时候调用了 notify、notifyAll、unPark，它才会回到被执行态
  - 如果在线程执行过程中需要进入到 sync 同步代码块的锁，这时会进入 Blocked 阻塞态，什么时候获得锁，什么时候会回到被执行态

## 二十二、Intepreter解释器

### 1、概述

- 解释脚本语言的，基本底层开发会用
- 基本用不上

## 二十三、6大设计原则

### 1、设计模式列表

- 创建型模式
  1. Abstract Factory（*）：抽象方法模式
  2. Builder：构建器模式
  3. Factor Method（*）：工厂方法模式
  4. Prototype：原型模式
  5. Singleton（*）：单例模式
- 结构型模式
  1. Adapter：适配器模式
  2. Bridge：桥接模式
  3. Composite：组合模式
  4. Decorator：装饰器模式
  5. Facade：门面，外观模式
  6. Flweight：享元模式
  7. Proxy：代理模式
- 行为型模式
  1. Chain Of Responsibility：责任链模式
  2. Command：命令模式
  3. Interpreter：解释器模式
  4. Iterator：迭代器模式
  5. Mediator：调停者，中介模式
  6. Memento：备忘录模式
  7. Observer：观察者模式
  8. State：状态模式
  9. Strategy：策略模式
  10. Template Method：模板方法模式
  11. Visitor：访问者模式

### 2、设计模式的指导思想

1. 可维护性 Maintainbility
   - 修改功能，需要改动的地方越少，可维护性越好
2. 可复用性 Reusability
   - 代码可以被以后重复利用
3. 可扩展性 Extensibility /Scalability
   - 添加功能无需修改原来代码
4. 灵活性 flexibility / mobility / adaptability
   - 代码接口可以灵活调用

### 3、设计原则 —— 单一职责原则

- Single Responsibility Principle
- 一个类别太大，负责单一的职责
  - 比如 Person 代表一个人
  - PersonManager 负责一个人的管理
- 高内聚，低耦合

### 4、设计原则 —— 开闭原则

- Open-Closed Principle
- 对扩展开放，对修改关闭
  - 尽量修改原来代码的情况下进行扩展
- 抽象化，多态是开闭原则的关键

### 5、设计原则 —— 里氏替换原则

- Liscov Substitution Principle
- 所有使用父类的地方，必须能够透明的使用子类对象
  - F  a = new F();  也可以 new 其他子类，其他代码不变

### 6、设计原则 —— 依赖倒置原则

- Dependency Inversion Principle
- 依赖抽象，而不是依赖具体
- 面向抽象、接口编程
  - 因为里面的具体实现可以随便换

### 7、设计原则 —— 接口隔离原则

- Interface Segregation Principle
- 每一个接口应该承担独立的角色，不干不该自己干的事
  - Flyable（能飞的） Runnable（能跑的） 不该合二为一
  - 避免子类实现不需要实现的方法
  - 需要对客户提供接口时，只需要暴露最小的接口

### 8、设计原则 —— 迪米特法则

- Law Of Demeter
- 尽量不要和陌生人说话
- 在迪米特法则中，对于一个对象，非陌生人包括以下几类
  - 当前对象本身（this）
  - 以参数形式传入到当前对象方法中的对象
  - 当前对象的成员对象
  - 如果当前对象的成员对象是一个集合，那么集合中的元素也都是朋友
  - 当前对象所创建的对象
- 和其它类的耦合度变低

### 9、总结

- OCP：总纲，对扩展开放，对修改关闭
- SRP：类的职责要单一
- LSP：子类可以透明替换父类
- DIP：面向接口编程
- ISP：接口的职责要单一
- LOD：降低耦合

