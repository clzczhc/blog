---
title: canvas 渐变动效
date: 2024-08-02 22:00:00
categories: 前端
tags:
  - Canvas
  - 动画
---

# canvas 渐变动效

## 前言

> 看到一个渐变动效的库[granimjs](https://github.com/sarcadass/granim.js)的一个效果，简单看了下源码，感觉还挺有意思的，自己捣鼓一个简单版玩玩

## 核心

1. 利用`canvas`的`createLinearGradient`创建渐变，以及`addColorStop`添加偏移、颜色。
2. 渐变动效是将颜色转换为 rgb 色系，计算出颜色之前的差值，然后慢慢根据时间戳的变化，逐渐修改颜色

## canvas 绘制渐变

```js
const canvas = document.querySelector("canvas");
const ctx = canvas.getContext("2d");

canvas.width = 200;
canvas.height = 200;

const gradient = ctx.createLinearGradient(0, 0, canvas.width, canvas.height); // 创建左上到右下的渐变
gradient.addColorStop(0, "#ff0000");
gradient.addColorStop(1, "#0000ff"); // 添加渐变一半一半

ctx.fillStyle = gradient;
ctx.fillRect(0, 0, canvas.width, canvas.height);
```

![](https://www.clzczh.top/CLZ_img/images/202407281516218.png)

## 渐变动效

### `makeGradient`创建渐变

```js
const canvas = document.querySelector("canvas");
const ctx = canvas.getContext("2d");

canvas.width = 600;
canvas.height = 300;

function makeGradient() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  const gradient = ctx.createLinearGradient(0, 0, canvas.width, canvas.height);

  paintColorPair.forEach((paintColor, index) => {
    gradient.addColorStop(
      index === 0 ? 0 : 1,
      `rgb(${paintColor[0]}, ${paintColor[1]}, ${paintColor[2]})`
    );
  });

  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, canvas.width, canvas.height);
}
makeGradient();
```

### `convertHextoRgb`将 hex 字符串转化为 rgb 数组

```js
function convertHextoRgb(hex) {
  const result = /^#([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result
    ? [
        parseInt(result[1], 16),
        parseInt(result[2], 16),
        parseInt(result[3], 16),
      ]
    : null;
}

const hex = "#ff0000";
const rgbColors = convertHextoRgb(hex);
console.log(rgbColors); // [255, 0, 0]
```

### `getRgbDiff`获取两个颜色之间的差异

```js
function getColorDiff(colorA, colorB) {
  const colorDiff = [];

  for (let i = 0; i < 3; i++) {
    colorDiff.push(colorB[i] - colorA[i]);
  }

  return colorDiff;
}

const rgb1 = [100, 150, 200];
const rgb2 = [120, 160, 180];

const rgbDiff = getColorDiff(rgb1, rgb2);
console.log(rgbDiff); // [ 20, 10, -20 ]
```

> 获取两个颜色之前的差异之后，就可以较平滑地过渡颜色了。比如上面的`rgb1`过渡到`rgb2`，如果中间只过渡一次的话，那么中间状态就是`[100 + 10, 150 + 5, 200 - 10]`

### `animateColors`搭配动效，实现过渡动画

> 将之前得到的颜色差值，根据当前时间戳占切换颜色时间的比例以百分比的形式增加。得到当前时间戳对应的色值后，绘制渐变

```js
const duration = 5000; // 切换颜色的时间

let previousTimeStamp = 0; // 记录上一次的时间戳
let process = 0; // 进度，通过process /duration获取颜色累加的百分比
function animateColor(timestamp) {
  process = process + (timestamp - previousTimeStamp);
  previousTimeStamp = timestamp;

  const progressPercent = ((process / duration) * 100).toFixed(2);

  // 获取绘制颜色（颜色 + 偏移值）。偏移值会根据时间戳的变化，从0%到100%
  paintColorPair = rgbColorPairs[currentPairIndex].map((rgbColor, i) => {
    return rgbColor.map((item, j) => {
      return (
        item +
        Math.round(
          (rgbDiffPairs[currentPairIndex][i][j] / 100) * progressPercent
        )
      );
    });
  });

  // 绘制
  makeGradient();

  if (progressPercent < 100) {
    requestAnimationFrame(animateColor);
  } else {
    process = 0; // 进度超过百分比，重置进度

    if (currentPairIndex === rgbColorPairs.length - 1) {
      currentPairIndex = 0; // 更新到下一组颜色
    } else {
      currentPairIndex++;
    }

    requestAnimationFrame(animateColor);
  }
}

requestAnimationFrame(animateColor);
```

### 效果

![](https://www.clzczh.top/CLZ_img/images/202408020108532.gif)

> 稍微减少了切换颜色的时间

### 完整代码

html

```html
<!DOCTYPE html>
<html>
  <head>
    <style></style>
  </head>

  <body>
    <canvas></canvas>

    <script src="./helper.js"></script>
    <script src="./granim.js"></script>
  </body>
</html>
```

help.js

```js
function convertHextoRgb(hex) {
  const result = /^#([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result
    ? [
        parseInt(result[1], 16),
        parseInt(result[2], 16),
        parseInt(result[3], 16),
      ]
    : null;
}

function getColorDiff(colorA, colorB) {
  const colorDiff = [];

  for (let i = 0; i < 3; i++) {
    colorDiff.push(colorB[i] - colorA[i]);
  }

  return colorDiff;
}
```

granim.js

```js
const hexColorPairs = [
  ["#ff0000", "#0000ff"],
  ["#000000", "#ffffff"],
];

const rgbColorPairs = hexColorPairs.map((hexColorPair) => {
  return hexColorPair.map((hex) => convertHextoRgb(hex));
});

const rgbDiffPairs = rgbColorPairs.map((rgbColorPair, i) => {
  return rgbColorPair.map((rgbColor, j) => {
    if (i === rgbColorPairs.length - 1) {
      return getColorDiff(rgbColor, rgbColorPairs[0][j]);
    } else {
      return getColorDiff(rgbColor, rgbColorPairs[i + 1][j]);
    }
  });
});

let currentPairIndex = 0;
let paintColorPair = rgbColorPairs[currentPairIndex];

const canvas = document.querySelector("canvas");
const ctx = canvas.getContext("2d");

canvas.width = 600;
canvas.height = 300;

function makeGradient() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  const gradient = ctx.createLinearGradient(0, 0, canvas.width, canvas.height);

  paintColorPair.forEach((paintColor, index) => {
    gradient.addColorStop(
      index === 0 ? 0 : 1,
      `rgb(${paintColor[0]}, ${paintColor[1]}, ${paintColor[2]})`
    );
  });

  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, canvas.width, canvas.height);
}
makeGradient();

const duration = 5000; // 切换颜色的时间

let previousTimeStamp = 0; // 记录上一次地时间戳
let process = 0; // 进度，通过process /duration获取颜色累加的百分比
function animateColor(timestamp) {
  process = process + (timestamp - previousTimeStamp);
  previousTimeStamp = timestamp;

  const progressPercent = ((process / duration) * 100).toFixed(2);

  // 获取绘制颜色（颜色 + 偏移值）。偏移值会根据时间戳的变化，从0%到100%
  paintColorPair = rgbColorPairs[currentPairIndex].map((rgbColor, i) => {
    return rgbColor.map((item, j) => {
      return (
        item +
        Math.round(
          (rgbDiffPairs[currentPairIndex][i][j] / 100) * progressPercent
        )
      );
    });
  });

  // 绘制
  makeGradient();

  if (progressPercent < 100) {
    requestAnimationFrame(animateColor);
  } else {
    process = 0; // 进度超过百分比，重置进度

    if (currentPairIndex === rgbColorPairs.length - 1) {
      currentPairIndex = 0; // 更新到下一组颜色
    } else {
      currentPairIndex++;
    }

    requestAnimationFrame(animateColor);
  }
}

requestAnimationFrame(animateColor);
```

## 小彩蛋

在[granimjs](https://github.com/sarcadass/granim.js)中还发现了，还可以使用`globalCompositeOperation`来实现图片和渐变混合的效果。
[globalCompositeOperation](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation)

可以选择合适的类型，玩出花样。

```js
const image = new Image();
image.src = "./imgs/bg-forest.jpg";
ctx.globalCompositeOperation = "color-burn";

function makeGradient() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.drawImage(image, 0, 0, canvas.width, canvas.height);

  const gradient = ctx.createLinearGradient(0, 0, canvas.width, canvas.height);
  // ...
}
```

![](https://www.clzczh.top/CLZ_img/images/202408020120151.gif)
