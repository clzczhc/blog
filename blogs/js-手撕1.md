---
title: JS手撕(一)    类型判断、instanceof、数组去重
categories: 前端
date: 2022-08-23 10:30:40
tags:
    - JavaScript
    - 手撕
---

# JS手撕(一)    类型判断、instanceof、数组去重

## 前言

> 看这篇文章的小伙伴，建议看完每一节，都尝试自己手撕一遍，最好就是弄懂原理后再开始手撕(不要边看边手撕，会产生依赖)，本人就是看别人的文章后手撕的(因为手撕题型记不住，还有一些大佬有很有意思的解法)

## 类型判断

有用过JS一段时间的小伙伴应该对`typeof`比较属性，我们需要进行类型判断的时候一般都会先想到它。但是呢，它有一个很大的局限性，比如如果是`null`、`array`都会被认为是`object`，`array`是因为是一个特殊的对象，而`null`则是因为JS诞生以来`null`的实现导致的。

所以现在就来手撕一个类型判断函数。原理就是使用`Object.prototype.toString`来获取具体的类型。

```js
function myTypeof(arg) {
  console.log(Object.prototype.toString.call(arg))
}


myTypeof(123);      // [object Number]
myTypeof('Hello');  // [object String]
myTypeof([]);       // [object Array]
myTypeof({});       // [object Object]
myTypeof(null);     // [object Null]
```

接着只需要把得到的字符串`[object Type]`后面的具体类型取出来即可。

```js
function myTypeof(arg) {
  return Object.prototype.toString.call(arg).slice(8, -1).toLowerCase();
}


console.log(myTypeof(123));       // number
console.log(myTypeof('Hello'));   // string
console.log(myTypeof([]));        // array
console.log(myTypeof({}));        // object
console.log(myTypeof(null));      // null
```

## instanceof

上面已经有小小的提到的`typeof`，那么当然不能拉下它的好兄弟`instanceof`啦。

开始手撕之前，先复习一下`instanceof`究竟是啥。

MDN上的介绍已经很简单易懂了：`instanceof`运算符用于检测构造函数的`prototype`属性是否出现在某个实例对象的原型链上。原型和原型链可以看之前的详解原型链的文章。

所以只需要沿着原型链，依次判断实例的`_proto_`和构造函数的`prototype`是不是相同就行了。

```js
function myInstanceof(obj, func) {
  // obj: 要检测的对象，也可以是特殊的对象，如数组
  // func：要检测的构造函数

  if (typeof func !== 'function') {
    throw new Error('第二个参数应该是构造函数');
  }


  let proto = obj.__proto__;
  let prototype = func.prototype;

  while (true) {
    if (proto === null) {
      return false;
    }

    if (proto === prototype) {
      return true;
    }

    proto = proto.__proto__;
  }
}


console.log(myInstanceof([], Array));
console.log(myInstanceof([], Object));
```

上面用的是`__proto__`来获取实例对象的原型，但是实际上，**更推荐使用`Object.getPrototypeOf(obj)`来获取实例对象的原型**。

上面的代码还有有一些大问题，因为**`instanceof`是用来检测实例对象的**，所以我们还得去掉检测基本数据类型，如`123 instanceof Number;`得到的结果是`false`

```js
function myInstanceof(obj, func) {
  // obj: 要检测的对象，也可以是特殊的对象，如数组
  // func：要检测的构造函数

  if (!(obj && ['object', 'function'].includes(typeof obj))) {
    // 判断这里是参考前端胖头鱼的仓库里的，比较巧妙
    return false;
  }

  if (typeof func !== 'function') {
    throw new Error('第二个参数应该是构造函数');
  }


  let proto = Object.getPrototypeOf(obj);
  let prototype = func.prototype;

  while (true) {
    if (proto === null) {
      return false;
    }

    if (proto === prototype) {
      return true;
    }

    proto = Object.getPrototypeOf(proto);
  }
}


console.log(myInstanceof([], Array));     // true
console.log(myInstanceof([], Object));    // true

console.log(myInstanceof(null, Object));  // false
console.log(myInstanceof(123, Number));   // false
```

## 数组去重

### 双转换法

先转换成`Set`对象，再转成数组，因为`Set`对象的性质就是不重复。

转换成`Set`对象，就是通过`Set`的构造函数转换就行。不过将`Set`对象转成数组有两种方法：

1. 扩展运算符`...`

2. `Array.from()`

```js
function myUniqueArr(arr) {
  return Array.from(new Set(arr));
  // return [...new Set(arr)];
}

const arr = [1, 2, 2, 3, 4, 2, 1, 3, 6];
console.log(myUniqueArr(arr));    // [ 1, 2, 3, 4, 6 ]
```

### 新数组+遍历

原理就是遍历一遍原数组，如果不能在新数组中找到该元素，则新增到新数组里。因为一旦能找到，也就是说已经重复了，就不需要再添加到数组里了。

```js
function myUniqueArr(arr) {
  const result = [];

  arr.forEach(item => {
    if (!result.includes(item)) {
      result.push(item);
    }
  })

  return result;
}


const arr = [1, 2, 2, 3, 4, 2, 1, 3, 6];
console.log(myUniqueArr(arr));    // [ 1, 2, 3, 4, 6 ]
```

上面用的是`includes`来判断数组有没有包含元素，实际上也能够使用`indexOf`等不等于`-1`来判断。

### filter

使用`filter`来实现数组去重就有点有意思了。`filter`就是遍历一遍数组，只返回满足条件的。所以只需要判断当前索引是不是等于该元素在数组中的第一位置即可

```js
function myUniqueArr(arr) {
  return arr.filter((item, index) => index === arr.indexOf(item));
}


const arr = [1, 2, 2, 3, 4, 2, 1, 3, 6];
console.log(myUniqueArr(arr));    // [ 1, 2, 3, 4, 6 ]
```

### reduce

实际上使用`reduce`也是可行的。`reduce`的第一个参数是一个函数，第二个参数是初始值。

**该函数的第一个参数是上一个函数的返回值，第二个参数是当前处理的元素。**

通过`reduce`来实现数组去重就是设置初始值为`[]`，如果**累加的数组**中不包含当前处理的元素则添加到数组中去。然后返回**累加的数组**。

```js
function myUniqueArr(arr) {
  return arr.reduce((pre, cur) => {
    if (!pre.includes(cur)) {
      pre.push(cur);
    }

    return pre;
  }, [])
}


const arr = [1, 2, 2, 3, 4, 2, 1, 3, 6];
console.log(myUniqueArr(arr));    // [ 1, 2, 3, 4, 6 ]
```

## 参考

[死磕 36 个 JS 手写题（搞懂后，提升真的大） - 掘金](https://juejin.cn/post/6946022649768181774)

[GitHub - qianlongo/fe-handwriting: 手写各种js Promise、apply、call、bind、new、deepClone....](https://github.com/qianlongo/fe-handwriting)
