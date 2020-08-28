# 浏览器环境和 Node 环境下的全局 this 问题

浏览器中，全局环境下的 this 指向的其实是 Window 对象。

Node 环境下则有所不同，因为每个 js 文件是一个模块，因此每个 js 文件中定义在最外层的 this 并不是真正的全局 this，而是 module.exports。全局对象是一个特殊的称为 global 的对象。

```js
function foo() {
  console.log(this);
}

foo(); // Object [global] {...}

console.log(this === module.exports); // true
console.log(this); // {}
this.a = 'a';
console.log(module.exports); // { a: 'a' }
```
