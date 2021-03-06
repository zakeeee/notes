# Java 并发-JUC

## 原子类

JUC 包中的原子类都是使用 CAS 实现的，一个 CAS 操作的定义类似于 `compareAndSwap(obj, offset, expected, update)`，其中 expected 是对要修改的变量当前值的期望值，如果变量的值不是这个值，CAS 操作会失败，update 是要把变量修改修改成的值。

除了 AtomicXXX 的类之外，JDK 1.8 之后还提供了 LongAdder，DoubleAdder，LongAccumulator，DoubleAccumulator 等原子操作类。AtomicXXX 类是多个线程对同一个值进行操作，而 Adder 和 Accumulator 是对一组 Cell 对象进行操作，每个 Cell 对象都保存了一个值以及一个 CAS 操作方法。

Accumulator 比 Adder 功能更强大，可以提供初始值和双目运算方法。

## AQS

AQS（Abstract Queued Synchronizer，抽象队列同步器）是构建锁或者其他同步组件的基础框架（如 ReentrantLock、ReentrantReadWriteLock、Semaphore 等），是 JUC 并发包中的核心基础组件。

AQS 是一个抽象类，Sync 类继承自 AQS。

## 常用类

### CountDownLatch

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

### CyclicBarrier

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

### Semaphore

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

### FutureTask

### Blocking Queue

### Fork Join
