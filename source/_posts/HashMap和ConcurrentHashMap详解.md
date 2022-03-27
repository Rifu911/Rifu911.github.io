---
title: HashMap和ConcurrentHashMap详解
date: 2021-05-23 10:36:10
categories: 学习笔记
tags: [HashMap, ConcurrentHashMap]
---





#### 为什么JDK8的ConcurrentHashMap放弃使用Segment分段锁

jdk1.7分段锁的实现

和hashmap一样，在jdk1.7中ConcurrentHashMap的底层数据结构是数组加链表。和hashmap不同的是ConcurrentHashMap中存放的数据是一段段的，即由多个Segment(段)组成的。每个Segment中都有着类似于数组加链表的结构。

##### 关于Segment

ConcurrentHashMap有3个参数：

1. initialCapacity：初始总容量，默认16
2. loadFactor：加载因子，默认0.75
3. concurrencyLevel：并发级别，默认16

其中并发级别控制了Segment的个数，在一个ConcurrentHashMap创建后Segment的个数是不能变的，扩容过程过改变的是每个Segment的大小。<!-- more -->

##### 关于分段锁

段Segment继承了重入锁ReentrantLock，有了锁的功能，每个锁控制的是一段，当每个Segment越来越大时，锁的粒度就变得有些大了。

- 分段锁的优势在于保证在操作不同段 map 的时候可以并发执行，操作同段 map 的时候，进行锁的竞争和等待。这相对于直接对整个map同步synchronized是有优势的。
- 缺点在于分成很多段时会比较浪费内存空间(不连续，碎片化); 操作map时竞争同一个分段锁的概率非常小时，分段锁反而会造成更新等操作的长时间等待; 当某个段很大时，分段锁的性能会下降。

#### jdk1.8的map实现

和hashmap一样,jdk 1.8中ConcurrentHashmap采用的底层数据结构为数组+链表+红黑树的形式。数组可以扩容，链表可以转化为红黑树。

##### 什么时候扩容？

1. 当前容量超过阈值
2. 当链表中元素个数超过默认设定（8个），当数组的大小还未超过64的时候，此时进行数组的扩容，如果超过则将链表转化成红黑树

##### 什么时候链表转化为红黑树？

当数组大小已经超过64并且链表中的元素个数超过默认设定（8个）时，将链表转化为红黑树

ConcurrentHashMap的put操作代码如下:

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                //初始化整个table
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                //初始化某个桶
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                //当前table正在迁移，当前线程加入到数据迁移的工作中
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                //对桶中的头结点进行同步加锁操作
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        //当前桶里面是链表
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        //桶里面是一颗红黑树
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    //判断是否需要扩容
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```



把数组中的每个元素看成一个桶。可以看到大部分都是CAS操作，加锁的部分是对桶的头节点进行加锁，锁粒度很小。

##### 为什么不用ReentrantLock而用synchronized ?

- 减少内存开销:如果使用ReentrantLock则需要节点继承AQS来获得同步支持，增加内存开销，而1.8中只有头节点需要进行同步。
- 内部优化:synchronized则是JVM直接支持的，JVM能够在运行时作出相应的优化措施：锁粗化、锁消除、锁自旋等等。