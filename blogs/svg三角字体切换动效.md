---
title: svg三角形字体切换动效
categories: 前端
date: 2024-09-01 12:53:55
tags:
  - 动画
---

# Svg 三角形字体切换动效

## 前言

之前在[http://species-in-pieces.com/](http://species-in-pieces.com/)上就看到过三角形组成的动物，切换动物时的动效比较炫酷。之后又在 codepen 上看到[Triangles - deer or rabbit](https://codepen.io/ainalem/pen/VWJRav)上看到鹿转兔子的动效。所以想到做一个简单的三角形字体转换（字母），比较简单，甚至可以自己画。

> 本文主要讲的可能更多是搜索、制作（调整）资源的过程，这类动效之前也看到过其他博主发文，但是大部分是用别人的坐标。但是个人感觉这种三角形切换动效，本身最重要的就是这种坐标，结果坐标直接拿别人的做好的没太大意义。

## 三角形字体

一开始是打算是打算直接使用[svg-path-editor](https://yqnn.github.io/svg-path-editor/)绘制的，但是实际使用不是很方便。所以后面就想看看有没有能直接把字符按指定个数拆分成三角形的在线工具。

结果是没找到，各种方式问 gpt 也找不到。找的过程中，机缘巧合看到了一篇文章[11+ Free Low Poly Fonts](https://www.hipsthetic.com/11-free-low-poly-fonts/)，看到有这种免费低多边形字体，所以转换思路 => 下载这种字体，然后转成 svg。

最后，在[fontspace](https://www.fontspace.com/)上找到比较符合想法的字体。

![](https://www.clzczh.top/CLZ_img/images/202409011758983.png)

## 字体编辑

在上面找到字体后，但是每个符号的三角形个数并不一样，这样子，过渡动效做起来就不是很方便，需要考虑增加、减少三角形的情况。所以对下载的字体进行调整。
使用[FontStore](https://font.qqe2.com/)对字体进行调整。

![](https://www.clzczh.top/CLZ_img/images/202409011802899.png)

> 上面的字体编辑器操作就比`svg-path-editor`操作方便。不过，选中元素的旋转只能是 90x(90°、180°)的旋转，没办法进行小角度调整就不是很方便就是了。

调整完之后，导出`ttf`字体文件。

## 字体导出 svg

字体导出 svg 工具挺多的，但是试了很多并不是`path`实现的，最后找到一个是`path`实现的工具--[fontforge](https://fontforge.org/en-US/)
![](https://www.clzczh.top/CLZ_img/images/202409011809838.png)

> 导出的时候选择`svg`格式即可。有可能会遇到导出失败的情况（我就遇到了，第一次导出成功，后面都失败了）。[stackflow](https://stackoverflow.com/questions/47902878/fontforge-export-a-glyph-to-svg-with-fontforge-command-line)上有解决方案。

导出的 svg 是只有一个`path`元素的，所以需要手动把路径拆解一下。

## animejs 实现动效

把资源都搞到手之后，接下来就是使用`animejs`实现动效了。
步骤:

1. 给每一个 path 添加唯一标识`id`
2. 构建映射，字符每一个 path 对应另一个字符的哪个 path
3. 使用`anime timeline`把每个 path 的动效都给加上

```html
<svg
  xmlns="http://www.w3.org/2000/svg"
  width="400"
  height="800"
  viewBox="0 0 400 800"
>
  <path
    id="path1"
    fill="currentColor"
    d="M265 164l52 30l53 31v-61v-62l-53 30z"
  />
  <path
    id="path2"
    fill="currentColor"
    d="M370 89l-53 -32l-52 -30v62v61l52 -31z"
  />
  <path
    id="path3"
    fill="currentColor"
    d="M145 89l52 30l53 31v-61v-62l-53 30z"
  />
  <path
    id="path4"
    fill="currentColor"
    d="M25 164l52 30l53 31v-61v-62l-53 30z"
  />
  <path
    id="path5"
    fill="currentColor"
    d="M130 239l-53 -32l-52 -30v62v61l52 -31z"
  />
  <path
    id="path6"
    fill="currentColor"
    d="M25 314l52 30l53 31v-61v-62l-53 30z"
  />
  <path
    id="path7"
    fill="currentColor"
    d="M130 389l-53 -32l-52 -30v62v61l52 -31z"
  />
  <path
    id="path8"
    fill="currentColor"
    d="M25 464l52 30l53 31v-61v-62l-53 30z"
  />
  <path
    id="path9"
    fill="currentColor"
    d="M145 539l52 30l53 31v-61v-62l-53 30z"
  />
  <path
    id="path10"
    fill="currentColor"
    d="M370 539l-53 -32l-52 -30v62v61l52 -31z"
  />
  <path
    id="path11"
    fill="currentColor"
    d="M265 464l52 30l53 31v-61v-62l-53 30z"
  />
</svg>
```

```js
const paths = [
  {
    id: "#path1",
    d: "M25 162l52 30l53 32v-62v-61l-53 30z",
  },
  {
    id: "#path2",
    d: "M130 237l-53 30l-52 32v-62v-61l52 30z",
  },
  {
    id: "#path3",
    d: "M25 312l52 30l53 32v-62v-61l-53 30z",
  },
  {
    id: "#path4",
    d: "M130 387l-53 30l-52 32v-62v-61l52 30z",
  },
  {
    id: "#path5",
    d: "M25 462l52 30l53 32v-62v-61l-53 30z",
  },
  {
    id: "#path6",
    d: "M130 537l-53 30l-52 32v-62v-61l52 30z",
  },
  {
    id: "#path7",
    d: "M25 612l52 30l53 32v-62v-61l-53 30z",
  },
  {
    id: "#path8",
    d: "M145 687l52 -31l53 -30v61v62l-53 -32z",
  },
  {
    id: "#path9",
    d: "M370 687l-53 30l-52 32v-62v-61l52 30z",
  },
  {
    id: "#path10",
    d: "M265 612l52 30l53 32v-62v-61l-53 30z",
  },
  {
    id: "#path11",
    d: "M490 612l-53 -31l-52 -30v61v62l52 -32z",
  },
];

const timeline = anime.timeline({
  autoplay: true,
  direction: "alternate",
  loop: true,
});

paths.forEach(function (path, index) {
  timeline.add(
    {
      targets: path.id,
      d: {
        value: path.d,
        duration: 1000, // 转换过程1s过渡
        easing: "easeInOutQuad",
      },
    },
    1000
  ); // 动画同时开始，一个字符停留1s（如果没设置，那么就会是一个一个三角形依次执行
});
```

效果：
![](https://www.clzczh.top/CLZ_img/images/202409011828664.gif)

> 如果想要有多段效果，比如a字符变成b字符后，再变成c字符这种，可以把上面的d元素改成数组，并且加多一个映射数组`path2s`.
```js
d: [{
  value: path.d,
  duration: 1000, // 转换过程1s过渡
  easing: "easeInOutQuad"
}, {
  value: path2s[index].d,
  duration: 1000,
  easing: "easeInOutQuad",
}],
```

## 完整代码
animejs版本为3.2.2。

[demo](https://www.clzczh.top/demos/triangle-font.html)

```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      svg {
        width: 600px;
        height: 600px;
      }
    </style>
  </head>

  <body>
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width="400"
      height="800"
      viewBox="0 0 400 800"
    >
      <path
        id="path1"
        fill="currentColor"
        d="M265 164l52 30l53 31v-61v-62l-53 30z"
      />
      <path
        id="path2"
        fill="currentColor"
        d="M370 89l-53 -32l-52 -30v62v61l52 -31z"
      />
      <path
        id="path3"
        fill="currentColor"
        d="M145 89l52 30l53 31v-61v-62l-53 30z"
      />
      <path
        id="path4"
        fill="currentColor"
        d="M25 164l52 30l53 31v-61v-62l-53 30z"
      />
      <path
        id="path5"
        fill="currentColor"
        d="M130 239l-53 -32l-52 -30v62v61l52 -31z"
      />
      <path
        id="path6"
        fill="currentColor"
        d="M25 314l52 30l53 31v-61v-62l-53 30z"
      />
      <path
        id="path7"
        fill="currentColor"
        d="M130 389l-53 -32l-52 -30v62v61l52 -31z"
      />
      <path
        id="path8"
        fill="currentColor"
        d="M25 464l52 30l53 31v-61v-62l-53 30z"
      />
      <path
        id="path9"
        fill="currentColor"
        d="M145 539l52 30l53 31v-61v-62l-53 30z"
      />
      <path
        id="path10"
        fill="currentColor"
        d="M370 539l-53 -32l-52 -30v62v61l52 -31z"
      />
      <path
        id="path11"
        fill="currentColor"
        d="M265 464l52 30l53 31v-61v-62l-53 30z"
      />
    </svg>

    <script src="./anime.js"></script>
    <script>
      const path1s = [
        {
          id: "#path1",
          d: "M25 162l52 30l53 32v-62v-61l-53 30z",
        },
        {
          id: "#path2",
          d: "M130 237l-53 30l-52 32v-62v-61l52 30z",
        },
        {
          id: "#path3",
          d: "M25 312l52 30l53 32v-62v-61l-53 30z",
        },
        {
          id: "#path4",
          d: "M130 387l-53 30l-52 32v-62v-61l52 30z",
        },
        {
          id: "#path5",
          d: "M25 462l52 30l53 32v-62v-61l-53 30z",
        },
        {
          id: "#path6",
          d: "M130 537l-53 30l-52 32v-62v-61l52 30z",
        },
        {
          id: "#path7",
          d: "M25 612l52 30l53 32v-62v-61l-53 30z",
        },
        {
          id: "#path8",
          d: "M145 687l52 -31l53 -30v61v62l-53 -32z",
        },
        {
          id: "#path9",
          d: "M370 687l-53 30l-52 32v-62v-61l52 30z",
        },
        {
          id: "#path10",
          d: "M265 612l52 30l53 32v-62v-61l-53 30z",
        },
        {
          id: "#path11",
          d: "M490 612l-53 -31l-52 -30v61v62l52 -32z",
        },
      ];

      const path2s = [
        {
          id: "#path1",
          d: "M294 137l-61 -36l-60 -34v70v72l60 -37z",
        },
        {
          id: "#path2",
          d: "M173 226l60 35l61 37v-72v-70l-61 34z",
        },
        {
          id: "#path3",
          d: "M433 221l-62 -36l-60 -35v71v71l60 -37z",
        },
        {
          id: "#path4",
          d: "M311 302l60 34l62 37v-71v-71l-62 35z",
        },
        {
          id: "#path5",
          d: "M433 388l-62 -36l-60 -34v70v72l60 -37z",
        },
        {
          id: "#path6",
          d: "M173 388l60 35l61 37v-72v-70l-61 34z",
        },
        {
          id: "#path7",
          d: "M42 475l60 34l61 37v-71v-71l-61 35z",
        },
        {
          id: "#path8",
          d: "M294 475l-61 -36l-60 -35v71v71l60 -37z",
        },
        {
          id: "#path9",
          d: "M181 558l60 35l61 37v-72v-70l-61 35z",
        },
        {
          id: "#path10",
          d: "M433 558l-62 -35l-60 -35v70v72l60 -37z",
        },
        {
          id: "#path11",
          d: "M316 636l60 34l61 37v-71v-71l-61 35z",
        },
      ];

      const timeline = anime.timeline({
        autoplay: true,
        direction: "alternate",
        loop: true,
      });

      path1s.forEach(function (path, index) {
        timeline.add({
          targets: path.id,
          d: [{
            value: path.d,
            duration: 1000, // 转换过程1s过渡
            easing: "easeInOutQuad"
          }, {
            value: path2s[index].d,
            duration: 1000,
            easing: "easeInOutQuad",
          }],
          offset: 1000 + 10 * index, // 在上一个字符停留1s后才开始转换（每个小块有10ms延时，前面的小块先启动
        }, 0);    // 动画同时开始（如果没设置，那么就会是一个一个三角形依次执行
      });
    </script>
  </body>
</html>
```