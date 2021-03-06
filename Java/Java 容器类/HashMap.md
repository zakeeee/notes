# HashMap

HashMap 是基于哈希的 Map 实现类。由于是基于哈希的，容器中元素是无序的。

## HashMap 扩容

### 扩容条件

JDK 1.7 中，HashMap 扩容需要同时满足两个条件：

- 键值对的数量大于阈值。
- 插入的元素刚好出现哈希冲突，即插入了一个非空的桶中。

---

JDK 1.8 中，只要键值对的数量大于阈值。

### 多线程不安全

HashMap 如果在扩容时有其他线程进行 put 操作，可能会出现死循环的问题。

为什么呢？

## JDK 1.8 中 HashMap 的优化

- 一个桶中的链表长度超过阈值时转红黑树。
- hash 方法优化，不用再进行多次位移，只需要位移一次。
- 扩容机制优化，不用再每个键值对进行重新映射，而是分为了两个链表，详情请看后面的源码分析。

**为什么转红黑树的阈值设置为 8**

首先为什么不是一开始就使用红黑树呢？根据 JDK 文档说明，TreeNode 的大小大约是普通 Node 大小的两倍，而且红黑树的维护代价也比链表高，因此一开始就使用红黑树是不值得的。

那么为什么阈值设置为 8 呢？根据 HashMap 文档里的说明，条目插入到同一个桶中的概率服从泊松分布，同一个桶中元素超过 8 的概率小于千万分之一，几乎是不可能事件，因此阈值设置为 8 的情况下，链表是很难转化到红黑树的，即使偶尔转化了，对整体性能影响也不大。

**为什么用红黑树不用平衡二叉树**

平衡二叉树为了保持树的平衡性，每次插入和删除都要进行大量的旋转，很耗费时间，而红黑树插入最多 2 次旋转，删除最多 3 次旋转。且红黑树极端情况下高度也只是差一倍，时间复杂度不会退化到线性。

## HashMap 和 HashTable 的区别

HashMap 是非线程安全的，键值可以为 null。

HashTable 是线程安全的，方法使用 synchronized 修饰，键值不能为 null。HashTable 现在已经被淘汰了，如果要保证线程安全应该用 ConcurrentHashMap。

## 关键源码

### JDK 1.7

#### hash

```java
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

#### put

```java
public V put(K key, V value) {
    // 如果数组为空，初始化数组
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }

    // 如果键为 null，使用 putForNullKey 插入然后返回。
    if (key == null)
        return putForNullKey(value);

    // 根据键得到哈希值
    int hash = hash(key);
    // 根据哈希值找到桶下标
    int i = indexFor(hash, table.length);

    // 遍历链表
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        // 如果找到了目标键值对，就修改它的值，并返回旧的值
        // 判断对象相等时首先判断两个哈希值，若哈希值相等再进行对象相等的判定
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    // 如果没找到对应键值对，说明需要插入一个，修改次数加 1
    modCount++;
    // 然后添加一个键值对
    addEntry(hash, key, value, i);
    // 并返回 null，因为是新添加的，所以没有旧的值
    return null;
}

void addEntry(int hash, K key, V value, int bucketIndex) {
    // 如果插入后键值对数量大于阈值，且待插入的数组位置不为空（发生了哈希冲突），就需要扩容
    if ((size >= threshold) && (null != table[bucketIndex])) {
        // 数组大小扩容为当前的 2 倍
        resize(2 * table.length);

        // 根据键得到哈希值，如果键为 null，默认哈希值为 0
        hash = (null != key) ? hash(key) : 0;

        // 根据哈希值得到桶下标
        bucketIndex = indexFor(hash, table.length);
    }

    // 在桶中添加一个键值对
    createEntry(hash, key, value, bucketIndex);
}

void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];

    // 头插法
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```

#### get

```java
public V get(Object key) {
    // 如果键为 null，返回 null 键对应的内容
    if (key == null)
        return getForNullKey();

    Entry<K,V> entry = getEntry(key);

    // 如果没找到对应键值对，就返回 null，否则返回键值对的值
    return null == entry ? null : entry.getValue();
}

final Entry<K,V> getEntry(Object key) {
    // 如果 HashMap 中没有元素，返回 null
    if (size == 0) {
        return null;
    }

    // 根据键得到哈希值，如果键为 null，默认为 0
    int hash = (key == null) ? 0 : hash(key);

    // 遍历链表
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
            e != null;
            e = e.next) {
        Object k;

        // 如果找到了目标键值对，就返回它
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }

    // 没找到目标键值对，返回 null
    return null;
}
```

#### resize

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;

    // 如果旧的容量已经达到了最大容量，就不扩容了，只是把阈值改为 Integer.MAX_VALUE
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    // 创建新数组
    Entry[] newTable = new Entry[newCapacity];

    // 转移数据到新数组
    transfer(newTable);
    table = newTable;

    // 计算阈值
    threshold = (int) (newCapacity * loadFactor);
}

void transfer(Entry[] newTable) {
    Entry[] src = table;
    int newCapacity = newTable.length;

    // 遍历旧的数组的每个桶
    for (int j = 0; j < src.length; j++) {
        Entry<K, V> e = src[j];

        // 如果这个桶不为空
        if (e != null) {
            src[j] = null;

            // 遍历链表，每个元素都重新映射并连接到新的位置里
            do {
                Entry<K, V> next = e.next;
                // 计算新的下标
                int i = indexFor(e.hash, newCapacity);
                // 头插法
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
}

static int indexFor(int h, int length) {
    return h & (length - 1);
}
```

