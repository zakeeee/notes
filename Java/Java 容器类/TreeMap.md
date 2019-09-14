# TreeMap

[TOC]

TreeMap 是基于红黑树的 Map 实现类。由于是基于红黑树的，容器中元素是有序的，默认是根据键的 `compareTo` 方法来作为排序依据。

## 关键源码

### JDK 1.8

#### put

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    // 如果红黑树为空，插入一个新的键值对
    if (t == null) {
        compare(key, key); // type (and possibly null) check

        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    // 如果使用比较器
    if (cpr != null) {
        // 二叉搜索树的二分查找
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // 否则不使用比较器，使用键的 compareTo 方法进行比较
    else {
        // 键不能为空，否则抛出异常
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        // 二叉搜索树的二分查找
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }

    // 如果没找到键值对，就插入一个新的键值对
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

#### get

```java
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p == null ? null : p.value);
}

final Entry<K,V> getEntry(Object key) {
    // 为了性能，卸载基于比较器的版本
    if (comparator != null)
        return getEntryUsingComparator(key);

    // 如果键为 null，抛出 NullPointerException 异常
    if (key == null)
        throw new NullPointerException();

    @SuppressWarnings("unchecked")
    Comparable<? super K> k = (Comparable<? super K>) key;

    // 二叉搜索树的二分查找
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}

final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
    K k = (K) key;

    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        // 二叉搜索树的二分查找
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
    }
    return null;
}
```