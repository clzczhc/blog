---
title: 攀爬TS之路(五)    类型断言
categories: 前端
date: 2022-06-18 12:47:05
tags:
  - TypeScript
---

# 攀爬TS之路(五)    类型断言

## 类型断言
第二段路时，已经提到联合类型：变量**只能访问**联合类型中**所有类型共有的属性或方法**

语法：`值 as 类型` 或 `<类型>值`

### 用途

#### 将联合类型断言成其中的具体类型

```ts
interface IFishman {
    // 摸鱼人
    name: string,
    play(): void
}


interface IWorker {
    // 干饭人
    name: string
    eat(): void
}


function myFunc(person: IFishman | IWorker) {
    console.log(person.name)

    person.play()       // 报错：类型“IFishman | IWorker”上不存在属性“play”。类型“IWorker”上不存在属性“play”。
}
```

但是，有时候我们就是需要访问非公有的属性或方法。比如上面的例子中，当是`Fishman`时，调用`play`方法，当是`Worker`时，调用`eat`方法。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/313cd5a1ed8b49a9b428997246575b46~tplv-k3u1fbpfcp-zoom-1.image)

这时候，断言就能用来**将联合类型断言成其中的具体类型**。

```ts
interface IFishman {
    // 摸鱼人
    name: string,
    play(): void
}


interface IWorker {
    // 干饭人
    name: string,
    eat(): void
}


function myFunc(person: IFishman | IWorker) {
    (person as IFishman).play()       // 将person断言成IFishman
}


const fishman: IFishman = {
    name: 'clz',
    play: function () {
        console.log('摸鱼')
    }
}

myFunc(fishman)     // 摸鱼
```

乍一看，挺好的，但是，实际上只是隐藏其他情况而已，比如上面`person as IFishman`隐藏了`person`为`IWorker`的情况，这时候如果传入的参数是`IWorker`类型的，那就会报错，而且**没法在编译阶段就暴露错误**

```ts
const worker: IWorker = {
    name: 'clz',
    eat: function () {
        console.log('干饭')
    }
}

myFunc(worker)
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16642fe94541457f90d20db6ae9d1bf5~tplv-k3u1fbpfcp-zoom-1.image)

所以，使用断言时，应该非常注意，不然会增加一些运行时错误

#### 将父类断言成更具体的子类
更准确来说，是将父类型断言成更具体的子类型，因为类的话，使用`instanceof`来判断就足够了。

但是，如果我们使用接口的话，它并不是类，而是类型，自然就不能使用`instanceof`来判断，这时候就需要使用断言来将父类型断言成更具体的子类型(实际上和**将联合类型断言成其中的具体类型**很像)

```ts
class Person {
    name: string
}

interface IFishman extends Person {
    // 摸鱼人
    play(): void
}

interface IWorker extends Person {
    // 干饭人
    eat(): void
}

function myFunc(person: Person) {
    // console.log(person instanceof IFishman)      // 会报错：“IFishman”仅表示类型，但在此处却作为值使用。

    if (typeof (person as IFishman).play === 'function') {
        return '摸鱼人'
    } else if (typeof (person as IWorker).eat === 'function') {
        return '干饭人'
    }

}

const worker: IWorker = {
    name: 'clz',
    eat: function () { }
}
console.log(myFunc(worker))     // 干饭人
```

#### 将任何一个类型断言成`any`
我们使用JS进行开发时，有时候可以在`window`对象上添加新的属性，这个属性就能够全局访问了，但是，在TS中是会报错的，因为`window`对象没有该属性，就会报错。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0e176c060ae4d7ea889cd7eba5b337e~tplv-k3u1fbpfcp-zoom-1.image)

但是，这个做法实际上在开发中能够很便利，这个时候可以使用断言将它断言成`any`类型，这样子就能够添加新属性了。

```ts
(window as any).a = 123
```

需要注意的是，**这样可能会掩盖真正的类型错误**

#### 将`any`断言成具体类型
设想一个情境，一个获取两个参数的和的函数，返回值按理应该是`number`类型，但是，结果却是`any`类型，这样子就会导致很多能在编译阶段暴露出的错误没法暴露出来。(这个情境是随便想的，简单来讲就是，历史代码不太好动，可能会引发蝴蝶效应)

```ts
function mySum(a: number, b: number): any {
    return a + b
}

