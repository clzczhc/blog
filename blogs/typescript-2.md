---
title: 攀爬TS之路(二) 联合类型、对象类型
categories: 前端
date: 2022-06-18 12:44:34
tags:
  - TypeScript
---

# 攀爬TS之路(二) 联合类型、对象类型

## 联合类型
联合类型表示变量的取值可以是指定的多个类型中的一种。(JS中没有的概念)

使用起来很简单，只需要在类型之间使用`|`分隔开就行了。
```ts
let strOrBool: string | boolean = '赤蓝紫'

strOrBool = 'clz'
console.log(strOrBool)
console.log(typeof strOrBool)

strOrBool = true
console.log(strOrBool)
console.log(typeof strOrBool)    

strOrBool = 123     // 这里会报错，因为联合类型里面没有包括`number`类型
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f743abe9205a414681184a3eed33eef0~tplv-k3u1fbpfcp-zoom-1.image)

有一个有点意思的地方，联合类型和任意值类型编译成的JS是一样的。
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181244851.png)

不过细想以下的话也会发现理所当然，毕竟JS是动态类型，也并没有联合类型的概念

联合类型的变量**只能访问**联合类型中**所有类型共有的属性或方法**。因为TS没法确定这个变量究竟是哪个类型。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca3bb581b5af48d68ac28f2b83f45eb4~tplv-k3u1fbpfcp-zoom-1.image)

当然，如果访问的是共有属性或方法的话，那就都没问题。
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181244766.png)

如果能够推断出当前类型是哪一个的话，就不再只能访问共有的了。
当我们访问的属性是推断出来的类型有的，就不会报错，如果是没有的才会报错。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b40fb24afdb4d1b96e3f530f985c2fe~tplv-k3u1fbpfcp-zoom-1.image)

## 对象类型(接口)
通过接口`interface`来定义对象的类型

这里的接口和开发时和后端对接的接口不是同一个东西。它是对行为的抽象，在Java中则是抽象方法的集合，类通过继承接口来继承接口的抽象方法并实现。

但是，在TS中，常用来定义对象的类型。

使用方法：
1. 定义一个接口`IPerson`，在接口中声明一些变量，并指定类型
2. 然后定义一个对象，并把它的类型定义成接口的类型`IPerson`

```ts
interface IPerson {
    name: string,
    age: number
}

const person: IPerson = {
    name: '赤蓝紫',
    age: 21
}

console.log(person)
```

这个时候，我们定义的对象必须要和接口有的属性一模一样，不能少、不能多、属性类型不能不匹配

少：
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19a1b87cb77942ee90a77840d00fd94b~tplv-k3u1fbpfcp-zoom-1.image)


多：
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26a70b9d485244e594b2d8c81f19fbc9~tplv-k3u1fbpfcp-zoom-1.image)


属性类型不匹配:
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e79f73610faa4e4db6227ba37b70adb1~tplv-k3u1fbpfcp-zoom-1.image)

### 可选属性
上面已经说过：我们定义的对象必须要和接口有的属性一模一样，这当然会很不方便，所以这个时候可以使用可选属性，就可以允许该属性不存在。

使用方法很简单，就是在类型定义时不再是用`:`，而是使用`?:`

```ts
name?: string;
```

实操：

```ts
interface IPerson {
    name?: string,
    age: number
}

const person: IPerson = {
    age: 21,
}

console.log(person)
```

当然，这个时候还是不允许添加未定义的属性。属性不匹配就更是如此，毕竟原本就是为了引入属性匹配才使用的TS。

### 任意属性
这个属性顾名思义，就是允许有任意的属性。

使用方法就是使用中括号包住属性名，并且属性名必须要定义类型。

```ts
[key: string]: any;     // 对象可以有任意属性，键是字符串类型的，值是任意类型的
```

实操：

```ts
interface IPerson {
    name?: string,
    age: number,
    [key: string]: any
}

const person: IPerson = {
    age: 21,
    job: 'Coder'
}

console.log(person)
```

**注意**：如果定义了任意属性，那么一般属性和可选属性的类型都要是**任意属性的值的类型的子集**
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181244984.png)

简单来说就是，任意类型是其他属性的老大哥，所以其他属性不能是其他类型，因为其他属性只能听老大哥的。

可以使用联合类型来更方便地使用任意属性。

```ts
interface IPerson {
    name?: string,
    age: number,
    [key: string]: string | number,
}

const person: IPerson = {
    age: 21,
    job: 'Coder'
}

console.log(person)
```

### 只读属性

使用方法：在定义接口的属性时，在想设置为只读属性的属性前，添加关键字`readonly`即可，这时候该属性在不允许被重新赋值。

```ts
readonly id: string;    // 只读属性
```

如果只读是对象类型，那就可以修改它里面的成员，当然还是不允许重新赋值的。


实操：
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1ca3e6e91774509a3bbbb9156f951b2~tplv-k3u1fbpfcp-zoom-1.image)

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181244989.png)
