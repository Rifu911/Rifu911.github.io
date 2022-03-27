---
layout: posts
title: Interview
date: 2021-04-17 22:43:00
tags: [面试]
categories: 面试篇
---

## 计算机基础

#### 1.软件设计的六大原则

* **单一职责原则**：不要存在多于一个导致类变更的原因。通俗的说，即一个类只负责一项职责。
* **里氏替换原则**：子类可以扩展父类的功能，但不能改变父类原有的功能。
* **依赖倒置原则**：高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。
* **接口隔离原则**：客户端不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上。
* **迪米特法则**：一个对象应该对其他对象保持最少的了解。<!--more-->

#### 2.设计模式

* **单例模式**：懒汉模式，饿汉模式
* **工厂模式**：
* **代理模式**：Proxy
* **委派模式**：JDBC的驱动加载，启动类加载器委派应用类加载器来加载对应的class文件
* **策略模式**：eg:操作符号->加、减、乘、除；一个接口有多个实现，一个抽象类有多个实现
* **适配器模式**：比如vga-html接头，实现两个接口，使得适配器同时拥有这两个接口的方法





## Redis

#### 1.Redis的应用场景

缓存、简单消息队列、分布式锁、共享session。

#### 2.Redis支持的数据类型

* String  	支持最大512M

* List

* Set

* ZSet    底层的数据结构是跳表，增删查都达到了预期效果。

* Hash

#### 3.Redis的持久化

* RDB(默认)   快照的方式，fork一个子进程定期将数据转换为占用空间小的快照（例如每十分钟）,dump.rdb文件，使用的具体指令是bgsave，恢复数据快
* AOF   将执行过的指令都存到一个aof文件里面，如果指令过多，恢复数据的效率很慢，但是顶多损坏一秒钟的数据

Best：一般是使用这两种持久化方式结合，优先采用aof进行数据恢复，然后再使用rdb

#### 4.数据过期策略和内存淘汰机制

> Redis是使用**定期删除**和**惰性删除**两者配合的过期策略。

定期删除：默认每个100ms就随机抽取设置了过期时间的key，判断是否过期，过期就删除。因为key太多，全盘扫描会耗费很多时间，所以是随机抽取，但是这样的策略不够完善，需要惰性删除来配合。

惰性删除：客户端获取某个key，redis会检测是否过期，是，则删除并且不会返回给客户端。

由于以上两种策略都无法解决redis使用内存过高问题，所以有了下面的内存淘汰机制。

* noeviction：当内存不足以容纳新写入数据时，新写入操作会报错。**默认策略**
* allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。

* allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。

* volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。

* volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。

* volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。

#### 5.Redis哈希槽的概念

Redis集群没有使用一致性哈希，而是引入了hash槽的概念，Redis集群有16384个哈希槽，每个key经过CRC16校验后对16384进行取模，来决定放置到哪个槽里面，集群的每个节点都会负责部分槽。

#### 6.Redis支持事务吗

Redis支持部分事务，但是不满足像mysql数据库定义的事务的四大特性ACID，不满足的部分是强一致性。

相关的指令：

* MULTI：开启一个事务，MULTI 执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中。
* EXEC：执行队列中所有的命令。
* DISCARD：清空事务队列,并放弃执行事务。
* UNWATCH：取消 WATCH 命令对所有 key 的监视。
* WATCH key1 key2 ...	：监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

#### 7.如何保证 redis 中的数据都是热点数据

这个其实考验的是Redis的6种内存淘汰机制，我们可以将内存淘汰机制设置为allkeys-lru或者volatile-lru。

具体的操作是到redis的配置文件里面修改以下属性：

> maxmemory-policy volatile-lru

#### 8.keys和scan指令

使用keys 和scan可以扫出指定模式的key。

但是由于Redis是单线程的，keys指令会导致线程阻塞一段时间，如果是线上环境，建议使用scan指令，scan指令可以无阻塞的取出符合条件的key，但是会有一定的重复概率，需要客户端再进行一次去重，效率会比keys慢。

#### 9.使用Redis的分布式锁（基于Redission，看门狗的使用）

具体指令就是SETNX(SET IF NOT EXIST)：尝试给某个key赋值，如果key不存在则赋值成功，并返回1，如果赋值不成功则返回0；设置成功后会给这个key设置过期时间。

看门狗的作用：有可能业务时间大于-锁的过期时间，这时候就会导致事务无法提交；看门狗能给占有的锁自动续期，保证业务完成之后能够正常释放持有的Redis锁。

#### 10.为什么Redis是单线程的，但是又能支持高并发

因为Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，多线程会增加一个上下文切换的消耗和锁的竞争条件，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了。

**总结**：

* 操作的数据都是在内存里面
* 单线程，没有上下文的切换消耗资源
* NIO多路复用机制

#### 11.如何保证Redis和数据库的双写一致性

* 懒加载模式缓存可采取双删+TTL失效来实现；
* 双删失败情况下可采取重试措施，重试有业务通过mq重试以及组件消费mysql的binlog再写入mq重试两种方式；
* 主动加载由于操作本身不具有幂等性，所以需要考虑加载的有序性问题，采取mq的分区机制实现串行化处理，实现缓存和mysql数据的最终一致，此时读和写操作的缓存加载事件是走的同一个mq。



