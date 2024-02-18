---
title: Node.js笔记
date: 2022-02-09 15:51:59
keywords: nodejs Nodejs
categories: "前端"
tags:
  - 青训营
  - Node.js
---

# Node.js 笔记

参加字节跳动的青训营时写的笔记。这部分是欧阳亚东老师讲的课。

## 1. 应用场景

- 前端工程化
- Web 服务端应用
  - 运行效率接近常见的编译语言
  - 社区生态丰富、工具链成熟(npm，V8 inspector)
  - 与前端结合的场景有优势(服务端渲染 SSR)
- Electron 跨端桌面应用
  - 商业应用：vscode, slack, discord
  - 大型公司内的效率工具

## 2. 运行时结构

- V8：JavaScript Runtime，诊断调试工具(inspector)
- libuv：eventloop(事件循环)，syscall(系统调用)

### 2.1 特点

- 异步 I/O：当 Node.js 执行 I/O 操作时，会在响应返回后恢复操作，而不需要阻塞线程(占用额外线程)。

- 单线程：

  - JS 单线程

    - 实际：JS 线程+uv 线程池+V8 任务线程池+V8 inspector 线程

  - worker_thread 可以单独起独立线程，每一个线程的模型没有太大变化

  - 优点：不需要考虑多线程状态同步问题(不需要锁)，能够高效地利用系统资源
  - 缺点：阻塞会产生更多的负面影响， 解决方法：多进程或多线程

- 跨平台(大部分功能， api)：开发成本低(大部分情景不需要考虑跨平台问题)，学习成本低
  - Node.js 跨平台 + JavaScript 无需编译环境 + Web 跨平台 + 诊断工具跨平台

## 3. 编写 Http Server

