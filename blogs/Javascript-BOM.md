---
title: BOM
date: 2021-10-01 21:14:05
categories: 前端
tags:
  - JavaScript
---

# BOM(浏览器对象模型)

## 概述

BOM(Brower Object Model) 即浏览器对象，它提供了独立于内容而与浏览器窗口进行交互的对象，核心对象是 window。

BOM 缺乏标准，Javascript 语法的标准化组织是 ECMA，DOM 的标准化组织是 W3C，BOM 最初是 Netscape 浏览器标准的一部分

### BOM 和 DOM

![](https://pic.imgdb.cn/item/612e171044eaada739b68a93.jpg)

### BOM 构成

BOM 比 DOM 更大，它包括 DOM。

![](https://pic.imgdb.cn/item/612e176e44eaada739b76e73.jpg)

window 对象是浏览器的顶级对象。

1. 它是 JS 访问浏览器窗口的一个接口
2. 它是一个全局对象。定义在全局作用域中的变量函数都会变成 window 对象的属性和方法。调用时可以省略 window,alert()和 prompt()都是 window 对象方法。

例子：

```
		var num = 1;
        console.log(num); //实际console.log(window.num);

        function fn() {
            alert(1); //window.alert(1);
        }
        fn(); //window.fn();


```

## window 对象的常见事件

### 窗口加载事件

#### load 事件

`window.addEventListener("load", function(){});`

是窗口（页面）加载事件，当文档内容完全加载完成后会触发事件（包括图像、脚本文件、CSS 文件等），就调用的处理函数。

作用：有了窗口加载事件就可以把 JS 代码放在页面元素上方。因为 load 事件是等页面内容完全加载完毕，才去执行事件处理函数。

例子：

```
		<!DOCTYPE html>
		<html lang="zh-CN">

		<head>
    		<meta charset="UTF-8">
    		<meta http-equiv="X-UA-Compatible" content="IE=edge">
    		<meta name="viewport" content="width=device-width, initial-scale=1.0">
    		<title>Myself</title>
    		<script>
        		window.addEventListener("load", () => {
            		const btn = document.querySelector("button");
            		btn.addEventListener("click", () => {
                		alert(1);
            		});
        		});
    		</script>
		</head>

		<body>
    		<button>点击</button>

		</body>

		</html>


```

#### DOMContentLoaded 事件

`document.addEventListener("DOMContentLoaded", function(){});`

DOMContentLoaded 事件触发时，仅当 DOM 加载完成时，不包括样式表，图片等。IE9 以上才支持。

应用背景：当页面的图片很多时，从用户访问到 onload 触发可能需要较长的时间，会影响到用户的体验，此时用 DOMContentLoaded 事件更合适。

用法和 load 事件类似。

### 调整窗口大小事件

`window.addEventListener("resize", function(){});`

只要窗口大小发生变化，就会触发事件。

例子：

```
		const btn = document.querySelector("button");
        window.addEventListener("resize", () => {
            if (window.innerWidth < 800) { //window.innerWidth是当前窗口大小
                btn.style.display = "none";
            } else {
                btn.style.display = "inline";
            }
        })

```

## 定时器

### setTimeout()定时器

`window.setTimeout(调用函数，[延迟的毫秒数]);`

用于设置一个定时器，在时间到后执行调用函数。

注意：

1. window 可以省略
2. 延迟的毫秒数默认是 0
3. 一般给定时器一个标识符，方便停止定时器等操作

调用函数也称为回调函数 callback。普通函数按照代码顺序直接调用，而 setTimeout 需要等待时间，时间到了才调用函数，因此被称为回调函数。

注册事件时的事件处理函数也是回调函数。

例子：

```
		let timer = setTimeout(fn, 1000);

        function fn() {
            console.log("时间到了");
        }

```

### 停止 setTimeout()定时器

`window.clearTimeout(timeoutID)`

例子：

```
		let timer = setTimeout(fn, 1000);

        function fn() {
            console.log("时间到了");
        }

        clearTimeout(timer);

```

### setInterval()定时器

`window.setInterval(回调函数，[延迟的毫秒数]);`

和 setTimeout()基本一样，不一样的是，setInterval()会重复调用回调函数，每隔一段时间，就调用一次回调函数。

注意：第一次执行也是需要等待延迟的毫秒数才会执行

例子：

```
		let timer = setInterval(fn, 1000);

        function fn() {
            console.log("时间到了");
        }

```

#### 案例

倒计时效果

### 停止 setInterval()定时器

`window.clearInterval(intervalID)`

需要注意的是 setTimeout()和 setInterval()共用一个编号池，技术上，clearTimeout()和 clearInterval() 可以互换。但是，为了避免混淆，不要混用取消定时函数。[来源](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout)

例子：

```
		const start = document.querySelector(".start");
        const stop = document.querySelector(".pause");
        let timer = null;

        start.addEventListener("click", () => {
            timer = setInterval(() => {
                console.log("时间到了");
            }, 1000);
        });
        stop.addEventListener("click", () => {
            clearInterval(timer);
        });

```

#### 案例

发送短信案例

## this 指向

this 的指向只有在函数执行的时候才能确定指向，一般情况下 this 指向的是调用它的对象。

1. 全局作用域和普通函数中 this 指向全局对象 window,定时器里面的 this 也是指向 window

例子：

```
		console.log(this);

        function fn() {
            console.log(this);
        }
        fn();

        setTimeout(function () {
            console.log(this);
        }, 200);

```

2. 方法调用中谁调用 this，this 就指向谁

<p style ="color: red">注意：匿名函数和箭头函数的区别：匿名函数和传统方式一样会创建独有的this对象（即触发事件的元素），而箭头函数是继承绑定它所在函数的this对象</p>

例子：

```
		let o = {            sayHi: function () {                console.log(this);            }        };        o.sayHi();
```

3. 构造函数中 this 指向构造函数的实例

例子：

```
		function Student() {            console.log(this);        }        let st = new Student();
```

## JS 执行机制

Javascript 语言的一个特点是单线程。

### 同步和异步

单线程会导致所有任务都要排队，即假如有计时器，程序会堵住。为了解决这个问题，利用多核 CPU 的计算能力，HTML5 提出 Web Worker 标准，允许 Javascript 开启多个线程，于是，JS 出现了同步和异步。

同步：前一个任务结束后再执行下一个任务。

异步：可以同时执行多个任务。

JS 为防止任务有排队或者等待时间较长的问题，把任务分为<b>同步任务</b>和<b>异步任务</b>两大类。

同步任务都在主线程上执行，形成一个执行栈。

异步任务：JS 的异步时通过回调函数实现的。一般有三种类型。异步任务的相关回调函数放在<b>任务队列</b>(消息队列)中。

1. 普通事件，如 click,resize 等
2. 资源加载，如 load,error 等
3. 定时器，如 setTimeout,setInterval 等

例子：

```
		console.log(1);    //①        setTimeout(function () {            console.log(3);        }, 0);             //②        console.log(2);     //③
```

分析：

1. 首先，执行主线程执行栈第一个任务，打印出 1
2. 第二个任务有回调函数，通过异步进程处理， 满足条件后（即点击事件点击了，定时器事件时间到了），把异步任务（回调函数）添加到任务队列中，但是不执行
3. 继续执行第三个任务，打印出 2；
4. 如果执行栈中的同步任务执行完后，系统会按顺序读取任务队列的异步任务，被读取的异步任务进入执行栈，执行。
5. 执行栈中没有任务后，还会一直监听着任务队列（比如 click 事件，用户一直有点击的可能），又称为"事件循环",任务队列中有新任务，则该任务进入执行栈。

![](https://pic.imgdb.cn/item/6132123344eaada739c8261a.jpg)

[更详细见 P280](https://www.bilibili.com/video/BV1Sy4y1C7ha)

## location 对象

### 什么是 location 对象

window 对象给我们提供了一个 location 属性用于<b style="color:red">获取或设置窗体的 URL</b>,并且可以用于<b style="color:red">解析 URL</b>。这个属性返回的是一个对象，所以这个属性也称为 location 对象。

#### URL

统一资源定位符（Uniform Resource Locator, URL）是互联网上标准资源的地址。互联网上每个文件都有一个唯一的 URL。

URL 的一般语法格式:

`protocal://host[:port]/path/[?query]#fragment`

样例：http://www.itcast.cn/index.html?name=andy&age=18#link

![](https://pic.imgdb.cn/item/61321aca44eaada739d92d9a.jpg)

### location 对象的属性

| location 对象属性                      |                   返回值                    |
| -------------------------------------- | :-----------------------------------------: |
| <b style="color:red">location.href</b> |             获取或设置整个 URL              |
| location.host                          |                  返回域名                   |
| location.post                          |                 返回端口号                  |
| location.pathname                      |                  返回路径                   |
| location.search                        |                  返回参数                   |
| location.hash                          | 返回片段，上图的 fragment 部分（#后面内容） |

### 案例

5 秒钟之后跳转页面

获取 URL 参数

### location 对象的方法

| location 对象方法  |           作用           |
| ------------------ | :----------------------: |
| location.assign()  |     跳转页面，可回退     |
| location.replace() |  替换当前页面，不能回退  |
| location.reload()  | 重新加载页面，即刷新页面 |

## navigator 对象

navigator 对象包含有关浏览器的信息，有很多属性，最常用的是 userAgent，作用是可以实现通过识别用户使用手机还是电脑打开页面，并跳转到对应的页面。

## history 对象

history 对象与浏览器历史记录进行交互。它包含用户在浏览器窗口中访问的 URL。

| history 对象方法   |                 作用                 |
| ------------------ | :----------------------------------: |
| history.back()     |               后退功能               |
| history.forward()  |               前进功能               |
| history.go(参数 n) | n>0,前进 n 个页面；n<0,后退 n 个页面 |

参考链接：
[pink 老师前端入门](https://www.bilibili.com/video/BV1Sy4y1C7ha)
