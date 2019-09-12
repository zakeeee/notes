# Java 基础

[TOC]

## Java 中的关键字

关键字一律用小写字母标识，按其用途划分为如下几组。

|            用途            |                                                      关键字                                                      |
| ------------------------- | --------------------------------------------------------------------------------------------------------------- |
| 用于数据类型                | boolean、byte、char、double、float、int、long、new、short、void、instanceof                                      |
| 用于语句                   | break、case、catch、continue、default 、do、else、for、if、return、switch、try、while、finally、throw、this、super |
| 用于修饰                   | abstract、final、native、private、protected、public、static、synchronized、transient、volatile                    |
| 用于方法、类、接口、包和异常 | class、extends、implements、interface、package、import、throws                                                   |
| 保留字                     | true、false、null                                                                                               |

## 引用传递与值传递

Java 里面只有值传递，当调用一个方法并传入引用时，这个引用会被拷贝一份。因此在方法中直接操作引用（比如让它指向另一个对象）对方法的调用者来说是没有影响的。

```java
class Main {
    public static void foo(String str) {
        str = "123";
    }

    public static void main(String[] args) {
        String str = "abc";
        foo(str);
        System.out.println(str);
    }
}

/*
输出：
abc
*/
```

## == 与 equals()

`==` 是判断两个对象的地址是不是相等。即判断两个对象是不是同一个对象。(基本数据类型比较的是值，引用数据类型比较的是内存地址)

`equals()` 也是判断两个对象是否相等，但它一般有两种使用情况：

- 情况1：类没有覆盖 `equals()` 方法。则通过 `equals()` 比较该类的两个对象时，等价于通过 `==` 比较这两个对象。
- 情况2：类覆盖了 `equals()` 方法。一般我们都覆盖 `equals()` 方法来判断两个对象的内容是否相等，如果内容相等，则返回 true（即认为这两个对象相等）。

## hashCode() 与 equals()

### 为什么重写 equals() 还要重写 hashCode()

Java 里 HashMap 判断是否存在某个对象时，首先需要计算出对象的哈希值，然后找到对应桶下标，并遍历桶中的元素；如果找到一个哈希值相同的对象，再使用 equals() 判断是否真的相同，如果相同就说明存在，如果不相同就继续遍历，直到遍历结束。因此如果重写了 equals 而没有重写 hashCode，那么相同的对象也会被映射到不同的桶下标，这显然是不符合原意的。

> HashSet 本质上也是使用 HashMap 实现的，所以道理是一样的。

### hashCode() 与 equals() 的相关规定

1. 如果两个对象相等，则哈希值一定也是相同的。
2. 两个对象相等，对两个对象分别调用 equals() 方法都返回 true，即 a.equals(b) == b.equals(a) == true。
3. 两个对象有相同的哈希值，它们也不一定是相等的，可能存在哈希冲突。
4. **因此，equals() 方法被覆盖过，则 hashCode() 方法也必须被覆盖**。
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该类的两个对象无论如何都不会相等。（即使这两个对象指向相同的数据）

## String、StringBuffer、StringBuilder 的区别

- String 是不可变的，StringBuffer 和 StringBuilder 是可变的。
- StringBuffer 是线程安全的，StringBuilder 是非线程安全的。

一般情况下，执行字符串拼接时，效率排名：StringBuilder > StringBuffer > String。

但是，如果代码中是这样

```java
String str = "123" + "456";
```

那么 JVM 会把两个字符串常量对象优化为一个字符串常量对象，即 "123456"，这种情况下 String 速度最快。

## 序列化

序列化就是将一个 Java 对象转化为字节流，或者说 byte 数组，而反序列化就是从一个字节流中还原出这个对象。

## Java 实现多态的方式

- 编译时多态：方法的重载，具有相同名字但是可以有不同参数列表。编译时编译器根据传入的参数列表来确定实际应该调用的是哪个方法。
- 运行时多态：子类重写父类方法或者类实现接口方法，方法名字和参数列表都要一样。运行时使用父类引用指向子类对象，调用该方法就会实际调用子类对象重写后的方法。

## Scanner 的 next 和 nextLine

- next：从遇到的第一个有效字符开始扫描，遇到第一个分隔符或者结束符时结束。
- nextLine：扫描当前行剩余的字符串，直到遇到换行符时结束，并跳转到下一行的开头。

```java
Scanner sc = new Scanner(System.in);
// 假如输入的是：aaa bbb ccc
str1 = sc.next();  // str1 = "aaa"
str2 = sc.nextline();  // str2 = " bbb ccc"，注意这里 bbb 前面的空格也被扫描了
```