---
layout: posts
title: Java线程池的正确使用姿势
date: 2020-04-13 16:18:58
tags: [线程池, JUC]
categories: 高并发
---



Executor框架是从JKD1.5之后开始引入，位于java.util.concurrent包下，其内部使用了线程池机制，控制线程的创建、执行和关闭，简化了并发变成的操作。

![img](https://img-blog.csdnimg.cn/20190921152211773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5nbGlmdTUyMA==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

<!-- more -->

Executor

Executor是一个接口，其定义了一个接收Runnable对象的方法executor，其方法签名为executor(Runnable command),该方法接收一个Runable实例，它用来执行一个任务，任务即一个实现了Runnable接口的类，一般来说，Runnable任务开辟在新线程中的使用方法为：new Thread(new RunnableTask())).start()，但在Executor中，可以使用Executor而不用显示地创建线程：executor.execute(new RunnableTask()); // 异步执行

ExecutorService

ExecutorService是一个使用比Executor更加广泛的子类接口，提供了对生命周期的管理方法，可跟踪一个或者多个异步任务的执行状况返回Future。可调用本身的shutdown()来平滑关闭ExecutorService(拒绝接受新的任务，已提交的任务会继续执行完毕)。

通过ExecutorService.submit()方法，可以得到Future,可以通过future.isDone()来判断当前任务是否已执行完成;也可以用future.get()来直接获取执行完成返回的结果。

Executors

Executors提供了一系列的工厂方法来创建线程池，返回的线程池都实现了ExecutorService接口。

- public static ExecutorService newFixedThreadPool(int Threads) 创建固定数目线程的线程池。

- public static ExecutorService newCachedThreadPool()：创建一个可缓存的线程池，调用execute 将重用以前构造的线程（如果线程可用）。如果没有可用的线程，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。

- *public static ExecutorService newSingleThreadExecutor()：*创建一个单线程化的Executor。

- *public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)*

  创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。



### 线程池的正确使用

以下是阿里编程规范的一段话：

线程池不允许使用Executors来创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加清楚线程池的运行规则，规避资源耗尽的风险。

Executors的各个方法的弊端：

newFixedThreadPool和newSingleThreadExecutor:

  堆积的处理请求队列可能会耗费非常大的内存，甚至造成OOM

newCachedThreadPool和newScheduledThreadPool:

 线程的最大数是Integer.MAX_VALUE,可能会创建数量非常大的线程，甚至OOM



正确的使用方式：

手动创建线程池

