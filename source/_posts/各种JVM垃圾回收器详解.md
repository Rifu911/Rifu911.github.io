---
title: 各种JVM垃圾回收器详解
date: 2021-05-19 17:02:13
categories: 学习笔记
tags: [JVM，垃圾收集器]
---

#### G1

这个被问的太多了，先来聊这个吧。栽过...

##### 前言

> G1 GC，全称Garbage-First Garbage Collector，通过-XX:+UseG1GC参数来启用，作为体验版随着JDK 6u14版本面世，在JDK 7u4版本发行时被正式推出，相信熟悉JVM的同学们都不会对它感到陌生。在JDK 9中，G1被提议设置为默认垃圾收集器（JEP 248）。
>
> 从官网的描述中，我们知道G1是一种服务器端的垃圾收集器，应用在多处理器和大容量内存环境中，在实现高吞吐量的同时，尽可能的满足垃圾收集暂停时间的要求。它是专门针对以下应用场景设计的: 
>
> * 像CMS收集器一样，能与应用程序线程并发执行。
> * 整理空闲空间更快。
> * 需要GC停顿时间更好预测。
> * 不希望牺牲大量的吞吐性能。
> * 不需要更大的Java Heap。

<!-- more -->

##### G1中的几个概念

###### Region

传统的垃圾收集器，分为了新生代、老年代、和永久代(方法区的实现)，他们是连续的内存区域，如下图。

![内存区域](image-20210519172400375.png)

而G1的各代储存地址不是连续的，每一代都被分成了n个大小一致的region（总2048个），每个Region的内存区域是连续的，单个Region的大小可通过参数-XX:G1HeapRegionSize设定，取值范围从1M到32M，且是2的指数。如果不设定，那么G1会根据Heap大小自动决定，如下图。

![Region](image-20210519172611759.png)

###### SATB

全称是Snapshot-At-The-Beginning，字面理解就是gc开始之前的存活对象的快照。它是通过Root Tracing得到的，作用是维持并发GC的正确性。 那么它是怎么维持并发GC的正确性的呢？根据三色标记算法，我们知道对象存在三种状态： 

* 白：对象没有被标记到，标记阶段结束后，会被当做垃圾回收掉。 
* 灰：对象被标记了，但是它的field还没有被标记或标记完。
* 黑：对象被标记了，且它的所有field也被标记完了。

由于并发阶段的存在，Mutator和Garbage Collector线程同时对对象进行修改，就会出现白对象漏标的情况，这种情况发生的前提是：

* Mutator赋予一个黑对象该白对象的引用。 
* Mutator删除了所有从灰对象到该白对象的直接或者间接引用。

对于第一个条件，在并发标记阶段，如果该白对象是new出来的，并没有被灰对象持有，那么它会不会被漏标呢？Region中有两个top-at-mark-start（TAMS）指针，分别为prevTAMS和nextTAMS。在TAMS以上的对象是新分配的，这是一种隐式的标记。对于在GC时已经存在的白对象，如果它是活着的，它必然会被另一个对象引用，即条件二中的灰对象。如果灰对象到白对象的直接引用或者间接引用被替换了，或者删除了，白对象就会被漏标，从而导致被回收掉，这是非常严重的错误，所以SATB破坏了第二个条件。也就是说，一个对象的引用被替换时，可以通过write barrier 将旧引用记录下来。



###### RSet

全称是Remembered Set，是辅助GC过程的一种结构，典型的空间换时间工具，和Card Table有些类似。还有一种数据结构也是辅助GC的：Collection Set（CSet），它记录了GC要收集的Region集合，集合里的Region可以是任意年代的。在GC的时候，对于old->young和old->old的跨代对象引用，只要扫描对应的CSet中的RSet即可。 逻辑上说每个Region都有一个RSet，RSet记录了其他Region中的对象引用本Region中对象的关系，属于points-into结构（谁引用了我的对象）。而Card Table则是一种points-out（我引用了谁的对象）的结构，每个Card 覆盖一定范围的Heap（一般为512Bytes）。G1的RSet是在Card Table的基础上实现的：每个Region会记录下别的Region有指向自己的指针，并标记这些指针分别在哪些Card的范围内。 这个RSet其实是一个Hash Table，Key是别的Region的起始地址，Value是一个集合，里面的元素是Card Table的Index。

 RSet究竟是怎么辅助GC的呢？在做YGC的时候，只需要选定young generation region的RSet作为根集，这些RSet记录了old->young的跨代引用，避免了扫描整个old generation。 而mixed gc的时候，old generation中记录了old->old的RSet，young->old的引用由扫描全部young generation region得到，这样也不用扫描全部old generation region。所以RSet的引入大大减少了GC的工作量。

