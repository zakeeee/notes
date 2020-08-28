# 箭头函数中 this 作用域

## JavaScript 中作用域

1. 全局作用域
2. 函数作用域
3. 块作用域（ES6），比如条件、循环语句后的代码块

从下面的例子可以看出三个作用域互不影响。

```js
// 1. 全局作用域
let a = 1;

function foo() {
  // 2. 函数作用域
  let a = 2;
  if (true) {
    // 3. 块作用域（ES6）
    let a = 3;
    console.log('块' + a);
  }
  console.log('函数' + a);
}

foo();
console.log('全局' + a);

// 输出：
// 块3
// 函数2
// 全局1
```

## 普通函数和箭头函数中 this 指向

网上解释：

- 普通函数内部的 this 指向调用该函数的对象。
- 箭头函数内部的 this 指向定义这个箭头函数的位置的上下文环境中的 this 对象。

下面用几个具体的例子来看看它们到底有什么区别

### 没有闭包的时候

```js
this.a = 'global.a';

const obj = {
  a: 'obj.a',
  test1: function () {
    console.log(this.a);
  },
  test2: () => {
    console.log(this.a);
  },
};

const obj2 = { a: 'obj2.a' };

obj.test1(); // obj.a
obj.test1.call(obj2); // obj2.a

obj.test2(); // global.a
obj.test2.call(obj2); // global.a
```

test1 中，函数中的 this 指向的是调用这个函数的对象。其中[call 方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)可以指定哪个对象来调用这个函数。

```js
obj.test1(); // obj.a
```

调用这个函数的对象是 obj，因此输出 obj.a。

```js
obj.test1.call(obj2); // obj2.a
```

使用了 call 方法指定调用对象为 obj2，因此输出 obj2.a，这里 obj.test1 返回的是一个函数，这个函数是由 obj2 调用的。

test2 中，箭头函数中的 this 与调用者无关，它指向定义这个箭头函数的位置的上下文环境中的 this 对象。根据前面的作用域的介绍，可以知道 test2 中的上下文是全局，因为它并不定义在函数作用域中。

```js
obj.test2(); // global.a
```

上下文的 this 是全局，因此输出 global.a。

```js
obj.test2.call(obj2); // global.a
```

与调用者无关，因此还是输出 global.a。

这两种是最基本的情况，接下来介绍的是稍微复杂一些的情况，也就是存在闭包的情况。其实闭包的情况也可以一层一层地分析。

### 普通函数形式闭包

这两种情况都是外层是普通函数，内层分别是普通函数和箭头函数。这两种情况中，首先可以分析最外层的函数里的 this 是指向谁，然后再分析内层。

```js
this.a = 'global.a';

const obj = {
  a: 'obj.a',
  test3: function () {
    return function () {
      console.log(this.a);
    };
  },
  test4: function () {
    console.log(this.a);
    return () => {
      console.log(this.a);
    };
  },
};

const obj2 = { a: 'obj2.a' };

obj.test3()(); // 浏览器中：global.a，Node中：undefined
obj.test3().call(obj2); // obj2.a
obj.test3.call(obj2)(); // 浏览器中：global.a，Node中：undefined

obj.test4()(); // obj.a
obj.test4().call(obj2); // obj.a
obj.test4.call(obj2)(); // obj2.a
```

test3 是普通函数包普通函数，其实这种情况下不论套多少层，都很容易分析，因为普通函数中的 this 一定是指向调用它的对象的，只需要关注最后是谁调用它即可。**注意：这里有一个特殊情况，就是浏览器环境和 Node 环境的输出结果有所不同，[原因](./浏览器环境和Node环境下的全局this.md)，下面以浏览器环境作为例子。**

```js
obj.test3()(); // global.a
```

这里其实可以分为两步来分析，首先`obj.test3()`返回了一个普通匿名函数，接着在全局环境下调用了这个匿名函数，因此输出 global.a。

```js
obj.test3().call(obj2); // obj2.a
```

同上，只不过最后通过 call 方法指定了调用者为 obj2，因此输出为 obj2.a。

```js
obj.test3.call(obj2)(); // global.a
```

这里是先指定了 obj2 来调用外层函数，返回普通匿名函数，然后在全局环境下调用了这个匿名函数，因此输出 global.a。实际上外层函数是谁调用并不影响内层函数，因为内层函数内部的 this 只指向调用内层函数的对象。

test4 中则稍有不同，因为内层函数是箭头函数了，这就意味着内层函数内部的 this 是指向外层函数的 this 的，因此只要分析出外层函数内部的 this 指向就可以知道内层函数内部的 this 指向了。

```js
obj.test4()(); // obj.a
```

这里 obj 对象调用了外层函数，因此外层函数内部的 this 指向的就是 obj，因此内层函数内部的 this 也是指向 obj，因此输出 obj.a。

```js
obj.test4().call(obj2); // obj.a
```

同上，因为最后内层函数的 this 指向与调用者无关，因此还是输出 obj.a。

```js
obj.test4.call(obj2)(); // obj2.a
```

这里就不一样了，因为 obj2 调用了外层函数，因此外层函数内部的 this 指向了 obj2，所以内层函数内部的 this 也指向 obj2，因此输出 obj2.a。

### 箭头函数形式闭包

这两种情况都是外层是箭头函数，内层分别是普通函数和箭头函数。这两种情况的分析方法和上面类似，同样是从外层到内层分析。这里也同样只分析浏览器环境下的情况。

```js
this.a = 'global.a';

const obj = {
  a: 'obj.a',
  test5: () => {
    return function () {
      console.log(this.a);
    };
  },
  test6: () => {
    return () => {
      console.log(this.a);
    };
  },
};

const obj2 = { a: 'obj2.a' };

obj.test5()(); // 浏览器中：global.a，Node中：undefined
obj.test5().call(obj2); // obj2.a
obj.test5.call(obj2)(); // 浏览器中：global.a，Node中：undefined

obj.test6()(); // global.a
obj.test6().call(obj2); // global.a
obj.test6.call(obj2)(); // global.a
```

test5 是箭头函数包普通函数，因此实际上内层函数内部的 this 指向与外层无关，可以直接分析最后是谁调用内层函数即可。

```js
obj.test5()(); // global.a
```

这里最后是全局对象调用内层的匿名函数，因此输出 global.a。

```js
obj.test5().call(obj2); // obj2.a
```

这里是由 obj2 调用内层匿名函数，因此输出 obj2.a。

```js
obj.test5.call(obj2)(); // global.a
```

这里是先由 obj2 调用外层函数得到内层匿名函数，然后由全局对象调用内层匿名函数，因此输出 global.a。

test6 是箭头函数包箭头函数，因此内层函数内部的 this 指向外层函数内部的 this。而外层函数本身也是箭头函数，因此它的 this 又指向定义它的位置的上下文中的 this，这里也就是全局对象了，因此所有输出都是 global.a。

```js
obj.test6()(); // global.a
obj.test6().call(obj2); // global.a
obj.test6.call(obj2)(); // global.a
```
