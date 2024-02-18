---
title: rAF实现表格内容自动滚动
categories: 前端
date: 2022-06-18 12:53:55
tags:
  - 动画
  - JavaScript
---

# rAF实现表格内容自动滚动

## 前言

实习导师让我实现表格内容自动滚动，实际就是大屏项目可能会用上的。正好之前看到`rAF`的介绍，实际上也没用过，所以就用`rAF`来玩一波。

**目标**：

1. 让表格的内容会自动滚动
2. 鼠标悬浮，动画停止
3. 到底后会自动回到顶部，继续滚动



## element表格自带滚动

首先呢，`element`的表格是自带了滚动效果的，但是需要小小的设置一下。

```html
<template>
  <div class="mytable">
    <el-table :data="tableData" ref="mytable">
      <el-table-column prop="date" label="Date" />
      <el-table-column prop="name" label="Name" />
      <el-table-column prop="address" label="Address" />
    </el-table>
  </div>
</template>

<script setup>
import { reactive } from "vue";

const tableData = reactive([
  {
    date: "2016-05-03",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-02",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-04",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-01",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-02",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-04",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-01",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-02",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-04",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-01",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
]);
</script>

<style lang="less" scoped>
.mytable {
  width: 400px;
  height: 420px;
}
</style>
```

![image-20220603093857433](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181253320.png)

可以发现，这个时候超过表格的部分直接溢出来了。



这个时候可能会想：直接使用`overflow: auto;`不就能实现表格滚动了吗？但是，这个表格滚动效果并不是想要的表格内容滚动。表头也会滚不见。

![table](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db9f3e75855b4efda9fe95f4d6590e23~tplv-k3u1fbpfcp-zoom-1.image)



然后，我们使用`Devtools`工具看一下：

![image-20220603094900993](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181253368.png)



发现上面红框框的元素高是`100%`，但是它们的父元素的高不是100%，所以外层的高并没有传过来。所以我们只需要把` el-scrollbar`的祖先元素的高设置为`100%`即可。

![table](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ea2daccad6a449cab89bd6f536a4eb1~tplv-k3u1fbpfcp-zoom-1.image)



又有新问题出现了：表格有一部分内容被切掉了。

这时候我们仔细想一下就会发现，上面的滚动的只是表格内容，但是实际上我们把高度慢慢地传下来了，所以这个时候的滚动盒子` el-scrollbar`的高度也是整个表格的高度。所以我们需要将它的高度减掉表头的高度。

```css
.el-scrollbar {
  height: calc(100% - 40px);
}
```

![table](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efeceed6d092494fb0435f7e607f7868~tplv-k3u1fbpfcp-zoom-1.image)

这样子，前置任务就初步完成了。



## rAF介绍

rAF：**`requestAnimationFrame`**，实际上就是一个函数，会告诉浏览器：希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。

优点：

* **由系统来决定回调函数的执行时机**。如果显示器刷新的频率是60Hz，那么回调韩式就是每`1/60`s，即16.7ms执行一次。也就是说`rAF`会跟着显示器地刷新频率走，能保证回调函数在每一次的刷新间隔制备执行一次，这样就不会引起丢帧，动画更流畅。
* **窗口没激活时，动画将暂停以提升性能和电池寿命**



## 实现自动滚动

### 获取`el-scrollbar` 容器

需要实现自动滚动，首先得获取`el-scrollbar` 容器。访问路径在`mytable -> scrollBarRef -> wrap$`。

![image-20220603184941374](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d94cc0aafdd54fbbbf39f9a4a48280ce~tplv-k3u1fbpfcp-zoom-1.image)



```js
onMounted(() => {
  const { proxy } = getCurrentInstance();

  const mytable = proxy.$refs.mytable;

  const el = mytable.scrollBarRef.wrap$;

  console.log(el);
});
```



### 编写动画方法，并使用rAF添加回调

首先，需要编写我们的滚动动画方法，很简单，只需要让滚动容器的`scrollTop`一直加就行了。但是，为了让这个动画不只是会执行一次，所以在最后还得使用`rAF`添加回调。当然，**在`onMounted`钩子中也需要添加一次回调**。



```js
function myscroll(el) {
  el.scrollTop += 2;
  window.requestAnimationFrame(myscroll.bind(null, el));	// 该动画方法需要携带参数，所以使用bind方法来携带参数
}
```

![table](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181254740.gif)



### 鼠标悬浮，动画停止

我们上面已经初步让表格内容滚动起来了，接下来实现一下第二个步骤**鼠标悬浮，动画停止**

停止rAF动画，需要获取调用`window.requestAnimationFrame()`方法时返回的 ID，和定时器方法一样，所以我们调用`window.requestAnimationFrame()`方法时，需要使用一个变量接住它，方便停止的时候使用。

然后，绑定鼠标事件：`mouseenter`：动画停止，`mouseleave`：继续动画。

