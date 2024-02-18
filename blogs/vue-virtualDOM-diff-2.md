---
title: Vue源码之虚拟DOM和diff算法(二)    手写diff算法
categories: 前端
date: 2022-04-04 14:22:41
tags:
  - Vue源码
---

# Vue源码之虚拟DOM和diff算法(二)    手写diff算法

个人练习结果仓库(持续更新)：[Vue源码解析](https://github.com/13535944743/Vue_Source_Code_Practise)

## patch函数简要流程

![image-20220318114622291](https://s2.loli.net/2022/04/23/E5qZJSoy4CzYtwA.png)

<br>

## 新旧节点不是同一个虚拟节点(新节点内容是` text`)

不做过多解释了，代码中已经把每一步都解释了

src \ mysnabbdom \ patch.js

```js
import vnode from './vnode.js'
import createElement from './createElement.js'

export default function (oldVnode, newVnode) {
  // 表示是真实DOM，真实DOM需要先转换成虚拟DOM后才进行下面的操作。因为真实DOM是没有sel这个属性的
  if (oldVnode.sel === '' || oldVnode.sel === undefined) {
    // 转换成虚拟DOM
    oldVnode = vnode(oldVnode.tagName.toLowerCase(), {}, [], '', oldVnode)
  }

  // 判断oldVnode和newVnode是不是同一个节点
  if (oldVnode.sel === newVnode.sel && oldVnode.key === newVnode.key) {
    // 是同一个虚拟节点，需要进行精细化比对，最小化更新
    console.log('需要精细化比对')
  } else {
    // 不是同一个虚拟节点，暴力删除旧的，插入新的

    const newDomNode = createElement(newVnode)     // 把新虚拟节点变成真实DOM

    const pivot = oldVnode.elm

    // 将新创建的孤儿节点上树
    pivot.parentNode.insertBefore(newDomNode, pivot)
    // 删除旧的
    pivot.parentNode.removeChild(pivot)
  }
}
```

<br>

如果` oldVnode`和` newVnode`不是同一个虚拟节点，那么就直接暴力删除旧的，插入新的。

所以需要一个函数` createElement`，它的功能是将新虚拟节点创建为DOM节点并返回。

<br>

src \ mysnabbdom \  createElement.js

```js
// 真正创建节点，将vnode创建为DOM

export default function (vnode) {
  // 创建一个DOM节点，此时还是孤儿节点
  const domNode = document.createElement(vnode.sel)

  if (vnode.text !== '' && (vnode.children === undefined || vnode.children.length === 0)) {
    // 内部是文字
    domNode.innerText = vnode.text
    vnode.elm = domNode
  }

  // 返回真实DOM对象
  return vnode.elm
}
```

<br>

测试

src \ index.js

```js
import h from './mysnabbdom/h.js'
import patch from './mysnabbdom/patch.js'

const myVnode1 = h('h2', {}, '文字')

// container只是占位符，上树后会消失
const container = document.getElementById('container')

patch(container, myVnode1)     // 上树
```

![image-20220318145123182](https://s2.loli.net/2022/04/04/hs9Knb345AB6MES.png)

<br>

**新旧虚拟节点不是同一个节点，都能实现上树(不只是第一次)**

```js
import h from './mysnabbdom/h.js'
import patch from './mysnabbdom/patch.js'


const myVnode1 = h('h2', {}, 'hello')

// container只是占位符，上树后会消失
const container = document.getElementById('container')

patch(container, myVnode1)     // 上树

const myVnode2 = h('h3', {}, 'hi')

patch(myVnode1, myVnode2)
```

![image-20220320152958956](https://s2.loli.net/2022/04/04/tgNlzY5FqxAS8Up.png)

<br>

## 新旧节点不是同一个虚拟节点(新节点内容是子节点)

src \ mysnabbdom \  createElement.js(部分)

```js
else if (Array.isArray(vnode.children) && vnode.children.length >= 0) {
  // 内部是子节点,需要递归创建子节点
  for (let i = 0; i < vnode.children.length; i++) {
    const childDomNode = createElement(vnode.children[i])   // 递归创建子节点

    // 创建的子节点需要添加到DOM节点里
    domNode.appendChild(childDomNode)
  }
}
```

<br>

测试

src \ index.js

```js
import h from './mysnabbdom/h.js'
import patch from './mysnabbdom/patch.js'


const myVnode1 = h('ul', {}, h('li', {}, [
  h('li', {}, '赤'),
  h('li', {}, '蓝'),
  h('li', {}, '紫'),
  h('li', {}, [
    h('div', {}, [
      h('ol', {}, [
        h('li', {}, '白'),
        h('li', {}, '黑')
      ])
    ])
  ]),
]
))

// container只是占位符，上树后会消失
const container = document.getElementById('container')

patch(container, myVnode1)     // 上树
```

![image-20220318160421847](https://s2.loli.net/2022/04/04/bDWmUx3TZsJR84c.png)

<br>



## patch函数精细化比对流程

![image-20220404140651575](https://s2.loli.net/2022/04/04/3gsT4NoezujpiaD.png)

<br>

## patch函数

**实现简单部分的精细化比对，即不包括流程图中星星部分**

```js
// 判断oldVnode和newVnode是不是同一个节点
if (oldVnode.sel === newVnode.sel && oldVnode.key === newVnode.key) {
  // 是同一个虚拟节点，需要进行精细化比对，最小化更新
  patchVnode(oldVnode, newVnode)
  if (oldVnode === newVnode) {
    // oldVnode和newVnode是内存上的同一对象，即完全相同，不做任何处理
    return
  }

  if (newVnode.text !== undefined && (newVnode.children === undefined || newVnode.children.length === 0)) {
    // newVnode的内容是text，而不是子节点
    if (newVnode.text === oldVnode.text) {
      return
    }

    oldVnode.elm.innerText = newVnode.text
  } else {
    // newVnode的内容是子节点
    if (oldVnode.children !== undefined && oldVnode.children.length > 0) {
      // newVnode的内容是子节点，oldVnode的内容也是子节点

      console.log('最复杂的情况')
    } else {
      // newVnode的内容是子节点，而oldVnode的内容是text：需要清空oldVnode，然后再把newVnode的children添加DOM上
      oldVnode.elm.innerText = ''

      for (let i = 0; i < newVnode.children.length; i++) {
        let domNode = createElement(newVnode.children[i])
        oldVnode.elm.append(domNode)
      }
    }
  }

}
```

<br>

**测试**

oldVnode和newVnode是内存中同一个对象

```js
const myVnode1 = h('h2', {}, 'hello')

// container只是占位符，上树后会消失
const container = document.getElementById('container')

patch(container, myVnode1)     // 上树

const myVnode2 = myVnode1

patch(myVnode1, myVnode2)
```

<br>

` newVnode`的内容是` text`，` oldVnode`的内容是子节点

```js
const myVnode1 = h('h2', {}, h('p', {}, 'hello'))

// container只是占位符，上树后会消失
const container = document.getElementById('container')

patch(container, myVnode1)     // 上树

const myVnode2 = h('h2', {}, 'hi')

patch(myVnode1, myVnode2)
```

<br>

`newVnode`和` oldVnode`的内容都是` text`(只写不同的，相同的自测)

```js
const myVnode1 = h('h2', {}, 'hello')

// container只是占位符，上树后会消失
const container = document.getElementById('container')

patch(container, myVnode1)     // 上树

const myVnode2 = h('h2', {}, 'hi')

patch(myVnode1, myVnode2)
```

<br>

` newVnode`的内容是子节点，` oldVnode`的内容是` text`

```js
const myVnode1 = h('section', {}, 'hello')

// container只是占位符，上树后会消失
const container = document.getElementById('container')

patch(container, myVnode1)     // 上树

const myVnode2 = h('section', {}, [
  h('p', {}, '赤'),
  h('p', {}, '蓝'),
  h('p', {}, '紫')
])
patch(myVnode1, myVnode2)
```

<br>

## diff的子节点更新策略

### 准备

1. 将精细化比对，最小化更新部分代码封装成函数` patchVnode`

2. 修改` vnode.js`文件，将data中的`key`取出来

   ```js
   export default function (sel, data, children, text, elm) {
     return {
       sel,
       data,
       children,
       text,
       elm,
       key: data.key
     }
   }
   ```

<br>

### 原理

![image-20220321093329463](https://s2.loli.net/2022/04/04/tQLGAk5iPlr1Ocj.png)

<br>

如上图所示，一共有四个指针，其中，旧前、旧后指向旧子节点的首尾，新前、新后指向新子节点的首尾。

**有四种命中查找：命中一种就不会再进行命中判断了。没有命中的话，则按箭头方向换一种命中查找方式**

![image-20220321093851898](https://s2.loli.net/2022/04/04/pyjT7KADSEGwCb8.png)

**规则：**

* `前指针`只能向下移动，`后指针`只能向上移动
* 当`前指针`在`后指针`下面时，循环完毕、(不包括在相同位置的情况)

<br>

#### 新增

<b style="color: red">为了简便，直接把子节点用一个字母来表示</b>

简单版本：

![image-20220321115109525](https://s2.loli.net/2022/03/21/VvKgDAT4fwesByR.png)

> **如果旧节点先循环完毕，则此时新前指针、新后指针范围内的节点是新增节点**(包括新前指针、新后指针指向的节点)

<br>

复杂版本：

![image-20220321150325730](https://s2.loli.net/2022/03/21/s1Vzk8ptXr2wRUN.png)

<br>

> **如果四种方式的查找都无法命中，则直接在旧子节点中寻找相同key的元素，不存在的话，新增并将该元素追加到`旧前指针`之前，`新前指针`下移**

<br>

#### 删除

![image-20220321150243560](https://s2.loli.net/2022/03/21/OyIpN57q8h2QYLJ.png)

<br>

#### 位置变换

![image-20220321155833724](https://s2.loli.net/2022/03/21/alzrCRy84DJLpSZ.png)

<br>

#### 增 + 删 + 位置变化

![image-20220321162253189](https://s2.loli.net/2022/03/21/fTLxJsXcowZriDu.png)

<br>

#### key一样，节点内容却不同的情况

[详解Vue的Diff算法(例6)](https://zhuanlan.zhihu.com/p/408728265)

<br>

### 原理总结

1. **新前旧前**：
   * 命中，`新前指针`、`旧前指针`下移，回到1，继续看有没有命中
   * 未命中，继续向下尝试命中
2. **新后旧后**：
   * 命中，`新后指针`、`旧后指针`上移，回到1，继续看有没有命中
   * 未命中，继续向下尝试命中
3. **新后旧前**：
   * 命中，移动`旧前指针`指向的节点到`旧后指针`的后面，并将原位置设置为` undefined`，`旧前指针`下移，`新后指针`上移
   * 未命中，继续向下尝试命中
4. **新前旧后**：
   * 命中，移动`旧后指针`指向的节点到`旧前指针`的前面，并将原位置设置为` undefined`，`旧后指针`上移，`新前指针`下移
   * 未命中
     * 在旧节点中寻找相同`key`的节点
       * 存在
         * 在旧节点中找到的和`新前指针`指向的节点是同一个节点的话，将该节点追加到` 旧前`之前，并将原位置设置为` undefined`，` 新前指针`下移一位
         * 在旧节点中找到的和`新前指针`指向的节点不是同一个节点的话，新增` 新前指针`指向的节点，将该节点追加到` 旧前指针`之前，` 新前指针`下移一位
       * 不存在
         * 新增并将该节点追加到` 旧前指针`之前，` 新前指针`下移一位
5. 循环结束
   * 新节点先循环完毕：删除`旧前指针`、`旧后指针`之间的节点，包括` 旧前`、` 旧后`指向的节点
   * 旧节点先循环完毕：新增`新前指针`、`新后指针`之间的节点到` 旧前指针`前，包括` 新前`、` 新后`指向的节点

<br>

### 实操

src \ patchVnode.js(最复杂的情况，单独抽出来，在updateChildren中操作)

```js
if (oldVnode.children !== undefined && oldVnode.children.length > 0) {
  // newVnode的内容是子节点，oldVnode的内容也是子节点
  updateChildren(oldVnode.elm, oldVnode.children, newVnode.children)
}
```

<br>

src \ updateChildren.js

没什么难度，看原理总结慢慢写就行了(**谨慎点**)

阉割版本，只需要` sel`和` key`相同就认为是同一个虚拟节点。(即不需要判断内容是否相同)

```js
// 精细化比对，最小化更新的，其中新旧节点的内容都是节点的情况

import createElement from "./createElement.js"
import patchVnode from "./patchVnode.js"

// parentElm：oldVnode对应的真实DOM，用于更新DOM
// oldCh：旧节点的子节点
// newCh：新节点的子节点

// 判断是不是同一个虚拟节点
function checkSamVnode(v1, v2) {
  return v1.sel === v2.sel && v1.key === v2.key
}

export default function updateChildren(parentElm, oldCh, newCh) {
  // 旧前指针
  let oldStartIdx = 0
  // 旧后指针
  let oldEndIdx = oldCh.length - 1
  // 旧前节点
  let oldStartVnode = oldCh[0]
  // 旧后节点
  let oldEndVnode = oldCh[oldEndIdx]

  // 新前指针
  let newStartIdx = 0
  // 新后指针
  let newEndIdx = newCh.length - 1
  // 新前节点
  let newStartVnode = newCh[0]
  // 新后节点
  let newEndVnode = newCh[newEndIdx]

  let map = {};

  while (oldStartIdx <= oldEndIdx & newStartIdx <= newEndIdx) {
    if (oldStartVnode === undefined) {
      // 跳过undefined
      oldStartVnode = oldCh[++oldStartIdx]
    }
    if (oldEndVnode === undefined) {
      // 跳过undefined
      oldEndVnode = oldCh[--oldEndIdx]
    }

    if (checkSamVnode(newStartVnode, oldStartVnode)) {
      // 新前旧前命中
      console.log('新前旧前命中')

      patchVnode(oldStartVnode, newStartVnode)
      oldStartVnode = oldCh[++oldStartIdx]
      newStartVnode = newCh[++newStartIdx]
    } else if (checkSamVnode(newEndVnode, oldEndVnode)) {
      // 新后旧后命中
      console.log('新后旧后命中')

      // 继续精细化比较两个节点
      patchVnode(oldEndVnode, newEndVnode)
      oldEndVnode = oldCh[--oldEndIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (checkSamVnode(newEndVnode, oldStartVnode)) {
      // 新后旧前命中
      console.log('新后旧前命中')

      // 继续精细化比较两个节点
      patchVnode(oldStartVnode, newEndVnode)

      parentElm.insertBefore(oldStartVnode.elm, oldEndVnode.elm.nextSibiling)
      oldStartVnode = undefined

      newEndVnode = newCh[--newEndIdx]
      oldStartVnode = oldCh[++oldStartIdx]
    } else if (checkSamVnode(newStartVnode, oldEndVnode)) {
      // 新前旧后命中
      console.log('新前旧后命中')

      // 继续精细化比较两个节点
      patchVnode(oldEndVnode, newStartVnode)

      parentElm.insertBefore(oldEndVnode.elm, oldStartVnode.elm)
      oldEndVnode = undefined

      newStartVnode = newCh[++newStartIdx]
      oldEndVnode = oldCh[--oldEndIdx]
    } else {
      // 四种情况都没命中
      for (let i = oldStartIdx; i <= oldEndIdx; i++) {
        const item = oldCh[i]
        if (item === undefined) {
          // 跳过undefined
          continue
        }
        const key = item.key
        map[key] = i
      }

      // 看一下map中有没有和新前指向的节点相同的
      const indexOld = map[newStartVnode.key]
      if (indexOld) {
        // 存在，则是移动
        const elmToMove = oldCh[indexOld]
        // 继续精细化比较
        patchVnode(elmToMove, newStartVnode)

        parentElm.insertBefore(elmToMove.elm, oldStartVnode.elm)
        oldCh[indexOld] = undefined
      } else {
        // 不存在，需要新增节点
        const newDomNode = createElement(newStartVnode)
        parentElm.insertBefore(newDomNode, oldStartVnode.elm)
      }

      newStartVnode = newCh[++newStartIdx]
    }
  }

  if (newStartIdx <= newEndIdx) {
    // 旧节点先循环完毕，需要新增`新前指针`、` 新后指针`之间节点
    for (let i = newStartIdx; i <= newEndIdx; i++) {
      const item = newCh[i]
      if (item === undefined) {
        continue;
      }

      const newDomNode = createElement(item)

      if (!oldStartVnode) {
        // 如果此时旧前指针指向的是undefined，则直接在最后插入
        parentElm.appendChild(newDomNode)
      } else {
        parentElm.insertBefore(newDomNode, oldStartVnode.elm)
      }
    }
  } else if (oldStartIdx <= oldEndIdx) {
    // 新节点先循环完毕，需要删除`旧前指针`、` 旧后指针`之间节点
    for (let i = oldStartIdx; i <= oldEndIdx; i++) {
      const item = oldCh[i]
      if (item === undefined) {
        continue;
      }
      parentElm.removeChild(item.elm)
    }
  }

}
```

<br>

参考资料：

[详解Vue的Diff算法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/408728265)