const sum = mySum(9, 8)
console.log(sum.length)
```
比如上面，我们应该在访问`sum.length`时报错才对，但是因为是任意类型，所以不会报错，所以这时候就可能使用断言，将`any`断言成具体的类型，恢复它的在编译阶段报错的功能。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18a71f0462f142f88a5a683cfa6477e4~tplv-k3u1fbpfcp-zoom-1.image)

### 断言规则
**如果A兼容B或者B兼容A，那么A能够被断言为B**。
这里的兼容简单来说就是：A兼容B就是指类型A是类型B的子集。

```ts
class Person {
    name: string
}

interface IFishman extends Person {
    // 摸鱼人
    play(): void
}

function testPerson(person: Person) {
    return (person as IFishman)
}

function testIFishman(fishman: IFishman) {
    return (fishman as Person)
}
```

上面的例子中，类型`Person`是类型`IFishman`的子集，即`Person`兼容`IFishman`，所以`Person`能被断言为`IFishman`，`IFishman`也能被断言为`Person`。

上面使用的`Person`是类，而`IFishman`是继承了`Person`，所以可能会误以为是继承关系导致的能否被断言。实际上，**断言并不是根据是否有继承关系，而是看有没有兼容关系**。所以下面的做法也是可以的。
```ts
interface IPerson {
    name: string
}

interface IFishman {
    // 摸鱼人
    name: string,
    play(): void
}

function testPerson(person: IPerson) {
    return (person as IFishman)
}

function testIFishman(fishman: IFishman) {
    return (fishman as IPerson)
}
```

如果类型A不兼容B，并且类型B不兼容A，那么A不能断言为B，B也不能断言为A。
比如`number`和`string`。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a43796ef61134bf5897196ddd92c7b21~tplv-k3u1fbpfcp-zoom-1.image)


### 禁术：双重断言
* 任何类型都可以被断言成`any`
* `any`可以被断言成任何类型

所以，可以使用禁术**双重断言**把任何一个类型断言成任何另一个类型。

```ts
function mytest(num: number) {
    return (num as any as string)
}
```

> **除非迫不得已，千万别用双重断言**。禁术：伤敌一千，自损八百

### 类型断言不会进行类型转换
**类型断言只在TS编译时有效果**，在编译结果中会被删除，不会影响到编译结果的类型。
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181247912.png)

这个例子中，乍一看，断言还同时实现了类型的转换。

但是，都是假象。编译结果，立马打回原形。
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181247813.png)

所以，需要进行类型转换还是得老老实实直接调用类型转换的方法。
```ts
function mytest(num: number) {
    return String(num)
}

const num: number = 1
const str = mytest(num)


console.log(str)
console.log(typeof str)
```

### 类型断言VS类型声明
```ts
function mytest(num: number): any {
    return num
}

const num = mytest(123) as number
console.log(typeof num)
```

我们可以使用断言将`any`类型断言为`number`类型。

当然我们也可以使用类型声明的方式来实现。
```ts
const num: number = mytest(123)
```

这么一看，结果几乎是一样的。

实际上，类型声明的使用会比类型断言要更严格，所以使用类型断言很可能会导致一些隐藏问题。

先来看看，类型断言和类型声明的核心区别：
* A断言为B：需要满足A兼容B，或者B兼容A
* A赋值给B：只有满足B兼容A才行


```ts
interface IPerson {
    name: string
}

interface IFishman {
    // 摸鱼人
    name: string,
    play(): void
}

const person: IPerson = {
    name: 'clz'
}

const fishman = person as IFishman
```

上面的断言：`person`兼容`IFishman`，所以`IPerson`类型能断言为`IFishman`类型，不过会有一些隐藏问题，比如`IFishman`
类型原本是需要有`play`方法的，但是用断言就直接出现了一个没有`play`方法的`IFishman`。

使用类型声明会更严格，但是也能够避免一些隐藏错误。
```ts
const fishman: IFishman = person
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/753b4770d6b34406a902ff4013ee84bf~tplv-k3u1fbpfcp-zoom-1.image)