## Java基础

#### 1.synchronzied（jdk1.6以后）

> 锁粗化的过程是不可逆的

偏向锁->CAS->自旋锁->轻量级锁->重量级锁

接下来谈下锁升级的过程。

默认使用的锁是Mark Word。

线程来请求资源锁，会先判断当前资源的是否有锁，有锁的话回去判断锁的对象头里面存的线程id是否为自己，是的话可直接获得当前锁（偏向锁的体现）。如果不是自己，则会进行一次CAS，判断锁标记的线程是否还存活，如果线程已经结束，则直接改线程id修改为自己，如果线程是存活，则会继续进行CAS，重复上面步骤，此时晋升为轻量级锁，在一定的自旋失败后(默认10次)，会进一步变成重量级锁，会将自己挂起。

<img src="https://image-static.segmentfault.com/237/957/2379573156-e0be89c7e6213336" alt="img" style="zoom:200%;" />



#### 2.volatile

* 线程中读取的时候，每次读取都会去主内存中读取共享变量最新的值，然后将其复制到工作内存

* 线程中修改了工作内存中变量的副本，修改之后会立即刷新到主内存

Extra：volatile是如何实现内存的可见性？

> 通过加入内存屏障和禁止重排优化来实现的。
>
> 在每个volatile写操作前插入StoreStore屏障，在写操作后插入StoreLoad屏障
> 在每个volatile读操作前插入LoadLoad屏障，在读操作后插入LoadStore屏障

## Java高并发

#### 1.必须要清楚的几个概念

* **同步（Synchronous）和异步（Asynchronous）**：同步方法一旦开始调用，调用者必须要等方法放回结果才能继续执行；异步更像是一个消息传递，一旦开始调用，立即返回，调用者继续后续的操作。
* **并发（Concurrency）和并行（Parallelism）**：并发侧重于线程的上下文切换；而并行是多个线程一起执行。
* **临界区**：临界区用来表示一种公共资源或者说共享数据，可以被多个线程使用，但是每一次只能有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源就必须等待。
* **阻塞（Blocking）和非阻塞（Non-Blocking）**：阻塞指的是多个线程请求同一个资源（临界区），但是某个时刻只能有一个线程享用，其他线程只能等待。非阻塞的意思和其相反，它强调的是多个线程之间不会互相影响，都可以正常执行。
* **死锁（DeadLock）、饥饿（Stravation）和活锁（LiveLock）**：死锁就不再阐述；饥饿优先级低的线程总是被优先级高的线程插队，导致很长时间拿不到资源锁，处于饥饿的状态；活锁指的是秉承“谦让”的原则，互相让出自己所占有的锁，导致资源都是在两个线程之前切换，而没有一个线程可以拿到所有的资源正常执行。

#### 2.ReentrantLock

* ReentrantLock可以实现公平锁和非公平锁。
* ReentrantLock默认实现的是非公平锁。
* ReentrantLock的获取锁和释放锁必须成对出现，锁了几次，也要释放几次。
* 释放锁的操作必须放在finally中执行。
* lockInterruptibly()实例方法可以相应线程的中断方法，调用线程的interrupt()方法时，lockInterruptibly()方法会触发 `InterruptedException`异常。
* 关于 `InterruptedException`异常说一下，看到方法声明上带有 `throwsInterruptedException`，表示该方法可以响应线程中断，调用线程的interrupt()方法时，这些方法会触发 `InterruptedException`异常，触发InterruptedException时，线程的中断中断状态会被清除。所以如果程序由于调用 `interrupt()`方法而触发 `InterruptedException`异常，线程的标志由默认的false变为ture，然后又变为false。
* 实例方法tryLock()获会尝试获取锁，会立即返回，返回值表示是否获取成功。
* 实例方法tryLock(long timeout, TimeUnit unit)会在指定的时间内尝试获取锁，指定的时间内是否能够获取锁，都会返回，返回值表示是否获取锁成功，该方法会响应线程的中断。

| 获取锁的方法                         | 是否立即响应(不会阻塞) | 是否响应中断 |
| :----------------------------------- | :--------------------- | :----------- |
| lock()                               | ×                      | ×            |
| lockInterruptibly()                  | ×                      | √            |
| tryLock()                            | √                      | ×            |
| tryLock(long timeout, TimeUnit unit) | ×                      | √            |

#### 3.Condition

