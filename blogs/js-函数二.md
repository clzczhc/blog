---
title: JavaScript之函数(二) 函数内部三个不常见的属性
categories: 前端
date: 2022-04-30 17:20:21
tags:
  - JavaScript
---

# JavaScript之函数(二) 函数内部三个不常见的属性
> 看红宝书+查资料，重新梳理JavaScript的知识。

## arguments.callee
`arguments`就不多说了，但是`arguments`有一个`callee`属性，是一个指向`arguments`对象所在函数的指针。

先来一下阶乘函数看看

```js
function factorial(num) {
    if (num <= 1) {
        return 1
    } else {
        return num * factorial(num - 1)
    }
}

console.log(factorial(4))       // 24
```

这样一看，是没什么问题的。

但是上面的函数必须要保证函数名是`factorial`，导致了紧密耦合。


```js
function factorial(num) {
    if (num <= 1) {
        return 1
    } else {
        return num * factorial(num - 1)
    }
}

const copyFactorial = factorial

console.log(copyFactorial(4))       // 24
```

还是没有问题？
这里没有问题其实就是因为虽然函数名变化了，但是，递归的时候用的函数还是之前的函数。所以如果，我们修改`factorial`就会引发问题了。

```js
function factorial(num) {
    if (num <= 1) {
        return 1
    } else {
        return num * factorial(num - 1)
    }
}

const copyFactorial = factorial

factorial = () => 0
console.log(copyFactorial(4))       // 0
```

这时候，我们可以使用`arguments.callee`来让函数逻辑和函数名解耦。这样子，无论函数叫什么名字，都能够正确的引用正确的函数。

```js
function factorial(num) {
    if (num <= 1) {
        return 1
    } else {
        return num * arguments.callee(num - 1)
    }
}

const copyFactorial = factorial

factorial = () => 0
console.log(copyFactorial(4))       // 24
```

## caller
函数对象会有一个属性`caller`，这个属性的值是**调用当前函数的函数**，如果是在全局作用域调用的话，则是`null`

```js
function outer() {
    console.log(outer.caller)
    inner()
}

function inner() {
    console.log(typeof inner.caller)
    console.log(inner.caller)
}

outer()
```

## new.target

`new.target`属性用于检测函数是否是使用`new`关键字调用。如果是使用`new`关键字调用的，则`new.target`就是被调用的构造函数；如果是作为普通函数调用，那么`new.target`的值就是`undefined`。

```js
function Person() {
    this.name = 'clz'
    this.age = 21
    console.log(new.target)
}
const person = new Person()


function sum(a, b) {
    console.log(new.target)
    return a + b;
}
sum(1, 2)
```

那么，这个属性有什么用呢？
我们的构造函数通过`new`关键字可以实例化一个新对象，也可以直接作为普通函数调用，虽然会有构造函数需要首字母为大写的不成文规定，但是开发时还是有可能会搞错的。(虽然类的话，我都是直接用的ES6的`class`了)

```js
function Person() {
    if (!new.target) {
        throw '这是构造函数!!!'
    }

    this.name = 'clz'
    this.age = 21
}

let person = new Person()
console.log(person)

person = Person()
console.log(person)
```

## 赠品：函数的`length`属性

函数的`length`属性指该函数期望传入的参数数量，即形参的个数。

```js
function fn1() { }
function fn2(a) { }
function fn3(a, b) { }

console.log(fn1.length)     // 0
console.log(fn2.length)     // 1
console.log(fn3.length)     // 2
```

问题：
```js
function fn1(a, b, c = 2, d) { }
function fn2(a, b, ...c) { }

console.log(fn1.length)     // 2
console.log(fn2.length)     // 2
```

这是怎么回事呢？
我们再重新看下它的定义：函数的`length`属性指该函数期望传入的参数数量，即形参的个数。

所以说，形参的数量是不包括剩余参数个数，只包括第一个具有默认值之前的参数个数。

因为剩余参数和默认值参数都是可传可不传的。至于默认值参数后面的非默认值参数`d`，应该算是被默认值参数连累了。不过，因为非默认值参数不会有顺序问题，即`d`不需要依赖`a`、`b`、`c`的值，所以只需要把它的定义往前移即可。

```js
function fn1(a, b, d, c = 2) { }
```

这样子，`fn1.length`就会变成3。
