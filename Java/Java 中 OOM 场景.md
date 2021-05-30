# Java 中 OOM 场景

## 堆内存溢出

**错误信息**：

```
java.lang.OutOfMemoryError: Java heap space
```

**出现原因**：

此 OOM 是由于 JVM 中 heap 的最大值不满足需要，将设置 heap 的最大值调高即可，参数样例为：`-Xmx2G`

**解决方法**：

调高 heap 的最大值，即 `-Xmx` 的值调大。

## 永久代内存溢出

**错误信息**：

```
java.lang.OutOfMemoryError: Java perm space
```

**出现原因**：

此 OOM 是由于 JVM 中 perm 的最大值不满足需要，将设置 perm 的最大值调高即可，参数样例为 `-XX:MaxPermSize=512M`

**解决方法**：

调高 heap 的最大值，即 `-XX:MaxPermSize` 的值调大。

另外，注意一点，perm 一般是在 JVM 启动时加载类进来，如果是 JVM 运行较长一段时间而不是刚启动后溢出的话，很有可能是由于运行时有类被动态加载进来，此时建议用 CMS 策略中的类卸载配置。如 `-XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled`。

## GC 时对象过多导致内存溢出

**错误信息**：

```
java.lang.OutOfMemoryError: GC overhead limit exceeded
```

**出现原因**：

此 OOM 是由于 JVM 在 GC 时，对象过多，导致内存溢出，建议调整 GC 的策略，在一定比例下开始 GC 而不要使用默认的策略，或者将新代和老代设置合适的大小，需要进行微调存活率。

**解决方法**：

改变 GC 策略，在老代 80% 时就开始 GC，并且将 `-XX:SurvivorRatio（-XX:SurvivorRatio=8）` 和 `-XX:NewRatio（-XX:NewRatio=4）`设置的更合理。

## 创建大量本地线程导致内存溢出

**错误信息**：

```
java.lang.OutOfMemoryError: unable to create new native thread
```

**出现原因**：

通常是由于操作系统限制或者操作系统没有足够的内存来分配。

```
线程数 = (MaxProcessMemory - JVMMemory - ReservedOsMemory) / (ThreadStackSize)

MaxProcessMemory：进程的最大寻址空间
JVMMemory：JVM 内存
ReservedOsMemory：保留的操作系统内存，如 Native heap，JNI 之类，一般 100 多 M
ThreadStackSize：线程栈的大小，JVM 启动时由 -Xss 指定
```

如果 JVM 内存调的过大或者可利用率小于 20%，可以建议将 heap 及 perm 的最大值下调，并将线程栈调小，即 `-Xss` 调小，如 `-Xss128k`。

[JVM 最多能创建多少个线程](https://www.cnblogs.com/firstdream/p/7802290.html)

**解决方法**：

在 JVM 内存不能调小的前提下，将 `-Xss` 设置较小，如 `-Xss:128k`。

## 申请超大数组导致内存溢出

**错误信息**：

```
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
```

**出现原因**：

此类信息表明应用程序（或者被应用程序调用的 APIs）试图分配一个大于堆大小的数组。例如，如果应用程序 new 一个数组对象，大小为 512M，但是最大堆大小为 256M，因此 OutOfMemoryError 会抛出，因为数组的大小超过虚拟机的限制。

**解决方法**：

1. 首先检查 heap 的 `-Xmx` 是不是设置的过小。
2. 如果 heap 的 `-Xmx` 已经足够大，那么请检查应用程序是不是存在 bug，例如：应用程序可能在计算数组的大小时，存在算法错误，导致数组的 size 很大，从而导致巨大的数组被分配。

## 交换区小导致内存溢出

**错误信息**：

```
java.lang.OutOfMemoryError: request <size> bytes for <reason>. Out of swap space?
```

**出现原因**：

抛出这类错误，是由于从 native 堆中分配内存失败，并且堆内存可能接近耗尽。这类错误可能跟应用程序没有关系，例如下面两种原因也会导致错误的发生：

- 操作系统配置了较小的交换区。
- 系统的另外一个进程正在消耗所有的内存。

**解决方法**：

1. 检查 os 的 swap 是不是没有设置或者设置的过小。
2. 检查是否有其他进程在消耗大量的内存，从而导致当前的 JVM 内存不够分配。

注意：虽然有时 `<reason>` 部分显示导致 OOM 的原因，但大多数情况下，`<reason>` 显示的是提示分配失败的源模块的名称，所以有必要查看日志文件，如 crash 时的 hs 文件。

[参考文章](https://www.cnblogs.com/robertsun/p/4183270.html)