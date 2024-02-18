---
title: Vue源码之虚拟DOM和diff算法(一)    使用snabbdom
categories: 前端
date: 2022-04-04 14:20:15
tags:
  - Vue源码
---

# Vue源码之虚拟DOM和diff算法(一)    使用snabbdom

## 什么是虚拟DOM和diff算法

### diff算法简介

![image-20220317115218615](https://s2.loli.net/2022/04/04/TP8AxVESywIU1WK.png)

要把左图装修成右图的样子。(哪里不同？仔细找)

有两种方案。

方案一：拆掉重建(效率低，代价大)

方案二：diff(精细化比对，最小量更新)

<br>

怎么看都应该会选择方案二。

<br>

那么在Vue中使用` diff`的情景呢?

![image-20220317121207880](https://s2.loli.net/2022/04/04/hJZGpIsD7AWLb8C.png)

上图就是在Vue中使用` diff`的情景(比如左图中，有一些元素的` v-if`为`false`，所以不显示，而右图中，` v-if`为` true`)

<br>

### 虚拟DOM简介

 **虚拟DOM：用来描述DOM的层次结构的js对象。真实DOM中的一切属性在虚拟DOM中都存在。**

![image-20220317121849612](https://s2.loli.net/2022/04/04/C6zu4QEKBUxIikg.png)

****

**diff是发生在虚拟DOM上的**

![image-20220317154226660](https://s2.loli.net/2022/04/04/7ihfAjSg4EaNLI6.png)

****

**优点：**

* 减少对真实DOM的操作
* 虚拟 DOM 本质上是 JavaScript 对象，可以跨平台，比如服务器渲染等

<br>

## snabbdom

[snabbdom仓库](https://github.com/snabbdom/snabbdom)

snabbdom是著名的虚拟DOM库，是`diff`算法的鼻祖。(Vue源码也借鉴了` snabbdom`)

### 安装

` npm install snabbdom`

### webpack配置

上一篇中，有` webpack`配置可查看Vue源码系列的上一篇文章。

**webpack.config.js**

```js
const path = require('path');

module.exports = {
  entry: path.join(__dirname, 'src', 'index.js'),
  mode: 'development',
  output: {
    filename: 'bundle.js',
    // 虚拟打包路径，bundle.js文件没有真正的生成
    publicPath: "/virtual/"
  },

  devServer: {
    // 静态文件根目录
    static: path.join(__dirname, 'www'),
    // 不压缩
    compress: false,
    port: 8080,
  }
}
```

<br>

## h函数使用

**h函数用来创建虚拟节点(vnode)**

![image-20220317154552800](https://s2.loli.net/2022/04/04/5UikzM3OL9n1GH6.png)

****

参数介绍：

* 第一个参数：是生成的虚拟节点对应DOM节点的标签名
* 第二个参数：一个对象，虚拟节点的属性(可选)
* 第三个参数：标签中的内容

****

### h函数体验

```js
import {
  init,
  classModule,
  propsModule,
  styleModule,
  eventListenersModule,
  h,
} from "snabbdom";


const myVnode = h('a', {
  props: {
    href: 'https://clz.vercel.app'
  }
}, '赤蓝紫')

console.log(myVnode)
```

![image-20220317155922806](https://s2.loli.net/2022/04/04/y7Dh4Fm3ujp6wHO.png)

<br>

**虚拟DOM节点属性介绍**：

* children: 子元素，没有则为` undefined`
* data(对象形式): 类名、属性、样式、事件(对象形式)
* elm: 对应的真实DOM节点(如果没有对应的，则为`undefined` )
* key：唯一标识
* sel：选择器
* text：文字

<br>

### 搭配` patch`函数生成真实DOM节点

通过引入的` init`函数把所有的模块(类名模块、属性模块、样式模块、事件监听模块)作为参数(少的话，则上树后也会少，比如少事件监听模块，上树后，事件将不再生效)

```js
const patch = init([classModule, propsModule, styleModule, eventListenersModule])
```

<br>

**`container`只是占位符，上树后会消失**

```js
// container只是占位符，上树后会消失
const container = document.getElementById('container')
patch(container, myVnode)	// 上树(第一个参数如果不是虚拟节点，则是执行上树操作，否则是diff算法，第一个参数是旧虚拟节点，第二个参数是新虚拟节点)
```

<br>

**完整版**

```js
import {
  init,
  classModule,
  propsModule,
  styleModule,
  eventListenersModule,
  h,
} from "snabbdom";

// 加载模块，，创建出patch函数。没有对应模块的话，上树后，也对应没有。比如少事件监听模块，上树后，事件将不再生效
// 类名模块、属性模块、样式模块、事件监听模块
const patch = init([classModule, propsModule, styleModule, eventListenersModule])


const myVnode = h('button', {
  class: {            // 类名
    "btn": true
  },
  props: {            // 属性, id也在这里面
    id: 'btn',
    title: '赤蓝紫'
  },
  style: {          // 样式
    backgroundColor: 'red',
    border: 0,
    color: '#fff',
  },
  on: {             // 事件监听
    click: function () {
      location.assign('https://clz.vercel.app')
    }
  }
}, '赤蓝紫')

console.log(myVnode)

// container只是占位符，上树后会消失
const container = document.getElementById('container')
patch(container, myVnode)     // 上树
```

<br>

### h函数嵌套使用

h函数可以嵌套使用，从而得到虚拟DOM树

![image-20220317174152025](https://s2.loli.net/2022/03/17/pTQr1R8MBdLHCPG.png)

<br>

**动手实践**

```js
import {
  init,
  classModule,
  propsModule,
  styleModule,
  eventListenersModule,
  h,
} from "snabbdom";


const patch = init([classModule, propsModule, styleModule, eventListenersModule])


const myVnode = h('ul', [     // 没有数据参数
  h('li', '赤'),        // 可以没有数据参数
  h('li', {}, '蓝'),    // 数据参数可为空
  h('li', h('span', '紫'))     // 内容参数为调用h函数且只有一个时，可以不是数组形式
])

console.log(myVnode)


const container = document.getElementById('container')
patch(container, myVnode)
```

![image-20220317174911551](https://s2.loli.net/2022/03/17/H5K9Crs8dzv6Ryx.png)

<br>

## 手写h函数

### 编写vnode函数

` vnode`功能：把传入的参数组合成对象后返回

src \ mysnabbdom \ vnode.js

```js
export default function (sel, data, children, text, elm) {
  return {
    sel,
    data,
    children,
    text,
    elm
  }
}
```

<br>

 本次手写的是低配版本的h函数，必须接收3个参数

调用可以有以下三种形式

1. h('div', {}, '文字')

2. h('div', {}, [])

3. h('div', {}, h())

<br>

### 实现第一种形式

直接调用` vnode`函数，返回即可

src \ mysnabbdom \ h.js

```js
import vnode from './vnode.js'

export default function (sel, data, content) {
  if (arguments.length !== 3) {
    throw new Error('这是低配版h函数, 必须接收3个参数')
  } else if (typeof content === 'string' || typeof content === 'number') {
    // 形式1
    return vnode(sel, data, undefined, content, undefined)
  } else if (Array.isArray(content)) {
    // 形式2
  } else if (typeof c === 'object' && c.hasOwnProperty('sel')) {
    // 形式3：因为h函数最终会返回一个一定会有`sel`属性的对象

  } else {
    throw new Error('传入的第三个参数类型不对')
  }
}
```

<br>

测试

src \ index.js

```js
import h from './mysnabbdom/h.js'

console.log(h('div', {}, '文字'))
```

![image-20220317230452293](https://s2.loli.net/2022/03/17/KM3PFd8Ea69R1rZ.png)

<br>

### 实现第二种形式

src \ mysnabbdom \ h.js

```js
import vnode from './vnode.js'


export default function (sel, data, content) {
  if (arguments.length !== 3) {
    throw new Error('这是低配版h函数, 必须接收3个参数')
  } else if (typeof content === 'string' || typeof content === 'number') {
    // 形式1
    return vnode(sel, data, undefined, content, undefined)
  } else if (Array.isArray(content)) {
    // 形式2
    const children = []

    for (let i = 0; i < content.length; i++) {
      if (!(typeof content[i] === 'object' && content[i].hasOwnProperty('sel'))) {
        throw new Error('传入的数组中又有不是调用h函数的')
      }

      children.push(content[i])      // 因为传的数组里的元素就是调用h函数的，即得到的已经是处理过后返回的对象了
    }

    return vnode(sel, data, children, undefined, undefined)
  } else if (typeof content === 'object' && content.hasOwnProperty('sel')) {
    // 形式3：因为h函数最终会返回一个一定会有`sel`属性的对象

  } else {
    throw new Error('传入的第三个参数类型不对')
  }
}
```

<br>

测试

src \ index.js

```js
import h from './mysnabbdom/h.js'

console.log(h('div', {}, [
  h('p', {}, '赤'),
  h('p', {}, '蓝'),
  h('p', {}, '紫'),
  h('p', {}, [
    h('span', {}, '黑'),
    h('span', {}, '白')
  ])
])) 
```

![image-20220317230809710](https://s2.loli.net/2022/03/17/TX2K8VIrhmfz4uG.png)  

### 第三种形式

src \ mysnabbdom \ h.js

```js
/*
 * 低配版本的h函数，必须接收3个参数
 * 调用可以有以下三种形式
 * 1. h('div', {}, '文字')
 * 2. h('div', {}, [])
 * 3. h('div', {}, h())
*/

import { h } from 'snabbdom'
import vnode from './vnode.js'


export default function (sel, data, content) {
  if (arguments.length !== 3) {
    throw new Error('这是低配版h函数, 必须接收3个参数')
  } else if (typeof content === 'string' || typeof content === 'number') {
    // 形式1
    return vnode(sel, data, undefined, content, undefined)
  } else if (Array.isArray(content)) {
    // 形式2
    const children = []

    for (let i = 0; i < content.length; i++) {
      if (!(typeof content[i] === 'object' && content[i].hasOwnProperty('sel'))) {
        throw new Error('传入的数组中又有不是调用h函数的')
      }

      children.push(content[i])      // 因为传的数组里的元素就是调用h函数的，即得到的已经是处理过后返回的对象了
    }

    return vnode(sel, data, children, undefined, undefined)
  } else if (typeof content === 'object' && content.hasOwnProperty('sel')) {
    // 形式3：因为h函数最终会返回一个一定会有`sel`属性的对象

    return vnode(sel, data, [content], undefined, undefined)    //把它当成只有一个元素的数组来处理

  } else {
    throw new Error('传入的第三个参数类型不对')
  }
}
```



测试

src \ index.js

```js
import h from './mysnabbdom/h.js'

console.log(h('div', {}, h('span', {}, '赤蓝紫')))
```

![image-20220317232336683](https://s2.loli.net/2022/03/17/DG3LTSUI5rY2fnx.png)  

## diff算法初体验

### 在后面插入

```js
import {
  init,
  classModule,
  propsModule,
  styleModule,
  eventListenersModule,
  h,
} from "snabbdom";

// 加载模块，创建出patch函数。没有对应模块的话，上树后，也对应没有。比如少事件监听模块，上树后，事件将不再生效
// 类名模块、属性模块、样式模块、事件监听模块
const patch = init([classModule, propsModule, styleModule, eventListenersModule])


const myVnode1 = h('ul', {}, [
  h('li', {}, 'a'),
  h('li', {}, 'b'),
  h('li', {}, 'c'),
  h('li', {}, 'd'),
])
const myVnode2 = h('ul', {}, [
  h('li', {}, 'a'),
  h('li', {}, 'b'),
  h('li', {}, 'c'),
  h('li', {}, 'd'),
  h('li', {}, 'e')
])

const container = document.getElementById('container')
patch(container, myVnode1)     // 上树
const btn = document.getElementById('btn')
btn.addEventListener('click', function () {
  patch(myVnode1, myVnode2)     // 点击后，从myVnode1变为myVnode2
})
```

![diff](https://s2.loli.net/2022/03/18/AkMaWPzLBOCnTYg.gif)

<br>

怎么知道是不是真的是最小量更新呢?

可以用老师用的巧妙法：在` DevTools`的` Elements`面板修改内容，查看有没有变化

![diff](https://s2.loli.net/2022/03/18/DN6x41u3nfImF8y.gif)

可以发现，确确实实是最小量更新。仔细看上面的图，发现不需要修改` Elements`面板，有更新的话，会变紫，闪烁一下

<br>

### 在前面插入

那么，接下来就试一下在开头加入新节点的情况咯

```js
const myVnode2 = h('ul', {}, [
  h('li', {}, 'e'),
  h('li', {}, 'a'),
  h('li', {}, 'b'),
  h('li', {}, 'c'),
  h('li', {}, 'd')
])
```

![diff](https://s2.loli.net/2022/03/18/U8oeIQZSaEKYgb6.gif)

可以说是，上面的情况压根就不是最小量更新。

这是为什么呢？这时候就需要` key`的闪亮登场了

**没有key的时候**：会先把节点插到最后，再把插入的节点移动到要去的位置，其他节点也需要移动到要去的位置

<br>

### 在中间插入

可以再来测试一下

```js
const myVnode2 = h('ul', {}, [
  h('li', {}, 'a'),
  h('li', {}, 'e'),
  h('li', {}, 'b'),
  h('li', {}, 'c'),
  h('li', {}, 'd')
])
```

这时候，只有a不会变化，因为e插入的位置不会影响到a

<br>

****

### 使用key，真正实现最小量更新

有` key`的时候，就不一样了，每一个虚拟节点都有一个唯一标识，所以能够精准定位，真正实现最小化更新

```js
const myVnode1 = h('ul', {}, [
  h('li', { key: 'a' }, 'a'),
  h('li', { key: 'b' }, 'b'),
  h('li', { key: 'c' }, 'c'),
  h('li', { key: 'd' }, 'd'),
])
const myVnode2 = h('ul', {}, [
  h('li', { key: 'e' }, 'e'),
  h('li', { key: 'a' }, 'a'),
  h('li', { key: 'b' }, 'b'),
  h('li', { key: 'c' }, 'c'),
  h('li', { key: 'd' }, 'd')
])
```

![diff](https://s2.loli.net/2022/03/18/bwVOU5AZWy9CEa8.gif)

<br>

### 使用key，并完全调换位置

```js
const myVnode1 = h('ul', {}, [
  h('li', { key: 'a' }, 'a'),
  h('li', { key: 'b' }, 'b'),
  h('li', { key: 'c' }, 'c'),
  h('li', { key: 'd' }, 'd'),
])
const myVnode2 = h('ul', {}, [
  h('li', { key: 'd' }, 'd'),
  h('li', { key: 'a' }, 'a'),
  h('li', { key: 'e' }, 'e'),
  h('li', { key: 'b' }, 'b'),
  h('li', { key: 'c' }, 'c'),
])
```

![diff](https://s2.loli.net/2022/03/18/I4ucHBbE25iGv8A.gif)

还是最小量更新。另外，闪烁法还是不太可靠，建议还是修改Element法

<br>

### 总结

* **最小量更新**：需要`key`，` key`是节点的唯一标识，用于告诉` diff`算法，在更改前后是同一个DOM节点

* **只有是同一个虚拟节点，才会进行精细化比较**，否则就是暴力删除旧的、插入新的。如上面的例子中，从` ul`变为` ol`

  **同一个虚拟节点**：选择器相同且` key`相同

  > ```js
  > // 供测试用：可以使用回上面说的修改Elemens面板法(不过，下面的例子实际开发遇到的可能性很小)
  > 
  > const myVnode1 = h('ul', { key: 'ul1' }, [
  > h('li', { key: 'a' }, 'a'),
  > h('li', { key: 'b' }, 'b'),
  > h('li', { key: 'c' }, 'c'),
  > h('li', { key: 'd' }, 'd'),
  > ])
  > const myVnode2 = h('ul', { key: 'ul2' }, [
  > h('li', { key: 'e' }, 'e'),
  > h('li', { key: 'a' }, 'a'),
  > h('li', { key: 'b' }, 'b'),
  > h('li', { key: 'c' }, 'c'),
  > h('li', { key: 'd' }, 'd')
  > ])
  > ```

* **只进行同层比较，不进行跨层比较**。如果跨层了，则依然是暴力删除旧的，然后插入新的

  ```js
  // 下面的例子实际开发遇到的可能性很小
  const myVnode1 = h('div', { key: 'box' }, [
    h('p', { key: 'a' }, 'a'),
    h('p', { key: 'b' }, 'b'),
    h('p', { key: 'c' }, 'c'),
    h('p', { key: 'd' }, 'd'),
  ])
  const myVnode2 = h('div', { key: 'box' },
    h('section', { key: 'section' }, [
      h('p', { key: 'e' }, 'e'),
      h('p', { key: 'a' }, 'a'),
      h('p', { key: 'b' }, 'b'),
      h('p', { key: 'c' }, 'c'),
      h('p', { key: 'd' }, 'd')
    ])
  )
  ```

  ![diff](https://s2.loli.net/2022/03/18/4QUgJWP5nlZwrcb.gif)









