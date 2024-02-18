---
title: var、let和const之间的区别
categories: 前端
date: 2022-04-13 12:17:45
tags:
  - JavaScript
---

# var、let和const之间的区别

## 作用域不同

**` var`是函数作用域，` let`、`const`是块级作用域**

* 函数作用域就是在函数中声明了` var`变量，那么这个变量在整个函数里都是有效的。

* 块级作用域则是在块里是有效的。块级作用域就是用`{}`包住的区域，常用的有`for`，`while`，`if`等，只是有` {}`包住也是块级作用域

```js
{
  var a = 111
  let b = 222
  const c = 333

  console.log(a)  // 111
  console.log(b)// 222
  console.log(c)// 333
}

console.log(a)  // 111
// console.log(b) // b is not defined
console.log(c)  // c is not defined
```

## 有无变量提升

**` var`有变量提升，` let`和` const`没有变量提升**

即` let`和` const`不需要先声明，再使用，否则会报错，而` var`不需要先声明再使用，可以先使用后声明，不会报错，不过赋值的时候，值一直是` undefined`

```js
console.log(a)    // undefined
console.log(b)    // Cannot access 'b' before initialization
// console.log(c)  // Cannot access 'c' before initialization

var a = 111
let b = 222
// const c = 333
```

**注意，这里报的错不是` b is not defined`而是` Cannot access 'b' before initialization`**。也就是说：

* 从广义上来说，` let`和` const`没有变量提升，因为在声明前使用会报错
* 从狭义上来说，` let`和` const`是有变量提升的，因为实际上用它们定义的变量已经被执行上下文记住了，否则应该会报错` is not defined`才对。

**`let`和` const`究竟有没有变量提升取决于怎么定义变量提升**：

* 如果变量提升指的是变量可以在声明前使用，则没有变量提升
* 如果变量提升指的是变量在声明前有没有被执行上下文记住的话，则是有变量提升的。

## 能否被重新定义

`let`和` const`不能被重新声明，但是`var`可以被重新声明

```js
var a = 1
var a = 2
console.log(a)    // 2

// let b = 1
// let b = 2    // Identifier 'b' has already been declared

// const c = 1
// const c = 2     // Identifier 'b' has already been declared
```

在这个例子中，把下面的注释去掉后，就能够再次证明上面说的` let`、` const `有没有变量提升是取决于怎么定义的。

![image-20220411210448047](https://s2.loli.net/2022/04/23/n2KgUkhuYe1dEwJ.png)

因为JavaScript是单线程的，如果预编译执行上下文没有记住用` let`或` const`定义的变量的话，按理来说会先输出a，才报错，但是不是，也就是说，实际上执行上下文在预编译的时候，也已经记住用` let`和` const`声明的变量了。

**不能在函数内部重新声明参数**

```js
function mytest(args) {
    let args    // 报错
}
```

但是，如果是在函数里的另一个块级作用域里则可以重新声明参数

```js
function mytest(args) {
  {
    let args = 111;
    console.log(args)    // 111

    {
      let args = 222;
      console.log(args)    // 222
    }

  }
}

mytest(123)
```

## 暂时性死区

块中用` let`和` const`声明的变量，在使用` let`或` const`声明变量之前，该变量都是不可用的。这在语法上称为**暂时性死区**(temporal dead zone)

```csharp
var a = 111

{
  console.log(a)    // Cannot access 'a' before initialization
  let a = 222
}
```

上面的例子，就是典型的暂时性死区的例子。在上面有全局变量，在下面有块级变量，但是在中间则是谁都没法访问到。

## 全局作用域下是否会挂载到window对象

全局作用域下，使用` var`声明的变量会被挂载到` window`对象上，而使用` let`和` const 则不会`

```js
var a = 111
console.log(window.a)   // 111

let b = 222
console.log(window.b)   // undefined

const c = 333
console.log(window.c)   // undefined 
```

## `const`与`let`的区别

` const`与` var`的区别如上。` const`和` let`的区别就是**const声明的是常量，声明后不能够修改**

## 常见面试题

```js
for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i)    // 输出5，5次
  }, 0)
}
```

因为setTimeout是宏任务(JS执行机制可参考之前的文章)，所以执行输出语句时，for循环已经执行完成了，然后用` var`声明的变量是函数作用域，所以会打印出5个5。可以尝试在for循环外打印`i`，能得到` 5`的结果

把上面的` var`换成` let`后，会依次输出0、1、2、3、4

```js
for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i)    // 输出0、1、2、3、4
  }, 0)
}
```

因为` let`是块级作用域，用` let`声明的变量会被绑定到` setTimeout`上，所以并不会受到外界的影响，输出的结果就会是正常的0、1、2、3、4。

另外，for循环还有一个特别的地方：**设置循环变量的部分是一个父作用域，而循环体内是一个单独的子作用域**

```js
for (let i = 0; i < 1; i++) {
  let i = 222       // 这里不会报错，因为这里实际上是另一个子作用域
  console.log(i)    // 222
}
```

这里顺便再来回顾一下，暂时性死区。

```js
for (let i = 0; i < 1; i++) {
  console.log(i)        // Cannot access 'i' before initialization

  let i = 222      
  console.log(i)   
}
```

## 小贴士：`var`声明的变量不能被`delete`

```js
var a = 123;
let b = 456;
c = 789;

delete a;
delete b;
delete c;

console.log(a);
console.log(b);
console.log(c);
```

建议稍微思考下再看答案：

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208250944363.png)

解析：

上面的意思其实就是**只有`delete` c的时候成功了，其他时候都失败了。**

`delete`操作符用于删除对象的某个属性。直接使用`delete`变量实际上相当于删除全局对象上的属性。

所以使用`let`声明的变量就没法被`delete`，因为都没有被绑定到全局对象上。直接不用关键字声明的能够被`delete`是因为直接`c = 789;`其实就是普通的往全局对象里添加`c`属性。

问题来了：使用`var`声明的变量也会绑定到全局对象上，为什么它不能被`delete`掉呢？

> 是因为：使用`var`声明的变量，它的`configurable`是`false`，所以`window`对象上的`a`属性不能被设置（如被`delete`）。

```js
var a = 123;

console.log(Object.getOwnPropertyDescriptor(window, 'a'));  
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208251000767.png)

## 参考链接：

* [let 到底有无变量提升](https://blog.csdn.net/qq_58875046/article/details/123067627)