```java
public ThreadPoolExecutor(int paramInt1, int paramInt2, long paramLong, TimeUnit paramTimeUnit,
            BlockingQueue<Runnable> paramBlockingQueue, ThreadFactory paramThreadFactory,
            RejectedExecutionHandler paramRejectedExecutionHandler) {
        this.ctl = new AtomicInteger(ctlOf(-536870912, 0));
        this.mainLock = new ReentrantLock();
        this.workers = new HashSet();
        this.termination = this.mainLock.newCondition();
        if ((paramInt1 < 0) || (paramInt2 <= 0) || (paramInt2 < paramInt1) || (paramLong < 0L))
            throw new IllegalArgumentException();
        if ((paramBlockingQueue == null) || (paramThreadFactory == null) || (paramRejectedExecutionHandler == null))
            throw new NullPointerException();
        this.corePoolSize = paramInt1;
        this.maximumPoolSize = paramInt2;
        this.workQueue = paramBlockingQueue;
        this.keepAliveTime = paramTimeUnit.toNanos(paramLong);
        this.threadFactory = paramThreadFactory;
        this.handler = paramRejectedExecutionHandler;
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

- corePoolSize :线程池的核心池大小，在创建线程池之后，线程池默认没有任何线程。当有任务过来的时候才会去创建创建线程执行任务。换个说法，线程池创建之后，线程池中的线程数为0，当任务过来就会创建一个线程去执行，直到线程数达到corePoolSize 之后，就会被到达的任务放在队列中。（注意是到达的任务）。换句更精炼的话：corePoolSize 表示允许线程池中允许同时运行的最大线程数。如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。
- maximumPoolSize :线程池允许的最大线程数，他表示最大能创建多少个线程。maximumPoolSize肯定是大于等于corePoolSize。
- keepAliveTime :表示线程没有任务时最多保持多久然后停止。默认情况下，只有线程池中线程数大于corePoolSize 时，keepAliveTime 才会起作用。换句话说，当线程池中的线程数大于corePoolSize，并且一个线程空闲时间达到了keepAliveTime，那么就是shutdown。
- Unit:keepAliveTime 的单位。
- workQueue ：一个阻塞队列，用来存储等待执行的任务，当线程池中的线程数超过它的corePoolSize的时候，线程会进入阻塞队列进行阻塞等待。通过workQueue，线程池实现了阻塞功能。
- threadFactory ：线程工厂，用来创建线程。
- handler :表示当拒绝处理任务时的策略。



### 手动创建线程池应注意点

1.**任务独立。**如何任务依赖于其他任务，那么可能产生死锁。例如某个任务等待另一个任务的返回值或执行结果，那么除非线程池足够大，否则将发生线程饥饿死锁。

2.**合理配置阻塞时间过长的任务。**如果任务阻塞时间过长，那么即使不出现死锁，线程池的性能也会变得很糟糕。在Java并发包里可阻塞方法都同时定义了限时方式和不限时方式。例如

Thread.join,BlockingQueue.put,CountDownLatch.await等，如果任务超时，则标识任务失败，然后中止任务或者将任务放回队列以便随后执行，这样，无论任务的最终结果是否成功，这种办法都能够保证任务总能继续执行下去。

3.**设置合理的线程池大小。**只需要避免过大或者过小的情况即可，上文的公式**线程池大小=NCPU \*UCPU(1+W/C)**。

4.**选择合适的阻塞队列。**newFixedThreadPool和newSingleThreadExecutor都使用了无界的阻塞队列，无界阻塞队列会有消耗很大的内存，如果使用了有界阻塞队列，它会规避内存占用过大的问题，但是当任务填满有界阻塞队列，新的任务该怎么办？在使用有界队列是，需要选择合适的拒绝策略，队列的大小和线程池的大小必须一起调节。对于非常大的或者无界的线程池，可以使用SynchronousQueue来避免任务排队，以直接将任务从生产者提交到工作者线程。



### 线程池大小设定

**线程池的大小决定着系统的性能，过大或者过小的线程池数量都无法发挥最优的系统性能。**

当然线程池的大小也不需要做的太过于精确，只需要避免过大和过小的情况。一般来说，确定线程池的大小需要考虑CPU的数量，内存大小，任务是计算密集型还是IO密集型等因素

**NCPU = CPU的数量**

**UCPU = 期望对CPU的使用率 0 ≤ UCPU ≤ 1**

**W/C = 等待时间与计算时间的比率**

**如果希望处理器达到理想的使用率，那么线程池的最优大小为：**

**线程池大小=NCPU \*UCPU(1+W/C)**

在Java中使用

```java
int ncpus = Runtime.getRuntime().availableProcessors();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

获取CPU的数量。





### 拒绝策略

> 默认采用的是丢弃策略AbortPolicy

- AbortPolicy:丢弃任务并抛出RejectedExecutionException

- CallerRunsPolicy：只要线程池未关闭，该策略直接在调用者线程中，运行当前被丢弃的任务。显然这样做不会真的丢弃任务，但是，任务提交线程的性能极有可能会急剧下降。

- DiscardOldestPolicy：丢弃队列中最老的一个请求，也就是即将被执行的一个任务，并尝试再次提交当前任务。

- DiscardPolicy：丢弃任务，不做任何处理。



如果当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中，若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试创建新的线程去执行这个任务；如果当前线程池中的线程数目达到maximumPoolSize，则会采取任务**拒绝策略**进行处理；