---
title: 重走Ajax之路2
categories: 前端
date: 2022-06-18 12:42:18
tags:
  - JavaScript
---

# 重走Ajax之路(二) 
前一篇已经简单介绍了下Ajax的用法了(只是简单的GET请求)，下面就来捣鼓下Ajax的其他内容
> 后端可以使用上一篇最后的`express`。

## 同步请求
调用`open`方法时，第三个参数是`false`时，就是同步请求，这时候，JavaScript会堵塞，当服务器响应之后才继续执行。

这时候，绑定的`readystatechange`事件不会有反应，因为同步请求的话，**JavaScript会堵塞，当服务器响应之后才继续执行**，所以当我们绑定事件时，已经接收到响应了，所以状态不会再变也就不会触发`readystatechange`事件。

这时候可以直接把`readystatechange`事件的逻辑外移，不再需要绑定事件。
```js
function fetchData() {
    const xhr = new XMLHttpRequest()

    xhr.open('get', 'http://localhost:8088', false)

    xhr.send(null)

    if (xhr.readyState === 4) {
        if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
            console.log(xhr.responseText)
        }
    }
}
```



也可以把绑定`readystatechange`事件的步骤提前到调用`open`方法之前，这样子，当服务器响应之后，状态会从0变成4，就会触发事件。



## 设置请求头
有时候，我们发送请求时，还需要设置请求头，比如请求头需要携带`token`。这时候就需要在`open()`之后，`send()`之前调用`setRequestHeader`设置请求头。
```js
function fetchData() {
    const xhr = new XMLHttpRequest()

    xhr.open('get', 'http://localhost:8088')

    // 设置请求头
    xhr.setRequestHeader('token', '123456')

    xhr.send(null)

    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
                console.log(xhr.responseText)
            }
        }
    }
}
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bce233fa0ce4bfe9a490e13a5f0c771~tplv-k3u1fbpfcp-zoom-1.image)



## 获取响应头

和设置请求头类似，我们有时候从服务器响应中获取响应头，比如把`token`放到了响应头里。

我们可以通过`getAllResponseHeaders`方法得到能访问的所有响应头，也可以通过`getResponseHeader('myheader')`来获取特定的响应头。

```js
xhr.onreadystatechange = function () {
    if (xhr.readyState === 4) {
        if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
            // 获取全部响应头
            const allHeaders = xhr.getAllResponseHeaders()
            console.log(allHeaders)

            // 获取特定响应头
            const token = xhr.getResponseHeader('token')
            console.error(token)

        }
    }
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d9928ecfa424c07a79594478bfe15fd~tplv-k3u1fbpfcp-zoom-1.image)



## POST请求发送数据(json格式)

GET请求就是通过给GET请求URL后面添加查询字符串参数，如`/getName?user=clz&age=21`

这个只需要使用字符串拼接即可。


POST请求稍微复杂一点点。
先改造一下提供接口的`express`先。
```js
const express = require('express')
const cors = require('cors')

const app = express()

app.use(cors())

// 解析请求体的中间件(json格式)
app.use(express.json())

app.post('/login', function (req, res) {
    console.log(req.body)

    res.status(200).json({
        data: {
            ...req.body,
            tt: 'ttt'
        },
        msg: '登录'
    })
})

app.listen(8088, () => {
    console.error('http://localhost:8088')
})
```



我们可以通过`send`方法，接收一个参数，作为请求体发送出去。
那么能不能直接把一个对象作为请求体发送出去呢？
试一下。

```js
function fetchData() {
    const xhr = new XMLHttpRequest()

    xhr.open('post', 'http://localhost:8088/login')

    xhr.send({
        name: 'clz',
        age: 21
    })

    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
                console.log(xhr.responseText)

            }
        }
    }
}
```

答案是不行的。如果我们直接将对象发过去，会自动调用`toString`方法变成字符串形式。
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181242968.png)



那么，我们在换成JSON字符串再试试。

```js
xhr.send(JSON.stringify({
    name: 'clz',
    age: 21
}))
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f94fefb0d7f480a98b57127429d5e6f~tplv-k3u1fbpfcp-zoom-1.image)

这时候，我们的数据已经正常的发出去了，但是，后端那边并没有介绍到，打印的`req.body`是空的。
这是因为浏览器发送请求是由严格规范的，我们的请求体是JSON字符串格式，还得设置内容类型`Content-Type`为`json`。

```js
xhr.setRequestHeader('Content-Type', 'application/json; charset=utf-8')
xhr.send(JSON.stringify({
    name: 'clz',
    age: 21
}))
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89c0b80f2fd0419cb811fbc5933afa47~tplv-k3u1fbpfcp-zoom-1.image)
后端正常收到请求体

## 超时
我们可以给`XHR`对象增加`timeout`属性，表示发送请求后等待多少毫秒，如果在这段时间内响应不成功就中断请求，并且会触发`timeout`事件。(可以在`express`增加一个定时器响应，时间设置长一点，来模拟请求超时)
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78fbeccc38444ba5b6242318a928e235~tplv-k3u1fbpfcp-zoom-1.image)

```js
// 设置2s超时
xhr.timeout = 2000

xhr.ontimeout = function () {
    alert('响应超时，要中断请求啦')
}
```

![ajax](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181242225.gif)



## 取消未响应的请求

上面我们可以设置超时取消请求，当然也可以手动取消请求。
在没收到响应之前，调用`abort`方法可以取消请求。

```js
const xhr = new XMLHttpRequest()

const cancelBtn = document.getElementById('cancel-btn')
cancelBtn.addEventListener('click', () => {
    xhr.abort()
})
```

![ajax](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181242213.gif)
