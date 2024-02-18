---
title: JS手撕(九) 常用Promise API
categories: 前端
date: 2022-12-24 12:46:37
tags:
    - JavaScript
    - 手撕
---

# JS手撕(九) 常用Promise API

## 前言

上一篇已经手撕了一个简单版本的`Promise`。现在就在继续手撕常用的`Promise API`。

## Promise.resolve()

`Promise.resolve(value)`方法返回一个以给定值解析后的`Promise`对象。如果`value`是`Promise`对象，则直接返回该`promise`。否则返回一个新的`Promise`对象。

```js
Promise.myResolve = function (value) {
  if (value instanceof Promise) {
    return value;
  }

  return new Promise(resolve => resolve(value));
}
```

测试

```js
// 1. 普通值
Promise.myResolve(123).then(console.log);

// 2. 成功状态的Promise
const p1 = new Promise(resolve => resolve("11111"));
Promise.myResolve(p1).then(console.log);

// 3. 失败状态的Promise
const p2 = new Promise((resolve, reject) => reject("error"));
Promise.myResolve(p2).catch(console.log);

// 4. 不传参
Promise.myResolve().then(console.log);

// 5. thenable对象
const p3 = {
  then(resolve) {
    setTimeout(() => resolve(4), 1000)
  }
}
Promise.myResolve(p3).then(console.log) 
```

换成原生的结果也一样。

## Promise.reject()

`Promise.reject()` 方法返回一个带有拒绝原因的 `Promise` 对象。所以只需要直接返回一个新的`Promise`对象就行了。

```js
Promise.myReject = function (value) {
  return new Promise((resolve, reject) => reject(value));
}
```

测试：

```js
// 1. 普通值
Promise.myReject(123).catch(console.log);

// 2. 成功状态的Promise
const p1 = new Promise(resolve => resolve("11111"));
Promise.myReject(p1).catch(console.log);

// 3. 失败状态的Promise
const p2 = new Promise((resolve, myReject) => myReject("error"));
Promise.myReject(p2).catch(console.log);

// 4. 不传参
Promise.myReject().catch(console.log); 
```

## Promise.all()

参数：promises，promise 的`iterable`类型(`Array`、`Map`、`Set`)

返回一个新的 Promise，当所有的 promise 都成功才成功，且结果为成功的结果组成的数组；有一个失败就**直接**失败，返回的结果就是失败的那一个的结果。(如果有多个，则返回第一个错误的)

```js
const p1 = new Promise((resolve, reject) => {
  resolve("p1: OK");
});
const p2 = Promise.resolve("p2: OK");
const result1 = Promise.all([p1, p2]); // 所有Promise的结果都成功
console.log(result1);

const p3 = Promise.resolve("p3: OK");
const p4 = Promise.reject("p4: Err");
const p5 = Promise.reject("p5: Err");
const result2 = Promise.all([p3, p4, p5]); // 有Promise的结果失败
console.log(result2);
```

实现起来其实就是遍历`promises`，并且用一个数组来存成功的结果，当数组的长度等于`promises`的长度才调用`resolve()`方法，遇到错误的结果的话，直接调用`reject()`方法结束。

```js
Promise.myAll = function (promises) {
  return new Promise((resolve, reject) => {
    const result = [];

    promises.forEach(promise => {
      promise.then(
        (v) => {
          result.push(v);

          if (result.length === promises.length) {
            resolve(result);
          }
        },
        (r) => reject(r)
      );
    })
  })
}
```

## Promise.race()

参数：promises，promise 的`iterable`类型(`Array`、`Map`、`Set`)

返回一个新的 Promise，第一个完成的结果是成功则成功，反之则失败。

有前面的`Promise.all()`的经验就能发现，`race()`和`all()`很像，其实就只是`all()`如果遇到成功的，需要存储成功的结果。而`race()`不需要，第一个完成的结果就直接结束，不会再遍历后面的了。

```js
Promise.myRace = function (promises) {
  return new Promise((resolve, reject) => {
    promises.forEach(promise => {
      promise.then(resolve, reject);
    })
  })
}
```

测试：

```js
const p1 = new Promise(resolve => {
  setTimeout(() => {
    resolve("p1: OK");
  });
});

const p2 = Promise.reject("p2: Err");

Promise.myRace([p1, p2])
  .then(console.log, console.log);    // p2: Err。因为p2先得到结果



const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("p3: OK");
  });
});

const p4 = Promise.resolve("p4: ok");

Promise.myRace([p3, p4])
  .then(console.log, console.log);  // p4: ok。因为p4先得到结果
```

## 参考

[Promise学习笔记(一) | 赤蓝紫](https://www.clzczh.top/2022/03/05/promise-1)

[GitHub - qianlongo/fe-handwriting: 手写各种js Promise、apply、call、bind、new、deepClone....](https://github.com/qianlongo/fe-handwriting)
