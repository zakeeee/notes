# 取消 Promise

```js
function CancelToken(cb) {
  return function () {
    cb('cancelled');
  };
}
let cancel;
let promise = new Promise((resolve, reject) => {
  const poller = setInterval(() => {
    console.log('poll');
  }, 1000);
  cancel = new CancelToken(() => {
    clearInterval(poller);
    reject('cancelled');
  });
});
promise.catch((err) => {
  console.error(err);
});
promise.cancel = cancel;
promise.cancel();
```
