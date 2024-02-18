---
title: JavaScript冷门知识
categories: 前端
date: 2022-04-17 12:07:00
tags:
  - JavaScript

---

# JavaScript冷门知识

看红宝书，重新梳理JavaScript的知识。这部分**主要是梳理冷门的知识点**(对个人来说是冷门的)

## 组成

* ECMAScript(核心)：提供核心功能。每门语言的根本，大同小异，会有一些特殊的地方，比如JS有变量提升。
* DOM(文档对象模型)：提供与网页内容交互的方法和接口。主要就是操作DOM元素，包括样式修改、新增节点、删除节点等。
* BOM(浏览器对象模型)：提供与浏览器交互的方法和接口。比如` location`对象可以获取或设置窗口的URL等。



## script元素

首先，学习过JS的话，都知道` script`的使用方式有两种。

* 直接在`script`标签内写JS代码
* 在另一个文件中写JS代码，再引入



那么，如果两种方式一起用会怎样呢？

mytest.js

```js
console.log(111)
```



index.html

```html
<script src="./js/mytest.js">
  console.log(222)
</script>
```



![image-20220413211327017](https://s2.loli.net/2022/04/23/alOs8JV4jvNPgZF.png)



也就是说，如果两者一起使用的话，那么只有外部文件的代码会执行，会忽略行内代码。

也算可以看出是更推荐外部文件的做法。(个人想法)



使用外部文件有什么好处呢？

1. 可维护性。如果使用的是行内代码，且一个html文件中有很多业务逻辑的话，后期维护会很困难，首先找到问题代码都要花点时间。
2. 缓存。使用外部文件的话，如果两个页面用到同一个文件，该文件只需要下载一次，但是如果是行内代码，则会反复下载
3. 可复用性。可以把通用的代码抽离成一个文件，实现代码复用，而不需要有大量重复代码。



### 异步加载

首先，script如果没有` defer`和` async`，那么浏览器会立即加载并执行脚本，也就是说，会阻塞后面的文档元素的加载和渲染。

* 有` defer`属性的话，**会异步加载js文件**，即和加载渲染后续文档元素并行进行。**加载完成后并不一定是立即执行，而是要等到所有元素解析完成后(图片是在之后解析完成)，在` DOMContentLoaded`事件触发之前完成**

* 有` async`属性的话，**会异步加载js文件**。加载完成会立即执行，阻塞后面的文档元素的加载和渲染。所以不一定按顺序执行，谁先加载完成就谁先执行。

  

![异步加载](https://s2.loli.net/2022/04/23/KxbBzN6DQjgO5Ua.jpg)

图片来源：https://www.pianshen.com/article/2104972721/

由图总结：

* **`defer`和` async`都是异步加载**
* **` defer`：加载完成后，会等到所有元素都解析完成后才执行。使用` defer`的` js`代码会按顺序执行**
* **` async`：加载完成后，立即执行。使用` async`的js代码不一定会按顺序执行**



开始案例测试：下面最好在` network`面板中设置` Fast 3G`，让效果看得更明显。

```html
<body>
  <p>123</p>
  <img src="https://www.clzczh.top/medias/featureimages/17.png" alt="" style="width: 400px">
  <p>456</p>

  <script>

    window.onload = () => {
      console.log('load')
    }

    document.addEventListener('DOMContentLoaded', () => {
      console.log('DomContentLoaded')
    })
  </script>
</body>
```



什么都没加(在` head`内，js代码分别是弹出111、222、333)：

```html
<script src="./js/mytest1.js">
</script>
<script src="./js/mytest2.js">
</script>
<script src="./js/mytest3.js">
```

![js](https://s2.loli.net/2022/04/23/ntcbPHReAak7Vsr.gif)

可以发现，任何的元素都没有加载就开始执行js代码了，也就是说js加载会阻塞元素的加载和渲染



defer属性

```html
<script defer src="./js/mytest1.js">
</script>
<script defer src="./js/mytest2.js">
</script>
<script defer src="./js/mytest3.js">
```

![js](https://s2.loli.net/2022/04/23/n3Q7qyxltKcLJOv.gif)

先加载完所有的元素(不包括图片，图片会在` DOMContentLoaded`后加载)，然后才执行js代码，执行完js代码后触发` DOMContentLoaded`事件，然后再加载图片。**使用` defer`属性的` js`代码会按顺序执行**



async属性

```html
<script async src="./js/mytest1.js">
</script>
<script async src="./js/mytest2.js">
</script>
<script async src="./js/mytest3.js">
```

![js](https://s2.loli.net/2022/04/23/ItR8WFarXpSckiE.gif)

有点混乱，因为async是异步加载js，而且加载完就会阻塞并执行。**添加` async`属性的` js`代码不一定按顺序执行**(多刷新几次)

所以上面的图中是执行js代码前就执行完` DOMContentLoaded`事件了，然后在执行js的代码途中，加载出图片



除了使用` async`和` defer`属性来实现异步加载js外，也可以通过动态创建脚本的形式来实现。

```js
window.onload = () => {
  console.log('load')
}

document.addEventListener('DOMContentLoaded', () => {
  console.log('DomContentLoaded')
})

const fragment = document.createDocumentFragment()    // 这里使用一个小优化手段：先创建一个文档碎片，把需要新增的内容先添加到文档碎片上，最后才把文档碎片添加到真实DOM上。这样就能够减少操作DOM。

const myscript1 = document.createElement('script')
myscript1.src = './js/mytest1.js'
fragment.appendChild(myscript1)

const myscript2 = document.createElement('script')
myscript2.src = './js/mytest2.js'
fragment.appendChild(myscript2)

const myscript3 = document.createElement('script')
myscript3.src = './js/mytest3.js'
fragment.appendChild(myscript3)

document.head.appendChild(fragment)
```

效果就相当于添加了` async`属性。



最后再来一下结论：

* **`defer`和` async`都是异步加载**
* **` defer`：加载完成后，会等到所有元素都解析完成后才执行。使用` defer`的` js`代码会按顺序执行**
* **` async`：加载完成后，立即执行。使用` async`的js代码不一定会按顺序执行**



## 标签退出循环

说到退出循环的方法，常用的就是` break`和` continue`。` break`是退出整个循环，而` continute`是跳过当前那一个循环

```js
// 输出0、1
for (let i = 0; i < 5; i++) {
  if (i === 2) {
    break
  }
  console.log(i)
}

// 输出0、1、3、4
for (let i = 0; i < 5; i++) {
  if (i === 2) {
    continue
  }
  console.log(i)
}
```



那么嵌套循环，需要跳出两层的循环怎么办呢？

很明显，使用` break`只能跳出一层，而使用` continute`能跳出一层的一轮。



这个时候就到我们的标签退出循环法闪亮登场了。

其实用的也是` break`，只是我们使用` break`一般后面不带东西而已，也就是说没带标签则默认跳一层循环。所以，我们要跳两层循环的话，首先得在要使用的位置添加上标签，然后跳出循环时使用` break 标签`。

```js
let num = 0;
label:
for (let i = 0; i < 5; i++) {
  for (let j = 0; j < 5; j++) {
    if (i == 2 && j == 2) {
      break label;
    }
    num++;
  }
}
console.log(num); // 12
```

结果是12的原因：

* i=0时，执行5次
* i=1时，执行5次
* i=2时，执行2次



## with语句

作用：将代码作用域设置为特定的对象

```js
const person = {
  name: 'clz',
  age: 21,
  job: 'frontEnd'
}

with (person) {
  console.log(name)
  console.log(age)
  console.log(job)
}
```



## 对象常量`Object.freeze`

对象是引用类型，所以即使使用` const`来定义对象变量，它也是能够改变内容的，只是它的地址没有变化而已。

```js
const o = {
  name: 'clz'
}

o.name = 'czh'
console.log(o)    // {name: 'czh'}
```



通过` Object.freeze`冻结一个对象后，这个对象就不能再被修改，包括添加新属性、删除已有属性、修改属性等。

所以可以通过` Object.freeze`来实现对象常量。

```js
const o = {
  name: 'clz'
}

Object.freeze(o)

o.name = 'czh'
console.log(o)    // {name: 'clz'}
```



## 函数参数按值传递

这个一直都是这么听别人说的，但是没有证明过，其他语言倒是有看过证明，现在就来证明一波js版本的。

```js
function mytest(o) {
  o.age = 21
}

const person = {
  name: 'clz'
}

mytest(person)

console.log(person)  // {name: 'clz', age: 21}
```



可以发现，修改函数里的对象` o`会导致函数外的` person`也发生变化。

这是什么原因呢?可能的原因有两个：

1. 当函数参数是对象时，是按引用传递的
2. 函数参数是按值传递的，但是对象是引用类型。所以` o`还是会通过引用访问对象，那么函数内部给` o`添加` age `属性时，函数外部的对象也会反映这个变化。因为` o`指向的对象保存在全局作用域的堆内存中。



然后，先来说一下按值传递和按引用传递的概念。

* 按值传递：**值会被复制到函数内的局部变量**。也就是说，此时**直接给局部变量赋新值是不会修改到函数外的变量的**，但是，如果参数是对象，而且不是直接给局部变量赋新值，而是给它添加新属性，或者修改属性值的话，还是影响到函数外的变量的，因为**对象是引用类型的**。
* 按引用传递：**值在内存的位置会被保存到函数内的局部变量中**。也就是说，此时**直接给局部变量赋新值是会修改到函数外的变量的**，因为此时局部变量是地址，也就是说，直接给局部变量赋新值的话，就是将那个地址改变，既然修改的是地址，那么函数外部的变量肯定也会跟着变，因为它指向的地址都被别人改了。



下面对上面的例子进行修改，来证明对象也是按值传递的。

```js
function mytest(o) {
  o.age = 21

  o = new Object()
  o.age = 999
}

const person = {
  name: 'clz'
}

mytest(person)

console.log(person)  // {name: 'clz', age: 21}
```



如果` person`是按引用传递的，那么当我们将` o`重新定义为一个新对象时，` person`也应该会自动将指针改成指向` age`为999的对象。

但是最后的结果是` {name: 'clz', age: 21}`，也就是说，函数中参数的值改变之后，原始的引用还是没有改变。即**函数参数按值传递**



参考链接：

* JS红宝书

* [浅谈 defer 和 async 的区别](https://www.pianshen.com/article/2104972721/)

