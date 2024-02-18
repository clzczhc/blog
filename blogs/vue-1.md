---
title: Vue学习笔记(一)
date: 2021-10-01 20:13:30
categories: "前端"
tags:
  - Vue
---

# Vue 学习笔记(一)

## 1. vue 简介

Vue 是一套**用于构建用户界面**的前端**框架**

### 1.1 vue 的两个特性

1. **数据驱动视图**

   - 使用了 vue 的页面，vue 会监听数据的变化，**自动**重新渲染页面的结构
   - ![](https://pic.imgdb.cn/item/614957ae2ab3f51d917d98ae.jpg)
   - 数据驱动视图是**单向的数据绑定**，即只能由数据来影响页面结构

2. **双向数据绑定**

   - <b style="color: red">填写表单</b>时，双向数据绑定可以让开发者在**不操作 DOM 的前提下**，自动把用户填写的内容同步到数据源中

     ![](https://pic.imgdb.cn/item/614958cb2ab3f51d917f049a.jpg)

### 1.2 MVVM

vue 实现**数据驱动视图**和**双向数据绑定**的核心原理。

M 指的是 Model，V 指的是 View，VM 指的是 ViewModel

​ **Model**：表示当前页面渲染时依赖的数据源

​ **View**：表示当前页面所渲染 DOM 结构

​ **ViewModel**：表示 vue 的实例，是 MVVM 的核心

**MVVM 的工作原理**：ViewModel 作为 MVVM 的核心，它把当前页面的**数据源**(Model)和**页面的结构**(View)连在一起。

- 当数据源发生变化时，会被 ViewModel 监听到，VM 会自动更新页面的结构
- 当表单元素的值发生变化时，也会被 VM 监听到，VM 会把更新的值自动同步到数据源(Model)中

![](https://pic.imgdb.cn/item/61495b702ab3f51d9182657f.jpg)

## 2. vue 的基本使用

**步骤**

1. 导入 vue.js 文件
2. 在页面中声明要被 vue 操作的 DOM 区域
3. 创建 vue 实例对象

![](https://pic.imgdb.cn/item/61873bf12ab3f51d9112f478.jpg)

## 3. vue 的指令

**指令**是 vue 为开发者提供的**模板语法**，用于**辅助开发者渲染页面的基本结构**

按照用途可分为 6 大类：

1. 内容渲染指令
2. 属性绑定指令
3. 事件绑定指令
4. 双向绑定指令
5. 条件绑定指令
6. 列表渲染指令

### 3.1 内容渲染指令

1. **v-text**

   会覆盖元素内默认内容

   ```html
   <div id="app">
     <p v-text="username">姓名：</p>
     <!-- 这里的姓名会被直接覆盖成下面数据源中的username -->
   </div>

   <script src="./lib/vue-2.6.12.js"></script>
   <script>
     const vm = new Vue({
       el: "#app", //表示操作的区域，值是选择器
       data: {
         //Model数据源
         username: "clz",
       },
     });
   </script>
   ```

2. **插值表达式`{{}}`**

   不会覆盖元素内默认内容

   ```html
   <div id="app">
     <p>姓名：{{ username }}</p>
     <!-- 下面数据源中的username会被渲染到姓名后面 -->
   </div>

   <script src="./lib/vue-2.6.12.js"></script>
   <script>
     const vm = new Vue({
       el: "#app", //表示操作的区域，值是选择器
       data: {
         //Model数据源
         username: "clz",
       },
     });
   </script>
   ```

3. **v-html**

   可以把包含 html 标签的字符串渲染成页面的 HTML 元素

   ```html
   <div id="app">
     <p>{{ test_v_html }}</p>
     <p v-text="test_v_html"></p>
     <p v-html="test_v_html"></p>
   </div>

   <script src="./lib/vue-2.6.12.js"></script>
   <script>
     const vm = new Vue({
       el: "#app", //表示操作的区域，值是选择器
       data: {
         //Model数据源
         test_v_html: '<b style="color: red">Hello!</b>',
       },
     });
   </script>
   ```

   结果：

   ![](https://pic.imgdb.cn/item/614962692ab3f51d918b4755.jpg)

### 3.2 属性绑定指令

为**元素的属性**动态绑定属性值时，需要用到**v-bind**属性绑定指令

<b style="color: red">简写形式":"</b>

![](https://pic.imgdb.cn/item/614966832ab3f51d919111c5.jpg)

vue 提供的模板渲染语法，除了支持**绑定简单的数据**之外，还**支持 Javascript 表达式的运算**

```html
<div id="app">
  <div>1 + 1 = {{ 1 + 1 }}</div>

  <div>{{ hello }} 反转后: {{ hello.split('').reverse().join('') }}</div>

  <div :title="'box' + index">鼠标悬浮一下</div>
  <!-- 使用属性绑定指令时，进行字符串拼接的字符串需要使用嵌套引号 -->
  <!-- 否则，会到data中找要渲染的数据，找不到会报错 -->
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      hello: "Hello World!",
      index: 3,
    },
  });
</script>
```

### 3.3 事件绑定指令

vue 提供**v-on 事件绑定指令**，用来辅助程序员为 DOM 元素绑定事件监听。

```html
<div id="app">
  <h2>count的值是: {{ count }}</h2>
  <button v-on:click="addCount">+1</button>
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      count: 0,
    },
    methods: {
      //事件处理函数放在methods节点里
      // addCount: function () {
      //     this.count++;
      // }
      addCount() {
        //简便方式
        this.count++;
      },
    },
  });
</script>
```

<b style="color: red">v-on 事件绑定指令简写形式"@"</b>，而且如果事件处理函数的代码只有一行，可以直接写在行内

```html
<div id="app">
  <h2>count的值是: {{ count }}</h2>
  <button @click="count += 1">+1</button>
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      count: 0,
    },
  });
</script>
```

**事件参数对象**：通过 e.target 使被点击的按钮变色

```html
<div id="app">
  <h2>count的值是: {{ count }}</h2>
  <button @click="addCount">+1</button>
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      count: 0,
    },
    methods: {
      addCount(e) {
        this.count++;
        e.target.style.backgroundColor = "red";
      },
    },
  });
</script>
```

**传参**：

```html
<div id="app">
  <h2>count的值是: {{ count }}</h2>
  <button @click="addCount(3)">+3</button>
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      count: 0,
    },
    methods: {
      addCount(step) {
        this.count += step;
      },
    },
  });
</script>
```

可实现传参后，可以发现事件参数对象被参数覆盖了，而 vue 提供一个特殊变量**$event**，用来表示**原生的事件参数对象 event**，然后手动把**$event**当成参数传进去用。

```html
<div id="app">
  <h2>count的值是: {{ count }}</h2>
  <button @click="addCount($event, 3)">+3</button>
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      count: 0,
    },
    methods: {
      addCount(e, step) {
        e.target.style.backgroundColor = "red";
        this.count += step;
      },
    },
  });
</script>
```

**事件修饰符**：vue 提供了事件修饰符的概念，辅组程序员更方便地**对事件的触发进行控制**

![](https://pic.imgdb.cn/item/614982882ab3f51d91bd5be9.jpg)

![](https://pic.imgdb.cn/item/614982232ab3f51d91bcb021.jpg)

**按键修饰符**：在监听**键盘事件**时，如果需要**判断详细的按键**，可以为键盘相关事件添加**按键修饰符**

```html
<div id="app">
  <input type="text" @keyup.esc="clearIpt" />
  <!-- 通过按键修饰符可以判断详细的按键 -->
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
    },
    methods: {
      clearIpt(e) {
        e.target.value = "";
      },
    },
  });
</script>
```

### 3.4 双向绑定指令

用来辅助开发者在**不操作 DOM**的前提下，**快速获取表单的数据**

```html
<div id="app">
  <h2>用户名是: {{ username }}</h2>

  <input type="text" v-model="username" />
  <!-- 在input中填入的数据会让h2中的数据实时变化 -->
  <!-- 可以添加修饰符.lazy,实现在"change"时才更新，比如，用户把焦点移出input了 -->
  <input type="text" v-model.lazy="username" />
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      username: "clz",
    },
  });
</script>
```

**v-model 指令的修饰符**

![](https://pic.imgdb.cn/item/6149894c2ab3f51d91c870ec.jpg)

```html
<div id="app">
  <input type="text" v-model.number="n1" /> +
  <input type="text" v-model.number="n2" /> =
  <span>{{ n1 + n2 }}</span>
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      n1: 1,
      n2: 2,
    },
  });
</script>
```

### 3.5 条件渲染指令

条件渲染指令用来辅助开发者**按需来控制 DOM 的显示与隐藏**。

有两个条件渲染指令

- v-if
- v-show

```html
<div id="app">
  <!-- v-if和v-show都是根据"="后的部分为true或false来决定是显示还是隐藏-->
  <!-- 为true时显示，为false时隐藏 -->
  <p v-if="flag">v-if</p>
  <p v-if="flag">v-if</p>
  <p v-show="flag">v-show</p>
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      flag: false,
    },
    methods: {},
  });
</script>
```

区别：

实现原理不同：

- v-if 是通过**动态创建或移除 DOM 元素**来控制元素在页面上的显示与隐藏，隐藏后，还贴心的把隐藏的节点所在的位置变为空注释，暗示有东西藏着
- v-show 指令会动态为元素**添加或移除 style="display: none;"样式**，来控制元素的显示与隐藏

性能消耗不同：

- v-if 的切换开销更高，而 v-show 的初始渲染开销更高
- **需要频繁切换**，使用 v-show
- **运行时条件很少变化**，用 v-if

![](https://pic.imgdb.cn/item/61498d7d2ab3f51d91cf68e5.jpg)

v-if 可以单独使用，也可以搭配 v-else、v-else-if 使用

```html
<div id="app">
  <select v-model="type">
    <option value="A">A</option>
    <option value="B">B</option>
    <option value="C">C</option>
    <option value="D">D</option>
    <option value="E">E</option>
  </select>
  <p v-if="type === 'A'">优秀</p>
  <p v-else-if="type === 'B'">良好</p>
  <p v-else-if="type === 'C'">及格</p>
  <p v-else>差</p>
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      type: "A",
    },
    methods: {},
  });
</script>
```

### 3.6 列表渲染指令

v-for 列表渲染指令，用于实现**基于一个数组来渲染一个列表结构**。v-for 指令需要使用**item in items**形式的特殊语法，items 是要循环的数组，item 是循环的每一项

v-for 指令支持一个可选的第二个参数，即当前项的索引。语法为` (item, index) in items`

<b style="color: red">需要为每一项提供唯一的 key 属性，用于维护列表的状态</b>

```html
<div id="app">
  <table class="table table-bordered table-hover table-striped">
    <thead>
      <th>索引</th>
      <th>Id</th>
      <th>姓名</th>
    </thead>
    <tbody>
      <tr v-for="(user, index) in list" :key="user.id">
        <td>{{ index }}</td>
        <td>{{ user.id }}</td>
        <td>{{ user.name }}</td>
      </tr>
    </tbody>
  </table>
</div>

<script src="./lib/vue-2.6.12.js"></script>
<script>
  const vm = new Vue({
    el: "#app", //表示操作的区域，值是选择器
    data: {
      //Model数据源
      list: [
        {
          id: 1,
          name: "张三",
        },
        {
          id: 2,
          name: "李四",
        },
        {
          id: 3,
          name: "王五",
        },
      ],
    },
    methods: {},
  });
</script>
```

**key 的注意事项**：

1. key 的值只能是\*字符串或**数字**类型

2. key 的值必须具有**唯一性**

3. 建议把**数据项的 id 属性**作为 key 的值(因为 id 属性的值具有唯一性)

4. 使用 v-for 指令时**一定要指定 key 的值**(既可以提升性能，又可以防止列表状态混乱)

5. 使用 index 的值作为 key 的值没有意义(因为 index 的值不具有唯一性)

   index 的值看起来像是具有唯一性，但是这个是假唯一性

   例子：

   现在又一个数组 list，list=["张三"，"李四"]，假如选择 index 作为 key 值的话，选择 key=1,即现在选中了李四，这个时候在数组的头那里插入一个新人"王五"，那么李四的 key 值将会变成 2,而之前选的是 key 为 1 的，所以，张三就篡位了。

## 4. 过滤器

过滤器常用于**文本的格式化**，可用于**插值表达式**和**v-bind 属性绑定**

过滤符由**管道符"|"**进行调用

![](https://pic.imgdb.cn/item/614bd7912ab3f51d91a452e0.jpg)

<b style="color: red">在 filters 节点下定义的过滤器，是私有过滤器，只能在当前的 vm 实例所控制的 el 区域内可以使用。</b>要实现**多个 vue 实例之间共享过滤器**，可以定义**全局过滤器**。

<b style="color: red">注意，全局过滤器要放在要用到的 vm 实例之前</b>

![](https://pic.imgdb.cn/item/614bdc422ab3f51d91a963a7.jpg)

<b style="color: red">注意，查看上面的结果可以发现，只有 vm2 控制的区域后面会跟着"全局版本",这是因为 vm 也有一个私有过滤器 mychange，所以 vm 就直接用自己的了（懒得再去全局那里拿来用）</b>

过滤器可以**串联的**进行调用。

```js
{
  {
    message | fileterA | filterB;
  }
} //先把message的值给过滤器filterA处理，之后把处理的结果给B处理，最后再把最后的结果渲染到页面上
```

**过滤器可以传参**：过滤器本质是函数，可以传参，只不过，<b style = "color: red">第一个参数已经规定好了，是管道符"|"之前的数据。</b>

## 5. 侦听器

允许开发者监视数据的变化，从而**针对数据的变化做特定的操作**

```html
<div id="app">
  <input type="text" v-model="username" />
</div>

<script src="./lib/vue-2.6.12.js"></script>

<script>
  const vm = new Vue({
    el: "#app",
    data: {
      username: "admin",
    },
    watch: {
      username(newVal, oldVal) {
        //要监听谁，就把谁的名字作为方法名，新值在前，旧值在后
        console.log(newVal, oldVal);
      },
    },
  });
</script>
```

### 5.1 immediate 选项

**默认情况下，组件在初次加载完时，不会调用 watch 侦听器**。想要 watch 侦听器**立即被调用**，需要把**immediate**选项变为 true(默认值为 false)，<b style="color: red">这个时候的侦听器应该是对象形式的。</b>

```html
<div id="app">
  <input type="text" v-model="username" />
</div>

<script src="./lib/vue-2.6.12.js"></script>

<script>
  const vm = new Vue({
    el: "#app",
    data: {
      username: "admin",
    },
    watch: {
      username: {
        //对象形式
        handler(newVal, oldVal) {
          console.log(newVal, oldVal);
        },
        immediate: true, //实现初次加载完成时也会调用监听器
      },
    },
  });
</script>
```

### 5.2 deep 选项

如果 watch 监听的是一个对象，如果对象中的属性值发生了变化，则无法被监听到。（就像是监视一个人，只能看到他干了什么，没法看到他里面的消化系统在干什么），这个时候需要把**deep 选项**变为 true

```html
<div id="app">
  <input type="text" v-model="info.username" />
</div>

<script src="./lib/vue-2.6.12.js"></script>

<script>
  const vm = new Vue({
    el: "#app",
    data: {
      info: {
        username: "admin",
      },
    },
    watch: {
      info: {
        handler(newVal) {
          console.log(newVal.username);
        },
        immediate: true,
        deep: true,
      },
    },
  });
</script>
```

上面的例子，如果**只是想监听单个属性的变化**，可以不变化 deep 选项，按以下方式即可

```html
<div id="app">
  <input type="text" v-model="info.username" />
</div>

<script src="./lib/vue-2.6.12.js"></script>

<script>
  const vm = new Vue({
    el: "#app",
    data: {
      info: {
        username: "admin",
      },
    },
    watch: {
      "info.username": {
        //监听单个属性需要用字符串形式
        handler(newVal) {
          //这个时候的新值newVal，就是属性变化后的值
          console.log(newVal);
        },
        immediate: true,
      },
    },
  });
</script>
```

## 6. 计算属性

计算属性是指通过一系列计算之后，最终得到一个**属性值**，这个**动态计算出来的属性值**可以被模板结构或 methods 方法使用。

**特点**：

1. 计算属性声明的时候被定义为方法，但是计算属性的**本质是一个属性**
2. 只要**计算属性依赖的数据源变化**了，那么计算属性就会**自动重新求值**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Myself</title>
    <style>
      .box {
        width: 100px;
        height: 100px;
        border: 1px solid #ccc;
      }
    </style>
  </head>

  <body>
    <div id="app">
      <div>
        <span>R: </span>
        <input type="text" v-model.number="r" />
      </div>

      <div>
        <span>G: </span>
        <input type="text" v-model.number="g" />
      </div>

      <div>
        <span>B: </span>
        <input type="text" v-model.number="b" />
      </div>

      <hr />

      <div class="box" :style="{backgroundColor: rgb}">
        {{ rgb }}
        <!-- 使用时，就和普通属性一样用法 -->
      </div>
      <button @click="showColor">按钮</button>
    </div>

    <script src="./lib/vue-2.6.12.js"></script>

    <script>
      const vm = new Vue({
        el: "#app",
        data: {
          r: 0,
          g: 0,
          b: 0,
        },
        methods: {
          showColor() {
            console.log(this.rgb);
          },
        },
        computed: {
          rgb() {
            //计算属性，声明时是函数形式
            return `rgb(${this.r}, ${this.g}, ${this.b})`;
          },
        },
      });
      console.log(vm);
    </script>
  </body>
</html>
```

![](https://pic.imgdb.cn/item/614c3c872ab3f51d912f40cc.jpg)

## 7. axios

axios 是一个专注于数据请求的库。

**基本使用**：

安装命令

` npm install axios -S`

```html
<script src="lib/axios.js"></script>
<script>
  //axios()方法返回一个promise对象
  axios({
    method: "get", //请求方式
    url: "http://www.liulongbin.top:3006/api/getbooks", //请求路径
  }).then((result) => {
    console.log(result); //这里得到的result不是得到的数据，result.data才是，result是数据被axios包装后得到的
    console.log(result.data);
  });
</script>
```

1. 发起 get 请求

   ```html
   <script src="lib/axios.js"></script>
   <script>
     //axios()方法返回一个promise对象
     axios({
       method: "get", //请求方式
       url: "http://www.liulongbin.top:3006/api/getbooks", //请求路径
       params: {
         //查询参数，即请求方式为get时的参数
         id: 1,
       },
       // data: {
       //     //请求体参数：post时的参数，和params二选一
       // }
     }).then((result) => {
       console.log(result.data);
     });
   </script>
   ```

2. 发起 post 请求

   ```html
   <button>发起POST请求</button>
   <script src="lib/axios.js"></script>
   <script>
     document.querySelector("button").addEventListener("click", async () => {
       // //axios()方法返回一个promise对象
       // axios({
       //     method: 'post', //请求方式
       //     url: 'http://www.liulongbin.top:3006/api/post', //请求路径

       //     data: {
       //         name: '张三',
       //         age: 11
       //     }
       // }).then((result) => {
       //     console.log(result.data);
       // })

       //不用then，简化版本
       //如果调用某个方法的返回值时Promise实例，则前面可以加await
       //await只能在被async修饰的方法中
       const { data: result } = await axios({
         //使用解构赋值,把data属性结构出来,并重命名为result
         method: "post",
         url: "http://www.liulongbin.top:3006/api/post",

         data: {
           name: "张三",
           age: 11,
         },
       });
       console.log(result);
     });
   </script>
   ```

### 7.1 axios.get()

语法：

```js
axios.get("url地址", {
  params: {
    get参数,
  },
});
```

```html
<button class="get">get</button>
<script src="lib/axios.js"></script>
<script>
  document.querySelector(".get").addEventListener("click", async () => {
    const { data: result } = await axios.get(
      "http://www.liulongbin.top:3006/api/getbooks",
      {
        params: {
          id: 1,
        },
      }
    );
    console.log(result.data);
  });
</script>
```

### 7.2 axios.post()

语法：

```js
axios.post("url", {
  请求体参数, //注意，这里不需要写在data里面
});
```

```html
<button class="post">post</button>
<script src="lib/axios.js"></script>
<script>
  document.querySelector(".post").addEventListener("click", async () => {
    const { data: result } = await axios.post(
      "http://www.liulongbin.top:3006/api/post",
      {
        name: "张三",
        age: 21,
      }
    );
    console.log(result);
  });
</script>
```

学习链接：

[黑马程序员 Vue 全套视频教程](https://www.bilibili.com/video/BV1zq4y1p7ga)

[Vue.js (vuejs.org)](https://vuejs.org/)
