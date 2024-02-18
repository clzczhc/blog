---
title: JSON的使用之灵活版
categories: 前端
date: 2022-06-18 12:40:26
tags:	
  - JavaScript
---

# JSON的使用之灵活版

## 前言

> JSON在正常开发的时候我们经常能够遇到，但是一般情况下，`JSON.stringify()`和`JSON.parse()`已经够用了。在看红宝书的时候，感觉打开了新世界的大门。下面就来探秘一下灵活的JSON使用方法。

## `JSON.stringify()`的几个作用

`JSON.stringify()`方法将一个JavaScript对象转换成JSON字符串。

当属性值为`undefined`、函数时会跳过这个属性,转换的JSON字符串中不存在跳过的属性。



### 过滤结果

#### 第二个参数是数组
如果第二个参数是一个数组，那么`JSON.stringify()`得到的结果只包含该数组中列出的对象属性。 得到的结果也会按数组中对象的属性顺序
```js
let person = {
    name: "clz",
    age: 22,
    hobbies: [
        "Animate",
        "Music",
        'FrontEnd'
    ],
    nowTime: 2022
};

const jsonText = JSON.stringify(person, ['age', 'name'])

console.log(jsonText)	// {"age":22,"name":"clz"}
```



#### 第二个参数是函数

如果第二个参数是一个函数,那么该函数接收两个参数: 属性名(key)和属性值(value)。返回的值就是得到的对象字符串,该属性的值。

如果函数返回的值不是对象,那么得到的结果就是返回的值。

```js
let person = {
    name: "clz",
    age: 22,
    hobbies: [
        "Animate",
        "Music",
        'FrontEnd'
    ],
    nowTime: 2022
};

const jsonText = JSON.stringify(person, (key, value) => {
    console.log('key: ', key)
    console.log('value: ', value)

    // return 123

})

console.log(jsonText)
```

![image-20220529125044285](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82d04f0e5a814a53999529069556569b~tplv-k3u1fbpfcp-zoom-1.image)



那么,怎么得到最初的结果呢?
答案很简单,可以直接返回`value`值就行了。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44fb823dad0147a49b2831cc360985ec~tplv-k3u1fbpfcp-zoom-1.image)




那么如果想修改某一个属性值,就只需要增加条件,根据条件返回就行。
```js
let person = {
    name: "clz",
    age: 22,
    hobbies: [
        "Animate",
        "Music",
        'FrontEnd'
    ],
    nowTime: 2022
};

const jsonText = JSON.stringify(person, (key, value) => {
    switch (key) {
        case 'name':
            return '赤蓝紫';
        case 'hobbies':
            return value.join(',')
        default:
            return value
    }

})

console.log(jsonText)   // {"name":"赤蓝紫","age":22,"hobbies":"Animate,Music,FrontEnd","nowTime":2022}
```



如果想要得到的结果中过滤掉某个属性,那么只需要当`key`等于该属性时,返回`undefined`即可

```js
switch (key) {
    case 'name':
        return '赤蓝紫';
    case 'hobbies':
        return undefined
    default:
        return value
}
```



假如我们想要全部属性都变成`ccc`,是不是只需要返回`ccc`就行了呢?
这个上面已经说过了,如果只是返回`ccc`的话,那么最后的结果只是一个`ccc`字符串,而不是`JSON对象字符串`。

如果想要把全部属性都变成`ccc`,除了返回`ccc`外，还需要当`key`等于空串时,需要返回`value`。

因为只有当`key`等于空串时返回一个对象,才会继续去转换该对象的属性为字符串。否则,就直接拿到值就返回了。(注意: **如果返回的是函数,得到的结果是`undefined`**)
```js
if (key === '') {
    return value
} else {
    return 'ccc'
}

// 结果:{"name":"ccc","age":"ccc","hobbies":"ccc","nowTime":"ccc"}
```



接下来玩点有意思的,我们可以在`key`等于空串时,返回另一个对象,这样就能做到**狸猫换太子**了。

```js
if (key === '') {
    return {
        test: {
            test1: 1,
            test2: 2
        }
    }
} else {
    return value
}

// 结果:{"test":{"test1":1,"test2":2}}
```



### 字符串缩进

上面我们通过`JSON.stringify()`得到的`JSON对象字符串`是没有格式的,看起来有点杂乱无章。这时候就轮到我们的第三个参数出场了,第三个参数用来控制缩进字符。

如果第三个参数是数值,表示缩进的空格数。最大缩进值为10,大于10的值会自动设置为10。
```js
let person = {
    name: "clz",
    age: 22,
    hobbies: [
        "Animate",
        "Music",
        'FrontEnd'
    ],
    nowTime: 2022
};

const jsonText = JSON.stringify(person, null, 4)
console.log(jsonText) 
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/314460d2d2cc41b88104a713a473afda~tplv-k3u1fbpfcp-zoom-1.image)

第三个参数也可以是字符串,这样子就会使用这个字符串来缩进。
比如:
```js
const jsonText = JSON.stringify(person, null, '--')
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0532f12e2be945f6b2719f1467a743a9~tplv-k3u1fbpfcp-zoom-1.image)

