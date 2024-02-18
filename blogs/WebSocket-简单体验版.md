---
title: WebSocket-简单体验版
categories: 前端
date: 2022-07-05 20:23:07
tags:
  - JavaScript
---

# WebSocket(简单体验版)

## 简介

Web Socket(套接字)：就是通过**一个长时连接**实现与服务器**全双工、双向**的通信。

Web Socket使用的并不是HTTP协议而是自定义的`Web Socket`协议，所以如果我们使用Web Socket的时候，URL不再是`http://`或`https://`，而是`ws://`或`wss://`(但是，实际上是看红宝书才想着玩一下下，在开发中还没试过用这个来开发的)

主要特点：服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息。



## 使用

### 实例化

要创建一个新的Web Socket，首先需要实例化一个`WebSocket`对象。

**我们实例化WebSocket对象时，传的参数应该是一个绝对`URL`**，**同源策略不适用于WebSocket**

```js
const socket = new WebSocket("ws://localhost:8088/mysocket");
```

![image-20220529170446392](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb19ca431d9b4f47b5199dd5e362daed~tplv-k3u1fbpfcp-zoom-1.image)

**http请求会有跨域，但是WebSocket不会跨域**



### 后端express

主要部分都注释了(需要安装` express-ws`)

```js
const express = require('express')
const expressWs = require('express-ws')

const app = express()

// 将WebSocket服务加到app里，简单来说就是让app添加了ws方法
expressWs(app)

// 建立WebSocket服务
app.ws('/mysocket', function (ws) {
  // ws：建立WebSocket的实例
    
  // 向客户端发送信息
  ws.send('你连接成功啦')
})

app.listen(8088, () => {
  console.log('ws://localhost:8088')
})
```

![image-20220529172415750](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207052023437.png)

如果连接上了，那么http状态码会是101，因为要切换成ws协议



### 绑定事件

如果我们，在实例化后就开始发送信息，那就会导致信息发不出，因为还没连接上。这时候我们就得了解一下什么时候能发，进而得了解一下WebSocket的相关事件。

* `open`：在连接成功建立时触发
* `error`：在连接发生错误时触发(此时已经不能再发信息了)
* `close`：在连接关闭时触发(此时已经不能再发信息了)
* `message`：收到消息后触发(收到的消息在事件对象中的`data`里)

```js
const socket = new WebSocket("ws://localhost:8088/mysocket");
socket.onopen = function () {
    console.log("连接建立");
    socket.send('hello');
    // socket.close()
};
socket.onerror = function () {
    console.log("连接发生错误");
};
socket.onclose = function () {
    console.log("连接关闭");
};

socket.onmessage = function (e) {
    console.log(e)
}
```

![image-20220529173743731](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207052023377.png)

![image-20220529174046932](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207052023443.png)



### 模拟两人对话

上面已经说了，收到消息会触发message事件，所以我们可以在message事件里根据收到的信息发送对应的信息。

客户端：

```js
const socket = new WebSocket("ws://localhost:8088/mysocket");
socket.onopen = function () {
    console.log("连接建立");
    socket.send('hello');
    // socket.close()
};
socket.onerror = function () {
    console.log("连接发生错误");
};
socket.onclose = function () {
    console.log("连接关闭");
};

socket.onmessage = function ({ data }) {
    if (data.includes('你好')) {
        socket.send('再见，服务端ddd')
    } else if (data.includes('再见')) {
        // 调用close()方法关闭WebSocket连接
        socket.close()
    } else {
        socket.send('你好，我是客户端ccc')
    }
}
```



服务器：

```js
const express = require('express')
const expressWs = require('express-ws')

const app = express()

// 将WebSocket服务加到app里，简单来说就是让app添加了ws方法
expressWs(app)

// 建立WebSocket服务
app.ws('/mysocket', function (ws) {
  // ws：建立WebSocket的实例
  ws.send('你连接成功啦')

  ws.onmessage = function ({ data }) {
    if (data.includes('你好')) {
      ws.send('你好，我是服务器ddd')
    } else if (data.includes('再见')) {
      ws.send('再见，客户端ccc')
    }
  }
})

app.listen(8088, () => {
  console.log('ws://localhost:8088')
})
```

![image-20220529175742257](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d54d3a931f6e4e8399e43804e83848cd~tplv-k3u1fbpfcp-zoom-1.image)



注意：如果收发部分处理，需要注意一下，如果没有处理好，可能会出现循环卡住的情况。

比如，服务器和客户端的message事件回调都是：

```js
socket.onmessage = function ({ data }) {
    socket.send('hello')
}
```

![image-20220529180303794](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e52e8bdcc5754feeac2bfca740df9a2c~tplv-k3u1fbpfcp-zoom-1.image)
