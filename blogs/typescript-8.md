---
title: 攀爬TS之路(八)    泛型
categories: 前端
date: 2022-06-18 12:50:17
tags:
  - TypeScript
---

# 攀爬TS之路(八)    泛型

> 泛型是指在定义函数、接口或类时，不预先指定具体的类型，而是在使用的时候再指定类型的一种特性。

## 泛型的简单使用

先来一个简单的例子，加深了解。
目标：创建一个函数`createArr`，实现创建一个指定长度的数组。第一个参数是数组，第二个参数是数组每一项的值。

首先，我们想要实现这个功能，第一时间可能想到的是使用任意类型`any`来实现。
```ts
function createArr(length: number, value: any): any[] {
    const ret = []

    for (let i = 0; i < length; i++) {
        ret[i] = value
    }

    return ret
}


console.log(createArr(4, 1))
console.log(createArr(4, 'hello'))
```
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181250209.png)

结果是出来了，但TS还需要看下类型。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fc1ba0eee3f44fcb937573a76b650b4~tplv-k3u1fbpfcp-zoom-1.image)

不对劲的地方：
* 数组是`any`类型
* 数组的元素也都是`any`类型

但是，我们想要的效果应该是**无论传什么类型，就得到对应类型**。使用泛型就能很简单地实现这种效果。

使用起来也比较简单，在函数名后添加`<T>`，这个`T`就是表示输入的类型，之后就能把这个`T`当成类型来使用。
如：
```ts
function createArr<T>(length: number, value: T): T[] {
    const ret: T[] = []

    for (let i = 0; i < length; i++) {
        ret[i] = value
    }

    return ret
}
```

调用的时候，可以指定具体类型。也可以不手动指定，TS的类型推论会自动得到结果。
```ts
createArr<number>(4, 1)
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c56698cd5e8e4041add48f76befaa377~tplv-k3u1fbpfcp-zoom-1.image)

## 多个类型参数
定义泛型的时候，可以使用多个不同的字母来表示多个类型参数。

```ts
function createPerson<T, U>(name: T, age: U): { name: T, age: U } {
    const person: { name: T, age: U } = {
        name,
        age
    }

    return person;
}
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0702f8fefd7846c6a92101e8a6aedefc~tplv-k3u1fbpfcp-zoom-1.image)

## 泛型约束
在我们使用泛型变量的时候，因为不知道该变量是哪种类型(具体是哪种类型只有调用函数后才知道)，所以就不能操作它的属性和方法。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88ab8396acd54d4c8da48c43b8eb801e~tplv-k3u1fbpfcp-zoom-1.image)

这时候，可以对泛型进行约束，只允许有`length`属性的变量。具体使用就是定义一个接口`Lengthwise`,去限制泛型必须符合该接口的形状(即必须包含`length`属性)，然后通过`extends`来约束泛型`T`。

```ts
interface Lengthwise {
    length: number
}

function getLength<T extends Lengthwise>(arg: T): number {
    return arg.length;
}
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a43ddf6534554db081347ff686cd36b6~tplv-k3u1fbpfcp-zoom-1.image)

## 泛型接口
泛型也可以用来定义接口。
```ts
interface MyType<T> {
    data: T
}


const info1: MyType<number> = {
    data: 123
}

const info2: MyType<string> = {
    data: 'hello'
}
```

### 泛型类
泛型也可以用来定义类。
```ts
class Person<T> {
    private name: T;

    constructor(initValue: T) {
        this.name = initValue
    }

    getName(): T {
        return this.name
    }
}

let person1 = new Person(123)
let person2 = new Person('clz')

console.log(person1.getName(), person2.getName())
```

### 泛型数组
之前在数组一节中已经介绍了，现在再回顾一下。和泛型接口、泛型类的使用方法类似，不过并不需要提前使用`T`来定义，而是直接将`Array<type>`当成类型来使用即可。

```ts
let nums: Array<number> = [1, 2, 3]
```
