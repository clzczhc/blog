---
title: JS手撕(二) 数组扁平化、浅拷贝、深拷贝
categories: 前端
date: 2022-12-24 12:41:08
tags:
    - JavaScript
---

# JS手撕(二) 数组扁平化、浅拷贝、深拷贝

## 数组扁平化

数组扁平化就是将多层数组拍平成一层，如`[1, [2, [3, 4]]]`变成`[1, 2, 3, 4]`

可以使用递归来实现，就直接遍历最外层数组，如果遍历的元素是数组，那就继续递归，直到不是数组为止。

```js
function myFlatten(arr) {
  let result = [];

  for (let i = 0; i < arr.length; i++) {
    if (Array.isArray(arr[i])) {
      result = result.concat(myFlatten(arr[i]));
    } else {
      result.push(arr[i]);
    }
  }

  return result;
}


console.log(myFlatten([1, [2, [3, 4]]]));
```

简单画了一下递归的过程，包括`return`后回到上一级的部分。

也可以使用`some()`方法来更简单地实现，因为`some()`方法返回数组是否有元素满足条件的布尔值，因为可以将条件设置为数组中是否有元素是数组。

```js
function myFlatten(arr) {
  while (arr.some(item => Array.isArray(item))) {
    // 如果还有数组中还有数组，那就使用`concat`+扩展运算符来一层一层打平数组

    console.log(arr);

    arr = [].concat(...arr);
  }

  return arr;
}


console.log(myFlatten([1, [2, [3, 4]]]));
```

也可以使用`reduce`来实现。

```js
function myFlatten(arr) {
  return arr.reduce((pre, cur) => {
    return pre.concat(Array.isArray(cur) ? myFlatten(cur) : cur);
  }, [])
}


console.log(myFlatten([1, [2, [3, 4]]]));
```