[之前的笔记](https://13535944743.github.io/2021/11/18/node/)

### 3.1 Hello

```js
const http = require("http");

const port = 8081;

const server = http.createServer((req, res) => {
  res.end("Hello");
});

server.listen(port, () => {
  console.log(`Listen at ${port}`);
});
```

![image-20220120145018966](https://s2.loli.net/2022/02/09/NePcG1S2Io8QT5b.png)

### 3.2 JSON 数据

用户把 JSON 数据 POST 给服务器，服务器再把数据中的 msg 取出来，返回给用户

**服务器端**：

```js
const http = require("http");

const server = http.createServer((req, res) => {
  const bufs = [];
  req.on("data", (buf) => {
    bufs.push(buf); // 把数据收集起来
  });

  req.on("end", () => {
    const buf = Buffer.concat(bufs).toString("utf8");
    let msg = "Hello";
    try {
      const ret = JSON.parse(buf); // 转换成JSON对象
      msg = ret.msg;
    } catch (err) {
      // 如果抛出异常的话，则msg是初始值Hello，无需处理异常
    }

    const responseJson = {
      msg: `receive: ${msg}`,
    };
    res.setHeader("Content-Type", "application/json");
    res.end(JSON.stringify(responseJson)); // JSON对象转换为字符串
  });
});

const port = 8081;

server.listen(port, () => {
  console.log(`Listen at ${port}`);
});
```

**客户端**：

```js
const http = require("http");

const body = JSON.stringify({
  msg: "Hello from client",
});

const req = http.request(
  "http:/127.0.0.1:8081",
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
  },
  (res) => {
    const bufs = [];
    res.on("data", (buf) => {
      bufs.push(buf);
    });

    res.on("end", () => {
      const buf = Buffer.concat(bufs);
      const json = JSON.parse(buf);
      console.log("json msg is: ", json.msg);
    });
  }
);

req.end(body);
```

先打开服务器端，再打开客户端。(第一个文件为 json.js，第二个为 client.js。则先执行` node json.js`，再执行` node client.js`)

收到返回信息：

![image-20220120190645660](https://s2.loli.net/2022/02/09/vVEG134JCUOFs5e.png)

### 3.3 用 Promise + async await 重写 3.2

技巧：将 callback 转换成 promise

不是所有的回调函数都适合转换成 promise，而是只调用一次的回调函数才适合转换为 promise。即 createServer()不适合转换为 promise。

json.js 修改后(输出结果一样)

```js
const http = require("http");

const server = http.createServer(async (req, res) => {
  // 从客户端接收数据
  const msg = await new Promise((resolve, reject) => {
    const bufs = [];

    req.on("error", (err) => {
      reject(err);
    });

    req.on("data", (buf) => {
      bufs.push(buf); // 把数据收集起来
    });

    req.on("end", () => {
      const buf = Buffer.concat(bufs).toString("utf8");
      let msg = "Hello";
      try {
        const ret = JSON.parse(buf); // 转换成JSON对象
        msg = ret.msg;
      } catch (err) {
        // console.log(err);
      }

      resolve(msg); // 返回msg
    });
  });

  // 响应
  const responseJson = {
    msg: `receive: ${msg}`,
  };
  res.setHeader("Content-Type", "application/json");
  res.end(JSON.stringify(responseJson));
});

const port = 8081;

server.listen(port, () => {
  console.log(`Listen at ${port}`);
});
```

### 3.4 静态文件服务

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <p>Hello</p>
    <script>
      alert("Hello");
    </script>
  </body>
</html>
```

static_server.js

```js
const http = require("http");
const fs = require("fs");
const path = require("path");
const url = require("url");

const folderPath = path.join(__dirname, "static");

const server = http.createServer((req, res) => {
  const info = url.parse(req.url);
  const filePath = path.join(folderPath, info.path);

  const filestream = fs.createReadStream(filePath);
  // createReadStream好处: 减少占用内存空间

  filestream.pipe(res);
});

const port = 8081;

server.listen(port, () => {
  console.log(`Listen at ${port}`);
});
```

![image-20220120201216995](https://s2.loli.net/2022/01/26/1pfIKGNvWD38HjU.png)

执行` node static_server.js` 后，打开 http://localhost:8081/index.html

![image-20220120201343572](https://s2.loli.net/2022/02/09/OQagcVCHJFqjykA.png)

之后会报点小错，因为没有 ico 图标(忽视就好)

### 3.5 React SSR

SSR(server side rendering)：服务端渲染

- 相对于传统 HTML 模板引擎：可以避免重复编写代码
- 相比于 SPA：首屏渲染更快，SEO 友好(SPA 应用需要加载完所有的 js 代码后，才可以给用户返回数据)

<b style="color: red">首先要先安装 react 相关的包</b>，` npm i react react-dom`

下面就是通过 React SSR 实现显示 Hello 的代码(有一点不太明白，还是得等会用 ReactDOM、ReactDOMServer 模块)

ssr.

```js
const React = require("react");
const ReactDOMServer = require("react-dom/server");

const http = require("http");

function App(props) {
  return React.createElement("div", {}, props.children || "Hello");
}

const port = 8081;

const server = http.createServer((req, res) => {
  res.end(`
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <title>App</title>
      </head>
      <body>
        ${ReactDOMServer.renderToString(React.createElement(App, {}, "Hello"))}
      </body>
    </html>
  `);
});

server.listen(port, () => {
  console.log(`Listen at ${port}`);
});
```

### 3.6 Debug

V8 Inspector：开箱即用、与前端开发已知、跨平台

场景：

- 查看 console.log 内容
- breakpoint
- 性能分析

使用：

1. `node --inspect ssr.js`

2. 打开http://127.0.0.1:9229/json

3. 复制打开下图红框内容

   ![image-20220120204024901](https://s2.loli.net/2022/01/26/iyUwYbxmZqzfe2L.png)

4. 进入下图界面

   ![image-20220120204058030](https://s2.loli.net/2022/01/26/LqloF6ZD1JUEBbm.png)

   控制台显示连接上

   ![image-20220120204152109](https://s2.loli.net/2022/01/26/THy9sv7ERPwlO3i.png)

5. 对于阻塞的情况，可以使用 log point，相当于 console.log

   ![动画](https://s2.loli.net/2022/02/09/oCiFzcSsamvT9u2.gif)
