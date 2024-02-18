---
title: 前端开发调试知识
date: 2022-01-18 15:56:14
keywords: 前端 调试 Debug 青训营
categories: "前端"
tags:
  - 青训营
  - 调试
---

# 前端开发调试知识

参加字节跳动的青训营时写的笔记。这部分是秃头披风侠老师讲的课。

## 1. 前端 Debug 特点

- **多平台**：浏览器、NodeJs、小程序
- **多环境**：本地开发环境、线上环境
- **多工具**：Chrome devTools、Whistle
- **多技巧**：Console、BreakPoint、sourceMap、代理

## 2. Chrome DevTools

### 2.1 动态修改元素和样式

- 点击.cls 开始动态修改元素的 class

- 输入类名即可给元素动态添加类名

  ![image-20220118144027415](https://s2.loli.net/2022/01/18/dMCIWHT1Q9ytfKr.png)

- 勾选/取消类名查看类名生效效果

- 点击 Styles 下具体的样式值，可以进行编辑，且可以在浏览器中实时预览

- Computed 下点击样式的箭头可以跳转到 Styles 下的 css 规则去

  ![image-20220118144618256](https://s2.loli.net/2022/01/18/5agmRKDvMzp3qYf.png)

- 强制激活伪类

  1. 选中具有伪类的元素，点击`:hov`

     ![image-20220118144806343](https://s2.loli.net/2022/01/18/98zMuRA3SJqk5Gf.png)

  2. DOM 树右键菜单，选择 Force State

     ![image-20220118144945504](https://s2.loli.net/2022/01/18/EzeyCUwJPk2dcs8.png)

### 2.2 Console

- console.log
- console.warn
- console.error
- console.debug
- console.info
- console.table：具象化地展示 JSON 和数组数据
- 占位符：用于给日志添加样式，可以突出重要的信息
  - %s：字符串占位符
  - %o：对象占位符
  - %c：样式占位符
  - %d：数字占位符

```js
console.log("log");
console.warn("warn");
console.error("error");
console.debug("debug");
console.info("info");

const peoples = [
  {
    name: "clz",
    age: 21,
  },
  {
    name: "czh",
    age: 22,
  },
];
console.table(peoples);

console.log("%s %c%s", "Hello", "font-size: 24px;color: red;", "World");
```

![image-20220118150413480](https://s2.loli.net/2022/01/18/OhTrEu2CmDBGIRb.png)

### 2.3 Sorce Tab

![image-20220118150447113](https://s2.loli.net/2022/01/18/r1DxKLAFanMEhcu.png)

- 源码中使用关键字 debugger 或代码预览区域点击行号设置断点

- 代码执行到断点处，代码暂停执行

- 展开 Breakpoints 列表可以查看断点列表，勾选可以激活对应断点

  ![image-20220118150828969](https://s2.loli.net/2022/01/18/fma5wDF6ZKcY3tW.png)

- 鼠标放在变量上可以查看变量的值

- 在调试器 Watch 可以添加监听变量

[**Scope**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)：展开 Scope 可以查看作用域列表(包含闭包)

[**Call Stack**](https://developer.mozilla.org/zh-CN/docs/Glossary/Call_stack)：展开 Call Stack 可以查看当前 JavaScript 代码的调用栈

**压缩后的代码调试**：通过 Source Map 映射源码实现调试，Source Map 文件不跟着部署上线，从而实现安全调试。

### 2.4 Performance

暂时没有用到过，先收藏一波图，方便以后重复查看。

![image-20220118151939298](https://s2.loli.net/2022/01/18/3z81cJrnHvgZFed.png)

### 2.5 NetWork

![image-20220118152125528](https://s2.loli.net/2022/01/18/9GTYrm2ZcJsHO8W.png)

### 2.6 Application

展示与本地存储相关的信息

- Local Storage
- Session Storage
- IndexedDB
- Web SQL
- Cookie

![image-20220118152311271](https://s2.loli.net/2022/01/18/Ot1AJq8SEgfnXQb.png)

点击 Stroage 面板下的 Clear Site Data 可以清除网页的本地存储数据

![image-20220118152702648](https://s2.loli.net/2022/01/18/widKTPfumOpWMNq.png)

### 2.7 线上即时修改

可以实现在浏览器中修改样式，并且刷新页面是，修改不会消失

1. 打开 Sources 面板下的 Overrides

   ![image-20220118154133335](https://s2.loli.net/2022/01/18/XkT39KcFSbs45f2.png)

2. 点击 Select folders...选择一个本地的空文件夹(可以新建)

3. 允许授权

4. 修改代码

5. 点击 DevTools 的三点->More tools->Changes，就能看到所有修改

   ![image-20220118154836504](https://s2.loli.net/2022/01/18/XczjCkP9QKmFYO6.png)

## 3. 利用代理解决跨域问题

原理：浏览器有同源策略策略的限制，会出现跨域问题。但是服务器之间不需要同源，所以，通过代理服务器接收浏览器的请求(代理服务器和浏览器同源)，代理服务器再转发请求给真实的服务器。

![image-20220118155230783](https://s2.loli.net/2022/01/18/lg9KPrT1JYVhIFO.png)