* 使用condition的步骤：创建condition对象，获取锁，然后调用condition的方法
* 一个ReentrantLock支持床多个condition对象
* **void await() throws InterruptedException;**方法会释放锁，让当前线程等待，支持唤醒，支持线程中断
* **void awaitUninterruptibly()**;方法会释放锁，让当前线程等待，支持唤醒，不支持线程中断
* **long awaitNanos(long nanosTimeout) throws InterruptedException**;参数为纳秒，此方法会释放锁，让当前线程等待，支持唤醒，支持中断。超时之后返回的，结果为负数；超时之前被唤醒返回的，结果为正数（表示返回时距离超时时间相差的纳秒数）
* **boolean await(longtime,TimeUnit unit) throws InterruptedException**;方法会释放锁，让当前线程等待，支持唤醒，支持中断。超时之后返回的，结果为false；超时之前被唤醒返回的，结果为true
* **boolean awaitUntil(Date deadline)throws InterruptedException**;参数表示超时的截止时间点，方法会释放锁，让当前线程等待，支持唤醒，支持中断。超时之后返回的，结果为false；超时之前被唤醒返回的，结果为true
* **void signal()**;会唤醒一个等待中的线程，然后被唤醒的线程会被加入同步队列，去尝试获取锁
* **void signalAll()**;会唤醒所有等待中的线程，将所有等待中的线程加入同步队列，然后去尝试获取锁

| 对比项                                     | Object 监视器方法           | Condition                                                    |
| :----------------------------------------- | :-------------------------- | :----------------------------------------------------------- |
| 前置条件                                   | 获取对象的锁                | 调用Lock.lock获取锁，调用Lock.newCondition()获取Condition对象 |
| 调用方式                                   | 直接调用，如：object.wait() | 直接调用，如：condition.await()                              |
| 等待队列个数                               | 一个                        | 多个，使用多个condition实现                                  |
| 当前线程释放锁并进入等待状态               | 支持                        | 支持                                                         |
| 当前线程释放锁进入等待状态中不响应中断     | 不支持                      | 支持                                                         |
| 当前线程释放锁并进入超时等待状态           | 支持                        | 支持                                                         |
| 当前线程释放锁并进入等待状态到将来某个时间 | 不支持                      | 支持                                                         |
| 唤醒等待队列中的一个线程                   | 支持                        | 支持                                                         |
| 唤醒等待队列中的全部线程                   | 支持                        | 支持                                                         |

#### 4.LockSupport

**LockSupport**位于**java.util.concurrent**包中，是juc中一个基础类。

目前为止的3种让线程等待和唤醒的方式：

1.Object的wait和notify,notifyAll

2.Condition的await和signal,signalAll

3.LockSupport的park和unpark

|                                            | Object                   | Condtion           | LockSupport |
| :----------------------------------------- | :----------------------- | :----------------- | ----------- |
| 前置条件                                   | 需要在synchronized中运行 | 需要先获取Lock的锁 | 无          |
| 无限等待                                   | 支持                     | 支持               | 支持        |
| 超时等待                                   | 支持                     | 支持               | 支持        |
| 等待到将来某个时间返回                     | 不支持                   | 支持               | 支持        |
| 等待状态中释放锁                           | 会释放                   | 会释放             | 不会释放    |
| **唤醒方法先于等待方法执行，能否唤醒线程** | 否                       | 否                 | 可以        |
| 是否能响应线程中断                         | 是                       | 是                 | 是          |
| 线程中断是否会清除中断标志                 | 是                       | 是                 | 否          |
| 是否支持等待状态中不响应中断               | 不支持                   | 支持               | 不支持      |

#### 5.Semaphore

常用场景：**限流**。

* **Semaphore(int permits)**：构造方法，参数表示许可证数量，用来创建信号量
* **Semaphore(int permits,boolean fair)**：构造方法，当fair等于true时，创建具有给定许可数的计数信号量并设置为公平信号量
* **void acquire() throws InterruptedException**：从此信号量获取1个许可前线程将一直阻塞，相当于一辆车占了一个车位，此方法会响应线程中断，表示调用线程的interrupt方法，会使该方法抛出InterruptedException异常
* **void acquire(int permits) throws InterruptedException** ：和acquire()方法类似，参数表示需要获取许可的数量；比如一个大卡车要入停车场，由于车比较大，需要申请3个车位才可以停放
* **void acquireUninterruptibly(int permits)** ：和acquire(int permits) 方法类似，只是不会响应线程中断
* **boolean tryAcquire()**：尝试获取1个许可，不管是否能够获取成功，都立即返回，true表示获取成功，false表示获取失败
* **boolean tryAcquire(int permits)**：和tryAcquire()，表示尝试获取permits个许可
* **boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException**：尝试在指定的时间内获取1个许可，获取成功返回true，指定的时间过后还是无法获取许可，返回false
* **boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException**：和tryAcquire(long timeout, TimeUnit unit)类似，多了一个permits参数，表示尝试获取permits个许可
* **void release()**：释放一个许可，将其返回给信号量，相当于车从停车场出去时将钥匙归还给门卫
* **void release(int n)**：释放n个许可
* **int availablePermits()**：当前可用的许可数

#### 6.CountDownLatch（闭锁）

常用场景：等待所有线程一起到了再执行，或者等所有线程执行完了给回调。

常用方法：

* **public CountDownLatch(int count)**：构造方法，count表示计数器的值，不能小于0，否者会报异常。

* **public void await() throws InterruptedException**：调用await()会让当前线程等待，直到计数器为0的时候，方法才会返回，此方法会响应线程中断操作。

* **public boolean await(long timeout, TimeUnit unit) throws InterruptedException**：限时等待，在超时之前，计数器变为了0，方法返回true，否者直到超时，返回false，此方法会响应线程中断操作。

