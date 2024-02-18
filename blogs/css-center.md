---
title: 居中对齐的几种方法
date: 2022-03-25 18:40:18
categories: 前端
tags:
  - CSS
---

# 居中对齐的几种方法

看面试题，自己总结了下，顺便写了对应例子，加深印象。

## 水平居中

### 给` div`设置一个宽度，再添加` margin: 0 auto`

<b style="color: red">必须要添加宽度，只对块级元素有效</b>

```html
<style>
  .container {
    width: 400px;
    height: 400px;
    background-color: pink;
    color: #fff;
  }

  .box {
    width: 100px;
    height: 100px;
    background-color: purple;
    margin: 0 auto;
  }

  .inline-block-box {
    display: inline-block;
    width: 100px;
    height: 100px;
    background-color: purple;
    margin: 0 auto;
  }
</style>

<div class="container">
  <div class="box">块级元素</div>
  <span class="inline-block-box">行内块元素</span>
  <br />
  <span>行内元素</span>
</div>
```

![image-20220324213321588](https://s2.loli.net/2022/03/24/d3PXbaQ47yeNMcB.png)

<br>

### 给父元素添加` text-aligin: center`

<b style="color: red">只对行内块元素、行内元素有效</b>

```css
.container {
  width: 400px;
  height: 400px;
  background-color: pink;
  color: #fff;
  text-align: center;
}

.box {
  width: 100px;
  height: 100px;
  background-color: purple;
}

.inline-block-box {
  display: inline-block;
  width: 100px;
  height: 100px;
  background-color: purple;
}

span {
  background-color: skyblue;
}
```

![image-20220324214157484](https://s2.loli.net/2022/03/24/34np6EksUagW8X5.png)

<br>

## 水平垂直居中

### 计算法

#### 父元素跟着子元素 margin-top 移动问题

开始之前，先看下一个小问题

下面的例子中，我们想要让子元素离父元素有距离

```html
<style>
  .container {
    width: 600px;
    height: 400px;
    background-color: pink;
    color: #fff;
  }

  .box {
    width: 100px;
    height: 100px;
    background-color: purple;
    margin: 150px auto;
  }
</style>

<div class="container">
  <div class="box"></div>
</div>
```

![image-20220324220516244](https://s2.loli.net/2022/03/24/Lwy6iaJDdVrYnKm.png)

<br>

结果，子元素并没有外边距效果，反而是父元素出现了外边距的效果。

这是因为，根据规范，父元素的子元素的上边距(` margin-top`)，如果碰不到有效的` border`或者` padding`，就会一层一层的找自己的祖先元素，直到找到祖先元素有有效的` border`或`border`为止

<br>

解决方案：

1. 给父元素添加` padding-top`
2. 给父元素添加` border-top`
3. **给父元素添加` overflow: hidden`**(<b style="color: red">推荐</b>)

```css
.container {
  width: 600px;
  height: 400px;
  background-color: pink;
  color: #fff;

  /* padding-top: 1px; */
  /* border-top: 1px solid transparent; */
  overflow: hidden;
}
```

<br>

#### 开始

首先` margin`左右可以直接设置` auto`实现居中，但是的上下不行。

计算法：`margin上下值 = (父元素高度-子元素高度)/2`

在这个例子中，父元素的高度为` 400px`，子元素的高度为` 100px`，所以` margin上下值`设置为` 150px`

```css
.container {
  width: 600px;
  height: 400px;
  background-color: pink;
  color: #fff;
  overflow: hidden;
}

.box {
  width: 100px;
  height: 100px;
  background-color: purple;
  margin: 150px auto;
}
```

![image-20220324222200326](https://s2.loli.net/2022/03/24/zV8YmsNHEGtWyXP.png)

<br>

### 绝对定位四 0 法

**设置四个方向都为 0，然后设置` margin`为` auto`，因为宽高固定，所以对应方向平分，可以实现水平垂直居中**

```js
.container {
  position: relative;
  width: 600px;
  height: 400px;
  background-color: pink;
  color: #fff;
}

.box {
  position: absolute;
  width: 200px;
  height: 100px;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  margin: auto;
  background-color: purple;
}
```

<br>

### 绝对定位 + 计算法(` margin`负值)

```css
.container {
  position: relative;
  width: 600px;
  height: 400px;
  background-color: pink;
  color: #fff;
}

.box {
  position: absolute;
  width: 200px;
  height: 100px;
  top: 50%;
  left: 50%;
  margin: -50px 0 0 -100px;
  /* margin-top、margin-left 分别是该子元素高宽的一半(负值)*/
  background-color: purple;
}
```

<br>

### 绝对定位 + ` transform`

```css
.container {
  position: relative;
  width: 600px;
  height: 400px;
  background-color: pink;
  color: #fff;
}

.box {
  position: absolute;
  width: 200px;
  height: 100px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: purple;
}
```

<br>

### flex 布局法

```css
.container {
  display: flex;
  width: 600px;
  height: 400px;
  background-color: pink;
  /* 垂直居中 */
  align-items: center;
  /* 水平居中 */
  justify-content: center;
}

.box {
  width: 200px;
  height: 100px;
  background-color: purple;
}
```

<br>

对于**宽高不定**的元素，后面两种方法(<b style="color: red">绝对定位+` transform`、` flex`布局法</b>)，可以实现元素的水平垂直居中。

<br>

参考资料：

- [104 道 CSS 面试题，助你查漏补缺](https://segmentfault.com/a/1190000022021557)

- [父盒子跟随子盒子 margin-top 移动问题](https://blog.csdn.net/weixin_45786214/article/details/106297864)
