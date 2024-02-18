---
title: 刷新页面后this.$route.params 为空
date: 2022-03-08 16:14:47
categories: "前端"
tags:
  - Vue
  - Vue3
  - Vue Router
---

# 刷新页面后 this.$route.params 为空

深入学习` vue-router`时，按官方文档的教程看下来，结果发现刷新页面后，打印的`this.$route.params `为空

## Vue2

### 问题复现

路由配置：

```js
import Vue from "vue";
import VueRouter from "vue-router";

Vue.use(VueRouter);

const router = new VueRouter({
  routes: [
    {
      path: "/",
      redirect: "/home",
    },
    {
      path: "/home",
      name: "Home",
      component: () => import("../components/Home.vue"),
    },
    {
      path: "/user/:id",
      name: "User",
      component: () => import("../components/User.vue"),
    },
  ],
});

export default router;
```

App.vue

```html
<template>
  <div>
    <router-view></router-view>
  </div>
</template>

<script>
  export default {
    name: "App",
    created() {
      console.log(this.$route.params);
    },
  };
</script>
```

![image-20220302152502581](https://s2.loli.net/2022/03/08/FWeYxVU8bqB29yJ.png)

### 解决方案

#### 1. 在导航守卫中获取

```js
router.beforeEach((to, from, next) => {
  console.log(to.params);
  next();
});
```

![vue-router2](https://s2.loli.net/2022/03/08/dqoSPzYgaDeFAUy.gif)

#### 2. 在跳转后的页面获取，而不是在 app.vue 中获取

User.vue

```html
<template>
  <h2>User</h2>
</template>

<script>
  export default {
    created() {
      console.log(this.$route.params);
    },
  };
</script>

<style scoped>
  h2 {
    color: purple;
  }
</style>
```

#### 3. 在` updated`生命周期钩子中获取(可能实际开发用不上)

为什么会出这个问题呢？

以下是个人现阶段的理解。(可能有误)

结论：<b style="Color: red">此时打印` this.$route.params`应该在` updated`生命周期钩子中打印</b>

首先先在` created`和` mounted`钩子中打印` this.$route`看一下情况。

![image-20220302153712535](https://s2.loli.net/2022/03/08/pOfQMSabTkjCt3Z.png)

发现，信息不符合。猜测可能是组件创建、渲染阶段时，路由还没有跳转，所以打印的信息不对。路由跳转后，修改数据` this.$route`是在数据更新阶段，所以获取最新的路由信息应该在` updated`中获取。

![image-20220302154158334](https://s2.loli.net/2022/03/08/oBSyKRIdsVfhw83.png)

问题解决

## Vue3

首先，路由配置也不太一样

```js
import { createRouter, createWebHashHistory } from "vue-router";

const routes = [
  {
    path: "/",
    redirect: "/home",
  },
  {
    path: "/home",
    name: "Home",
    component: () => import("../components/Home.vue"),
  },
  {
    path: "/user/:id",
    name: "User",
    component: () => import("../components/User.vue"),
  },
];

const router = new createRouter({
  history: createWebHashHistory(),
  routes,
});

export default router;
```

### 解决方案

#### 1. 在导航守卫中获取

和 Vue2 的相同。

#### 2. 在跳转后的页面获取，而不是在 app.vue 中获取

**这个在开发中用到的可能性还大一些。毕竟开发时每个页面都需要路由信息的很少，都需要的话就可以采用上面在导航守卫中获取的做法**

User.vue

```html
<template>
  <h2>User</h2>
</template>

<script setup>
  import { useRoute } from "vue-router";
  import { onUpdated } from "vue";

  const route = useRoute();

  console.log(route.params);
</script>

<style scoped>
  h2 {
    color: purple;
  }
</style>
```

效果和上图一样

#### 3. 强行实现(不建议)

Vue3 中，针对 Vue2 的解决方案 3 不再有效。在 Vue3 中，路由的变化不再属于是数据的更新，所以也不会触发` onUpdated`钩子

app.vue

```html
<template>
  <router-link to="/home">首页</router-link>
  <router-view></router-view>
</template>

<script setup>
  import { useRoute } from "vue-router";
  import { onUpdated } from "vue";

  const route = useRoute();

  // console.log(route.params)

  onUpdated(() => {
    console.log(route.params);
  });
</script>
```

![vue-router](https://s2.loli.net/2022/03/08/6mokixOAbLd2PrR.gif)

那么怎么解决呢？

<br />

这个只是个人学习时，想到的暴力法。(现在也只会这个暴力法，开发时应该是嗤之以鼻的做法)

app.vue

```html
<template>
  <router-view></router-view>
</template>

<script setup>
  import { useRoute } from "vue-router";
  import { onUpdated } from "vue";

  const route = useRoute();

  setTimeout(() => {
    console.log(route.params);
  }, 200);
</script>
```

效果还是同上

## 最后的坑(又能解释的希望评论告知)

app.vue

```html
<template>
  params: {{ route.params }}
  <router-view></router-view>
</template>

<script setup>
  import { useRoute } from "vue-router";
  import { onUpdated } from "vue";

  const route = useRoute();

  console.log(route);
  console.log(route.params);
</script>
```

![image-20220308160130992](https://s2.loli.net/2022/03/08/v7SWpIYkjZx9FCV.png)
