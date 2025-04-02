# 工作中的那点事之 JavaScript

## Navigator.sendBeacon（埋点）

`sendBeacon`利用浏览器的后台任务队列，可以实现在页面关闭后还可以继续发送数据。有数据大小限制（通常为几十`KB`），所以适用于发送少量数据，尤其是上报埋点信息这种场景。

> 返回结果为`true/false`。`true`表示请求排队成功，`false`表示失败（超过大小限制就会失败，所以可以通过判断返回值为`false`的时候，改用`XHR`）。数据传一个超长超长的字符串就可以触发。

特点：

1. 页面关闭还可以继续发送数据
2. 有数据大小限制
3. 支持跨域([异步请求只知道 ajax？sendBeacon 这个新的请求策略也很香！](https://juejin.cn/post/7071128380170567710)这篇文章有引用 w3c 文档佐证)
4. 只支持`POST`请求

### demo

html

```html
<button id="send-btn">sendBeacon</button> <button id="xhr-btn">xhr get</button>
```

js

```js
function sendBeacon(url, data) {
  const result = navigator.sendBeacon(
    url,
    `data=${btoa(JSON.stringify(data))}`
  );

  if (result) {
    console.log("beacon send success!");
  }
}

function sendXhr(url, data) {
  const xhr = new XMLHttpRequest();
  xhr.onreadystatechange = function () {
    if (xhr.readyState === 4 && xhr.status === 200) {
      console.log("xhr send success!");
    }
  };

  xhr.open("get", `${url}?data=${btoa(JSON.stringify(data))}`);
  xhr.send();
}

const url = "http://localhost:3000/track/v.png";
const data = {
  name: "clz",
  type: "svip",
};

document.getElementById("send-btn").addEventListener("click", () => {
  sendBeacon(url, data);
});

document.getElementById("xhr-btn").addEventListener("click", () => {
  sendXhr(url, data);
});
```

服务端 js

```js
const express = require("express");

const app = express();

app.use(express.text());

app.post("/track/v.png", (req, res) => {
  console.log(req.body);
  res.sendStatus(200);
});

app.get("/track/v.png", (req, res) => {
  console.log(req.query);
  res.sendStatus(200);
});

app.listen(3000, () => {
  console.log("Server is running on http://localhost:3000");
});
```

![](https://www.clzczh.top/CLZ_img/images/202504022341617.gif)

> 从上图中，可以看到`sendBeacon`的**支持跨域**这个特点
