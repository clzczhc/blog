---
title: nodejs 使用 Worker
categories: node.js
date: 2024-02-29 20:41:00
tags:
  - node.js
  - javascript
  - Worker
---

# nodejs 使用 Worker

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

> 把任务分给 32 个 worker 线程。**注意，一般不会开启这么多worker线程，一般太多会浪费资源之类的。但是在这个demo中，使用32个线程效率很高，更能衬托出worker线程的强大就这么弄了。**

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
