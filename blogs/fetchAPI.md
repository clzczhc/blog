---
title: 简单入门Fetch API
categories: 前端
date: 2022-06-18 12:55:37
tags:
  - JavaScript
---

# 简单入门Fetch API

## 前言

`Fetch API`是使用 JavaScript请求资源的优秀工具。虽然我们开发时可能是经常使用`axios`，但是实际上`Fetch API`也能做很多一样的事。并且使用`Fetch API`不需要安装`axios`，所以我们做一些小案例，但是需要调接口的话，`Fetch API`便是很好的选择，不需要安装`axios`，也不需要像XMLHttpRequest 对象那样子需要较多步骤。

## 基本用法

接口有需要可以到最后自取(express接口)

### 分派请求

只需要使用`fetch()`方法即可，传参为获取资源的URL。该方法返回一个`Promise`对象。(和`axios`使用非常像)

```js
const r = fetch('http://localhost:8088/getInfo')

console.log(r)

r.then(res => {
    console.log(res)
})
```

![image-20220604132403546](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72e9dc5fe06f4f1481df43a07e35310b~tplv-k3u1fbpfcp-zoom-1.image)



### 读取响应

上面我们已经把响应结果打印出来了，但是并没有得到真正的响应体的数据。

这时候可以使用`text()`方法，这个方法会返回一个`Promise`对象，这个对象会`resolve`为读取资源的完整内容。

```js
fetch('http://localhost:8088/getInfo?name=clz')
    .then(async (res) => {
        const data = await res.text()
        console.log(data)
        console.log(typeof data)
    })
```

![image-20220611132502823](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03a8451581e04a29ac892d24da3ec015~tplv-k3u1fbpfcp-zoom-1.image)



从结果来看，发现这时候得到的数据是`string`类型的，之后还需要通过`JSON.parse()`来操作。很显然不太好，这个时候只需要不是使用`text()`方法，而是使用`json()`方法即可。(使用方式和`text()`方法一样)

![image-20220611132921936](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/597b7263d4f94c82960aa8fd5714d8c2~tplv-k3u1fbpfcp-zoom-1.image)



### 请求失败

请求失败的时候还是会正常执行`then`方法里的处理函数。(这里的失败是指服务器返回了响应，但是不是成功的请求。)

```js
fetch('http://localhost:8088/getBadRequest')
    .then(async (res) => {
        const data = await res.json()
        console.log(data)
    })
```

![image-20220611141429705](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206111414772.png)



如果服务器没有响应导致浏览器超时的话，这时候就不会再执行`then()`方法的处理函数，而是执行`catch()`方法的，因为这时候的`Promise`不再是`resolved`状态，而是`rejected`状态。(比如跨域时候)

后端接口注释掉`app.use(cors())`，不再处理跨域

```js
fetch('http://localhost:8088/getBadRequest')
    .then(async (res) => {
        const data = await res.json()
        console.log(data)
    })
    .catch(reason => {
        console.log('catch()方法里的处理函数')
        console.log(reason)
    })
```

![image-20220611142139336](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ae2540a3b88488c95afaa7a9ecd4ef9~tplv-k3u1fbpfcp-zoom-1.image)



### POST方法

上面我们直接使用`fetch()`方法就是`GET`请求，那么假如我们想要使用`POST`方法来进行新增数据之类的操作呢？

`fetch`方法的第二个参数就是自定义选项，通过自定义选项就能实现`GET`请求之外的请求。比如使用`POST`方法的时候，自定义选项就需要`method`来确定请求方法，以及`body`来确定请求体的数据。

```js
const data = {
    name: 'clz',
    age: 21
}

fetch('http://localhost:8088/postInfo', {
    method: 'POST',
    body: JSON.stringify(data)
})
    .then(async (res) => {
        const data = await res.json()
        console.log(data)
    })
```

![image-20220611151000558](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e46b910573ce453aa428e8c068ce4fb8~tplv-k3u1fbpfcp-zoom-1.image)



结果发现：请求得到的响应的状态码是400，提示信息是需要姓名和年龄，但是我们明明已经把姓名和年龄传过去了。这种时候，有可能是后端处理的问题，也有可能是前端传出去的格式的问题(即请求头的`Content-Type`)

![image-20220611151154399](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206111511462.png)



果不其然，我们传的数据是json形式的，但是`Content-Type`却不是json，所以我们的自定义选项还需要添加一个`headers`选项来设置选项的请求头。



```js
const data = {
    name: 'clz',
    age: 21
}

const headers = new Headers({
    'Content-Type': 'application/json; charset=utf-8'
})

fetch('http://localhost:8088/postInfo', {
    method: 'POST',
    body: JSON.stringify(data),
    headers
})
    .then(async (res) => {
        const data = await res.json()
        console.log(data)
    })
```



![image-20220611151402801](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/390ad0d4a8fa48dabf39a8eeee45589f~tplv-k3u1fbpfcp-zoom-1.image)





## express接口

```js
const express = require('express')
const cors = require('cors')

const app = express()

// 解决跨域
app.use(cors())

// 解析请求体的中间件(json格式)
app.use(express.json())

// GET请求
app.get('/getInfo', (req, res) => {

  res.json({
    code: 200,
    data: {
      name: '赤蓝紫',
      age: 21
    },
    msg: '获取信息成功',
  })
})

// 响应状态码为400
app.get('/getBadRequest', (req, res) => {
  res.status(400).json({
    code: 400,
    msg: 'Bad Request',
  })
})

// POST请求
app.post('/postInfo', (req, res) => {
  if (req.body.name === undefined || req.body.age === undefined) {
    res.status(400).json({
      code: 400,
      msg: '必须要有姓名、年龄'
    })
  } else {
    res.json({
      code: 200,
      data: req.body
    })
  }

})

const port = 8088
app.listen(port, () => {
  console.log(`http://localhost:${port}`)
})
```
