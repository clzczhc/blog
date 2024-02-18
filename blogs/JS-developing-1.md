---
title: JavaScript生成验证码和32位随机码
date: 2021-12-11 14:08:37
keywords: 前端 JS JavaScript
categories: "前端"
tags:
  - JavaScript
---

# JavaScript 生成验证码和 32 位随机码

## 1.使用 canvas 实现生成验证码功能

本文的 html 文件如下图所示，实现验证码的 js 文件为 verify.js

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>验证码</title>
  </head>

  <body>
    <canvas id="vetifyCanvas" width="100" height="40"></canvas>
    <script src="./verify.js"></script>
  </body>
</html>
```

### 1.1 设置背景为随机颜色

```js
const canvas = document.getElementById("vetifyCanvas");
const w = canvas.width;
const h = canvas.height;
const context = canvas.getContext("2d");

context.fillStyle = randomColor(0, 155); // 设置背景为一个随机颜色
context.fillRect(0, 0, w, h); // 绘制矩形

function randomNum(min, max) {
  return Math.floor(Math.random() * (max - min) + min); // 生成一个随机数
}

function randomColor(min, max) {
  const r = randomNum(min, max);
  const g = randomNum(min, max);
  const b = randomNum(min, max);
  return "rgb(" + r + ", " + g + ", " + b + ")";
}
```

### 1.2 生成 4 位随机验证码

```js
const canvas = document.getElementById("vetifyCanvas");
const w = canvas.width;
const h = canvas.height;
const context = canvas.getContext("2d");

let text = []; //用来存放用于随机的字符
let verifyCode = []; // 用来存放验证码

context.fillStyle = randomColor(0, 155); // 设置背景为一个随机颜色
context.fillRect(0, 0, w, h); // 绘制矩形

addText("a", "z");
addText("A", "Z");
addText("0", "9");

generateCode(4);
console.log(verifyCode);

function randomNum(min, max) {
  return Math.floor(Math.random() * (max - min) + min); // 生成一个随机数
}

function randomColor(min, max) {
  const r = randomNum(min, max);
  const g = randomNum(min, max);
  const b = randomNum(min, max);
  return "rgb(" + r + ", " + g + ", " + b + ")";
}

function addText(start, end) {
  // 添加用于随机的字符
  for (let i = start.charCodeAt(); i <= end.charCodeAt(); i += 1) {
    // 根据ASCII字符得到ASCII值
    text.push(String.fromCharCode(i)); // 根据ASCII值获得ASCII字符
  }
}

function generateCode(num) {
  // 生成验证码
  for (let i = 0; i < num; i++) {
    let index = randomNum(0, text.length - 1);
    verifyCode.push(text[index]);
  }
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112111344310.jpeg)

### 1.3 把生成的验证码画出来

```js
const canvas = document.getElementById("vetifyCanvas");
const w = canvas.width;
const h = canvas.height;
const context = canvas.getContext("2d");

let text = []; //用来存放用于随机的字符
let verifyCode = []; // 用来存放验证码

context.fillStyle = randomColor(0, 255); // 设置背景为一个随机颜色
context.fillRect(0, 0, w, h); // 绘制矩形
context.textBaseline = "middle";

addText("a", "z");
addText("A", "Z");
addText("0", "9");

generateCode(4);

function randomNum(min, max) {
  return Math.floor(Math.random() * (max - min) + min); // 生成一个随机数
}

function randomColor(min, max) {
  const r = randomNum(min, max);
  const g = randomNum(min, max);
  const b = randomNum(min, max);
  return "rgb(" + r + ", " + g + ", " + b + ")";
}

function addText(start, end) {
  // 添加用于随机的字符
  for (let i = start.charCodeAt(); i <= end.charCodeAt(); i += 1) {
    // 根据ASCII字符得到ASCII值
    text.push(String.fromCharCode(i)); // 根据ASCII值获得ASCII字符
  }
}

function generateCode(num) {
  // 生成验证码
  for (let i = 0; i < num; i++) {
    let index = randomNum(0, text.length - 1);
    verifyCode.push(text[index]);
    let x = (w / 5) * i; // 用来控制开始写字的坐标
    let y = h / 2;

    context.font = randomNum(h / 2, h) + "px SimHei"; // 随机生成字体大小
    context.fillStyle = randomColor(100, 200); // 随机生成字体颜色

    context.translate(x, y); // 设置坐标原点
    context.fillText(text[index], 0, 0); // 写字
    context.translate(-x, -y); // 恢复坐标原点
  }
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112111344362.jpeg)

<b style="color: red">写完字后，为什么要恢复坐标原点？</b>这里写完一个字后，它的坐标也会跟着去到右下角，所以不回到起点的话，会写出斜着的一行字，单独把画布的宽高变大即可看到结果。分析可能有错，麻烦见谅。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112111344270.jpeg)

### 1.4 添加特效(文字阴影和旋转)

```js
const canvas = document.getElementById("vetifyCanvas");
const w = canvas.width;
const h = canvas.height;
const context = canvas.getContext("2d");

