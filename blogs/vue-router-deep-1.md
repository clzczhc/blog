---
title: Vue Router深入学习(一)
date: 2022-03-12 11:34:18
keywords: Vue Router
categories: "前端"
tags:
  - Vue Router
  - Vue
---

# Vue Router 深入学习(一)

之前的笔记：[Vue 路由](https://clz.vercel.app/2021/10/15/vue-3/#toc-heading-9)

通过阅读文档，自己写一些 demo 来加深自己的理解。(主要针对 Vue3)

## 1. 动态路由匹配

### 1.1 捕获所有路由(404 路由)

> ```js
> const routes = [
>   // 将匹配所有内容并将其放在 `$route.params.pathMatch` 下
>   { path: "/:pathMatch(.*)*", name: "NotFound", component: NotFound },
>   // 将匹配以 `/user-` 开头的所有内容，并将其放在 `$route.params.afterUser` 下
>   { path: "/user-:afterUser(.*)", component: UserGeneric },
> ];
> ```

**使用**：

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
    path: "/user-:afterUser(.*)",
    // 将匹配以 `/user-` 开头的所有内容，并将其放在 `$route.params.afterUser` 下
    name: "User",
    component: () => import("../components/User.vue"),
  },
  {
    path: "/:pathMatch(.*)*",
    // 将匹配所有内容并将其放在 `$route.params.pathMatch` 下
    name: "NotFound",
    component: () => import("../components/NotFound.vue"),
  },
];

const router = new createRouter({
  history: createWebHashHistory(),
  routes,
});

export default router;
```

app.vue

```html
<template>
  {{ route.params }}
  <router-view></router-view>
</template>

<script setup>
  import { useRoute } from "vue-router";

  const route = useRoute();
</script>
```

![image-20220302183444782](https://s2.loli.net/2022/03/12/NpOPi2VMot3wJfh.png)

## 2 路由的匹配语法

主要是通过正则表达式的语法来实现

### 2.1 在参数中自定义正则

语法：

> ```js
> const routes = [
>   // /:orderId -> 仅匹配数字
>   { path: "/:orderId(\\d+)" },
>   // /:productName -> 匹配其他任何内容
>   { path: "/:productName" },
> ];
> ```

实践：

路由配置：

```js
import { createRouter, createWebHashHistory, useRoute } from "vue-router";

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
    path: "/user/:userid(\\d+)", // 两个\是因为会被转义
    name: "UserId",
    component: () => import("../components/UserId.vue"),
  },
  {
    path: "/user/:username",
    name: "UserName",
    component: () => import("../components/UserName.vue"),
  },
];

const router = new createRouter({
  history: createWebHashHistory(),
  routes,
});

export default router;
```

![vue-router](https://s2.loli.net/2022/03/12/kU9syn7R3pvidCm.gif)

### 2.2 可重复的参数

可以使用` *`(0 个或多个)和` +`(1 个或多个)将参数标记为可重复

语法：

> ```js
> const routes = [
>   // /:chapters ->  匹配 /one, /one/two, /one/two/three, 等
>   { path: "/:chapters+" },
>   // /:chapters -> 匹配 /, /one, /one/two, /one/two/three, 等
>   { path: "/:chapters*" },
> ];
> ```

实践：

<b style="color: red">\*</b>：

```js
import { createRouter, createWebHashHistory, useRoute } from "vue-router";

const routes = [
  {
    path: "/:chapters*",
    name: "Chapters",
    component: () => import("../components/Chapters.vue"),
  },
];

const router = new createRouter({
  history: createWebHashHistory(),
  routes,
});

export default router;
```

![vue-router](https://s2.loli.net/2022/03/12/EBaOnpYLSKrdQjz.gif)

<b style="color: red">+</b>：

![vue-router](https://s2.loli.net/2022/03/12/Yhb82rAMQsOk1vF.gif)

### 2.3 可选参数

使用 `?` 修饰符(0 个或 1 个)将一个参数标记为可选

语法：

> ```js
> const routes = [
>   // 匹配 /users 和 /users/posva
>   { path: "/users/:userId?" },
>   // 匹配 /users 和 /users/42
>   { path: "/users/:userId(\\d+)?" },
> ];
> ```

实践：

```js
const routes = [
  {
    path: "/user/:userid(\\d+)?",
    name: "User",
    component: () => import("../components/User.vue"),
  },
  {
    path: "/:pathMatch(.*)*",
    name: "NotFound",
    component: () => import("../components/NotFound.vue"),
  },
];
```

![image-20220303103934235](https://s2.loli.net/2022/03/12/TVynubejJ6OfsFa.png)

如果没加可选限制，那么访问/user 时也会匹配到 404 去

![image-20220303104039713](https://s2.loli.net/2022/03/12/cIr1pqQnsf4CeRE.png)

## 3. 编程式导航

**`params` 不能与 `path` 一起使用，而应该使用`name`(命名路由)**

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

      // // query编程式导航传参
      // router.push({
      //   path: "/user/123",
      //   query: {
      //     id: 666
      //   }
      // })

      // params编程式导航传参
      router.push({
        name: "user", // 需要使用命名路由
        params: {
          userid: 666,
        },
      });
    },
  };
</script>
```

### 3.1 替换当前位置

不会向` history`添加新纪录，而是替换当前的记录

**声明式**：

```html
<router-link to="/home" replace>home</router-link>
```

**编程式**：

```js
router.replace({
  path: "/home",
});

// 或
// router.push({
//   path: '/home',
//   replace: true
// })
```

## 4. 命名视图

需要同时同级展示多个视图，而不是嵌套展示时，命名视图就能够派上用场了

首先路由配置需要使用` components`配置

```js
const routes = [
  {
    path: "/",
    name: "home",
    components: {
      default: () => import("./views/First.vue"),
      second: () => import("./views/Second.vue"),
      third: () => import("./views/Third.vue"),
    },
  },
];
```

使用` router-view`时，添加上`name`属性即可

```html
<router-view></router-view>
<router-view name="second"></router-view>
<router-view name="third"></router-view>
```

示例：

[命名视图](https://codesandbox.io/s/ming-ming-shi-tu-8275iy)

## 5. 路由组件传参

首先可通过` route`来实现路由传参，不过也可以通过` props`配置来开启` props传参`

```js
import { createRouter, createWebHistory } from "vue-router";

const routes = [
  {
    path: "/user/:id",
    component: () => import("../components/User.vue"),
    props: true,
  },
];

export default new createRouter({
  history: createWebHistory(),
  routes,
});
```

通过` props`获取参数

```html
<template>
  <h2>User</h2>
  <p>{{ id }}</p>
</template>

<script setup>
  const props = defineProps(["id"]);
</script>

<style></style>
```

![image-20220303194719540](https://s2.loli.net/2022/03/12/q7KwpDYVS4EzRkN.png)

[更多](https://router.vuejs.org/zh/guide/essentials/passing-props.html)

参考链接：[Vue Router](https://router.vuejs.org/zh/)
