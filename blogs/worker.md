---
title: nodejs 使用 Worker、Web Worker使用
categories: node.js
date: 2024-02-29 20:41:00
tags:
  - node.js
  - javascript
  - Worker
  - Web Worker
---

# nodejs 使用 Worker、Web Worker 使用

## 前言

> 之前用 ffmpeg 跟 worker 实现批量合成视频时就有用到 worker 来实现，最近弄一个批量转音频格式又用到了 ffmpeg。然后发现不用 worker 根本行不通，电脑直接卡死。简单记录一下 worker 的用法。

## 使用步骤

**主线程**：

1. 通过创建一个 Worker 实例，参数是 worker 脚本的路径
2. 通过`worker.postMessage`给 worker 线程传递数据
3. 绑定`message`事件监听，监听到完成后通过`worker.terminate()`结束 worker 线程。

**Worker 线程**：

1. 引入`parentPort`，绑定`message`监听事件，参数就是主线程通过`worker.postMessage`传递的参数
2. 任务执行完毕后，通过`parentPort.postMessage()`给主线程传递信息。

## 实践

> 处理一千个数字相加，并且模拟每次相加都需要额外耗时。（_我的电脑太拉了，用公司的 mac 实际可以处理一万个数字，并且大概需要 10s_）

### 直接处理

首先，先不使用 Worker 多线程来看看效率如何。

```js
const wait1ms = () => {
  return new Promise((resolve) => {
    setTimeout(() => resolve(), 1);
  });
};

(async () => {
  const numbers = Array(1000)
    .fill(0)
    .map((value, index) => index);

  console.time();

  let sum = 0;
  for (let i = 0; i < numbers.length; i++) {
    await wait1ms();
    sum += numbers[i];
  }

  console.log(sum);
  console.timeEnd();
})();
```

![](https://www.clzczh.top/CLZ_img/images/202402292032340.png)

> 差不多 15s 这样子

### 使用 Worker 多线程处理

> 把任务分给 32 个 worker 线程。**注意，一般不会开启这么多 worker 线程，一般太多会浪费资源之类的。但是在这个 demo 中，使用 32 个线程效率很高，更能衬托出 worker 线程的强大就这么弄了。**

`index.js`

```js
const { Worker } = require("worker_threads");

(async () => {
  const numbers = Array(10000)
    .fill("")
    .map((value, index) => index);

  console.time();

  const workerNum = 7;
  const count = Math.ceil(numbers.length / workerNum);

  let sum = 0;
  let completeWorkerNum = 0;

  while (numbers.length > 0) {
    const workerData = numbers.splice(0, count);
    const worker = new Worker("./worker.js");
    worker.postMessage(workerData);

    worker.on("message", (msg) => {
      sum += msg;
      completeWorkerNum++;

      if (completeWorkerNum === workerNum) {
        console.log(sum);
        console.timeEnd();
      }

      worker.terminate();
    });
  }
})();
```

`worker.js`

```js
const { parentPort } = require("worker_threads");

const wait1ms = () => {
  return new Promise((resolve) => {
    setTimeout(() => resolve(), 1);
  });
};

parentPort.on("message", async (numbers) => {
  let sum = 0;
  for (let i = 0; i < numbers.length; i++) {
    await wait1ms();
    sum += numbers[i];
  }

  parentPort.postMessage(sum);
});
```

![](https://www.clzczh.top/CLZ_img/images/202402292038556.png)

> 😔。用自己的电脑真的跟用公司电脑跑差太多了。公司电脑处理一万条数据，一样 32 个 worker 线程，只需要几百毫秒。

### Web Worker 使用

上面的 Worker 使用是在 nodejs 环境中运行的，实际上还可以在浏览器环境中使用 Web Worker。

使用方式跟 nodejs 环境的类似，有三点区别：

1. nodejs 绑定监听事件是`.on('message', () => {})`的形式绑定的，而浏览器则还是`addEventListener`或者`onmessage`那一套
2. nodejs 监听`message`事件，参数就是传递的数据，而浏览器的参数是`event`对象，`event`的`data`属性才是数据
3. `nodejs`的`workerjs`是引入`parentPort`进行事件的监听，而浏览器可以直接使用`self`来进行监听
   > $\color{red}{Worker线程的全局对象是`self`，而不是浏览器环境中的`window`}$

代码：
html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>

  <body>
    <script>
      (async () => {
        const numbers = Array(10000)
          .fill("")
          .map((value, index) => index);

        console.time();

        const workerNum = 7;
        const count = Math.ceil(numbers.length / workerNum);

        let sum = 0;
        let completeWorkerNum = 0;

        while (numbers.length > 0) {
          const workerData = numbers.splice(0, count);
          const worker = new Worker("./worker.js");
          worker.postMessage(workerData);

          worker.onmessage = (event) => {
            sum += event.data;
            completeWorkerNum++;

            if (completeWorkerNum === workerNum) {
              console.log(sum);
              console.timeEnd();
            }

            worker.terminate();
          };
        }
      })();
    </script>
  </body>
</html>
```

worker.js

```js
const wait1ms = () => {
  return new Promise((resolve) => {
    setTimeout(() => resolve(), 1);
  });
};

self.addEventListener("message", async (event) => {
  const numbers = event.data;

  let sum = 0;
  for (let i = 0; i < numbers.length; i++) {
    await wait1ms();
    sum += numbers[i];
  }

  self.postMessage(sum);
});
```
