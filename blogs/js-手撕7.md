---
title: JS手撕(七) 事件总线
categories: 前端
date: 2022-12-24 12:45:13
tags:
    - JavaScript
    - 手撕
---

# JS手撕(七)    事件总线

## 事件总线

事件总线是什么呢？
事件总线其实就是发布订阅模式的一种实现。

学习JS的话，就一定会接触到事件的概念。比如给一个按钮绑定点击事件，绑定事件后，点击按钮会触发回调函数。

用发布订阅的说法来讲就是：给按钮绑定点击事件就是让按钮订阅点击事件，点击按钮就会发布事件，就会触发绑定事件时的回调函数。

### 实现

开始写之前，先需要分析一下解题思路，方便后面一马平川(假)。

基础结构：

```js
class EventBus {
  constructor() {
    this.callbacks = { };
  }
}
```

上面的`callbacks`就是用来存储所有的订阅事件的回调函数的。这里使用对象的形式而不是使用数组，是因为一个事件应该可以有多个回调，即该对象的键是事件名称，值是事件对应的回调函数数组。

#### 订阅事件

订阅事件实现原理就是：会先判断有没有该对象的回调。如果有就会通过`push`方法来添加新的回调，没有则赋值为数组再添加回调。

如果都直接使用`push`方法的话，因为第一次添加回调的时候，该事件还没有回调，所以此时的值是`undefined`，而不是数组，调用`push`方法的时候会报错。

如果都不使用`push`方法，而是直接赋值的话，就会导致一个事件只能有一个回调。

```js
on(eventName, callback) {
  if (!this.callbacks[eventName]) {
    // 如果是第一次给该事件添加回调，需要赋值为空数组
    this.callbacks[eventName] = [];
  }

  this.callbacks[eventName].push(callback);
}
```

```js
const eventbus = new EventBus();

eventbus.on('login', (name) => {
  console.log(`${name}登录了！`);
})

eventbus.on('login', (name) => {
  console.log(`bug，${name}连续登录了！`);
})

console.log(eventbus.callbacks);
```

#### 发布事件

发布事件就是取出对应事件的回调数组，然后依次执行回调。需要判断有没有回调函数，以及回调数组是不是空数组。

> 突然发现设计模式也不都是很难的(之前看设计模式时，疯狂催眠，完全看不进去，现在实操发现还行)

```js
emit(eventName, data) {
  const callbacks = this.callbacks[eventName];

  if (callbacks && callbacks.length > 0) {
    callbacks.forEach(callback => {
      callback(data);
    })
  } else {
    throw new Error('没有订阅该事件或事件回调为空');
  }
}
```

```js
const eventbus = new EventBus();

eventbus.on('login', (name) => {
  console.log(`${name}登录了！`);
})

eventbus.on('login', (name) => {
  console.log(`bug，${name}连续登录了！`);
})

eventbus.on('logout', (name) => {
  console.log(`${name}退出登录了`);
})

eventbus.emit('login', '赤蓝紫');

eventbus.emit('logout', '赤蓝紫');
eventbus.emit('logout', '赤蓝紫');

eventbus.emit('buy', '123');
```

#### 取消订阅

取消订阅分为两种情况：没有传，则清空所有的事件。传了事件名，删除该事件的全部回调。(不考虑传回调函数，清除指定事件回调的情况)

```js
off(eventName) {
  if (typeof eventName === 'undefined') {
    this.callbacks = {};
    return;
  }

  if (this.callbacks[eventName]) {
    delete this.callbacks[eventName];
  } else {
    throw new Error('没有订阅该事件');
  }
}
```

```js
const eventbus = new EventBus();

eventbus.on('login', (name) => {
  console.log(`${name}登录了！`);
})

eventbus.on('logout', (name) => {
  console.log(`${name}退出登录了！`)
});


eventbus.emit('login', '赤蓝紫');
eventbus.off('login');

// 取消`login`事件的订阅后，还能发布`logout`事件，但不能发布`login`事件
eventbus.emit('logout', '赤蓝紫');
eventbus.emit('login', '赤蓝紫');
```

### 完整代码

```js
class EventBus {
  constructor() {
    this.callbacks = {};
  }

  on(eventName, callback) {
    if (!this.callbacks[eventName]) {
      // 如果是第一次给该事件添加回调，需要赋值为空数组
      this.callbacks[eventName] = [];
    }

    this.callbacks[eventName].push(callback);
  }

  emit(eventName, data) {
    const callbacks = this.callbacks[eventName];

    if (callbacks && callbacks.length > 0) {
      callbacks.forEach(callback => {
        callback(data);
      })
    } else {
      throw new Error('没有订阅该事件或事件回调为空');
    }
  }

  off(eventName) {
    if (typeof eventName === 'undefined') {
      this.callbacks = {};
      return;
    }

    if (this.callbacks[eventName]) {
      delete this.callbacks[eventName];
    } else {
      throw new Error('没有订阅该事件');
    }
  }

}
```
