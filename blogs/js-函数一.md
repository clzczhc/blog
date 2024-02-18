---
title: JavaScript之函数(一)
categories: 前端
date: 2022-04-30 17:19:33
tags:	
  - JavaScript
---

# JavaScript之函数(一)
> 看红宝书+查资料，重新梳理JavaScript的知识。

## 默认参数值

在ES6之前，我们想要实现默认参数的话，需要先检测某个参数是否等于`undefined`，如果是的话，证明此时并没有传这个参数，那就给它一个默认值。

```js
function mytest(name) {
    name = typeof name === 'undefined' ? 'clz' : name
    return name
}

console.log(mytest('赤蓝紫'))   // 赤蓝紫
console.log(mytest())       // clz
```


但是,ES6之后，就不需要这么麻烦了，ES6支持显示定义默认参数，只需要在函数定义的参数后面使用`=`就可以给参数设置一个默认值。另外，给参数传` undefined`相当于没有传值

```js
function mytest(name = 'clz', age = 21) {
    return `${name}的年龄是${age}`
}


console.log(mytest())   // clz的年龄是21
console.log(mytest('赤蓝紫'))   // 赤蓝紫的年龄是21
console.log(mytest(undefined, 999))   // clz的年龄是999
```

使用默认参数时，参数的默认值不会影响到`arguments`对象，只会影响函数的参数。
```js
function mytest(name = 'clz', age = 21) {
    console.log(name, age)
    return `${arguments[0]} ${arguments[1]}`
}


console.log(mytest())
console.log('%c%s', 'color: red;font-size:24px;', '===========================')

console.log(mytest('赤蓝紫'))
console.log('%c%s', 'color: red;font-size:24px;', '===========================')

console.log(mytest(undefined, 999))   
```


默认参数不仅可以是原始值、对象，还可能是JS表达式，如调用函数

```js
function sum(a, b) {
    return a + b
}

function mytest(name = 'clz', age = sum(10, 11)) {
    return `${name} ${age}`
}

console.log(mytest())
```

### 默认参数作用域与暂时性死区
给多个参数定义默认值跟使用`let`声明一样，所以后面定义默认值的参数能使用前面定义的参数。

```js
function mytest(name = 'clz', nickname = name) {
    console.log(name, nickname)
}


mytest()        // clz clz
```

既然和`let`声明一样，那么就也会遵循**暂时性死区**，在前面定义的参数不能使用后面定义的。

```js
function mytest(name = nickname, nickname = 'clz') {
    console.log(name, nickname)
}


mytest()        

// ReferenceError: Cannot access 'nickname' before initialization
```


**参数也有自己的作用域。不能使用函数体定义的变量**

```js
function mytest(name = nickname) {
    let nickname = 'clz'
}


mytest()
// nickname is not defined
```

为什么说参数拥有自己的作用域，而不是参数和函数体属于同一个作用域，只是因为参数使用了后面才定义的变量呢？

这个和我们上面说的**暂时性死区**就拖不开关系了。如果参数和函数体是同一个作用域，在预编译阶段，执行上下文就会认识函数体的变量了。所以报错消息不会是`nickname is not defined`，而应该是` Cannot access 'nickname' before initialization `

```js
function mytest() {
    let name = nickname
    let nickname = 'clz'
}


mytest()

// Cannot access 'nickname' before initialization
```

但是，如果默认参数没有使用的机会的话，即参数有值，那么就不会报错。

```js
function mytest(name = nickname) {
    let nickname = 'clz'

    return name
}


console.log(mytest('czh'))      // czh
console.log(mytest())       // nickname is not defined
```


### 扩展参数
首先，先来看一下代码
```js
function mymax() {
    let result = arguments[0]
    for (let i = 1; i < arguments.length; i++) {
        if (arguments[i] > result) {
            result = arguments[i]
        }
    }

    return result
}


console.log(mymax(1, 2, 3, 4))      // 4
console.log(mymax(4, 2, 3, 1, 1))   // 4
console.log(mymax(1, 2, 3, 88, 9))  // 88
```

上面的函数可以接收不限定个数的参数，并得到最大值。


那么，假如给你一个数组，让你调用上面的方法找出数组中的最大值呢？

在ES6之前呢，我们可以使用`apply`方法来实现
```js
let nums = [1, 2, 3, 88, 9]
console.log(mymax.apply(null, nums))
```

但是，在ES6中，我们可以通过扩展操作符`...`更方便地实现。对可迭代对象使用扩展操作符，可以将它作为一个参数传入，然后由扩展运算符将可迭代对象拆分，并将迭代返回的每个值单独传入。所以说，函数并不会知道扩展操作符的存在，只是会干好老本行(正常的按照调用函数时传入的参数接收每一个值)

```js
let nums = [1, 2, 3, 88, 9]
console.log(mymax(...nums)) // 88
```

使用扩展操作符传参时，并不会妨碍在前面或后面再传其他的值。

```js
let nums = [1, 2, 3, 88, 9]
console.log(mymax(...nums)) // 88
console.log(mymax(99, ...nums)) // 99
console.log(mymax(...nums, 111)) // 111
```

上面的例子中，扩展操作符和`arguments`一起使用，但这并不意味着使用扩展通识符需要配合`arguments`才能使用。

```js
function getSum(a, b, c = 100) {
    return a + b + c
}


console.log(getSum(...[1, 1]))      // 102
console.log(getSum(...[1, 1, 2]))       // 4
console.log(getSum(...[1, 1, 2, 99]))   // 4



let myset = new Set([1, 2, 3, 2])

console.log(myset)  // Set(3) { 1, 2, 3 }
console.log([...myset]) // [ 1, 2, 3 ]
```

### 剩余参数(收集参数)
在上面，我们使用扩展操作符可以将一个可迭代对象展开，但是呢？如果扩展操作符用在函数的参数中，还可以手机参数，把参数组合成数组。

```js
const mytest = (...nums) => {
    console.log(nums)
    return nums.reverse()
}

console.log(mytest(1, 2, 3))
console.log(mytest(4, 3, 2, 6, 9))
```

如果剩余参数的前面还有命名参数，那么就只会收集其余的参数。因为剩余参数的结果可变，所以只能把剩余参数作为最后一个参数。

```js
const mytest = (msg, ...nums) => {
    console.log(msg)
    console.log(nums)
}

mytest('消息', 4, 3, 2, 6, 9)
```

**剩余参数不会影响到`arguments`对象，仍然反映调用时传给函数的参数，另外箭头函数不支持`arguments`对象**

```js
function mytest(msg, ...nums) {
    console.log(msg)
    console.log(nums)
    console.log(arguments.length)
}


mytest('消息', 4, 3, 2, 6, 9)
```

### 函数作为值
可以将函数作为参数传给另一个函数，还可以在一个函数中返回另一个函数。

```js
function callSomeFn(fn, arg) {
    return fn(arg)
}

const result = callSomeFn(num => num ** 2, 9)
console.log(result)     // 81
```


```js
function createFn(propertyName) {
    return function (obj1, obj2) {
        return obj1[propertyName] - obj2[propertyName]
    }
}


const person1 = {
    name: 'clz',
    age: 62
}

const person2 = {
    name: 'czh',
    age: 21
}

const ageDifference = createFn('age')
console.log(ageDifference(person1, person2))    // 41
```