let text = []; //用来存放用于随机的字符
let verifyCode = []; // 用来存放验证码

context.fillStyle = randomColor(0, 255); // 设置背景为一个随机颜色
context.fillRect(0, 0, w, h); // 绘制矩形
context.textBaseline = "middle";

addText("a", "z");
addText("A", "Z");
addText("0", "9");

generateCode(4);

function randomNum(min, max) {
  return Math.floor(Math.random() * (max - min) + min); // 生成一个随机数
}

function randomColor(min, max) {
  const r = randomNum(min, max);
  const g = randomNum(min, max);
  const b = randomNum(min, max);
  return "rgb(" + r + ", " + g + ", " + b + ")";
}

function addText(start, end) {
  // 添加用于随机的字符
  for (let i = start.charCodeAt(); i <= end.charCodeAt(); i += 1) {
    // 根据ASCII字符得到ASCII值
    text.push(String.fromCharCode(i)); // 根据ASCII值获得ASCII字符
  }
}

function generateCode(num) {
  // 生成验证码
  for (let i = 0; i < num; i++) {
    let index = randomNum(0, text.length - 1);
    verifyCode.push(text[index]);

    let x = (w / 5) * i; // 用来控制开始写字的坐标
    let y = h / 2;

    context.shadowOffsetX = randomNum(-3, 3);
    context.shadowOffsetY = randomNum(-3, 3);
    context.shadowBlur = randomNum(-3, 3);
    context.shadowColor = "rgba(0, 0, 0, 0.3)";

    context.font = randomNum(h / 2, h) + "px SimHei"; // 随机生成字体大小
    context.fillStyle = randomColor(100, 200); // 随机生成字体颜色

    const deg = randomNum(-20, 20);

    context.translate(x, y); // 设置坐标原点和旋转角度
    context.rotate((deg * Math.PI) / 180);
    context.fillText(text[index], 0, 0); // 写字
    context.translate(-x, -y); // 恢复坐标原点和旋转角度
    context.rotate((-deg * Math.PI) / 180);
  }
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112111407878.jpeg)

### 1.5 添加干扰线和干扰点

```js
const canvas = document.getElementById("vetifyCanvas");
const w = canvas.width;
const h = canvas.height;
const context = canvas.getContext("2d");

let text = []; //用来存放用于随机的字符
let verifyCode = []; // 用来存放验证码

context.fillStyle = randomColor(0, 255); // 设置背景为一个随机颜色
context.fillRect(0, 0, w, h); // 绘制矩形
context.textBaseline = "middle";

addText("a", "z");
addText("A", "Z");
addText("0", "9");

generateCode(4);

ganrao();

function randomNum(min, max) {
  return Math.floor(Math.random() * (max - min) + min); // 生成一个随机数
}

function randomColor(min, max) {
  const r = randomNum(min, max);
  const g = randomNum(min, max);
  const b = randomNum(min, max);
  return "rgb(" + r + ", " + g + ", " + b + ")";
}

function addText(start, end) {
  // 添加用于随机的字符
  for (let i = start.charCodeAt(); i <= end.charCodeAt(); i += 1) {
    // 根据ASCII字符得到ASCII值
    text.push(String.fromCharCode(i)); // 根据ASCII值获得ASCII字符
  }
}

function generateCode(num) {
  // 生成验证码
  for (let i = 0; i < num; i++) {
    let index = randomNum(0, text.length - 1);
    verifyCode.push(text[index]);

    let x = (w / 5) * i; // 用来控制开始写字的坐标
    let y = h / 2;

    context.shadowOffsetX = randomNum(-3, 3);
    context.shadowOffsetY = randomNum(-3, 3);
    context.shadowBlur = randomNum(-3, 3);
    context.shadowColor = "rgba(0, 0, 0, 0.3)";

    context.font = randomNum(h / 2, h) + "px SimHei"; // 随机生成字体大小
    context.fillStyle = randomColor(100, 200); // 随机生成字体颜色

    const deg = randomNum(-20, 20);

    context.translate(x, y); // 设置坐标原点和旋转角度
    context.rotate((deg * Math.PI) / 180);
    context.fillText(text[index], 0, 0); // 写字
    context.translate(-x, -y); // 恢复坐标原点和旋转角度
    context.rotate((-deg * Math.PI) / 180);
  }
}

function ganrao() {
  // 绘制干扰线
  // for (var i = 0; i < 4; i++) {
  //   context.strokeStyle = randomColor(40, 180);
  //   context.beginPath();
  //   context.moveTo(randomNum(0, w), randomNum(0, h));
  //   context.lineTo(randomNum(0, w), randomNum(0, h));
  //   context.stroke();
  // }

  // 绘制干扰点
  for (var i = 0; i < w / 4; i++) {
    context.fillStyle = randomColor(0, 255);
    context.beginPath();
    context.arc(randomNum(0, w), randomNum(0, h), 1, 0, 2 * Math.PI);
    context.fill();
  }
}
```

