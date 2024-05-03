---
title: 拾色器的使用、实现
date: 2024-05-03 16:40:00
keywords: 拾色器
categories: "前端"
tags:
  - Canvas
---

# 拾色器的使用、实现

## 前言

> 调研取色器组件的时候，看到原生`input:color`就有拾色器功能，觉得有点意思，稍微“玩”一下。

## EyeDropper

> 实验性技术，兼容性不是特别好。
> ![](https://www.clzczh.top/CLZ_img/images/202404172246428.png)

使用方法很简单，实例化一个`EyeDropper`对象，调用`open`方法即可，`open`方法返回`Promise`对象，在`then`方法中可以获取到拾取的颜色。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>拾色器</title>
    <style>
      .container {
        width: 100px;
        height: 100px;
      }
    </style>
  </head>

  <body>
    <button>打开拾色器</button>
    <div class="container"></div>

    <script>
      document.querySelector("button").addEventListener("click", () => {
        const eyeDropper = new EyeDropper();

        eyeDropper
          .open()
          .then((result) => {
            const color = result.sRGBHex;
            document.querySelector(".container").style.backgroundColor = color;
          })
          .catch((e) => console.log(e));
      });
    </script>
  </body>
</html>
```

![](https://www.clzczh.top/CLZ_img/images/202404172302971.gif)

[更多](https://developer.mozilla.org/zh-CN/docs/Web/API/EyeDropper)

## Canvas 实现

> 利用 dom2svg 把 dom 节点转成图片，并且绘制到 canvas 上，然后利用`ctx.getImageData`获取图片的颜色信息来实现。

html

```html
<button id="open-btn">打开拾色器</button>
<div class="container">
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
</div>

<script src="./dom2svg.js"></script>
<script src="./index.js"></script>
```

css

```css
body {
  margin: 0;
  padding: 0;
  background: #fff;
}

.container {
  display: flex;
  flex-wrap: wrap;
  width: 400px;
  height: 400px;
}

.box {
  width: 200px;
  height: 200px;
}

.box:nth-child(1) {
  background-color: red;
}

.box:nth-child(2) {
  background-color: blue;
}

.box:nth-child(3) {
  background-color: purple;
}

.box:nth-child(4) {
  background-color: pink;
}

#open-btn {
  cursor: pointer;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
```

js

```js
let canvas;
let ctx;

const handleMouseDown = (e) => {
  const { data } = ctx.getImageData(e.clientX, e.clientY, 1, 1);

  const [r, g, b] = data;
  const a = data[3] / 255;

  console.log(`rgba(${r}, ${g}, ${b}, ${a})`);
};

document.querySelector("#open-btn").addEventListener("click", async () => {
  canvas = document.createElement("canvas");
  const width = document.body.clientWidth;
  const height = document.body.clientHeight;

  canvas.width = width;
  canvas.height = height;

  canvas.style.width = `${width}px`;
  canvas.style.height = `${height}px`;

  ctx = canvas.getContext("2d");

  const img = await domToSvg(document.body);
  ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
  document.body.appendChild(canvas);

  canvas.addEventListener("mousedown", handleMouseDown);
});
```

简单的取色器这样子就实现了。
![](https://www.clzczh.top/CLZ_img/images/202405031558269.gif)

> 这里插一嘴，dom2svg 后通过 canvas 的`drawImage`会有点糊。网上搜到的全是设置`canvas`的宽高放大，在但是样式不放大，就能实现一个比较好的清晰效果，但是实际上治标不治本，`getImageData`的时候还是会是放大后的。比如设置`canvas`的宽高为 2 倍，这样子，点击蓝色区域的时候，`imageData`还是红色，点击蓝色右边的区域的时候才是蓝色。没有找到一个能真正的避免 canvas 绘制图片不糊的方法。
> ![](https://www.clzczh.top/CLZ_img/images/202405031603673.gif)
> 如果不需要用到，`ctx.getImageData`这类方法，这种方式也不是不能用（确实，肉眼上看是变清晰了）

### 获取鼠标周围颜色信息，并放大到放大镜中

> 原理就是利用，`getImageData`获取鼠标周围指定大小区域的颜色信息，然后将颜色绘制到另一个放大镜 canvas 中，颜色信息放大，比如一个像素点的颜色变成 8 个像素点来绘制。

color-util.js

```js
const getPointColor = (ctx, x, y) => {
  const { data } = ctx.getImageData(x, y, 1, 1);

  const [r, g, b] = data;
  const a = data[3] / 255;

  return `rgba(${r}, ${g}, ${b}, ${a})`;
};

const getRectColors = (ctx, x, y, size) => {
  const halfSize = Math.floor(size / 2);
  const { data: imageData } = ctx.getImageData(
    x - halfSize,
    y - halfSize,
    size,
    size
  );

  const colors = [];
  for (let i = 0; i < size; i++) {
    if (!colors[i]) {
      colors[i] = [];
    }

    for (let j = 0; j < size; j++) {
      const position = (size * i + j) * 4; // 获取点阵信息。公式：(width * y + x) * 4，先遍历yz轴，后遍历x轴
      const r = imageData[position];
      const g = imageData[position + 1];
      const b = imageData[position + 2];
      const a = imageData[position + 3] / 255;
      colors[i][j] = `rgba(${r}, ${g}, ${b}, ${a})`;
    }
  }

  return {
    colors,
  };
};

// 1个像素放大成8个像素
const drawMagnifier = (colors, size = 8) => {
  const count = colors.length;

  const diameter = size * count;
  const radius = diameter / 2;

  const canvas = document.createElement("canvas");
  canvas.width = diameter;
  canvas.height = diameter;

  canvas.style = `
    position: static;
    width: ${diameter}px;
    height: ${diameter}px;
  `;

  const ctx = canvas.getContext("2d");

  colors.forEach((row, i) =>
    row.forEach((color, j) => {
      ctx.fillStyle = color;
      ctx.fillRect(j * size, i * size, size, size);
    })
  );

  ctx.fillStyle = "#000";
  ctx.lineWidth = 1;
  ctx.strokeRect(radius - size / 2, radius - size / 2, size, size);

  return canvas;
};

const getColorContainer = ({ colors, containerDOM, pos }) => {
  let container = containerDOM;

  if (!container) {
    const magnifierContainer = document.createElement("div");
    container = magnifierContainer;
  }

  // pointer-events: none; 取消container的事件，避免影响大canvas的事件
  container.style = `
    position: fixed;
    left: ${pos.x}px;
    top: ${pos.y}px;
    transform: translate(-50%, -50%);
    z-index: 999;
    pointer-events: none; 
  `;

  container.innerHTML = "";

  if (!container.classList.contains("maginifier-container")) {
    container.classList.add("maginifier-container");
  }

  const maginifierCanvas = drawMagnifier(colors);
  container.appendChild(maginifierCanvas);

  return container;
};
```

添加`mouseMove`事件。
index.js

```js
const handleMousemove = (e) => {
  const { colors } = getRectColors(ctx, e.clientX, e.clientY, magnifierSize);
  const pos = {
    x: e.clientX,
    y: e.clientY,
  };

  const colorContainer = getColorContainer({
    colors,
    pos,
    containerDOM: magnifierContainer,
  });

  if (!magnifierContainer) {
    magnifierContainer = colorContainer;
    document.body.appendChild(colorContainer);
  }
};
```

放大镜容器的样式。

```css
.maginifier-container {
  width: 160px;
  height: 160px;
  box-sizing: border-box;
  border: 1px solid #000;
  border-radius: 50%;
  overflow: hidden;
}
```

![](https://www.clzczh.top/CLZ_img/images/202405031630757.gif)

### 完整代码

index.html

```html
<button id="open-btn">打开拾色器</button>
<div class="container">
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
</div>

<script src="./dom2svg.js"></script>
<script src="./color-util.js"></script>
<script src="./index.js"></script>
```

[dom2svg](https://www.clzczh.top/2024/04/21/dom-to-svg/)

color-util.js

```js
const getPointColor = (ctx, x, y) => {
  const { data } = ctx.getImageData(x, y, 1, 1);

  const [r, g, b] = data;
  const a = data[3] / 255;

  return `rgba(${r}, ${g}, ${b}, ${a})`;
};

const getRectColors = (ctx, x, y, size) => {
  const halfSize = Math.floor(size / 2);
  const { data: imageData } = ctx.getImageData(
    x - halfSize,
    y - halfSize,
    size,
    size
  );

  const colors = [];
  for (let i = 0; i < size; i++) {
    if (!colors[i]) {
      colors[i] = [];
    }

    for (let j = 0; j < size; j++) {
      const position = (size * i + j) * 4; // 获取点阵信息。公式：(width * y + x) * 4，先遍历yz轴，后遍历x轴
      const r = imageData[position];
      const g = imageData[position + 1];
      const b = imageData[position + 2];
      const a = imageData[position + 3] / 255;
      colors[i][j] = `rgba(${r}, ${g}, ${b}, ${a})`;
    }
  }

  return {
    colors,
  };
};

// 1个像素放大成8个像素
const drawMagnifier = (colors, size = 8) => {
  const count = colors.length;

  const diameter = size * count;
  const radius = diameter / 2;

  const canvas = document.createElement("canvas");
  canvas.width = diameter;
  canvas.height = diameter;

  canvas.style = `
    position: static;
    width: ${diameter}px;
    height: ${diameter}px;
  `;

  const ctx = canvas.getContext("2d");

  colors.forEach((row, i) =>
    row.forEach((color, j) => {
      ctx.fillStyle = color;
      ctx.fillRect(j * size, i * size, size, size);
    })
  );

  ctx.fillStyle = "#000";
  ctx.lineWidth = 1;
  ctx.strokeRect(radius - size / 2, radius - size / 2, size, size);

  return canvas;
};

const getColorContainer = ({ colors, containerDOM, pos }) => {
  let container = containerDOM;

  if (!container) {
    const magnifierContainer = document.createElement("div");
    container = magnifierContainer;
  }

  // pointer-events: none; 取消container的事件，避免影响大canvas的事件
  container.style = `
    position: fixed;
    left: ${pos.x}px;
    top: ${pos.y}px;
    transform: translate(-50%, -50%);
    z-index: 999;
    pointer-events: none; 
  `;

  container.innerHTML = "";

  if (!container.classList.contains("maginifier-container")) {
    container.classList.add("maginifier-container");
  }

  const maginifierCanvas = drawMagnifier(colors);
  container.appendChild(maginifierCanvas);

  return container;
};
```

index.js

```js
let canvas;
let ctx;

let magnifierContainer = null;
const magnifierSize = 20;

const handleMousemove = (e) => {
  const { colors } = getRectColors(ctx, e.clientX, e.clientY, magnifierSize);
  const pos = {
    x: e.clientX,
    y: e.clientY,
  };

  const colorContainer = getColorContainer({
    colors,
    pos,
    containerDOM: magnifierContainer,
  });

  if (!magnifierContainer) {
    magnifierContainer = colorContainer;
    document.body.appendChild(colorContainer);
  }
};

const handleMouseDown = (e) => {
  const color = getPointColor(ctx, e.clientX, e.clientY);
  console.log(color);

  document.body.removeChild(document.querySelector("canvas"));

  magnifierContainer = null;
  document.body.removeChild(document.querySelector(".maginifier-container"));
};

document.querySelector("#open-btn").addEventListener("click", async () => {
  canvas = document.createElement("canvas");
  const width = document.body.clientWidth;
  const height = document.body.clientHeight;

  canvas.width = width;
  canvas.height = height;

  canvas.style.width = `${width}px`;
  canvas.style.height = `${height}px`;

  ctx = canvas.getContext("2d");

  const img = await domToSvg(document.body);
  ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
  document.body.appendChild(canvas);

  canvas.addEventListener("mousemove", handleMousemove);
  canvas.addEventListener("mousedown", handleMouseDown);
});
```
