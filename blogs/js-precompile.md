---
title: 一道问题引起的重学预编译
categories: 前端
date: 2022-04-12 08:47:43
tags:
  - JavaScript
---

# 一道问题引起的重学预编译

> 前言：变量提升与函数提升本来是我个人觉得没必要写笔记来复习的知识。因为这部分看的面试题都能做对，就是说确实学的挺扎实的。直到遇到了下面这道题。

## 前情回顾

参加掘金日新计划时，在群里看到的问题(改造了下)

```js
var a;

if (true) {
  console.log(a)
  a = 111
  function a() { }
  a = 222
  console.log(a)
}

console.log(a)
```



## 基础知识回顾

### 变量提升

实际上，变量的提升其实算是JS的诟病了，所以es6出来了` let`和` const`之后，都是推荐使用` let`和` const`了。



在执行函数前，会先预编译，把<b style="color: red">用` var`声明的变量的声明提升到前面</b>。

首先，假如我们只有一个语句` console.log(a)`，这样子会直接报错` a is not defined`。

如果只声明变量，但是不赋值，则会得到` undefined`。

```js
var a;
console.log(a)
```



那么，如果先打印` a`，之后再定义` a`呢？

```js
console.log(a)
var a
```

首先呢？JS是单线程的，所以JS理论上是从上到下执行代码的，所以按理来说会报错` a is not defined`。

但是，实际上在执行代码前，会先进行一次预编译，把` var变量`的声明提升到前面。

所以上面的代码实际上也相当于

```js
var a
console.log(a)
```



<b style="color: red">变量提升只会把变量的声明提升到前面，赋值则不会提升到前面。</b>

```js
console.log(a)
var a = 123
console.log(a)
```

会先输出` undefined`，然后输出` 123`



预编译后的代码如下，

```js
var a
console.log(a)
a = 123
```



### 函数提升

<b style="color: red">函数声明整体提升，函数调用不提升</b>

```js
console.log(mytest)
console.log(111)

function mytest() {
  console.log(222)
}

console.log(mytest)
console.log(333)

mytest()
console.log(444)
```

预编译后：

```js
function mytest() {
    console.log(222)
}

console.log(test)
console.log(111)

console.log(mytest)
console.log(333)

mytest()
console.log(444)
```

