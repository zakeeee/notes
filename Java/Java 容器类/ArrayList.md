# ArrayList

ArrayList 是一个基于可变长数组的 List 实现类。

### ArrayList 与 Vector 的区别

ArrayList和Vector底层都是用数组来实现的。

ArrayList是非线程安全的。

Vector所有方法都是同步的，是线程安全的，但是如果不需要考虑线程安全性，Vector的性能就低很多。