更多方法可查看：[面试官连环追问：数组拍平（扁平化） flat 方法实现 - 掘金](https://juejin.cn/post/6844904025993773063)

大佬讲的非常细，循序渐进介绍了很多种方法。

## 拷贝

如果我们给把一个对象直接赋值给另一个对象，那么我们修改其中的一个对象都会影响到另一个对象(**非重新赋值**)，因为它们是同一个引用。

而拷贝的话，两个对象就不再是同一个引用了，所以修改对象不会影响到另一个对象。但是拷贝还分为浅拷贝和深拷贝两种。

### 浅拷贝

浅拷贝就是只能拷贝第一层，如果有嵌套对象，那么嵌套对象是没法拷贝的，所以修改嵌套对象还是会影响到另一个对象。而在后面讲的深拷贝则是即使有嵌套对象，也能够正常拷贝全部的方法。

> 下面的拷贝只是简单版本的，只考虑普通对象。主要学习思想(其实是懒)。

#### 遍历法

因为浅拷贝只需要拷贝第一层，所以只需要通过遍历，然后给新对象赋值旧对象的属性值即可，因为如果是只有一层的话，那么就不会是对象。如果是对象，即嵌套对象，那就不是浅拷贝能解决的了，而应该给后面的深拷贝来处理。

```js
function myShadowCopy(obj) {
  let newObj = {};

  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      // 因为for in遍历还能遍历到原型链上的不属于obj的属性
      newObj[key] = obj[key];
    }
  }

  return newObj;
}


const obj = {
  name: 'clz',
  job: {
    type: 'Coder'
  }
};

const shadowCopyObj = myShadowCopy(obj);

shadowCopyObj.name = 'czh';
console.log(obj);             // { name: 'clz', job: { type: 'Coder' } }
console.log(shadowCopyObj);   // { name: 'czh', job: { type: 'Coder' } }

// 浅拷贝：嵌套对象修改还是会影响到另一个对象
shadowCopyObj.job.type = 'tester';
console.log(obj);             // { name: 'clz', job: { type: 'tester' } }
console.log(shadowCopyObj);   // { name: 'czh', job: { type: 'tester' }
```

#### Object.assign()

可以使用`Object.assign()`来实现浅拷贝，因为`Object.assign()`的目的就是将一个或多个源对象复制给目标对象，并且返回修改后的对象。

```js
function myShadowCopy(target) {
  return Object.assign({}, target);
}


const obj = {
  name: 'clz',
  job: {
    type: 'Coder'
  }
};

const shadowCopyObj = myShadowCopy(obj);

shadowCopyObj.name = 'czh';
console.log(obj);             // { name: 'clz', job: { type: 'Coder' } }
console.log(shadowCopyObj);   // { name: 'czh', job: { type: 'Coder' } }

// 浅拷贝：嵌套对象修改还是会影响到另一个对象
shadowCopyObj.job.type = 'tester';
console.log(obj);             // { name: 'clz', job: { type: 'tester' } }
console.log(shadowCopyObj);   // { name: 'czh', job: { type: 'tester' } }
```

#### 扩展运算符`...`

```js
function myShadowCopy(target) {
  return { ...target };
}
```

效果和上面一样。

顺带一提：通过`concat`和`slice`可以浅拷贝数组。

### 深拷贝

浅拷贝只能拷贝对象的第一层，如果遇到嵌套对象，又会变成对象的引用。这时候就可以使用深拷贝，深拷贝就是拷贝整个对象，而不仅仅是第一层。

深拷贝主要是通过递归来实现，如果属性是对象，则递归调用深拷贝函数。

```js
const isObject = (target) => typeof target === 'object' && target !== 'null';

function myDeepCopy(obj) {
  let newObj = {};

  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
        newObj[key] = isObject(obj[key]) ? myDeepCopy(obj[key]) : obj[key];
    }  
  }

  return newObj;
}


const obj = {
  name: 'clz',
  job: {
    type: 'Coder'
  }
};

const deepCopyObj = myDeepCopy(obj);

deepCopyObj.name = 'czh';
console.log(obj);             // { name: 'clz', job: { type: 'Coder' } }
console.log(deepCopyObj);   // { name: 'czh', job: { type: 'Coder' } }

deepCopyObj.job.type = 'tester';
console.log(obj);             // { name: 'clz', job: { type: 'Coder' } }
console.log(deepCopyObj);   // { name: 'czh', job: { type: 'tester' }
```

上面的代码还有一个很大的问题，如果存在循环引用会报错。

循环引用就是上面的**`y`中有`z`，`z`中有`y`**，这种情况下会一直递归，直到超出最大调用*堆栈大小。

那么，如何解决这种情况呢？只需要使用`map`来缓存拷贝过的数据即可，键为拷贝的目标，值为拷贝的结果。先判断有没有拷贝过，如果有，直接返回之前拷贝过的数据。

```js
const isObject = (target) => typeof target === 'object' && target !== 'null';

function myDeepCopy(target, map = new Map()) {

  const cache = map.get(target)
  if (cache) {
    // 如果有缓存，即拷贝过，直接返回之前拷贝的结果
    return cache;
  }

  if (isObject) {
    let newObj = {};
    map.set(target, newObj);

    for (const key in target) {
      if (target.hasOwnProperty(key)) {
        newObj[key] = isObject(target[key]) ? myDeepCopy(target[key], map) : target[key];
      }
    }

    return newObj;

  } else {
    // 如果不是对象，直接返回target
    return target;
  }
}


const obj = {
  x: 'x',
  y: {
    name: 'clz'
  },
  z: {
    age: '21'
  }
};

obj.y.z = obj.z;
obj.z.y = obj.y;

console.log(obj);

const deepCopyObj = myDeepCopy(obj);

console.log(deepCopyObj);
```

### structuredClone()

顺带提一下这个方法，之前看到的。(node环境没有这个方法)

全局的 **`structuredClone()`** 方法使用**结构化克隆算法**将给定的值进行**深拷贝**

```js
const obj = {
  name: 'clz',
  job: {
    type: 'Coder'
  }
};

const deepCopyObj = structuredClone(obj);

deepCopyObj.name = 'czh';
console.log(obj);             // { name: 'clz', job: { type: 'Coder' } }
console.log(deepCopyObj);   // { name: 'czh', job: { type: 'Coder' } }

deepCopyObj.job.type = 'tester';
console.log(obj);             // { name: 'clz', job: { type: 'Coder' } }
console.log(deepCopyObj);   // { name: 'czh', job: { type: 'tester' }
```

## 参考

[死磕 36 个 JS 手写题（搞懂后，提升真的大） - 掘金](https://juejin.cn/post/6946022649768181774)

[GitHub - qianlongo/fe-handwriting: 手写各种js Promise、apply、call、bind、new、deepClone....](https://github.com/qianlongo/fe-handwriting)

[面试官连环追问：数组拍平（扁平化） flat 方法实现 - 掘金](https://juejin.cn/post/6844904025993773063)

[(建议精读)原生JS灵魂之问(中)，检验自己是否真的熟悉JavaScript？ - 掘金](https://juejin.cn/post/6844903986479251464)
