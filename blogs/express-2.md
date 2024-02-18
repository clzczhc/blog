---
title: Express(二) ——中间件
date: 2022-03-01 14:08:48
keywords: Express
categories: "后端"
tags:
  - Express
---

# Express(二) ——中间件

在 Express 中，中间件是一个可以访问请求对象、响应对象和调用 next 方法的一个函数。

## 1. 简单例子(打印请求日志)

一个 Express 应用，就是由许许多多的中间件来完成的。

```js
const express = require("express");

const app = express();

app.use((req, res, next) => {
  console.log(`请求日志：${req.method} ${req.url} ${new Date()}`);
  next(); // 放行
});

app.get("/", (req, res) => {
  res.send("get / ");
});

app.post("/", (req, res) => {
  res.send("post / ");
});

app.delete("/", (req, res) => {
  res.send("delete /");
});

const port = 3000;

app.listen(port, () => {
  console.log(`http://localhost:${port}/`);
});
```

可以发现：任何请求进来都会先打印请求日志，然后才会执行具体的业务处理函数

![image-20220209221247946](https://s2.loli.net/2022/03/01/cmV8f4dyuF3RCn5.png)

## 2. 中间件的组成

```js
app.use("/", (req, res, next) => {
  // 其中use可以是get、post等，用于限定请求路径
  next(); // 这个例子就是所有请求路径为根路径的请求都会通过这个中间件
  // 如果当前中间件没有结束请求相应周期，则需要通过next()调用下一个中间件，否则，该请求将会被挂起
});
```

<b style="color:red">如果当前中间件没有结束请求相应周期，则需要通过 next()调用下一个中间件，否则，该请求将会被挂起</b>

## 3. 中间件功能

- 执行任何代码
- 修改 request 或 response 对象
- 结束请求响应周期
- 调用下一个中间件

## 4. 中间件分类

### 4.1 应用程序级别中间件

#### 4.1.1 不做任何限定的中间件

即所有请求都会通过该中间件

```js
app.use(function (req, res, next) {
  res.send("不做任何限定的中间件");
});
```

#### 4.1.2 限定请求路径

即只有请求路径匹配才会通过该中间件

```js
app.use("/user/:id", function (req, res, next) {
  res.send("限定请求路径的中间件");
});
```

#### 4.1.3 限定请求方法 + 请求路径

不能只限定请求方法，因为 app.get()第一个参数必须

```js
app.get("/", (req, res) => {
  res.send("限定请求方法 + 请求路径的中间件");
});
```

#### 4.1.4 多个处理函数

```js
app.use(
  function (req, res, next) {
    console.log("第一次处理");
    next(); // 这个next()之后就是第二个处理函数
  },
  function (req, res, next) {
    console.log("第二次处理");
    next(); // 这个next()之后则是脱离当前处理栈，往后寻找匹配调用
  }
);
```

也可以通过回调函数数组

```js
const first = (req, res, next) => {
  console.log("第一次处理");
  next();
};

const second = (req, res, next) => {
  console.log("第二次处理");
  next();
};

app.use([first, second]);
```

#### 4.1.5 多个路由处理函数

```js
app.use("/", (req, res, next) => {
  console.log("第一个路由处理函数");
  next(); // 如果这里是res.end()，那么就会结束响应。第二个路由处理函数就没有机会执行
});
app.use("/", (req, res, next) => {
  console.log("第二个路由处理函数");
  res.end();
});
```

#### 4.1.6 显示一个中间件子堆栈

```js
app.get(
  "/:id",
  function (req, res, next) {
    if (req.params.id === "0") {
      next("route"); // 跳过当前堆栈之后的所有中间件。即不执行后面的处理函数，而是直接去执行后面的路由处理函数
      // (处理函数和路由处理函数看上面的例子)
    } else {
      next();
    }
  },
  function (req, res, next) {
    res.send("regular");
  }
);

