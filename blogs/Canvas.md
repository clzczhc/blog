---
title: Canvas简单入门
categories: 前端
date: 2022-05-29 23:18:36
tags:
  - JavaScript
---

# Canvas简单入门

创建`canvas`至少需要提供`width`和`height`属性，才能通知浏览器需要多大位置画图。标签的内容是后备数据，在浏览器不支持`canvas`元素时显示。

```html
<canvas id="mycanvas" width="200" height="200">haha</canvas>
```

可以通过`if(canvas.getContext)`来判断浏览器是否支持`canvas`。

通过`canvas.getContext('2d')`可以获取 2D 绘图上下文。2D 绘图上下文提供了绘制 2D 图形的方法。左边原点(0, 0)在` canvas`元素的左上角，x 坐标向右增长，y 坐标向下增长。



## 从画布上导出一张 PNG 格式的图片

```html
<body>
  <canvas id="mycanvas" width="200" height="200">haha</canvas>

  <script>
    const mycanvas = document.getElementById("mycanvas");

    // 确保浏览器支持canvas
    if (mycanvas.getContext) {
      // 获得图像的数据URI
      const imgURI = mycanvas.toDataURL("image/png");
      console.log(imgURI);
    }
  </script>
</body>
```

![image-20220522110709840](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292320516.png)

我们查看控制台可以发现，输出了一串`base64`编码，也就是说，`canvas.toDataURL`就是将画布` canvas`转换成`base64`编码。



## 填充与描边

* 填充就是以特定的样式填充形状，包括颜色、渐变、图像

* 描边就是只给形状边界着色。

  

  

显示效果取决于两个属性：`fillStyle`和`strokeStyle`。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  context.fillStyle = "#000";
  context.strokeStyle = "red";
}
```

没有效果？
别急，这是因为我们只是设置了填充和描边而已，想要它生效，还需要绘制出来才能有效果。



## 绘制矩形

与绘制矩形相关的方法有三个。它们都接收 4 个参数：矩形 x 坐标、矩形 y 坐标、矩形宽度和矩形高度。(单位是像素，但是传参时不需要传单位)

1. `fillRect`
2. `strokeRect`
3. `clearRect`



### `fillRect`：绘制并填充矩形

`fillRect`：以指定颜色在画布上绘制并填充矩形，填充色使用`fillStyle`来设置。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  context.fillStyle = "pink";
  context.fillRect(10, 10, 50, 50);

  context.fillStyle = "rgba(0, 0, 0, .1)";
  context.fillRect(30, 30, 50, 50);
}
```

![image-20220522110314237](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292320709.png)



### `stokeRect`：绘制矩形轮廓

`stokeRect`：绘制矩形轮廓，颜色由`strokeStyle`来指定。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  context.strokeStyle = "red";
  // 设置描边宽度
  context.lineWidth = 5;
  context.strokeRect(10, 10, 50, 50);

  context.strokeStyle = "blue";
  context.fillStyle = "rgba(0, 0, 0, .1)";
  context.strokeRect(30, 30, 50, 50);
}
```

![image-20220522110410841](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292319241.png)



### `clearRect`：擦除画布中某个区域

`clearRect`：擦除画布中某个区域，把擦除的区域变透明。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  context.fillStyle = "red";
  context.fillRect(0, 0, 200, 200);

  context.clearRect(50, 50, 100, 100);
}
```

![image-20220522110449360](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292320228.png)



## 绘制路径

绘制路径需要先调用`beginPath`，表示要开始绘制路径，再调用以下方法来绘制路径。

- `lineTo(x, y)`：绘制一条从上一个点到(x, y)的直线
- `moveTo(x, y)`：不绘制线条，只是把画笔移动到(x, y)
- [更多](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D)

绘制完路径后，可以指定`fillStyle`属性并调用`fill`方法来填充路径，也可以指定`strokeStyle`属性并调用`stoke`方法来描画路径。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  // 创建路径
  context.beginPath();

  // 绘制圆弧，参数分别是圆心x坐标、圆形y坐标、圆弧半径、圆弧起始点(单位：弧度)、圆弧终点(单位：弧度)、绘制方向(false为顺时针绘制，true为逆时针绘制)
  context.arc(100, 100, 99, 0, 2 * Math.PI, true);

  // context.fillStyle = 'pink'
  // context.fill()

  context.strokeStyle = "pink";
  context.stroke();
}
```

![image-20220522111214015](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292321089.png)



还可以调用`clip`方法创建一个新的剪切区域。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  // 创建路径
  context.beginPath();

  // 绘制圆弧，参数分别是圆心x坐标、圆形y坐标、圆弧半径、圆弧起始点(单位：弧度)、圆弧终点(单位：弧度)、绘制方向(false为顺时针绘制，true为逆时针绘制)
  context.arc(100, 100, 50, 0, 2 * Math.PI, true);

  context.fillStyle = "pink";
  context.clip();

  context.fillRect(0, 0, 100, 100);
}
```