* **public void countDown()**：让计数器减1

  CountDownLatch使用步骤：

  1. 创建CountDownLatch对象
  2. 调用其实例方法 `await()`，让当前线程等待
  3. 调用 `countDown()`方法，让计数器减1
  4. 当计数器变为0的时候， `await()`方法会返回

#### 7.CyclicBarrier（循环栅栏/循环屏障）

常用场景：让一批线程等待然后一起执行。

它和CountDownLatch非常相似，不过CountDownLatch是让一批线程等待另一批线程执行完后再执行，而CyclicBarrier是使等待的线程达到一定数目后执行。

常用方法：

* public CyclicBarrier(int parties)：构造方法，指定等待线程的数目。

* public void await() throws InterruptedException,BrokenBarrierException：调用await()让当前线程等待，如果有线程响应中断，则会抛出BrokenBarrierException，破坏当前的规则，然后会调用notifyAll()，唤醒之前所有在等待的线程。
* public void await(long timeout, TimeUnit unit) throws InterruptedException,BrokenBarrierException,TimeoutException：某个线程只等待timeout时间，超过之后await会触发TimeoutException，然后线程继续往下执行，其他等待中的线程，新到的线程会在await方法直接抛出BrokenBarrierException，然后继续执行。
* public void reset()：重置规则，新来的线程要继续等待（前提是之前有线程响应了中断抛出了BrokenBarrierException），达到一定数目后一起执行。

#### 8.Unsafe

Unsafe是位于sum.misc下的一个类，主要提供一些执行低级别、不安全的操作方法。

常用场景：**内存操作**，**CAS操作**，**Class相关**，**对象操作**，**线程调度**，**系统信息获取**，**内存屏障**，**数组操作**

其中我们常说的AtomicInteger这些原子操作类，底层都是使用了Unsafe的cas操作api；LockSupport则是用了Unsafe的park和unpark方法。

#### 9.JUC中的原子类

原子类底层对数据的修改是用了CAS，但是CAS有可能产生ABA问题，一般我们的做法是给数据添加一个版本号，而每次修改都会讲版本号version+1，JUC也考虑到了这个问题，为我们提供了AtomicStampedReference。

public boolean compareAndSet(V   expectedReference,V   newReference,int expectedStamp, int newStamp)

#### 10.ThreadLocal

定义一个ThreadLocal.ThreadLocalMap，里面存着键值对，键是当前的线程id，值可以是基础类型的包装或者对象。（存储的位置在主内存）

#### 11.Queue

| 操作类型                           | 抛出异常               | 返回特殊值                    |
| :--------------------------------- | :--------------------- | :---------------------------- |
| 插入                               | `add(e)`               | `offer(e)`-返回（true/false） |
| 移除                               | `remove()`为空抛出异常 | `poll()`为空返回null          |
| 检查（返回队列头部元素，但不移除） | `element()`            | `peek()`                      |

#### 12.BlockingQueue

1. 队列满的情况下，调用put方法会导致当前线程阻塞
2. 队列为空的情况下，调用take方法会导致当前线程阻塞
3. `offer(e,timeuout,unit)`，超时之前，插入成功返回true，否者返回false
4. `poll(timeout,unit)`，超时之前，获取到头部元素并将其移除，返回true，否者返回false

| 操作类型 | 抛出异常    | 返回特殊值 | 一直阻塞 | 超时退出               |
| :------- | :---------- | :--------- | :------- | :--------------------- |
| 插入     | `add(e)`    | `offer(e)` | put(e)   | offer(e,timeuout,unit) |
| 移除     | `remove()`  | `poll()`   | take()   | poll(timeout,unit)     |
| 检查     | `element()` | `peek()`   | 不支持   | 不支持                 |

#### 13.可供选择的各种阻塞队列

* **ArrayBlockingQueue**

基于数组的阻塞队列实现，其内部维护一个定长的数组，用于存储队列元素。线程阻塞的实现是通过ReentrantLock来完成的，数据的插入与取出共用同一个锁，因此ArrayBlockingQueue并不能实现生产、消费同时进行。而且在创建ArrayBlockingQueue时，我们还可以控制对象的内部锁是否采用公平锁，默认采用非公平锁。

* **LinkedBlockingQueue**

基于单向链表的阻塞队列实现，在初始化LinkedBlockingQueue的时候可以指定大小，也可以不指定，默认类似一个无限大小的容量（Integer.MAX_VALUE），不指队列容量大小也是会有风险的，一旦数据生产速度大于消费速度，系统内存将有可能被消耗殆尽，因此要谨慎操作。另外LinkedBlockingQueue中用于阻塞生产者、消费者的锁是两个（锁分离），因此生产与消费是可以同时进行的。

* **PriorityBlockingQueue**

一个支持优先级排序的无界阻塞队列，进入队列的元素会按照优先级进行排序。

优先级队列放入元素的时候，会进行排序，所以我们需要指定排序规则，有2种方式：

1. 创建`PriorityBlockingQueue`指定比较器`Comparator`
2. 放入的元素需要实现`Comparable`接口

