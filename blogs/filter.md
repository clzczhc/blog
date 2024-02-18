---
title: 从微信聊天框开始学习CSS属性filter
categories: 前端
date: 2022-06-18 12:54:52
tags:
  - CSS
---

# 从微信聊天框开始学习CSS属性filter

## 前言

给别人发图片时，`Ctrl+A`选中图片发生了颜色反转。

下面重现一下

![filter](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181254212.gif)



至于为什么会联想到`filter`属性，主要是因为小时候经常玩手机的拍照功能，黑白滤镜、复古。。。

所以第一印象就是搜索CSS的滤镜属性，就找到了，所以来简单学习一下。(微信的那个具体怎么实现并不了解)



说是学习，但是其实就只是了解一下怎么使用而已。使用`filter`属性主要用法就是通过`Filter`函数来实现具体效果。



## invert()

刚开始就先从实现遇到的反转先。`invert()`函数反转输入图像，参数是转换的比例，值为`0%`表示无变化，值为`100%`表示完全反转。

![filter](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206112214442.gif)



超出`100%`之后也是和`100%`一样的效果。也就是说需要反转只需要设置CSS属性`filter`为`invert(100%)`即可，当然也不一定需要是`100%`。上面测试的是图像，但是实际上非图像该属性也是起作用的。

```css
div {
    width: 200px;
    height: 200px;
    background-color: red;
    filter: invert(100%)
}
```

![image-20220611222043656](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181254224.png)



上面的反转是不是很有意思。这是因为红色的rgb值为`(255, 0, 0)`，所以反转后的rgb值为`(0, 255, 255)`，即上面的效果。



### 实用技巧

我们可以给html元素添加`filter: invert(100%)`，即可实现切换亮暗模式。

```js
document.documentElement.style.filter='invert(100%)'
```



![filter](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206112232412.gif)

可以使用该方法开启黑暗模式看pdf文件的(虽然有一些地方会有点怪)



## blur()

调整输入图像的模糊程度，参数可以设置为CSS长度(`px`、`em`等，不接受百分比)

![image-20220612124521327](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206121245538.png)



### 使用技巧

`filter`属性的`blur()`可以将模糊应用于元素。说到模糊，可能想到的应用就是自己制作一下有毛玻璃效果的背景图片了。接下来来耍一下。(在网上看到的效果，下面的例子也是参考网上的)

基本解构：

css

```css
body {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background: url('https://i.loli.net/2019/11/17/GAYyzeKsiWjP5qO.jpg') no-repeat;
    background-size: cover;
}

.grass {
    position: fixed;
    top: 50px;
    width: 72vw;
    height: 36vh;
    box-shadow: 0 0.3px 0.7px rgb(0 0 0 / 13%), 0 0.9px 1.7px rgb(0 0 0 / 18%), 0 1.8px 3.5px rgb(0 0 0 / 22%), 0 3.7px 7.3px rgb(0 0 0 / 28%), 0 10px 20px rgb(0 0 0 / 40%);
}
```



html

```html
<body>
    <div class="grass"></div>
</body>
```

![image-20220612132534829](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181254567.png)



现在给`grass`盒子添加一下模糊度。

```css
filter: blur(4px);
```

![image-20220612132921567](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206121329390.png)

但是结果和我们想象的不太一样，只有阴影有模糊。这是因为`filter`是将模糊等图形效果应用于元素，而后面的背景图片是该元素后面的`body`元素的，所以添加的模糊并不会添加到后面的背景图片中。



这时候，就轮到`filter`的好兄弟`backdrop-filter`登场了，它可以让你为一个元素后面区域添加图形效果（如模糊或颜色偏移）。值和`filter`的一样用法。

```css
backdrop-filter: blur(4px);
```

![image-20220612135449495](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206121354290.png)



## drop-shadow()

对输入图像应用阴影效果。(和`box-shadow`很相似，不过，**在部分浏览器中通过`filter`可以提供硬件加速**)

* `offset-x`：设置阴影的水平偏移量
* `offset-y`：设置阴影的垂直偏移量

* `blur-radius`：设置阴影的模糊半径，值越大，越模糊，阴影也会更大、更淡
* `color`：颜色



```css
div {
    width: 200px;
    height: 200px;
    background-color: pink;
    filter: drop-shadow(4px 4px 6px black);
}
```

![image-20220612151813219](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3138398e857f479ab5d807ee4564c5dc~tplv-k3u1fbpfcp-zoom-1.image)



## 复合函数

`Filter`函数可以任意组合来控制渲染。

```css
div {
    width: 200px;
    height: 200px;
    background-color: red;
    filter: invert(100%) drop-shadow(4px 4px 6px black);
}
```

![image-20220612152120004](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17a75ac95979471fa383b1d4bb1deaf3~tplv-k3u1fbpfcp-zoom-1.image)



`filter`属性还有很多很有意思的用法，可以设置对比度、灰度等。这里就不再过多赘述了，有想了解的可以到官方文档查阅。
