---
title: 详解JavaScript 执行机制
date: 2022-03-14 12:27:51
categories: 前端
tags:
  - JavaScript
---

# 详解 JavaScript 执行机制

## 热身

```js
/* 先打印1， 3， 2s后打印2 */
console.log(1);
setTimeout(() => {
  console.log(2);
}, 1000);
console.log(3);

/* 先打印1， 3， 后打印2 */
console.log(1);
setTimeout(() => {
  console.log(2);
}, 0);
console.log(3);
```

第一个例子的话不难理解，定时器函数就是 1s 后才调用回调函数.

而第二个例子则可能优点小问题，JavaScript 从上到下执行，那么遇到 0s 的计时器函数，就应该先输出 2 才对啊。这就是因为后面要提到的 JavaScript 执行机制导致的啦，因为 setTimeout 是异步任务。

## JavaScript 是单线程

JavaScript 的核心特征就是**单线程**，即同一时间只能做一件事。

为什么它是单线程呢？因为 JavaScript 作为浏览器脚本语言，它的主要用途就是与用户互动、操作 DOM。既然如此，如果它不是单线程的话，假如一个线程在 DOM 节点上添加内容，同时另一个线程删除这个节点。可以看出，如果 JavaScript 不是单线程的话，那么将会导致同步问题。

<br>

## 任务队列

JavaScript 是单线程语言，这也就导致了如果有一个任务等待很长的时间，这个时候就会导致阻塞，程序就会“卡死”，用户体验非常差。所以 JavaScript 需要异步任务。

<br>

那么，为什么 JavsScript 明明是单线程的，为什么能异步呢？这是因为浏览器是多线程的，通过事件循环` Event Loop`即可实现异步。

<br>

所有任务都可以分成两种。

- **同步任务**：在主线程上排队执行的任务，只有前一个任务执行完，才能执行后一个任务
- **异步任务**：不进入主线程，而是进入任务队列的任务。<b style="color: red">当异步任务的触发条件满足时，异步任务才会进入任务队列，而当主线程空了，就会去任务队列中取异步任务到主线程中执行</b>

<br>

**常见异步任务**：

- JS 事件
- AJAX 请求
- setTimeout 和 setInterval
- Promise(<b style="color: red">Promise 定义部分为同步任务，回调部分为异步任务</b>)

<br>

## Event Loop 事件循环机制 1

1. 所有同步任务进入主线程，而异步任务则是进入` Event Table`注册回调函数
2. 当异步任务的<b style="color: red">触发条件满足</b>时，异步任务注册的回调函数将会从` Event Table`移入到任务队列` Event Queue`中
3. 当主线程中的所有同步任务执行完毕后，系统就会去看看` Event Queue`中看看有没有回调函数，有的话就推到主线程中
4. 主线程不断重复上面的步骤

