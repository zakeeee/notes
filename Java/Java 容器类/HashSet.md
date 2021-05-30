# HashSet

HashSet 是基于哈希的 Set 实现类。由于是基于哈希的，容器中元素是无序的。

HashSet 底层使用 HashMap 实现，所有键值对的值都是一个 PRESENT 实例，它是一个虚拟值。

```java
private static final Object PRESENT = new Object();
```