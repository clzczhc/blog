---
title: 【源码共读】Vue2工具函数
categories: 前端
date: 2022-07-25 10:05:55
tags:
    - Vue.
    - 源码共读
---

# 【源码共读】Vue2工具函数

**本文参加了由**[公众号@若川视野](https://link.juejin.cn/?target=https%3A%2F%2Flxchuan12.gitee.io "https://lxchuan12.gitee.io") **发起的每周源码共读活动，** [点击了解详情一起参与。](https://juejin.cn/post/7079706017579139102 "https://juejin.cn/post/7079706017579139102")

**这是源码共读的[第24期](https://juejin.cn/post/7079765637437849614)**

## 前言

github仓库地址 [在线地址](https://github1s.com/vuejs/vue/blob/dev/src/shared/util.js)

点击在线地址查看，会发现该文件实际上有很多函数。实际上就是Vue2的工具函数库。下面就来简单学习一下。因为源码用的是ts，理解起来可能会加点成本，所以下面讲解会把类型部分去掉(其实是本人的ts水平不高，很难很好的解释)

## 工具函数

### 1. Object.freeze({})

```js
const emptyObject = Object.freeze({});
```

`Object.freeze()`方法可以冻结一个对象。如果对象被冻结后，就不能再被修改、不能添加新的属性、不能删除已有属性、不能修改已有属性的配置(可枚举性、可写性等)。

```js
const freezeObj = Object.freeze({
  name: 'clz'
});

console.log(freezeObj);   // { name: 'clz' }

// 因为被冻结了，所以修改属性、删除属性都没有效果
freezeObj.age = 21;
console.log(freezeObj);   // { name: 'clz' }

delete freezeObj.name;
console.log(freezeObj);   // { name: 'clz' }
```

**冻结对象后，只是第一层无法修改，第二层还是能够修改滴**

```js
const freezeObj = Object.freeze({
  job: {
    type: 'Coder',
    salary: -111
  }
});

console.log(freezeObj);   // { job: { type: 'Coder', salary: -111 } }

freezeObj.job.salary = 111;
console.log(freezeObj);   // { job: { type: 'Coder', salary: 111 } }

delete freezeObj.job.salary;
console.log(freezeObj);   // { job: { type: 'Coder' } }

// 冻结只会影响第一层。所以给第一层添加属性还是会没有效果
freezeObj.name = 'clz';
console.log(freezeObj);   // { job: { type: 'Coder' } }
```

我们可以通过`Object.isFrozen`来判断对象是不是冻结状态。

```js
const freezeObj = Object.freeze({
  name: 'clz'
});

console.log(Object.isFrozen(freezeObj));  // true
console.log(Object.isFrozen({}));   // false
```

那么这个工具库的这个不是函数的变量有什么作用呢？

我知道的场景就是通过赋值冻结的空对象，防止不小心添加属性等操作，比较程序员有可能手误。(这里想要感谢一下若川大佬，评论问问题，很耐心地解答)

### 2. 判断系列

#### 2.1 isUndef

判断是不是没有定义。

```js
function isUndef(v) {
  return v === undefined || v === null
}
```

**注意**：这个函数名是叫`isUndef`，但是实际上，如果参数是`null`的话，因为`null`也没有什么实际意义，所以把`null`和`undefined`捆绑了(个人感觉此处命名有点点瑕疵)。

#### 2.2 isDef

判断是不是已经定义了。

```js
function isDef(v) {
  return v !== undefined && v !== null
}
```

这里其实就只是上面的取反就行了。因为定义和未定义本就是相反的。

#### 2.3 isTrue

判断是不是`true`。

```js
function isTrue(v) {
  return v === true
} 
```

#### 2.4 isFalse

判断是不是`false`。

```js
function isTrue(v) {
  return v === true
}
```

#### 2.5 isPrimitive

判断是不是原始值，即原始类型的值。

```js
function isPrimitive(value) {
  return (
    typeof value === 'string' ||
    typeof value === 'number' ||
    // $flow-disable-line
    typeof value === 'symbol' ||
    typeof value === 'boolean'
  )
}
```

JS的基础类型有：

- `string`

- `number`

- `boolean`

- `undefined`

- `null`

- `Symbol`

- `BigInt`

`undefined`和`null`已经在前面的未定义、已定义那里切出去了，而`BigInt`是`ES2020`新增的基本类型。这里的`isPrimitive`更像是判断是不是有用的原始类型。

#### 2.6 isObject

判断是不是对象。

```js
function isObject(obj) {
  return obj !== null && typeof obj === 'object'
}
```

因为`typeof null`的结果也是`object`，所以还需要确定不是`null`才行。

这里简单讲一下原因：JS存储数据用的是二进制存储，数据的前三位是存储的类型。对象的前三位是`000`，而`null`则是全为`0`，即前三位也是`000`。所以`typeof null`也是`object`。

#### 2.7 isPlainObject

判断是不是纯对象。

```js
function isPlainObject(obj) {
  return _toString.call(obj) === '[object Object]'
}
```

上面的`_toString`实际上是`Object.prototype.toString`，因为比较常用，所以处理成`_toString`变量。

`isObject`只是判断是不是对象，但是实际上用来判断数组也能得到`true`的结果，因为数组也是对象。所以还得有一个判断是不是纯对象的方法。而判断的方法也比较简单，只需要调用`Object.prototype.toString`方法即可(要使用`bind`方法来调用)。如果是数组得到的结果会是`'[object Array]'`，而纯对象得到的结果是`'[object Object]'`。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207031456939.png)

#### 2.8 isRegExp

判断是不是正则表达式

```js
function isRegExp(v) {
  return _toString.call(v) === '[object RegExp]'
}
```

#### 2.9 isValidArrayIndex

判断是不是有效的数组索引值。

```js
function isValidArrayIndex(val) {
  const n = parseFloat(String(val))
  return n >= 0 && Math.floor(n) === n && isFinite(val)
}
```

第一步是将参数变成字符串，第二步是转成浮点数。(第一行)

第二行是重点：

1. `n >= 0`，因为数组索引值不能是负数

2. `Math.floor(n) === n`，因为数组索引值不能是负数

3. `isFinite(val`，参数只能是有限数值。在必要情况下，参数会转换称为数值。

下面稍微举几个例子方便理解`isFinite`。其实也没啥好说的，就是限制参数只能是有限数值。顺带提一嘴，实际上第一行已经和前两个条件已经能过滤掉一些情况了，因为像是`NaN`，`true`这些参数，转成字符串再转成浮点数就已经是`NaN`了。

```js
console.log(isFinite(Infinity));    // false
console.log(isFinite(NaN));         // false
console.log(isFinite(-Infinity));   // false
console.log(isFinite('12345a'));    // false

console.log(isFinite('0'));         // true
console.log(isFinite('123'));       // true
console.log(isFinite(123));         // true
console.log(isFinite(-123));        // true
console.log(isFinite(true));        // true

console.log(isFinite(0b101));       // true
console.log(isFinite(0xFF));        // true

console.log(isFinite(1.234))        // true
```

#### 2.10 isPromise

判断是不是Promise对象。这个方法非常有效，并且有意思。判断是不是Promise并不是直接判断，而是通过判断它的`then`和`catch`属性是不是函数来间接判断是不是Promise对象

```js
function isPromise() {
  return (
    isDef(val) &&
    typeof val.then === 'function' &&
    typeof val.catch === 'function'
  )
}
```

### 3. 转换系列

#### 3.1 toRawType

转换成原始类型(得到参数的类型)

```js
function toRawType(value) {
  return _toString.call(value).slice(8, -1)
}
```

其实就是借助前面提到过的`Object.prototype.call(value)`能得到形似`[object Type]`的字符串。比较精确，如数组也是对象，通过这个方法能得到是数组，而不只是对象。然后通过`slice(8, -1)`把参数的类型部分拿到。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207042321474.png)

#### 3.2 toString

转换成字符串。

```js
function toString(val) {
  return val == null
    ? ''
    : Array.isArray(val) || (isPlainObject(val) && val.toString === _toString)
      ? JSON.stringify(val, null, 2)
      : String(val)
}
```

首先，原始类型通过`String()`方法就能直接转换成对应的字符串，但是`undefined`和`null`转换成字符串应该是空串才更合理，所以上面用了普通的`==`来判断是不是这两个值，如果是，则返回空串。

但是，当值是对象、数组的时候，结果会有点差强人意。

```js
const obj = {
  name: 'clz'
};

const arr = [1, 2, 3, 4];

console.log(String(obj));   // [object Object]
console.log(String(arr));   // 1,2,3,4
```

这时候就需要使用`JSON.stringify()`来将它们转换成对象了，可以看一下之前写的笔记[JSON的使用之灵活版 | 赤蓝紫](https://www.clzczh.top/2022/06/18/JSON%E7%9A%84%E4%BD%BF%E7%94%A8%E4%B9%8B%E7%81%B5%E6%B4%BB%E7%89%88)

```js
const obj = {
  name: 'clz'
};

const arr = [1, 2, 3, 4];

console.log(JSON.stringify(obj));   // {"name":"clz"}
console.log(JSON.stringify(arr));   // [1,2,3,4]
```

至于源码中的第三个参数，其实就只是指定缩进的空格是2个，用于美化输出的。

#### 3.3 toNumber

转换成数字型，如果没法转换成数字型就返回原字符串。该方法参数只能是字符串类型。

```js
function toNumber(val) {
  const n = parseFloat(val)
  return isNaN(n) ? val : n
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207052203999.png)

注意，由于`parseFloat`只要参数字符串的第一个字符能被解析成数字，就会返回数字，即使后面不是数字也一样，如上面例子的`123a`。

另外`Infinity`也能被解析并返回`Infinity`。

#### 3.4 toArray

将伪数组转换成真数组。第二个参数可选，可以控制真数组的开始位置，默认是0。

```js
function toArray (list, start) {
  start = start || 0
  let i = list.length - start
  const ret = new Array(i)
  while (i--) {
    ret[i] = list[i + start]
  }
  return ret
}
```

伪数组具有`length`属性，但是不具备数组的`push`、`forEach`等方法。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207091133085.png)

#### 3.5 extend

扩展对象，把一个对象的属性值扩展到另一个对象上。放在这里主要是后面的`toObject`会使用到。

```js
function extend(to, _from) {
  for (const key in _from) {
    to[key] = _from[key]
  }
  return to
}
```

注意，如果`to`上也有`_from`的属性，那么`_from`的该属性值会覆盖掉`to`上的。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207091420573.png)

#### 3.6 toObject

将一个对象数组合并到另一个对象中去。

```js
function toObject(arr) {
  const res = {}
  for (let i = 0; i < arr.length; i++) {
    if (arr[i]) {
      extend(res, arr[i])
    }
  }
  return res
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207091444691.png)

注意：上面例子中，最后生成的对象不存在之前的`456`，这是因为`456`不能被`for in`遍历，而`Hello`能被遍历，只是会被拆解。不过，该方法用法应该只是将数组里的对象合并到另一个对象中去(从注释猜测的)

### 4. makeMap系列

主要介绍`makeMap`方法以及使用`makeMap`方法的。

#### 4.1 makeMap

生成一个`map`，注意：这里的`map`只是键值对形式的对象。并且返回的并不是生成的`map`，而是一个函数，用来判断`key`在不在`map`中的对象。具体可以实例可以查看下面的`isBuiltInTag`的。`expectsLowerCase`是可选参数，表示会将字符串参数变为小写，即不区分大小写。

```js
function makeMap (
  str,
  expectsLowerCase
) {
  const map = Object.create(null)
  const list = str.split(',')
  for (let i = 0; i < list.length; i++) {
    map[list[i]] = true
  }
  return expectsLowerCase
    ? val => map[val.toLowerCase()]
    : val => map[val]
}
```

这个方法并没有很复杂。简单讲一下步骤：

1. `Object.create(null)`生成没有原型链的空对象

2. `str.split(',')`把字符串以`,`为分隔符，将字符串分割为字符串数组

3. 遍历分割的数组，以子字符串为`key`，以`true`为值添加到`map`中，表示该`key`在生成的`map`中。

4. 返回一个函数，判断`key`在不在`map`中。这里会判断第二个参数是不是`true`，如果是，则不区分大小写。

#### 4.2 isBuiltInTag

判断是不是内置的`tag`(这里的内置并不是`html`的标签，而是Vue的`slot`和`component`)。用的是上面的`makeMap`方法。

```js
const isBuiltInTag = makeMap('slot,component', true)
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207062055853.png)

上面第二个参数为`true`，表示不区分大小写，也可以不传，从而区分大小写。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207062057604.png)

#### 4.3 isReservedAttribute

判断是不是保留的属性。还是通过`makeMap`方法来实现，和`isBuiltInTag`原理一样，就不再介绍了。

```js
const isReservedAttribute = makeMap('key,ref,slot,slot-scope,is')
```

### 5. remove

从数组中删除指定元素。如果有多个指定元素，只删除第一个。

```js
function remove(arr, item) {
  if (arr.length) {
    const index = arr.indexOf(item)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}
```

原理就是通过`indexOf`找到要删除的元素的位置，然后通过`splice()`方法删除掉该元素。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207062138092.png)

### 6. hasOwn

判断是不是自己的属性，而不是继承过来的。

其实通过`obj.hasOwnProperty(key)`好像就可以了，但是还是封装成了一个方法。本质应该和`isTrue`一样，更适合大型项目的开发，比如代码易读性之类的。(猜的，没做过很大型的项目，泪目)

```js
const hasOwnProperty = Object.prototype.hasOwnProperty
function hasOwn(obj, key) {
  return hasOwnProperty.call(obj, key)
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207062203610.png)

### 7. cached

利用闭包的特性，缓存数据。

```js
function cached(fn) {
  const cache = Object.create(null)
  return (function cachedFn(str) {
    const hit = cache[str]
    return hit || (cache[str] = fn(str))
  })
}
```

接受一个函数，返回一个函数，返回的函数会判断有没有缓存数据，如果有，则直接返回缓存数据，如果没有，才会调用传入的函数，并且会缓存数据。

### 8. 字符转换系列

#### 8.1 camelize

连字符转驼峰，如`on-click`转成`onClick`

```js
const camelizeRE = /-(\w)/g
const camelize = cached(str => {
  return str.replace(camelizeRE, (_, c) => c ? c.toUpperCase() : '')
})
```

上面用了正则表达式比较巧妙的实现：

1. `/-(\w)/g`会匹配像是`-click`之类的

2. `replace()`替换掉匹配的部分，其中，第二个参数可以是函数，在上面的例子中，该函数的第二个参数代表第1个括号匹配的字符串，即`click`，然后让它首字母大写。如果没有匹配到的，则不变化。

3. **注意**：上面用了缓存方法`cached`来优化。

#### 8.2 capitalize

首字母大写。

```js
const capitalize = cached(str => {
  return str.charAt(0).toUpperCase() + str.slice(1)
})
```

#### 8.3 hyphenate

驼峰转连字符，如`onClick`转成 `on-click`

```js
const hyphenateRE = /\B([A-Z])/g
const hyphenate = cached(str => {
  return str.replace(hyphenateRE, '-$1').toLowerCase()
})
```

这个原理其实和上面的连字符一样，还简单了一点，因为都是小写字母。

原理就是通过正则表达式去匹配字符串，使用括号和`$1`实现将匹配到的括号内的字符串变成添加上`-`的形式。最后再将整个字符串小写化。

那么`\B`有什么用呢？

`\B`元字符匹配非单词边界。匹配位置的上一个和下一个字符的类型是相同的，即必须同时是单词，或同时是非单词字符。字符串的开头和结尾处被视为非单词字符。

所以当大写字母在字符串开头时不会被转化，即`Onclick`不会变成`-onclick`。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207091013770.png)

顺便看下没有`\B`元字符的情况。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207091022261.png)

### 9. noop

```js
function noop(a, b, c) {}   // 参数都是可选的
```

第一反应：？？？

后面查了下资料：`noop`的主要作用是为一些函数提供默认值，避免传入`undefined`之类的数据导致代码出错。即如果参数原本是函数，但是最后传了`undefined`的话，就会报`xx is not a function`的错。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207091526816.png)

