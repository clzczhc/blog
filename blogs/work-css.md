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

### 注意点

icon 混合后的颜色可以是设置的渐变色的前提是 icon 的颜色是黑/白，如果是其他颜色，则不一定能得到想要的效果。