![preview](https://s2.loli.net/2022/03/14/hANCZjRUYIsFrgf.jpg)

这就是**Event Loop**

## 宏任务和微任务

异步任务又可以进行更精细的划分为宏任务和微任务。

### 宏任务

setTimeout、setInterval、requestAnimationFrame

- 当宏任务队列中的任务全部都执行完之后，如果微任务队列不为空，则先执行微任务队列中的所有任务

### 微任务

Promise 回调部分、process.nextTick

- 在上一个宏任务队列执行完毕后如果有微任务就会执行微任务队列中的所有元素

## Event Loop 事件循环机制 2

1. 首先执行` script`下的同步任务
2. 执行过程中，如果遇到异步任务，则需要把它放到对应的任务队列中(遇到宏任务，则放到宏任务中；遇到微任务，则放到微任务队列中)
3. 同步任务执行完毕，查看微任务队列
   - 如果存在微任务，则将微任务队列全部执行(<b style="color: red">包括执行微任务中产生的新微任务</b>)
   - 如果不存在微任务，则查看宏任务队列，执行第一个宏任务，宏任务执行完后，又看看微任务队列是否有任务，有的话，又先全部执行完微任务队列，重复上述操作，知道宏任务队列为空。

![preview](https://s2.loli.net/2022/03/14/xzo2leVm165AkLd.png)

### 练手 1

```js
console.log(1);

setTimeout(() => {
  console.log(2);
}, 0);

new Promise((resolve, reject) => {
  for (let i = 3; i < 6; i++) {
    console.log(i);
  }
  resolve();
}).then(() => {
  console.log(6);
});

console.log(7);
```

打印顺序：1, 3, 4, 5, 7, 6, 2

<br>

解析：

1. 首先，程序从上往下走，直接输出 1，遇到` setTimeout`后，把它放到宏任务队列中
   - 此时，宏任务队列中为`[setTimeout]`(**这里用数组表示任务队列，左边代表先进入的任务队列**)
2. 继续往下跑，遇到` Promise`，因为<b style="color: red">Promise 定义部分为同步任务</b>，依次输出 3, 4, 5，遇到` Promise.then()`，把它放到微任务队列中
   - 此时，宏任务队列为`[setTimeout]`
   - 此时，微任务队列为`[Promise.then()]`
3. 输出 7 后，执行微任务队列中全部的任务，输出 6， 再执行宏任务队列中的任务，输出 2

### 练手 2

**题目是本人自己想的，分析有误请见谅(希望评论指示)**

```js
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3);
  });
}, 10);

const p = new Promise((resolve, reject) => {
  console.log(4);
  resolve(5);
})
  .then((value) => {
    console.log(value);
    Promise.resolve().then(() => {
      console.log(6);
    });
    setTimeout(() => {
      console.log(7);
    });
  })
  .then(() => {
    console.log(8);
  });

console.log(9);
```

输出顺序：1, 4, 9, 5, 6, 8, 7, 2, 3

<br>

解析：

1. **先输出 1**， 遇到定时器，但是此时并不满足触发条件，所以 2(后面还有其他的内容)只能存放在` Event Table`中
   - 此时，` Event Table`中有` [2(后面还有其他的内容)]`。<b style="color: red">` Event Table`中没有顺序，满足触发条件后，就会进入对应的任务队列</b>
2. **输出 4**，5(后面还有内容)进入微任务队列
   - 此时，` Event Table`中有` [2(后面还有其他的内容)]`
   - 微任务队列为[5(后面还有内容)]
3. 8 进入微任务队列
   - 此时，` Event Table`中有` [2(后面还有其他的内容)]`
   - 微任务队列为[5(后面还有内容), 8]
4. **输出 9**， 然后执行微任务中的任务
   1. **输出 5**， 6 进入微任务队列, 7 进入宏任务队列
      - 此时，` Event Table`中有` [2(后面还有其他的内容)]`
      - 微任务队列为`[6, 8]`，<b style="color: red">6 会在 8 之前，因为 6 是微任务队列 5 里的微任务</b>
      - 宏任务队列为[7]
   2. 依次执行完微任务队列中的任务，然后再执行宏任务队列的任务。**输出顺序为 6，8，7**
5. 10ms 后，满足触发条件，进入宏任务队列，此时，宏任务队列和微任务队列中都没有任务，所以直接执行。**输出 2**，3 进入微任务队列，**输出 3**

<br>

## async, await

> async/await 本质上还是基于 Promise 的一些封装，而 Promise 是属于微任务的一种。所以在使用 await 关键字与 Promise.then 相同。
> async 函数在 await 之前的代码都是同步执行的，await 之后的代码则是属于微任务(类似于 Promise)<b style="color: red">await 的表达式还是属于同步任务</b>

下面就继续练手

```js
console.log(1);

async function async1() {
  console.log(2);
  await async2();
  console.log(3);
}
async1();

async function async2() {
  console.log(4);

  await new Promise((resolve, reject) => {
    console.log(5);
    resolve(6);
  }).then((value) => {
    console.log(value);
  });

  console.log(7);
}

console.log(8);
```

输出顺序为：1, 2, 4, 5, 8, 6, 7, 3

<br>

分析：

1. **先输出 1**，调用` async1`函数，因为` await`之前包括` await`的表达式都是同步任务，所以，**输出 2**后，进入到` async2`函数中
2. **输出 4**，`await`一个` Promise`也是同理，**输出 5**，6 进入微任务队列，因为<b style="color: red">await 之后的代码则是属于微任务(不包括 await 的表达式)</b>，所以 7 进入微任务队列
   - 此时，微任务队列为[6, 7]
3. 执行完` async2`函数后，回到` async1`函数中，之后的 3 进入微任务队列
   - 此时，微任务队列为[6, 7, 3]
4. **输出 8**，执行微任务队列中的任务，**输出 6, 7, 3**

<br>

参考链接：
[JavaScript 运行机制详解：再谈 Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
[彻底搞懂 JavaScript 执行机制](https://zhuanlan.zhihu.com/p/379475079)
[JavaScript 之彻底理解 EventLoop](https://juejin.cn/post/7020328988715270157)
[10 分钟理解 JS 引擎的执行机制](https://segmentfault.com/a/1190000012806637)