上面2种方式必须选一个，如果2个都有，则走第一个规则排序。

* **SynchronousQueue**

同步阻塞队列，SynchronousQueue没有容量，与其他BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue，每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然。

* **DelayQueue**

DelayQueue是一个支持延时获取元素的无界阻塞队列，里面的元素全部都是“可延期”的元素，列头的元素是最先“到期”的元素，如果队列里面没有元素到期，是不能从列头获取元素的，哪怕有元素也不行，也就是说只有在延迟期到时才能够从队列中取元素。

* **LinkedTransferQueue**

LinkedTransferQueue是基于链表的FIFO无界阻塞队列，它出现在JDK7中，Doug Lea 大神说LinkedTransferQueue是一个聪明的队列，它是ConcurrentLinkedQueue、SynchronousQueue(公平模式下)、无界的LinkedBlockingQueues等的超集，`LinkedTransferQueue`包含了`ConcurrentLinkedQueue、SynchronousQueue、LinkedBlockingQueues`三种队列的功能。

#### 14.高并发限流

如果是微服务的话，可借助豪猪哥**Hystrix**或者阿里巴巴的**Sentinel**来进行流控。

如果是单体应用可以考虑使用**漏桶算法、令牌桶算法**。

* **Redis**：给接口级别或者ip设置对应kv,并设置过期时间，每次请求+1，达到限流的效果。

* **漏桶算法**：水（请求）先进入到漏桶里，漏桶以一定的速度出水，当水流入速度过大会直接溢出，可以看出漏桶算法能强行限制数据的传输速率。
* **令牌桶算法**：系统以恒定的速率产生令牌，然后把令牌放到令牌桶中，令牌桶有一个容量，当令牌桶满了的时候，再向其中放令牌，那么多余的令牌会被丢弃；当想要处理一个请求的时候，需要从令牌桶中取出一个令牌，如果此时令牌桶中没有令牌，那么则拒绝该请求。

## JVM

#### 1.JDK动态代理和CGlib的区别

首先这两个都是动态代理，静态代理只有一个就是使用aspectj对java对象在编译时期就进行静态织入；而jdk动态代理是通过实现InvocationHandler，再使用proxy来调用其实现类的方法来进行反射代理，cglib是通过asm框架在对象加载时期，对其字节码进行修改生成子类来实现。

**编译**：.java文件==>.class文件

**加载**：字节码文件通过加载，验证，准备，解析和初始化阶段（验证，准备，解析又统称作链接），被加载到jvm内存。

**java动态代理：** 利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

**cglib动态代理：** 利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

**JDK动态代理和CGLIB字节码生成的区别？**

1. JDK动态代理只能对实现了接口的类生成代理，而不能针对类
2. CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法 因为是继承，所以该类或方法不要声明成final



#### 2.JVM的内存模型（JMM）

> 只是一种规范，并不是实际实现。

堆：

方法区：

虚拟机栈：

本地方法栈：

程序计数器：

#### 3.垃圾回收机制

强引用：即使爆出OOM也不会被回收。

软引用：内存不够就会被回收。

弱引用：每次GC都会被回收。

虚引用：可以用来跟踪GC，对象准备被回收时发现他还有虚引用，会把这个虚引用加入一个引用队列，可以观察这个队列中虚引用是否存在来判断对象是否被回收了。

#### 4.垃圾回收算法

引用计数法：每有一个对该对象的引用，则引用加1，缺点：如果两个对象互相引用，将导致无法回收。

GCRoots：根可达法

#### 5.垃圾回收器

<table><thead><tr><th style="width:284px;">垃圾回收器</th>
			<th style="width:685px;">特性</th>
		</tr></thead><tbody><tr><td style="width:284px;">串行（Serial）回收器</td>
			<td style="width:685px;">单线程的一个回收器，简单、易实现、效率高。</td>
		</tr><tr><td style="width:284px;">并行（ParNew）回收器</td>
			<td style="width:685px;">Serial的多线程版，可以充分的利用CPU资源，减少回收的时间。</td>
		</tr><tr><td style="width:284px;">吞吐量优先（Parallel Scavenge）回收器</td>
			<td style="width:685px;">侧重于吞吐量的控制</td>
		</tr><tr><td style="width:284px;">并发标记清除（CMS，Concurrent Mark Sweep）回收器</td>
			<td style="width:685px;">以获取最短回收停顿时间为目标的回收器，该回收器是基于“标记-清除”算法实现的。</td>
		</tr><tr><td style="width:284px;">Serial Old</td>
			<td style="width:685px;">Serial Old是Serial的老年代版本。同Serial一样， 它也是<strong>单线程</strong>收集器。用的是“<strong>标记——整理</strong>”算法。</td>
		</tr><tr><td style="width:284px;">Parallel Old</td>
			<td style="width:685px;">Parallel Old 是 Parallel Scavenge 的老年代版本，使用<strong>多线程</strong>和“<strong>标记——整理算法</strong>”。</td>
		</tr><tr><td style="width:284px;"><strong>G1（Garbage-First）</strong>收集器</td>
			<td style="width:685px;">G1 能充分利用多CPU的硬件优势，使用多个CPU来缩短“Stop The World”停顿时间，部分其他收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方式让Java程序继续执行。G1从整体来看是基于<strong>“标记-整理”</strong>算法实现的收集器，从局部（两个Region之间）上来看是基于<strong>“复制”</strong>算法实现的，这意味着G1运行期间不会产生内存空间碎片。</td>
		</tr></tbody></table>


