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
