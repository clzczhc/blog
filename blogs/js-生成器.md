---
title: JavaScript之生成器
categories: 前端
date: 2022-04-30 17:18:03
tags:
  - JavaScript
---

# JavaScript之生成器

> 看红宝书+查资料，重新梳理JavaScript的知识。

生成器是一个函数的形式，通过在函数名称前加一个星号(`*`)就表示它是一个生成器。所以**只要是可以定义函数的地方，就可以定义生成器**

```js
function* gFn() { }

const gFn = function* () { }

const o = {
    * gFn() { }
}
```

**箭头函数不能用来定义生成器函数**，因为生成器函数使用**` function*`语法**编写。

那为啥上面的第三个例子可以不使用` function*`语法呢？

因为那个是简写版本。等价于：

```js
const o = {
    gFn: function* () { }
}
```

## 生成器的简单使用

```js
function* gFn() {
    console.log(111)
}

gFn()
```

然后，我们会很"开心"地发现控制台没有打印任何信息。

这是因为调用生成器函数会产生一个生成器对象，但是这个**生成器一开始处于暂停执行的状态，需要调用` next`方法才能让生成器开始或恢复执行。**

**` return`会直接让生成器到达` done: true`状态**

```js
function* gFn() {
    console.log(111)
    return 222
}

const g = gFn()
console.log(g)

console.log(g.next())
console.log(g.next())
```

![image-20220416114721721](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b72657e50a89426795185b1f3592dab5~tplv-k3u1fbpfcp-zoom-1.image)

有种迭代器的既视感。实际上，**生成器实现了Iterable接口**，它们默认的迭代器是自引用的。

```js
function* gFn() {
    console.log(111)
    return 222
}

const g = gFn()

console.log(gFn()[Symbol.iterator]())   // gFn {<suspended>}
console.log(g === g[Symbol.iterator]()) // true
```

也就是说，**生成器其实就是一个特殊的迭代器**

## yield关键字

提到生成器，自然不能忘记` yield`关键字。` yield`能让生成器停止，此时**函数作用域的状态会被保留**，只能通过在生成器对象上调用` next`方法来恢复执行。

上面我们已经说了，**` return`会直接让生成器到达` done: true`状态**，而**` yield`则是让生成器到达` done: false`状态，并停止**

```js
function* gFn() {
  yield 111
  return 222
}

const g = gFn()

console.log(g.next())
console.log(g.next())
```

![image-20220416162345189](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51670b15ec7843c2a71907169d99732f~tplv-k3u1fbpfcp-zoom-1.image)

简单分析一下：

![image-20220416163137033](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4690e87666443d6bd55d7e5a68cccc8~tplv-k3u1fbpfcp-zoom-1.image)

另外，**` yield`关键字只能在生成器内部使用，用在其他地方会抛出错误。**

## 生成器拥有迭代器的特性

正如上面所说，生成器是特殊的迭代器，所以生成器也拥有迭代器的特性。

### 生成器对象之间互不干扰

```js
function* gFn() {
    yield 'red';
    yield 'blue';
    return 'purple';
}

const g1 = gFn()
const g2 = gFn()

console.log(g1.next())
console.log(g2.next())
```

![image-20220416164330795](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42dfb5b7ba6f4d4a862f849db050fedc~tplv-k3u1fbpfcp-zoom-1.image)

### 生成器对象可作为可迭代对象

```js
function* gFn() {
  yield 'red'
  yield 'blue'
  yield 'purple'
  return 'white'
}

const g = gFn()

for (const i of g) {
  console.log(i)    // 依次输出red、blue、purple
}
```

**` return`的内容不能被迭代。**

### 完成但并不完成

```js
function* gFn() {
    yield 'red';
    return 'blue';
    yield 'purple';
}


let g = gFn()
console.log(g.next())
console.log(g.next())
console.log(g.next())
console.log(g.next())
```

![image-20220416165321695](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e0eb27fc18b40a0989a4b5fb7484041~tplv-k3u1fbpfcp-zoom-1.image)

` return`会让当前这一次的` next`调用得到` value: return的数据, done: true`的结果，此时生成器已经完成了，但是还能继续调用` next`方法，只是返回的结果都会是` {value: undefined, done: true}`，即使后面还有` yield`关键字或` return`关键字。

## 使用yield实现输入和输出

```js
function* gFn(initial) {
  console.log(initial)
}

const g = gFn('red')
console.log(g.next())
```

![image-20220416231233648](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60fc092a07804f48afed7ac06fdad615~tplv-k3u1fbpfcp-zoom-1.image)

### yield关键字作为函数的中间参数使用

首先，生成器肯定是能够传参的，因为生成器是一个特殊的函数。只不过它不是一调用就执行的，它需要通过调用生成器对象的` next`方法才能开始执行。

那么，如果` next`方法传参应该怎么接收参数呢？

` yield`关键字会接收传给` next`方法的第一个值。

直接来实践比较好理解。

