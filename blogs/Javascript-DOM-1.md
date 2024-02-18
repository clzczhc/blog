---
title: Javascript DOM(一)
date: 2021-08-26 19:08:11
categories: 前端
tags:
  - JavaScript
---

# JavaScript DOM（一）

整理一下学习的 DOM 部分知识，首先小复习一下知识点"预解析"。.

案例只留下案例名称，需复习的话，[下载素材](https://gitee.com/xiaoqiang001/java-script)，按名字搜索后可找到文件

## 预解析

运行 js 会分为两步。

1. 预解析
2. 代码执行

预解析：js 引擎会把 js 里面所有的 var 和 function 提升到当前作用域的最前面

预解析分为：

1. 变量预解析(变量提升）

把所有的变量声明提升到当前的最前面。<b style = "color:red;">只提升声明，不提升赋值，函数同理，不提升调用</b> 2. 函数预解析(函数提升）

把所有的函数声明提升到当前的最前面。

<p style="color:red">实际上，变量提升，可能会引发很多问题，会导致变量可以先使用后申明。函数提升暂时没有遇到什么问题。</p>

例子：

```javascript
var num = 10;
fn();

function fn() {
  console.log(num);
  var num = 20;
}

// 执行代码之前预解析，预解析后的代码如下所示，上面和下面的代码不同，但执行是一样的

var num;
function fn() {
  var num;
  console.log(num);
  num = 20;
}
num = 10;
fn();
```

## DOM 简介

文档对象模型(Document Object Model，简称 DOM), ，是 W3C 推荐的处理可扩展标记语言（HTML 和 XML）的标准编程接口。

DOM:对节点结构化表诉，并定义了一种方式可以使程序对该结构进行访问，将 web 页面和脚本语言连接起来。

通过 DOM 接口可以改变网页的内容、结构和样式。

![](https://pic.imgdb.cn/item/61162a995132923bf8ee7848.jpg)

## 获取元素

用 console.dir() 可以打印我们获取的元素对象，更好的查看对象里面的属性和方法

### 根据 id 获取

```javascript

    document.getElementById(id名字符串形式);
    document可以换成已经得到的元素，相应的获取的元素就只能是它的子元素

```

实例：

```html
<div id="my"></div>
<script>
  var my = document.getElementById("my");
  console.log(my);
</script>
```

### 根据标签名获取

`document.getElementsByTagName(标签名，字符串形式);`

得到的是一个对象的集合

### 通过 HTML5 新增的方法获取

#### 根据类名获取

`document.getElementsByClassName(类名，字符串形式);`

得到的是一个对象的集合

#### 根据选择器获取

1. document.querySelector('选择器')，返回第一个元素对象

例子：

```javascript
var id = document.querySelector("#id");
var tagName = document.querySelector("div");
var className = document.querySelector(".className");
```

2. document.querySelectorAll('选择器')，返回一个对象的集合

### 获取特殊元素(body, html)

```javascript
var body = document.body;
var html = document.documentElement;
```

## 事件基础

### 事件概述

事件是指可以被 Javas 侦测到的行为。例如，点击按钮，鼠标移动等。

### 事件三要素

1. 事件源
2. 事件类型
3. 事件处理程序

实例： 点击按钮弹出窗口

其中，事件源是按钮，事件类型则是点击，事件处理程序是弹出窗口

### 步骤

1. 获取事件源
2. 注册事件（绑定事件）
3. 添加事件处理程序

例子：

```javascript
var btn = document.querySelector("button");
btn.onclick = function () {
  alert("Hello!");
};
```

## 操作元素

### 改变元素内容

1. element.innerText

不识别 html 标签，空格和换行也会去掉 2. element.innerHTML

识别 html 标签，保留空格和换行

### 常见元素的属性操作

src、href、id、alt、title 等

element.src

### 表单元素的属性操作

type、value、checked、disabled、selected 等

element.value

#### 案例：

仿京东显示隐藏密码

![](https://pic.imgdb.cn/item/61163eee5132923bf814e9a5.jpg)

### 样式属性操作

#### element.style

1. 样式采用驼峰命名法，如 fontSize，backgroundColor;
2. 产生的是行内样式，CSS 权重比较高

```javascript
var div = document.querySelector("div");
div.onclick = function () {
  div.style.backgroundColor = "purple";
};
```

点击前：

![](https://pic.imgdb.cn/item/611638e75132923bf80a3008.jpg)

点击后：

![](https://pic.imgdb.cn/item/611639035132923bf80a66df.jpg)

##### 案例：

1. 关闭淘宝二维码案例

![](https://pic.imgdb.cn/item/61163f0d5132923bf8152175.jpg) 2. 循环精灵图

![](https://pic.imgdb.cn/item/61163f225132923bf8154542.jpg) 3. 显示隐藏文本框内容

![](https://pic.imgdb.cn/item/61163f355132923bf81566c6.jpg)

#### element.className

通过另外写 CSS，然后通过 className 来更改类名

1. 适合用于样式修改过多，通过行内样式操作会很复杂
2. class 是保留字，所以通过使用 className 来操作元素类名属性
3. 会直接更改元素的类名，即覆盖原来的类名。想要保留原来的类名的基础上改的话，则通过 element.className = '原来的类名 新的类名'来保留。

##### 案例：

仿新浪注册页面

![](https://pic.imgdb.cn/item/61163f595132923bf815ab5f.jpg)

![](https://pic.imgdb.cn/item/61163bc15132923bf80f520e.jpg)

### 排他思想

情境：

有一组元素，我们只想要一个元素实现某种样式。

方法：

1. 所有元素全部清除样式
2. 给当前元素设置样式

实例:

三个按钮，点击按钮，对应的按钮变色，其他的原来的默认色。相当于多选一。

```html
<body>
  <button>按钮1</button>
  <button>按钮2</button>
  <button>按钮3</button>
  <script>
    var btns = document.querySelectorAll("button");
    for (var i = 0; i < btns.length; i++) {
      btns[i].addEventListener("click", function () {
        for (var j = 0; j < btns.length; j++) {
          btns[j].style.backgroundColor = "";
        }
        this.style.backgroundColor = "pink";
      });
    }
  </script>
</body>
```

#### 案例：

1. 百度换肤效果
2. 表格隔行变色
3. 全选反选

![](https://pic.imgdb.cn/item/61163f9a5132923bf81623b1.jpg)

### 元素属性操作

#### 获取属性值

1. element.属性

<p style="color: red">只能获取内置属性值，无法获取自定义属性值，如index、data-index等，其中data-*是H5的自定义属性</p>

例子：

```javascript
var div = document.querySelector("#demo");
console.log(div.id);
```

2. element.getAttribute('属性');

可以获取自定义属性

例子：

```javascript
var div = document.querySelector("#demo");
console.log(div.getAttribute("id"));
console.log(div.getAttribute("index"));
```

#### 设置属性值

和获取一样，第一种方法也无法设置自定义属性值

1. element.属性 = '值'

例子：

```javascript
var div = document.querySelector("#demo");
div.id = "box";
```

2. element.setAttribute('属性', '值');

例子：

```javascript
var div = document.querySelector("#demo");
div.setAttribute("id", "box");
div.setAttribute("index", 2);
```

#### 移除属性

只有一种方法，element.属性 = ''；只能令属性值为空，而不会移除属性

element.removeAttribute('属性');

例子：

```javascript
var div = document.querySelector("#demo");
div.removeAttribute("id");
div.removeAttribute("index");
```

#### 案例

tab 栏切换

### H5 自定义属性

自定义属性目的：为了保存和使用数据。有些数据可以保存到页面中而不用保存到数据库中。<b style="color: red">未解：保存到数据库：怎么存？存在哪里怎么看？怎么用？</b>

由上面的元素属性操作可知，Attribute 系列函数(get、set、remove)可以对自定义属性进行操作。

出现问题：不容易判断是内置属性还是自定义属性

#### 设置 H5 自定义属性

H5 规定自定义属性 data-开头作为属性名

1. 直接在标签后给属性赋值

例子：

```html
<div data-index="1"></div>
```

2. element.setAttribute('属性', '值');

```javascript
var div = document.querySelector("#demo");
div.setAttribute("data-index", "2");
```

这里设置为数值时可以不用字符串形式

#### 获取 H5 自定义属性

1. element.getAttribute('属性');和上面用法一样，不同的只是自定义属性以 data-开头
2. H5 新增 element.dataset.'data-后面部分'或者
   element.dataset['data-后面部分'（字符串形式）]

示例：

```javascript
var div = document.querySelector("#demo");
console.log(div.dataset.index);
console.log(div.dataset["index"]);
```

dataset 这个方法有兼容性问题，只有 ie11 才开始支持。（不用 ie，现在一般也不去考虑兼容性，而且用 Attribute 系列就行了）

#### 移除 H5 自定义属性

element.removeAttribute('属性');用法类似

学习链接：[pink 老师前端入门](https://www.bilibili.com/video/BV1Sy4y1C7ha)