上面的扇形怎么出来的呢？
我们可以把`clip`变成`fill`，看下没有被剪切的话，是什么样子。

![img](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292321085.png)



也就是说，实际上剪切就是两个图形相交部分。

如果使用`lineTo`需要注意：没有设置`moveTo`时，这个位置并不是(0, 0)，而是空，所以第一次的`lineTo`没法画出结果。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");
  // 创建路径
  context.beginPath();

  // context.moveTo(0, 0);
  context.lineTo(100, 50);
  context.lineTo(200, 0);

  context.lineWidth = 8;
  context.strokeStyle = "pink";

  // 描画路径
  context.stroke();
}
```



没有`moveTo`：

![image-20220522112404156](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292321928.png)



有`moveTo`：

![image-20220522112440786](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292321959.png)



### beginPath 的作用

上面的例子中，`beginPath`并没有作用，也就是说上面的例子中，其实有没有`beginPath`都一样。那么`beginPath`有什么作用呢？

**`beginPath`表示下面绘制的图形是一个新的路径**。具体看下实例。

```js
const context = mycanvas.getContext("2d");
// 创建路径
context.beginPath();

context.moveTo(0, 0);
context.lineTo(100, 50);

context.lineWidth = 8;
context.strokeStyle = "pink";

// 描画路径
context.stroke();

context.lineTo(200, 0);
context.strokeStyle = "purple";
context.stroke();
```

![image-20220522112513152](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292320049.png)



想要的效果是画出两条不一样颜色的线，但是最后是一种颜色折线，这是因为我们只是用了一次`beginPath`，所以就会把这两条线当成同一个路径，最后调用的`stroke`就会把原本是粉色的线再用紫色画一遍，所以最终的效果就是只有一条折线。

所以需要使用`beginPath`创建新路径，新的路径还是会有**没有设置`moveTo`时，这个位置并不是(0, 0)，而是空**的问题，所以需要使用`moveTo`设置位置

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");
  // 创建路径
  context.beginPath();

  context.moveTo(0, 0);
  context.lineTo(100, 50);

  context.lineWidth = 8;
  context.strokeStyle = "pink";
  context.stroke();

  context.beginPath();
  // 创建新的路径，需要重新设置位置
  context.moveTo(100, 50);
  context.lineTo(200, 0);
  context.strokeStyle = "purple";
  context.stroke();
}
```

![image-20220522112538976](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292321827.png)



### closePath 的作用

有可能会陷进`closePath`是结束路径的误区，认为`closePath`就是`beginPath`的配套。但是`closePath`和`beginPath`并不是配套的，它们的功能不一样。所以`closePath`之后的路径也不是新的路径，只有`beginPath`才行。

而`closePath`的作用是将最近绘制的路径闭合，**和之前有没有`beginPath`无关**。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  // context.beginPath();  // 有无`beginPath`都没有影响

  context.moveTo(10, 10);
  context.lineTo(100, 50);
  context.lineTo(20, 70);
  context.closePath();

  context.lineWidth = 8;
  context.strokeStyle = "pink";
  context.stroke();
}
```

![image-20220522112629070](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292320241.png)

上面我们只绘制了两条线，但是最终得到的结果是一个三角形，这是因为我们使用`closePath`把最近绘制的路径闭合了。



## 绘制文本

绘制文本有两种方法。

1. `fillText`：使用`fillStyle`属性绘制文本
2. `strokeText`：使用`strokeStyle`属性绘制文本

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  context.moveTo(10, 10);
  context.lineTo(150, 75);
  context.lineTo(30, 100);
  context.closePath();

  context.lineWidth = 1;
  context.strokeStyle = "pink";

  context.fillStyle = "purple";
  context.fillText("CLZ", 50, 60);
  context.strokeText("CLZ", 50, 80);

  context.stroke();
}
```

![image-20220522112741066](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292320806.png)



可以通过`font`、`textAlign`、`textBaseline`属性设置文本的字体、对齐方式、基线。

示例：

```js
context.font = "700 16px Arial";
```

![img](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292320835.png)



`textAlign`：

