---
title: Grid布局 项目属性
categories: 前端
date: 2022-07-05 20:41:28
tags:
    - CSS
    - 布局
---

# Grid布局 项目属性

## 容器的基础代码

HTML

```html
<div class="box">
    <div>1</div>
    <div>2</div>
    <div>3</div>
    <div>4</div>
    <div>5</div>
    <div>6</div>
</div>
```

CSS

```css
.box {
    display: grid;
    grid-template-rows: repeat(3, 100px);
    grid-template-columns: repeat(3, 100px);
}

.box div:nth-child(odd) {
    background-color: pink;
}

.box div:nth-child(even) {
    background-color: purple;
}

.box div {
    border: 1px solid red;
}
```

## 项目属性

### `grid-row`系列属性

一共有三个：  

* `grid-row-start`属性：上边框所在的水平网格线  

* `grid-row-end`属性：下边框所在的水平网格线  

* `grid-row`属性：`grid-row-start`和`grid-row-end`的简写形式。  
  
  ```css
  grid-row: <start-line> / <end-line>;
  ```

说的有点玄玄的，实际体验更清晰。

```css
.box div:nth-child(1) {
    grid-row-start: 1;
    grid-row-end: 3;
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ef2e45703e64241b2817fd8404899b5~tplv-k3u1fbpfcp-zoom-1.image)
上面意思就是第一根水平网格线到第三根网格线的部分都是该项目的。其实也可以用数学的取值区间来解释：[1, 3)取第一行到第三行的部分，包含第一行，但不包含第三行。

上面的代码也可以使用`grid-row`属性来实现。

```css
grid-row: 1 / 3;
```

**属性值还可以使用`span`关键字，表示跨越多少个网格**

如`grid-row: 1 / span 3;`

### `grid-column`系列属性

一共有三个：  

* `grid-column-start`属性：左边框所在的垂直网格线  
* `grid-column-end`属性：右边框所在的垂直网格线  
* `grid-column`属性：`grid-column-start`和`grid-column-end`的简写形式。  

和`grid-row`系列基本一样，只是换一下方向而已。

```css
grid-row: 1 / span 2;
grid-column: 1/ span 2;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6540ae84314b4b47ab1e128da081361d~tplv-k3u1fbpfcp-zoom-1.image)

### 排列属性

项目的排列属性有三个：  

* `justify-self`: 设置单元格内容的水平位置，跟`justify-items`属性用法一样，只作用于单个项目  
* `align-self`: 设置单元格内容的垂直位置，跟`align-items`属性用法一样，只作用于单个项目  
* `place-self`: `justify-self`和`align-self`的简写形式，跟`place-items`属性用法一样，只作用于单个项目  

```css
.box div:nth-child(1) {
    justify-self: center;
    align-self: center;

    /* 或 */
    /* place-self: center; */
}
```

# `grid-area`属性

之前讲解容器属性时，已经使用过`grid-template-areas`和`grid-area`来划分区域了。

```css
.box {
    display: grid;
    width: 300px;
    height: 300px;
    grid-template-areas:
        "header header header"
        "nav main main";
}

.box div:nth-child(1) {
    grid-area: header;
    background-color: skyblue;
}

.box div:nth-child(2) {
    grid-area: nav;
    background-color: purple;
}

.box div:nth-child(3) {
    grid-area: main;
    background-color: pink;
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5acd6eb0144d46dd8046f2a7c72cdb99~tplv-k3u1fbpfcp-zoom-1.image)

实际上，`grid-area`是`grid-row-start`、`grid-column-start`、`grid-row-end` 和 `grid-column-end` 的简写。

语法：

```css
grid-area: <row-start> / <column-start> / <row-end> / <column-end>
```

上面例子中`grid-area`其实也是可以拆分的。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d6253a7c0c44084bfa18290296b377c~tplv-k3u1fbpfcp-zoom-1.image)

之前也有讲过，划分区域划分有两大原则：

* 不能当墙头草(跨行的同时跨列)
* 不能太贪心(同时拿两份不紧贴的)

所以最后划分的区域都一块，而且不能折。

所以其实`grid-area: header;`包含了以下的信息
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a489d160ec040a4aa1c48a7e994f4be~tplv-k3u1fbpfcp-zoom-1.image)

非划分区域用法：

```css
.box div:nth-child(1) {
    grid-area: 1 / 2 / 3 / 4;
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ef7fcc1db6e443d937aa9b364096452~tplv-k3u1fbpfcp-zoom-1.image)
所以项目的范围是：行：[1, 3)，列: [2, 4)，也就是1、2行，2、3列。

参考链接：

* [CSS Grid 网格布局教程](http://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html)
* [MDN](https://developer.mozilla.org/zh-CN/)
