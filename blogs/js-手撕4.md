---
title: JS手撕(四) call、apply、bind、柯里化、偏函数
categories: 前端
date: 2022-12-24 12:42:44
tags:
    - JavaScript
    - 手撕
---

# JS手撕(四)    call、apply、bind、柯里化、偏函数

## call

`call()`方法就是使用一个指定的`this`值和一个或多个参数来调用一个函数。

所以原理就是给传入的第一个参数添加临时方法，然后去调用这个临时方法，这个时候，这个方法的`this`就会指向第一个参数啦，最后当然还要临时方法删除掉，不留下痕迹。

```js
Function.prototype.myCall = function (context, ...args) {
  context = context || window;

  // 这里的this实际上就是要调用的函数
  context.fn = this;
  console.log(this);

  const result = context.fn(...args);

  delete context.fn;

  return result;
}
```

测试：

```js
function myTest(a, b, c) {
  console.log(this);
  console.log(a, b, c);
}

const obj = {
  name: 'clz',
  age: 21
}

myTest(1, 2, 3);

console.log('%c%s', 'font-size: 24px;color: red;', '==============')
myTest.myCall(obj, 3, 4, 5);
```

## apply

`apply()`方法和`call()`方法一样，都是**使用一个指定的`this`值**，不同的是：**`apply`的第二个参数是参数数组**。

所以和`call`非常像。

```js
Function.prototype.myApply = function (context, args) {
  context = context || window;

  context.fn = this;

  const result = context.fn(...args);

  delete context.fn;

  return result;
}
```

当然调用的时候还是有一点点不同的，第二个参数是参数数组，而不是像`call()`方法一样，是一个或多个参数。

## bind

`bind()`也能够像上面一样，使用指定的`this`值，不过有一个很大的不同，那就是`bind()`是返回一个函数，而不是立即调用。

顺带提一下一个注意点：调用`bind()`时，可以传参，调用该方法返回的函数也可以传参。所以会有两拨参数。

```js
function myTest(a, b, c, d, e, f) {
  console.log(this);
  console.log(a, b, c, d, e, f);

  console.log(arguments)
}

const obj = {
  name: 'clz',
  age: 21
}

const fn = myTest.bind(obj, 1, 2, 3);
fn(4, 5, 6);
```

上面`myTest`方法是需要六个参数的，通过`bind`调用会先把第一个参数之外的`1, 2, 3`作为`myTest`的前三个参数，然后真正调用时，又把`4, 5, 6`作为后三个参数。

手撕(简单版本)

```js
Function.prototype.myBind = function (context, ...args1) {
  context = context || window;

  context.fn = this;

  return function (...args2) {
    context.fn(...args1, ...args2);
    delete context.fn;
  }
}
```

如果是构造函数，会有点问题。

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const bindPerson = Person.myBind(null, 'tttt');

const person = new bindPerson(21);
console.log(person);    // {}
```

我们得到的是空对象，实际上，我们想要的是`Person {name: 'tttt', age: 21}`的结果，而且使用原生的`bind`是能正常得到结果的。

这是因为我们没有区分构造函数的情况。所以还得改造一下之前的方法。

```js
Function.prototype.myBind = function (context, ...args1) {
  context = context || window;

  context.fn = this;

  const _this = this;

  const result = function (...args2) {
    if (this instanceof _this) {
      // 2. 此时是构造函数，不应该通过`context.fn()`来调用
      this.fn = _this;
      this.fn(...args1, ...args2);
      delete this.fn;
    } else {
      // 3. 普通函数
      context.fn(...args1, ...args2);
      delete context.fn;
    }
  }

  // 1. 让返回的函数的原型对象继承构造函数的原型
  result.prototype = Object.create(this.prototype);
  return result;
}
```

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const bindPerson = Person.myBind({ name: 'czz' }, 'tttt');

const person = new bindPerson(21);
console.log(person);    // result {name: 'tttt', age: 21}
```

## curry(函数柯里化)

柯里化的意思就是将使用多个参数的函数转换成一系列使用一个参数的函数的技术。

即`add(a, b) ===> add(a)(b)`。

使用柯里化能够实现参数复用。

就拿上面的`add()`方法为例子，我们需要`5` + `[5, 7]`的结果，不能循环而是只能一行一行地写。

没有柯里化：

```js
add(5, 5);
add(5, 6);
add(5, 7);
```

使用柯里化之后：(效率提升了，能打更多螺丝了)

```js
const addCurry = curry(add);
const add5 = addCurry(5);

add(5);
add(6);
add(7);
```

具体实现就是返回一个函数，该函数会判断参数的数量有没有大于等于原函数的参数数量，如果等于直接返回结果，否则继续递归，直到数量大于等于原函数的参数数量。

```js
function curry(fn) {
  const result = function (...args1) {
    if (args1.length >= fn.length) {
      return fn.apply(this, args1)
    } else {
      return (...args2) => result(...args1, ...args2)
    }
  }

  return result;
}
```

测试：

```js
function add(a, b, c) {
  return a + b + c;
}

const addCurry = curry(add);

const add5 = addCurry(5);
console.log(add5(1)(6));    // 12: 5 + 1 + 6
console.log(add5(2)(7));    // 14
console.log(add5(3)(8));    // 16

console.log('%c%s', 'color: red;font-size: 20px;', '====================')

const add56 = add5(6);
console.log(add56(1));    // 12: 5 + 6 + 1
console.log(add56(2));    // 13
console.log(add56(3));    // 14
```

## 偏函数

yysy，在写这篇文章前，不知道这个这个点。

偏函数就是将一个`n参`的函数转换成`x参`的函数，剩余`n-x`参数将在下次调用全部传入。和柯里化非常像。

```js
function partial(fn, ...args1) {
  return function (...args2) {
    return fn(...args1, ...args2);
  }
}
```

测试：

```js
function add(a, b, c) {
  return a + b + c;
}

const addPartial = partial(add, 6);
console.log(addPartial(7, 8));    // 21 

console.log(addPartial(7));       // NaN, 不够参数不会像柯里化一样，返回一个函数，而是不够的参数直接把值当成`undefined`

addPartial(7)(8);                 // 报错，不能像柯里化一样(有点点像低配柯里化)
```

## 参考

[死磕 36 个 JS 手写题（搞懂后，提升真的大） - 掘金](https://juejin.cn/post/6946022649768181774)

[GitHub - qianlongo/fe-handwriting: 手写各种js Promise、apply、call、bind、new、deepClone....](https://github.com/qianlongo/fe-handwriting)

[JavaScript 专题之函数柯里化 - 掘金](https://juejin.cn/post/6844903490771222542)

[2021年前端各大公司都考了那些手写题(附带代码) - 掘金](https://juejin.cn/post/7033275515880341512)