```js
function* gFn(initial) {
  console.log(initial)
  console.log(yield)
  console.log(yield)
  console.log(yield)
}

const g = gFn('red')
g.next('white')
g.next('blue')
g.next('purple')
```

![image-20220416232423112](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ec69ea4cf8843aca8eca14bce035cd0~tplv-k3u1fbpfcp-zoom-1.image)

1. 生成生成器，此时处于暂停执行的状态
2. 调用` next`，让生成器开始执行，输出` red`，然后准备输出` yield`，发现是` yield`，暂停执行，出去外面一下。
3. 外面给` next`方法传参` blue`，又恢复执行，然后之前暂停的地方(即` yield`)就会接收到` blue`。然后又遇到` yield`暂停。
4. 又恢复执行，输出` purple`

然后，可能就会有人问，第一次传的` white`怎么消失了?

它确实消失了，因为第一次调用` next`方法是为了开始执行生成器函数，而刚开始执行生成器函数时并没有` yield`接收参数，所以第一次调用` next`的值并不会被使用。

### yield关键字同时用于输入和输出

` yield`可以和` return`同时使用，同时用于输入和输出

```js
function* gFn() {
  yield 111
  return yield 222
}

const g = gFn()

console.log(g.next(333))
console.log(g.next(444))
console.log(g.next(555))
```

![image-20220417091733574](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f15b793457e4cfe897c43a8e2b88f6a~tplv-k3u1fbpfcp-zoom-1.image)

1. 生成生成器，此时处于暂停执行的状态
2. 调用` next`，让生成器开始执行，遇到` yield`，暂停执行，因为` yield`后面还有111，所以带着111作为输出出去外面。
3. 调用` next`，生成器恢复执行，遇到` return`，准备带着后面的数据跑路，结果发现后面是` yield`，所以又带着222，作为输出到外面。
4. 调用` next`，又又又恢复执行，不过这个时候`return`的内容是` yield`表达式，所以` yield`会作为输入接收` 555`，然后再把它带到外面去输出。

 **` yield`表达式需要计算要产生的值，如果后面没有值，那就默认是` undefined`**。` return yield x`的语法就是，遇到` yield`，先计算出要产生的值` 111`，在暂停执行的时候作为输出带出去，然后调用` next`方法时，` yield`又作为输入接收` next`方法的第一个参数。

## 产生可迭代对象

上面已经提过了，生成器是一个特殊的迭代器，那么它能怎样产生可迭代对象呢？

```js
function* gFn() {
  for (const x of [1, 2, 3]) {
    yield x
  }
}

const g = gFn()

for (const x of g) {
  console.log(x)    // 1、2、3
}
```

如上所示，利用` yield`能让生成器暂停执行的特性。

但是呢？我们还可以使用` *`来加强` yield`的行为，让它能迭代一个可迭代对象，从而一次产生一个值。这样子就能让我们能更简单的产生可迭代对象。

```js
function* gFn() {
  yield* [1, 2, 3]
}

const g = gFn()

for (const x of g) {
  console.log(x)    // 1、2、3
}
```

那么能不能不用` *`呢？

```js
function* gFn() {
  yield [1, 2, 3]
}

const g = gFn()

for (const x of g) {
  console.log(x)    // [1, 2, 3]
}
```

不用` *`的话，就会变成只有一个数组的迭代对象。

下面再来尝试一下` *`的用法

```js
function* innerGFn() {
  yield 111
  yield 222
  return 333
}

function* outerGFn() {
  console.log('iter value: ', yield* innerGFn())
}

for (const x of outerGFn()) {
  console.log('value: ', x)
}
```

![image-20220417094830971](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25a6fcf628504df8b1eff5c0acfdefd8~tplv-k3u1fbpfcp-zoom-1.image)

1. 上面有两个生成器函数，首先需要拿到` outerGFn`生成器产生的可迭代对象去迭代。
2. 然后发现` outerGFn`也需要拿到` innerGFn`产生的可迭代对象，去迭代，再产生一个给最外面迭代的可迭代对象
3. 所以最外面的迭代结果会是` 111`，` 222`，而` outerGFn`的输出则是` innerGFn`返回的值

### 利用` yield*`实现递归算法

利用` yield*`可以实现递归操作，此时生成器可以产生自己。

话不多说，开干

```js
function* nTimes(n) {
    if (n > 0) {
        // debugger
        yield* nTimes(n - 1)
        yield n - 1
    }
}

for (const x of nTimes(5)) {
    console.log(x)
}
```

![image-20220417100611148](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca9f42326d48480aabc48dbd1f2cca79~tplv-k3u1fbpfcp-zoom-1.image)

1. 首先，需要迭代`nTimes(5)`产生的可迭代对象
2. 进入到` nTimes`里，又需要一直` yield* nTimes(n-1)`，一直递归自己，到n===0为止
3. n===0，不满足条件不再继续递归，回退到上一层，此时n==1，执行` yield n - 1`
4. 依次回退到上一层，执行` yield n - 1`，最终` nTimes(5)`产生的可迭代对象内的值就是`0, 1, 2, 3 ,4`