#### 6.指针压缩

从JDK1.6开始，在64位的操作系统中，JVM支持指针压缩，启用指针压缩：-XX:+UseCompressedOops；禁止指针压缩：-XX:-UseCompressedOops

**触发条件**：在堆大小在【4-32】G的时候，会被触发；在堆内存小于4G时，不需要启用指针压缩，JVM会直接去除高32位地址，使用低虚拟地址空间。

**为什么要启用指针压缩**：首先是提高了堆内存的可配置空间（32位最大堆空间为4G）；其次在64位操作系统中使用大指针在主内存和缓存之间移动数据，占用较大带宽，GC也会承受较大的压力，启用指针压缩可以减少消耗。

**压缩的是哪些信息**：

- 对象的全局静态变量(即类属性)

- 对象头信息:64位平台下，原生对象头大小为16字节，压缩后为12字节

- 对象的引用类型:64位平台下，引用类型本身大小为8字节，压缩后为4字节

- 对象数组类型:64位平台下，数组类型本身大小为24字节，压缩后16字节

- 以下信息不会被压缩：

  - 指向非Heap的对象指针
  - 局部变量、传参、返回值、NULL指针

  

## 网络协议篇

![1594720933210](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1594720933210.png)

#### 1.TCP的三次握手和四次挥手

* 建立连接

  <img src="https://user-gold-cdn.xitu.io/2017/11/9/d8bf92c7906718271fdb8b0d2d5fe5b4?imageslim" alt="img" style="zoom: 80%;" />

* 断开连接

<img src="https://user-gold-cdn.xitu.io/2017/11/9/8c7874fafe233c9278509e40e906055c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" alt="img" style="zoom:80%;" />

问：为什么握手是三次而挥手是四次？

首先是客户端发送Fin报文告知服务端，我已发送数据完毕了，服务端收到报文后会立刻给客户端发送一个确认码，但是有可能服务端还有数据发送，所以需要等待发送完后才能发送Fin报文给客户端，当客户端收到Fin的回复，才能确认双方都发送完数据了，然后发送ack报文给服务端，至此，双方完成挥手操作。





#### 2.HTTP和HTTPS有什么区别

http监听的是22端口，HTTPS监听的是443接口，HTTPS是在http的基础上增加了一个ssl的验证。



#### 3.网络模型

* OSI  7层网络模型---------只是一个标准，并非实现。
* TCP/IP 4层网络模型----------是一个实现的应用模型，由7层模型简化而来。

