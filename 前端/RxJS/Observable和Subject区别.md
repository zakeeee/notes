# Observable 和 Subject 的区别

## Observable

```js
import { Observable } from 'rxjs';

let obs = Observable.create((observer) => {
  observer.next(Math.random());
});

obs.subscribe((res) => {
  console.log('subscription a :', res); //subscription a :0.2859800202682865
});

obs.subscribe((res) => {
  console.log('subscription b :', res); //subscription b :0.694302021731573
});
```

- 延迟启动：只有当有 Observer 订阅时，Observable 才会开始发布数据。
- 单向传播：数据只能由 Observable 传给 Observer，反过来不行。
- 单播（uni-cast）：每有一个 Observer 订阅，都会使得回调函数被执行一次。并且订阅了 Observable 的每一个 Observer 都会获得一份独立的数据，彼此不会影响。
- 获取所有数据：Observer 订阅时会从头开始获取 Observable 发出的所有数据。

## Subject

```js
import { Subject } from 'rxjs';

let obs = new Subject();

obs.subscribe((res) => {
  console.log('subscription a :', res); // subscription a : 0.91767565496093
});

obs.subscribe((res) => {
  console.log('subscription b :', res); // subscription b : 0.91767565496093
});

obs.next(Math.random());
```

- 立即启动：即使没有 Observer 订阅，数据也可以发布，比如可以调用 `next()` 来让 Subject 发布一个数据。
- 双向传播：可以向 Subject 传递数据。
- 多播（multi-cast）：订阅了同一个 Subject 的所有 Observer，在 Subject 发布数据时，都会获得相同数据。
- 丢失历史数据：Observer 订阅 Subject 时会丢失 Subject 已经发出过的数据，除非用 BehaviorSubject，ReplaySubject 等。

## 总结

|              |          Observable          |                       Subject                       |
| :----------- | :--------------------------: | :-------------------------------------------------: |
| 数据发布时机 |     有订阅后数据才会发布     |            创建后即可调用 next 发布数据             |
| 传播方向     |        单向，只出不进        |                        双向                         |
| 使用场景     |             单播             |                        多播                         |
| 数据         | 每次订阅都获取所有发布的数据 | 丢失历史数据，除非用 BehaviorSubject，ReplaySubject |
