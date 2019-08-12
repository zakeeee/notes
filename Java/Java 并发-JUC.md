# Java 并发-JUC

[TOC]

## AQS

AQS（Abstract Queued Synchronizer，队列同步器）是构建锁或者其他同步组件的基础框架（如 ReentrantLock、ReentrantReadWriteLock、Semaphore 等），是 JUC 并发包中的核心基础组件。

## CountDownLatch

类似于一个计数器，用于一个线程等待其他若干线程都执行完毕再继续往下执行。

构造器

```java
//参数count为计数值
public CountDownLatch(int count)
```

重要方法

```java
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public void await() throws InterruptedException

//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException

//将count值减1
public void countDown()
```

## CyclicBarrier

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier 可以被重用。我们暂且把这个状态就叫做 barrier，当调用 `await()` 方法之后，线程就处于 barrier 了。

构造器

```java
// 参数 parties 指让多少个线程或者任务等待至 barrier 状态；参数barrierAction 为当这些线程都达到 barrier 状态时会执行的内容。
public CyclicBarrier(int parties, Runnable barrierAction)

public CyclicBarrier(int parties)
```

重要方法

```java
// 用来挂起当前线程，直至所有线程都到达 barrier 状态再同时执行后续任务；
public int await() throws InterruptedException, BrokenBarrierException

// 让这些线程等待至一定的时间，如果还有线程没有到达 barrier 状态就直接让到达 barrier 的线程执行后续任务。
public int await(long timeout, TimeUnit unit) throws InterruptedException,BrokenBarrierException,TimeoutException
```

## Semaphore

信号量，允许有限个线程同时访问。

构造器

```java
// 参数permits表示许可数目，即同时可以允许多少线程进行访问
public Semaphore(int permits)

// 这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
public Semaphore(int permits, boolean fair)
```

重要方法

```java
// 获取一个许可
public void acquire() throws InterruptedException

//获取permits个许可
public void acquire(int permits) throws InterruptedException

//释放一个许可
public void release()

//释放permits个许可
public void release(int permits)
```

## FutureTask

## Blocking Queue

## Fork Join

## ConcurrentHashMap

### ConcurrentHashMap 和 Hashtable 的区别

ConcurrentHashMap 和 Hashtable 的区别主要体现在实现线程安全的方式上不同。

#### 底层数据结构

JDK1.7的 ConcurrentHashMap 底层采用**分段的数组+链表**实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用**数组+链表**的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的。

#### 实现线程安全的方式（重要）

1. **在JDK1.7的时候，ConcurrentHashMap（分段锁）** 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了Segment的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6以后 对 synchronized锁做了很多优化）** 整个看起来就像是优化过且线程安全的 HashMap，虽然在JDK1.8中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本。
2. **Hashtable(同一把锁)** 使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

