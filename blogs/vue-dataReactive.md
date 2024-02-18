---
title: Vue源码之数据响应式原理
categories: 前端
date: 2022-04-04 14:20:45
tags:
  - Vue源码
---

# Vue源码之数据响应式原理

本文写了好久(个人菜+没时间)，看了很多博客，才写完这篇博客。如果对你有所帮助，希望点赞一波。

## MVVM

* **M( Model )**：视图渲染时依赖的数据
* **V( View )**：视图(页面渲染的DOM结构)
* **VM( ViewModel )**：Vue的实例，MVVM的核心

****

![img](https://s2.loli.net/2022/03/27/ZXd3PB9fOKN2xTL.jpg)

<br>

* **数据驱动视图**：数据变化，会被` ViewModel`监听到，然后就会自动更新视图
* **双向数据绑定**：表单元素的值变化时，也会被` ViewModel`监听到，然后更新数据

****

**数据响应式其实也就是数据驱动视图(不同说法)**

<br>

简单例子：

![image-20220327091241111](https://s2.loli.net/2022/03/27/Ke8g1aujfEiLU4H.png)

<br>

## Vue与React的小差别

<b style="color: red">Vue可以直接对数据进行修改，而React需要` setState`来修改</b>

```js
// Vue数据变化
this.a ++;

// React数据变化
this.setState({
    a: this.state.a + 1
});
```

<br>

这就是因为` React`不是响应式更新，无法做到检测属性的变化，去驱动` render`函数的执行，所以需要使用` setState`，也就是说` setState`不只是更新数据，还会根据数据的变化去更新视图。而` Vue`是响应式的，所以可以做到检测属性的变化。

> 二者各有各的优缺点。` Vue`的响应式比较方便，但` React`的则是更规范，可以避免不小心改掉数据的问题，实际上` Vue3`有点看齐的意思，修改数据是必须要` 数据.value`才能修改(Vue3还没有用很多，可能有错误理解)

## Object.defineProperty()

开始数据响应式的原理讲解之前，先来一下前置知识

> **`Object.defineProperty()`** 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

### 简单使用

```js
const obj = {}

Object.defineProperty(obj, 'name', {
  value: 'clz'
})

Object.defineProperty(obj, 'age', {
  value: 21
})


console.log(obj)                // {name: 'clz', age: 21}
console.log(obj.name, obj.age)        // clz 21
```

<br>

这么一看，**`Object.defineProperty()`**不是没啥用吗?下面的代码也能实现效果

```js
const obj = {}

obj.name = 'clz'

obj.age = 21

console.log(obj)
console.log(obj.name, obj.age)
```

<br>

### 属性配置

可以通过第三个参数，设置是都可以枚举、是否可写等

```js
const obj = {}

Object.defineProperty(obj, 'name', {
  value: 'clz',
  enumerable: false
})

Object.defineProperty(obj, 'age', {
  value: 21,
  writable: false,     // 不可写
  enumerable: true
})


console.log(obj)         // {name: 'clz', age: 21}
console.log(obj.name, obj.age)    // clz 21

obj.age++             // 不报错，但也不会被修改
console.log(obj)   // {name: 'clz', age: 21}。writable若为true，则age为22


for (let item in obj) {
  console.log(item)   // age。因为name属性设置了不可枚举，所以只能打印出age
}
```

<br>

### 数据劫持原理

**数据劫持就是当访问数据或修改数据时，然后执行我们想做的事(即通过自定义的` get`和` set`方法来重写原来的行为)**

<b style="color: red">注意：如果已经设置`set`或`get`, 就不能设置`writable`和`value`中的任何一个了,不然会报错</b>

```js
const obj = {}

Object.defineProperty(obj, 'name', {
  get() {
    console.log('试图访问obj的name属性')
  },
  set() {
    console.log('试图改变obj的name属性')
  }
})


console.log(obj)
console.log(obj.name)
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')

obj.name = 'czh'
console.log(obj)
console.log(obj.name)
```

![image-20220327125438377](https://s2.loli.net/2022/04/04/RNszBvL342rKPqD.png)

<br>

`get`方法`return`的值就是访问该属性时的值，` set`方法的参数就是该属性的新值

<br>

## 实现数据劫持

### 临时变量实现

使用临时变量` temp`，修改数据时，` temp`储存新值；获取数据时, 返回` temp`

```js
const obj = {}

let temp

Object.defineProperty(obj, 'name', {
  get() {
    console.log('试图访问obj的name属性')
    return temp
  },
  set(newVal) {
    console.log('试图改变obj的name属性', newVal)
    temp = newVal
  }
})

console.log(obj.name)
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')

obj.name = 'czh'
console.log(obj.name)
```

![image-20220327145335132](https://s2.loli.net/2022/04/04/Ldf95HnAYuzQ3cx.png)

<br>

### 闭包实现

通过闭包实现，减少临时变量的使用。

```js
const obj = {}

function defineReactive(data, key, val) {
  Object.defineProperty(data, key, {
    // 可被枚举
    enumerable: true,
    // 可被配置，如被delete
    configurable: true,
    // getter
    get() {
      console.log(`试图访问obj的${key}属性`)
      return val
    },
    // setter
    set(newVal) {
      console.log(`试图改变obj的${key}属性`, newVal)
      val = newVal
    }
  })
}

defineReactive(obj, 'name', 'clz')

console.log(obj.name)
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')

obj.name = 'czh'
console.log(obj.name)
```

![image-20220327160734232](https://s2.loli.net/2022/04/04/fqtRTc5ZlxnH1Y9.png)

效果也比临时变量版本好一点，因为调用` defineReactive`的时候可以传第三个参数(可以添加属性的同时添加值)

<br>

有趣的事：直接打印` obj`，无法直接看到属性和属性值，展开后才能看到，并且也还是会触发` get`方法

![reactive](https://s2.loli.net/2022/04/04/7lOueB18IT9NExq.gif)

### 对象深层属性全部劫持

首先,对` defineReactive`函数进行小修改，看一下对象情况时的问题

```js
const obj = {
  job: {
    salary: 1111
  }
}

function defineReactive(data, key, val) {
  if (arguments.length === 2) {
    val = data[key]
  }

  Object.defineProperty(data, key, {
    // 可被枚举
    enumerable: true,
    // 可被配置，如被delete
    configurable: true,
    // getter
    get() {
      console.log(`试图访问obj的${key}属性`)
      return val
    },
    // setter
    set(newVal) {
      console.log(`试图改变obj的${key}属性`, newVal)
      val = newVal
    }
  })
}

defineReactive(obj, 'job')

console.log(obj.job.salary)
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')

obj.job.salary += 1111
console.log(obj.job.salary)
```

![image-20220327170516140](https://s2.loli.net/2022/03/27/fMmd8WPz1tgic6V.png)

可以看到,当访问` salary`属性,只能劫持到` job`属性,无法劫持到更深一层的` salary`,修改时更离谱,应为没有修改到` job`属性(即只是把` job`属性下的` salary`属性修改了, `job`对应的地址并没有发生变化)

<br>

#### 如何实现思路及流程

既然正常的对象不能被劫持所有的属性,那么就定义一个` Observer`类,用于把一个正常的对象转换为每一层的属性都是响应式的对象(即深层的属性也能被侦测到)

<br>

但是呢,如果属性值已经是响应式的了,那就没有必要再创建` Observer`实例了。

所以还需要有一个函数` observe`。

![image-20220326111642691](https://s2.loli.net/2022/03/26/PnE89l4zW12qpKO.png)

<br>

#### observe.js

> 监听value。如果value已经是响应式的对象了，那么就直接返回已经创建的` Observer`实例即可，否则创建` Observer`实例。
> 
> 那么怎么判断对象是不是响应式的对象呢？通过实例是否存在` __ob__`属性来判断，如果已经是响应式的，那么该对象的` __ob__`属性的值就是` Observer`的实例。

```js
import Observer from './Observer.js'

export default function observe(value) {
  // 如果value不是对象，就什么都不做
  if (typeof value !== 'object') {
    return
  }

  let ob
  if (typeof value.__ob__ !== 'undefined') {
    ob = value.__ob__
  } else {
    ob = new Observer(value)
  }

  return ob
}
```

<br>

#### def.js

> 工具方法。用于定义对象的属性，可传参控制是否可以枚举。
> 
> 上面说到，` Observer`实例要有` __ob__`属性，但是呢，这个属性应该是不可枚举的，所以需要一个工具方法来定义` __ob__`属性。

```js
export const def = function (obj, key, value, enumerable) {
  Object.defineProperty(obj, key, {
    value,
    enumerable,
    writable: true,
    configurable: true,   // 可被配置，如被delete
  })
}
```

<br>

#### Observer.js

> 把一个正常的对象转换为每一层的属性都是响应式的对象(即深层的属性也能被侦测到)。

```js
// 将一个正常的对象转换为每个层级的属性都是响应式的object(可以被侦测到的object)

import { def } from './def.js'
import defineReactive from './defineReactive.js'

export default class Observer {
  constructor(value) {
    // 给value添加__ob__属性，值是实例，且__ob__属性不可枚举
    // ` __ob__`属性用于标记是否已经被转换成响应式数据了，并且值是Observer的实例
    def(value, '__ob__', this, false)
    this.walk(value)
  }


  // 遍历对象的属性，并让它们都变成响应式的
  walk(value) {
    for (let k in value) {
      defineReactive(value, k)
    }
  }
}
```

<br>

#### defineReactive.js

```js
import observe from './observe.js'

export default function defineReactive(data, key, val) {
  // console.log(data, key)
  if (arguments.length === 2) {
    val = data[key]
  }

  // 对子属性进行侦测。避免子属性是对象，且还不是响应式的情况
  let childOb = observe(val)

  Object.defineProperty(data, key, {
    // 可被枚举
    enumerable: true,
    // 可被配置，如被delete
    configurable: true,

    // getter
    get() {
      console.log(`试图访问${key}属性`)
      return val
    },
    // setter
    set(newValue) {
      console.log(`试图改变${key}属性`, newValue)

      if (val === newValue) {
        return
      }

      val = newValue
      // 设置的新值同样需要observe(防止赋值的新值是对象，同样需要侦测)
      childOb = observe(newValue)
    }
  })
}
```

<br>

#### index.js

测试代码

```js
import observe from './observe.js'

const obj = {
  job: {
    salary: 1111
  },
  color: ['red', 'blue', 'purple']
}


observe(obj)

console.log(obj.job.salary)
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')

obj.job.salary += 1111
console.log(obj.job.salary)
```

![image-20220327213314578](https://s2.loli.net/2022/03/27/Pv6yjTKG7ip84Bs.png)

可以看到，当访问` salary`属性时，会先访问`job`属性，然后也访问` salary`属性。当然，修改` salary`属性值时，并不会修改` job`属性，应为` job`是对象，是引用类型，它指向的地址没有变化,自然触发不了对应的` set`方法。

#### 简单流程图

![image-20220327225344833](https://s2.loli.net/2022/03/27/4XnvyC9EiVApOkb.png)

**这不就是真正的三角恋吗？(偷笑)**

* `observe`依赖` Observer`(男A喜欢女A)
* ` Observer`依赖` defineReactive`(女A喜欢男B)
* `defineReactive`依赖` observe`(男B喜欢男A，瑟瑟发抖)

<br>

### 劫持数组方法的调用

首先，让我们看下数组的情况。

```js
import observe from './observe.js'

const obj = {
  color: ['red', 'blue', 'purple']
}


observe(obj)

console.log(obj.color[2])
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')

obj.color[2] = 'Purple'
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')

obj.color.push('white')
console.log(obj.color)
console.log(obj.color[3])
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')
```

![image-20220327230501029](https://s2.loli.net/2022/03/27/vanqOXsRfYAypgF.png)

**数组新增数据不是响应式的，且数组内的普通数据不应该是对象形式的(即属性名为索引值)**

<br>

**所以我们需要改写七个数组方法，微操一下**

* `push`
* `pop`
* ` shift`
* ` unshift`
* ` splice`
* ` sort`
* ` reverse`

> 为什么只要改写这七个方法呢?
> 
> <b style="color: red">因为只有这七个方法会改变原数组</b>

怎么改写呢？就是利用原型链的知识，在` Array`与` Array.prototype`中间插一个` arrayMethods`，在` arrayMethods`上改写数组方法

![image-20220328131543234](https://gitee.com/clzczh/picgo/raw/master/images/bu4lkTjvAdJ2qLc.png)

<br>

#### array.js

1. 备份原来的方法

2. 定义新的方法
   
   2.1 调用备份的方法(需要通过` apply`变更` this`指向)
   
   2.2 对新增项进行` objectArray`，将新增项响应式化

```js
// 重写7个数组方法

import { def } from './def.js'

const ArrayPrototype = Array.prototype

// 以Array.prototype为原型创建arrayMethods对象(其实就是arrayMethods对象就能继承Array.prototype的方法)
export const arrayMethods = Object.create(ArrayPrototype)

// 要被改写的7个数组方法
const methodsNeedChange = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]

methodsNeedChange.forEach(methodName => {
  // 备份原来的方法
  const original = ArrayPrototype[methodName]

  // 定义新的方法
  def(arrayMethods,
    methodName,
    function () {
      // 这里不能使用箭头函数，因为apply调用需要this来绑定调用此方法的数组

      // 恢复数组方法原有的功能
      const result = original.apply(this, arguments)

      // 把类数组变成数组
      const args = [...arguments]

      // 把数组的__ob__属性取出来。用于把数组的每一项都变成响应式
      const ob = this.__ob__

      // 新增数据也要变成响应式
      let inserted = []
      switch (methodName) {
        case 'push':
        case 'unshift':
          inserted = arguments
          break
        case 'splice':
          inserted = args.slice(2)
          break
      }

      if (inserted) {
        // 有新增的项，需要变为响应式
        this.__ob__.observeArray(inserted)
      }

      return result
    }, false)
})
```

<br>

#### Observer.js

因为数组和普通的对象不一样，所以` Observer`类也需要修改一下

```js
// 将一个正常的object对象转换为每个层级的属性都是响应式(可以被侦测到的object)

import { def } from './def.js'
import defineReactive from './defineReactive.js'
import { arrayMethods } from './array.js'
import observe from './observe.js'

export default class Observer {
  constructor(value) {
    // 给value添加__ob__属性，值是实例，且__ob__属性不可枚举
    // ` __ob__`属性用于标记是否已经被转换成响应式数据了，并且值是Observer的实例
    def(value, '__ob__', this, false)

    // 判断是不是数组
    if (Array.isArray(value)) {
      // 设置数组的原型，这样子才可以调用自己改写后的数组方法
      Object.setPrototypeOf(value, arrayMethods)

      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }


  // 遍历对象的属性，并让它们都变成响应式的
  walk(value) {
    for (let k in value) {
      defineReactive(value, k)
    }
  }

  // 遍历数组。因为数组是特殊的对象，属性名是索引值，所以不适合用和普通对象的遍历方式
  observeArray(arr) {
    for (let i = 0, len = arr.length; i < len; i++) {
      // 判断数组的每一项是否已经是响应式的了。
      observe(arr[i])
    }
  }
}
```

<br>

#### index.js

测试

```js
import observe from './observe.js'

const obj = {
  color: ['red', 'blue', 'purple']
}


observe(obj)

console.log(obj.color[2])
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')

obj.color[2] = 'Purple'
console.log('%c%s', 'font-size: 28px;color: red', '----------------------')

obj.color.push({
  name: 'white'
})
console.log(obj.color)
console.log(obj.color[3].name)
```

![image-20220328195821834](https://s2.loli.net/2022/03/28/JyDontshHEZLSwd.png)

<br>

## 收集依赖与派发更新

![image-20220326131006787](https://s2.loli.net/2022/04/03/xp31PRDMLO8ZWUi.png)

这部分学习自[0年前端的Vue响应式原理学习总结1：基本原理](https://juejin.cn/post/6932659815424458760)。因为视频讲的太迷了。

### 什么是依赖

首先，要实现数据响应式，那就得先订阅数据，然后才能在数据发生变化后接收到数据发生变化的消息，订阅的数据就是**依赖**，。当依赖发生变化后，订阅者就会接收到数据发生变化的消息，然后执行回调函数，如更新页面。**在Vue的响应式中的订阅者就是` Watcher`实例**。

实际上，这种例子在现实中也比比皆是。比如关注一个歌手。关注就是订阅这个歌手，然后歌手发生变化后(即发布新歌)，关注者就会收到数据发生变化的消息(歌手发布新歌)。

****

初始化Watcher实例时，需要订阅数据，在下面就是调用get方法，get方法后面再实现。

```js
export default class Watcher {
  constructor(data, expression, callback) {
    // data: 数据对象。
    // expression: 表达式。如通过点语法访问深层属性。a.b.c
    // callback: 依赖发生变化后，执行的回调函数
    this.data = data
    this.expression = expression
    this.callback = callback

    // 订阅数据
    this.value = this.get()
  }
}
```

<br>

订阅数据呢，首先因为我们的表达式是点语法，所以需要` parsePath`方法去一步一步地获取深层属性，因为JS不支持`data[a.b.c]`的形式，需要变为` data[a][b][c]`。

```js
function parsePath(obj, expression) {
  if (!obj) {
    return
  }

  const segments = expression.split('.')

  for (const key of segments) {
    obj = obj[key]
  }

  return obj
}
```

<br>

然后添加` get`方法和` update`方法，功能分别是初始化时订阅数据，数据变化后更新数据以及执行回调函数

```js
get() {
  const value = this.parsePath(this.data, this.expression)
  return value
}

update() {
  // 数据变化时，调用此方法
  this.value = parsePath(this.data, this.expression) //更新数据
  console.log('派发更新, 更新后的数据', this.value)
  this.callback(this.value) // 执行回调
}
```

<br>

### 收集依赖

一个` Watcher`实例不只是能订阅一个数据，就像一个人能关注很多个歌手一样。既然如此，那就需要用一个数组来收集依赖了。

那么，我们什么时候收集依赖呢？

首先，`watcher`初始化的时候，会调用` get`方法去获取它依赖的数据，但是，上面我们已经实现了数据劫持，重写了数据的访问行为，所以会执行` getter`函数(即` Object.defineProperty`中的` get`)，那么就在` getter`函数中把当前的` watcher`添加到` dep`数组中，就能完成收集依赖了。

这里需要获取` watcher`实例，所以应该要在` Watcher`的` get`方法中，先把` watcher`实例挂载到` window`对象中，这样子就变成全局的了。

****

Watcher.js

```js
get() {
  window.target = this    // 挂载实例到window对象上
  const value = parsePath(this.data, this.expression)
  return value
}
```

<br>

defineReactive.js

下面为了篇幅不太长，只弄了部分，前面还应该有收集依赖的数组` const dep = []`才对。

```js
// getter
get() {
  dep.push(window.target)
  console.log('收集依赖')
  return val
},
```

<br>

### 派发更新

数据发生变化后，调用` watcher`的` update`方法，更新数据，并执行回调。

> 在使用数据时收集依赖，在改变数据时派发更新。
> 
> <b style="color: red">在` getter`中收集依赖，在` setter`中派发更新</b>

<br>

defineReactive.js

```js
set(newValue) {
  console.log(`试图改变${key}属性`, newValue)

  if (val === newValue) {
    return
  }

  val = newValue
  // 设置的新值同样需要observe(防止赋值的新值是对象，同样需要侦测)
  childOb = observe(newValue)

  dep.forEach(item => item.update())    // 派发更新
}
```

<br>

### 测试

index.js

```js
import observe from './observe.js'
import Watcher from './Watcher.js'

const obj = {
  job: {
    salary: 1111
  }
}

observe(obj)

new Watcher(obj, "job.salary", (val) => {
  console.log("watcher监听", val)
})

obj.job.salary = {
  rank1: 1111,
  rank2: 2222
}
```

![reactive](https://s2.loli.net/2022/04/03/YRJKlZwgmri49eb.gif)

<br>

分析：

* watcher初始化，订阅数据时，会一层一层地去拿到深层的数据，所以依次访问` obj.job`、` obj.job.salary`，因此会触发两次` getter`函数，所以会有两次收集依赖
* 修改` obj.job.salary`时，会先触发` getter`，再触发` setter`。所以还会收集一次依赖
* 修改数据后，派发更新，要更新数据，又要调用` parsePath`，所以又会有两次收集依赖
* 之后打印测试信息后，会执行回调
* 最后就是，因为前面实现的对象深层属性全部劫持，所以即使是新增属性，访问时也会触发` getter`，即也会收集依赖。

**这里疯狂收集依赖，下面再来优化代码**

<br>

### 优化代码

#### Dep类

将` dep`数组抽象成一个类

Dep.js

```js
export default class Dep {
  constructor() {
    // dep数组。储存依赖的订阅者，即watcher的实例
    this.subs = []
  }

  // 添加依赖
  depend() {
    // 之前是添加到window对象上，但是实际上只要是全局的都可以
    console.log('添加依赖')
    this.addSub(Dep.target)
  }

  // 派发更新
  notify() {
    const subs = [...this.subs]
    subs.forEach(s => s.update())
  }

  // 添加订阅
  addSub(sub) {
    this.subs.push(sub)
  }
}
```

<br>

#### 重置Dep.target

从上面的测试例子分析中，可以发现，我们派发更新时，会触发` parsePath`去取值，这个时候也会添加依赖，所以需要重置`Dep.target`为`null`，并根据` Dep.target`是否是`null`，来判断是不是要收集依赖

<br>

Watcher.js

```js
get() {
  Dep.target = this    // 挂载实例到window对象上
  const value = parsePath(this.data, this.expression)

  Dep.target = null   // 重置Dep.target
  return value
}
```

<br>

defineReactive.js

```js
// getter
get() {
  if (Dep.target) {
    // 条件符合才收集依赖
    dep.depend()
  }

  return val
},
```

![image-20220403202742315](https://s2.loli.net/2022/04/04/hd7gwFl1yqW8o5E.png)

<br>

#### 实现数组版本

因为数组和普通对象不太一样

****

首先，需要为每一个`Observer`实例添加` dep`，然后数组才能够调用

Observer.js

```js
this.dep = new Dep()
```

<br>

然后，如果有子元素，也需要收集依赖(数组收集依赖)。**依赖了数组，就等价于依赖了数组中的所有元素**

defineReactive.js

```js
// getter
get() {
  if (Dep.target) {
    // 条件符合才收集依赖
    dep.depend()

    if (childOb) {
      // 如果有子元素
      childOb.dep.depend()
    }
  }

  return val
},
```

<br>

最后，触发改写的7个数组方法时，需要派发更新。

array.js

所以，**改写后的数组方法最后需要加上下面的语句**

```js
 ob.dep.notify()
```

<br>

测试代码。

index.js

```js
import Watcher from './Watcher.js'
import observe from './observe.js'

const obj = {
  color: ['red', 'blue', 'purple']
}


observe(obj)

new Watcher(obj, 'color', (val) => {
  console.log(val)
})

obj.color.push('white')
```

![image-20220403220551493](https://s2.loli.net/2022/04/03/kv6TXRNocGZd134.png)

<br>

暂时收工，看了视频教程+很多博客，才初略看懂，好菜啊(哭)