##### G1 的GC模式

G1提供了两种GC模式，Young GC和Mixed GC，两种都是完全Stop The World的。 

* Young GC：选定所有年轻代里的Region。通过控制年轻代的region个数，即年轻代内存大小，来控制young GC的时间开销。 
* Mixed GC：选定所有年轻代里的Region，外加根据global concurrent marking统计得出收集收益高的若干老年代Region。在用户指定的开销目标范围内尽可能选择收益高的老年代Region。

由上面的描述可知，Mixed GC不是full GC，它只能回收部分老年代的Region，如果mixed GC实在无法跟上程序分配内存的速度，导致老年代填满无法继续进行Mixed GC，就会使用serial old GC（full GC）来收集整个GC heap。所以我们可以知道，G1是不提供full GC的。

上文中，多次提到了global concurrent marking，它的执行过程类似CMS，但是不同的是，在G1 GC中，它主要是为Mixed GC提供标记服务的，并不是一次GC过程的一个必须环节。global concurrent marking的执行过程分为四个步骤：

* 初始标记（initial mark，STW）。它标记了从GC Root开始直接可达的对象。
* 并发标记（Concurrent Marking）。这个阶段从GC Root开始对heap中的对象标记，标记线程与应用程序线程并行执行，并且收集各个Region的存活对象信息。
* 最终标记（Remark，STW）。标记那些在并发标记阶段发生变化的对象，将被回收。 
* 清除垃圾（Cleanup）。清除空Region（没有存活对象的），加入到free list。

第一阶段initial mark是共用了Young GC的暂停，这是因为他们可以复用root scan操作，所以可以说global concurrent marking是伴随Young GC而发生的。第四阶段Cleanup只是回收了没有存活对象的Region，所以它并不需要STW。

Young GC发生的时机大家都知道，那什么时候发生Mixed GC呢？其实是由一些参数控制着的，另外也控制着哪些老年代Region会被选入CSet。 

* G1HeapWastePercent：在global concurrent marking结束之后，我们可以知道old gen regions中有多少空间要被回收，在每次YGC之后和再次发生Mixed GC之前，会检查垃圾占比是否达到此参数，只有达到了，下次才会发生Mixed GC。
* G1MixedGCLiveThresholdPercent：old generation region中的存活对象的占比，只有在此参数之下，才会被选入CSet。
* G1MixedGCCountTarget：一次global concurrent marking之后，最多执行Mixed GC的次数。 
* G1OldCSetRegionThresholdPercent：一次Mixed GC中能被选入CSet的最多old generation region数量。

除了以上的参数，G1 GC相关的其他主要的参数有：

| 参数                               | 含义                                                         |
| :--------------------------------- | :----------------------------------------------------------- |
| -XX:G1HeapRegionSize=n             | 设置Region大小，并非最终值                                   |
| -XX:MaxGCPauseMillis               | 设置G1收集过程目标时间，默认值200ms，不是硬性条件            |
| -XX:G1NewSizePercent               | 新生代最小值，默认值5%                                       |
| -XX:G1MaxNewSizePercent            | 新生代最大值，默认值60%                                      |
| -XX:ParallelGCThreads              | STW期间，并行GC线程数                                        |
| -XX:ConcGCThreads=n                | 并发标记阶段，并行执行的线程数                               |
| -XX:InitiatingHeapOccupancyPercent | 设置触发标记周期的 Java 堆占用率阈值。默认值是45%。这里的java堆占比指的是non_young_capacity_bytes，包括old+humongous |