### 10. no

永假函数。

```js
const no = (a, b, c) => false    // 参数都是可选的
```

### 11. identity

返回自身。

```js
const identity = (_) => _ 
```

### 12. genStaticKeys

生成静态键的字符串。

```js
function genStaticKeys(modules) {
  return modules.reduce((keys, m) => {
    return keys.concat(m.staticKeys || [])
  }, []).join(',')
}
```

接收一个对象数组，将`staticKeys`的值(数组)，拼接成静态键的数组，最后将该数组转化成字符串形式，用`,`连接。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207091628039.png)

### 13. looseEqual

宽松相等：两个对象(包括数组)比较，如果它们形状相同，就返回`true`。

`{}`和另一个`{}`是不相等的，因为对象是引用类型，但是用`looseEqual`来判断是会认为相等的，因为形状(内容)完全相同。

```js
function looseEqual (a: any, b: any): boolean {
  if (a === b) return true
  const isObjectA = isObject(a)
  const isObjectB = isObject(b)
  if (isObjectA && isObjectB) {
    try {
      const isArrayA = Array.isArray(a)
      const isArrayB = Array.isArray(b)
      if (isArrayA && isArrayB) {
        return a.length === b.length && a.every((e, i) => {
          return looseEqual(e, b[i])
        })
      } else if (a instanceof Date && b instanceof Date) {
        return a.getTime() === b.getTime()
      } else if (!isArrayA && !isArrayB) {
        const keysA = Object.keys(a)
        const keysB = Object.keys(b)
        return keysA.length === keysB.length && keysA.every(key => {
          return looseEqual(a[key], b[key])
        })
      } else {
        /* istanbul ignore next */
        return false
      }
    } catch (e) {
      /* istanbul ignore next */
      return false
    }
  } else if (!isObjectA && !isObjectB) {
    return String(a) === String(b)
  } else {
    return false
  }
}
```

