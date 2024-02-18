---
title: Vue源码之mustache模板引擎(一)
date: 2022-03-24 18:38:00
categories: 前端
tags:
  - Vue源码
  - Vue
---

# Vue 源码之 mustache 模板引擎(一)

个人练习结果仓库(持续更新)：[Vue 源码解析](https://github.com/13535944743/Vue_Source_Code_Practise)

<br>

抽空把之前学的东西写成笔记。

学习视频链接：[【尚硅谷】Vue 源码解析之 mustache 模板引擎](https://www.bilibili.com/video/BV1EV411h79m)

## 模板引擎是什么

**模板引擎是将数据变为视图最优雅的解决方案。**

![image-20220312180129318](https://s2.loli.net/2022/03/12/wQJFumIdge6ZzbT.png)

其中，Vue 中的列表渲染指令` v-for`就是一种模板引擎。而**插值表达式`{{}}`**便是本次要研究的` mustache模板引擎`的语法

<br>

## 将数据变为视图的方法

### 纯 DOM 法

很笨拙。需要频繁创建节点，添加数据，添加节点。

```js
const arr = [
  {
    name: "clz",
    age: 21,
    sex: "男",
  },
  {
    name: "cc",
    age: 21,
    sex: "女",
  },
  {
    name: "赤蓝紫",
    age: 21,
    sex: "男",
  },
];

const list = document.getElementById("list");

for (let i = 0; i < arr.length; i++) {
  const li = document.createElement("li"); // 新建li元素

  const bd = document.createElement("div");
  bd.className = "bd";
  bd.innerText = arr[i].name + "的基本信息";
  li.appendChild(bd);

  const hd = document.createElement("div");
  hd.className = "hd";

  for (const item in arr[i]) {
    const p = document.createElement("p");
    p.innerText = item + ": " + arr[i][item];
    hd.appendChild(p);
  }

  li.appendChild(hd);
  list.appendChild(li);
}
```

![image-20220312182355046](https://s2.loli.net/2022/03/12/chJLPs9TlfwmSey.png)

<br>

### 数组 join 法

本质上就是字符串拼接，只是用过数组 join 法，可以让结构变得更清晰

```js
const arr = [
  {
    name: "clz",
    age: 21,
    sex: "男",
  },
  {
    name: "cc",
    age: 21,
    sex: "女",
  },
  {
    name: "赤蓝紫",
    age: 21,
    sex: "男",
  },
];

for (let i = 0; i < arr.length; i++) {
  list.innerHTML += [
    "<li>",
    '  <div class="hd">' + arr[i].name + "的基本信息</div>",
    '  <div class="bd">',
    "    <p>name: " + arr[i].name + "</p>",
    "    <p>age: " + arr[i].age + "</p>",
    "    <p>sex" + arr[i].sex + "</p>",
    "  </div>",
    "</li>",
  ].join("");
}
```

### ES6 的模板字符串法

- 反引号中，文本可以直接换行
- 反引号中的${expression}占位符中 expression 可以为任意的 JavaScript 表达式，甚至为模板字符串

```js
const arr = [
  {
    name: "clz",
    age: 21,
    sex: "男",
  },
  {
    name: "cc",
    age: 21,
    sex: "女",
  },
  {
    name: "赤蓝紫",
    age: 21,
    sex: "男",
  },
];

for (let i = 0; i < arr.length; i++) {
  list.innerHTML += `
      <li>
          <div class="hd">${arr[i].name} 的基本信息</div>
          <div class="bd">
            <p>name: ${arr[i].name} </p>
            <p>age: ${arr[i].age} </p>
            <p>sex: ${arr[i].sex} </p>
          </div>
        </li>
      `;
}
```

<br>

### 模板引擎 mustache

[mustache 仓库](https://github.com/janl/mustache.js)

mustache 是**最早的模板引擎库**。

<br>

```html
<script src="./lib/mustache.js"></script>
<script>
  // console.log(Mustache)

  const templateStr = `
    <ul>
      {{ #arr }}
        <li>
          <div class="hd">{{ name }}的基本信息</div>
          <div class="bd">
            <p>name: {{ name }} </p>
            <p>age: {{ age }} </p>
            <p>sex: {{ sex }} </p>
          </div>
        </li>
      {{ /arr }}
    </ul>
  `;
  const data = {
    arr: [
      {
        name: "clz",
        age: 21,
        sex: "男",
      },
      {
        name: "cc",
        age: 21,
        sex: "女",
      },
      {
        name: "赤蓝紫",
        age: 21,
        sex: "男",
      },
    ],
  };

  const domStr = Mustache.render(templateStr, data);

  document.getElementsByClassName("container")[0].innerHTML = domStr;
</script>
```

引入` mustache`后，就会后一个` Mustache`对象，其中有一个方法` render`就可以用来实现将数据变为视图。

- render 的第一个参数是模板字符串，第二个参数是数据
- 如果需要使用数据，直接通过` {{ }}`使用即可
- 要实现循环的话，则需要用` {{ #arr }}`,` {{ /arr }}`包住要循环的内容

## <span id="jump">mustache 的基本使用</span>

[mustache.js](https://unpkg.com/mustache@4.2.0/mustache.js)

### 简单使用

```js
const templateStr = `
  <h2>我是{{name}}, 年龄为{{age}}岁</h2>
`;
const data = {
  name: "clz",
  age: 21,
};

const domStr = Mustache.render(templateStr, data);

document.getElementsByClassName("container")[0].innerHTML = domStr;
```

![image-20220312211938118](https://s2.loli.net/2022/03/12/I9EnyTRuYc1ibga.png)

### 循环简单数组

循环的不是对象数组，而是简单数组时，使用` .`即可

```js
const templateStr = `
  {{#arr}}
    <h2 style="color: {{.}}">{{.}}</h2>
  {{/arr}}
`;
const data = {
  arr: ["red", "blue", "purple"],
};

const domStr = Mustache.render(templateStr, data);

document.getElementsByClassName("container")[0].innerHTML = domStr;
```

![image-20220312212348808](https://s2.loli.net/2022/03/12/t5acM2UO3xPIDAB.png)

<br>

### 数组嵌套

就是上面两部分的结合版本。

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
`;
const data = {
  arr: [
    {
      name: "clz",
      colors: ["red", "blue", "purple"],
    },
    {
      name: "cc",
      colors: ["white", "red", "black"],
    },
  ],
};

const domStr = Mustache.render(templateStr, data);

document.getElementsByClassName("container")[0].innerHTML = domStr;
```

![image-20220312214124863](https://s2.loli.net/2022/03/12/zHboe6pFn8cBfyq.png)

<br>

### 布尔值

和循环类似，通过使用` {{#布尔值属性}}`,`{{/布尔值属性}}`，包住要条件渲染的内容即可

```js
const templateStr = `
  {{#arr}}
    {{#show}}
      <h2>{{name}}</h2>
    {{/show}}
  {{/arr}}
`;
const data = {
  arr: [
    {
      name: "clz",
      show: true,
    },
    {
      name: "czh",
      show: false,
    },
  ],
};

const domStr = Mustache.render(templateStr, data);

document.getElementsByClassName("container")[0].innerHTML = domStr;
```

![image-20220312214938151](https://s2.loli.net/2022/03/12/Vxk2CwiaAsbRHcM.png)

<br>

通过查看 DOM 树，可以发现和 Vue 中的` v-if`指令类似，是压根就没有上 DOM 树。另外，Vue 中的` v-show`指令则是动态为元素添加或移除` display: none;`来控制元素的显示与隐藏。

<br>

### es6 之前使用 mustache

众所周知，es6 之前是没有模板字符串(反引号)的。那么方便的使用 mustache 呢？

当然，可以使用上面的数组 join 法，不过，还有一个更方便的方法。

通过使用` script`标签，只要添加`type`为` text/template`，然后在里面填模板字符串即可(实际上，只要不被浏览器识别即可)

```js
<script type="text/template" id="templateStr">
  {{#arr}}
    {{#show}}
      <h2>{{name}}</h2>
    {{/show}}
  {{/arr}}
</script>

<script src="./lib/mustache.js"></script>
<script>
  const templateStr = document.getElementById('templateStr').innerHTML

  const data = {
    arr: [
      {
        name: 'clz',
        show: true
      },
      {
        name: 'czh',
        show: false
      }
    ]
  }

  const domStr = Mustache.render(templateStr, data)

  document.getElementsByClassName('container')[0].innerHTML = domStr
</script>
```

只能说**想到这个方法的人太优秀了**

<br>

## mustache 底层原理

<br>

### 正则表达式实现最简单的 mustache

<br>

#### String.prototype.replace()

在开始之前，首先需要了解一下字符串的` replace`方法

> **语法**
>
> ```js
> str.replace(regexp|substr, newSubStr|function)
> ```
>
> **参数**
>
> - `regexp `(pattern)：一个[`RegExp`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp) 对象或者其字面量。该正则所匹配的内容会被第二个参数的返回值替换掉。
>
> - `substr `(pattern)：一个将被 `newSubStr` 替换的 [`字符串`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)。其被视为一整个字符串，而不是一个正则表达式。仅第一个匹配项会被替换。
> - `newSubStr` (replacement)：用于替换掉第一个参数在原字符串中的匹配部分的[`字符串`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)。该字符串中可以内插一些特殊的变量名。参考[使用字符串作为参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace#使用字符串作为参数)。
> - `function` (replacement)：一个用来创建新子字符串的函数，该函数的返回值将替换掉第一个参数匹配到的结果。参考[指定一个函数作为参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace#指定一个函数作为参数)。
>
> **返回值**
>
> 一个部分或全部匹配由替代模式所取代的新的字符串。

```js
const templateStr = `
  <h2>我是{{name}}, 年龄为{{age}}岁</h2>
`;
const data = {
  name: "clz",
  age: 21,
};

console.log(templateStr.replace(/\{\{\w+\}\}/g, "123"));
```

![image-20220313091349829](https://s2.loli.net/2022/03/13/rRUxDw2Gg9IYu1e.png)

<br>

可以发现，上面的做法还无法实现，所以研究一下，第二个参数为函数的情况

> | 变量名            | 代表的值                                                                                                                                                                                                                                                                                         |
> | :---------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
> | `match`           | 匹配的子串。（对应于上述的$&。）                                                                                                                                                                                                                                                                 |
> | `p1,p2, ...`      | 假如 replace()方法的第一个参数是一个[`RegExp`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp) 对象，则代表第 n 个括号匹配的字符串。（对应于上述的$1，$2 等。）例如，如果是用 `/(\a+)(\b+)/` 这个来匹配，`p1` 就是匹配的 `\a+`，`p2` 就是匹配的 `\b+`。 |
> | `offset`          | 匹配到的子字符串在原字符串中的偏移量。（比如，如果原字符串是 `'abcd'`，匹配到的子字符串是 `'bc'`，那么这个参数将会是 1）                                                                                                                                                                         |
> | `string`          | 被匹配的原字符串。                                                                                                                                                                                                                                                                               |
> | NamedCaptureGroup | 命名捕获组匹配的对象                                                                                                                                                                                                                                                                             |

```js
const templateStr = `
  <h2>我是{{name}}, 年龄为{{age}}岁</h2>
`;
const data = {
  name: "clz",
  age: 21,
};

templateStr.replace(/\{\{(\w+)\}\}/g, (match, p1, offset, string) => {
  console.log(match);
  console.log(p1);
  console.log(offset);
  console.log(string);
});
```

![image-20220313092139759](https://s2.loli.net/2022/03/13/7AVX6LyhJfCvual.png)

可以发现，只需要在正则表达式中使用` ()`把要捕获的内容包起来，然后通过` replace`方法的函数参数中的 p1 参数获取捕获内容，既然如此，那就可以开始使用正则表达式实现简单的 mustache 了。

<br>

#### 实现简单的 mustache

```js
const templateStr = `
  <h2>我是{{name}}, 年龄为{{age}}岁</h2>
`;
const data = {
  name: "clz",
  age: 21,
};

const render = (templateStr, data) => {
  return templateStr.replace(
    /\{\{(\w+)\}\}/g,
    function (match, p1, offset, string) {
      return data[p1]; // 把正则所匹配的内容替换成return的内容
    }
  );
};

const domStr = render(templateStr, data);
document.querySelector(".container").innerHTML = domStr;
```

<br>

### mustache 底层 tokens 原理

![image-20220313094603475](https://s2.loli.net/2022/03/13/mF5Bg6EGYriKyVz.png)

**mustache 底层主要干两件事**：

- 将模板字符串编译为 tokens 形式
- tokens 结合数据，解析为 dom 字符串

<br>

#### tokens 是什么

- tokens 是一个嵌套数组，也可以说是**模板字符串的 JS 表示**。
- **tokens**是**抽象语法树**(AST)、**虚拟节点**的开山鼻祖

<br>

看下下面的例子，就能明白了

---

简单 tokens

**模板字符串**

```html
<h2>我是{{name}}, 年龄为{{age}}岁</h2>
```

<br>

**tokens**

```js
[
  ["text", "<h2>我是"],
  ["name", "name"],
  ["text", ", 年龄为"],
  ["name", "age"],
  ["text", "岁</h2>"],
];
```

---

简单数组情况下的 tokens

**模板字符串**

```html
{{#arr}}
<h2 style="color: {{.}}">{{.}}</h2>
{{/arr}}
```

<br>

**tokens**

```js
[
    ["#", "arr", [
        ["text": "<h2 style='color: "],
        ["name": "."],
        ["text": "'>"],
        ["name", "."],
        ["text", "</h2>"]
    ]]
]
```

---

嵌套数组情况下的 tokens

**模板字符串**

```html
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
```

<br>

**tokens**

```js
[
    ["text", "<ul>"],
    ["#", "arr", [
        ["text", "li"],
        ["name", "name"],
        ["text": "喜欢的颜色是:<ol>"],
        ["#", "colors", [
            ["text", "<li>"],
            ["name", "."],
            ["text", "</li>"]
        ]],
        ["text", "</ol></li>"]
    ]],
    ["text", "</ul>"]
]
```

<br>

#### 查看 mustache 的 tokens

进入之前下载的源码文件中，` ctrl+f`，搜索` parseTemplate`，到该方法最后把返回值存好并打印

![image-20220313122831330](https://s2.loli.net/2022/03/13/KRfBS2sopDuvXYg.png)

<br>

重新去跑[mustache 的基本使用](#jump)的代码，就可以在控制台中看到` tokens`

如循环简单数组

![image-20220313123423702](https://s2.loli.net/2022/03/13/IslPNEDt2xLywkp.png)
