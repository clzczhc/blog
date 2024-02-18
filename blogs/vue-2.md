---
title: Vue学习笔记(二)
date: 2021-10-08 22:08:39
categories: "前端"
tags:
	- Vue
---

# Vue 学习笔记(二)

**单页面应用程序 SPA**，指的是**一个 Web 网站中只有唯一一个 HTML 页面**，所有的功能和交互都在这个唯一的页面内完成。

## 1. vue-cli

vue-cli 是 Vue.js 开发的标准工具。

### 1.1 安装

` npm install -g @vue/cli`

### 1.2 vue 项目的部分文件功能

vue 通过 main.js 把 App.vue 渲染到 index.html 的指定区域中。

- App.vue 用来编写待渲染的模板结构
- index.html 需要预留一个 el 区域等待渲染
- main.js 把 App.vue 渲染到 index.html 预留的区域中

**$(mount)用法**：

```js
import Vue from "vue";
import MyCom from "./components/myComponent";

Vue.config.productionTip = false;

// new Vue({
//   el: '#app',
//   render: h => h(MyCom),
// })

new Vue({
  render: (h) => h(MyCom),
}).$mount("#app"); //$(mount)和上面的el属性用法一样
```

## 2. 组件

**组件化开发**：把**页面上可复用的 UI 结构封装成组件**，从而方便项目的开发和维护

vue**支持组件化开发**。组件的后缀名是**.vue**

每个.vue 组件可有三个部分组成：

- **template**：组件的模板结构(**必须包含**), <b style="color: red">只能有一个根节点</b>
- **script**：组件的 Javascript 行为
- **style**：组件的样式

```html
<template>
  <div>
    <h2>用户名：{{ username }}</h2>
  </div>
</template>

<script>
  export default {
    // data: {
    //     username: 'admin'
    // }        //.vue文件中的数据源不可以用对象形式，而应是函数形式

    data() {
      return {
        username: "admin",
      };
    },
  };
</script>

<style>
  div > h2 {
    color: red;
  }
</style>
```

**在 vue 中使用 less**：

```html
<style lang="less">
  .box {
    background-color: pink;
    h2 {
      color: red;
    }
  }
</style>
```

### 2.1 组件使用的三个步骤

1. 使用 import 语法导入要用的组件
2. 在 components 节点注册组件
3. 直接把组件当成标签在要渲染的地方使用

