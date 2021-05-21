# 取消 Promise

```js
let cancelFunction;

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