app.get("/:id", function (req, res, next) {
  res.send("special");
});
```

上面例子中，动态参数 id 不为 0 时，第一个处理函数会调用 next()，然后会执行第二个处理函数，然后会收到 regular 的响应。而当 id 为 0 时，会调用 next('route')，会跳过当前堆栈之后的所有中间件。即不执行后面的处理函数，而是直接去执行后面的路由处理函数。所以不会打印 regular，而是打印 special。

### 4.2 路由级别中间件

router.js

```js
const express = require("express");

// 1.创建路由实例
const router = express.Router();

// 2. 配置路由
router.get("/aaa", (req, res) => {
  res.send("get /aaa");
});

router.post("/bbb", (req, res) => {
  res.send("post /bbb");
});

// 3. 导出路由实例
module.exports = router;

// 4. 将路由挂载(集成)到Express实例应用中(见app.js)
```

app.js

```js
const express = require("express");
const app = express();

const router = require("./router.js");

app.get("/", (req, res) => {
  res.send("Hello World!");
});

// 4. 将路由挂载(集成)到Express实例应用中
app.use("/abc", router); // 给路由限定访问前缀/abc

app.listen(3000, () => {
  console.log("http://localhost:3000/");
});
```

![路由级别中间件](https://s2.loli.net/2022/03/01/2lUS43vHrokDjqp.gif)

**链式路由处理程序**：

```js
app
  .route("/abc")
  .get((req, res) => {
    res.send("get");
  })
  .post((req, res) => {
    res.send("post");
  });
```

### 4.3 错误处理中间件

```js
app.use((err, req, res, next) => {
  // 四个参数都要有才是错误处理中间件。如果只有err、req、res，则err实际上是req对象
  console.log("错误: ", err);
  res.status(500).json({
    error: err.message,
  });
});
```

<b style="color: red">四个参数都要有才是错误处理中间件。如果只有 err、req、res，则 err 实际上是 req 对象</b>

app.js

```js
const express = require("express");
const app = express();

const router = require("./router.js");

app.use(router);

app.use((err, req, res, next) => {
  // 四个参数都要有才是错误处理中间件。如果只有err、req、res，则err实际上是req对象
  console.log("错误: ", err);
  res.status(500).json({
    error: err.message,
  });
});

app.listen(3000, () => {
  console.log("http://localhost:3000/");
});
```

router.js

```js
const express = require("express");

// 1.创建路由实例
const router = express.Router();

// 2. 配置路由
router.get("/", (req, res, next) => {
  try {
    const d = b + 1;
  } catch (err) {
    next(err); // 跳过所有剩余的无错误处理路由和中间件函数
  }
});

router.get("/", (req, res, next) => {
  console.log("第二个路由处理函数");
  res.end();
});

// 3. 导出路由实例
module.exports = router;

// 4. 将路由挂载(集成)到Express实例应用中
```

![image-20220210103104627](https://s2.loli.net/2022/03/01/Nanp5BT98fPM4Fr.png)

### 4.4 内置中间件

- **express.json()**：解析 Content-Type 为` application/json`格式的请求体
- **express.urlencoded()**：解析 Content-Type 为` application/x-www-form-urlencoded`格式的请求体
- **express.raw()**：解析 Content-Type 为` application/octet-stream`格式的请求体
- **express.text()**：解析 Content-Type 为` text/plain`格式的请求体
- **express.static()**：托管静态资源文件

### 4.5 第三方中间件

[Express middleware](https://www.expressjs.com.cn/resources/middleware.html)

使用示例：morgan 日志中间件

1. ` npm install morgan`

2. ```js
   const express = require("express");
   const morgan = require("morgan"); // 1. 引入

   const app = express();

   app.use(morgan("tiny")); // 2. 挂载

   app.get("/", (req, res) => {
     res.send("Hello World!");
   });

   app.listen(3000, () => {
     console.log("http://localhost:3000/");
   });
   ```

3. 每次请求都会打印出请求日志

   ![image-20220210105212413](https://s2.loli.net/2022/03/01/Ol7QrKS9HZB8Yin.png)