![](https://pic.imgdb.cn/item/614d68ff2ab3f51d91b8ffbd.jpg)

### 2.2 注册全局组件

上面的注册组件的方法是私有注册，即每一个需要用到经常用的组件，都需要反复注册。这个时候，可以直接在 main.js 中进行全局组件的注册

```js
import Vue from "vue";
import App from "./App.vue";
import Test from "./components/Test.vue"; //导入要注册的全局组件

Vue.config.productionTip = false;

Vue.component("Mytest", Test); //使用Vue.component()来进行全局组件的注册，第一个参数是要注册的组件的名称，第二个参数是要注册的组件

new Vue({
  render: (h) => h(App),
}).$mount("#app");
```

### 2.3 组件的 props

props 是组件的自定义属性，在封装通用组件时，合理使用 props 可以**提高组件的复用性**，允许使用者通过自定义属性，为当前组件指定初始值

![](https://pic.imgdb.cn/item/614d7b6d2ab3f51d91d23d39.jpg)

<b style="color: red">props 是只读的</b>。

![](https://pic.imgdb.cn/item/614d7bfe2ab3f51d91d32164.jpg)

要修改的话，可以把得到的初始值赋给 data 中的属性，再进行修改，props 中的属性的值会一直是初始值

![](https://pic.imgdb.cn/item/614d7cff2ab3f51d91d4a226.jpg)

**default 属性、type 属性和 required 属性**：如果使用者使用使用组件时，没有传递 init 属性, 则默认值生效

使用语法示例：

```html
props: { init: { default: 0, //用default属性定义属性的默认值 type: Number,
//限定传递的属性的类型，不匹配会报错 required: true,
//设置为true时，必须要传递参数，否则即使有默认值，也会报错 } },
```

### 2.4 组件之间的样式冲突

默认情况下，写在**.vue 组件中的样式会全局生效**，所以很容易造成**多个组件之间的样式冲突问题**

导致组件之间的样式冲突的原因：

1. 单页面应用程序中，所有的组件的 DOM 结构都是基于**唯一的 index.html 页面**呈现的
2. 每个组件中的样式都会**影响到整个 index.html 页面**中的所有 DOM 元素

![](https://pic.imgdb.cn/item/614d8ab52ab3f51d91e9188c.jpg)

**通过给要设置样式的组件的 style 标签中添加"scoped"属性，可以实现不影响到其他组件的样式**

![](https://pic.imgdb.cn/item/614d8b8b2ab3f51d91ea2bae.jpg)

原理：给组件里的所有标签都来一个自定义样式，然后通过属性选择器实现样式只会影响到该组件

上面使用"scoped"的代码，不使用"scoped"属性实现:

```html
<template>
  <div class="left-container" data-d-001>
    <h3 data-d-001>Left 组件</h3>
    <MyCount :init="5" data-d-001></MyCount>
  </div>
</template>

<script>
  export default {};
</script>

<style lang="less">
  .left-container {
    padding: 0 20px 20px;
    background-color: orange;
    min-height: 250px;
    flex: 1;
  }
  h3[data-d-001] {
    color: red;
  }
</style>
```

<b style="color: red">只是使用"scoped"的话，无法实现单独修改用上的其他组件的样式</b>

![](https://pic.imgdb.cn/item/614d8e572ab3f51d91edfcc3.jpg)

如果想要让某些样式对子组件生效，可以使用**/deep/ 深度选择器**

![](https://pic.imgdb.cn/item/614d8f3a2ab3f51d91ef4356.jpg)

通过浏览器查看生效样式：

![](https://pic.imgdb.cn/item/61697df82ab3f51d91535348.jpg)

**属性选择器使用不一定需要依靠其他选择器，单独使用表示选择所有有对应属性的元素**

![](https://pic.imgdb.cn/item/614d91de2ab3f51d91f2f9cb.jpg)

## 3. 组件的生命周期

**生命周期**是指一个组件从**创建**->**运行**->**销毁**的整个阶段，强调的是一个时间段

**生命周期函数**：由 vue 框架提供的**内置函数**，会伴随组件的生命周期，**自动按次序执行**。

<b style="color: red">生命周期强调的是时间段，生命周期函数强调的是时间点</b>

**组件生命周期函数的分类**：

![](https://pic.imgdb.cn/item/614daf602ab3f51d9118fe5c.jpg)

**生命周期图示**：

![](https://cn.vuejs.org/images/lifecycle.png)

### 3.1 组件创建阶段

#### 3.1.1 beforeCreate()

组件的**props**、**data**、**methods**还没有被创建，都处于**不可用**状态

```html
<template>
  <div>
    <h2>test组件</h2>
  </div>
</template>

<script>
  export default {
    props: ["info"],
    data() {
      return {
        message: "message",
      };
    },
    methods: {
      show() {
        console.log("show");
      },
    },
    beforeCreate() {
      // console.log(this.info);
      // console.log(this.message);
      this.show();
    },
  };
</script>

<style></style>
```

![](https://pic.imgdb.cn/item/614dd1372ab3f51d914cdc07.jpg)

#### 3.1.2 created()

组件的**props**、**data**、**methods**已经创建完，处于**可用**状态。

![](https://pic.imgdb.cn/item/614dd21e2ab3f51d914e5be3.jpg)

<b style="color: red">created 方法很重要，经常在里面调用 methods 的方法，请求服务器的数据，并把请求到的数据转存到 data 中，供渲染时使用</b>,因为应该尽可能早的请求数据。

```html
<template>
  <div>
    <h2>共有 {{ books.length }} 本书</h2>
  </div>
</template>

<script>
  export default {
    props: ["info"],
    data() {
      return {
        message: "message",
        books: [],
      };
    },
    methods: {
      show() {
        console.log("show");
      },
      initBooklist() {
        const xhr = new XMLHttpRequest();
        xhr.open("get", "http://www.liulongbin.top:3006/api/getbooks");
        xhr.send(null);
        xhr.addEventListener("load", () => {
          const result = JSON.parse(xhr.responseText);
          this.books = result.data;
        }); //使用ajax获取图书信息示例
      },
    },
    created() {
      this.initBooklist();
    },
  };
</script>

<style></style>
```

**组件的模板结构还没生成**(DOM 节点现在不能操作)

![](https://pic.imgdb.cn/item/614dd72c2ab3f51d91571ed9.jpg)

#### 3.1.3 beforeMount()

**将要**把内存中编译好的**HTML 结构**渲染到当前组件的 DOM 结构

![](https://pic.imgdb.cn/item/614dd9232ab3f51d915a7d7e.jpg)

#### 3.1.4 mounted()

<b style="color: red">重要，要操作当前组件的 DOM，最早只能在 mounted 阶段</b>，已经把内存中的 HTML 结构成功的渲染到了浏览器中，此时已经包含了当前组件的**DOM 结构**

![](https://pic.imgdb.cn/item/614ddac12ab3f51d915d7206.jpg)

### 3.2 组件的运行阶段

#### 3.2.1 beforeUpdate()

**将要**根据变化过后、最新的数据，**重新渲染**到组件的模板结构。(即现在 data 里的数据已经变化了，但是 DOM 元素里的数据还没来得及变)

```html
<template>
  <div>
    <h3>
      用于测试运行阶段的message:
      <span>{{ message }}</span>
    </h3>
    <button @click="datachange">更改数据</button>
  </div>
</template>

<script>
  export default {
    props: ["info"],
    data() {
      return {
        message: "Hello",
      };
    },
    methods: {
      datachange() {
        this.message += "!";
      },
    },
    beforeUpdate() {
      const span = document.querySelector("h3 span");

      console.log("data里的数据：" + this.message);
      console.log("DOM中的数据：" + span.innerHTML);
    },
  };
</script>

<style></style>
```

![](https://pic.imgdb.cn/item/614e760c2ab3f51d9122b3c6.jpg)

#### 3.2.2 updated()

已经**完成了**组件 DOM 结构的**重新渲染**

![](https://pic.imgdb.cn/item/614e76782ab3f51d9123223b.jpg)

<b style="color: red">数据发生变化时，如果要操作重新渲染过的 DOM，应在 updated()中执行</b>

### 3.3 组件销毁阶段

#### 3.3.1 beforeDestroy()

**将要**销毁组件,组件还处于正常工作的状态

#### 3.3.2 destroyed()

DOM 结构已经完全销毁

## 4. 组件间的数据共享

### 4.1 父组件向子组件传递数据

父组件

```html
<template>
  <div class="app-container">
    <h1>
      父组件数据: {{ message }} <br />
      {{ present }}
    </h1>
    <Test :msg="message" :pst="present"></Test>
  </div>
</template>

<script>
  import Test from "@/components/Test.vue";

  export default {
    data() {
      return {
        message: "Hello Son!",
        present: {
          fruit: "apple",
          toy: "car",
        },
      };
    },
    components: {
      Test,
    },
  };
</script>

<style lang="less"></style>
```

子组件

```html
<template>
  <div class="test">
    <h3 class="de">子组件</h3>
    <p>接收父组件传来的message: {{msg }}</p>
    <p>接收父组件传来的present: {{pst }}</p>
  </div>
</template>

<script>
  export default {
    props: ["msg", "pst"],
    data() {
      return {
        message: "Hello",
      };
    },
    methods: {
      datachange() {
        this.message += "!";
      },
    },
  };
</script>

<style scope>
  .de {
    color: red;
  }
</style>
```

实现父组件向子组件传递数据，主要依靠

- <b style="color: red">在子组件自定义属性</b>
- <b style="color: red">在父组件通过 v-bind 传递数据给子组件的自定义属性</b>

<b style="color: red">通过上面的方法，传递给子组件的数据在 props 的属性中，只读，所以需要修改的话，要把接收的数据赋给子组件 data 中的元素</b>

![](https://pic.imgdb.cn/item/614e88112ab3f51d91389e87.jpg)

### 4.2 子组件向父组件传递数据

通过<b style="color: red">在父组件处自定义事件</b>,和<b style="color: red">在子组件处通过$.emit()方法触发自定义事件</b>来实现子组件向父组件传递数据

![](https://pic.imgdb.cn/item/614e8b3c2ab3f51d913d17f9.jpg)

### 4.3 兄弟组件组件的数据共享

兄弟组件之间的数据共享方案是**EventBus**

步骤：

1. 创建 eventBus.js 文件，向外共享一个 Vue 的实例对象(用法相当于中转站)
2. 在数据**发送方**，调用**bus.$emit**('事件名称', 要发送的数据)方法**触发自定义事件**
3. 在数据**接收方**，调用**bus.$on**('事件名称', 事件处理函数)方法**注册一个自定义事件**

![](https://pic.imgdb.cn/item/614e925e2ab3f51d91467cbc.jpg)

## 5. ref 引用

ref 用来辅助开发者在**不依赖 jQuery 的情况下**，获取 DOM 元素或组件的引用。

每个 vue 的组件实例上，都包含一个**$refs 对象**，里面存储着对应的 DOM 元素或组件的引用。默认情况下，**组件的$refs 指向一个空对象**。

![](https://pic.imgdb.cn/item/61605a9c2ab3f51d9141b3d1.jpg)

**使用 ref 引用 DOM 元素**：

![](https://pic.imgdb.cn/item/614f36e72ab3f51d9129cdaf.jpg)

所以上面要操作 DOM 元素可以通过**this.$refs.myh3**来修改，如：

![](https://pic.imgdb.cn/item/614f379f2ab3f51d912adbae.jpg)

**使用 ref 引用组件实例**：
![](https://pic.imgdb.cn/item/614f3c552ab3f51d9131ad80.jpg)

**控制文本框和按钮的按需切换**：(点击按钮，按钮隐藏，文本框显示；文本框失去焦点，按钮显示，文本框隐藏；文本框显示时自动获取焦点)

```html
<template>
  <div class="app-container">
    <h3 ref="myh3">MyRef组件</h3>
    <input type="text" v-if="inputVisible" ref="myipt" @blur="showButton" />
    <button v-else @click="showInput">展示输入框</button>
  </div>
</template>

<script>
  // import Left from '@/components/Left.vue'
  // import Right from '@/components/Right.vue'

  export default {
    data() {
      return {
        inputVisible: false,
      };
    },
    methods: {
      showInput() {
        this.inputVisible = true;
        console.log(this.$refs.myipt); //调用showInput时，数据刚刚发生了改变，而这行和上一行代码之间的时间间隔太短，
        // 导致DOM结构没有进行完渲染，所以此时出现undefined
        this.$nextTick(() => {
          //组件的$nextTick()方法，会把回调函数推迟到下一个DOM更新周期之后执行
          console.log(this.$refs.myipt);
          this.$refs.myipt.focus();
        });
      },
      showButton() {
        this.inputVisible = false;
      },
    },
    // components: {
    //   Left,
    // }
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

<b style="color: red">this.$nextTick(callback)方法</b>：

组件的**$nextTick(callback)方法**会把 callback**推迟到下一个 DOM 更新周期之后执行**，即等组件的 DOM 更新完之后，再执行 callback 回调函数。从而保证回调函数能操作到最新的 DOM 元素。

上面的例子不能在生命周期函数中的 updated()中使用 input.focus()，因为 data 中的数据一发生变化，updated()都会执行一次，即 input 隐藏时也会执行，此时压根没有 input 元素，又何来 input.focus()之说呢。

## 6. 动态组件

动态组件指的是**动态切换组件的显示与隐藏**

vue 提供了一个内置的<component>组件，**专门用来实现动态组件的渲染**。

- 使用组件的三大步骤：

  1. 引入组件
  2. 注册组件
  3. 通过标签使用组件

  ![](https://pic.imgdb.cn/item/61605ab92ab3f51d9141f278.jpg)

  第三步，可以使用内置的 component 组件，通过 is 属性来动态指定要渲染的组件

  即上面的 3 可以换成` <component :is="'Left'"></component>`

动态指定渲染组件示例：

App 组件

```html
<template>
  <div class="app-container">
    <h1>App 根组件</h1>
    <button @click="comName='Left'">展示Left组件</button>
    <button @click="comName='Right'">展示Right组件</button>
    <hr />

    <div class="box">
      <component :is="comName"></component>
    </div>
  </div>
</template>

<script>
  import Left from "@/components/Left.vue";
  import Right from "@/components/Right.vue";
  export default {
    data() {
      return {
        comName: "Left",
      };
    },
    components: {
      Left,
      Right,
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

Left 组件(注释代码是后面的部分才用上的)

```html
<template>
  <div class="left-container">
    <h3>Left 组件</h3>
    <h4>count的值是 {{ count }}</h4>
    <button @click="count += 1">+1</button>
  </div>
</template>

<script>
  export default {
    // name: 'MyLeft',
    data() {
      return {
        count: 0,
      };
    },
    created() {
      console.log("左侧被创建了");
    },
    destroyed() {
      console.log("左侧被销毁了");
    },
    // activated() {
    //   console.log('左侧被激活了');
    // },
    // deactivated() {
    //   console.log('左侧休息了');
    // }
  };
</script>

<style lang="less">
  .left-container {
    padding: 0 20px 20px;
    background-color: orange;
    min-height: 250px;
    flex: 1;
  }
</style>
```

**问题**：在上面的示例中，点击 Left 组件的+1 按钮，已经改变了数值，但是，点击展示 Right 组件后，再重新展示 Left 组件，会发现数值又回归了初始状态。从控制台中的输出，可以知道，原因是动态指定渲染 Right 组件时，Left 组件会被销毁，之后右重新创建，所以数据会是初始状态。

可以通过 vue 内置的<b style="color: red">` <keep-alive>组件 `</b>保持动态组件的状态。

用法：用 keep-alive 组件包住动态组件就可以。

```html
<keep-alive>
  <component :is="comName"></component>
</keep-alive>
```

当组件被激活时，会自动触发组件的**activated**生命周期函数。

当组件休眠时，会自动触发组件的**deactivated**生命周期函数。

把上面 Left 组件的注释代码启用。可以发现，当 Left 组件激活时(展示 Left)，会打印出"左侧被激活了"；而 Left 休眠时(展示 Right)，会打印出"左侧休息了"。

<b style="color: red">include 属性</b>**：实现只有名字匹配的组件才会被缓存。就是说，上面的例子中，Left 组件中有数据，想要保存 Left 的状态，但是 Right 组件没有必要缓存，甚至有可能 Right 组件是一次性的用法。这个时候通过 include 属性，可以指定谁会被缓存。多个组件间用**英文的逗号\*\*分隔。

![](https://pic.imgdb.cn/item/61528f9f2ab3f51d9172712f.jpg)

**逗号左右不要有空格**

用法：

```html
<keep-alive include="Left">
  <component :is="comName"></component>
</keep-alive>
```

![](https://pic.imgdb.cn/item/61528db32ab3f51d916ff37c.jpg)

可以发现，上面的 Right 组件在调试工具中显示的名字并不是 Right，而是自定义的 MyRight。这是通过组件的 name 节点修改的。

![](https://pic.imgdb.cn/item/61528e8d2ab3f51d917118c6.jpg)

如果修改了组件的名称，那么在 include 属性中的名字应该是修改后的名字。

<b style="color: red">exclude 属性</b>：表示不缓存哪些组件。和 include 属性一起用没有意义，要不就是取消掉 include 属性的作用。要不就是多此一举。还会让代码可读性下降。