### toJSON()方法
如果对象的值是函数,那么我们使用`JSON.stringify()`时,就会忽略掉这个属性。
```js
let person = {
    name: "clz",
    age: 22,
    hobbies: [
        "Animate",
        "Music",
        'FrontEnd'
    ],
    nowTime: 2022,
    tt: function () {

    }
};

const jsonText = JSON.stringify(person, null, 4)
console.log(jsonText) 
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b69f3898fe14c4f96410194215efb92~tplv-k3u1fbpfcp-zoom-1.image)

但是,当名称是`toJSON`时,就不太一样了,如果调用`toJSON`能获取到实际的值,那么`JSON.stringify()`会得到`toJSON`返回的实际值。当然,这个实际值也是会受到第二个参数、第三个参数的影响的。

```js
let person = {
    name: "clz",
    age: 22,
    hobbies: [
        "Animate",
        "Music",
        'FrontEnd'
    ],
    nowTime: 2022,
    toJSON: function () {
        return {
            name: 'haha',
            age: 22,
            job: {
                type: 'Coder',
            }
        }
    }
};

const jsonText = JSON.stringify(person, ['name', 'age'], 4)
console.log(jsonText) 
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a07fcb6e925d4a9793f65b398e364030~tplv-k3u1fbpfcp-zoom-1.image)


## JSON.parse()
`JSON.parse()`方法可以把JSON字符串转换为JSON对象
```js
let person = '{"name": "clz"}'

const jsonObj = JSON.parse(person)

console.log(jsonObj)    // {name: 'clz'}
```



**JSON字符串属性必须使用双引号,单引号会导致语法错误**

```js
let person = "{'name': 'clz'}"
// SyntaxError: Unexpected token ' in JSON

const jsonObj = JSON.parse(person)
console.log(jsonObj)    
```



如果属性值是字符串,也需要使用`双引号`,否则会语法错误。如果属性值不是字符串,那么就不需要引号。

```js
let person = `{"name": "clz"}`      // 正常,不报错
person = '{"age": 21}'              // 属性值不是字符串,属性值就不需要双引号(属性名还是需要滴)

person = `{"name": 'clz'}`       // 报语法错误
const jsonObj = JSON.parse(person)
console.log(jsonObj)    
```



值不能是`undefined`和函数,会报错,加上双引号后,不会报错,但值也变成字符串类型了。

```js
let person = `{"name": "undefined"}`      // 不报错,但是值会变成字符串类型
// person = `{"name": undefined}`      // 语法错误

const jsonObj = JSON.parse(person)
console.log(jsonObj)    
```



### 第二个参数

`JSON.parse()`可以接收第二个参数(函数), 用来修改解析生成的原始值。每个键值对都会调用一次,有点点像数组的`map`函数。

```js
let person = {
    name: "clz",
    age: 21,
    nowTime: 2022,
};

const jsonStr = JSON.stringify(person)

const jsonObj = JSON.parse(jsonStr, (key, value) => {
    if (key === 'name') {
        return '赤蓝紫'
    } else if (key === 'age') {
        // 如果返回undefined，则相当于跳过这个属性，实际就会隐藏这个属性
        return undefined
    }

    return value
})

console.log(typeof jsonObj)    // object
console.log(jsonObj)        // {name: '赤蓝紫', nowTime: 2022}
```



用处: 比如可以把日期字符串转换为`Date`对象

```js
let person = {
    name: "clz",
    age: 21,
    nowTime: new Date(2022, 11, 31) // 月：0代表1月，11代表12月
};

const jsonStr = JSON.stringify(person)

const jsonObj = JSON.parse(jsonStr)

console.log(jsonObj.nowTime)        // 2022-12-30T16:00:00.000Z
console.log(typeof jsonObj.nowTime) // string
```

如果不进行任何操作，上面的`Date对象`转换成字符串，再转换成对象，会变成日期字符串了。而且，日期还减了一天。



如果我们想要还原成`Date对象`，并得到正确的时间，可以使用第二个参数来实现，只需要用该日期字符串实例化一个日期对象，再返回就行。

```js
let person = {
    name: "clz",
    age: 21,
    nowTime: new Date(2022, 11, 31) // 月：0代表1月，11代表12月
};

const jsonStr = JSON.stringify(person)

const jsonObj = JSON.parse(jsonStr, (key, value) => {
    if (key === 'nowTime') {
        return new Date(value)
    }

    return value
})

console.log(jsonObj.nowTime)        // Sat Dec 31 2022 00:00:00 GMT+0800 (中国标准时间)
console.log(typeof jsonObj.nowTime) // object
```
