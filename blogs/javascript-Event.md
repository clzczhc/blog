---
title: 事件
date: 2021-09-09 16:25:53
categories: 前端
tags:
  - JavaScript
---

# JavaScript 事件

## 注册事件（绑定事件）

给元素添加事件，称为注册事件或者绑定事件。

有传统方式和方法监听方式

### 传统方式

利用 on 开头的事件，如 onclick, 同一个元素同一个事件只能设置一个处理函数，出现多个处理函数的话，后面的会覆盖前面的。

例子：

```

	<script>
        var btn = document.querySelector("button");
        btn.onclick = function () {
            console.log(1);
        };
        btn.onclick = function () {
            console.log(2);
        }
        //结果是2
    </script>


```

### 方法监听注册方式

#### addEventListener():

`eventTarget.addEventListener(type, listener[, useCapture])`

type: 事件类型字符串，如 click、mouseover 等，不带 on

listener: 事件处理函数，事件发生会调用该监听函数

useCapture: 可选参数，是一个布尔值，默认是 false。决定监听器的触发阶段是捕获阶段还是冒泡阶段<a href="#anchor">详见</a>。

addEventListener() 是 W3C DOM 规范中提供的注册事件监听器的方法。

优点：

1. 允许给一个事件注册多个监听器

例子：

addEventListener:

```
        const btn = document.getElementById("btn");

        btn.addEventListener("click", function () {
            console.log(1);
        });
        btn.addEventListener("click", function () {
            console.log(1);
        });

		//结果：1, 1

```

此处是个人见解：

当两个监听函数一样时，由上可发现会输出两次 1,这个其实是因为上面两个匿名函数看似一样，实际它们所开辟的内存空间不一样。

把相同的方法抽出来后会发现，无法实现多个监听，就是因为两个方法变成完全一样了。

```

    const btn = document.getElementById("btn");

    btn.addEventListener("click", fn)
    btn.addEventListener("click", fn)

    function fn() {
      console.log(1);
    }

	//结果：1

```

2. 可以控制监听器的触发阶段（可选捕获或冒泡）
3. 对任何 DOM 元素都是有效的，而不仅仅只对 HTML 元素有效。(<b style="color:red">未找到合适案例</b>)

参数是匿名函数和是箭头函数区别：

它们绑定不同的 this 对象。匿名函数和传统方式一样会创建独有的 this 对象（即触发事件的元素），而箭头函数是继承绑定它所在函数的 this 对象。

例子：

匿名函数：

```

	<div id="div">
        <button id="btn">点击</button>
    </div>

    <script>
        const div = document.getElementById("div");

        const btn = document.getElementById("btn");
        div.addEventListener("mouseenter", function () {
            console.log(this);
            btn.addEventListener("click", function () {
                console.log(this);
            })
        });
    </script>

```

结果：

