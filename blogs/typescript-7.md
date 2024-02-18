---
title: 攀爬TS之路(七)    类与接口
categories: 前端
date: 2022-06-18 12:48:44
tags:
  - TypeScript
---

# 攀爬TS之路(七)    类与接口

## 类
这里不会赘述JS中的类的用法，而是单刀直入，直接来TS中的类的用法。

### 访问修饰符
先提一嘴，JS中的类有私有属性，在属性名之前使用`#`表示。**私有属性只能在类的内部使用**。
```js
class Person {
    #name = 'clz'

    get val() {
        return this.#name
    }
}

const person = new Person()
console.log(person.val)
```

有学过其它语言的可能就会用的有点不太习惯，因为很多语言(指本人课程教的，C++、Java)使用的访问修饰符是`public`、`private`、`protected`。而TS可以使用这三种访问修饰符。
* `public`：修饰的属性和方法是公有的，可以在任何地方被访问。默认都是`public`
* `private`：私有的，**只能在声明该属性的类中访问**，即也不能被子类访问
* `protected`：受保护的，和`private`类似，不过，**能被子类访问**

#### public
修饰的属性和方法是公有的，可以在任何地方被访问。

实例：

```ts
class Person {
    public name;
    public constructor(name) {
        this.name = name
    }
}

class Student extends Person {
    public grade;

    public constructor(name, grade) {
        super(name)

        console.log(this.name)
        this.grade = grade
    }
}


const student = new Student('czh', 3)      // czh

const person = new Person('clz')
console.log(person.name)                    // clz
```


#### private
**只能在声明该属性的类中访问**，即也不能被子类访问。

实例：把上面的例子中，`name`的修饰符变成`private`即可。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6017eab0cd7441ab50bbf46a62a5573~tplv-k3u1fbpfcp-zoom-1.image)


#### protected
和`private`类似，不过，**能被子类访问**
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f4d4f6a323a440cb046a196987d9cc4~tplv-k3u1fbpfcp-zoom-1.image)

### 参数属性
访问修饰符和`readonly`修饰符能够**直接使用在构造函数的参数中**。相当于在类中定义该属性的同时赋值。只读属性的用法在对象那一节已经介绍过了。

原版本：
```ts
class Person {
    public name;
    public constructor(name) {
        this.name = name
    }
}
```

使用参数属性的简洁版：
```ts
class Person {
    public constructor(public name) { }
}
```

另外，如果需要同时使用访问修饰符和`readonly`修饰符的话，**访问修饰符要在`readonly`修饰符之前**，如`public readonly name`。

### 抽象类
`abstract`用于定义抽象类和其中的抽象方法。对一个学过Java的人来说，就是面向对象这一块，TS和Java感觉上就是一样的。

抽象类主要是一些**没有足够信息来描绘一个具体的对象的类**。所以**抽象类必须被继承获取足够信息，才能被使用**。**抽象类不能被实例化对象**，但是类的其他功能依然存在。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab74d74dbf04439b85f657d5d15ac118~tplv-k3u1fbpfcp-zoom-1.image)


抽象类的使用：
```ts
abstract class Person {
    public constructor(public name) { }
}

class Student extends Person {
    public grade;
    public constructor(name, grade) {
        super(name)
        this.grade = grade
    }
}


const student = new Student('clz', 3)
console.log(student)    // Student {name: 'clz', grade: 3}
```

抽象类还可以有抽象方法，**抽象方法只能出现在抽象类中，子类必须实现抽象方法**
```ts
abstract class Person {
    public constructor(public name) { }

    public abstract listen()
}

class Student extends Person {
    public grade;
    public constructor(name, grade) {
        super(name)
        this.grade = grade
    }

    // public listen() {
    //     console.log('Kylee-大好きなのに')
    // }
}


const student = new Student('clz', 3)
student.listen()    // Kylee-大好きなのに
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c00e500131344231b674432523f7c9e9~tplv-k3u1fbpfcp-zoom-1.image)


### 类限制变量、函数类型
```ts
class Person {
    public name: string;
    public constructor(name: string) {
        this.name = name
    }

    getName(): string {
        return this.name
    }
}


const person = new Person('clz')
console.log(person.getName())   // clz
```

## 类与接口
**一般来说，一个类只能继承自另一个类**。(C++可以多继承)

但是，有时候不同类之间有一些共有特性，可以将它们封装成接口。

### 类实现接口

就拿前面的`Person`类举例子，所有人都需要吃、睡，即可以封装一个`Normal`接口，包含必须的行为。然后通过`implements`关键字去实现接口。**接口只是声明，需要类通过`implements`关键字实现**

```ts
interface Normal {
    eat(): void
    sleep(): void
}

class Person implements Normal {
    public name: string;
    public constructor(name: string) {
        this.name = name
    }

    eat() {
        console.log('吃')
    }

    sleep() {
        console.log('睡')
    }
}


const person = new Person('clz')

person.eat()    // 吃
```

### 类实现多个接口
还可以把`Normal`接口分解成`Eat`接口和`Sleep`接口，然后同时实现两个接口。
```ts
interface Eat {
    eat(): void
}

interface Sleep {
    sleep(): void
}

class Person implements Eat, Sleep {}
```

上面的Person类中省略了代码，代码和`Normal`接口的案例一样。

### 接口继承接口
* **类可以继承类，接口也可以继承接口。**
* **类只能继承一个类，但是接口可以继承多个接口**

```ts
interface Eat {
    eat(): void
}

interface Drink {
    drink(): void
}

interface Sleep {
    sleep(): void
}

interface Normal extends Eat, Drink, Sleep { }
```