- 如果是`start`，那么 x 坐标就是文本的左侧坐标
- 如果是`center`，那么 x 坐标就是文本的中心点坐标
- 如果是`end`，那么 x 坐标就是文本的右侧坐标

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  context.moveTo(10, 10);
  context.lineTo(150, 75);
  context.lineTo(30, 100);
  context.closePath();

  context.lineWidth = 1;
  context.strokeStyle = "pink";

  context.font = "700 16px Arial";
  context.fillStyle = "purple";

  context.textAlign = "start";
  context.strokeText("CLZ", 50, 50);

  context.textAlign = "center";
  context.fillText("CLZ", 50, 65);

  context.textAlign = "end";
  context.strokeText("CLZ", 50, 80);

  context.stroke();
}
```

![image-20220522112924112](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292320368.png)

**`textBaseline`类似**



## 变换

2D 换图上下文支持所有常见的绘制变化。
`rotate(a)`：围绕原点把图像旋转 a 弧度
`scale(x, y)`：缩放图像
`translate(x, y)`：移动原点

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  // 创建路径
  context.beginPath();

  // 绘制圆弧，参数分别是圆心x坐标、圆形y坐标、圆弧半径、圆弧起始点(单位：弧度)、圆弧终点(单位：弧度)、绘制方向(false为顺时针绘制，true为逆时针绘制)
  context.arc(100, 100, 50, 0, 2 * Math.PI, true);

  context.lineWidth = "8";
  context.strokeStyle = "pink";

  // 移动原点
  context.translate(100, 100);

  // 旋转
  context.rotate(Math.PI);

  // 缩放
  context.scale(0.75, 0.75);

  // 因为已经移动过原点了，所以这时候(0, 0)就是圆心
  context.moveTo(0, 0);
  context.lineTo(25, 30);

  context.stroke();
}
```

![image-20220522113048545](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292321235.png)

上面的例子中，已经把很多变化都使用上了，如果想要了解具体例子可以注释掉其他部分。



### save 和 restore 的作用

`save`方法可以保存应用到绘图上下文的设置和变换，不保存绘图上下文的内容。后续可以通过`restore`方法，恢复上下文的设置和变换。`save`和`restore`的使用类似于栈，后进先出。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  context.fillStyle = "red";
  context.save();

  context.fillStyle = "blue";
  context.translate(100, 100);
  context.save();

  context.fillStyle = "purple";
  context.translate(-100, -100);
  context.fillRect(0, 0, 100, 100);

  context.restore();
  context.fillRect(0, 0, 100, 100);

  context.restore();
  context.fillRect(100, 0, 100, 100);

  context.restore();
  context.fillRect(0, 100, 100, 100);
}
```

![image-20220522113308234](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292321143.png)



分析：设 XXX 为**绘图上下文的设置和变化**

1. 设置填充色为红色，`save`保存
2. 设置填充色为蓝色，移动原点，`save`保存
3. 设置填充色为紫色，移动原点，画出紫色的矩形
4. `restore`恢复**XXX**，此时，原点为(100, 100)，填充色为蓝色。画出蓝色的矩形
5. `restore`恢复**XXX\*\*，此时，原点为(0, 0)，填充色为红色。画出红色的矩形
6. `restore`已经没有保存的**XXX**，所以**XXX**不会变化



## 绘制图像

```html
<img src="./avatar.png" alt="">
<canvas id="mycanvas" width="200" height="200">haha</canvas>
```



通过`drawImage`把 HTML 的 img 元素或另一个 canvas 元素绘制到当前画布中。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  // 获取图像
  const img = document.images[0];

  // 在画布的坐标出绘制图像，此时图像和原来的图像一样大，指的是原文件的大小
  // context.drawImage(img, 10, 10)

  // 传入另外两个参数，设置绘制图像的宽高
  context.drawImage(img, 10, 10, 100, 100);
}
```

只传3个参数，画到画布上的跟原来的图像一样大，但画布没那么大。所以会只有一部分。

![image-20220522113739829](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292322993.png)



传入五个参数，可以让设置图像的宽高，显示完整的图像。

![image-20220522113901338](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292321509.png)



### 去掉DOM树上的img

上面的做法是需要`html`中有`img`元素才能执行的.实际上,我们也可以通过`image`对象来实现。

即获取图像不再是通过`document.images[0]`，而是

```js
const img = new Image();
img.src = "./avatar.png";
```

另外，绘制图像应该在`img`的`load`事件回调中调用。

```js
const img = new Image();
img.src = "./avatar.png";
img.onload = () => {
  // 传入另外两个参数，设置绘制图像的宽高
  context.drawImage(img, 10, 10, 100, 100);
};
```

