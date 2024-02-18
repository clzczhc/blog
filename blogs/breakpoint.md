---
title: 调试用到的几种断点
categories: 小技能
date: 2023-03-22 11:05:58
tags:
    - 调试
---

# 调试用到的几种断点

## VSCode

### 1. 条件断点

顾名思义，就是只有满足条件才会中断的断点。

#### 1.1 表达式断点

在表达式结果为真时中断。

简单使用：

```js
function add(a, b) {
  return a + b;
}

add(1, 2);
add(2, 3);
```

首先，添加普通的断点。

![](https://www.clzczh.top/CLZ_img/images/202303071625670.gif)

可以看到会断两次，但是如果添加的是条件断点的话，就可能不是断两次了。

![](https://www.clzczh.top/CLZ_img/images/202303071629260.png)

![](https://www.clzczh.top/CLZ_img/images/202303071631402.gif)

另外，VSCode的断点是即添（改）即用的，所以配合条件断点能干很多事情：

![](https://www.clzczh.top/CLZ_img/images/202303071700521.gif)

#### 1.2 命中次数中断

当命中次数满足条件才会中断。

\color{red}{不能只是输入一个数字，而应是`== 9`或`> 9`这种形式}

![](https://www.clzczh.top/CLZ_img/images/202303072020045.png)

![](https://www.clzczh.top/CLZ_img/images/202303072021856.gif)

### 2. 记录点

断点命中时记录的信息。直接输入的内容会当成字符串来处理，要输入表达式的话，需要用`{}`包住。

\color{red}{条件节点和记录点不能混合使用，混合使用，记录点会失效。}

![](https://www.clzczh.top/CLZ_img/images/202303071706953.png)

![](https://www.clzczh.top/CLZ_img/images/202303071707902.gif)

实际上，记录点和`console`效果基本一样。不过，记录点并不会污染代码。

### 3. 异常断点

出现异常后才会中断的断点。会分为捕获和未捕获两种。

![](https://www.clzczh.top/CLZ_img/images/202303072024862.png)

异常断点的好处自然就是能够知道出现异常时的一些变量信息、调用堆栈信息。

![](https://www.clzczh.top/CLZ_img/images/202303072032099.png)

### 4.内联断点

只有当执行到与内联断点关联的行时，才会命中内联断点。（不知道为什么网上都说是列）

把光标移动到要断的位置，然后点击`Shift + F9`。或者点击`运行>新建断点`。

内联断点比较适合调试一行中包含多个语句的代码，比如for循环，可以等到满足条件时，再进入循环体。这时候，调试自由度比条件断点要高一点点。

![](https://www.clzczh.top/CLZ_img/images/202303072130245.gif)

## Chrome

这部分介绍的是Chrome提供的一些断点。但是，也是可以通过VSCode去调试的，只不过需要在Chrome中设置断点。（下面为了方便录屏就不用VSCode来调试了）

### 1. 事件断点

添加事件断点后，当触发该事件时，就会中断。可以用于查看一下组件库触发事件后会进行哪些操作。

```html
<button onclick="handleClick()">click</button>

<script>
  function handleClick() {
    let a = 3;
    let b = 4;

    console.log(a + b);
  }
</script>
```

![](https://www.clzczh.top/CLZ_img/images/202303072153453.png)

### 2. DOM断点

DOM断点的设置并不是在Sources面板中，而是在Elements面板中选中DOM元素，右键，选择`Break on`设置，一共有三种类型。

![](https://www.clzczh.top/CLZ_img/images/202303072312532.png)

#### 2.1 `subtree modifications`（子树修改）

当前选择的节点的子节点被移除或添加，以及子节点的**内容**（不包括属性）更改时触发。

HTML：

```html
<div class="father">
  <div class="son">son</div>
</div>

<button onclick="handleRemove()">remove</button>
<button onclick="handleAdd()">add</button>
<button onclick="handleChange()">change</button>
```

CSS：

```html
<style>
  .father {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 200px;
    height: 200px;
    background-color: pink;
  }

  .son {
    width: 100px;
    height: 100px;
    background-color: purple;
    line-height: 100px;
    text-align: center;
    color: #fff;
  }
</style>
```

JavaScript：

```js
<script>
  const father = document.querySelector('.father');
  const son = document.querySelector('.son');

  function handleRemove() {
    father.removeChild(son);
  }

  function handleAdd() {
    father.appendChild(son);
  }

  function handleChange() {
    son.textContent = 'ccc';
  }
</script>
```

首先，设置好断点。

![](https://www.clzczh.top/CLZ_img/images/202303072336952.png)

接着，点击三个按钮的其中一个都会中断。

![](https://www.clzczh.top/CLZ_img/images/202303072338621.gif)

#### 2.2 `attribute modifications`（属性修改）

当前节点添加、删除、更改属性值时触发。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>

  <style>
    div {
      width: 100px;
      height: 100px;
      text-align: center;
      line-height: 100px;
    }

    .bg {
      background-color: pink;
    }
  </style>
</head>
<body>
  <div data-name="clz">
    clz
  </div>

  <button onclick="handleAdd()">add</button>
  <button onclick="handleRemove()">remove</button>
  <button onclick="handleChange()">change</button>

  <script>
    const div = document.querySelector('div');

    function handleAdd() {
      div.classList.add('bg');
    }

    function handleRemove() {
      div.classList.remove('bg');
    }

    function handleChange() {
      div.setAttribute('data-name', 'czh');
    }
  </script>
</body>
</html>
```

和子树修改基本一样操作。

#### 2.3 `node removal` （节点移除）

当前节点被移除时触发。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>

  <style>
    div {
      width: 100px;
      height: 100px;
      text-align: center;
      line-height: 100px;
    }
  </style>
</head>
<body>
  <div>
    clz
  </div>
  <button onclick="handleRemove()">remove</button>

  <script>
    const div = document.querySelector('div');

    function handleRemove() {
      div.parentElement.removeChild(div);
    }
  </script>
</body>
</html>
```

顺带一提：DOM断点能同时添加多种类型。

### 3. 请求断点

当发送请求的时候中断。如果不输入内容则是所有请求都中断，如果输入内容，则是当`url`中包含该内容的请求会中断。

> 请求断点不会考虑请求能不能发送到服务器。而是在发送请求的时候中断。

```js
fetch("http://localhost:8088/getInfo")
.then((res) => {
  console.log(res);
});

fetch('http://localhost:8088/postInfo', {
  method: 'POST'
})
.then((res) => {
  console.log(res);
})
```

所有请求都中断：

![](https://www.clzczh.top/CLZ_img/images/202303080014094.gif)

只有`url`包含给定内容的请求才会被中断：

![](https://www.clzczh.top/CLZ_img/images/202303080015089.gif)
