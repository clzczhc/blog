---
title: 利用H5实现自动选择二倍图、三倍图
categories: 前端
date: 2023-10-09 22:16:33
tags:
  - 优化
  - HTML5
---

# 利用 H5 实现自动选择二倍图、三倍图

## 前言

> 为了更明显地看出效果，三种情况下的图片用不一样的图片来搞。
>
> ![](https://www.clzczh.top/CLZ_img/images/202310092231483.png)

## picture + -webkit-min-device-pixel-ratio

用`picture`标签来提供`source`和`img`。并在`source`标签的`media`属性中设置条件，类似于媒体查询。`-webkit-device-pixel-ratio`表示设备像素比（设备像素 / CSS 像素）。

> 一倍图：当这个比率为 1:1 时，使用 1 个设备像素显示 1 个 CSS 像素。
> 二倍图：当这个比率为 2:1 时，使用 4 个设备像素显示 1 个 CSS 像素，
> 三倍图：当这个比率为 3:1 时，使用 9 个设备像素显示 1 个 CSS 像素

```html
<picture>
  <source srcset="./imgs/3x.jpg" media="(-webkit-min-device-pixel-ratio: 3)" />
  <source srcset="./imgs/2x.jpg" media="(-webkit-min-device-pixel-ratio: 2)" />
  <img src="./imgs/1x.jpg" alt="1x" />
</picture>
```

> 在公司的效果更明显，普通显示屏一倍图，mac 二倍图，苹果手机三倍图。写这个博客，只能用 Chrome 来模拟出来。

window（一倍图）

![](https://www.clzczh.top/CLZ_img/images/202310092241000.png)

iphone 6/7/8（二倍图）

![](https://www.clzczh.top/CLZ_img/images/202310092245773.png)

iphone 12 pro（三倍图）

![](https://www.clzczh.top/CLZ_img/images/202310092246288.png)

## picture + resolution

> -webkit-min-device-pixel-ratio 是非标准的。mdn 也表示它是`resolution`的替代方案。

`resolution`表示设备像素密度的值，当它的单位是`dppx`时，效果和`dpr`一样。（`-webkit-min-device-pixel-ratio`在语法中不允许单位，但是它的隐式单位是`dppx`）

```html
<picture>
  <source srcset="./imgs/3x.jpg" media="(min-resolution: 3dppx)" />
  <source srcset="./imgs/2x.jpg" media="(min-resolution: 2dppx)" />
  <img src="./imgs/1x.jpg" alt="1x" />
</picture>
```

效果和上面一样

## img srcset

> 最后的大招：`img`本身就可以支持自动选择三倍图、二倍图

`srcset`：一个包含单个或多个以逗号分隔的图像候选列表，表示在`img`里可以展示哪些图片。每个图像候选字符串由两部分组成，它们间以`,`分隔。第一部分是图像 url，第二部分是条件描述符，表示什么环境下使用该图像。（类似于上面的`media`）

而要实现自动选择三倍图、二倍图，条件描述符就可以使用像素密度描述符，它指定了在什么样的显示器像素密度下应用相应的图像资源。**简单来说就是跟 dpr 差不多**。它由一个非零浮点值和小写字母`x`组成，如`2x`或`2.0x`。

```html
<img
  srcset="./imgs/3x.jpg 3x, ./imgs/2x.jpg 2x, ./imgs/1x.jpg 1x"
  src="images.jpg"
  alt="Image"
/>
```

效果基本和上面一样。

> 有一点点不一样的是，`picture`在切换设备时，图片也是会变的。如`window`变成`iphone 12 pro`。但是，`img`的`srcset`则需要刷新才能看到效果。

### background 版本

虽然 background 的样式可以通过添加媒体查询来实线，但是也可以用`image-set`来实现，效果跟`img`的`srcset`类似。($\color{red}{Chrome版本113及以上才支持}$)

> Chrome 太久没更新，就体验一波`invalid value`（版本 110）

```css
.box {
  width: 100px;
  height: 100px;
  background-image: image-set(
    url("./imgs/1x.jpg") 1x,
    url("./imgs/2x.jpg") 2x,
    url("./imgs/3x.jpg") 3x
  );
  background-repeat: no-repeat;
  background-size: cover;
}
```
