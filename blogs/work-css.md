# 工作中的那点事之 CSS

## 前言

记录一下工作的 CSS 相关的问题。

## 伪元素写多行内容

> 使用场景：不操作 DOM，通过 css 来简单调整内容。
> 在公司中遇到的场景则是：使用 answer 打造问答平台，只是改配置，并没有做魔改。需要调整文案，并且文案是可以换行的。

```css
.container {
  position: relative;
  width: 200px;
  height: 200px;
  background-color: #eee;
}

.container:hover::before {
  content: "赤\A蓝\A紫";
  white-space: pre;
  position: absolute;
  top: 0;
  left: 0;
}
```

效果：![](https://www.clzczh.top/CLZ_img/images/fake.gif)

## `mix-blend-mode`实现 icon 渐变色

使用场景：同一个 icon，对应不同的会员体系，icon 的颜色不同，并且是渐变色。

> color 并不支持渐变颜色，如果是每种类型都保存一张切图，既占存储，浏览网页的时候还得下载多张不同颜色的图片。

使用`mix-blend-mode`则可以实现这个问题。

> 定义：mix-blend-mode CSS 属性描述了元素的内容应该与元素的直系父元素的内容和元素的背景如何混合。

先上代码，跟效果图。

svg

```html
<svg
  width="16"
  height="16"
  viewBox="0 0 16 16"
  fill="none"
  xmlns="http://www.w3.org/2000/svg"
>
  <circle cx="8" cy="8" r="7.25" stroke="currentColor" stroke-width="1.5" />
  <path
    d="M8 15.25C7.7649 15.25 7.48181 15.1442 7.15894 14.8321C6.83246 14.5165 6.50419 14.0235 6.21224 13.3562C5.62932 12.0239 5.25 10.1307 5.25 8C5.25 5.86928 5.62932 3.97615 6.21224 2.64376C6.50419 1.97645 6.83246 1.48352 7.15894 1.16789C7.48181 0.855751 7.7649 0.75 8 0.75C8.2351 0.75 8.51819 0.855751 8.84106 1.16789C9.16754 1.48352 9.49581 1.97645 9.78776 2.64376C10.3707 3.97615 10.75 5.86928 10.75 8C10.75 10.1307 10.3707 12.0239 9.78776 13.3562C9.49581 14.0235 9.16754 14.5165 8.84106 14.8321C8.51819 15.1442 8.2351 15.25 8 15.25Z"
    stroke="currentColor"
    stroke-width="1.5"
  />
  <line
    x1="1"
    y1="5.75"
    x2="15"
    y2="5.75"
    stroke="currentColor"
    stroke-width="1.5"
  />
  <line
    x1="1"
    y1="10.25"
    x2="15"
    y2="10.25"
    stroke="currentColor"
    stroke-width="1.5"
  />
</svg>
```

```html
<style>
  .container {
    display: flex;
    width: max-content;
    background: linear-gradient(
      to right,
      rgba(255, 0, 0) 0%,
      rgba(0, 255, 0) 100%
    );
  }

  .icon-box {
    background: #fff;
    color: #000;
    mix-blend-mode: lighten;
  }

  .icon-box svg {
    width: 64px;
    height: 64px;
  }
</style>
<div class="container">
  <div class="icon-box">
    <svg>...</svg>
  </div>
</div>
```

效果：![](https://www.clzczh.top/CLZ_img/images/20250313233928.png)

**说明**：
`mix-blend-mode: lighten`: 混合模式为对每个像素取 RGB 通道最大值。

1. 渐变色和 icon 混合后，icon 显示渐变色，因为 icon 的颜色为`#000`，即`rgb(0, 0, 0)`。所以它和其他颜色混合后，就会变成其他颜色。
2. 渐变色和 icon 外的地方混合后，还是显示白色，白色是`rgb(255, 255, 255)`，所以混合后还是白色。

`mix-blend-mode: darken`: 混合模式为对每个像素取 RGB 通道最小值。
同理，暗黑模式的时候，把颜色修改为暗黑模式的颜色，同时把`mix-blend-mode: lighten;`改为`mix-blend-mode: darken;`即可

```css
.icon-box {
  background: #000;
  color: #fff;
  mix-blend-mode: darken;
}
```

![](https://www.clzczh.top/CLZ_img/images/20250313234815.png)

**注意点**：
icon 混合后的颜色可以是设置的渐变色的前提是 icon 的颜色是黑/白，如果是其他颜色，则不一定能得到想要的效果。

## 给透明图片的文字添加阴影

```css
backdrop-filter: drop-shadow(0 0 15px rgba(0, 0, 0));
```

[drop-shadow](https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter-function/drop-shadow)

**注意点**：
理论上，`drop-shadow`的阴影跟`box-shadow`阴影的值是一样的。但是实际上，大部分浏览器不支持`drop-shadow`的扩散半径这个参数（Chrome 最新版本 134 都不支持）

所以相当于跟`box-shadow`的值有所差异

## `clip-path`绘制特殊图形

```html
<style>
  .arrow {
    width: 200px;
    height: 100px;
    clip-path: polygon(0 0, 75% 0, 100% 50%, 75% 100%, 0 100%, 25% 50%);
    background: pink;
  }
</style>
<div class="arrow"></div>
```

![](https://www.clzczh.top/CLZ_img/images/20250317215908.png)

> `clip-path`还支持 svg 的`path`，所以基本上大部分图形都可以使用`clip-path`绘制。

## 利用`background`的层级概念给背景添加遮罩

> 需要给背景添加遮罩，通过绝对定位添加遮罩层，还需要解决阻挡事件的问题，可以利用`background`的层级概念给背景添加遮罩。

`background`支持设置多个背景，第一个背景在最上面，最后一个背景在最下面。

所以说，理论上来说，设置`background-image: rgba(0, 0, 0, 0.5), url("./21.png")`即可。
但是实际上会报错：
![](https://www.clzczh.top/CLZ_img/images/20250317221806.png)

这是因为`background-image`不支持纯色背景，所以需要将纯色改为起止渐变一样的渐变色，
即`background-image: linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.5)), url("./21.png");`

```css
.box {
  width: 400px;
  height: 200px;
  background-image: linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.5)),
    url("./21.png");
  background-repeat: no-repeat;
  background-size: cover;
}
```

效果：
![](https://www.clzczh.top/CLZ_img/images/20250317222619.png)

## 栅格系统轻量化工具

### flexboxgrid

[flexboxgrid](https://github.com/kristoferjoseph/flexboxgrid)

| flexboxgrid                                                                            | antd grid                                       |
| :------------------------------------------------------------------------------------- | :---------------------------------------------- |
| 12 栅格                                                                                | 24 栅格                                         |
| 添加指定类名用法，`row`、`col-xs-12`，`col-lg-4`                                       | 组件用法，Row、Col 组件                         |
| 不需要响应式时，没有专门的用法，而是通过`col-xs-*`来使用。（媒体查询设置的 min-width） | 不需要响应时，可以直接使用 span={8}的方式来使用 |

flexboxgrid 有大部分 antd grid 的效果。

flexboxgrid 实现原理跟 antd grid 类似，都是通过 flex-basis 和 max-width 来实现

### react-flexbox-grid

[react-flexbox-grid](https://github.com/roylee0704/react-flexbox-grid)

flexboxgrid 的 react 组件版本
![](https://www.clzczh.top/CLZ_img/images/20250317223008.png)

## flex 左右定宽，中间自适应溢出省略号

关键点：中间设置`min-width: 0`
