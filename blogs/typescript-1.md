---
title: 攀爬TS之路(一) 原始数据类型、任意值类型
categories: 前端
date: 2022-06-18 12:43:40
tags:
  - TypeScript
---

# 攀爬TS之路(一) 原始数据类型、任意值类型

## 前言
> 之前简单了解过TypeScript，但是没有系统、深入学习，现在就来系统学习一下。实际上，也算是必备知识了，印象最深的就是`Element-Plus`的示例代码都是TS了。

## 简介
TypeScript是JavaScript的超集(添加了**类型系统**)，**适用于任何规模**的项目。

TypeScript也可以编译为JavaScript：
1. `npm install -g typescript`全局安装TypeScript的命令行工具
2. `tsc hello.ts`编译TypeScript为JavaScript

### TypeScript是静态类型
静态类型：在编译阶段就能确定变量的类型，能**在编译阶段暴露大部分的错误**
动态类型：在运行时才会确定变量的类型，会导致更多错误(如类型匹配错误)

**TS是静态类型。JS是动态类型**

JS

```js
let num = 1
num.split('')
```
编译阶段不报错，运行时才发现`number`类型调用`split`，报错。

<br>

TS
```ts
let num = 1
num.split('')   // 类型“number”上不存在属性“split”。
```
上面这段代码在编译阶段就会报错，能够提前知道问题所在。

同样的代码在JS中运行阶段报错，在TS中编译阶段报错。
这是因为虽然我们没有声明num的类型，但是在变量初始化时，就已经自动推出它是`number`类型了，所以上面那一段代码等价于下面：
```ts
let num: number = 1
num.split('')   // 类型“number”上不存在属性“split”。
```

### TypeScript是弱类型
强类型：不允许隐式类型转换。
弱类型：允许隐式类型转换。如`1+'1'`不会报错

**TS和JS都是弱类型**


```ts
console.log(2 + '1')
```
在TS和JS中都不会报错，因为TS是完全兼容JS的，不会修改JS运行时的特性，所以它们都是弱类型。

## 原始数据类型
原始数据类型包括：`number`、`string`、`boolean`、`null`、`undefined`和`Symbol`、`BigInt`(ES6新增)

这个部分实际上，TS和JS差别不大，举个例子就能懂了。
```ts
let myNumber: number = 1
let myString: string = '赤蓝紫'
let myBoolean: boolean = false
let myNull: null = null
let myUndefined: undefined = undefined
let mySymbol: symbol = Symbol()

let myBigInt: bigint = 123n;

console.error(typeof myNumber)
console.error(typeof myString)
console.error(typeof myBoolean)
console.error(typeof myNull)
console.error(typeof myUndefined)
console.error(typeof mySymbol)
console.error(typeof myBigInt)
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c69d2a7a2f0a4d7585a511054ebfd654~tplv-k3u1fbpfcp-zoom-1.image)

**注意**：如果使用`bigint`类型时，可能会报错：`BigInt literals are not available when targeting lower than ES2020`

这时候需要在项目根目录下添加配置文件`tsconfig.json`添加`es`配置项，指定编译之后的版本目标
```json
{
    "compilerOptions": {
        "target": "ESNEXT"
    },
}
```

## 任意值类型
任意值类型(`any`)可以用来表示允许赋值为任意类型。但是，应该慎用，如果都是用`any`类型，那就是多此一举了，直接回到解放前了。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181243595.png)

允许在任意值上访问任何属性(有错误也是运行时才会报错)
```ts
let myNumber: any = 1
console.log(myNumber.a)     // undefined
console.log(myNumber.a.b)   // 运行时报错
```

也允许调用任何方法(有错误也是运行时才会报错)
```ts
let myNumber: any = 1
myNumber.sayHello()
myNumber.sayHello().sayHi()
```

因为**如果一个变量是任意值类型的话，那么对它的操作，返回的结果的类型都是任意值**，而且任意值也就意味着有可能会是对象，所以是没法在编译时暴露出错误的。

另外，除了上面声明时指定类型的情况，如果变量在声明时，没有指定它的类型，也没有被赋值，那么就会被识别成任意值类型。
```ts
let myAny

myAny = 1
myAny.sayHello()


// 等价于：
// let myAny: any

// myAny = 1
// myAny.sayHello()
```

**如果没有指定类型，但是在声明的同时赋值了，那就会按照类型推论的规则推断出一个类型**。
```ts
let myNum = 1
myNum = 'hello' // 编译时报错，因为类型推论的原因，myNum实际上已经被认为是`number`类型了

// 等价于
// let myNum: number = 1
// myNum = 'hello' // 编译时报错，因为类型推论的原因，myNum实际上已经被认为是`number`类型了
```