![](https://pic.imgdb.cn/item/612c38f844eaada739293bc3.jpg)

箭头函数：

```

	<div id="div">
        <button id="btn">点击</button>
    </div>

    <script>
        const div = document.getElementById("div");

        const btn = document.getElementById("btn");
        div.addEventListener("mouseenter", function () {
            console.log(this);
            btn.addEventListener("click", () => {
                console.log(this);
            })
        });
    </script>

```

结果：

![](https://pic.imgdb.cn/item/612c398044eaada7392a354b.jpg)

#### attachEvent():

`eventTarget.attachEvent(eventNameWithOn, callback)`

eventNameWithOn: 事件类型字符串。如 onclick、onmouseover，要带 on

callback: 事件处理函数，事件发生会调用该回调函数

IE9 之前的 IE 不支持，对应有 attachEvent()，用法和 addEventListener 近似，不过事件需要变回传统方式的 on 系列。

attachEvent 缺点：this 的值会变成 window 对象的引用而不是触发事件的元素。

## 删除事件（解绑事件）

### 传统方式

eventTarget.onclick = null;

例子:

```

		const btn = document.getElementById("btn");
        btn.onclick = function () {
            console.log(1);
        };

        btn.onclick = null;

```

### 方法监听注册方式

#### 对应 addEventListener

`eventTarget.removeEventListener(type, listener[, useCapture]);`

移除完全匹配的监听，只有事件处理函数完全一样，包括开辟的内存空间。

完全匹配例子：

```

		const btn = document.getElementById("btn");
        btn.addEventListener("click", fn); //第二个参数只要函数名就可以，不需要调用
        btn.removeEventListener("click", fn);

        function fn() {
            console.log(1);
        }

```

不完全匹配例子（开辟的内存空间不一样）:

```

		const btn = document.getElementById("btn");
        btn.addEventListener("click", function () {
            console.log(1);
        });
        btn.removeEventListener("click", function () {
            console.log(1);
        });

```

## DOM 事件流

<p id="anchor">事件流描述的是从页面中接收事件的顺序。</p>

事件发生时会在元素节点之间按照特定的顺序传播，这个传播过程就是 DOM 事件流。

例如给一个 div 注册了事件：

DOM 事件流分为 3 个阶段：

1. 捕获阶段
2. 当前目标阶段
3. 冒泡阶段

事件捕获：网景最早提出，由 DOM 最顶层节点开始，然后逐级向下传播到绑定事件的元素接受的过程。

事件冒泡：IE 最早提出，事件逐级向上传播到 DOM 最顶层节点的过程。

![](https://pic.imgdb.cn/item/612c5ace44eaada7396858fb.jpg)

事件发生时会在元素节点之间按照特定的顺序传播，这个传播过程即 DOM 事件流。

注意：

1. JS 代码只能执行捕获或者冒泡其中一个阶段
2. onclick 和 attachEvent 只能得到冒泡阶段

```

		const btn = document.getElementById("btn");
        btn.onclick = () => alert(1);
        document.body.onclick = () => alert(2);
        document.documentElement.onclick = () => alert(3);
        document.onclick = () => alert(4);
		//点击按钮后，弹出顺序1、2、3、4

```

3. addEventListener(type, listener[, useCapture])第三个参数默认是 false，表示在冒泡阶段调用事件处理程序，如果是 true，则表示在事件捕获阶段调用事件处理程序。

为 false:

```

		const btn = document.getElementById("btn");
        btn.addEventListener("click", () => alert(1), false);
        document.body.addEventListener("click", () => alert(2), false);
        document.documentElement.addEventListener("click", () => alert(3));
        document.addEventListener("click", () => alert(4));
		//点击按钮后，弹出顺序1、2、3、4

```

为 true:

```

		const btn = document.getElementById("btn");
        btn.addEventListener("click", () => alert(1), true);
        document.body.addEventListener("click", () => alert(2), true);
        document.documentElement.addEventListener("click", () => alert(3), true);
        document.addEventListener("click", () => alert(4), true);
		//点击按钮后，弹出顺序4、3、2、1

```

4. 有些事件是没有冒泡的，如 blur, focus, mouseenter, onmouseleave

例如：

```

		const btn = document.getElementById("btn");
        btn.addEventListener("mouseleave", () => alert(1));
        document.body.addEventListener("mouseleave", () => alert(2));
        document.documentElement.addEventListener("mouseleave", () => alert(3));
        document.addEventListener("mouseleave", () => alert(4));
		//当鼠标放在按钮里后，离开按钮，只会弹出1

```

5. 事件冒泡有时候会带来麻烦，可以通过 e.stopPropagation()方法阻止事件冒泡

## 事件对象

事件处理函数可以带参数，带的参数就是事件对象。

` eventTarget.onclick = function(event) {}`

`eventTarget.addEventListener("click", function(event){})`

如上式所示，event 就是事件对象，，它代表事件的状态，如键盘按键的状态、鼠标的位置、鼠标按钮的状态等。事件发生后，跟事件相关的一系列信息的集合都在这个对象里面。

<p style="color:red">不需要传递实参</P>

注册事件时，event 对象会被系统自动创建，并依次传递给事件监听器（事件处理函数）。

在 IE6~8 中，浏览器不会给方法传递参数，需要的话，要到 window.event 中获取。

e = e || window.event;

### 事件对象的常见属性和方法

![](https://pic.imgdb.cn/item/612cc0a644eaada7392c26d3.jpg)

e.target 和 this 的区别：

this 是事件绑定的元素（匿名函数形式），函数的调用者。

e.target 是事件触发的元素。

例子：

```

		const ul = document.querySelector("ul");
        ul.addEventListener("click", function (e) {
            console.log(this);
            console.log(e.target);
        });
		//点击li里面的文字，依次打印出的是ul和li

```

#### 阻止默认行为

`e.preventDefault()`

例子：

```

		const a = document.querySelector("a");
        a.addEventListener("click", (e) => e.preventDefault());

```

#### 阻止事件冒泡

`e.stopPropagation()`

例子：

```

		const btn = document.getElementById("btn");
        btn.addEventListener("click", (e) => {
            alert(1);
            e.stopPropagation();  //只要在事件处理函数里使用就会执行完这个函数，才阻止冒泡。
								  //不过建议放在最后面，更有逻辑性的感觉
        });
        document.body.addEventListener("click", () => alert(2));
        document.documentElement.addEventListener("click", () => alert(3));
        document.addEventListener("click", () => alert(4));

```

没加 e.stopPropagation()之前会依次弹出 1、2、3、4，在按钮绑定的事件中,加上之后只会弹出 1

### 事件委托

事件委托也被称为事件代理，在 jQuery 里面称为事件委派。

#### 事件委托原理

不需要给每个子结点单独设置事件监听器，而是把事件监听器设置在其父节点上，然后利用冒泡原理去影响子节点。

例子：

```

		const ul = document.querySelector("ul");
        ul.addEventListener("click", (e) => e.target.style.color = "red");

```

上面例子：直接给 li 的父节点绑定监听器，然后利用 e.target 找到当前点击的 li，点击 li，事件会冒泡到 ul 上，而 ul 上有注册事件，就会触发事件监听器。

#### 作用

只需要操作一次 DOM，提高了程序性能。

### 常用的鼠标事件

![](https://pic.imgdb.cn/item/612d849544eaada7399433ab.jpg)

ontextmenu：鼠标右键菜单，可用于取消默认的菜单

selectstart：开始选中，可用于禁止选中文字

#### 常用鼠标事件对象属性

![](https://pic.imgdb.cn/item/612d85bf44eaada7399611a9.jpg)

### 案例

跟随鼠标的天使

### 常用的键盘事件

![](https://pic.imgdb.cn/item/612d863444eaada73996cc24.jpg)

<p style="color:red">onkeypress不识别功能键，如ctrl、shift等<br>执行顺序是： keydown-->keypress-->keyup</p>

首先，keyup 是弹起时才会触发的，所以顺序是最后的，所以只需要记得 keydown 优先级更高就行

```

		document.addEventListener("keyup", () => console.log("up"));
        document.addEventListener("keydown", () => console.log("down"));
        document.addEventListener("keypress", () => console.log("press"));
		//按非功能键，依次输出顺序down、press、up
		//按功能键，则依次输出down、up

```

#### 常用键盘事件对象属性

keyCode:返回该键的 ASCII 值（数字）

<p style="color:red">onkeydown和onkeyup不区分字母大小写，onkeypress区分字母大小写</p>

### 案例

模拟京东快递单号查询案例

参考链接：[EventTarget.addEventListener()](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener#why_use_addeventlistener.3f)

[pink 老师前端入门](https://www.bilibili.com/video/BV1Sy4y1C7ha)
