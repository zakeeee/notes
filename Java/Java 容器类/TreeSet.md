# TreeSet

TreeSet 是基于红黑树的 Set 实现类。由于是基于红黑树的，容器中元素是有序的。

与 HashSet 和 HashMap 的关系相似，TreeSet 底层默认使用 TreeMap 实现，但是也可以是任何实现了 NavigableMap 接口的类。所有键值对的值都是一个 PRESENT 实例，它是一个虚拟值。

```java
private static final Object PRESENT = new Object();
```