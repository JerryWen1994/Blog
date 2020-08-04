# Promise 的 all、race、allSettled 方法

## Promise.all

### all 介绍

接收一个 Promise 数组，返回一个结果数组，只有传给 Promise.all 的参数数组中所有 Promise 全部 resolve 之后才返回一个结果数组，值为所有 promise resolve 返回的数据；如果其中一个 promise 返回 reject，则返回的 Promise 对象值为 reject reason

### all 运用

有的时候我们在执行多个异步请求时，我们需要等待所有请求成功后才进行下一步操作，任何一个请求失败都不继续操作，这时就可以使用 Promise.all 操作

### all 实现

```js
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError("arguments must be an array"));
    }
    let resolveCount = 0,
      resolveValues = [];
    for (let i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i]).then(
        (res) => {
          resolveCount++;
          resolveValues[i] = res;
          if (resolveCount === promises.length) {
            return resolve(resolveValues);
          }
        },
        (reason) => {
          return reject(reason);
        }
      );
    }
  });
}
```

## Promise.race

### race 介绍

同 all 一样接受一个数组，该数组里面都是 promise 对象。区别在于，这里返回的 Promise 对象是最先 resolve 返回的 promise 数据。

### race 运用

在多个 Promise 的并发情况下，返回最先 resolve 的 Promise 对象

### race 实现

```js
function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError("arguments must be an array"));
    }
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(resolve, reject);
    }
  });
}
```

## Promise.allSettled

### allSettled 介绍

同 all 一样接受一个数组，该数组里面都是 promise 对象。区别在于，这里返回的 Promise 对象是不区分 promise 数组中执行返回的状态，无论成功或失败，只要状态值改变，都会继续操作，并在最终返回的结果数组中，每一项都包含 status 和 value 或者 reason。

### allSettled 运用

在多个 promise 的并发情况下，我们不需要考虑每个 promise 的执行成功或是失败，只要所有 promise 都执行完毕，我们就进行下一步操作。

### allSettled 实现

```js
function promiseAllSettled(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError("arguments must be an array"));
    }
    let resolveCount = 0,
      promiseCount = promises.length,
      values = [];
    for (let i = 0; i < promiseCount; i++) {
      Promise.resolve(promises[i]).then(
        (res) => {
          resolveCount++;
          values[i] = { value: res, status: "fulfilled" };
          if (resolveCount === promiseCount) {
            return resolve(values);
          }
        },
        (reason) => {
          resolveCount++;
          values[i] = { reason: reason, status: "rejected" };
          if (resolveCount === promiseCount) {
            return resolve(values);
          }
        }
      );
    }
  });
}
```
