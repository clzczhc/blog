---
title: JS手撕(五) new、Object.create()、Object.assign()
categories: 前端
date: 2022-12-24 12:43:58
tags:
    - JavaScript
    - 手撕
---

# JS手撕(五)    new、Object.create()、Object.assign()

## new关键字

实现`new`关键字，首先得了解一下`new`关键字究竟干了什么。

`new`关键字主要干了四件事：

1. 创建一个新对象

2. 设置该对象的原型为构造函数的原型（保留原有原型链）

3. 执行构造函数，`this`指向新对象

4. 如果构造函数返回值是对象，返回该对象。否则，返回`1`创建的对象

```js
function myNew(Func, ...args) {
  // 1. 创建一个新对象
  const obj = Object.create(null);

  // 2. 设置该对象的原型为构造函数的原型（保留原有原型链）
  Object.setPrototypeOf(obj, Func.prototype);

  // 3. 执行构造函数，`this`指向新对象
  const result = Func.apply(obj, args);

  // 如果构造函数返回值是对象，返回该对象。否则，返回`1`创建的对象
  return (typeof result === 'object' && result !== null)
    ? result
    : obj;
}
```

因为`Object.create()`可以使用现有的对象来作为新建对象的原型，所以第`1`、`2`步是可以合在一起的。

即：

```js
const obj = Object.create(Func.prototype);
```

测试：

```js
function Person(name, age) {
  this.name = name;
  this.age = age;

  // return { test: 'test' }
}

const person = myNew(Person, 'clz', 21);
console.log(person);                      // Person {name: 'clz', age: 21}
```

## Object.create()

`Object.create()`方法用于创建一个新对象，使用现有的对象来作为新建对象的原型。

实现一个低配版的，不考虑第二个参数。

核心就是一种实现继承的方法。(道格拉斯·克罗克福德在一篇文章中介绍的一种实现继承的方法)

```js
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
```

完整代码：

```js
function object(o) {
  function F() { }
  F.prototype = o;
  return new F();
}


Object.myCreate = function (proto) {
  const obj = object(proto);
  return obj;
}
```

测试：

```js
function Animal(type) {
  this.type = type;
}

const pig = Object.myCreate(Animal);
pig.type = 'pig';
console.log(pig);     // F {type: 'pig'}
```

还有一个问题：我们有时候会使用`Object.create(null)`创建一个没有原型的对象，但是现在是有问题的。

```js
const obj = Object.myCreate(null);
console.log(obj);     
```

所以还需要判断参数是`null`的时候，设置原型为`null`来实现能够创建一个没有原型的对象。

```js
Object.myCreate = function (proto) {
  const obj = object(proto);

  if (proto === null) {
    Object.setPrototypeOf(obj, null);
  }

  return obj;
}
```

## Object.assign()

`Object.assign()`将所有可枚举并且是自身属性从一个或多个源对象复制到目标对象，返回修改后的对象。

```js
Object.myAssign = function (target, ...sources) {
  sources.forEach(source => {
    for (const key in source) {
      // 可枚举

      if (source.hasOwnProperty(key)) {
        // 自身属性
        target[key] = source[key];
      }
    }
  })

  return target;
}
```

测试：

```js
const target = {
  name: 'clz'
};

const source1 = {
  name: '赤蓝紫'
};
const source2 = {
  age: 21
};
const source3 = {
  age: 999
};

const result = Object.myAssign(target, source1, source2, source3);
console.log(result);              // {name: '赤蓝紫', age: 999}
console.log(target === result);   // true 
```

## 参考

[GitHub - qianlongo/fe-handwriting: 手写各种js Promise、apply、call、bind、new、deepClone....](https://github.com/qianlongo/fe-handwriting)

[JavaScript之手撕new_战场小包的博客-CSDN博客](https://blog.csdn.net/qq_32036091/article/details/120608863)
