---
title: Vue3快速入门(一)
date: 2022-02-21 20:59:46
keywords: Vue3
categories: "前端"
tags:
  - Vue
  - Vue3
---

# Vue3 快速入门(一)

实习进入公司后，简单安装 vscode、node、git 等必备工具后，导师直接问有没有学过 vue3，说了只学过 vue2 后，就给我分享了两篇文章，以及让我查看文档，快速从 vue2 加速到 vue3。本文将参考到很多文章、文档，部分借用的可能没有备注出来，侵权请联系。(不过就算是参考，例子我很多按自己的理解弄成自己以后看更容易理解的了，虽然也差不了多少)

顺带附上以前的笔记：[Vue2](https://clz.vercel.app/tags/Vue/)

## 1. 构建项目

开始的第一步，当然就是构建项目啦。这里就有一个重大的区别了，vue3 使用 web 开发构建工具[vite](https://clz.vercel.app/)，而不是 webpack

vite 优点：

- 无需打包，快速的冷服务器启动
- HMR(热更新)
- 按需编译

另外，vite 开启的服务器端口默认是 3000，而不是 8080

```shell
npm init vite@latest

# 使用yarn
yarn create vite
```

按需选择

![image-20220219154708582](https://s2.loli.net/2022/03/02/ZRuVP9eHDmi4Wqs.png)

<b style="color: red">项目启动的方式不再是` npm run serve`了，而是` npm run dev`</b>

![image-20220219155029403](https://s2.loli.net/2022/02/21/j4pBO5MFQqJnV89.png)

## 2. 入口文件变化

**Vue2**

```js
import Vue from "vue";
import App from "./App.vue";

import router from "./router";

Vue.config.productionTip = false;

new Vue({
  router,
  render: (h) => h(App),
}).$mount("#app");
```

**Vue3**

```js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

createApp(App).use(router).mount("#app");
```

Vue3 引入的不再是 Vue 构造函数，而是` createApp`工厂函数。

## 3. 组合式 API

### 3.1 set up

setup 函数是 Composition API(组合 API)的入口

<b style="color: red">在 setup 函数中定义的变量和方法需要 return 出去，才能在模板中使用</b>

实际上和 vue2 类似，只不过在 vue2 中数据在 data 函数中，方法在 methods 节点中，而 vue3 则是"更有人情味了"，和原生 js 更相似，数据和方法就是普通的变量和方法，只需要 return 出去，就能在模板中使用。

**Vue3**

```html
<template>
  <button @click="alertName">alert name</button>
  <p>{{ nickname }}</p>
</template>

<script>
  export default {
    name: "App",

    setup() {
      let nickname = "赤蓝紫";

      function alertName() {
        alert(`我是${nickname}`);
      }

      return {
        nickname,
        alertName,
      };
    },
  };
</script>
```

<b style="color: red">template 里不再必须要一个唯一的根标签了</b>，这个改进个人感觉很舒服。查了一下资料，发现是虽然看似没有根节点，但是只是 vue3 的根节点是一个虚拟节点，不会映射到一个具体节点。因为一棵树必须有一个根节点，所以使用虚拟节点作为根节点非常有用。

**Vue2**

```html
<template>
  <div id="app">
    <button @click="alertName">alert nickname</button>
    <p>{{ nickname }}</p>
  </div>
</template>

<script>
  export default {
    name: "App",
    data() {
      return {
        nickname: "赤蓝紫",
      };
    },
    methods: {
      alertName() {
        alert(`我是${this.nickname}`);
      },
    },
  };
</script>
```

### 3.2 ref

先看一下，下面这个例子

```html
<template>
  <button @click="changeName">change name</button>
  <p>{{ nickname }}</p>
</template>

<script>
  export default {
    name: "App",
    setup() {
      let nickname = "赤蓝紫";

      function changeName() {
        nickname = "clz";
        console.log(nickname);
      }

      return {
        nickname,
        changeName,
      };
    },
  };
</script>
```

点击后，发现数据改变了，但是 p 标签中的名字却没变，难道是 vue 的**数据驱动视图**失效了。

当然，并不是，只是 vue3 中多出了响应式数据和普通数据的区别，只有响应式数据才能驱动视图的改变。而上面的 nickname 只是字符串，不是响应式数据，试图自然也不会发生改变。

而将字符串变成响应式数据也非常简单，只需要引入并使用` ref`即可

```html
<template>
  <button @click="changeName">change name</button>
  <p>{{ nickname }}</p>
</template>

<script>
  import { ref } from "vue";

  export default {
    name: "App",
    setup() {
      let nickname = ref("赤蓝紫");

      function changeName() {
        nickname = "clz";
        console.log(nickname);
      }

      return {
        nickname,
        changeName,
      };
    },
  };
</script>
```

然而，还是不行，这是为什么呢？原因就是 ref 把 nickname 变成 RefImpl 的实例对象了，修改的时候要` .value`去修改，底层还是用的 get 和 set 去操作。

把上面 changeName 方法中的` nickname = "clz"`注释掉后，再点击按钮，就能知道变成 RefImpl 的实例对象了

![image-20220219171513259](https://s2.loli.net/2022/02/21/NbxsXdIfng37UDA.png)

最终版本：

```html
<template>
  <button @click="changeName">change name</button>
  <p>{{ nickname }}</p>
</template>

<script>
  import { ref } from "vue";

  export default {
    name: "App",
    setup() {
      let nickname = ref("赤蓝紫");

      function changeName() {
        nickname.value = "clz";
        // console.log(nickname)
      }

      return {
        nickname,
        changeName,
      };
    },
  };
</script>
```

修改要用` .value`来修改，为什么显示时不用` {{ nickname.value }}`来显示呢？这是因为 vue3 检测到是 ref 对象后，直接给你 nickname.value 了（还挺人性化）

### 3.3 reactive

```html
<template>
  <button @click="changeName">change name</button>
  <p>name: {{ people.name }}</p>
  <p>age: {{ people.age }}</p>
</template>

<script>
  import { ref } from "vue";

  export default {
    name: "App",
    setup() {
      let people = ref({
        name: "赤蓝紫",
        age: 21,
      });

      function changeName() {
        people.value.name = "clz";
      }

      return {
        people,
        changeName,
      };
    },
  };
</script>
```

乍一看和上面的 ref 中一样，但是实际上如果是将对象类型转换成响应式数据是应该使用函数` reactive`的，只是如果 ref 中是对象的话，会自动调用` reactive`而已。打印` people.value`可以发现不再是` RefImpl`对象了，而是` Proxy`对象。

![image-20220219173758962](https://s2.loli.net/2022/02/21/MRLKGiglNEsDhjf.png)

- 基本数据类型：根据` Object.defineProperty`里的` get`和` set`进行数据劫持来实现响应式
- 对象类型：通过` Proxy`来实现响应式

```html
<template>
  <button @click="changeName">change name</button>
  <p>name: {{ people.name }}</p>
  <p>age: {{ people.age }}</p>
  <p>hobby: {{hobbys[0]}}, {{hobbys[1]}}</p>
</template>

<script>
  import { reactive } from "vue";

  export default {
    name: "App",
    setup() {
      let people = reactive({
        name: "赤蓝紫",
        age: 21,
      });
      let hobbys = reactive(["音乐", "动漫"]);
      // let hobbys = ["音乐", "动漫"]

      function changeName() {
        people.name = "clz";
        hobbys[0] = "学习";
      }

      return {
        people,
        changeName,
        hobbys,
      };
    },
  };
</script>
```

上面有个问题：<b style="color: red">数组形式的不通过 reactive()转换成响应式也是响应式数据</b>，暂不知道原因

也可以按 vue2 中 data 的形式来写

```html
<template>
  <button @click="changeName">change name</button>
  <p>name: {{ data.people.name }}</p>
  <p>age: {{ data.people.age }}</p>
  <p>hobby: {{data.hobbys[0]}}, {{data.hobbys[1]}}</p>
</template>

<script>
  import { reactive } from "vue";

  export default {
    name: "App",
    setup() {
      let data = reactive({
        people: {
          name: "赤蓝紫",
          age: 21,
        },
        hobbys: ["音乐", "动漫"],
      });

      function changeName() {
        data.people.name = "clz";
        data.hobbys[0] = "学习";
      }

      return {
        data,
        changeName,
      };
    },
  };
</script>
```

**ref 和 reactive 的区别**：

|                                 ref                                  |                              reactive                              |
| :------------------------------------------------------------------: | :----------------------------------------------------------------: |
|                         定义**基本类型数据**                         |                     定义**对象或数组类型数据**                     |
| 通过` Object.defineProperty()`的` get`和` set`来实现响应式(数据劫持) | 通过` Proxy`实现响应式(数据劫持)，通过` Reflect`操作源代码内部数据 |
|                  操作数据需要` .value`，读取不需要                   |                  操作个读取数据都不需要` .value`                   |

### 3.3 computed

计算属性，和 vue2 差不多

```html
<template>
  r: <input type="text" v-model.number="color.r" /><br />
  g: <input type="text" v-model.number="color.g" /><br />
  b: <input type="text" v-model.number="color.b" /><br />
  rgb: <input type="text" v-model="color.rgb" :style="{color: color.rgb}" />
</template>

<script>
  import { reactive, computed } from "vue";

  export default {
    name: "App",
    setup() {
      let color = reactive({
        r: 255,
        g: 0,
        b: 0,
      });

      color.rgb = computed(() => {
        return `rgb(${color.r}, ${color.g}, ${color.b})`;
      });

      return {
        color,
      };
    },
  };
</script>
```

如果修改计算出来的东西，会报警告，因为计算属性是只读属性

![image-20220219200708560](https://s2.loli.net/2022/02/21/kGNxTC5dpZ19M3A.png)

实现可修改：

```html
<template>
  r: <input type="text" v-model="color.r" /><br />
  g: <input type="text" v-model="color.g" /><br />
  b: <input type="text" v-model="color.b" /><br />
  rgb: <input type="text" v-model="color.rgb" :style="{color: color.rgb}" />
</template>

<script>
  import { reactive, computed } from "vue";

  export default {
    name: "App",
    setup() {
      let color = reactive({
        r: 255,
        g: 0,
        b: 0,
      });

      color.rgb = computed({
        get() {
          console.log(color.r);
          return `rgb(${color.r},${color.g},${color.b})`;
        },
        set(value) {
          let rgbList = value.split(",");
          color.r = rgbList[0].slice(4);
          color.g = rgbList[1];
          color.b = rgbList[2].slice(0, -1);
        },
      });

      return {
        color,
      };
    },
  };
</script>
```

<b style="color: red">实现计算属性可修改的关键</b>：computed()参数为一个对象，对象中有一个 get 方法用来获取值，set 方法用来修改值

### 3.4 watch

监听器

```html
<template>
  <div class="home">
    <h1>当前数字为：{{num}}</h1>
    <button @click="num++">+1</button>
  </div>
</template>

<script>
  import { ref, watch } from "vue";

  export default {
    name: "Home",
    setup() {
      let num = ref(0);
      watch(
        num, // 如果想监听多个数据，则第一个参数是要监听的数据数组，其中newValue、oldValue也是数组，第一个元素就是监听的第一个参数
        (newValue, oldValue) => {
          console.log(`数字增加了，现在的值${newValue}, 原值${oldValue}`);
        }
        // {
        //   immediate: true,
        //   deep: true
        // } // 第三个参数就是选项设置，可设置立即监听、深度监听
      );

      return {
        num,
      };
    },
  };
</script>
```

```html
<template>
  <div class="home">
    <h1>age:{{people.age}}</h1>
    <button @click="people.age++">年龄加1</button>
    <h1>salary:{{people.job.salary}}</h1>
    <button @click="people.job.salary+=100">薪水加100</button>
  </div>
</template>

<script>
  import { reactive, watch } from "vue";
  export default {
    name: "Home",
    setup() {
      let people = reactive({
        name: "赤蓝紫",
        age: 21,
        job: {
          salary: -10,
        },
      });
      watch(
        people,
        (newValue, oldValue) => {
          console.log(`信息改变了`, newValue, oldValue); // 直接监听reactive所定义的一个响应式数据，会出现oldValue也是更新后的值，且默认开启深度监听，还无法关闭
        },
        {
          deep: false,
        }
      );

      // 监听reactive所定义的一个响应式数据中的某个属性
      // 如果是对象还需要开启深度监听，才能监听到变化。
      // 或者变为监听people.job.salary(直接手动直接指出最终的目标)
      // watch(
      //   () => people.job,
      //   (newValue, oldValue) => {
      //     console.log(`工作信息改变了`, newValue, oldValue)
      //   },
      //   {
      //     deep: true
      //   }
      // )

      return {
        people,
      };
    },
  };
</script>
```

![image-20220219220716205](https://s2.loli.net/2022/02/21/ZBVr5nihTLOuwmp.png)

<b style="color: red">如果监听器用来监听 reactive 定义的响应式数据，那么无法获取到旧数据，而且默认开启深度监听，无法关闭深度监听</b>

**watchEffect**：

- 默认开启了立即更新(`immediate: true`)
- 用到谁就监听谁

```html
<template>
  <div class="home">
    <h1>age:{{people.age}}</h1>
    <button @click="people.age++">年龄加1</button>
    <h1>salary:{{people.job.salary}}</h1>
    <button @click="people.job.salary+=100">薪水加100</button>
  </div>
</template>

<script>
  import { reactive, watchEffect } from "vue";
  export default {
    name: "Home",
    setup() {
      let people = reactive({
        name: "赤蓝紫",
        age: 21,
        job: {
          salary: -10,
        },
      });
      watchEffect(() => {
        const salary = people.job.salary;
        console.log("工资变更");
      });

      return {
        people,
      };
    },
  };
</script>
```

### 3.5 生命周期钩子

![image-20220219223623431](https://s2.loli.net/2022/02/21/ecLryuxfdEZmqhM.png)

vue3 中，**beforeDestroy**改为**beforeUnmount**，**destroyed**改为**unmounte**

<b style="color: red">beforeCreate 和 created 没有 API，因为 setup 实际上就相当于这两个生命周期函数</b>

使用示例：

```html
<template>
  <h2>{{num}}</h2>
  <button @click="num++">+1</button>
</template>

<script>
  import { onMounted, onUpdated } from "vue";
  import { ref } from "vue";

  export default {
    name: "Home",
    setup() {
      onMounted(() => {
        console.log("onMounted");
      }),
        onUpdated(() => {
          console.log("数据更新啦");
        });

      let num = ref(0);

      return {
        num,
      };
    },
  };
</script>
```

### 3.6 toRef 和 toRefs

toRef 就是把数据变成 ref 类型的数据，` toRefs`就是将多个数转换成响应式数据

先引用一下之前的例子：

```html
<template>
  <div class="home">
    <h1>age:{{people.age}}</h1>
    <button @click="people.age++">年龄加1</button>
    <h1>salary:{{people.job.salary}}</h1>
    <button @click="people.job.salary+=100">薪水加100</button>
  </div>
</template>

<script>
  import { reactive, watchEffect } from "vue";
  export default {
    name: "Home",
    setup() {
      let people = reactive({
        name: "赤蓝紫",
        age: 21,
        job: {
          salary: -10,
        },
      });
      watchEffect(() => {
        const salary = people.job.salary;
        console.log("工资变更");
      });

      return {
        people,
      };
    },
  };
</script>
```

仔细观察，可以发现` template`中，使用了很多次` people.`，于是想偷一下懒,return 的时候耍点小聪明

```js
return {
  age: people.age,
  salary: people.job.salary,
};
```

哦豁，点击按钮不再能改变数据了，原因就是因为 return 出去的数据不是响应式，而是 number，自然不能改变。验证也很简单，只要在` watchEffect()`中顺便打印出` people.age`就行了。

通过` toRef`就可以实现自动修改` people`里的数据，不要忘记引入` toRef`了

```js
return {
  age: toRef(people, "age"),
  salary: toRef(people.job, "salary"),
};
```

这种时候，有可能会想到使用 ref 就可以了，即以下形式

```js
return {
  age: ref(people.age), // ref()不会改变people里的数据，而toRef可以修改
  salary: ref(people.job.salary),
};
```

这样子，乍一看，效果确实一样，但是，实际上的数据并没有发现改变，通过监听器就可以发现

![vue3](https://s2.loli.net/2022/02/21/CBnTzxYiJtmjEDs.gif)

为什么呢？实际上使用 ref 的话，有类似于 new 出来一个对象，new 出来的对象自然和原来的数据没有什么实质上的联系

使用` toRefs`就可以稍微偷一下懒

```js
return {
  ...toRefs(people), // 解构，只能解构一层，所以深层的还是要写
  ...toRefs(people.job),
};
```

## 4. 其他改变

- 移除` keyCode`作为` v-on`的修饰符

  ```html
  <input v-on:keyup.13="submit" />
  <input v-on:keyup.enter="submit" />

  <!-- vue2上面两种形式都支持，vue3只支持下面那种，即只能通过按钮别名 -->
  ```

- 移除` native`作为` v-on`的修饰符

- 移除` filter`过滤器

[Vue 3 迁移策略笔记](https://blog.csdn.net/weixin_44869002/category_10771155.html)

### 4.1 $refs

```html
<template>
  <button @click="getValue" ref="btn">点击</button>
</template>

<script>
  import { getCurrentInstance, nextTick, reactive, ref } from "vue";

  export default {
    setup() {
      const ci = getCurrentInstance();
      const { proxy } = getCurrentInstance();
      // console.log(getCurrentInstance())    //只有在setup()这一直接块才可以，如果在函数中，则会得到null

      function getValue() {
        console.log(ci.refs.btn);
        console.log(proxy.$refs.btn); // 也可以通过proxy.$refs来获取
      }

      return {
        getValue,
      };
    },
  };
</script>
```

### 4.2 nextTick

nextTick()：在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

```html
<template>
  <button @click="change" ref="btn">{{msg}}</button>
</template>

<script>
  import { getCurrentInstance, nextTick, ref } from "vue";

  export default {
    setup() {
      let { proxy } = getCurrentInstance();

      let msg = ref("Hi");
      function change() {
        const btn = proxy.$refs.btn;
        msg.value = "Hello";

        console.log("直接打印:", btn.innerText);

        nextTick(() => {
          console.log("nextTick：", btn.innerText);
        });
      }

      return {
        msg,
        change,
      };
    },
  };
</script>
```

### 4.3 teleport

使用`<teleport>`，可以通过`to`将 teleport 下的 html 传送到指定位置(如传送到`body`中)。

```html
<template>
  <div class="one">
    <div class="two">
      <teleport to="body">
        <div class="three">
          <div class="four"></div>
        </div>
      </teleport>
    </div>
  </div>
</template>

<script>
  export default {
    name: "App",
  };
</script>
```

![image-20220220141404043](https://s2.loli.net/2022/02/20/c86uJK35AQOhm2f.png)

### 4.4 路由

app.vue

```html
<template>
  <router-view></router-view>
</template>
<script>
  import { useRoute, useRouter } from "vue-router";

  export default {
    setup() {
      const route = useRoute();
      const router = useRouter();

      console.log(route.path);
      router.push("/home");
    },
  };
</script>
```

main.js

```js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

createApp(App).use(router).mount("#app");
```

router \ index.js

```js
import { createRouter, createWebHashHistory } from "vue-router";

const routes = [
  {
    path: "/home",
    name: "home",
    component: () => import("../components/home.vue"),
  },
  {
    path: "/login",
    name: "login",
    component: () => import("../components/login.vue"),
  },
];

export default createRouter({
  history: createWebHashHistory(),
  routes,
});
```

<b style="color: red">注意：</b>

- <b style="color: red">在模板中仍然可以访问 `$router` 和 `$route`，所以不需要在 `setup` 中返回 `router` 或 `route`</b>

- 从` vue-router`中引入的`useRoute`,`useRouter`相当于 vue2 的 `this.$route`，`this.$router`
- 引入组件时，**必须**加上` .vue`后缀

编程式导航传参

**`params` 不能与 `path` 一起使用，而应该使用`name`**

```html
<template>
  <router-view></router-view>
</template>
<script>
  import { useRoute, useRouter } from "vue-router";

  export default {
    setup() {
      const route = useRoute();
      const router = useRouter();

      // query编程式导航传参
      // router.push({
      //   path: "/home",
      //   query: {
      //     id: 666
      //   }
      // })

      // params编程式导航传参
      router.push({
        name: "login", // 需要使用命名路由
        params: {
          // 路由规则那里也要配成path: "/login/:id",
          id: 666,
        },
      });
    },
  };
</script>
```

### 4.5 导航守卫

#### 4.5.1 局部导航守卫

home.vue

```html
<template> home </template>

<script>
  import { onBeforeRouteLeave } from "vue-router";

  export default {
    setup() {
      onBeforeRouteLeave((to, from) => {
        // 默认可前往，可通过return false禁止前往
        console.log("去", to);
        console.log("来自", from);
      });
    },
  };
</script>
```

#### 4.5.2 全局导航守卫

main.js

```js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

router.beforeEach((to, from) => {
  // next是可选参数，可写可不写，return false是取消导航，否则意味着通过验证
  console.log("去", to);
  console.log("来自", from);
  // return false
});

createApp(App).use(router).mount("#app");
```

## 5. 法宝(setup 语法糖)

<b style="color: red">Vue3.0 通过 setup()函数，需要把数据和方法 return 出去才能使用</b>，但是 Vue3.2 中，只需要在 srcipt 标签上加上` setup`属性，这样子就无需 return，template 就可以直接使用了

```html
<template>
  <button @click="alertName">alert name</button>
  <p>{{ nickname }}</p>
</template>

<script setup>
  let nickname = "赤蓝紫";

  function alertName() {
    alert(`我是${nickname}`);
  }
</script>
```

参考：

[vue3 保姆级教程](https://juejin.cn/post/7030992475271495711)

[官方文档](https://v3.cn.vuejs.org/)
