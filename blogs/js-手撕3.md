---
title: JS手撕(三) 节流、防抖
categories: 前端
date: 2022-12-24 12:42:03
tags:
    - JavaScript
    - 手撕
---

# JS手撕(三)    节流、防抖

## 节流和防抖

前端开发中会遇到一些频繁的事件触发，像是`resize`、`scroll`、`mousedown`、`mousemove`、`keyup`、`keydown`等。

可能造成的问题：

- 触发频率高，造成界面卡顿

- 如果需要向后台发送请求，频繁触发的话，对服务器会造成不必要的麻烦

我们可以通过节流和防抖来限制函数的频繁调用。节流和防抖都是高阶函数，以函数为参数，以函数为返回值。

### 节流(`throttle`)

节流就是函数执行一次后，经过一定间隔后才能执行第二次。

实现思路：定义一个定时器，当定时器到点时，清除之前的计时器，清除定时器后才可以再次执行函数。

```js
function throttle(fn, time = 500) {
  let timer = null;

  return function (...args) {
    if (timer === null) {
      fn.apply(this, args);
      timer = setTimeout(() => {
        timer = null;
        // 这里不能使用`clearTimeout`来清除，因为`clearTimeout`清除后，timer不为null
      }, time);
    }
  }
}
```

实际测试：

未节流版本：

```html
<body>
  <button>点击</button>

  <script>
    document.getElementsByTagName('button')[0].addEventListener('click', handleClick);

    function handleClick() {
      console.log('click');
    }
  </script>
</body>
```

节流版本：

```js
document.getElementsByTagName('button')[0].addEventListener('click', throttle(handleClick, 1000));

function handleClick() {
   console.log('click');
}
```

对比可以发现：没有节流的话，每点击一下，都会触发事件处理函数。添加了节流之后，点击之后1s内，没法再次触发事件处理函数。1s之后才能重新触发。

### 防抖(`debounce`)

防抖就是在规定时间内，只让最后一次生效，前面的不生效。

所以**简单来说的话**，节流和防抖的区别就是：节流是第一次有效，防抖是最后一次有效。

实现原理也和节流很像：定义一个定时器，返回一个函数，每次执行返回的函数都会先清除定时器，然后设置定时器，该定时器的回调就是执行传入的函数。先清除定时器就是为了实现让最后一次生效，前面的不生效的关键。

```js
function debounce(fn, time = 500) {
  let timer;

  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, time)
  }
}
```

实际测试：

未防抖：

```html
<body>
  <input type="text">

  <script>
    document.getElementsByTagName('input')[0].addEventListener('input', handleInput);

    function handleInput() {
      console.log('input');
    }
  </script>
</body>
```

防抖版本：

```js
document.getElementsByTagName('input')[0].addEventListener('input', debounce(handleInput, 1000));

function handleInput() {
  console.log('input');
}
```

对比可以发现：没有防抖的，只要输入都会触发事件处理函数，而有防抖的在连续输入的时候(间隔小于1s)，是不会触发事件处理函数的，只有当1s内都没有新的输入才会触发事件处理函数。

## 参考

[死磕 36 个 JS 手写题（搞懂后，提升真的大） - 掘金](https://juejin.cn/post/6946022649768181774)

[GitHub - qianlongo/fe-handwriting: 手写各种js Promise、apply、call、bind、new、deepClone....](https://github.com/qianlongo/fe-handwriting)
