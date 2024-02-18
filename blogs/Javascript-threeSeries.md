---
title: Javascript_元素三大系列
date: 2021-10-03 20:43:11
categories: "前端"
tags:
  - JavaScript
---

# 元素三大系列

<p style="color:red">这里讲的三大系列的属性返回的是数值，不带单位</p>

## 元素偏移量 offset 系列

使用 offset 系列相关属性可以动态得到该元素的位置（偏移）、大小等。

作用：

1. 获得元素距离定位父元素的位置（如果没有父元素或者父元素都没有定位，则是距离 body 的位置）
2. 获得元素自身宽度高度

<p style="color:red">返回的是数值，不带单位</p>

offset 系列常用属性:

| offset 系列属性      |                        作用                         |
| -------------------- | :-------------------------------------------------: |
| element.offsetParent | 返回该元素带有定位的父级元素，都没有定位则返回 body |
| element.offsetTop    |      返回该元素相对于带有定位父元素上方的偏移       |
| element.offsetLeft   |     返回该元素相对于带有定位父元素左边框的偏移      |
| element.offsetWidth  |  返回该元素自身包括 padding、边框、内容区的宽度）   |
| element.offsetHeight |             返回该元素自身高度（同上）              |

<p style="color:red">返回的是数值，不带单位</p>

[offsetParent 示例](https://codepen.io/13535944743/pen/YzQqBLb)

[offsetTop 和 offsetLeft 示例](https://codepen.io/13535944743/pen/QWgKEer)

[offsetWidth 和 offsetHeight 示例](https://codepen.io/13535944743/pen/VwWKKKm)

### offset 和 style 的区别

| offset                                      |                       style                       |
| ------------------------------------------- | :-----------------------------------------------: |
| offset 可以得到所有样式表的样式值           |        style 只能得到行内样式表中的样式值         |
| offset 得到的是数值,没有单位                |          style 得到的是带有单位的字符串           |
| offsetWidth 包含 width+padding+border       | style.width 只包含 width,不包含 padding 和 border |
| offset 系列属性是只读属性，只能获取不能赋值 |   style.width 等是可读写属性，可以获取也能赋值    |

<p style="color:red">获取元素大小位置，offset更合适，更改元素大小位置，style更合适</p>

offset 各属性示意图：

![](https://i.loli.net/2021/09/04/jV21Y8vt6dJeKTO.png)

例子：

[计算鼠标在盒子里的坐标](https://codepen.io/13535944743/pen/rNwMmqz)

### 案例

拖动的模态框

## 元素可视区 client 系列

使用 client 系列相关属性来获取元素可视区的相关信息。

作用：

1. 动态得到该元素的边框大小
2. 动态得到该元素的元素大小

<p style="color:red">返回的是数值，不带单位</p>

| client 系列属性      |                     作用                     |
| -------------------- | :------------------------------------------: |
| element.clientTop    |              返回元素上边框大小              |
| element.clientLeft   |              返回元素左边框大小              |
| element.clientWidth  | 返回自身包括 padding、内容区的宽度，不含边框 |
| elemeng.clientHeight |             返回自身高度（同上）             |

<p style="color:red">返回的是数值，不带单位</p>

示例：

[client 系列](https://codepen.io/13535944743/pen/YzQGvJR)

## 立即执行函数

```
	(function(){
  		//代码
	})();

	或
	(function() {
  		//代码
	}())


```

作用：创建一个独立的作用域，避免命名冲突的问题，因为里面的变量都是局部变量。

第一种形式比较好理解，首先需要定义函数，但是是立即执行函数，所以不需要函数名，不加函数名的话有可能是写错代码了，所以立即执行函数的语法就是用"()"包住立即执行函数，就可以区分出错误代码和立即执行函数。之后的"()"便是函数调用。

示例：

```
	(function(a, b) {
  		console.log(a * b);
	})(9, 8);

	//********************

	(function(a, b) {
  		console.log(a * b);
	}(9, 8));


```

## 元素滚动 scroll 系列

使用 scroll 系列的相关属性可以动态的得到该元素的大小，滚动距离等。

<p style="color:red">返回的是数值，不带单位</p>

| scroll 系列属性      |                         作用                         |
| -------------------- | :--------------------------------------------------: |
| element.scrollTop    |                 返回被卷去的上侧距离                 |
| element.scrollLeft   |                   被卷去的左侧距离                   |
| element.scrollWidth  | 返回自身<b style="color: red">实际</b>宽度，不含边框 |
| element.scrollHeight |                返回自身实际高度(同上)                |

<p style="color:red">返回的是数值，不带单位</p>

scrollWidth、scrollHeight：返回自身实际宽度、高度，即使内容区的内容不显示出来

示例：

[scroll 系列](https://codepen.io/13535944743/pen/ZEyBQZz)

### 案例：

仿淘宝固定侧边栏

知识点：

element.scrollTop 是返回元素被卷去的上侧距离，而 window.pageYOffset 是页面被卷去的距离

## 总结

<p style="color:red">返回的是数值，不带单位</p>

| 三大系列大小对比    |                     作用                     |
| ------------------- | :------------------------------------------: |
| element.offsetWidth |   返回自身包括 padding、边框、内容区的宽度   |
| element.clientWidth | 返回自身包括 padding、内容区的宽度，不含边框 |
| element.scrollWidth |         返回自身实际的宽度，不含边框         |

用法：

1. offset 系列经常用于获得元素位置（<b style="color: red">offsetTop、offsetLeft</b>）
2. client 系列经常用于获取元素大小（<b style="color: red">clientWidth、clientHeight</b>）
3. scroll 经常用于获取滚动距离（<b style="color: red">scrollTop、scrollLeft</b>）
4. 页面滚动距离不是通过 scrollTop、scrollLeft 获取，而是通过 window.pageXOffset、window.pageYOffset 获取

学习链接：
[pink 老师前端入门](https://www.bilibili.com/video/BV1Sy4y1C7ha)
