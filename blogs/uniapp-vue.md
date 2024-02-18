---
title: uni-app开发和常规Vue开发
date: 2022-02-24 18:19:56
categories: 前端
tags:
  - uni-app
  - Vue
---

# uni-app 开发和常规 Vue 开发

`uni-app` 是一个使用 [Vue.js](https://vuejs.org/) 开发所有前端应用的框架，开发一次，可以发行为 App、小程序、网站，

常规 Web 开发只能发行为网站。

## 标签

- Vue 中的 div 标签对应于 uni-app 是 view

- Vue 中的 span 标签对应于 uni-app 中是 text

## 生命周期

<b style="color: red">uni-app 中有三种类型的生命周期，应用生命周期、页面生命周期、组件生命周期</b>

### 应用生命周期

**应用生命周期只在` App.vue`中有效**

` uni-app`支持以下生命周期函数（部分）

| 函数名   | 说明                                           |
| -------- | ---------------------------------------------- |
| onLaunch | 当` uni-app`初始化完成时触发（全局只触发一次） |
| onShow   | 当` uni-app`启动时，或从后台进入前台时显示     |
| onHide   | 当` uni-app`从前台进入后台                     |
| onError  | 当` uni-app`报错时触发                         |

app.vue

```vue
<script>
export default {
  onLaunch: function () {
    console.log("App Launch");
  },
  onShow: function () {
    console.log("App Show");
  },
  onHide: function () {
    console.log("App Hide");
  },
};
</script>

<style>
/*每个页面公共css */
</style>
```

### 页面生命周期

[页面生命周期](https://uniapp.dcloud.net.cn/collocation/frame/lifecycle?id=page)

### 组件生命周期

[Vue 组件生命周期](https://clz.vercel.app/2021/10/08/vue-2/#toc-heading-9)

`uni-app` 组件支持的生命周期，与 vue 标准组件的生命周期相同

![image-20220224165359762](https://s2.loli.net/2022/02/25/HDmvkn6If12ZPcx.png)

## 路由

### uni-app

`uni-app`页面路由为框架统一管理，开发者需要在[pages.json](https://uniapp.dcloud.net.cn/collocation/pages?id=pages)里配置每个路由页面的路径及页面样式。

新建页面时，会在` pages.json`中自动生成路由

pages.json

```json
"pages": [ //pages数组中第一项表示应用启动页，参考：https://uniapp.dcloud.io/collocation/pages
		{
			"path": "pages/index/index",
			"style": {
				"navigationBarTitleText": "uni-app"
			}
		}
        ,{
            "path" : "pages/login/login",
            "style" :
            {
                "navigationBarTitleText": "",
                "enablePullDownRefresh": false
            }

        }
    ]
```

**`uni-app` 有两种页面路由跳转方式：使用[navigator](https://uniapp.dcloud.net.cn/component/navigator)组件跳转、调用[API](https://uniapp.dcloud.net.cn/api/router)跳转。**

- 使用[navigator](https://uniapp.dcloud.net.cn/component/navigator)组件跳转

  ```html
  <navigator url="/" class="link">首页</navigator>
  ```

- 调用[API](https://uniapp.dcloud.net.cn/api/router)跳转

  ```js
  uni.navigateTo({
    url: "/pages/login/login",
  });
  ```

pages \ index.vue

```vue
<template>
  <view class="index">
    <text class="text">首页</text>
    <button class="btn" @click="login">登录</button>
  </view>
</template>

<script>
export default {
  methods: {
    login() {
      uni.navigateTo({
        url: "/pages/login/login",
      });
    },
  },
};
</script>

<style lang="less" scoped>
.index {
  text-align: center;

  .btn {
    width: 100px;
    height: 50px;
  }
}
</style>
```

pages \ login \ login.vue

```vue
<template>
  <view class="login">
    login
    <navigator url="/" class="link">首页</navigator>
  </view>
</template>

<script>
export default {
  data() {
    return {};
  },
  methods: {},
};
</script>

<style>
.login {
  text-align: center;
}
.link {
  color: blue;
}
</style>
```

![uniapp](https://s2.loli.net/2022/02/25/amgiZdwJkLOhrS4.gif)

插一嘴：HBuilderX 感觉挺人性化的，缺少的插件保存时会自动帮你安装

![image-20220224161236235](https://s2.loli.net/2022/02/25/zWogary5BFfTLnq.png)

### Vue 路由

[Vue 路由](https://clz.vercel.app/2021/10/15/vue-3/#toc-heading-9)

相对 uni-app 的路由设置来说，Vue 的路由配置稍稍麻烦一点，先安装 vue-router，再建 router 文件夹，设置路由规则，并导出路由给 Vue 使用。详情点击上面的链接

Vue3 相对于 Vue2,也有所变化：[Vue3 路由](https://clz.vercel.app/2022/02/21/vue3-1/#toc-heading-15)

## 路由守卫

通过安装` uni-simple-router`插件，实现路由守卫功能

**安装**：` npm install uni-simple-router`

**文档**：[uni-simple-router](https://hhyang.cn/)

使用：[快速上手](https://hhyang.cn/v2/start/quickstart.html)

## 页面之间传值

### 通过查询参数

**首页**

```vue
<template>
  <view class="index">
    首页
    <navigator :url="'/pages/login/login?msg=' + msg" class="login"
      >登录</navigator
    >
  </view>
</template>

<script>
export default {
  data() {
    return {
      msg: "Hello, login Pages",
    };
  },
};
</script>

<style lang="less" scoped>
.index {
  width: 500px;
  text-align: center;
  margin: 20px auto;
}
.login {
  color: blue;
}
</style>
```

**登录页**

```vue
<template>
  <view class="login">
    <h2>登录</h2>
    首页发送的信息: {{ msg }}
  </view>
</template>

<script>
export default {
  data() {
    return {
      msg: "",
    };
  },
  methods: {},
  onLoad(options) {
    // 其参数为上个页面传递的数据，参数类型为 Object（用于页面传参）
    this.msg = options.msg;
  },
};
</script>

<style scoped>
.login {
  text-align: center;
  width: 300px;
  margin: 20px auto;
}
</style>
```

![image-20220225104643142](https://s2.loli.net/2022/02/25/LMoHn8hr4meK3BC.png)

<b style="color: red">以下部分，uni-app 和常规 Vue 开发一样，属于是复习内容</b>

## 父子组件传值

和 Vue 一样，顺便复习一下

[Vue 组件间的数据共享](https://clz.vercel.app/2021/10/08/vue-2/#toc-heading-21)

### 父传子(props)

子组件

```vue
<template>
  <view class="son"> 子组件: 父组件发送的信息:{{ msg }} </view>
</template>

<script>
export default {
  name: "Son",
  props: {
    msg: {
      type: String,
      default: "",
    },
  },
};
</script>

<style scoped>
.son {
  width: 300px;
  height: 150px;
  line-height: 150px;
  background-color: pink;
  margin: 20px auto;
}
</style>
```

父组件

```vue
<template>
  <view class="index">
    <text class="text">父组件: {{ msg }}</text>
    <Son :msg="msg"></Son>
  </view>
</template>

<script>
import Son from "../../components/Son.vue";
export default {
  components: {
    Son,
  },
  data() {
    return {
      msg: "Hello, Son",
    };
  },
};
</script>

<style lang="less" scoped>
.index {
  width: 500px;
  height: 250px;
  text-align: center;
  background-color: skyblue;
  margin: 0 auto;
}
</style>
```

![image-20220224181528797](https://s2.loli.net/2022/02/25/iwh4M7AxaVQBTPg.png)

### 子传父(emit)

子组件

```vue
<template>
  <view class="son">
    子组件: {{ msg }}
    <button @click="sendMsg">发送信息给父组件</button>
  </view>
</template>

<script>
export default {
  name: "Son",
  data() {
    return {
      msg: "Hello Father",
    };
  },
  methods: {
    sendMsg() {
      this.$emit("getMsg", this.msg);
    },
  },
};
</script>

<style scoped>
.son {
  width: 300px;
  height: 150px;
  line-height: 150px;
  background-color: pink;
  margin: 20px auto;
}
</style>
```

父组件

```vue
<template>
  <view class="index">
    <text class="text">父组件: 子组件发送的信息: {{ msg }}</text>
    <Son @getMsg="getMsg"></Son>
  </view>
</template>

<script>
import Son from "../../components/Son.vue";
export default {
  components: {
    Son,
  },
  data() {
    return {
      msg: "",
    };
  },
  methods: {
    getMsg(msg) {
      this.msg = msg;
    },
  },
};
</script>

<style lang="less" scoped>
.index {
  width: 500px;
  height: 250px;
  text-align: center;
  background-color: skyblue;
  margin: 0 auto;
}
</style>
```

![uniapp](https://s2.loli.net/2022/02/25/fwIDAingrLjJKcu.gif)