## 提前终止生成器

生成器也能和迭代器一样提前终止，不过和迭代器的不太一样。

可以通过`return`和` throw`两种方法，提前终止生成器，都会强制生成器进入关闭状态(`generatorFn {<closed>}`)。

**一旦进入关闭状态，之后再调用next()都会显示` done: true`状态。**

### return

```js
function* gFn() {
  yield 111
  yield 222
  yield 333
}

const g = gFn()

console.log(g.next())

console.log(g.return(99))

console.log(g)

console.log(g.next(11))
console.log(g.next(22))
console.log(g.return(88))
```

![image-20220417102018496](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2f36b7b93654ff4a51bac86cfd616dd~tplv-k3u1fbpfcp-zoom-1.image)

当我们调用生成器的` return`方法时，生成器会进入关闭状态，后续再调用` next`方法，都会显示` done: true`状态，且` value`也只有在再次调用` return`的时候才能得到不是` undefined`的值。

`for-of循环`会忽略状态为` done: true`的，即如果提前终止生成器，那么实际上就相当于退出` for-of循环`了

```js
function* gFn() {
  yield 111
  yield 222
  yield 333
}

const g = gFn()

for (const x of g) {
  console.log(x)

  if (x === 222) {
    console.log(g.return(444))
    console.log(g)
  }
}
```

![image-20220417103100765](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57960e1b45264bad9eefcc28075a25a4~tplv-k3u1fbpfcp-zoom-1.image)

### throw

`throw`也可以提前终止生成器，且会抛出异常，需要捕获处理抛出的异常。

```js
function* gFn() {
  yield 111
  yield 222
  yield 333
}

const g = gFn()
// g.throw(444)       // 如果异常没有被处理的话，会直接报错

try {
  g.throw(444)
} catch (e) {
  console.log(e)
}

console.log(g)

console.log(g.next())
```

![image-20220417103544691](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8875d694b0be4e78b4ce589a897ff2a2~tplv-k3u1fbpfcp-zoom-1.image)



不过，如果处理异常是在生成器内部的话，情况就不太一样了。

```js
function* gFn() {
  for (const x of [1, 2, 3]) {
    try {
      yield x
    } catch (e) {
      console.log(e)
    }
  }
}

const g = gFn()

console.log(g.next())

g.throw(444)
console.log(g)

console.log(g.next())
```

![image-20220417104645347](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d62235b420442bab99c4ec11026878f~tplv-k3u1fbpfcp-zoom-1.image)

> 如果是在生成器内部处理这个错误，那么生成器不会关闭，还可以恢复执行，只是会跳过对应的yield，即会跳过一个值。



**throw语句会得到被跳过的那个值**

```js
console.log(g.throw(444))
```

![image-20220417104806188](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/880f12f0b6774089900d29a2e5f79658~tplv-k3u1fbpfcp-zoom-1.image)



如果我们处理异常是在生成器内部的话，还是会正常终止生成器的。

```js
function* gFn() {
  try {
    yield 1
    yield 2
    yield 3
  } catch (e) {
    console.log(e)
  }
}

const g = gFn()

console.log(g.next())

console.log(g.throw(444))
console.log(g)

console.log(g.next())
```

![image-20220417105316091](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11bcdf0fdc3841b798321a3fabd13a15~tplv-k3u1fbpfcp-zoom-1.image)



具体原因就得讲一下`next()`、`throw()`和`return()`的共同点了(本部分是在看阮一峰老师的ES6教程学到的)

### `next()`、`throw()`、`return()`的共同点

它们实际上都是干同样的事，就是让`Generator`函数恢复执行，并使用不同的语句替换`yield`表达式。



`next()`将`yield`表达式替换成一个值。

```js
function* g() {
  let result = yield 1 + 2;
  return result;
}


const gen = g();

console.log(gen.next());    // {value: 3, done: false}
console.log(gen.next(123)); // {value: 123, done: true}
```

上面的例子中，当我们第一次调用`next()`方法时，`Generator`函数开始执行，直到遇到第一个`yield`表达式为止。而第二个`next(123)`实际上就是把`yield`表达式替换成值`123`，如果`next()`方法没有参数，那么就替换成`undefined`。



`throw()`将`yield`表达式替换成一个`throw`语句。

```js
gen.throw(123);
// 相当于将 let result = yield 1 + 2;
// 替换成 let result = throw 123;
```

而这就是`为什么如果我们处理异常是在生成器内部的话，还是会正常终止生成器的`原因。因为抛出异常之后`try`之后的语句就不会再执行了。



`return()`将`yield`表达式替换成一个`return`语句。

```js
gen.return(123);
// 相当于将 let result = yield 1 + 2;
// 替换成 let result = return 123;
```





参考资料：

* 红宝书
* MDN
