---
title: MutationObserver接口-1-基本用法
categories: 前端
date: 2022-07-05 20:36:02
tags:
    - JavaScript
---



# MutationObserver接口(一)    基本用法

## 前言

> 看红宝书的时候学习到的一个新知识点，感觉很有意思。只能说是比较典型的观察者模式了(个人只是简单了解过一点点的设计模式，有误请评论)。

## 基本用法

使用`MutationObserver`可以观察整个文档、DOM树的一部分或某个元素。使用`MutationObserver`需要通过`MutationObserver`的构造函数实例化对象，参数是一个回调函数。

### observe()方法

实例化出一个`MutationObserver`对象之后，这个对象实际上就是一个观察者，但是，这个观察者这个时候还不知道自己要观察什么。这个时候需要调用`observer`方法来将它和DOM关联起来。此方法接收两个必须参数：**要观察其变化的DOM节点**、**一个用于控制观察哪些方面的配置对象**。

```js
const observer = new MutationObserver(() => {
  console.log('DOM元素有变化')
})

observer.observe(document.body, {
  attributes: true
})

document.body.className = 'foo'

console.log('script')        // 先打印script，再打印DOM元素有变化
```

上面设置了观察`body`元素的属性变化，所以修改`className`属性时会触发注册的回调函数，另外回调函数是异步执行的，所以会先打印script。

### MutationRecord实例

回调函数会接收一个`MutationRecord`实例的数组。`MutationRecord`实例会包含发生变化的信息，包括发生了什么变化，哪个地方发生了变化。

```js
const observer = new MutationObserver((mutationsRecord) => {
    console.log(mutationsRecord)
})

observer.observe(document.body, {
    attributes: true
})

document.body.setAttribute('name', 'clz')
```

![image-20220618130843972](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54566fa3baea453d9c846a62d1918532~tplv-k3u1fbpfcp-zoom-1.image)

连续修改会生成多个对应的 `MutationRecord`实例，下次回调执行时就会收到包含所有这些实例的数组，
顺序为变化事件发生的顺序：

```js
const observer = new MutationObserver((mutationsRecords) => {
    console.log(mutationsRecords)
})

observer.observe(document.body, {
    attributes: true
})

document.body.setAttribute('name', 'clz')
document.body.setAttribute('name', 'czh')
document.body.setAttribute('name', 'ccc')
```

![image-20220618133330300](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3073ec96fa4d484387390f12af1a2468~tplv-k3u1fbpfcp-zoom-1.image)

回调函数还可以接收第二个参数，就是`MutationObserver`的实例

```js
const observer = new MutationObserver((mutationRecords, mutationObserver) => {
  console.log(mutationRecords)
  console.log(mutationObserver === observer)        // true
});
observer.observe(document.body, { attributes: true });
document.body.setAttribute('name', 'clz')
```

### disconnect()方法

可以调用 `disconnect()`方法，来取消`observer`后续的观察，并且也会导致之前已经观察到，但是还没有执行毁掉的结果被抛弃。

```js
const observer = new MutationObserver((mutationRecords, mutationObserver) => {
  console.log(mutationRecords)
});


observer.observe(document.body, { attributes: true });

document.body.setAttribute('name', 'clz')

observer.disconnect()
document.body.setAttribute('name', 'czh')
```

上面的例子不会打印任何结果，这就是因为`disconnect()`不仅取消掉了`observer`后续的观察，还抛弃了之前已经观察到但还没执行回调的结果。

如果我们想`disconnect()`不影响之前已经观察道德结果的话，可以使用`setTimeout()`让之前的回调执行完毕再调用
`disconnect()`。因为`MutationObserver`的回调是微任务，而`setTimeout()`是宏任务，执行完一开始的同步任务后，会先执行微任务，再执行宏任务。

```js
const observer = new MutationObserver((mutationRecords, mutationObserver) => {
  console.log(mutationRecords)
});


observer.observe(document.body, { attributes: true });

document.body.setAttribute('name', 'clz')

setTimeout(() => {
    observer.disconnect()
    document.body.setAttribute('name', 'czh')
}, 0)
```

上面这个例子会打印只有一个`MutationRecord`实例的数组。

### 复用MutationObserver

如果我们想要观察多个节点，不需要新建很多个`MutationObserver`对象。只需要多次调用`observe()`方法，就能够复用一个`MutationObserver`对象观察不同的目标节点。还可以通过` MutationRecord`的`target`属性可以标识发生变化的目标节点。

```js
const observer = new MutationObserver((mutationRecords) => {
  console.log(mutationRecords)
  console.log(mutationRecords.map(x => x.target))
})

let childA = document.createElement('div')
let childB = document.createElement('span')

document.body.appendChild(childA)
document.body.appendChild(childB)

observer.observe(childA, { attributes: true })
observer.observe(childB, { attributes: true })

// observer.disconnect()    // 一刀切，会停用全部观察

childA.setAttribute('name', 'clz')
childB.setAttribute('age', 21)
```

![image-20220618212445463](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6962938b98bd45bab29e6ef70f02bf8c~tplv-k3u1fbpfcp-zoom-1.image)

### 重用 MutationObserver

上面我们有试过通过调用`disconnect()`方法来结束观察的，结束观察之后这个观察者不就没事干了吗？

为了不让这个观察者无所事事，可以重新使用它，让它观察新的目标节点(也可以是之前观察过的节点)，实际方法还是调用`observe()`方法。

```js
const observer = new MutationObserver(() => {
  console.log('change')
})


observer.observe(document.body, { attributes: true })

// 使用异步任务，防止disconnect影响到上面的观察
setTimeout(() => {
  observer.disconnect()    // 结束观察
  document.body.setAttribute('name', 'clz')       // 结束观察了，不会输出东西
})

setTimeout(() => {
  // 重用观察者
  observer.observe(document.body, { attributes: true })
  document.body.setAttribute('name', 'clz')
})


document.body.setAttribute('name', 'clz')
```

上面的例子会打印两次`change`。接下来就来分析一波。

* 首先，`observer.observe()`添加观察，之后遇到了两个定时器，因为是异步任务所以添加到任务队列中。也就是说此时不会结束观察，最后的属性设置就会**触发回调函数**。
* 同步任务执行完之后，就开始执行异步任务，第一个定时器就会结束观察了，所以之后的属性设置不会触发回调
* 但是，第二个定时器又重用该定时器，还是让它观察`body`，所以之后又生效了，**再次触发回调函数**
