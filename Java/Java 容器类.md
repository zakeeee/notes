# Java 容器类

[TOC]

## ArrayList与Vector的区别

ArrayList和Vector底层都是用数组来实现的。

ArrayList是非线程安全的。
Vector所有方法都是同步的，是线程安全的，但是如果不需要考虑线程安全性，Vector的性能就低很多。

## HashMap和HashTable的区别

HashMap是非线程安全的，键值可以为null。
HashTable是线程安全的，键值不能为null，HashTable现在已经被淘汰了，如果要保证线程安全应该用ConcurrentHashMap。

## HashMap 扩容

时机：当 size 超过阈值（threshold = loadFactor * capacity）且插入的元素出现哈希冲突。

容量扩展为原来的两倍，并把所有元素进行重新映射。

HashMap 插入时是采用头插法，JDK 1.8 之后是插入尾部，并且一个桶中元素大于 8 时转为红黑树。

## ConcurrentHashMap 和 Hashtable 的区别

ConcurrentHashMap 和 Hashtable 的区别主要体现在实现线程安全的方式上不同。

### 底层数据结构

JDK1.7的 ConcurrentHashMap 底层采用**分段的数组+链表**实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用**数组+链表**的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的。

### 实现线程安全的方式（重要）

1. **在JDK1.7的时候，ConcurrentHashMap（分段锁）** 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了Segment的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6以后 对 synchronized锁做了很多优化）** 整个看起来就像是优化过且线程安全的 HashMap，虽然在JDK1.8中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本。
2. **Hashtable(同一把锁)** 使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。
