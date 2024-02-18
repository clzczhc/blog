---
title: Vue Router深入学习(二)
date: 2022-03-15 16:41:51
keywords: Vue Router
categories: "前端"
tags:
  - Vue Router
  - Vue
---

# Vue Router 深入学习(二)

通过阅读文档，自己写一些 demo 来加深自己的理解。(主要针对 Vue3)
上一篇：[Vue Router 深入学习(一)](https://clz.vercel.app/2022/03/12/vue-router-deep-1/)

## 1. 路由元信息

> 有时，你可能希望将任意信息附加到路由上，如过渡名称、谁可以访问路由等。这些事情可以通过接收属性对象的`meta`属性来实现，并且它可以在路由地址和导航守卫上都被访问到。定义路由的时候你可以这样配置 `meta` 字段

语法：

```js
const routes = [
  {
    path: "/user",
    component: () => import("../components/User.vue"),
    meta: {
      tag: "用户",
      isLogin: true,
    },
  },
];
```

### 1.1 简单使用

```js
import { createRouter, createWebHistory } from "vue-router";

const routes = [
  {
    path: "/user",
    component: () => import("../components/User.vue"),
    meta: {
      tag: "用户",
      isLogin: true,
    },
  },
];

export default new createRouter({
  history: createWebHistory(),
  routes,
});
```

```html
<template>
  <h2>User</h2>
  <p>{{ route.meta }}</p>
</template>

<script setup>
  import { useRoute } from "vue-router";

  const route = useRoute();
  console.log(route.meta);
</script>

<style></style>
```

![image-20220304103855475](https://s2.loli.net/2022/03/15/spOZRvIcYXwPtHm.png)

### 1.2 搭配路由守卫使用

```js
import { createRouter, createWebHistory } from "vue-router";

const routes = [
  {
    path: "/user",
    component: () => import("../components/User.vue"),
    meta: {
      tag: "用户",
      isLogin: true,
    },
    children: [
      {
        path: ":id(\\d+)",
        component: () => import("../components/UserId.vue"),
        meta: {
          type: "id",
          requireAuth: true,
        },
      },
      {
        path: ":name",
        component: () => import("../components/UserName.vue"),
        meta: {
          type: "name",
          requireAuth: false,
        },
      },
    ],
  },
];

export default new createRouter({
  history: createWebHistory(),
  routes,
});
```

路由前置守卫

```js
router.beforeEach((to, from) => {
  if (to.meta.requireAuth) {
    return {
      path: "/user",
      query: {
        redirect: to.path, // 保存要去的位置，获得权限后再去
      },
    };
  }
});
```

![vue-router](https://s2.loli.net/2022/03/15/nImTkXpSrcCadJB.gif)

## 2. 数据获取

> 有时候，进入某个路由后，需要从服务器获取数据。例如，在渲染用户信息时，你需要从服务器获取用户的数据。

### 2.1 导航完成后获取数据

> 当你使用这种方式时，我们会马上导航和渲染组件，然后在组件的 created 钩子中获取数据。这让我们有机会在数据获取期间展示一个 loading 状态，还可以在不同视图间展示不同的 loading 状态。

```html
<template>
  <div class="content">
    <p>id: {{ post.id }}</p>
  </div>
  <span>
    <router-link :to="{ name: 'Post', params: { id: 222 } }">222</router-link>
  </span>
  <span>
    <router-link :to="{ name: 'Post', params: { id: 333 } }">333</router-link>
  </span>
  <span>
    <router-link :to="{ name: 'Post', params: { id: 444 } }">444</router-link>
  </span>
</template>

<script setup>
  import { reactive, watchEffect } from "vue";
  import { useRoute } from "vue-router";

  let post = reactive({
    id: null,
  });
  const route = useRoute();

  const fetchData = () => {
    // 数据获取，不需要生命周期钩子。因为beforeCreate和created没有API，因为setup实际上就相当于这两个生命周期函数
    post.id = route.params.id;
  };

  watchEffect(() => {
    const id = post.id;
    fetchData();
  });
</script>

<style scoped>
  span {
    margin: 0 5px;
  }
</style>
```

![vue-router](https://s2.loli.net/2022/03/15/468dhyarpVXUOWm.gif)

### 2.2 在导航完成前获取数据

> 通过这种方式，我们在导航转入新的路由前获取数据。我们可以在接下来的组件的 `beforeRouteEnter` 守卫中获取数据，当数据获取成功后只调用 `next` 方法

```html
<template>
  <div class="content">
    <p>id: {{ post.id }}</p>
  </div>
  <span>
    <router-link :to="{ name: 'Post', params: { id: 222 } }">222</router-link>
  </span>
  <span>
    <router-link :to="{ name: 'Post', params: { id: 333 } }">333</router-link>
  </span>
  <span>
    <router-link :to="{ name: 'Post', params: { id: 444 } }">444</router-link>
  </span>
</template>

<script>
  export default {
    data() {
      return {
        post: {},
      };
    },
    beforeRouteEnter(to, from, next) {
      // 不要写在setup里
      next((vm) => {
        vm.setData(to.params);
      });
    },
    beforeRouteUpdate(to, from) {
      this.post = to.params;
    },
    methods: {
      setData(post) {
        this.post = post;
      },
    },
  };
</script>

<style scoped>
  span {
    margin: 0 5px;
  }
</style>
```

效果和上图一样。这里有点问题，通过`beforeRouteEnter`无法获取到` setup`里的函数、数据等，所以变成了使用 Vue2 的形式来实现。

## 3. 过渡动效

### 3.1 transition 简单了解

> `<Transition>` 会在一个元素或组件进入和离开 DOM 时应用动画

![image-20220304182013408](https://s2.loli.net/2022/03/15/FBeshZCdVrvYul6.png)

### 3.2 简单使用

```html
<router-view v-slot="{ Component }">
  <transition name="fade">
    <component :is="Component"></component>
  </transition>
</router-view>
```

在 router-view 上使用 v-slot 获取对应的组件，使用 component 动态组件来渲染这个组件，然后用 transition 包裹住这个动态组件

<b style="color: red">对应的路由组件只能有一个根元素，否则过渡将没有效果</b>

```css
.fade-enter-from,
.fade-leave-to {
  transform: translateX(-100px);
  opacity: 0;
}

.fade-enter-active {
  transition: all 1s;
}
```

效果：

![vue-router](https://s2.loli.net/2022/03/15/HNFL6CBem9nM3ao.gif)

#### 代码部分

```html
<template>
  <router-view v-slot="{ Component }">
    <transition name="fade">
      <component :is="Component"></component>
    </transition>
  </router-view>
</template>

<script setup></script>

<style>
  .fade-enter-from,
  .fade-leave-to {
    transform: translateX(-100px);
    opacity: 0;
  }

  .fade-enter-active {
    transition: all 1s;
  }
</style>
```

### 3.3 单个路由的过渡

原理很简单，路由配置时在`meta上`添加上`trasition`属性，再动态地和` name`结合在一起就行

```js
const routes = [
  {
    path: "/",
    component: () => import("../components/Home.vue"),
  },
  {
    path: "/post",
    name: "Post",
    component: () => import("../components/Post.vue"),
    meta: {
      transition: "slide-left",
    },
  },
  {
    path: "/user",
    name: "User",
    component: () => import("../components/User.vue"),
    meta: {
      transition: "slide-right",
    },
  },
];
```

```html
<router-view v-slot="{ Component, route }">
  <transition :name="route.meta.transition || fade">
    <component :is="Component"></component>
  </transition>
</router-view>
```

再添加上对应的 css 样式

```css
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.fade-enter-active,
.slide-left-enter-active,
.slide-right-enter-active {
  transition: all 1s;
}

.slide-left-enter-from,
.slide-left-leave-to {
  transform: translateX(-200px);
}

.slide-right-enter-from,
.slide-right-leave-to {
  transform: translateX(200px);
}
```

![](https://s2.loli.net/2022/03/15/ueE7mi691FaULNR.gif)

### 3.4 基于路由的动态过渡

根据目标路由和当前路由之间的关系，动态地确定使用的过渡

如：添加一个 [全局后置钩子](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#全局后置钩子)，根据路径的深度动态添加信息到 `meta` 字段

```js
router.afterEach((to, from) => {
  const toDepth = to.path.split("/").length;
  const fromDepth = from.path.split("/").length;

  to.meta.transition = toDepth < fromDepth ? "slide-right" : "slide-left";
});
```

![vue-router](https://s2.loli.net/2022/03/04/M2gyJ1QtPL4aqEV.gif)

## 4. 滚动行为

在创建 Router 示例时，提供一个` scrollBehavior`方法

```js
const router = createRouter({
  history: createWebHashHistory(),
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到哪个的位置
  }
})
```

### 4.1 普通用法

```js
const router = new createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    return {
      top: 50, // 始终滚动到距离顶部50px处
    };
  },
});
```

![vue-router1](https://s2.loli.net/2022/03/07/EV8YKSlftByWXr3.gif)

如果浏览器支持[滚动行为](https://developer.mozilla.org/en-US/docs/Web/API/ScrollToOptions/behavior)，可以通过`behavior`变得更流畅

```js
const router = new createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    return {
      top: 50, // 始终滚动到距离顶部50px处
      behavior: "smooth",
    };
  },
});
```

![vue-router1](https://s2.loli.net/2022/03/07/oOms145iPFjnIMG.gif)

### 4.2 通过` el`实现相对元素的偏移

` el`可接受一个 CSS 选择器或一个 DOM 元素

```js
const router = new createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    return {
      el: "h2",
      top: 50,
    };
  },
});
```

![vue-router1](https://s2.loli.net/2022/03/07/b4Dv5ZBGsw7OLNm.gif)

### 4.3 恢复之前的位置

返回 `savedPosition`，在按下 后退/前进 按钮时，就会恢复之前的位置。像浏览器的原生表现那样

```js
const router = new createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      // console.log(savedPosition)
      return savedPosition;
    } else {
      return {
        top: 0,
      };
    }
  },
});
```

![vue-router1](https://s2.loli.net/2022/03/07/bg2IuKXjRE1rPoY.gif)

### 4.4 延迟滚动

通过返回一个 Promise 来实现。

```js
const router = new createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve({
          top: 0,
          behavior: "smooth",
        });
      }, 500);
    });
  },
});
```

![vue-router1](https://s2.loli.net/2022/03/07/VkiJHyfjaZDvIPc.gif)

## 5. 路由懒加载

> 把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，会更高效

静态导入：

```js
import User from "../components/User.vue";

const routes = [
  {
    path: "/user",
    name: "User",
    component: User,
  },
];
```

动态导入：(实际上还省字)

```js
import { createRouter, createWebHistory } from "vue-router";

const routes = [
  {
    path: "/user",
    name: "User",
    component: () => import("../components/User.vue"),
  },
];
```

## 6. 动态路由

### 6.1 添加路由

路由配置：初始只有一个路由

```js
import { createRouter, createWebHistory } from "vue-router";

const routes = [
  {
    path: "/",
    component: () => import("../components/Home.vue"),
  },
];

const router = new createRouter({
  history: createWebHistory(),
  routes,
});

export default router;
```

在导航守卫处添加新路由：实际上要限制那些页面的权限就可以这样添加，只有满足条件才会动态添加路由

```js
router.beforeEach((to, from) => {
  router.addRoute({
    path: "/user",
    name: "User",
    component: () => import("./components/User.vue"),
  });
});
```

### 6.2 删除路由

<b style="color: red">当路由被删除时，所有的别名和子路由都会被同时删掉</b>

#### 6.2.1 通过添加一个名字冲突的路由

如果添加与现有名称相同的路由，会先删除路由，再添加路由。

> ```js
> router.addRoute({ path: "/about", name: "about", component: About });
> // 这将会删除之前已经添加的路由，因为他们具有相同的名字且名字必须是唯一的
> router.addRoute({ path: "/other", name: "about", component: Other });
> ```

#### 6.2.2 通过调用 `router.addRoute()` 返回的回调

情境：路由没有名称，没法覆盖删除掉路由

> ```js
> const removeRoute = router.addRoute(routeRecord);
> removeRoute(); // 删除路由如果存在的话
> ```

#### 6.2.3 通过使用 `router.removeRoute()` 按名称删除路由

> ```js
> router.addRoute({ path: "/about", name: "about", component: About });
> // 删除路由
> router.removeRoute("about");
> ```

### 6.3 添加嵌套路由

```js
router.addRoute({
  name: "admin",
  path: "/admin",
  component: Admin,
  children: [{ path: "settings", component: AdminSettings }],
});
```

也可以将路由的` name`作为第一个参数传递给` router.addRoute()`，这样就可以有效的添加路由

```js
router.addRoute({ name: "admin", path: "/admin", component: Admin });
router.addRoute("admin", { path: "settings", component: AdminSettings });
```

### 6.4 查看现有路由

- [`router.hasRoute()`](https://router.vuejs.org/zh/api/#hasroute)：检查路由是否存在。
- [`router.getRoutes()`](https://router.vuejs.org/zh/api/#getroutes)：获取一个包含所有路由记录的数组。
