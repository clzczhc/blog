---
title: Express(一) ——简单入门
date: 2022-02-16 23:07:17
keywords: Express
categories: "后端"
tags:
  - Express
---

# Express(一) ——简单入门

背景：参加的青训营项目，使用 Express 来实现后端，个人被分配到后端去。于是，简单速通了下 Express。项目结束，回头写下笔记，沉淀一下。

Express 是基于 [Node.js](https://nodejs.org/en/) 平台，快速、开放、极简的 Web 开发框架。

开始前可以先安装[Postman](https://www.postman.com/)，很好用的**接口测试工具**。

## 1. Hello World

首先，安装 express 到项目中` npm i express`

然后，开始代码世界。

```js
// 1. 加载express
const express = require("express");

// 2. 调用express()获取。express()函数是express模块​​导出的顶级函数
const app = express();

// 3. 设置请求对应的处理函数。下面的例子中，当客户端以GET方法请求/时就会调用处理函数
app.get("/", (req, res) => {
  res.send("Hello World!");
});

// 4. 启动web服务
app.listen(8080, () => {
  console.log("http://localhost:8080/");
});
```

最后，命令行执行` nodemon app.js`或` node app.js`。`nodemon`支持热更新。

![image-20220207151828081](https://s2.loli.net/2022/02/16/boPURrd42Yap6Cv.png)

## 2. 路由

路由是指服务器端应用程序如何响应特定端点的客户端请求。由一个 URI(路径标识)和一个特定的 HTTP 方法(GET、POST 等)组成的。

路由的定义结构：

```javascript
app.METHOD(PATH, HANDLER);
```

- app：express 实例
- METHOD：是一个 HTTP 请求方法
- PATH：服务端路径
- HANDLER：当路由匹配到时执行的处理函数。参数：` request`和` response`对象分别处理请求和响应数据

```js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  // GET请求
  res.send("Hello World!");
});

app.post("/", (req, res) => {
  // POST请求
  res.send("post /");
});

app.put("/user", (req, res) => {
  // PUT请求
  res.send("put user");
});

app.delete("/user", (req, res) => {
  // DELETE请求
  res.send("delete user");
});

app.listen(8080, () => {
  console.log("http://localhost:8080/");
});
```

![image-20220207161937956](https://s2.loli.net/2022/02/16/nRrPuxaXsd4gqtb.png)

### 2.1 请求对象

req 对象代表 HTTP 请求。

```js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  console.log("请求地址: ", req.url);
  console.log("请求方法: ", req.method);
  console.log("请求头: ", req.headers);
  console.log("请求参数: ", req.query);

  res.end(); // 结束响应。没有的话，客户端会一直等待回应
});

app.listen(8080, () => {
  console.log("http://localhost:8080/");
});
```

postman 测试用：http://localhost:8080/?name=clz

![image-20220207162928596](https://s2.loli.net/2022/02/16/ESJXsLO63ZnQ4gC.png)

### 2.2 响应对象

res 对象表示收到 HTTP 请求后发送的 HTTP 响应。

#### 2.2.1 状态码及状态信息

```js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.statusCode = 404; // 设置响应状态码
  res.statusMessage = "test"; // 设置响应状态信息。这里是测试，理论上来说404应该对应Not Found，这样子才有意义

  res.end(); // 结束响应
});

app.listen(8080, () => {
  console.log("http://localhost:8080/");
});
```

![image-20220207163814050](https://s2.loli.net/2022/02/16/QzX3ZEjpxmIRPDS.png)

#### 2.2.2 发送多段文本

```js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.write("hello ");
  res.write("world"); // res.write()只是发送数据，还是需要res.end()来结束响应

  // res.end('hello world');   // 结束相应的同时发送数据

  res.end(); // 结束响应
});

app.listen(8080, () => {
  console.log("http://localhost:8080/");
});
```

![image-20220207165808608](https://s2.loli.net/2022/02/16/RSh3NW4JLPxUk5F.png)

#### 2.2.3 cookie

```js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.cookie("name", "clz"); // 设置cookie

  res.end(); // 结束响应
});

app.listen(8080, () => {
  console.log("http://localhost:8080/");
});
```

![image-20220207164941307](https://s2.loli.net/2022/02/16/67rI4xtqigCc51a.png)

### 2.3 路由路径

可以使用正则表达式语法

```js
// 匹配根路径
app.get("/", function (req, res) {
  res.send("root");
});

// 匹配/abc
app.get("/abc", function (req, res) {
  res.send("abc");
});

// 匹配/test.text
app.get("/test.text", function (req, res) {
  res.send("test.text");
});

// 匹配/acd、/abcd
app.get("/ab?cd", function (req, res) {
  res.send("ab?cd");
});

// 匹配/abcd、/abxxxxcd
app.get("/ab*cd", function (req, res) {
  res.send("ab?cd");
});

// 匹配/abe、/abcde
app.get("/ab(cd)?e", function (req, res) {
  res.send("ab?cd");
});

// 匹配所有包含a
app.get(/a/, function (req, res) {
  res.send("/a/");
});

// 匹配以fly结尾的，包括/test.fly，/test/aaa/fly等
app.get(/.*fly$/, function (req, res) {
  res.send("/.*fly$/");
});
```

### 2.4 动态路径

```js
app.get("/users/:userId/books/:bookId", function (req, res) {
  res.send(req.params);
});

// 限制动态参数
app.get("/:a(\\d+)", function (req, res) {
  res.send(req.params);
});
```

![image-20220210123301562](https://s2.loli.net/2022/02/16/3qNmKThznV87fob.png)

## 3. 案例

创建一个简单的 CRUD 接口服务。增加(Create)、读取查询(Retrieve)、更新(Update)和删除(Delete)

- 查询任务列表：` GET /todos`
- 根据 ID 查询单个任务：`GET /todos/:id`
- 添加任务：` POST /todos`
- 修改任务：` PATCH /todos`
- 删除任务：` DELETE /todos/:id`

### 3.1 路由设计

```js
const express = require("express");

const app = express();

app.get("/todos", (req, res) => {
  res.send("查询任务列表");
});

app.get("/todos/:id", (req, res) => {
  res.send(`根据ID查询单个任务, id是${req.params.id}`); // 通过req.params.id来获取动态的路径参数id
});

app.post("/todos", (req, res) => {
  res.send("添加任务");
});

app.patch("/todos/:id", (req, res) => {
  res.send(`修改任务, id是${req.params.id}`);
});

app.delete("/todos/:id", (req, res) => {
  res.send(`删除任务, id是${req.params.id}`);
});

app.listen(8080, () => {
  console.log("http://localhost:8080/");
});
```

![image-20220207172638141](https://s2.loli.net/2022/02/16/ZeclNIh5qajyfBV.png)

### 3.2 获取任务列表

数据文件 db.json

```json
{
  "todos": [
    {
      "id": 1,
      "title": "express"
    },
    {
      "id": 2,
      "title": "笔记"
    },
    {
      "id": 3,
      "title": "更新博客"
    }
  ]
}
```

```js
app.get("/todos", (req, res) => {
  fs.readFile("./db.json", "utf8", (err, data) => {
    if (err) {
      return res.status(500).json({
        // res.json()专门发送json格式的数据，不是json格式会报错
        error: err.message,
      });
    }

    const db = JSON.parse(data); // 把字符串转成JSON对象
    res.status(200).json(db.todos);
  });
});
```

![image-20220207184102851](https://s2.loli.net/2022/02/16/eZxBPAKo6lLyUg9.png)

### 3.3 根据 ID 查询单个任务

```js
app.get("/todos/:id", (req, res) => {
  fs.readFile("./db.json", "utf8", (err, data) => {
    if (err) {
      return res.status(500).json({
        error: err.message,
      });
    }

    const db = JSON.parse(data);
    const todo = db.todos.find(
      (todo) => todo.id === Number.parseInt(req.params.id)
    ); // url中的动态参数是字符串

    if (!todo) {
      // 任务id不存在
      return res.status(404).end(); // 需要return阻止代码继续往下执行，否则会出现既发送404又发送200
    }

    res.status(200).json(todo);
  });
});
```

![image-20220208231000242](https://s2.loli.net/2022/02/16/OXaKcMJbZq8D5SI.png)

### 3.4 封装 db 模块

从上面的代码中可以发现，读取数据文件部分逻辑一样，即可以封装成单独的模块 db.js

db.js

```js
const fs = require("fs");
const { promisify } = require("util"); // 把callback形式的异步api转化成promise形式的
const path = require("path");

const readFile = promisify(fs.readFile);

const dbPath = path.join(__dirname, "./db.json");

exports.getDb = async () => {
  const data = await readFile(dbPath, "utf8");

  return JSON.parse(data);
};
```

封装后的 app.js(后面的路由没有变化)

```js
const express = require("express");

const { getDb } = require("./db.js");

const app = express();

app.get("/todos", async (req, res) => {
  try {
    // 处理异常的必要性：没有抛出异常的话，可能会一直在等待响应
    const db = await getDb(); // 因为getDb是async的，所以所有形式都会被封装成Promise，所以获取数据都要await
    res.status(200).json(db.todos); // // res.json()专门发送json格式的数据，不是json格式会报错
  } catch (err) {
    res.status(500).json({
      error: err.message,
    });
  }
});

app.get("/todos/:id", async (req, res) => {
  try {
    const db = await getDb();
    const todo = db.todos.find(
      (todo) => todo.id === Number.parseInt(req.params.id)
    ); // url中的动态参数是字符串

    if (!todo) {
      // 任务id不存在
      return res.status(404).end(); // 需要return阻止代码继续往下执行，否则会出现既发送404又发送200
    }

    res.status(200).json(todo);
  } catch (err) {
    res.status(500).json({
      error: err.message,
    });
  }
});
```

### 3.5 添加任务

```js
app.post("/todos", (req, res) => {
  // 1. 获取客户端请求体参数
  console.log(req.body);

  res.end();
});
```

然后，会发现很恐怖的事情

![image-20220208231856994](https://s2.loli.net/2022/02/16/9kbuOrwlJ4NnVEc.png)

那么，这个时候就需要配置表单请求体来解决上述问题

```js
app.use(express.json()); // 配置解析表单请求体：application/json。将json格式转成js对象
```

![image-20220208232402601](https://s2.loli.net/2022/02/16/2TIgyXGsfwQka7i.png)

完美!!!(然而，并不是)

![image-20220208233059259](https://s2.loli.net/2022/02/16/8GC1QZHcbnvDRie.png)

换种形式，就要换汤了。因为 express.json()只能解析 json 形式的

```js
app.use(express.urlencoded()); // 配置解析表单请求体：application/x-www-form-urlencoded
```

然后，因为需要保存到 db.json 中，所以也应该在 db.js 中封装一个 saveDb()方法（**app.js 自然也要引入 saveDb**，这部分就不行出来了）

db.js

```js
const fs = require("fs");
const { promisify } = require("util"); // 把callback形式的异步api转化成promise形式的
const path = require("path");

const readFile = promisify(fs.readFile);
const writeFile = promisify(fs.writeFile);

const dbPath = path.join(__dirname, "./db.json");

exports.getDb = async () => {
  const data = await readFile(dbPath, "utf8");

  return JSON.parse(data);
};

exports.saveDb = async (db) => {
  const data = JSON.stringify(db);
  await writeFile(dbPath, data);
};
```

添加任务的代码部分

```js
app.post("/todos", async (req, res) => {
  try {
    // 1. 获取客户端请求体参数
    const todo = req.body;

    // 2. 数据验证
    if (!todo.title) {
      return res.status(422).json({
        error: "The field title is required.",
      });
    }

    // 3. 数据验证通过后，把数据存储到db.json中
    const db = await getDb();
    const lastTodo = db.todos[db.todos.length - 1]; // 获取最后一个todo

    todo.id = lastTodo ? lastTodo.id + 1 : 1; // 如果一个todo都没有，则添加的todo的id是1
    db.todos.push(todo);

    await saveDb(db);

    // 4. 发送响应
    res.status(200).json(todo);
  } catch (err) {
    res.status(500).json({
      error: err.message,
    });
  }
});
```

![image-20220208235418267](https://s2.loli.net/2022/02/16/aLlj7UR4mMhOczG.png)

**小优化**：上面可以发现，添加任务后，db.json 格式很丑。其实就是把 JavaScript 对象转换为 JSON 字符串时的问题，所以只需要在`JSON.stringify()`上下点功夫就行。

[JSON.stringify()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

```js
exports.saveDb = async (db) => {
  const data = JSON.stringify(db, null, "  "); // 这里第三个参数是两个空格，就是缩进按两个空格来
  await writeFile(dbPath, data);
};
```

![image-20220209000445369](https://s2.loli.net/2022/02/16/f6CvMnG3bHw2F7t.png)

眼尖的同学可能发现，添加的任务 id 和之前的位置不太一样。那么，有点小强迫症的我自然还是要在微操一手。

![image-20220209001337659](https://s2.loli.net/2022/02/16/Un1avz8kDLJFisb.png)

终于。。。

![image-20220209001401680](https://s2.loli.net/2022/02/16/JoHXaw8uz4Fxd61.png)

### 3.6 修改任务

```js
app.patch("/todos/:id", async (req, res) => {
  try {
    // 1. 获取客户端请求体参数
    const todo = req.body;

    // 2. 查找到要修改的todo
    const db = await getDb();
    const ret = db.todos.find(
      (todo) => todo.id === Number.parseInt(req.params.id)
    );

    if (!ret) {
      // 要修改的todo不存在
      return res.status(404).end();
    }

    Object.assign(ret, todo); // 返回一个对象。如果添加的todo中有原本就有的属性，则修改属性值。如果没有，则新增属性

    await saveDb(db);
    res.status(200).json(ret);
  } catch (err) {
    res.status(500).json({
      error: err.message,
    });
  }
});
```

修改原有属性：

![image-20220209003447970](https://s2.loli.net/2022/02/16/nJ8f3jVFRbQSOMl.png)

新增属性

![image-20220209003636418](https://s2.loli.net/2022/02/16/ldx8WTQUqnHFKiz.png)

### 3.7 删除任务

```js
app.delete("/todos/:id", async (req, res) => {
  try {
    const todoId = Number.parseInt(req.params.id);
    const db = await getDb();

    const index = db.todos.findIndex((todo) => todo.id === todoId); // 造出要删除的todo的索引

    if (index === -1) {
      // 要删除的todo压根不存在
      return res.status(404).end();
    }

    db.todos.splice(index, 1);
    await saveDb(db);
    res.status(200).end();
  } catch (err) {
    res.status(500).json({
      error: err.message,
    });
  }
});
```

![image-20220209004455315](https://s2.loli.net/2022/02/16/Ldr7TchUJQ45DHm.png)

## 4. res.end()和 res.send()区别

官方说明：

- res.end() 终结响应处理流程。(不过，也可以在结束的同时发送响应)

- res.send() 发送各种类型的响应。

### 4.1 res.end()

结束响应流程。用于在没有任何数据的情况下快速结束响应。

- <b style="color: red">参数可以是 buffer 对象、字符串</b>
- <b style="color: red">只接受服务器响应数据，如果是中文会乱码</b>

### 4.2 res.send()

发送 HTTP 响应。

- <b style="color: red">参数可以是 buffer 对象、字符串、对象、数组</b>
- <b style="color: red">发送给服务端时，会自动发送更多的响应报文头，包括 Content-Type: text/html;charset=utf-8，所以中文不会乱码</b>

**res.send()发送对象响应**

```js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.send({
    name: "clz",
  });
});

app.listen(3000, () => {
  console.log("http://localhost:3000/");
});
```

![image-20220209213846401](https://s2.loli.net/2022/02/16/1IPGzkad7fSLpRb.png)

**改为用 res.end()发送**

![image-20220209214613515](https://s2.loli.net/2022/02/09/GHvOkV5D68yld1i.png)

**res.send()发送中文**(使用浏览器查看，postman 可能自动设置了响应头)

```js
res.send("测试");
```

![image-20220209215019228](https://s2.loli.net/2022/02/09/oeydE6RNjT58pX2.png)

**改为 res.edn()**：

![image-20220209215053732](https://s2.loli.net/2022/02/09/tqXpfQwc3yFCPj2.png)

学习参考视频：

[Node.js 系列教程之 Express](https://www.bilibili.com/video/BV1mQ4y1C7Cn)
