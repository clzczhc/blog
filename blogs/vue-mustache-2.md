---
title: Vue源码之mustache模板引擎(二)    手写实现mustache
date: 2022-03-28 18:31:20
categories: 前端
tags:
  - Vue源码
  - Vue
---

# Vue源码之mustache模板引擎(二)    手写实现mustache

> [mustache.js](https://unpkg.com/mustache@4.2.0/mustache.js)

个人练习结果仓库(持续更新)：[Vue源码解析](https://github.com/13535944743/Vue_Source_Code_Practise)

## webpack配置

可以参考之前的笔记[Webpack笔记](https://clz.vercel.app/2022/02/06/yc-webpack/)

安装：` npm i -D webpack webpack-cli webpack-dev-server`

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

修改` package.json`，更方便地使用指令

![image-20220313161823530](https://s2.loli.net/2022/03/28/2T3b4o5zwlVB8kj.png)

****

编写示例代码

src \ index.js

```js
import { mytest } from './test.js'

mytest()
```

<br>

src \ test.js

```js
export const mytest = () => {
  console.log('1+1=2')
}
```

<br>

www \ index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <h2>test</h2>
  <script src="/virtual/bundle.js"></script>
</body>

</html>
```

<br>

**` npm run dev`**，到http://localhost:8080/查看

![image-20220313161924306](https://s2.loli.net/2022/03/28/2fd8xU6Pnza3bMR.png)

<br>

## 实现Scanner类

Scanner类功能：将模板字符串根据指定字符串(如` {{` 和` }}`)切成多部分

有两个主要方法**scan**和**scanUtil**

* **scan**： 跳过指定内容，无返回值
* **scanUtil**：让指针进行扫描，遇到指定内容才结束，还会返回结束之前遍历过的字符

![image-20220314000348836](https://s2.loli.net/2022/03/28/4ixWmzltFCwhGMe.png)

### scanUtil方法

先来一下构造函数

```js
constructor(templateStr) {
  this.templateStr = templateStr
  // 指针
  this.pos = 0
  // 尾巴，用于获取除指定符号外的内容(即`{{`和`}}`)
  this.tail = this.templateStr
}
```

<br>

```js
// 让指针进行扫描，遇到指定内容才结束，还会返回结束之前遍历过的字符
scanUtil(stopTag) {
  const start = this.pos  // 存放开始位置，用于返回结束前遍历过的字符

  // 没到指定内容时，都一直循环，尾巴也跟着变化
  while (this.tail.indexOf(stopTag) !== 0 && this.pos < this.templateStr.length) {    // 后面的另一个条件必须，因为最后需要跳出循环
    this.pos++
    this.tail = this.templateStr.substring(this.pos)
  }

  return this.templateStr.substring(start, this.pos)      // 返回结束前遍历过的字符
}
```

<br>

### scan方法

```js
// 跳过指定内容，无返回值
scan(tag) {
  if (this.tail.indexOf(tag) === 0) {
    this.pos += tag.length
    this.tail = this.templateStr.substring(this.pos)
    // console.log(this.tail)
  }
}
```

<br>

### eos方法

因为模板字符串中需要反复使用**scan**和**scanUtil**方法去把模板字符串完全切成多部份，所以需要循环，而循环结束的条件就是已经遍历完模板字符串了

```js
// end of string：判断模板字符串是否已经走到尽头了
eos() {
  return this.pos === this.templateStr.length
}
```

### 完整类

```js
/*
* 扫描器类
*/

export default class Scanner {
  constructor(templateStr) {
    this.templateStr = templateStr
    // 指针
    this.pos = 0
    // 尾巴，用于获取除指定符号外的内容(即`{{`和`}}`)
    this.tail = this.templateStr
  }

  // 跳过指定内容，无返回值
  scan(tag) {
    if (this.tail.indexOf(tag) === 0) {
      this.pos += tag.length
      this.tail = this.templateStr.substring(this.pos)
      // console.log(this.tail)
    }
  }

  // 让指针进行扫描，遇到指定内容才结束，还会返回结束之前遍历过的字符
  scanUtil(stopTag) {
    const start = this.pos  // 存放开始位置，用于返回结束前遍历过的字符

    // 没到指定内容时，都一直循环，尾巴也跟着变化
    while (this.tail.indexOf(stopTag) !== 0 && this.pos < this.templateStr.length) {    // 后面的另一个条件必须，因为最后需要跳出循环
      this.pos++
      this.tail = this.templateStr.substring(this.pos)
    }

    return this.templateStr.substring(start, this.pos)      // 返回结束前遍历过的字符
  }

  // end of string：判断模板字符串是否已经走到尽头了
  eos() {
    return this.pos === this.templateStr.length
  }
}
```

<br>

### 测试使用

src / index.js

```js
import Scanner from './Scanner.js'

window.TemplateEngine = {
  render(templateStr, data) {
    // 实例化一个扫描器
    const scanner = new Scanner(templateStr)

    while (!scanner.eos()) {
      let words = scanner.scanUtil('{{')
      console.log(words)
      scanner.scan('{{')

      words = scanner.scanUtil('}}')
      console.log(words)
      scanner.scan('}}')
    }
  }
}
```

<br>

www / index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <h2>我是{{name}}, 年龄为{{age}}岁</h2>
  <script src="/virtual/bundle.js"></script>
  <script>
    const templateStr = `
      <h2>我是{{name}}, 年龄为{{age}}岁</h2>
    `
    const data = {
      name: 'clz',
      age: 21
    }

    const domStr = TemplateEngine.render(templateStr, data)

  </script>
</body>

</html>
```

![image-20220314002049092](https://s2.loli.net/2022/03/28/ZthsIen7zu5NdFf.png)

<br>

## 封装并实现将模板字符串编译成tokens数组

首先，把` src / index.js`的代码修改一下，封装成` parseTemplateToTokens`方法

**src \ index.js**

```js
import parseTemplateToTokens from './parseTemplateToTokens.js'

window.TemplateEngine = {
  render(templateStr, data) {
    const tokens = parseTemplateToTokens(templateStr)
    console.log(tokens)
  }
}
```

<br>

### 实现简单版本

```js
// 把模板字符串编译成tokens数组
import Scanner from './Scanner.js'

export default function parseTemplateToTokens() {
  const tokens = []

  // 实例化一个扫描器
  const scanner = new Scanner(templateStr)

  while (!scanner.eos()) {
    let words = scanner.scanUtil('{{')
    if (words !== '') {
      tokens.push(['text', words])  // 把text部分存好：：左括号之前的是text
    }

    scanner.scan('{{')

    words = scanner.scanUtil('}}')
    if (words !== '') {
      tokens.push(['name', words])    // 把name部分存好：：右括号之前的是name
    }

    scanner.scan('}}')
  }

  return tokens
}
```

![image-20220314142032812](https://s2.loli.net/2022/03/28/mtdPsIF2oiG5UyZ.png)

<br>

### 提取特殊符号

用上一个版本的试一下，嵌套数组

```js
const templateStr = `
  <ul>
    {{#arr}}
      <li>
        {{name}}喜欢的颜色是:
        <ol>
          {{#colors}}
            <li>{{.}}</li>
          {{/colors}}
        </ol>
      </li>
    {{/arr}}
  </ul>
`
```

![image-20220314142439828](https://s2.loli.net/2022/03/28/Rnf1vePQil6hjw2.png)

发现存在点问题，所以需要提取特殊符号` #`和` /`

<br>

**取到words时，判断一下第一位符号是不是特殊字符，对特殊字符进行提取**

```js
if (words !== '') {
  switch (words[0]) {
    case '#':
      tokens.push(['#', words.substring(1)])
      break
    case '/':
      tokens.push(['/', words.substring(1)])
      break
    default:
      tokens.push(['text', words])// 把text部分存好
  }
}
```

![image-20220315184648878](https://s2.loli.net/2022/03/15/HxSjc9GKiFX1n6w.png)

又发现，还是没有实现，框框部分应该是tokens里的嵌套tokens才对

<br>

### 实现嵌套tokens

关键：定义一个收集器`collector `，一开始指向要返回的` nestTokens`数组，每当遇到` #`，则把它指向新的位置，遇到` /`，时，又回到上一阶，且数组是引用变量，所以给` colleator` ` push`数据时，对应指向的位置也会跟着增加数据。

为了实现收集器` colleator`能顺利回到上一阶，那么就需要增加一个栈` sections`，每当遇到` #`时，token入栈；而当遇到` /`时，出栈，并判断` sections`是否为空，为空的话，则重新指向` nestTokens`，不空的话，则指向` 栈顶`下标为2的元素。

****

src \ nestTokens.js

```js
// 把#和/之间的tokens整合起来，作为#所在数组的下标为2的项

export default function nestTokens(tokens) {
  const nestTokens = []
  const sections = []   // 栈结构
  let collector = nestTokens

  for (let i = 0; i < tokens.length; i++) {
    const token = tokens[i]

    switch (token[0]) {
      case '#':
        collector.push(token)
        console.log(token)
        sections.push(token)    // 入栈

        token[2] = []
        collector = token[2]
        break
      case '/':
        sections.pop()
        collector = sections.length > 0 ? sections[sections.length - 1][2] : nestTokens
        break
      default:
        collector.push(token)
    }
  }

  return nestTokens
}
```

<br>

另外，`parseTemplateToTokens`函数中返回的不再是` tokens`，而是`nestTokens(tokens)`。

![image-20220315184914505](https://s2.loli.net/2022/03/15/fjVaqD6GJREiP9M.png)

<br>

## 将tokens数组结合数据解析成dom字符串

### 实现简单版本

直接遍历tokens数组，如果遍历的元素的第一个标记是` text`，则直接与要返回的字符串相加，如果是` name`，则需要数据` data`中把对应属性加入到要返回的字符串中。

src \ renderTemplate.js

```js
export default function renderTemplate(tokens, data) {
  let result = ''

  for (let i = 0; i < tokens.length; i++) {
    const token = tokens[i]

    if (token[0] === 'text') {
      result += token[1]
    } else if (token[0] === 'name') {
      result += data[token[1]]
    }
  }

  return result
}
```

<br>

src \ index.js

```js
import parseTemplateToTokens from './parseTemplateToTokens.js'
import renderTemplate from './renderTemplate.js'

window.TemplateEngine = {
  render(templateStr, data) {
    const tokens = parseTemplateToTokens(templateStr)

    const domStr = renderTemplate(tokens, data)
    console.log(domStr)
  }
}
```

![image-20220316110816619](https://s2.loli.net/2022/03/16/OtDYH2TmaMoqnJb.png)

快成功了，开心

****

问题：当数据中有对象类型的数据时，会出问题。

如

```js
const templateStr = `
  <h2>我是{{name}}, 年龄为{{age}}岁, 工资为{{job.salary}}元</h2>
`
const data = {
  name: 'clz',
  age: 21,
  job: {
    type: 'programmer',
    salary: 1
  }
}
```

![image-20220316110854642](https://s2.loli.net/2022/03/16/N2XoUAx3YhdzvBW.png)

<br>

为什么会出现这个问题呢？

我们再看一下上面的代码

```js
if (token[0] === 'text') {
  result += token[1]
} else if (token[0] === 'name') {
  result += data[token[1]]
}
```

把出问题的部分代进去，

```js
result += data['job.salary']
```

<br>

但是这样是不行的，JavaScript不支持对象使用数组形式时，下标为` x.y`的形式

![image-20220316111512509](https://s2.loli.net/2022/03/16/ozh8RSZcvr1mDk2.png)

那么该怎么办呢？

其实只需要把` obj[x.y]`的形式变为`obj[x][y] `的形式即可

src \ lookup.js

```js
// 把` obj[x.y]`的形式变为`obj[x][y] `的形式

export default function lookup(dataObj, keysStr) {

  const keys = keysStr.split('.')
  let temp = dataObj

  for (let i = 0; i < keys.length; i++) {
    temp = temp[keys[i]]
  }

  return temp
}
```

![image-20220316112721171](https://s2.loli.net/2022/03/16/LbJFqUzDG2yrv9c.png)

<br>

再优化一下，如果` keysStr`没有` .`的话，那么可以直接返回

```js
// 把` obj[x.y]`的形式变为`obj[x][y] `的形式

export default function lookup(dataObj, keysStr) {

  if (keysStr.indexOf('.') === -1) {
    return dataObj[keysStr]
  }


  const keys = keysStr.split('.')
  let temp = dataObj

  for (let i = 0; i < keys.length; i++) {
    temp = temp[keys[i]]
  }

  return temp
}
```

<br>

### 通过递归实现嵌套数组版本

数据以及模板字符串

```js
const templateStr = `
      <ul>
        {{#arr}}
          <li>
            {{name}}喜欢的颜色是:
            <ol>
              {{#colors}}
                <li>{{name}}</li>
              {{/colors}}
            </ol>
          </li>
        {{/arr}}
      </ul>
    `
    const data = {
      arr: [
        {
          name: 'clz',
          colors: [{
            name: 'red',
          }, {
            name: 'blue'
          }, {
            name: 'purple'
          }]
        },
        {
          name: 'cc',
          colors: [{
            name: 'red',
          }, {
            name: 'blue'
          }, {
            name: 'purple'
          }]
        }
      ]
    }
```

<br>

src \ renderTemplate(增加实现嵌套数组版本)

```js
// 将tokens数组结合数据解析成dom字符串

import lookup from './lookup.js'

export default function renderTemplate(tokens, data) {
  let result = ''

  for (let i = 0; i < tokens.length; i++) {
    const token = tokens[i]

    if (token[0] === 'text') {
      result += token[1]

    } else if (token[0] === 'name') {
      result += lookup(data, token[1])

    } else if (token[0] === '#') {
      let datas = data[token[1]]  // 拿到所有的数据数组

      for (let i = 0; i < datas.length; i++) {   // 遍历数据数组，实现循环
        result += renderTemplate(token[2], datas[i])    // 递归调用
      }
    }
  }

  return result
}
```

![image-20220316141222936](https://s2.loli.net/2022/03/16/8d1FsWNzgmCH9ah.png)

<br>

实现简单数组的那个` .`，因为数据中没有属性` .`，所以需要把该属性给加上

下面的代码只拿了改的一小段

src \ renderTemplate(增加实现嵌套数组版本)

```js
 else if (token[0] === '#') {
  let datas = data[token[1]]  // 拿到所有的数据数组

  for (let i = 0; i < datas.length; i++) {   // 遍历数据数组，实现循环
    result += renderTemplate(token[2], {// 递归调用
      ...datas[i],     // 使用扩展字符串...，把对象展开，再添加.属性为对象本身
      '.': datas[i]
    })
  }
}
```

但是，还是有问题

![image-20220316142004937](https://s2.loli.net/2022/03/16/6RNh5KeJTvMncx1.png)

<br>

回到` lookup`中查看

![image-20220316142324569](https://s2.loli.net/2022/03/16/9XvaqIdtHmsCUPD.png)

微操一手：

src \ lookup.js

```js
// 把` obj[x.y]`的形式变为`obj[x][y] `的形式

export default function lookup(dataObj, keysStr) {
  if (keysStr.indexOf('.') === -1 || keysStr === '.') {
    return dataObj[keysStr]
  }


  const keys = keysStr.split('.')
  let temp = dataObj

  for (let i = 0; i < keys.length; i++) {
    temp = temp[keys[i]]
  }

  return temp
}
```

![image-20220316142456833](https://s2.loli.net/2022/03/16/EQ5M49xkdV6jPZF.png)

成功。

<br>

最后把它挂到DOM树上

```js
const domStr = TemplateEngine.render(templateStr, data)
document.getElementsByClassName('container')[0].innerHTML = domStr
```

![image-20220316143056922](https://s2.loli.net/2022/03/16/42uIH5bKxpGcFU9.png)

<br>

学习视频：[【尚硅谷】Vue源码解析之mustache模板引擎_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1EV411h79m)
