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

## == 与 equals 的区别

== 是判断两个对象的地址是不是相等。即判断两个对象是不是同一个对象。(基本数据类型比较的是值，引用数据类型比较的是内存地址)

equals 也是判断两个对象是否相等，但它一般有两种使用情况：

- 如果类没有覆盖 equals 方法，则使用 Object 类的 equals 方法，等价于通过 == 比较这两个对象。
- 如果类覆盖了 equals 方法，一般是根据内容来比较两个对象是否相等，如果内容相等，则返回 true（即认为这两个对象相等）。

Object 类的 equals 方法实现：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

## hashCode 方法与 equals 方法

### 为什么重写 equals 方法还要重写 hashCode 方法

Java 里 HashMap 判断是否存在某个对象时，首先需要计算出对象的哈希值，然后找到对应桶下标，并遍历桶中的元素；如果找到一个哈希值相同的对象，再使用 equals 判断是否真的相同，如果相同就说明存在，如果不相同就继续遍历，直到遍历结束。因此如果重写了 equals 而没有重写 hashCode，那么相同的对象也会被映射到不同的桶下标，这显然是不符合原意的。

### hashCode 方法与 equals 方法的相关规定

1. 如果两个对象相等，则哈希值一定也是相同的。
2. 两个对象相等，对两个对象分别调用 equals 方法都返回 true，即 a.equals(b) == b.equals(a) == true。
3. 两个对象有相同的哈希值，它们也不一定是相等的，可能存在哈希冲突。
4. equals 方法被覆盖过，则 hashCode 方法也必须被覆盖。
5. hashCode 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode，则该类的两个对象无论如何都不会相等。（即使这两个对象指向相同的数据）

## String、StringBuffer、StringBuilder 的区别

- String 是不可变的，StringBuffer 和 StringBuilder 是可变的。
- StringBuffer 是线程安全的（通过给方法加上 synchronized 关键字），StringBuilder 是非线程安全的。

一般情况下，执行字符串拼接时，效率排名：StringBuilder > StringBuffer > String。

但是，如果代码中是这样

```java
String str = "123" + "456";
```

那么 JVM 会把两个字符串常量对象优化为一个字符串常量对象，即 "123456"，这种情况下 String 速度最快。

## 序列化

序列化就是将一个 Java 对象转化为字节流，或者说 byte 数组，而反序列化就是从一个字节流中还原出这个对象。

一个类的对象要想序列化成功，必须满足两个条件：

- 该类必须实现 `java.io.Serializable` 接口。
- 该类的所有属性必须是可序列化的。如果有一个属性不是可序列化的，则该属性必须注明是 transient。

实现了 `java.io.Serializable` 接口的类可以使用 `java.io.ObjectOutputStream` 来序列化到 `java.io.OutputStream` 里。反序列化可以用 `java.io.ObjectInputStream` 从 `java.io.InputStream` 里读。

```java
public class Main {
    static class A implements Serializable {
        private int val;
        private String str;
        private transient int t;  // transient 修饰的不会被序列化

        public A(int val, String str, int t) {
            this.val = val;
            this.str = str;
            this.t = t;
        }

        @Override
        public String toString() {
            return this.val + this.str + this.t;
        }
    }

    public static void main(String[] args) {
        try {
            OutputStream outputStream = new FileOutputStream(new File("d:\\1.txt"));
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
            A a = new A(123, "456", 789);
            System.out.println("序列化前：" + a);
            objectOutputStream.writeObject(a);

            InputStream inputStream = new FileInputStream(new File("d:\\1.txt"));
            ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);
            A aa = (A) objectInputStream.readObject();
            System.out.println("反序列化后：" + aa);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

/*
输出：
序列化前：123456789
反序列化后：1234560
*/
```

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

## 抽象类和接口的区别

- 抽象类可以有非抽象的方法，而接口都是抽象的方法（接口的方法隐式默认为 public abstract 的）。
- 一个类只能继承一个抽象类，但是可以实现多个接口。
- 类如果要实现一个接口，它必须要实现接口声明的所有方法。类可以不实现抽象类声明的所有方法，但是在这种情况下类也必须得声明成是抽象的。
- 抽象类可以在不提供接口方法实现的情况下实现接口。
- 接口中声明的变量默认都是 final 的。抽象类可以包含非 final 的变量。
- 接口中的成员函数默认是 public 的。抽象类的成员函数可以是 private，protected 或者是 public。
- 接口是绝对抽象的，不可以被实例化。抽象类也不可以被实例化，但是，如果它包含 main 方法的话是可以被调用的。

## Integer 的 valueOf 和 parseInt 区别

从源码可以看出 parseInt 会直接返回 int 类型，而 valueOf 实际会调用 parseInt，然后封装成 Integer 类型，并且会查询 Integer 缓存，如果存在直接返回。

```java
public static int parseInt(String s, int radix) throws NumberFormatException
```

```java
public static Integer valueOf(String s) throws NumberFormatException {
    return Integer.valueOf(parseInt(s, 10));
}

public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```