---
title: TypeScript查漏补缺(基础类型)
categories: 前端
date: 2022-06-18 12:50:53
tags:
  - TypeScript
---

# TypeScript查漏补缺(基础类型)

## 前言
TypeScript 入门教程看完了，大部分都按自己的理解来做了下笔记输出。但是，总感觉有遗漏的知识点。于是，找了一些大佬的博客，来查漏补缺一下。(但是这里只记录一下基本类型的，因为其他部分暂时看的还有点云里雾里)

## 基础类型
主要补充之前的笔记中没有的讲到的类型。

### unknown类型
**`unknown`类型是`any`类型对应的安全类型。**

**所有类型都可以赋值给`any`，也可以赋值给`unknown`**。
```ts
// unknown
let myunknown: unknown

myunknown = 123
myunknown = 'hello'

// any
let myany: any

myany = 123
myany = 'hello'

// number
let mynumber: number

mynumber = 123
mynumber = 'hello'  // 报错：不能将类型“string”分配给类型“number”。
```

**`any`类型能被赋值给任意类型(`any`、`unknown`、`number`等，`unknown`类型只能被赋值给`unknown`、`any`类型)**
```ts
// unknown
let myunknown: unknown

let value1: unknown = myunknown
let value2: any = myunknown
let value3: number = myunknown  // 报错：不能将类型“unknown”分配给类型“number”。


// any
let myany: any

let value4: unknown = myany
let value5: any = myany
let value6: number = myany 
```

**对`any`类型的值进行操作无需检查，`unknown`类型需要检查**
```ts
// unknown
let myunknown: unknown
console.log(myunknown.name) // 报错：类型“unknown”上不存在属性“name”。

// any
let myany: any
console.log(myany.name)

```


上面的例子简单讲一下本人的理解：任意类型`any`顾名思义，值可以是任意类型，也就包括是对象，而对象可能会有`name`，所以就不会报错。但是，`unknown`类型就会不知道该类型究竟存储了什么类型的值，虽然它也可能是对象，但是为了安全着想，会报错。**`unknown`类型是`any`类型对应的安全类型。**

### void类型
`void`类型表示没有任何类型。一般用来声明没有返回值的函数。(实际上，返回`undefined`和`null`也是可行的，`void`类型更像是不会返回有用的值)

```ts
function sayHello(): void {
    console.log('Hello')
}

sayHello()
```

但是，这里又有一个疑问：函数没有返回值时，默认返回`undefined`

那么，声明函数时的`void`类型和`undefined`类型有什么区别呢？

#### 返回值为`undefined`类型必须有返回值
虽然**函数没有返回值时，默认返回`undefined`**，但是当我们指定函数的返回值为`undefined`类型时，没有返回值会报错。

```ts
function sayHello(): void {
    console.log('Hello')
}

console.log(sayHello())


function sayHi(): undefined {
    console.log('Hi')
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0797eb0a70284717a9c5b18e6382eb8a~tplv-k3u1fbpfcp-zoom-1.image)


#### `undefined`能被赋值给`void`，但`void`不能赋值给`undefined`
`void`类型不能赋值给`undefined`这是符合正常的情况的：即只能赋值给自己和`any`类型
```ts
function sayHello(): void {
    console.log('Hello')
}

const s1: undefined = sayHello()    // 报错
const s2: void = sayHello()
const s3: null = sayHello()         // 报错
```
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181250687.png)

但是，有例外情况：`undefined`可以被赋值给`void`

```ts
const s1: undefined = undefined
const s2: void = undefined
```

顺带提一下：`null`和`undefined`的关系还是依然难解难分
```ts
const s1: null = undefined
const s2: undefined = null 
```

### never类型
`never`类型表示永不存在的值的类型。如抛出异常或不会有返回值的函数的返回值类型。

也就是说：如果看到`never`类型，很有可能是代码出问题了。

```ts
function errFunc(): never {
    throw new Error('clz')
}

function infiniteLoop(): never {
    while (true) { }
}
```

可以看下官方提示(VSCode翻译版)
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fb5d9ee945c47539792a6a404600b93~tplv-k3u1fbpfcp-zoom-1.image)


那么，`never`类型有什么用途呢？毕竟按上面的写法的话，就像是只能手动制造bug一样。
在TS中，**可以利用`never`类型来实现详细的检查**。

实例：
```ts
type Nickname = string | number

function checkNickname(nickname: Nickname) {
    if (typeof nickname === 'string') {
        console.log(`你的昵称是string类型${nickname}`)
    } else if (typeof nickname === 'number') {
        console.log(`你的昵称是number类型${nickname}`)
    } else {
        throw new Error('请检查类型')
    }
}

checkNickname('赤蓝紫')

checkNickname(123)

checkNickname(true)     
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c36989148cb444879fb566ee3fa465a9~tplv-k3u1fbpfcp-zoom-1.image)

从上面的例子中，可以看到`checkNickname`只是接受`string`和`number`类型，当我们传`boolean`类型的时候，会在编译期间报错。

但是，当同事修改`Nickname`的类型为`string | number | boolean`时，而且没修改`checkNickname`的逻辑的话，就会出问题。
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181250709.png)

可以发现：我们传参为`boolean`时，会在运行时抛出我们自定义的错误，但是再编译时没法检测出问题。这时候就能利用`never`来实现编译时就检测出问题。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c862f2c547394b80a592d252acc40f0b~tplv-k3u1fbpfcp-zoom-1.image)

上面的例子中，`else`分支的`nickname`会被收窄为`boolean`类型，而`boolean`类型无法被赋值给`never`类型，所以会出现编译错误，就能够提前检测出错误，避免很多没必要的问题。

**使用`never`类型能够避免新增联合类型，但是没有对应实现的情况**


参考链接：
* [一份不可多得的 TS 学习指南（1.8W字）](https://juejin.cn/post/6872111128135073806)
* [TypeScript never 类型](https://www.codercto.com/a/103516.html)