<img src="8f235085005aac233ae71be06350eb07.gif" alt="memo" style="zoom:125%;" />

#### CMS

##### 知识点

一般我们都是使用CMS+ParNew进行搭配，这是一款以缩短停顿时间为目的的垃圾收集器，而G1和ZGC也都是以这个为优化和调整的方向。

常用参数：

```bash
-XX:+UseConcMarkSweepGC：启用cms
-XX:ConcGCThreads：并发的GC线程数
-XX:+UseCMSCompactAtFullCollection：FullGC 之后做压缩整理（减少碎片）
-XX:CMSFullGCsBeforeCompaction：间隔多少次 FullGC 之后压缩整理一次，默认是0，代表间隔 0 次 FullGC 后会进行压缩整理，即每次 FullGC 后都会进行整理
-XX:CMSInitiatingOccupancyFraction: 当老年代使用达到该比例时会触发 FullGC（默认是92，这是百分比）
-XX:+UseCMSInitiatingOccupancyOnly：只使用设定的回收阈值(-XX:CMSInitiatingOccupancyFraction设定的值)，如果不指定，JVM仅在第一次使用设定值，后续则会自动调整
-XX:+CMSScavengeBeforeRemark：在CMS GC 标记前启动一次 minor gc，目的在于减少老年代对年轻代的引用，降低CMS GC的标记阶段时的开销，一般CMS的GC耗时 80%都在标记阶段
-XX:+CMSParallellnitialMarkEnabled：表示在初始标记的时候多线程执行，缩短STW
-XX:+CMSParallelRemarkEnabled：在重新标记的时候多线程执行，缩短STW;
```



##### GC执行步骤

![GC过程](20201010095749684.png)

* 初始标记。该过程是需要STW的，仅标记和GC Roots 直接关联的对象，这个过程在JDK7之前是单线程，8之后为多线程并行(没有用户线程在运行)，过程很快。
* 并发标记。该过程是和用户线程一起运行的，不会发生STW，主要是在【初始标记】的基础上，标记剩余的所有引用对象。由于该过程是和用户线程并行的，所以可能会产生误判或者漏标记的情况，后面通过三色标记法解决了这个问题。
* 重新标记。主要是整理在过程【并发标记】中已经被标记为可回收的对象重新被引用，会发生STW。
* 并发清理。这个过程如果有新的对象进来会被标记为黑色，不再处理，不会发生STW。

##### 优缺点分析

**优点**

低停顿，并发收集。

**缺点**

* 对 cpu 资源敏感，会与服务争抢 cpu 资源
* 无法处理浮动垃圾，在并发标记和并发清理阶段产生的浮动垃圾只能留到下次 gc 清理
* 因为使用的是标记清除算法，所以会产生大量的内存碎片
* 垃圾回收过程的不确定性，即在并标记和并发清理阶段产生了大量的对象导致本次 gc 没有进行完却又出发了回收即 concurrent mode failure，此时会 STW 并转换为使用 Serial Old 进行串行收集



#### 三色标记法

所谓三色标记，即垃圾收集器在标记过程中将对象分别标记为三种颜色：
黑色：通过 GC ROOT 扫描到且所有直接引用都扫描到的对象标记为黑色
灰色：通过 GC ROOT 扫描到但未扫描完所有直接引用对象标记为灰色
白色：为被扫描的的对象为白色
在回收时会回收所有标记为白色对象

![三色标记法](20201019214650906.png)

对于多标和漏标的情况垃圾回收器基于三色标记法和对象的读写屏障提供了两种解决方案，增量更新和原始快照 CMS 使用的是增量更新，而 G1 使用的是原始快照。
增量更新：增量更新即为在并发标记过程中在黑色对象成员变量在更新引用时会将该引用记录下来，在重新标记的时候重新对该对象进行扫描标记。也可以理解为如果对象写入了新值就将该对象的状态标记为灰色。
原始快照：原始快照顾名思义即保留原始的信息，在灰色对象成员变量引用更新的时候记录下老对象的引用，在重新标记过程中将这些老对象全都标记为黑色不进行回收，这样可能会产生浮动垃圾，但同时也减少了从新扫描的时间，与增量更新各有利弊。