---
title: JavaScript之迭代器
categories: 前端
date: 2022-04-30 17:17:03
tags:
  - JavaScript
---

# JavaScript之迭代器
> 看红宝书+查资料，重新梳理JavaScript的知识。

迭代就是指可以从一个**数据集**中按照一定的顺序，不断取出数据的过程。

那么迭代和遍历有啥子区别呢？

* 迭代强调依次取数据的过程，不保证把所有的数据都取完
* 遍历强调的是要把所有的数据依次全部取出



在JavaScript中，迭代器是能调用` next`方法实现迭代的一个对象，该方法返回一个具有两个属性的对象。

* ` value`：可迭代对象的下一个值
* ` done`：表示是否已经取出所有的数据了。` false`表示还有数据，` true`表示后面已经没有数据了。



## 迭代器简单使用

通过可迭代对象中的迭代器工厂函数` Symbol.iterator`来生成迭代器。

```js
const arr = []
console.log(arr)
```

![image-20220415105242128](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c25ad4ea4934f80a1806fda55debaa4~tplv-k3u1fbpfcp-zoom-1.image)



```js
const arr = [1, 2, 3]

const iter1 = arr[Symbol.iterator]()   // 通过迭代器工厂函数` Symbol.iterator`来生成迭代器。
console.log(iter1)

console.log(iter1.next())
console.log(iter1.next())
console.log(iter1.next())
console.log(iter1.next())

console.log('%c%s', 'color:red;font-size:24px;', '================')

const mymap = new Map()
mymap.set('name', 'clz')
mymap.set('age', 21)

const iter2 = mymap[Symbol.iterator]()   // 通过迭代器工厂函数` Symbol.iterator`来生成迭代器。
console.log(iter2)

console.log(iter2.next())
console.log(iter2.next())
console.log(iter2.next())
```

![image-20220415102139605](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29351431e05e4ede895a78638730b234~tplv-k3u1fbpfcp-zoom-1.image)

可以发现，迭代器**是取完**最后一个值之后，即迭代器下一个值` value`为` undefined`时，完成。

但是，上面的说法并不是很准确，并不是迭代器下一个值` value`为` undefined`时，就完成的。还需要判断是不是真的没有值，还是是可迭代对象里就有一个值为` undefined`。如果是可迭代对象里有一个值为` undefined`的情况，那么此时还是不会变成完成状态。

```js
const arr = [1, 2, 3, undefined]

const iter1 = arr[Symbol.iterator]()   // 通过迭代器工厂函数` Symbol.iterator`来生成迭代器。
console.log(iter1)

console.log(iter1.next())
console.log(iter1.next())
console.log(iter1.next())
console.log(iter1.next())
console.log(iter1.next())
```

![image-20220415102828917](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0a30494dc164ead83746dc7830d6d23~tplv-k3u1fbpfcp-zoom-1.image)



## 不同迭代器之间互不干扰

可以多次调用迭代器工厂函数来生成多个迭代器，每个迭代器都表示对可迭代对象的一次性有序遍历。**不同迭代器之间互不干扰，只会独立地遍历可迭代对象**。

```js
const arr = [1, 2, 3]

const iter1 = arr[Symbol.iterator]()   // 通过迭代器工厂函数` Symbol.iterator`来生成迭代器。
const iter2 = arr[Symbol.iterator]()

console.log('迭代器1:', iter1.next())
console.log('迭代器2:', iter2.next())
console.log('迭代器1:', iter1.next())
console.log('迭代器2:', iter2.next())
```

![image-20220415110326267](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/537304dd849446a0975f51b513f09a21~tplv-k3u1fbpfcp-zoom-1.image)

## 迭代器对象可作为可迭代对象

```js
const arr = [1, 2, 3]
const iter = arr[Symbol.iterator]()

for (const i of iter) {
    console.log(i)    // 依次输出1、2、3
}
```





## 迭代器"与时俱进"

如果可迭代对象在迭代期间被修改了，迭代器得到的结果也会是修改后的。

```js
const arr = [1, 2, 3]
console.log(arr)

const iter = arr[Symbol.iterator]()
console.log(iter.next())

arr[1] = 999

console.log(iter.next())
console.log(iter.next())
```

![image-20220415110751803](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a21103eab2e46f382d3b7263e7f9c7b~tplv-k3u1fbpfcp-zoom-1.image)



##  完成但并不完成

当我们迭代到` done: true`之后，再调用`next`是不是会报错，或者不返回任何内容呢？

然而，并不是，迭代器会处于一种**完成但并不完成**的状态，` done: true`表示已经完成了，但是后续还能一直调用` next`，虽然得到的结果一直都会是` { value: undefined, done: true }`。这就是为什么说**完成但并不完成**。

