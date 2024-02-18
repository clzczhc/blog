---
title: axios笔记(一)    简单入门
date: 2022-03-10 09:37:44
categories: 前端
tags:
  - axios
---

# axios 笔记(一) 简单入门

又是前端必备知识的笔记。

## HTTP

之前的笔记：[HTTP 笔记 | 赤蓝紫 (clz.vercel.app)](https://clz.vercel.app/2022/02/09/yc-http/)

### 1. 介绍

> HTTP 是一种能够获取如 HTML 这样的网络资源的[protocol](https://developer.mozilla.org/zh-CN/docs/Glossary/Protocol)(通讯协议)。它是在 Web 上进行数据交换的基础，是一种 client-server 协议，也就是说，请求通常是由像浏览器这样的接受方发起的。一个完整的 Web 文档通常是由不同的子文档拼接而成的，像是文本、布局描述、图片、视频、脚本等等。

文档：[HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview)

### 2. HTTP 请求交互的基本过程

![image-20220228094936947](https://s2.loli.net/2022/03/10/xBUuGF69vNS78mI.png)

1. 浏览器向服务器发送请求报文
2. 后台服务器接收到请求后，调度服务器应用处理请求，向浏览器返回 HTTP 响应(响应报文)
3. 浏览器接收到响应，解析显示响应体 / 调用监视回调

查看 HTTP 请求响应信息：DevTools Network 面板

![image-20220228095526783](https://s2.loli.net/2022/03/10/AZN5J6KEwiovrYu.png)

### 3. API 分类

#### 3.1 REST API(restful)

[RESTful 接口设计规范](https://clz.vercel.app/2022/02/28/RESTful/)

- 发送请求进行 CRUD 哪个操作由请求方式来决定
- 同一个请求路径可以进行多个操作
- 请求方式会用到 GET / POST / PUT / DELETE 等

#### 3.2 非 REST API(restless)

- 请求方式不决定请求的 CRUD 操作(甚至可以用 GET 请求进行删除操作)
- 一个请求路径只对应一个操作
- 请求方式一般只有 GET / POST

### 4. json-server 搭建 REST 接口

[json-server 仓库](https://github.com/typicode/json-server)

1. 全局安装

   ```shell
   npm install -g json-server
   ```

2. 新建` db.json`文件

   ```json
   {
     "posts": [{ "id": 1, "title": "json-server", "author": "typicode" }],
     "comments": [{ "id": 1, "body": "some comment", "postId": 1 }],
     "profile": { "name": "typicode" }
   }
   ```

3. 开启服务器(支持热更新)

   ```shell
   json-server --watch db.json
   ```

4. 打开` http://localhost:3000/`，可以在 Resources 中看到所有的接口

   ![image-20220228105153779](https://s2.loli.net/2022/03/10/aMDp4T9JQ2ruvqV.png)

5. 点击对应接口，可以获取对应数据

   ![image-20220228105310624](https://s2.loli.net/2022/03/10/pg9WGrXK1HhINBm.png)

6. 支持携带参数

   - params 参数

     ![image-20220228105412534](https://s2.loli.net/2022/03/10/Ux4IzmPwQ3BFLuA.png)

   - query 参数

     ![image-20220228105530166](https://s2.loli.net/2022/03/10/j65GDoirbMQa27A.png)

   - 两种参数区别：query 参数是从所有的数据中筛选，所以最后是数组的形式；params 参数则是特定查找的形式，所以最后是对象的形式

## 使用 axios 请求 REST 接口

<b style="color:red">上面开启的服务器不要关</b>

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
    <button onclick="testGet()">GET请求</button>
    <button onclick="testPost()">POST请求</button>
    <button onclick="testPut()">PUT请求</button>
    <button onclick="testDelete()">DELETE请求</button>

    <script src="./node_modules/axios/dist/axios.min.js"></script>
    <script>
      const testGet = () => {
        axios
          .get("http://localhost:3000/posts", {
            // params: {
            //   id: 2
            // }
          })
          .then((response) => {
            console.log("/posts get", response.data);
          });
      };

      const testPost = () => {
        axios
          .post("http://localhost:3000/posts", {
            title: "czh",
            author: "czh",
          })
          .then((response) => {
            console.log("/posts post", response.data);
          });
      };

      const testPut = () => {
        axios
          .put("http://localhost:3000/posts/3", {
            title: "czh...",
            author: "czh...",
          })
          .then((response) => {
            console.log("/posts put", response.data);
          });
      };

      const testDelete = () => {
        axios.delete("http://localhost:3000/posts/3").then((response) => {
          console.log("/posts delete", response.data);
        });
      };
    </script>
  </body>
</html>
```

## XHR

### 1. 介绍

> `XMLHttpRequest`（XHR）对象用于与服务器交互。通过 XMLHttpRequest 可以在不刷新页面的情况下请求特定 URL，获取数据。这允许网页在不影响用户操作的情况下，更新页面的局部内容。`XMLHttpRequest` 在 [AJAX](https://developer.mozilla.org/zh-CN/docs/Glossary/AJAX) 编程中被大量使用。

文档：[XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)

### 2. ajax 请求与一般的 http 请求

- ajax 请求是一种特殊的 http 请求

- 对服务器端来说，没有任何请求，区别在于**浏览器端**(ajax 请求有专门的 ajax 引擎帮忙发送)

- 浏览器端发送请求，只有 XHR 或 fetch 发出的才是 ajax 请求，其他的都不是 ajax 请求

- 浏览器端接收到响应(一般请求浏览器会自动更新页面，而 ajax 请求需要手动更新)

  - 一般请求：浏览器会直接显示响应体数据，即刷新/跳转页面

  - ajax 请求：浏览器不会对页面进行任何更新操作，而只是调用监视的回调函数并传入响应相关数据

### 3. 常用 API

- XMLHttpRequest()：创建 XHR 对象的构造函数

- status：响应状态码，如 200、404 等

- statusText：响应状态文本

- readyState：标识请求状态的只读属性

   0: 初始

   1: open()之后

   2: send()之后

   3: 请求中

   4: 请求完成

- onreadystatechange：绑定 readyState 改变的监听

- responseType：指定响应数据类型

- timeout：指定请求超时时间，默认为 0，表示没有限制

- open()：初始化一个请求。参数为` (method, url [, async])`

- send(data)：发送请求

- setRequestHeader(name, value)：设置请求头

- getResponseHeader(name)：获取指定名称的响应头值

## 封装 axios

axios

```js
function axios({ url, method = "GET", params = {}, data = {} }) {
  // 返回Promise对象
  return new Promise((resolve, reject) => {
    // 处理method大小写
    method = method.toUpperCase();

    // 把请求参数拼接到url中
    let queryString = "";
    Object.keys(params).forEach((key) => {
      queryString += `${key}=${params[key]}&`;
    });

    if (queryString) {
      // 有查询参数,需要把最后的&去掉
      queryString = queryString.substring(0, queryString.length - 1);
      url += `?${queryString}`;
    }

    // 1. 执行异步ajax请求
    // 1.1 创建xhr对象
    const xhr = new XMLHttpRequest();

    // 1.2 打开连接，初始化请求
    xhr.open(method, url, true); // 第三个参数表示是否异步执行操作，默认为true。如果值为false，send()方法直到收到答复前不会返回。

    // 1.3 发送请求
    if (method === "GET" || method === "DELETE") {
      xhr.send();
    } else if (method === "POST" || method === "PUT") {
      xhr.setRequestHeader("Content-Type", "application/json;charset=utf-8"); // 设置请求头，通知服务器请求体的格式是json
      xhr.send(JSON.stringify(data)); // 发送json格式请求体参数
    }

    // 1.4 绑定状态的监听,监听的定义能放在后面是因为这里是异步发送请求
    xhr.onreadystatechange = function () {
      if (xhr.readyState !== 4) {
        return;
      }

      const {
        status, // 响应状态在[200, 300)之间代表成功,否则失败
        statusText,
      } = xhr;

      // 2.1 如果请求成功，调用resolve()
      if (status >= 200 && status < 300) {
        const response = {
          data: JSON.parse(xhr.response), // 把响应转化成JSON对象
          status,
          statusText,
        };
        resolve(response);
      } else {
        // 2.2 如果请求失败，调用reject()
        reject(new Error("request error status is " + status));
      }
    };
  });
}
```

使用：

```js
// GET请求: 服务端获取数据
const testGet = () => {
  axios({
    url: "http://localhost:3000/posts",
    method: "GET",
    params: {
      id: 1,
    },
  }).then(
    (response) => {
      console.log(response);
    },
    (error) => {
      alert(error.message);
    }
  );
};

// POST请求: 服务端增加数据
const testPost = () => {
  axios({
    url: "http://localhost:3000/posts",
    method: "POST",
    data: {
      title: "axios",
      author: "clz",
    },
  }).then(
    (response) => {
      console.log(response);
    },
    (error) => {
      alert(error.message);
    }
  );
};

// PUT请求: 服务端更新数据
const testPut = () => {
  axios({
    url: "http://localhost:3000/posts/1",
    method: "put",
    data: {
      title: "axios!!!!!",
      author: "clz!!!!!!",
    },
  }).then(
    (response) => {
      console.log(response);
    },
    (error) => {
      alert(error.message);
    }
  );
};

// DELETE请求: 服务端删除数据
const testDelete = () => {
  axios({
    url: "http://localhost:3000/posts/2",
    method: "delete",
  }).then(
    (response) => {
      console.log(response);
    },
    (error) => {
      alert(error.message);
    }
  );
};
```

<b style="color: red">发送 POST、PUT 等需要修改服务器端的资源的请求时会发送 OPTIONS 请求，查看是否能够修改，即预请求。而 GET 请求不需要，因为 GET 请求不需要修改服务器上的资源</b>

![image-20220228151239576](https://s2.loli.net/2022/03/10/XfniK4GOZEIRcVs.png)

学习链接：[尚硅谷\_axios 核心技术](https://www.bilibili.com/video/BV1NJ41197u6)