流程：

1. `a`严格等于`b`直接返回`true`

2. 如果`a`和`b`都是对象(包括数组)，依次执行以下操作：
   
   1. 如果都是数组，判断数组长度是否相等，并通过`every`+`looseEqual`判断数组元素是否都宽松相等
   
   2. 如果都是`Date`对象，那就判断两者的绝对是件是否相同
   
   3. 如果两者都不是数组，那就分别获取`a`和`b`的键，判断数组长度是否相等，并通过`every`+`looseEqual`判断数组元素是否都宽松相等
   
   4. 上面都没有符合的话，就返回`false`。

3. 如果都不是对象，则比较它们转换为字符串后是否严格相等。

4. 最后返回`false`，此时是`a`和`b`一个是对象，一个不是对象，所以肯定不等。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207091836715.png)

### 14. looseIndexOf

返回数组中第一个与参数宽松相等的元素的位置。原生的`indexOf`是严格相等。

```js
function looseIndexOf(arr, val) {
  for (let i = 0; i < arr.length; i++) {
    if (looseEqual(arr[i], val)) return i
  }
  return -1
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207091841398.png)

### 15. once

确保函数只执行一次。

```js
function once(fn) {
  let called = false
  return function () {
    if (!called) {
      called = true
      fn.apply(this, arguments)
    }
  }
}
```

主要还是利用闭包缓存数据的特性，定义一个初始值为`false`的变量，返回一个函数，该函数会判断缓存的数据`called`是不是`false`，如果是，则将`called`变为`true`，并执行函数，通过`apply`调用来绑定上下文。

```js
function once(fn) {
  let called = false
  console.log(this);

  return function () {
    if (!called) {
      called = true
      console.log(this);
      fn.apply(this, arguments)
      // fn();
    }
  }
}

const obj = { age: 21 };
window.age = 100;

obj.fn = once(function () {
  console.log(this.age);
});

obj.fn();
```

### 16. polyfillBind

兼容老版本浏览器不支持原生的`bind`函数。并且会根据参数的多少来确定使用使用`apply`还是`call`，如果参数数量大于1，则使用`apply`，如果参数数量小于1，则使用`call`。

```js
function polyfillBind(fn, ctx) {
  function boundFn (a) {
    const l = arguments.length
    return l
      ? l > 1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)
      : fn.call(ctx)
  }

  boundFn._length = fn.length
  return boundFn
}

function nativeBind(fn, ctx) {
  return fn.bind(ctx)
}

export const bind = Function.prototype.bind
  ? nativeBind
  : polyfillBind
```

## 参考链接

- [github仓库在线地址](https://github1s.com/vuejs/vue/blob/dev/src/shared/util.js)

- [初学者也能看懂的 Vue2 源码中那些实用的基础工具函数 - 掘金](https://juejin.cn/post/7024276020731592741)

- MDN

- 小参考省略(不记得了)
