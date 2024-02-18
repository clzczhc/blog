---
title: arrify转数组
categories: 前端
date: 2022-07-17 10:39:45
tags:
    - JavaScript
    - 源码共读
---

# 【源码共读】从arrify的源码中简单复习迭代器

**本文参加了由**[公众号@若川视野](https://link.juejin.cn/?target=https%3A%2F%2Flxchuan12.gitee.io "https://lxchuan12.gitee.io") **发起的每周源码共读活动，** [点击了解详情一起参与。](https://juejin.cn/post/7079706017579139102 "https://juejin.cn/post/7079706017579139102")

**这是源码共读的[第33期](https://juejin.cn/post/7100218384918249503)**

## 前言

github仓库地址 [arrify](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Farrify "https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsindresorhus%2Farrify")

这个`arrify`包的功能就是把一个值转化为数组形式。

## 开胃菜

```js
export default function arrify(value) {
    // 如果值是null或undefined，直接返回空数组
    if (value === null || value === undefined) {
        return [];
    }

    // 如果值本身就是数组，直接返回该值
    if (Array.isArray(value)) {
        return value;
    }

    if (typeof value === 'string') {
        return [value];
    }

    if (typeof value[Symbol.iterator] === 'function') {
        return [...value];
    }

    // 所有情况都不符合的话，直接返回数组的唯一元素就是该值本身的元素
    return [value];
}
```

上面的代码就是本次的源码，只有短短十几行，大概这就是“浓缩的都是精华”吧。上面已经简单地注释了一下。

但是，有两个部分没有任何注释，而这就是本文的核心所在。

先留点悬念：`value`的类型是`string`时也是返回`[value]`，最后返回的也是`[value]`，为什么不能共用最后的`return [value];`呢？(懂的大佬请让一让)

## 迭代器

上面为什么不共用最后的`return [value];`就是因为`String`内置了`Iterator`接口，也就是说此时`value[Symbol.iterator]`的类型就是函数，所以，如果共用，那么字符串就会走进去最后的`if`块里。

那么，这个`Iterator`接口究竟是什么东西呢？

这个就是迭代器接口，调用该接口，就会返回一个迭代器对象(也可称遍历器对象)。有该接口的数据结构都会有`Symbol.iterator`属性。

那么，接下来就来简单试验一下(玩一下)。

```js
const str = 'Hello';

const gen = str[Symbol.iterator]();

console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207022115118.png)

可以看到我们调用`Symbol.iterator`方法得到迭代器对象，调用`next()`方法就能遍历该字符串。同理，内置的其他具备`Iterator`接口的数据结构也能够通过该方法遍历。

如：

- Array
- Map
- Set
- String
- 函数的 arguments 对象
- NodeList 对象

我们也可以自己实现一个迭代器接口，从而能够自定义遍历过程。

```js
const obj = {
  count: 0,
  [Symbol.iterator]() {
    const self = this;

    return {
      next() {
        if (self.count < 5) {
          return { value: self.count++, done: false };
        } else {
          return { value: undefined, done: true };
        }
      }
    }
  }
};

let gen = obj[Symbol.iterator]();

console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207022152975.png)

### 

### 扩展运算符的作用

上面我们是通过调用迭代器接口，得到遍历器对象，然后调用遍历器对象的`next()`方法来实现遍历。所以如果我们想让有`Iterator`接口的数据接口转换成数组就很麻烦。

但是实际上我们使用扩展运算符就能很简单的将可迭代对象转换成数组。

```js
const str = 'Hello';
console.log([...str]);    // [ 'H', 'e', 'l', 'l', 'o' ]

const set = new Set([1, 2, 3, 2, 1, 4]);
console.log([...set]);    // [ 1, 2, 3, 4 ]
```

从上面的例子也能看出为什么将字符串和其他的可迭代对象分开。如果不分开，那么得到的数组将会是将字符串拆解后的数组，实际上意义并不大。如果想要这种效果，也可以单独使用扩展运算符处理，毕竟工具函数不可能考虑所有的使用情境。

## 

## 测试源码

```js
import arrify from "arrify";

console.log(arrify(undefined));
console.log(arrify(null));

console.log(arrify('Hello'));
console.log(arrify([1, 2, 3, 4]));
console.log(arrify(new Set([1, 2, 3, 2, 1, 4])));
console.log(arrify(new Map([
  ['name', 'clz'],
  ['age', '21']
])));
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207022212111.png)