![file](https://user-gold-cdn.xitu.io/2017/8/25/3fe5c91aafa1173c22b944e0ed8746c8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

TCP/IP 协议中每层技术举例：

网络访问层：ARP、RARP

互联网层：ICMP、IP

传输层：TCP、UDP

应用层：DNS、FTP、HTTP、SMTP、TELNET、IRC、WHOIS



#### 4.TCP是如何保证传输的可靠性

* 三次握手、四次挥手是保证可靠传输的前提

* 传输数据的切分为合适的数据块
* **序列号和确认应答机制**：传输时将每个报文的数据都进行了编号，这就是序列号；每次接收方收到数据都需要进行应答(就是发送ACK报文)，如果接收端通过校验序列号来判断有没有丢包，在ACK里面返回下一次需要发送的序列号。
* **超时重传**：发送过程中全体丢包或者收不到接收端发过来的ACK，选择动态计算时间来进行重传。
* **使用滑动窗口进行流量控制**
* **拥塞窗口**：不是一开始就发送大量数据，而是慢慢增大达到阈值后窗口重置为1



#### 10.聊聊SSL和TLS

![1594720668312](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1594720668312.png)

## Mysql

#### 1.数据库引擎

* MYISAM：只支持表锁，不支持事务，不支持外键。非聚集索引，使用B+Tree作为索引结构。
* INNODB：支持行锁和表锁，支持事务，支持外键。聚集索引，使用B+Tree作为索引结构。

#### 2.索引

> 索引是一种数据结构。

* 创建索引的原则
  * 最左前缀匹配原则，在索引列上进行函数操作、范围查询、使用不等于!=，或者like '%tmp'，均会导致索引失效。
  * 频繁作为查询条件的字段才需要建立索引。
  * 更新频繁，或者散列值很小（就是重复度很高）的字段不能建索引。
  * 字段值太长的不能建索引。
* 聚集索引和非聚集索引
  * 聚集索引：将数据存储与索引放到了一块，找到索引也就找到了数据。
  * 非聚集索引：将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行，myisam通过key_buffer把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据。
* 三种常用的索引结构：Hash，B Tree ，B+ Tree（最好）
  * Hash：hash索引底层就是hash表，进行查找时，调用hash函数就可以得到键值，然后进行回表查询。所以使用等值查询很快，但是无法进行范围查询。当某个键值大量重复时，会发生hash碰撞，效率很低。
  * Btree：可以存储在内部节点存储键和值。只适合进行随机检索。
  * B+Tree：内部节点只存储键，叶子结点存储值。支持随机检索和顺序检索。在使用了聚集索引的前提下，不需要再次进行回表查询。

#### 3.事务的四大特性(ACID)

* 原子性(Atomic)
* 一致性(Consitentcy)
* 隔离性(Isolation)
* 持久性(Durability)

#### 4.标准的隔离级别

* READ-UNCOMMITTED(读取未提交)： 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
* READ-COMMITTED(读取已提交)： 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
* REPEATABLE-READ(可重复读)： 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
* SERIALIZABLE(可串行化)： 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| -------- | ---- | ---------- | ---- |
| 读未提交 | 0    | 0          | 0    |
| 读已提交 | 1    | 0          | 0    |
| 可重复读 | 1    | 1          | 0    |
| 可串行化 | 1    | 1          | 1    |

> Mysql 默认采用的 REPEATABLE_READ隔离级别 
>
> Oracle 默认采用的 READ_COMMITTED隔离级别

#### 5.Mysql支持的锁

> https://www.ibm.com/developerworks/cn/opensource/os-mysql-transaction-isolation-levels-and-locks/index.html

InnoDB 实现了两种类型的行级锁：

- 共享锁

  （也称为 S 锁）：允许事务读取一行数据。

  可以使用 SQL 语句 `select * from tableName where … lock in share mode;` 手动加 S 锁。

- 独占锁

  （也称为 X 锁）：允许事务删除或更新一行数据。

  可以使用 SQL 语句 `select * from tableName where … for update`; 手动加 X 锁。

S 锁和 S 锁是**兼容**的，X 锁和其它锁都**不兼容**，举个例子，事务 T1 获取了一个行 r1 的 S 锁，另外事务 T2 可以立即获得行 r1 的 S 锁，此时 T1 和 T2 共同获得行 r1 的 S 锁，此种情况称为**锁兼容**，但是另外一个事务 T2 此时如果想获得行 r1 的 X 锁，则必须等待 T1 对行 r 锁的释放，此种情况也成为**锁冲突**。

为了实现多粒度的锁机制，InnoDB 还有两种内部使用的**意向锁**，由 InnoDB 自动添加，且都是表级别的锁。

- **意向共享锁**（IS）：事务即将给表中的各个行设置共享锁，事务给数据行加 S 锁前必须获得该表的 IS 锁。
- **意向排他锁**（IX）：事务即将给表中的各个行设置排他锁，事务给数据行加 X 锁前必须获得该表 IX 锁。

意向锁的主要目的是为了使得**行锁**和**表锁**共存。下面列出了行级锁和表级意向锁的兼容性。

| **锁类型** | **X** | **IX** | **S** | **IS** |
| :--------- | :---- | :----- | :---- | :----- |
| **X**      | 冲突  | 冲突   | 冲突  | 冲突   |
| **IX**     | 冲突  | 兼容   | 冲突  | 兼容   |
| **S**      | 冲突  | 冲突   | 兼容  | 兼容   |
| **IS**     | 冲突  | 兼容   | 兼容  | 兼容   |





## Spring

#### 1.IOC和AOP

IOC控制反转：也叫依赖注入(DI)，通过工厂模式来创建并管理Bean，并对其进行一个容器化管理。依赖注入的方式有两种，一个是构造器注入，另一种是通过调用setter方法注入。

AOP切面编程：一般用于权限控制，事务管理，日志记录等。

#### 2.IOC的初始化过程

* 首先是初始化上下文-得到一个ClassPathXmlApplicationContext或者AnnotationApplicationContext
* 再获取resourcePatternResolver对象，将resources目录下的xml,properties文件解析生Resource对象
* 利用前面得到的上下文context和resource进行初始化工厂BeanFactory，将resource解析成BeanDefinition，然后将BeanDefinition注册到BeanFactory

#### 3.Bean的创建过程

* BeanDefinition解析：调用容器中InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation()方法，可以在Bean实例化前修改BeanDefinition。
* 实例化：将指定的BeanDefinition转换成BeanWrapper，然后调用createBeanInstance创建Bean的实例。Bean的实例化有多种策略，如工厂方法、指定参数构造器和默认无参构造器等。总之，createBeanInstance()方法会返回一个刚实例化好的、属性未赋值的空Bean。
* 属性填充：调用populateBean()方法，为Bean进行属性的赋值和依赖的注入。
* 属性填充后置处理：调用容器中InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation()方法，进行属性填充后处理。
* 初始化前置处理：调用BeanPostProcessor的postProcessBeforeInitialization()方法进行初始化前置处理。
* 初始化：调用Bean的初始化方法，顺序为：InitializingBean的afterPropertiesSet() -> 注册的init-method。
* 初始化后置处理：调用BeanPostProcessor的postProcessAfterInitialization()执行初始化后置处理。
* 调用registerDisposableBeanIfNecessary()方法，注册Bean的销毁处理逻辑：将所有实现了DisposableBean接口的或者注册了destroy-method方法的Bean,封装成一个DisposableBean,将其销毁的回调注册到容器中。

#### 4.如何解决Bean循环依赖的问题

解决循环依赖是在初始化的时候进行。Spring为了解决循环依赖的问题引入了**三级缓存**

```java
/** Cache of singleton objects: bean name --> bean instance：已经实例化并且初始化的对象 */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);

/** Cache of early singleton objects: bean name --> bean instance：已经实例化的对象 */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** Cache of singleton factories: bean name --> ObjectFactory:对象的构造工厂 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
```

下面是对源码的阅读解析

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    //判断对象是否为空&bean是否正在创建中
    //isSingletonCurrentlyInCreation:当前单例是否正在创建中，就是未完成初始化
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            //allowEarlyReference:是否允许从singletonFactories中通过getObject拿到对象
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
} 
```

总：Spring首先从一级缓存singletonObjects中获取对象，获取不到并且对象正在创建中，再去二级缓存里面获取，如果还是获取不到，并且允许从从singletonFactories中通过getObject拿到对象，再去三级缓存singletonFactories里面获取，最后是一定能够获取到对象的，获取到之后，将对应的beanFactory从三级缓存移除，并将拿到的对象放入到二级缓存。

*ps：Spring不能解决使用构造器注入的循环依赖。*



## SpringMVC

#### 1.处理请求的流程

* 所有的客户端请求都会被DispatcherServlet所拦截
* DispatcherServlet通过Handler Mappings来决定这个请求交给哪个Handler进行处理（我们可以理解为Controller）
* Handler处理之后会返回一个ModelAndView对象给DispatcherServlet
* DispatcherServlet将③的结果发送给View Resolver，经过解析后得到实际的View Page
* DispatcherServlet将Model渲染到View上面，得到最后用户所看到的的页面。

![img](https://user-gold-cdn.xitu.io/2019/5/19/16ad044ca097b7d4?imageslim)



## 分布式技术

#### 1.分布式锁和分布式事务





## RabbitMQ

#### 1.基础

**使用的通信协议**：AMQP，ActiveMQ使用的是JMS

**角色**：生产者、消费者、代理

**重要的组件**：

* ConnectionFactory：应用程序和RabbitMQ建立连接的管理器
* Channel：信道，用于消息传递的通道
* Exchange：交换机（也可称作路由），用于接收和分配消息。
* Queue：消息队列。
* RoutingKey：路由键，用于指定消息发送到交换机的规则。
* BindingKey：绑定键，用于交换机绑定消息队列。

**广播的方式**：

* fanout：扇出，所有绑定到该路由上面的消息队列都能收到消息。
* direct：通过routingKey和exchange决定哪个消息队列可以接收到消息。例如(direct-test-route-key)
* topic：所有符合routingkey绑定规则的消息队列都能收到消息(例如*-test-route-key)

#### 2.如何避免消息丢失

* 发送端进行发送确认

* 消息持久化（将交换机和队列进行持久化，设置队列的durable为true）
* 消费端ACK确认机制

#### 3.如何实现延迟队列

首先必须要了解两个概念：

消息的过期时间Time-To-Live(TTL)和死信队列Dead Letter Exchanges(DLX)

```java
argument.put("x-dead-letter-exchange", "your dlx name")
channel.queueDeclare(queueName, true, false, false, agruments);
```

消息变成死信的原因有：

* 消息过期了TTL
* 消息被拒绝了，并且reQueue = false
* 队列达到最大长度

实现原理：给需要延时的消息设置过期时间，并发送到队列上，当消息过期后，会再通过DLX发送到死信队列上面，客户端通过消费死信队列的消息，通过消息中指定的消费队列，来再次进行发送。





## ElasticSearch

#### 1.概述

ElasticSearch是一款基于Luncene高效的全文搜索引擎。

* index == Mysql中的数据库，库里面可以建很多的表
* type == 数据库表
* document == 相当于一行记录
* field == 相当于字段

> PS : 在7.0版本后，去掉了type这个mapping。
>
> 具体原因可查看该文档https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html

#### 2.关于IK分词器

ElasticSearch使用的是IK分词器，对中文的分词也比较好。如果需要自定义分词效果，可以在elasticsearch目下的\plugins\ik\config下新建custom.dic，添加自定义的词汇，使用换行符分隔，最后在同等级的配置文件IKAnalyzer.cfg.xml中的<properties>添加<entry key="ext_dict">custom.dic</entry>

#### 3.倒排索引

传统的通过关键字进行全文搜索，是遍历每篇文章是否包含改关键字；而倒排索引则恰恰相反，倒排索引在数据插入的时候，会先对文章进行分词，然后给所有词汇建立索引，格式为词汇 - > 文章主键，形成词汇-文章的关系映射表，时间复杂度为O(1)。

![img](https://user-gold-cdn.xitu.io/2019/1/22/16874bae6bd24813?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

加分项：倒排索引底层是使用了FST（Finite State Tranducer）数据结构，它有两个优点：

1.空间占用小，通过对词典中单词前缀和后缀的重复利用，压缩了存储空间。

2.查询速度快。O(len(str))的查询时间复杂度。