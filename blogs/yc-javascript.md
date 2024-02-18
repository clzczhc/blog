---
title: JavaScript温故而知新
date: 2022-01-17 16:52:08
keywords: 前端 JS Javascript 青训营
categories: "前端"
tags:
  - 青训营
  - JavaScript
---

# JavaScript 温故而知新

参加字节跳动的青训营时写的笔记。这部分是月影老师讲的课。

## 1. 各司其责

- HTML/CSS/JS 各司其责

- **避免不必要的直接使用 JS 操作样式**(element.style.color="red")

- 使用 class 来表示状态

- 纯展示类交互寻求零 JS 方案(checkbox 的 checked 和 label，其中 checkbox 可以隐藏)

## 2. 组件封装

组件是指 Web 页面上抽出来一个个包含模板(HTML)、样式(CSS)、功能(JS)的单元

好的组件：封装性、正确性、扩展性、复用性

### 2.1 基本方法

- 结构设计
- 展现效果
- 行为设计
  - API(功能)
  - Event(控制流)

### 2.2 重构

- **插件化**

  - 将控制元素抽象成插件

  - 插件与组件之间通过<b style="color: red">依赖注入</b>的方式建立联系

- **模板化**
  - 将 HTML 模板化，更易于扩展(即视图根据数据来更新，这样子需要变更图片之类的时候，就只需要在图片数组中操作，而不需要变更 HTML)
- **抽象化**(组件框架)

例子：轮播图把上一页、下一页和分页按钮抽象成插件，就可以根据需要添加了

## 3. 过程抽象

- 用来处理局部细节控制的一些方法
- 函数式编程思想的基础应用

为了能够让**只执行一次**的需求覆盖不同的事件处理，我们可以将这个需求剥离出来，这个过程就叫做**过程抽象**

### 3.1 高阶函数

- 以函数作为参数
- 以函数作为返回值
- 常用于作为函数装饰器

```js
function test(fn) {
  return function (...args) {
    return fn.apply(this, args);
  };
} // 等价范式
```

常用高阶函数：

- Once

  ```js
  function once(fn) {
    return function (...args) {
      if (fn) {
        const ret = fn.apply(this, args);
        fn = null;
        return ret;
      }
    };
  }
  ```

- [Throttle](https://code.h5jun.com/gale/1/edit?js,output)

  ```js
  function throttle(fn, time = 500) {
    let timer;
    return function (...args) {
      if (timer == null) {
        fn.apply(this, args);
        timer = setTimeout(() => {
          timer = null; /* 到期的话，清除之前的计时器 */
        }, time);
      }
    };
  }
  ```

- [Debounce](https://code.h5jun.com/wik/edit?js,output)

- [Consumer / 2](https://code.h5jun.com/roka/7/edit?js,output)

- [Iterative](https://code.h5jun.com/kapef/edit?js,output)

#### 3.1.1 为什么使用高阶函数？

两种函数：纯函数和非纯函数

- 纯函数：输入的值一定时，输出的值一定，比较适合用于单元测试
- 非纯函数：会依赖于外部环境

通过高阶函数可以减少非纯函数的数量，增加系统的可靠性、稳定性。
