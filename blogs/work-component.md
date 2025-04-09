---
title: 工作中的那点事之 组件
date: 2025-04-09 21:00:05
categories: "前端"
tags:
  - 工作中的那点事
  - 组件
---

# 工作中的那点事之 组件

> 记录一下工作中实现的基础组件的思路。

## 颜色选择器

### 实现原理

利用 hsv 色系，将颜色拆分成 hue(色相)、Saturation(饱和度)、Value(亮度)三部分。

> 也可叫`hsb`色系。Bright(明度)

其中 `Saturation` 组件负责`saturation`、`value`的变化，`Hue`组件负责 hue 的变化。

![](https://www.clzczh.top/CLZ_img/images/202504092106306.png)
上图中，红色是`Saturation`组件，蓝色则是`Hue`组件。

> Hue 的背景色可以直接使用下面的。算是一个标准。`react-color`、`ant-design`都是

```css
background: linear-gradient(
  to right,
  rgb(255, 0, 0) 0%,
  rgb(255, 255, 0) 17%,
  rgb(0, 255, 0) 33%,
  rgb(0, 255, 255) 50%,
  rgb(0, 0, 255) 67%,
  rgb(255, 0, 255) 83%,
  rgb(255, 0, 0) 100%
);
```

**计算公式**：
`Saturation`：

```js
const { width, height, left, top } = container.target.getBoundingClientRect();
let x = mouseX - left;
let y = mouseY - top;

const saturation = x / width;
const value = 1 - y / height;
```

`Hue`：

```js
const hue = (x / width) * 360;
```

**公式简解**：
色相的取值在 0-360 之间。所以计算出比例后需要乘以 360。而饱和度和亮度的取值在 0-1 之间，可以不做处理。

亮度需要用`1 - y / height`，是因为`mouseY - top`得到的结果是鼠标点距离顶部的距离，而亮度的值应该是鼠标点到底部的距离，所占的比例。

### 数据结构

> 这里颜色的值采用的是`number`类型，需要转成 16 进制才能获取到`hex`字符串。下图中的`value`就是对应的`number`类型

![](https://www.clzczh.top/CLZ_img/images/202504092134287.png)

`Saturation`、`Hue`都维护自己的一套数据结构。
初始值由父组件`ColorPicker`的`value`控制。

因为`Saturation`控制饱和度、亮度，`Hue`控制色相。所以它们的改动会互相影响，所以暴露方法，修改自身的状态。比如`Hue`控制的色相发生变化了，需要让`Saturation`修改它的色相值。

> 每个组件都维护自己的一套数据结构，通过暴露方法来修改自身状态这种方式，不会出现色系转换后的精度丢失问题。导致的问题就是会有抖动现象。因为`rgb`转换成`hsv`色系后，可能是无限不循环小数，再转换成`rgb`，值会有偏差。

颜色转换算法可以参考：
https://gist.github.com/mjackson/5311256

### 有问题版本

![](https://www.clzczh.top/CLZ_img/images/202504092149501.png)

这个版本的数据结构可以比较容易看出色系转换的问题。

> 这个版本的数据结构由`ColorPicker`控制，其他组件都是基于`value`进行一些转换。会出现问题：色相变化时，`Hue`需要把`hsv`转换成`value`，让父组件修改`value`。同时，父组件修改后的值又反过来修改`Hue`的值，就会出现抖动问题。

## 日期选择器（日历）

### 核心

```js
// 获取该月的第一天是星期几
const firstDayOfMonth = new Date(year, month, 1).getDay();

// 获取该月的总天数
const totalDays = new Date(year, month + 1, 0).getDate();
```

**注意**：

1. 上面的`month`是从 0 开始计数的，0-11。
2. `getDay`返回值中，0 代表星期天，1 代表星期一，依此类推
3. `getDate`可以返回一个指定的日期对象为一个月中的哪一日（从 1--31）。`new Date()`的第三个参数为 0 时，可以获取上个月的总天数。

通过上面这两个主要的方法，再拓展一下，就可以实现日期选择器了，包括支持选中范围等功能。

[整天用 Calendar 日历组件，不如自己手写一个吧！](https://juejin.cn/post/7257441765976506425)