```js
const arr = [1, 2, 3]

const iter = arr[Symbol.iterator]()

console.log(iter.next())
console.log(iter.next())
console.log(iter.next())
console.log(iter.next())
console.log(iter.next())
console.log(iter.next())
```

![image-20220415120158682](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e2c749e3b62439eac910f99280a8fb1~tplv-k3u1fbpfcp-zoom-1.image)



## 自定义迭代器

从上面的例子中，我们就可以知道是通过**通过迭代器工厂函数` Symbol.iterator`来生成迭代器**，所以我们需要实现一个迭代器迭代器工厂函数，然后迭代器可以调用` next`方法，所以还需要实现一个` next`方法，至于迭代器工厂函数，实际上直接返回实例` this`。

计数器例子：

```js
class Counter {
  constructor(limit) {
    this.count = 1
    this.limit = limit
  }
  next() {
    if (this.count <= this.limit) {
      return {
        done: false,
        value: this.count++
      }
    } else {
      return { done: true, value: undefined }
    }
  }
  [Symbol.iterator]() {
    return this
  }
}
```



```js
const counter = new Counter(3)
const iter = counter[Symbol.iterator]()

console.log(iter.next())
console.log(iter.next())
console.log(iter.next())
console.log(iter.next())
console.log(iter.next())
```

![image-20220416105256374](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1167066e0dae4ef1a03802bde864f9b9~tplv-k3u1fbpfcp-zoom-1.image)



乍一看，没啥问题，但是如果我们使用` for-of`来遍历就能发现问题。

```js
const counter = new Counter(3)

for (let i of counter) {
  console.log(i)
}

console.log('另一轮迭代：')
for (let i of counter) {
  console.log(i)
}
```

![image-20220416105416835](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6949ce37821f4dc29b825ed2b8789828~tplv-k3u1fbpfcp-zoom-1.image)



使用` for-of`循环也变成一次性的了。这是因为` count`是该实例的变量，所以两次迭代都是使用的那一个变量，但是该变量第一次循环完之后，就已经超过限制了，所以再次使用` for-of`循环就得不到任何的结果了。



可以把` count`变量放在闭包里，然后通过闭包返回迭代器，这样子每创建一个迭代器都会对应一个新的计数器。

```js
class Counter {
    constructor(limit) {
        this.limit = limit
    }
    
    [Symbol.iterator]() {

        let count = 1
        const limit = this.limit

        return {
            // 迭代器工厂函数必须要要返回一个带有next方法的对象，因为迭代实际就是通过调用next方法来实现的
            next() {
                if (count <= limit) {
                    return {
                        done: false,
                        value: count++
                    }
                } else {
                    return { done: true, value: undefined }
                }
            }
        }
    }
}
```



测试

```js
const counter = new Counter(3)

for (let i of counter) {
    console.log(i)
}

console.log('另一轮迭代：')
for (let i of counter) {
    console.log(i)
}
```

![image-20220416110540741](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbb72f540e014e499c7b12d5145554d1~tplv-k3u1fbpfcp-zoom-1.image)



## 提前终止迭代器

就和使用` for-of`循环一样，迭代器会很聪明地去调用` next`方法，当迭代器提前终止时，它也会去调用` return`方法。

```js
[Symbol.iterator]() {

    let count = 1
    const limit = this.limit

    return {
        // 迭代器工厂函数必须要要返回一个带有next方法的对象，因为迭代实际就是通过调用next方法来实现的
        next() {
            if (count <= limit) {
                return {
                    done: false,
                    value: count++
                }
            } else {
                return { done: true, value: undefined }
            }

        },

        return() {
            console.log('提前终止迭代器')
            return { done: true }
        }
    }
}
```



测试

```js
const counter = new Counter(5)

for (let i of counter) {
    if (i === 3) {
        break;
    }
    console.log(i)
}
```

![image-20220416111320222](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/464f7994fa2d4e0fab4d9600b474e230~tplv-k3u1fbpfcp-zoom-1.image)



**如果迭代器没有关闭，就可以继续从上次离开的地方继续迭代**。数组地迭代器就是不能关闭的。

```js
const arr = [1, 2, 3, 4, 5]
const iter = arr[Symbol.iterator]()

iter.return = function () {
    console.log('提前退出迭代器')

    return {
        done: true
    }
}

for (const i of iter) {
    console.log(i)
    if (i === 2) {
        break
    }
}

for (const i of iter) {
    console.log(i)
}
```

![image-20220416112206220](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abf4f2cef97d44659f1ed767baa74e70~tplv-k3u1fbpfcp-zoom-1.image)
