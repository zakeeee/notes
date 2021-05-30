# ArrayList

ArrayList 是一个基于可变长数组的 List 实现类。

默认初始容量是 10。

```java
private static final int DEFAULT_CAPACITY = 10;
```

## 关键源码

### JDK 1.8

#### grow

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;

    // 扩容为当前的 1.5 倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);

    // 如果扩容为 1.5 倍后还是比 minCapacity 小，那么新的容量为 minCapacity
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;

    // 如果新的容量比最大数组容量大，使用 hugeCapacity
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);

    // 拷贝数组元素
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    // 如果溢出了，抛出异常
    if (minCapacity < 0)
        throw new OutOfMemoryError();

    // 如果超过了最大数组容量，那么返回 Integer.MAX_VALUE，否则返回最大数组容量
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```