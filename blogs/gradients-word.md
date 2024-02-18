---
title: CSS实现渐变字
date: 2022-03-13 12:04:06
categories: 前端
tags:
  - CSS
---

# CSS 实现渐变字

![image-20220309200715875](https://s2.loli.net/2022/03/13/AmGSj5UWvXZO6gw.png)

先来下前置知识。如果想速通，也可指直接到[渐变字实现](#jump)

## 什么是渐变

> CSS3 渐变（gradients）可以让你在两个或多个指定的颜色之间显示平稳的过渡。
>
> 以前，你必须使用图像来实现这些效果。但是，通过使用 CSS3 渐变（gradients），你可以减少下载的时间和宽带的使用。此外，渐变效果的元素在放大时看起来效果更好，因为渐变（gradient）是由浏览器生成的。

## 渐变类型

渐变主要有三种类型：线性渐变(` linear-gradient`)、径向渐变(` radial-gradient`)、圆锥渐变(` conic-gradient`)

### 线性渐变

线性渐变创建了一条沿直线前进的颜色带。

```css
background-image: linear-gradient(direction, color-start, ..., color-end);
```

- 第一个参数为**渐变方向**
- 第二个参数为**渐变起点**
- 第三个参数为**渐变终点**

<br>

#### 基础线性渐变

使用` linear-gradient`函数，至少指定两种颜色即可(也被称为色标)

```css
.box1,
.box2 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 20px;
}

.box1 {
  background: linear-gradient(red, blue);
}

.box2 {
  background: linear-gradient(red, blue, purple);
}
```

![image-20220309000309356](https://s2.loli.net/2022/03/13/R8zcYlsv2UZXOED.png)

#### 改变渐变方向

线性渐变的方向默认是从上到下，可以通过关键字` to`改变渐变方向。

```css
.box1,
.box2 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 20px;
}

.box1 {
  background: linear-gradient(to right, red, blue, purple);
}

.box2 {
  background: linear-gradient(to bottom right, red, blue, purple);
}
```

![image-20220309000853385](https://s2.loli.net/2022/03/13/xY7GDQJC35ef24R.png)

#### 设置渐变角度

上面说了，可以通过关键字` to`来改变角度，但是可选方向有较大限制。此时可以给渐变设置一个具体的角度。

![image-20220309154501119](https://s2.loli.net/2022/03/13/CSpdiucQHfUPzsO.png)

**此图来自菜鸟教程**

```css
.box1,
.box2,
.box3,
.box4 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  background: linear-gradient(0deg, red, blue, purple);
}

.box2 {
  background: linear-gradient(60deg, red, blue, purple);
}

.box3 {
  background: linear-gradient(180deg, red, blue, purple);
}

.box4 {
  background: linear-gradient(-60deg, red, blue, purple);
}
```

![image-20220309001836218](https://s2.loli.net/2022/03/13/JQvosu2C3YmVD5z.png)

#### 颜色终止位置

可以给颜色设置像素值或百分比等其他数值来调整位置。没有明确设置的话，会自动计算

```css
.box1,
.box2,
.box3 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  background: linear-gradient(90deg, red 70%, blue);
}

.box2 {
  background: linear-gradient(90deg, red 30%, blue 70%);
}

.box3 {
  background: linear-gradient(180deg, red 200px, blue);
}
```

![image-20220309083036610](https://s2.loli.net/2022/03/13/ol751QfJuVFE4pR.png)

#### 创建实线

根据颜色终止位置的知识点，很容易就能知道可以通过设置相邻的颜色的终止位置设置为相同即可

```css
.box1,
.box2 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  background: linear-gradient(to right, red 50%, blue 50%);
}

.box2 {
  background: linear-gradient(135deg, red 33%, blue 33%, blue 66%, purple 66%);
}
```

![image-20220309084146642](https://s2.loli.net/2022/03/13/vponawje52IUPA6.png)

#### 设置渐变中心点

默认情况下，渐变会平滑地从一种颜色过渡到另一种颜色。但是可以设置一个值修改渐变的中心点。

```css
.box1,
.box2,
.box3 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  background: linear-gradient(to right, red, blue);
}

.box2 {
  background: linear-gradient(to right, red, 50%, blue);
}

.box3 {
  background: linear-gradient(to right, red, 80%, blue);
}
```

![image-20220309084757700](https://s2.loli.net/2022/03/09/RfwzUOsLdcDpEnl.png)

#### 创建色带和条纹

要创建一个颜色的区域的话，一个颜色需要两个位置，这样子，这个颜色在两个颜色起止点都将会是完全饱和(即会保持该饱和度)。而和相邻的不同颜色还是正常的过渡。

```css
.box1,
.box2 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  background: linear-gradient(
    to right,
    red 30%,
    blue 35%,
    blue 65%,
    purple 70%
  );

  /* 简洁写法 */
  /* background: linear-gradient(to right, red 30%, blue 35% 65%, purple 70%);   */
}

.box2 {
  background: linear-gradient(to right, red 33%, blue 33% 66%, purple 66%);
}
```

![image-20220309090648171](https://s2.loli.net/2022/03/13/EMTkJjYA6B38uc9.png)

#### 堆叠背景、渐变

渐变支持透明度，因此可以堆叠多个背景。背景从上到下堆叠，第一个指定在顶部

```css
.box1,
.box2 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  width: 800px;
  height: 480px;
  background: linear-gradient(to left, transparent 50%, red),
    url("https://s2.loli.net/2022/03/09/8OjEFf5GQy6iTcm.png") no-repeat;
  background-size: 800px;
}

.box2 {
  width: 800px;
  height: 480px;
  background: url("https://s2.loli.net/2022/03/09/8OjEFf5GQy6iTcm.png")
      no-repeat, linear-gradient(to left, transparent 50%, red);
  background-size: 800px;
}
```

![image-20220309114848001](https://s2.loli.net/2022/03/09/B3ObCwQDSXgoKVG.png)

<br>

同理：渐变也是可以和其他渐变叠加的。

```css
.box1 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  width: 800px;
  height: 480px;
  background: linear-gradient(to left, transparent, red), linear-gradient(to top, transparent, blue);
}
```

![image-20220309122310300](https://s2.loli.net/2022/03/09/m94dTQjl7fZE8au.png)

### 径向渐变

径向渐变类似于线性渐变，只是它们从中心点向外辐射。可以指定该中心点在哪里。也可以把它们做成圆形或椭圆形。

语法：

`radial-gradient(center, shape size, start-color, ..., last-color);`

- 第一个参数为**渐变起点**
- 第二个参数为**渐变形状**和**渐变大小**
- 第三个参数为**渐变起点色标**
- 第四个参数为**渐变终点色标**

<br>

#### 基础径向渐变

```css
.box1,
.box2 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  background: radial-gradient(red, blue);
}

.box2 {
  background: radial-gradient(red, blue, purple);
}
```

![image-20220309123443478](https://s2.loli.net/2022/03/09/4Lg2QEa5jIsRM9J.png)

#### 颜色终止位置

和线性渐变一样

```css
.box1 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  background: radial-gradient(red 10px, yellow 30%, #1e90ff 50%);
}
```

![image-20220309131245750](https://s2.loli.net/2022/03/09/tyrkoNGT74EevBm.png)

#### 设置渐变中心

通过关键字`at`实现: 第一个参数是横轴，最左是 0%, 最右是 100%. 第二个参数是纵轴.

```css
.box1,
.box2 {
  display: inline-block;
  width: 400px;
  height: 400px;
  margin: 60px;
}

.box1 {
  /* 关键字at: 第一个参数是横轴，最左是0%, 最右是100%. 第二个参数是纵轴. */
  background: radial-gradient(at 0% 100%, red 10px, yellow 30%, #1e90ff 50%);
}

.box2 {
  background: radial-gradient(
    at 100px 200px,
    red 10px,
    yellow 30%,
    #1e90ff 50%
  );
}
```

![image-20220309131916527](https://s2.loli.net/2022/03/09/dFDMOVX4xnE8tGK.png)

#### 设置形状

shape 参数定义了形状。可以是值 circle 或 ellipse。其中，circle 表示圆形，ellipse 表示椭圆形。默认值是 ellipse

```css
.box1,
.box2,
.box3 {
  display: inline-block;
  width: 800px;
  height: 400px;
  margin: 60px;
}

.box1 {
  background: radial-gradient(red 10px, yellow 30%, #1e90ff 50%);
}

.box2 {
  background: radial-gradient(circle, red 10px, yellow 30%, #1e90ff 50%);
}

.box3 {
  background: radial-gradient(ellipse, red 10px, yellow 30%, #1e90ff 50%);
}
```

![image-20220309154415832](https://s2.loli.net/2022/03/09/nV4MxdHZIANbzD7.png)

<b style="color: red">如果盒子是正方形，那么设置形状为椭圆可能不起效</b>

#### 设置渐变大小

size 参数定义了渐变的大小。它可以是以下四个值：

- `closest-side`，指定径向渐变的半径长度为**从圆心到离圆心最近的边**
- `farthest-side`，指定径向渐变的半径长度为**从圆心到离圆心最远的边**
- `closest-corner`，指定径向渐变的半径长度为**从圆心到离圆心最近的角**
- `farthest-corner`，指定径向渐变的半径长度为**从圆心到离圆心最远的角**

默认值为`farthest-corner`

```css
.box1,
.box2,
.box3,
.box4 {
  display: inline-block;
  width: 800px;
  height: 800px;
  margin: 60px;
}

.box1 {
  background: radial-gradient(closest-side, red 10px, yellow 30%, #1e90ff 50%);
}

.box2 {
  background: radial-gradient(farthest-side, red 10px, yellow 30%, #1e90ff 50%);
}

.box3 {
  background: radial-gradient(
    closest-corner,
    red 10px,
    yellow 30%,
    #1e90ff 50%
  );
}

.box4 {
  background: radial-gradient(
    farthest-corner,
    red 10px,
    yellow 30%,
    #1e90ff 50%
  );
}
```

![image-20220309171611188](https://s2.loli.net/2022/03/09/UGvjcOD1HkRQySW.png)

#### 堆叠径向渐变

和线性渐变一样

```css
.box1 {
  display: inline-block;
  width: 800px;
  height: 800px;
  margin: 60px;
}

.box1 {
  border-radius: 800px;
  background: radial-gradient(
      at 50% 0,
      rgba(255, 0, 0, 0.5),
      rgba(255, 0, 0, 0)
    ), radial-gradient(at 25% 75%, rgba(0, 0, 255, 0.5), rgba(0, 0, 255, 0)),
    radial-gradient(at 75% 75%, rgba(0, 255, 0, 0.5), rgba(0, 255, 0, 0));
}
```

![image-20220309174059818](https://s2.loli.net/2022/03/09/Ol4kNXsi9pQMyYh.png)

### 重复渐变

> `linear-gradient`和`radial-gradient`属性不支持自动重复色标。但是，`repeating-linear-gradient` 和`repeating-radial-gradient`属性可用于提供此功能。

#### 重复线性渐变

```css
.box1 {
  display: inline-block;
  width: 800px;
  height: 800px;
  margin: 60px;
}

.box1 {
  background: repeating-linear-gradient(45deg, red 0 20px, blue 20px 40px);
}
```

![image-20220309175418709](https://s2.loli.net/2022/03/09/E98RIZTLbDrVWgu.png)

看多难受，勿贪看。

#### 重复径向渐变

```css
.box1 {
  display: inline-block;
  width: 800px;
  height: 800px;
  margin: 60px;
}

.box1 {
  border-radius: 800px;
  background: repeating-radial-gradient(red 0 20px, blue 20px 40px);
}
```

![image-20220309175910740](https://s2.loli.net/2022/03/09/6womYBMjpVqb591.png)

## background-clip 属性

`background-clip` 设置元素的背景（背景图片或颜色）是否延伸到边框、内边距盒子、内容盒子下面。

- border-box：背景延伸至边框外沿（但是在边框下层）。

- padding-box：背景延伸至内边距（[`padding`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding)）外沿。不会绘制到边框处。

- content-box：背景被裁剪至内容区（content box）外沿。

- text：背景被裁剪成文字的前景色。（即文字的背景即为区块的背景，文字之外的区域都将被裁剪掉）

<br>

```css
body {
  margin: 40px 0;
}

.box1,
.box2,
.box3,
.box4 {
  display: inline-block;
  width: 400px;
  height: 400px;
  line-height: 400px;
  padding: 20px;
  border: 20px dashed blue;
  margin: 20px;
  background-color: pink;
  font-size: 80px;
  text-align: center;
}

.box1 {
  background-clip: padding-box;
}

.box2 {
  background-clip: border-box;
}

.box3 {
  background-clip: content-box;
}

.box4 {
  -webkit-background-clip: text;
}
```

![image-20220309193146881](https://s2.loli.net/2022/03/09/oqR4vUajV9PhWdz.png)

<b style="color: red">一眼望去，最后一个最特殊</b>，所以要加上前缀` -webkit`，好吧，原因并不是这样。网上有种说法是 background-clip: text; 只兼容 chrome,要想兼容其他浏览器就要用: `-webkit-background-clip: text;`。然而，我的 chrome 浏览器都需要`-webkit-background-clip: text;`才能实现。<b style="color: red">另外，文字的颜色应该设置为透明，否则会覆盖掉背景色。</b>

## <span id="jump">渐变字实现</span>

看到这里，基本就能实现渐变字啦。

代码如下。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>渐变字实现</title>
    <style>
      div {
        background: linear-gradient(
          to right,
          rgb(255, 0, 0) 0%,
          rgba(0, 0, 255, 0.8) 50%,
          rgb(128, 0, 128) 80%
        );
        -webkit-background-clip: text;
        color: transparent;
        font-size: 400px;
        text-align: center;
      }
    </style>
  </head>

  <body>
    <div>赤蓝紫</div>
  </body>
</html>
```

![image-20220309200709951](https://s2.loli.net/2022/03/13/BL9Zesh6qIzRw2g.png)

<br />

参考链接：

[使用 CSS 渐变 - CSS（层叠样式表） | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Images/Using_CSS_gradients)

[CSS3 渐变 | 菜鸟教程 ](https://www.runoob.com/css3/css3-gradients.html)

[CSS3 新特性概述\_阿锐丫的博客-CSDN 博客\_css3 新增特性](https://blog.csdn.net/weixin_45337959/article/details/123004306)
