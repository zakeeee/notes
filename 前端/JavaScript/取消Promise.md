# 取消 Promise

```js
let cancelFunction;
// Promise 的构造函数中的回调函数是同步执行的
let promise = new Promise((resolve, reject) => {
  const poller = setInterval(() => {
    console.log('poll');
  }, 1000);
  cancelFunction = () => {
    clearInterval(poller);
    reject('cancelled');
  };
});
promise.catch((err) => {
  console.error(err);
});
cancelFunction();
```