![image-20220407234205587](https://s2.loli.net/2022/04/23/DJL2lFnNQvuPCez.png)



使用**使用变量声明函数**，则走的是变量提升路线，而不是函数声明路线

```js
console.log(mytest)   // undefined
mytest()    // mytest is not a function

var mytest = function () {
  console.log(123)
}
```



函数内部也会有变量提升，这时候会先预处理全局的，再预处理函数的，且函数内的变量、函数提升不能提升到函数外。

```js
function mytest1() {
  console.log(a)    // undefined
  b()   // 456

  var a = 123

  function b() {
    console.log(456)
  }
}

mytest1()
console.log(a)	//  a is not defined
```

预编译后的代码：

```js
function mytest1() {
    function b() {
        console.log(456)
    }
    var a
    
    console.log(a)
    b()
    
    a = 123
}

mytest1()
console.log(a)
```



**如果函数内部的变量没有定义，直接赋值，则会直接变成全局变量**(应该算是遗留bug，**不要这样用**)

```js
function mytest1() {
  b()   // 456

  a = 123

  function b() {
    console.log(456)
  }
}

mytest1()
console.log(a)    // 123
```



那么，是先变量提升，还是先函数提升呢？

有不同意见的欢迎评论。

<b style="color: red">从结果上看是函数优先，但从过程来看是变量优先</b>

###  预编译步骤

这是怎么回事呢？

#### 全局预编译

首先先来看一下全局预编译的3个步骤：

1. 创建` GO对象(Global Object)`
2. 找变量声明，将变量作为`GO属性`(在浏览器中的话，实际上就是挂载到` window`对象上)，值为` undefined`
3. 找函数声明，作为` GO属性`值为函数体

案例分析：

```js
console.log(111)
console.log(a)

function a() {
  console.log(222)
}

var a = 333

console.log(a)

function b() {
  console.log(444)
}

console.log(b)

function b() {
  console.log(555)
}

console.log(b)
```

1. 创建` GO对象`，找变量声明

   ```js
   GO: {
       a: undefined
   }
   ```

2. 找函数声明(会覆盖掉重名的)

   ```js
   GO: {
       a: function a() {
           console.log(222)
       },
       b: function b() {
           console.log(555)
       } 
   }
   ```

3. 全局预编译过程结束，开始真正的编译过程(把提升的给去掉先)

   ```js
   console.log(111)
   console.log(a)
   
   a = 333
   
   console.log(a)
   
   console.log(b)
   
   console.log(b)
   ```

4. 结合` GO对象`的属性

   ```js
   console.log(111)
   console.log(a)		// f a() { console.log(222) }
   
   a = 333
   
   console.log(a)	// 333
   
   console.log(b)	// f b() { console.log(555) }
   
   console.log(b)	// f b() { console.log(555) }
   ```



#### 局部(函数)预编译

> **GO对象是全局预编译，所以它优先于AO对象所创建和执行。**

首先先来看一下局部预编译的4个步骤：

1. 创建` AO对象(Activation Object)`
2. 找形参和变量声明，将变量和形参作为` AO属性`，值为` undefined`
3. 实参和形参统一(将实参的值赋值给形参)
4. 找函数声明，值赋予函数体

案例分析：

```js
function mytest(a, b) {
  console.log(a)
  console.log(b)
  console.log(c)

  var a = 111
  console.log(a)

  function a() {
    console.log(222)
  }
  console.log(a)

  function a() {
    console.log(333)
  }
  console.log(a)

  var b = 444
  console.log(b)
    
  var c = 555
  console.log(c)
}

mytest(123, 456)
```

1. 创建` AO对象`

2. 找形参和变量声明

   ```js
   AO: {
       a: undefined,
       b: undefined,
       c: undefined
   }
   ```

3. 实参与形参统一

   ```js
   AO: {
       a: 123,
       b: 456,
       c: undefined
   }
   ```

4. 找函数声明

   ```js
   AO: {
       a: function a() {
           console.log(333)
       },
       b: 456,
       c: undefined
   }
   ```

5. 局部预编译过程结束，开始真正的编译过程(把提升的给去掉先)

   ```js
   function mytest(a, b) {
     console.log(a)
     console.log(b)
     console.log(c)
   
     a = 111
     console.log(a)
   
     console.log(a)
   
     console.log(a)
   
     b = 444
     console.log(b)
       
     c = 555
     console.log(c)
   }
   
   mytest(123, 456)
   ```

6. 结合` AO对象`的属性

   ```js
   function mytest(a, b) {
     console.log(a)	// f a() { console.log(333) }
     console.log(b)	// 456
     console.log(c)	// undefined
   
     a = 111
     console.log(a)	// 111
   
     console.log(a)	/// 111
   
     console.log(a)	// 111
   
     b = 444
     console.log(b)	// 444
       
     c = 555
     console.log(c)	// 456
   }
   
   mytest(123, 456)
   ```



<b style="color: red">从结果上看是函数优先，但从过程来看是变量优先，因为变量提升后被之后的函数提升给覆盖掉了。</b>



## 回归正题

准备好基础知识后，自然就是**不忘初心，开始解决最开始的问题**

参考：[Function declaration in block moving temporary value outside of block?](https://stackoverflow.com/questions/58619924/function-declaration-in-block-moving-temporary-value-outside-of-block)

```js
var a;

if (true) {
  console.log(a)
  a = 111
  function a() { }
  a = 222
  console.log(a)
}

console.log(a)
```

分析：

1. 会有两个变量声明` a`，一个在块内，一个在块外

2. 函数声明被提升，并**被绑定到内部的块变量上**

   ```js
    var a¹;
    if (true) {
      function a²() {} 
      console.log(a²)
      a² = 111
      a² = 222
      console.log(a²)
   }
   console.log(a¹);
   ```

3. 这么一看，这不是和局部变量提升差不多。但是，<b style="color: red">当到达原来的函数声明处，会把块变量赋值给外部变量</b>

   ```js
    var a¹;
    if (true) {
      function a²() {} 
      console.log(a²)
      a² = 111
      a¹ = a²		// 当到达原来的函数声明处，会把块变量赋值给外部变量
      a² = 222
      console.log(a²)
   }
   console.log(a¹);
   ```

4. 之后，块变量和外部变量不再有联系，即块变量变化不会导致外部变量的变化。

5. 依次输出` f a() {}`、` 222`、` 111` 

为什么<b style="color: red">当到达原来的函数声明处，会把块变量赋值给外部变量</b>？

> the spec says so. I have no idea why. – [Jonas Wilms](https://stackoverflow.com/users/5260024/jonas-wilms)



<b style="color: red">不要用块级声明式函数</b>

<b style="color: red">不要用块级声明式函数</b>

<b style="color: red">不要用块级声明式函数</b>



```js
if (true) {
  function b() {
    console.log(111)
  }

  console.log(b)	// f b() { console.log(111) }
}


console.log(b)		// f b() { console.log(111) }
```



根据上面的分析：

```js
if (true) {
  function b²() {
    console.log(111)
  }
    
  b¹ = b²			// 没有定义，直接赋值，变为全局变量

  console.log(b²)	// f b() { console.log(111) }
}


console.log(b¹)		// f b() { console.log(111) }
```



我们把`if语句`的条件变为`false`后：

* ` if语句`的内容不再执行，合理
* 函数没有被提升到外面
  * 但是考虑到` if条件`为` false`的话，可能不会预编译内容
  * 但是外边的` b`却不是报错` b is not defined`，而是输出` undefined`

为什么？不知道，想不到原因，有人知道的话，评论告诉一下。(不会这样用，纯好奇为什么)



实际上，想要根据条件切换函数，可以用以下形式

```js
let fn

if (true) {
  fn = function () {
    console.log(111)
  }
} else {
  fn = function () {
    console.log(222)
  }
}

fn()
```

