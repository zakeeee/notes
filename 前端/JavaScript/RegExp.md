# RegExp

使用 RegExp 对象时，如果已经匹配了一次，下一次再使用这个对象进行匹配测试时会出错，应该重新创建一个 RegExp 对象。

```js
// 如果不重新创建实例
let pattern1 = new RegExp(".at", "g")
console.log(pattern1.test("cat"))  // true
console.log(pattern1.test("bat"))  // false

// 重新创建实例
let pattern2 = new RegExp(".at", "g")
console.log(pattern2.test("cat"))  // true
pattern2 = new RegExp(".at", "g")
console.log(pattern2.test("bat"))  // true
```