效果：

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112111344263.jpeg)

### 1.6 增加点击可以实现刷新的功能

用了`context.save(); `和`context.restore();`来实现清空画布，重画的效果

![image-20211211235444063](https://s2.loli.net/2021/12/11/mF5d9ua7TxtKRUE.png)

完整代码：[验证码 (codepen.io)](https://codepen.io/13535944743/pen/Yzxbvod)

## 2. 生成 32 位随机码

在开展项目会议时，听到了数据表那边的 id 应该使用通用的生成 32 位随机码的方法，而不是使用 int 型+自增后，就想试一下自己实现生成 32 位随机码。

首先，需要获取一个用于生成随机码的字符的数组，这里可以使用手敲法，但太累了。还是可以用生成验证码时的方法。因为<b style="color: red">js 的字符无法自增，所以可以使用 charCodeAt()函数把字符转换成 ASCII 值之后，再进行自增操作，当然，每一次遍历拿到的值都是 ASCII 值了，所以再通过 String.fromCharCharCode()函数把 ASCII 值又转换成 ASCII 字符。</b>

具体实现如下：

```js
function getContent(start, end) {
  let arr = [];
  for (let i = start.charCodeAt(); i <= end.charCodeAt(); i++) {
    arr.push(String.fromCharCode(i));
  }
  return arr;
}
```

得到用于生成随机码的字符后，就可以直接通过随机函数 random()来实现生成 32 位随机码。

```js
function getRandom() {
  let arr = [];
  arr = arr.concat(getContent("a", "z"));
  arr = arr.concat(getContent("0", "9"));

  let id = "";
  for (let i = 0; i < 32; i++) {
    id += arr[parseInt(Math.random() * 36)];
  }
  console.log(id);
}

function getContent(start, end) {
  let arr = [];
  for (let i = start.charCodeAt(); i <= end.charCodeAt(); i++) {
    arr.push(String.fromCharCode(i));
  }
  return arr;
}

getRandom();
```

完整实现如下：

```js
function getRandom() {
  let arr = [];
  arr = arr.concat(getContent("a", "z"));
  arr = arr.concat(getContent("0", "9"));

  let id = "";
  for (let i = 0; i < 32; i++) {
    id += arr[parseInt(Math.random() * 36)];
  }
  console.log(id);
}

function getContent(start, end) {
  let arr = [];
  for (let i = start.charCodeAt(); i <= end.charCodeAt(); i++) {
    arr.push(String.fromCharCode(i));
  }
  return arr;
}

getRandom();
```

<b style="color: red">可能常用到的方法：记录一下</b>。有可能有更好的方法，发现后再更新。

```js
function getContent(start, end) {
  let arr = [];
  for (let i = start.charCodeAt(); i <= end.charCodeAt(); i++) {
    arr.push(String.fromCharCode(i));
  }
  return arr;
}
```

参考链接：[JS 实现图片验证码功能——用户输入验证码 - vickylinj - 博客园 (cnblogs.com)](https://www.cnblogs.com/vickylinj/p/12103308.html)
