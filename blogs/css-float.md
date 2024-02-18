---
title: 清除浮动的四种方式
categories: 前端
date: 2022-03-30 21:11:42
tags:
  - CSS
---

# 清除浮动的四种方式

## 浮动是什么

CSS 的 Float（浮动），会使元素向左或向右移动，直到外边缘碰到包含框或另一个浮动元素位置。

<br>

**浮动元素的特征**：

* 浮动的元素会脱离文档流
* 浮动的元素会紧挨在一起
* 浮动元素具有类似行内块元素的特性。所以行内元素有了浮动，不再需要转换为块级或行内块级元素元素
* 收缩。浮动的元素，如果没有设置width，会自动收缩为内容的宽度。比如块级盒子，如果块级盒子没有设置宽度，默认宽度和父级一样宽，但是添加浮动后，会收缩成内容的宽度

<br>

```html
<style>
  div[class^="box"] {
    width: 300px;
    height: 300px;
    background-color: pink;
    margin: 10px;
  }

  div[class^="box"]>div {
    height: 200px;
    background-color: purple;
  }

  div[class^="box"]>span {
    width: 200px;
    height: 200px;
    background-color: skyblue;
  }

  .box1>div {
    float: left;
  }

  .box1>span {
    float: left;
  }
</style>
<div class="box1">
  <div>浮动div</div>
  <span>浮动span</span>
</div>
<div class="box2">
  <div>不浮动div</div>
  <span>不浮动span</span>
</div>
```

![image-20220324192531879](https://s2.loli.net/2022/03/24/8XHOLg5cTheqoaJ.png)

<br>

## 浮动导致的问题1

浮动的元素，高度会塌陷，而高度的塌陷使我们页面后面的布局不能正常显示。

html

```html
<div class="container">
  <div class="box1 float">box1</div>
  <div class="box2 float">box2</div>
</div>
<div class="box3">box3</div>
```

<br>

css

```css
[class^='box'] {
  width: 200px;
  height: 200px;
  color: #fff;
}

.box1 {
  background-color: rgb(160, 3, 3);
}

.box2 {
  background-color: rgb(11, 11, 133);
}

.box3 {
  height: 300px;
  background-color: rgb(85, 7, 85);
}

.float {
  float: left;
}
```



![image-20220310151022141](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207221331271.png)

实际上box3确实有300px，只是box1浮起来了，所以，box3有200px在box1下，设置box1的` opacity`为0即可看出来。

![image-20220310151222520](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207221331300.png)

<br>

## 浮动导致的问题2

```html
<style>
    li {
      float: left;
      width: 40px;
      height: 20px;
      background-color: pink;
      border-bottom: 2px solid purple;
      list-style: none;
    }
</style>

<div>
  <ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
    <li>6</li>
  </ul>
</div>
<div>
  <ul>
    <li>7</li>
    <li>8</li>
    <li>9</li>
    <li>10</li>
    <li>11</li>
    <li>12</li>
  </ul>
</div>
```

<br>

想要的效果：

![image-20220324195050428](https://s2.loli.net/2022/03/24/BVLHE1lpuTGysfP.png)

<br>

实际效果：

![image-20220324195120085](https://s2.loli.net/2022/03/24/GacKEHTUoqwmMh6.png)

**原因：` div`没有高度，不能给浮动的子元素一个容器，所以第二个` div`的`li`就紧贴到第一个` div`中最后的一个` li`去了 **



## 清除浮动

**清除浮动是为了清除使用浮动产生的影响。**

### 1. 浮动元素的祖先元素添加高度

**很少用**

```js
.container {
  height: 200px;
}
```

![image-20220310152509811](https://s2.loli.net/2022/03/24/6pVklvaxe5OyDNZ.png)

<br>

### 2. 额外标签法

在浮动元素后使用一个空元素，并携带` clear: both`属性。(当然，也可以是` left`、` right`值)，就是`both`，会把所有情况的浮动都清掉。

问题：

html

```html
<!-- 初级版：外墙板。第一个div(container)的margin-bottom失效了 -->
<div class="container">
  <div class="box1 float">box1</div>
  <div class="box2 float">box2</div>
</div>
<div class="clear"></div>
<div class="box3">box3</div>

<!-- 升级版：内墙版 -->
 <div class="container">
  <div class="box1 float">box1</div>
  <div class="box2 float">box2</div>
  <div class="clear"></div>
</div>
<div class="box3">box3</div>
```

<br>

css

```css
.clear {
  clear: both;
}
```

效果同上

<br>

### 3. overflow法

给浮动元素的元素添加` overflow: hidden`或` overflow: auto`清除浮动。

```css
.container {
  overflow: hidden;
}
```

效果如上图所示。



### 4. after伪元素法

IE8以上和非IE浏览器才支持:after，zoom(IE专有属性)可解决ie6,ie7浮动问题(个人倒是感觉没啥意义，IE6版本也太老了吧，这都得兼容的话，更何况这年头，还真不知道谁还用IE)



给浮动元素的容器添加一个clearfix的class，然后给这个class添加一个:after伪元素清除浮动。

```html
<div class="container clearfix">
  <div class="box1 float">box1</div>
  <div class="box2 float">box2</div>
</div>
<div class="box3">box3</div>
```



```css
.clearfix {
  /* 兼容IE6、7，个人觉得大可不必(乱说的) */
  zoom: 1;
}

.clearfix::after {
  content: '';
  /*因为clear对块级元素有效，伪元素:before和:after添加的内容默认是inline元素*/
  display: block;
  /* 清除浮动 */
  clear: both;
  /*不显示该区域，content为空，实际可省略*/
  visibility: hidden;
}
```

效果如上图所示。



实际上，下面的代码就足够了

```css
.clearfix:before {
  content: '';
  display: block;
  clear: both;
}
```

<br>

### 5. 双伪元素法

```css
.clearfix:after,
.clearfix:before {
  content: '';
  display: block;
  clear: both;
}
```

原理就和上面一样



<br>

参考链接：[关于清除浮动塌陷的几种方法总结](https://www.jb51.net/css/502268.html[)
