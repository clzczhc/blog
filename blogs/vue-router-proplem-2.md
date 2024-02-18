---
title: params编程式导航踩坑
date: 2022-03-08 16:14:53
categories: "前端"
tags:
  - Vue
  - Vue3
  - Vue Router
---

# params 编程式导航踩坑

## 1. `params` 不能与 `path` 一起使用

先来一下路由配置

```js
import { createRouter, createWebHashHistory, useRoute } from "vue-router";

const routes = [
  {
    path: "/user/:userid",
    component: () => import("../components/User.vue"),
  },
];

const router = new createRouter({
  history: createWebHashHistory(),
  routes,
});

export default router;
```

先来一下：query 编程式导航

```html
<template>
  <router-view></router-view>
</template>
<script>
  import { useRoute, useRouter } from "vue-router";

  export default {
    setup() {
      const router = useRouter();

      // query编程式导航传参
      router.push({
        path: "/user/123",
        query: {
          userid: 666,
        },
      });
    },
  };
</script>
```

![image-20220303114728772](https://s2.loli.net/2022/03/08/lJkUbH12vCRnus6.png)

一切正常

然后，换成 params 编程式导航

```html
<template>
  <router-view></router-view>
</template>
<script>
  import { useRoute, useRouter } from "vue-router";

  export default {
    setup() {
      const router = useRouter();

      // params编程式导航传参
      router.push({
        path: "/user",
        params: {
          userid: 123,
        },
      });
    },
  };
</script>
```

最后跳转到 http://localhost:3000/#/user，没有后面的 params 参数，这是因为<b style="color: red">`params` 不能与 `path` 一起使用</b>，一起使用后，后面的 params 参数将不再起作用。

## 2. 需要和命名路由搭配使用

先说一下，一开始，本人还以为`name`就是类似`path`的用法。

```html
<template>
  <router-view></router-view>
</template>
<script>
  import { useRoute, useRouter } from "vue-router";

  export default {
    setup() {
      const router = useRouter();

      // params编程式导航传参
      router.push({
        name: "user",
        params: {
          userid: 123,
        },
      });
    },
  };
</script>
```

然后报错

![image-20220303115736810](https://s2.loli.net/2022/03/08/BUNkO9yWdXK4JEe.png)

通过查阅资料后，才知道，这里的 name 属性就是命名路由名称。

修改路由的配置：变成命名路由

```js
import { createRouter, createWebHashHistory, useRoute } from "vue-router";

const routes = [
  {
    path: "/user/:userid",
    name: "user",
    component: () => import("../components/User.vue"),
  },
];

const router = new createRouter({
  history: createWebHashHistory(),
  routes,
});

export default router;
```

![image-20220303120329957](https://s2.loli.net/2022/03/08/YI82MDXWtHruqwy.png)
