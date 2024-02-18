---
title: CSS隐藏元素的几种方式
categories: 前端
date: 2022-04-30 17:25:22
tags:
  - JavaScript
---

# CSS隐藏元素的几种方式

## 前言

开始之前，先来了解一下回流和重绘的概念。(经小伙伴评论提醒，后来加的内容)

回流：当我们修改元素的几何位置属性，如宽度、高度时，浏览器会重新布局，这个过程就叫回流

重绘：当我们修改元素的绘制属性，如背景色、颜色等，浏览器不会重新布局，但是需要重新进入绘制阶段，这个过程就是重绘。

可以通过**css triggers**网站查询元素是否会导致回流、重绘。

**回流一定会触发重绘，重绘不一定会触发回流**

## display: none

最常见的隐藏元素的方法，不会渲染该元素，所以该元素不会占位置，也不会响应绑定的事件。

html

```
<div></div>
<div onclick="alert('赤蓝紫')"></div>
<div></div>
```

css

```
body {
    display: flex;
}

div {
    width: 100px;
    height: 100px;
}

div:nth-child(1) {
    background-color: red;
}

div:nth-child(2) {
    display: none;
    background-color: blue;
}

div:nth-child(3) {
    background-color: purple;
}
```

![image-20220427230639837](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b3c393ff79c420cba57e8f9f1452328~tplv-k3u1fbpfcp-zoom-1.image)

那么`display`会不会引发回流、重绘呢？

答案是必然的，当我们修改`display`时，它会突然地出现或消失(即会修改元素的位置)，所以会引发回流，引发回流自然就会引发重绘。但是，如果一开始就是`none`，那么倒是不会多次触发回流，因为不会渲染该元素。

## visibility: hidden

元素在页面中会保留位置，但是不会响应绑定的事件

```
div:nth-child(2) {
    visibility: hidden;
    background-color: blue;
}
```

![image-20220427230927696](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60b9392120b7410b8fd4c39a9f2e53be~tplv-k3u1fbpfcp-zoom-1.image)

元素会在页面中保留位置，并没有几何位置属性的变化，所以并不会触发回流，会重绘。

## opacity: 0

将元素的透明度设置为0。所以元素在页面中会保留位置，且也**能响应元素绑定的监听事件**。

```
div:nth-child(2) {
    opacity: 0;
    background-color: blue;
}
```

![css](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b0729ec8cb147a992af72de2583a6dc~tplv-k3u1fbpfcp-zoom-1.image)

元素会在页面中保留位置，并没有几何位置属性的变化，所以并不会触发回流，会重绘。

## 定位法

### 绝对定位法

因为绝对定位可以让元素脱离标准流，所以只需要设置绝对定位，就可以让元素移出可视范围内，这样子就相当于隐藏了。又因为是移出可以范围，所以监听事件无效；元素脱离了标准流，所以不会保留位置。

```
div:nth-child(2) {
    position: absolute;
    top: -200px;
    background-color: blue;
}
```

![image-20220428212006431](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb7c2123696845e3b72d87d79a6b442a~tplv-k3u1fbpfcp-zoom-1.image)

绝对定位会会让元素的位置发生变化，所以会触发回流，但是让元素绝对定位后，再进行其他会导致回流的操作，就能减少回流代价。

### 相对定位法

相对定位法和绝对定位法类似，都是让元素移出可是范围内。不同的是，相对定位不会脱离标准流，所以会保留位置。

```
div:nth-child(2) {
    position: relative;
    top: -200px;
    background-color: blue;
}
```

![image-20220428213632477](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d945a13a16a44bddb11ac6d21bd64688~tplv-k3u1fbpfcp-zoom-1.image)

与绝对定位同理，也会触发回流

## 缩放法

通过`scale`将元素缩放为0，元素保留位置，监听事件无效

```
div:nth-child(2) {
    transform: scale(0, 0);
    background-color: blue;
}
```

![image-20220428213940493](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d8f56f4021f49c4810ac162c4003f60~tplv-k3u1fbpfcp-zoom-1.image)

`transform`属性不会触发回流、重绘。

简单地说下为什么`transform`属性为什么不会触发回流、重绘。

CSS的最终表现可以分为4步：计算样式 -> 排布 -> 绘制 -> 组合层(**Recalculate Style** -> **Layout** -> **Paint Setup and Paint** -> **Composite Layers**)

而`transform`是位于最后的组合层，所以不会触发回流重绘。而且一些浏览器也会针对`transform`开启GPU加速。

顺便提一嘴：只是查询属性也会强制发生回流。比如`width`位置`Layout`层，所以只是通过js访问它也会导致回流。

## 降低层次法

通过`z-index`来降低当前元素的层次，让其他元素遮盖该元素来实现隐藏。

```
div:nth-child(2) {
    position: absolute;
    z-index: -999;
    background-color: blue;
}
```

![image-20220428214355252](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/213df0c462d44386bd4c9f88494ccba2~tplv-k3u1fbpfcp-zoom-1.image)

上面我们用到了绝对定位，可能就会提出疑问：这不是算是绝对定位法吗？

但是上面的只是其中一种用法，也能通过搭配`margin`来实现隐藏，只要让降低层次的元素被更高层次的元素遮住就行。

```
div:nth-child(2) {
    margin-left: -100px;
    z-index: -999;
    background-color: blue;
}
```

`z-index`不会导致回流，但是会导致重绘，因为`z-index`只是降低层级，并不会导致几何位置的变化。

但是，如果像上面那样搭配`position`、`margin`使用，则会导致回流。

## clip-path法

> `clip-path`：使用裁剪方式创建元素的可显示区域。区域内的部分显示，区域外的隐藏。

只需要把元素的可显示区域裁剪为0即可，会保留位置

```
div:nth-child(2) {
    clip-path: circle(0);
    background-color: blue;
}
```

![image-20220428220242817](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe66c61e1b374c319b93c206edc9aafc~tplv-k3u1fbpfcp-zoom-1.image)

`clip-path`不会导致回流，但是会导致重绘

\
