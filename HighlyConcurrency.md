[TOC]

# 高并发常识

cpu多级缓存，基于MESI协议。

M: Modify
E:Exclusive(独享状态)

S: Share

I: Invalid

Local Read/LocalWrite Remote/Read/Remote Wirte



# JAVA 多线程

## 特点 

在多线程的情况下：

* 原子性，多线程互斥，同一时刻只能有一个线程对它进行操作；

  AtomicXX:  CAS/Unsafe.compareAndSwapInt等; 

  Volicate通过多线程进行计数看， 它并不具备原子性，但可以使用其boolean进行程序控制。

* 可见性，对所有线程可见，且同步；

  

* 有序性，

  Synchronized/Lock可保证有序性。

### 多线程下保证原子性变量的使用方法
>  AtomicInteger、AtomicLong(低并发，准确)、LongAddr(高并发时，强烈推荐使用！不一定准确，原理是分别散到各个节点)
>
>  AutomicBoolean中compareAndSet 可控制在多线程时，用于某段代码仅执行一次。 真心强悍！
>
>  AutomicReference<Integer>等等；
>
>  计算保证线程安全的，底层Java调用native本地方法，通过do/while实现，while会判断当前值是否有线程变更一致，不一致的话，直到while到一致，做累加动作。 工作中不常用到。
>
>  AutomicUpdater
>
>  AutomicStampReferencee:CAS 中的ABA问题，AutomicStampReference源码中使用stamp版本号来解决。
>
>   ImmutalbeMap
>
>  ImmutableList

### 原子性锁

 Synchronized

​	依赖JVM

Lock

​	ReentrantLock使用源码时行控制

### Happens Before

8个规则

### SpringWeb高并发时， 线程封闭

#### 1. 堆栈封闭

 没有高并发概念的时候，程序一般不会出现线程安全相关问题，为什么？

因为我们使用的局部变量，它存在堆栈中，所以高并发时一般不会出现线程安全相关的问题。所以在方法中尽量 使用性能高的结构，如StringBuilder 等；

#### 2. 线程封闭ThreadLocal

使用ThreadLocal可将指定对象放到内存，可至各阶段使用（如aop/interceptor/Filter/controller/service/dao等等）。

使用场景：如某阶段获得用户信息，只需要在登录的时候放至ThreadLocal中便可。

### Java高并发时，线程不安全问题

| 线程不安全的变量 | 替换为线程安全的 |
| ---- | ---- |
| StringBuilder | StringBuffer |
| SimpleDateForma | JodaTime中的DateTimeFormate |
| HashMap | HashTable |
|                  | Vector,Stack(推荐) |
| | Collections.synchronizedXX() 和 H ashTable |
| | |



* SimpleDateFormat若放到类中，容易造成线程安全问题，将其放至类中的具体方法才会保证线程封闭。建议推荐使用joa中的DataTimeFormatter。

* StringBuilder线程并不安全，StringBuffer是线程安全的，因为源码在定义时 +synchronized。

  ### Final

  使用方法参见springboot/thread包下的源码。

  final Integer a= 1; //线程安全

  final Map map = new HashMap(); //线程不安全，因后续还可以对其进行修改；

  特别注意：final User = new User; //等等都是线程不安全的；

  解决不安全方法有几种：

  1.  Collections.unmodifiableMap();//便是线程安全的；
  2. 使用ImmutableMap/ImmutableList等等，也是线程安全的，需要加上final进行修饰；

### 并发容器

java.util.concurrent ，简称：JUC。一般情况下，大多使用JUC来解决并发的问题。
| 线程不安全的变量 | 替换为线程安全的 |
| ---- | ---- |
| ArrayList | CopyOnWriteArrayList，源码使用Lock锁进行操作。 |
| HashSet/TreeSet | CopyOnWirteArraySet/ConcurrentSkipListSet。 |
| HashMap/TreeMap | ConcurrentHashMap（key无序）/ConcurrentSkipListMap（key有序，高并发时推荐使用）。 |
|                  |                                                |
|                  |                                                |
|                  | |

# 技术实现 

CountDownLatch

![1572394469226](pic\countDownLatch.png)