### JDK 1.8

#### hash

```java
static final int hash(Object key) {
    int h;
    // 如果键为 null，则默认是 0
    // 否则根据键得到哈希值 h，然后将 h 与 h  无符号右移 16 位后的数字异或
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

#### put

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;

    // 如果数组为 null 或者数组长度为 0，就初始化数组
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;

    // 根据哈希值获得桶的下标，然后获取桶中的第一个节点，如果为空，则插入新节点
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);

    // 如果不为空，说明发生了哈希冲突
    else {
        Node<K,V> e; K k;

        // 如果 p 就是目标节点，直接将 e 引用这个节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;

        // 否则如果 p 是红黑树节点类型，进行红黑树的插入操作
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);

        // 否则遍历链表并修改或者插入
        else {
            for (int binCount = 0; ; ++binCount) {

                // 如果链表中没找到，就插入一个新的到尾部，且如果长度大于阈值，就转为红黑树
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }

                // 如果找到了就直接退出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }

        // 如果目标键的 entry 存在，就用新的值覆盖旧的值，并返回旧的值
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }

    // 没有旧的值，修改次数加 1
    ++modCount;
    // 如果元素个数大于阈值，扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    // 没有旧的值，所以返回 null
    return null;
}
```

#### get

```java
public V get(Object key) {
    Node<K,V> e;
    // 如果没找到对应节点，返回 null，否则返回节点的值
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;

    // 如果桶数组合法且桶不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {

        // 判断桶中第一个节点是否是要找的节点，如果是就返回这个节点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;

        // 如果桶中有超过一个节点
        if ((e = first.next) != null) {

            // 如果第一个节点是红黑树节点，就使用红黑树的查找
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);

            // 否则遍历链表查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }

    // 否则说明不存在
    return null;
}
```

#### resize

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;

    // 如果旧的数组不为 null
    if (oldCap > 0) {
        // 如果旧的容量大于等于最大容量（1 << 30），就不扩容，只是阈值设置为 Integer.MAX_VALUE，然后直接返回旧的数组
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 否则如果旧的容量翻倍后比最大容量小，且旧容量大于等于默认初始容量（1 << 4）
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 阈值翻倍
            newThr = oldThr << 1;
    }
    // 否则如果旧的阈值大于 0，新的容量就为旧的阈值
    else if (oldThr > 0)
        newCap = oldThr;
    // 否则初始化，阈值因子默认为 0.75
    else {
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }

    // 如果新的阈值是 0
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        // 如果新的容量和 ft 都小于最大容量，那么新的阈值就是 ft，否则新的阈值是 Integer.MAX_VALUE
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;

    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];  // 新的数组
    table = newTab;

    // 元素重新映射
    if (oldTab != null) {
        // 遍历原数组每个桶
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            // 如果当前桶不为空
            if ((e = oldTab[j]) != null) {
                // 先把原数组这个位置置为 null，后面会重新映射
                oldTab[j] = null;
                // 如果只有一个元素，直接放置到新的映射位置上
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果是红黑树节点，执行红黑树的 split 操作
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 否则是链表
                else {
                    Node<K,V> loHead = null, loTail = null;  // low 链表
                    Node<K,V> hiHead = null, hiTail = null;  // high 链表
                    Node<K,V> next;

                    // 遍历旧的数组 j 位置的链表，并根据 oldCap 中 1 的那一位来分为两个链表 low 和 high
                    // 这么做是因为如果哈希值在那一位为 0，那么重新映射后他们在新数组中的下标与原数组中的下标相同
                    // 如果那一位为 1，那么重新映射后，他们会被映射到原下标加上 oldCap 的位置
                    // 以 oldCap 为 16 为例，扩容后 newCap 为 32
                    // 如果最后五位是 00000 到 01111，也就是 0 到 15，那么重新映射后还是 0 到 15
                    // 如果最后五位是 10000 到 11111，也就是 16 到 31，那么没扩容之前他们还是映射到 0 到 15
                    // 但重新映射后就不是之前的 0 到 15 的位置，而是 16 到 31 的位置了，也就是多了一个 oldCap 的偏移
                    do {
                        next = e.next;
                        // 由于 oldCap 是 2 的次幂，因此只会有一位是 1，比如 16 就是 10000
                        // 如果 e 的哈希值在 oldCap 为 1 的那一位上是 0，就连接到 low 链表上
                        // 如果是 1 就连接到 high 链表上
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);

                    // low 链表放在新的数组的 j 位置上
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }

                    // high 链表放在新的数组的 j + oldCap 位置上
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```