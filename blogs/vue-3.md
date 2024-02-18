---
title: Vue学习笔记(三)
date: 2021-10-15 12:26:53
categories: "前端"
tags:
	- Vue
---

# Vue 学习笔记(三)

## 1. 插槽

插槽允许开发者在封装组件时，把**不确定的、希望由用户指定的部分**定义为插槽。

我们使用标签时，开始标签和结束标签之间之前都没有写东西。组件的标签和正常的双标签，如 div、p 等一样，可以在里面写东西。但是，直接在里面写，会发现，写的东西不会显示出来，也找不到了，被丢弃了。其实这个也挺好理解的，组件本来就有东西了，vue 又不知道你写的东西要插到哪里去。所以，vue 提供了插槽，可以在想要插的地方加上一个插槽，之后再把内容插过去。

用法例子：

![](https://pic.imgdb.cn/item/61529ba52ab3f51d917efa3d.jpg)

效果：

![](https://pic.imgdb.cn/item/61529bb92ab3f51d917f0fce.jpg)

<b style="color: red">没有预留插槽的话，用户提供的自定义内容都会被丢弃。</b>

- 封装组件时，可以为预留的插槽提供默认内容，如果组件的使用者没有为插槽提供内容，默认内容就会生效。

  ![](https://pic.imgdb.cn/item/61529cdb2ab3f51d91804660.jpg)

### 1.1 具名插槽

![](https://pic.imgdb.cn/item/61529e812ab3f51d91821f07.jpg)

上面的例子中，有多个插槽，输入的文章头这段信息原本想插在第一个插槽里面的，但是会发现，它插到了所有的插槽中。

这个时候就需要使用**具名插槽了**。

<b style="color: red">具名插槽</b>：如果在封装组件时需要预留多个插槽，则需要为每个插槽指定具体的名称。这种带有具体名称的插槽就叫"具名插槽"。

![](https://pic.imgdb.cn/item/6152a22c2ab3f51d91870ab3.jpg)

如果没有给插槽起名字，则插槽默认叫"default"。要插入插槽的内容如果没有指定要插到哪里去，则会插到名为"default"的插槽中。

这就是为什么上面没有使用具名插槽时，内容会插到所有的插槽中去。

### 1.2 作用域插槽

在封装组件时，可以为预留的 slot 插槽绑定 props 数据，这个**带有 props 数据的 slot 插槽**叫做**作用域插槽**

![](https://pic.imgdb.cn/item/6152a5bd2ab3f51d918bc348.jpg)

**解构插槽**：因为得到的数据是对象形式的，所以可以解构，得到要用的数据

![](https://pic.imgdb.cn/item/6152a6332ab3f51d918c5a45.jpg)

## 2. 自定义指令

### 2.1 私有自定义指令

在每个 vue 组件中，可以在 directives 节点下声明**私有自定义指令**。

![](https://pic.imgdb.cn/item/6152a81d2ab3f51d918eacae.jpg)

- 为自定义指令动态绑定参数值

  通过=的方式，为当前指令动态添加参数值，通过形参中的第二个参数**binding**来接收指令的参数值。

  ```html
  <template>
    <div class="app-container">
      <h1 v-color="color">App根组件</h1>
      <hr />
    </div>
  </template>

  <script>
    export default {
      data() {
        return {
          color: "blue",
        };
      },
      directives: {
        color: {
          //自定义指令名字
          bind(el, binding) {
            //el是绑定了这个指令的DOM对象
            // el.style.color = 'red';
            console.log(binding);
          },
        },
      },
    };
  </script>

  <style lang="less">
    .app-container {
      padding: 1px 20px 20px;
      background-color: #efefef;
    }
    .box {
      display: flex;
    }
  </style>
  ```

  打印的结果：

  ![](https://pic.imgdb.cn/item/6152a97c2ab3f51d9190653f.jpg)

  可以知道 binding.value 就是参数值。

  所以上面的 bind()改为

  ```
   bind(el, binding) {  //el是绑定了这个指令的DOM对象
    el.style.color = binding.value;
  },
  ```

  就可以实现为自定义指令动态绑定参数值

- <b style="color: red">update 函数</b>：bind 函数只会调用一次，当指令第一次绑定到元素时调用，**当 DOM 更新时 bind 函数不会触发**。update()函数则是**每当 DOM 更新时**，都会触发。

  ```html
  <template>
    <div class="app-container">
      <h1 v-color="color">App根组件</h1>
      <button @click="color = color=='red' ? 'blue' : 'red'">变色</button>
      <hr />
    </div>
  </template>

  <script>
    export default {
      data() {
        return {
          color: "blue",
        };
      },
      directives: {
        color: {
          //自定义指令名字
          bind(el, binding) {
            //el是绑定了这个指令的DOM对象
            el.style.color = binding.value;
          },
        },
      },
    };
  </script>

  <style lang="less">
    .app-container {
      padding: 1px 20px 20px;
      background-color: #efefef;
    }
    .box {
      display: flex;
    }
  </style>
  ```

  上面的示例中，无论怎么点击变色按钮，颜色都不会变，这就是因为 bind()方法只在当指令第一次绑定到元素时调用，且**只调用一次**，所以此时需要用 update()方法。<b style="color: red">不能只有 update()，没有 bind()。每当 DOM 更新时，都会触发，但是指令第一次绑定到元素时，update()不会调用</b>。

  directives 节点改为以下代码

  ```html
  directives: { color:{ //自定义指令名字 bind(el, binding) {
  //el是绑定了这个指令的DOM对象 el.style.color = binding.value;
  console.log("bind"); }, update(el, binding) { //el是绑定了这个指令的DOM对象
  el.style.color = binding.value; console.log("update"); }, } }
  ```

  连续点击变色按钮。

  ![](https://pic.imgdb.cn/item/6152ae852ab3f51d9196e16e.jpg)

  简写：上面那张图，可以看到 bind()和 update()方法的业务逻辑一样，此时，可以使用简写形式。

  ```html
  color(el, binding) { //el是绑定了这个指令的DOM对象 el.style.color =
  binding.value; },
  ```

### 2.2 公有自定义指令

所有组件都能使用。

在 main.js 中通过**Vue.directive()**进行声明

```js
Vue.directive("color", (el, binding) => {
  el.style.color = binding.value;
});
```

## 3. ESlint 使用

ESLint 最初是由[Nicholas C. Zakas](http://nczonline.net/) 于 2013 年 6 月创建的开源项目。它的目标是提供一个插件化的 javascript 代码检测工具。(用来团队协作时，不会因为代码规范问题酿成大错，事先规定好代码的规范，不符合规范会报错或警告)

新建 vue 项目时选择

![](https://pic.imgdb.cn/item/6157f7df2ab3f51d91ce06f9.jpg)

![](https://pic.imgdb.cn/item/6157f7f22ab3f51d91ce2dcf.jpg)

![](https://pic.imgdb.cn/item/6157f84e2ab3f51d91cee142.jpg)

故意在 main.js 中空两行结果：

![](https://pic.imgdb.cn/item/6157f9cc2ab3f51d91d19a79.jpg)

复制上图红框框的字，到[ESLint - 插件化的 JavaScript 代码检测工具](https://eslint.bootcss.com/)查找错误原因

![](https://pic.imgdb.cn/item/6157fa4a2ab3f51d91d27d96.jpg)

ctrl+F,把复制的内容粘贴上去

![](https://pic.imgdb.cn/item/6157fb542ab3f51d91d46249.jpg)

**修改规则**：

可以自己修改规则

![](https://pic.imgdb.cn/item/6157fc4f2ab3f51d91d63059.jpg)

![](https://pic.imgdb.cn/item/6157fcdc2ab3f51d91d736d9.jpg)

![](https://pic.imgdb.cn/item/6157fdf62ab3f51d91d92893.jpg)

## 4. axios 优化

axios 用法可查看[Vue 学习笔记(一)](https://13535944743.github.io/2021/10/01/vue-1/)

用之前的方法每次新的组件需要使用 axios 时，都需要反复导入，通过 main.js 和原型链把 axios 挂载到 Vue 的原型上

![](https://pic.imgdb.cn/item/61580a122ab3f51d91f0224f.jpg)

用的时候不需要重新导入，而是直接通过 this.$http 调用即可

![](https://pic.imgdb.cn/item/61580a762ab3f51d91f0dc6f.jpg)

**全局配置 axios 的请求根路径**：

![](https://pic.imgdb.cn/item/61580bb22ab3f51d91f3338e.jpg)

<b style="color: red">较高效用法</b>：

1. 通过 axios.create()设置好基址

![](https://pic.imgdb.cn/item/6165589c2ab3f51d91012d2e.jpg)

2. 其他要用到的地方导入使用即可

   ![](https://pic.imgdb.cn/item/616559282ab3f51d9101d5cb.jpg)

## 5. 路由

### 5.1 前端路由的概念

路由(router)是**对应关系**，前端路由则是 Hash 地址与组件之间的对应关系

**SPA 和前端路由**：SPA 指的是一个 web 网站只有唯一的一个 HTML 页面，通过**组件的展示和切换**来实现类似多个 HTML 页面的效果。**不同组件之间的切换**需要通过**前端路由**来实现。

**前端路由的工作方式**：

1. 用户点击了页面上的**路由链接**
2. 导致 URL 地址栏中的 Hash 值发生变化
3. 前端路由监听到 Hash 地址的变化
4. 前端路由把当前 Hash 地址的组件渲染到浏览器中

例子：

![](https://pic.imgdb.cn/item/615853f62ab3f51d916a363a.jpg)

### 5.2 vue-router

只能结合 vue 项目进行使用，可以轻松地管理 SPA 项目中组件的切换。

#### 5.2.1 基本用法

1. 安装 vue-router

   ` npm install vue-router -S`

2. 创建路由模块

   ![](https://pic.imgdb.cn/item/61585e2e2ab3f51d917d6792.jpg)

3. 导入并挂载路由模块

   src/main.js 入口文件

   ![](https://pic.imgdb.cn/item/615861a52ab3f51d91856ad6.jpg)

4. 声明**路由链接**和**占位符**

   用 vue-router 提供的**router-link**来声明**路由链接**，

   用**router-view**来声明**占位符**，用来放路由链接对应的组件

   ![](https://pic.imgdb.cn/item/615863112ab3f51d91885e23.jpg)

5. 声明路由的**匹配规则**

   在 src/router/index.js 路由模块中，通过**routes 数组**声明路由的匹配规则。

   ![](https://pic.imgdb.cn/item/615866172ab3f51d918e4905.jpg)

6. 路由重定向

   经过上面五步后，会发现根路径不会出现首页，这个时候需要**路由重定向**。

   **路由重定向**：用户在访问地址 A 时，强制用户跳转到特定的组件页面。通过路由规则的 redirect 属性，指定一个新的路由地址。

   ![](https://pic.imgdb.cn/item/615868152ab3f51d9191f89c.jpg)

   用 component 也指定 Home 可以实现类似结果。区别是，用重定向的方法相当于是没有根路径，进入根路径时会强制重定向地址。而用 component 也指定 Home 的方法则是有两个一样的页面。

#### 5.2.2 嵌套路由

和路由的基本用法类似，不同的是用来声明路由的匹配规则不能直接写在 router/index.js 下的 routes 中，而应是在已经有的匹配规则中添加 chilaren 节点，再添加嵌套路由匹配规则。

声明路由链接和占位符和路由的基本用法一样

![](https://pic.imgdb.cn/item/6159155f2ab3f51d91547419.jpg)

![](https://pic.imgdb.cn/item/615915e92ab3f51d9155333a.jpg)

​ ![](https://pic.imgdb.cn/item/615afb912ab3f51d9106dd53.jpg)

#### 5.2.3 动态路由匹配

**动态路由**：把 Hash 地址中可变的部分定义为参数项，从而提高路由规则的复用性。使用`:`来定义路由的参数项。

![](https://pic.imgdb.cn/item/615919432ab3f51d9159c444.jpg)

![](https://pic.imgdb.cn/item/615919892ab3f51d915a217a.jpg)

经过上面的步骤后可以实现点击三部电影，都出现电影的组件。

可以在展示的组件中，通过**$route.params 参数对象**得到参数值

![](https://pic.imgdb.cn/item/615be3132ab3f51d91fe2c81.jpg)

<b style="color: red">获取参数的另一个方法，开启 props 传参</b>

![](https://pic.imgdb.cn/item/61591d3c2ab3f51d915f3ee0.jpg)

![](https://pic.imgdb.cn/item/61591fa02ab3f51d9162c542.jpg)

![](https://pic.imgdb.cn/item/6187434a2ab3f51d911cc2d4.jpg)

#### 5.2.4 编程式导航

- <b style="color: red">编程式导航</b>：通过**调用 API 方法**实现导航的方式，如通过**location.href**跳转到新页面的方式
- <b style="color: red">声明式导航</b>：**点击链接**实现导航的方式，如点击**a 链接**和点击 vue 项目中的**router-link**

**vue-router 中的编程式导航 API**：

1. **$router.push('hash 地址')**：跳转到指定的 hash 地址，并**增加一条历史记录**

2. **$router.replace('hash 地址')**：跳转到指定的 hash 地址，并**替换当前历史记录**

3. **$router.go('数值')**：实现导航历史的前进、后退

   ![](https://pic.imgdb.cn/item/61596ca92ab3f51d91ec585a.jpg)

4. **$router.back()**：回退到历史记录中的上一个界面

5. **$router.forward()**：前进到历史记录中的下一个界面

#### 5.2.5 导航守卫

**导航守卫**可以**控制路由的访问权限**。

![](https://pic.imgdb.cn/item/61596e432ab3f51d91ef09c9.jpg)

**全局前置守卫**：每次发生路由的导航跳转时，都会触发全局前置守卫。通过全局前置守卫可以对每个路由进行权限的控制。

通过 router.beforeEach(fn)可以实现声明全局前置守卫。

fn 接收 3 个形参(to, from, next)，**to**是将要访问的路由的信息对象, **from**是将要离开的路由的信息对象，next 是一个函数，调用 next()表示可以前往。

![](https://pic.imgdb.cn/item/6159a0ce2ab3f51d913fa530.jpg)

**导航守卫控制权限**示例：

![](https://pic.imgdb.cn/item/6159a50d2ab3f51d91477abf.jpg)

学习链接：

[黑马程序员 Vue 全套视频教程](https://www.bilibili.com/video/BV1zq4y1p7ga)

[Vue.js (vuejs.org)](https://vuejs.org/)

[Vue Router (vuejs.org)](https://router.vuejs.org/zh/guide/#html)