![image-20220522114113263](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292322851.png)



还可以接收 9 个参数，实现把原始图像的一部分绘制到画布上。
如：`context.drawImage(img, 0, 10, 50, 50, 0, 100, 20, 30)`，从原始图像的(0, 10)开始，50 像素宽、50 像素高，画到画布上(0, 100)开始，宽 40 像素、高 60 像素。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  // 获取图像
  const img = document.images[0];

  // // 9个参数
  context.drawImage(img, 0, 10, 300, 300, 100, 100, 40, 40);
}
```

![image-20220522114232024](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292321579.png)



### 下载图像

操作的结果可以使用`canvas.toDataURL()`方法获取。

再搭配下载图片的方式就能实现下载图片。(这里用的是`a`标签方法)

```js
const a = document.createElement("a");
a.href = mycanvas.toDataURL();

// 获取源图片的名字
a.download = img.src.split("/")[img.src.split("/").length - 1];

a.click();
```



## 阴影

设置好阴影有关的属性值，就能够自动为要绘制的形状或路径生成阴影

- `shadowOffsetX`：阴影相对于形状或路径的 x 坐标偏移。默认为 0
- `shadowOffsetY`：阴影相对于形状或路径的 y 坐标偏移。默认为 0
- `shadowBlur`：阴影的模糊量。默认值为 0，表示不模糊
- `shadowColor`：阴影的颜色。默认为黑色

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  context.shadowOffsetX = 5;
  context.shadowOffsetY = 10;
  context.shadowBlur = 5;
  context.shadowColor = "rgba(0, 0, 0, .2)";

  context.fillStyle = "red";
  context.fillRect(0, 0, 50, 50);

  context.moveTo(100, 100);
  context.lineTo(180, 20);

  context.lineWidth = 12;
  context.stroke();
}
```

![image-20220522114457032](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292323370.png)



## 渐变

### 线性渐变

线性渐变可以调用上下文的`createLinearGradient`方法，接收四个参数：起点 x 坐标、起点 y 坐标、终点 x 坐标、终点 y 坐标，创建`CanvasGradient`对象。

有了渐变对象后，就需要添加渐变色标了，通过`addColorStop`可以添加色标，第一个参数范围为 0~1，第二个参数是 CSS 颜色字符串。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  const gradient = context.createLinearGradient(10, 10, 180, 180);

  gradient.addColorStop(0, "red");
  gradient.addColorStop(0.5, "blue");
  gradient.addColorStop(1, "purple");

  context.fillStyle = gradient;
  context.fillRect(0, 0, 200, 200);
}
```

![image-20220522114554072](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292322784.png)



为了让渐变覆盖整个矩形，渐变的坐标和矩形的坐标应该搭配合适，不然只会显示部分渐变。

还可以调用上下文的`createRadialGradient`方法来创建径向渐变。接收 6 个参数，前 3 个参数指定起点圆形中心的 x 坐标、y 坐标和半径，后 3 个参数指定终点圆形中心的 x 坐标和半径。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  let gradient = context.createRadialGradient(100, 100, 20, 100, 100, 80);
  gradient.addColorStop(0, "white");
  gradient.addColorStop(1, "black");

  context.fillStyle = gradient;
  context.fillRect(0, 0, 200, 200);
}
```

上面这个渐变，简单理解就是内层圆为半径为 20 像素的`纯白圆`，外层圆为 80 像素的`白渐变黑圆`，剩余部分就是黑色。

![image-20220522114630535](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292323582.png)



## 图案

> 图案适用于填充和描画图形的重复图像。
> 通过`createPattern`方法，该方法接收两个参数，第一个参数是`img`元素，第二个参数是是否重复，和`background-repeat`属性一样。

然后，像渐变一样，把`pattern`对象赋值给`fillStyle`属性即可。

这个图案实际上就有点背景图像的味道了，通过创建`pattern`对象，来控制图像的重复。然后，给绘图上下文的`fillStyle`赋值，设置填充样式，最后再通过`fillRect`来设置图案的位置和大小。

```js
const mycanvas = document.getElementById("mycanvas");

// 确保浏览器支持canvas
if (mycanvas.getContext) {
  const context = mycanvas.getContext("2d");

  const image = document.images[0];

  const pattern = context.createPattern(image, "repeat");
  // const pattern = context.createPattern(image, 'repeat-y')
  // const pattern = context.createPattern(image, 'no-repeat')

  context.fillStyle = pattern;
  context.fillRect(0, 0, 190, 190);
}
```

![image-20220522115145807](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202205292322852.png)
