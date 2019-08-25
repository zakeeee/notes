# Java 容器类

[TOC]

## ArrayList

ArrayList 是一个基于可变长数组的 List 实现类。

### ArrayList与Vector的区别

ArrayList和Vector底层都是用数组来实现的。

ArrayList是非线程安全的。

Vector所有方法都是同步的，是线程安全的，但是如果不需要考虑线程安全性，Vector的性能就低很多。

## ArrayDeque

ArrayDeque 是一个基于可变长数组的 Deque（双向队列）实现类。

## LinkedList

LinkedList 是一个基于双向链表的 List 实现类。此外，它也实现了 Deque 接口，可以用来作为双向队列。

## HashMap

HashMap 是基于哈希的 Map 实现类。由于是基于哈希的，容器中元素是无序的。

### HashMap 扩容

HashMap 扩容需要同时满足两个条件：

- HashMap 中条目（Entry）的数量超过阈值（threshold）。
- 插入的元素刚好出现哈希冲突，即插入了一个非空的桶中。

> threshold = loadFactor * capacity。

容量扩展为原来的两倍，并把所有元素进行**重新映射**。

HashMap 插入时在 JDK 1.7 以前是采用头插法，JDK 1.8 之后是插入尾部，并且一个桶中元素大于 8 时转为红黑树。

#### 为什么阈值设置为 8

根据 HashMap 文档里的说明，条目插入到同一个桶中的概率服从泊松分布，同一个桶中元素超过 8 的概率小于千万分之一，几乎是不可能事件，因此阈值设置为 8 的情况下，链表是很难转化到红黑树的，即使偶尔转化了，对整体性能影响也不大。

#### 为什么用红黑树不用平衡二叉树

平衡二叉树为了保持树的平衡性，每次插入和删除都要进行大量的旋转，很耗费时间，而红黑树插入最多 2 次旋转，删除最多 3 次旋转。且红黑树极端情况下高度也只是差一倍，时间复杂度不会退化到线性。

### HashMap和HashTable的区别

HashMap 是非线程安全的，键值可以为 null。
HashTable 是线程安全的，键值不能为 null，HashTable 现在已经被淘汰了，如果要保证线程安全应该用 ConcurrentHashMap。

## HashSet

HashSet 是基于哈希的 Set 实现类。由于是基于哈希的，容器中元素是无序的。

它的底层还是使用 HashMap 实现的，所有的条目的 value 都是同一个 Object 实例。

## TreeMap

TreeMap 是基于红黑树的 Map 实现类。由于是基于红黑树的，容器中元素是有序的。

## TreeSet

TreeSet 是基于红黑树的 Set 实现类。由于是基于红黑树的，容器中元素是有序的。