```js
let rAFid;

onMounted(() => {
  const { proxy } = getCurrentInstance();

  const mytable = proxy.$refs.mytable;

  const el = mytable.$refs.scrollBarRef.wrap$;

  console.log(el);

  rAFid = window.requestAnimationFrame(myscroll.bind(null, el));

  el.addEventListener("mouseenter", () => {
    window.cancelAnimationFrame(rAFid);
  });

  el.addEventListener("mouseleave", () => {
    rAFid = window.requestAnimationFrame(myscroll.bind(null, el));
  });
});

function myscroll(el) {
  el.scrollTop += 2;
  rAFid = window.requestAnimationFrame(myscroll.bind(null, el)); // 该动画方法需要携带参数，所以使用bind方法来携带参数
}
```

![table](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181254701.gif)



### 到底后会自动回到顶部，继续滚动

这里我们需要判断是否到底了，所以需要使用`可视高度+距离顶部 >= 整个高度`的方式，即`el.clientHeight + el.scrollTop >= el.scrollHeight`.

我们判断到底后，就使用原生js的`scrollTo`方法，就能让它回到顶部。

```js
function myscroll(el) {
  el.scrollTop += 2;
  if (el.clientHeight + el.scrollTop >= el.scrollHeight) {
    // cancelAnimationFrame(rAFid);
    el.scrollTo({
      top: 0,
    });
  }

  rAFid = requestAnimationFrame(myscroll.bind(null, el));
}
```

![202206041101159.gif](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181254963.gif)


可以看到已经实现了，而且是无缝衔接。



### 完整代码

```html
<template>
  <div class="mytable">
    <el-table :data="tableData" ref="mytable">
      <el-table-column prop="date" label="Date" />
      <el-table-column prop="name" label="Name" />
      <el-table-column prop="address" label="Address" />
    </el-table>
  </div>
</template>

<script setup>
import { reactive, getCurrentInstance, onMounted } from "vue";

const tableData = reactive([
  {
    date: "2016-05-03",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-02",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-04",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-01",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-02",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-04",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-02",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
  {
    date: "2016-05-04",
    name: "Tom",
    address: "No. 189, Grove St, Los Angeles",
  },
]);

let rAFid;

onMounted(() => {
  const { proxy } = getCurrentInstance();

  const mytable = proxy.$refs.mytable;

  const el = mytable.$refs.scrollBarRef.wrap$;

  console.log(el);

  rAFid = window.requestAnimationFrame(myscroll.bind(null, el));

  el.addEventListener("mouseenter", () => {
    window.cancelAnimationFrame(rAFid);
  });

  el.addEventListener("mouseleave", () => {
    rAFid = window.requestAnimationFrame(myscroll.bind(null, el));
  });
});

function myscroll(el) {
  el.scrollTop += 2;
  if (el.clientHeight + el.scrollTop >= el.scrollHeight) {
    el.scrollTo({
      top: 0,
    });
  }

  rAFid = requestAnimationFrame(myscroll.bind(null, el));
}
</script>



<style lang="less" scoped>
.mytable {
  width: 400px;
  height: 420px;
  :deep(.el-table) {
    height: 100%;

    .el-table__inner-wrapper {
      height: 100%;
    }
    .el-table__body-wrapper {
      height: 100%;
    }

    .el-scrollbar {
      height: calc(100% - 40px);
    }
  }
}
</style>
```



## 额外研究

上面已经能够实现表格内容自动滚动了，但是有时候需要突出排在前面的话，可能会到顶部需要慢慢地回滚到顶部，再重新自动滚动。也就是说，`scrollTo`方法的参数添加` behavior: "smooth"`来让它圆滑的回滚到顶部。

但是，我们添加了这个选项后，反而不回滚了。这是因为动画一直都还在，回滚速度又不够动画的。所以我们回滚前还得把动画给停止掉。

```js
function myscroll(el) {
  el.scrollTop += 2;
  if (el.clientHeight + el.scrollTop >= el.scrollHeight) {
    window.cancelAnimationFrame(rAFid);

    el.scrollTo({
      top: 0,
      behavior: "smooth",
    });
  } else {
    rAFid = requestAnimationFrame(myscroll.bind(null, el));
  }
}
```

![table](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181254759.gif)



回滚后，动画停了，所以我们还得添加一个定时器，回滚后停一会，再重新开始动画。

```js
function myscroll(el) {
  el.scrollTop += 2;
  if (el.clientHeight + el.scrollTop >= el.scrollHeight) {
    window.cancelAnimationFrame(rAFid);

    el.scrollTo({
      top: 0,
      behavior: "smooth",
    });

    setTimeout(() => {
      rAFid = requestAnimationFrame(myscroll.bind(null, el));
    }, 1000);
  } else {
    rAFid = requestAnimationFrame(myscroll.bind(null, el));
  }
}
```

![table](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181254134.gif)



最后，还得添加一个钩子，清除`rAF`动画，避免内存泄漏等问题的发生。

```js
onBeforeUnmount(() => {
  window.cancelAnimationFrame(rAFid);
});
```





参考：

* requestAnimationFram优势
* MDN
