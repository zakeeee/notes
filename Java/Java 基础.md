# Java 基础

要点：

- Java里面只有值传递
- ==与equals()
- hashCode()与equals()
- String、StringBuffer、StringBuilder区别
- 序列化
- Java 实现多态的方式

## Java里面只有值传递

Java里面只有值传递，调用方法传入引用时，其实引用也会被拷贝。

## ==与equals()

== : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象。(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)

equals() : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

- 情况1：类没有覆盖equals()方法。则通过equals()比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况2：类覆盖了equals()方法。一般，我们都覆盖equals()方法来两个对象的内容相等；若它们的内容相等，则返回true(即，认为这两个对象相等)。

## hashCode()与equals()

Java 里 HashSet 或者 HashMap 判断是否存在相同对象时，首先判断计算出的哈希值，如果哈希值不同，就认为没有相同对象；如果哈希值相同，再调用 equals() 判断是否真的相同，如果相同，就不会将对象添加进来，如果不同，就重新散列到其他位置，减少了使用 equals() 比较的次数，提高效率。

### hashCode() 与 equals() 的相关规定

1. 如果两个对象相等，则 hashcode 一定也是相同的
2. 两个对象相等,对两个对象分别调用 equals 方法都返回 true
3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

## String、StringBuffer、StringBuilder区别

String是不可变的，StringBuffer和StringBuilder是可变的。
StringBuffer是线程安全的。
StringBuilder是非线程安全的。

一般情况下，执行字符串拼接时，效率排名：StringBuilder > StringBuffer > String。

但是，如果代码中是这样

```java
String str = "123" + "456";
```

那么JVM会优化为一个String对象，即"123456"，这种情况下String速度最快。

## 序列化

## Java 实现多态的方式

- 编译时多态：方法的重载，具有相同名字但是可以有不同参数列表。编译时编译器根据传入的参数列表来确定实际应该调用的是哪个方法。
- 运行时多态：子类重写父类方法或者类实现接口方法，方法名字和参数列表都要一样。运行时使用父类引用指向子类对象，调用该方法就会实际调用子类对象重写后的方法。

