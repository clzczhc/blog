---
title: 自定义工具函数库(一)  函数相关
date: 2022-03-19 11:40:10
categories: 前端
tags:
  - JavaScript
---

## 自定义工具函数库(一) 函数相关

最终仓库：[utils: 自定义工具库](https://github.com/13535944743/utils)

之前在哔哩哔哩看的视频的笔记。整理了一下。

### 1.1 call 函数封装实现

原理：为传入的 obj 添加临时方法，然后去调用这个临时方法，这样子，这个方法的`this`就会指向调用它的对象了，最后还需要把临时方法删除掉。

```js
// call函数封装实现
function call(fn, obj, ...args) {
  if (obj === undefined || obj === null) {
    // 如果call函数的第二个参数undefined(包括不传参)或null时，让obj等于全局对象
    obj = globalThis; // 浏览器下globalThis是window，而node环境下则是global
  }

  // 为obj添加临时方法
  obj.temp = fn;

  // 调用temp方法，此时方法中的this就是指向obj
  let result = obj.temp(...args);

  // 删除temp方法
  delete obj.temp;

  return result;
}
```

测试

```js
function add(a, b) {
  console.log(this);
  return a + b + this.c;
}

let obj = {
  c: 3,
};

// 添加全局属性
window.c = 100;

console.log(call(add, obj, 1, 2)); // 6：1 + 2 + obj.c，此时add函数中的this是obj
console.log(obj); // {c: 3}

console.log(call(add, null, 1, 2)); // 103：1 + 2 + window.c，此时add函数中的this是window
```

<br>

### 1.2 apply 函数

原理：和` call`函数一样，就只是第三个参数是数组，而不是多个参数而已，所以不需要使用扩展运算符` ...`

```js
// apply函数封装实现
function apply(fn, obj, args) {
  if (obj === undefined || obj === null) {
    obj = globalThis;
  }
  obj.temp = fn;

  let result = obj.temp(...args);

  delete obj.temp;

  return result;
}
```

<br>

### 1.3 bind 函数

需要依赖自定义 call 函数或内置 call 函数

这个函数功能和` call`函数一样，所以可以调用内置的` call`函数来实现，当然也可以调用自定义版本的。

不同的是，返回是一个函数，而不是立即调用。而且**在调用` bind`时可以传参，调用返回的函数也可以传参，只是如果传两次参数，则只有第一次的参数会起作用**

```js
// bind函数封装实现
function bind(fn, obj, ...args1) {
  return function (...args2) {
    return fn.call(obj, ...args1, ...args2); // 如果传两次参数，则只有第一次的参数会起作用。如果只传一次，则那一次的参数就会起作用
  };
}
```

测试用

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>函数封装实现</title>
    <script src="./bind.js"></script>
  </head>

  <body>
    <script>
      function add(a, b) {
        console.log(arguments);
        return a + b + this.c;
      }

      let obj = {
        c: 3,
      };

      // 添加全局属性
      window.c = 100;

      const fn1 = bind(add, obj, 3, 4);
      console.log(fn1());

      const fn2 = bind(add, obj, 3, 4);
      console.log(fn2(5, 6));

      const fn3 = bind(add, obj);
      console.log(fn3(5, 6));

      const fn4 = bind(add, null, 3, 4);
      console.log(fn4());

      // 内置版本
      // const fn1 = add.bind(obj, 3, 4)
      // console.log(fn1())

      // const fn2 = add.bind(obj, 3, 4)
      // console.log(fn2(5, 6))

      // const fn3 = add.bind(obj)
      // console.log(fn3(5, 6))

      // const fn4 = add.bind(null, 3, 4)
      // console.log(fn4())
    </script>
  </body>
</html>
```

<br>

### 1.4 函数节流和函数防抖

- 事件频繁触发可能造成问题
  - 一些浏览器事件如` window.onresize`、` window.mousedown`等，触发频率高，会造成界面卡顿
  - 向后台发送请求，频繁触发的话，对服务器会造成不必要的麻烦

解决方案：通过函数节流和函数防抖限制事件处理函数的频繁调用

<br>

#### 1.4.1 函数节流(throttle)

- **在函数需要频繁触发时：函数执行一次后，经过设定的间隔后才可以执行第二次。**

- 适合多次时间按时间平均分配触发

场景：

- resize 事件(窗口调整)
- scroll 事件(页面滚动)
- mousemove 事件(拖拽功能)
- click 事件(疯狂点击点击)

语法：` throttle(callback, wait)`

功能：创建一个节流函数，在 wait 毫秒内最多执行` callback`一次

实例：

```js
// 函数节流
function throttle(callback, wait) {
  // 定义开始时间
  let start = 0;

  // 返回结果是一个函数
  return function (event) {
    // 获取当前时间戳
    let now = Date.now();

    if (now - start >= wait) {
      callback.call(this, event); // 满足条件，执行回调函数

      // 修改开始时间
      start = now;
    }
  };
}

// // 之前青训营时，月影老师教的版本：通过定义一个计时器，当计时器到期时，清除之前的计时器，而清除计时器的时候才可以再次调用回调函数
// function throttle(fn, time = 500) {
//   let timer;
//   return function (...args) {
//     if (timer == null) {
//       fn.apply(this, args);
//       timer = setTimeout(() => {
//         timer = null;	/* 到期的话，清除之前的计时器 */
//       }, time)
//     }
//   }
// }
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>函数封装实现</title>
    <style>
      body {
        height: 2000px;
        background-color: skyblue;
      }
    </style>
    <script src="./throttle.js"></script>
  </head>

  <body>
    <script>
      // window.addEventListener('scroll', () => {
      //   console.log(this.scrollY) // 直接绑定滚动事件，一滚动，疯狂输出
      // })

      window.addEventListener(
        "scroll",
        throttle(function () {
          console.log(this.scrollY); // 成功实现节流
        }, 500)
      );
    </script>
  </body>
</html>
```

<br>

#### 1.4.2 函数防抖(debounce)

- 在函数需要频繁触发时：**在规定时间内，只让最后一次生效，前面的不生效**
- 适合多个事件一次相应的情况

场景：输入框实时搜索联想（keyup / input）

> 语法：` debounce(callback, wait)`
>
> 功能：创建一个防抖动函数，该函数会从上一次被触发后，延迟` wait`毫秒后调用` callback`
>
> 如果触发一次，还没过` wait`毫秒，再次触发，那么又得重新计时，依此类推，直到延迟` wait`毫秒后才调用` callback`(即<b style="color: red">频繁触发时，只让最后一次生效</b>)

实例：

```js
// 函数防抖
function debounce(callback, time) {
  let timer = null; // 定时器变量

  return function (e) {
    clearTimeout(timer); // 每一次新的触发都会把前一次的定时器给清除掉，直到没有新的触发且时间经过time毫秒后才调用callback

    // 启动计时器
    timer = setTimeout(() => {
      callback.call(this, e);
    }, time);
  };
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>函数封装实现</title>
    <script src="./debounce.js"></script>
  </head>

  <body>
    <input type="text" />
    <script>
      const input = document.querySelector("input");

      input.onkeydown = debounce((e) => {
        console.log(e.keyCode);
      }, 1000);
    </script>
  </body>
</html>
```

<br>

[尚硅谷 Web 前端自定义工具函数库视频教程\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1Cy4y117vt)
