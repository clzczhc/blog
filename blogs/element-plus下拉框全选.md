---
title: element-plus下拉框全选
categories: 前端
date: 2022-05-29 23:15:25
tags:
  - Vue
  - Element
---

# element-plus 下拉框实现全选功能

## 前言

实习确实能学到不少东西，但是学到的东西果然还是需要沉淀下来，不然后面立马又忘记了。

## 下拉框的简单使用

使用方法还是比较简单的

```html
<el-select v-model="user.name" placeholder="请选择">
    <el-option v-for="item in nameList" :key="item" :label="item" :value="item">
    </el-option>
  </el-select>
```

首先需要使用到` el-select`和` el-option`，` el-select`就是下拉框，所以需要使用` v-model`双向绑定数据。而` el-option`就是下拉框的选项。

```js
import { reactive, toRefs } from "vue";

const state = reactive({
  nameList: ["clz", "czh", "ccc"],
  user: {
    name: "",
  },
});

const { nameList, user } = toRefs(state);
```

![image-20220514103938933](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ddb4f206d7847f8a92cf773a586b1d3~tplv-k3u1fbpfcp-zoom-1.image)

## 全选互斥

需求：下拉框选项中有全选以及其他项，需要实现点击全选后不能选择其他项，选中了其他项同样不能选择全选。

### 下拉框多选

先来简单了解下下拉框的多选。

理论上来说，是只需要给` el-select`添加上` multiple`就能实现多选，但是效果不太好。选中的会挤在一起。

![image-20220514105832257](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7436da9652f94cd8b101414a3a135b92~tplv-k3u1fbpfcp-zoom-1.image)



这个时候，我们可以添加` collapse-tags`属性，这样子，这样子就只会显示一个选项，没显示的以数量的形式在后面。

![image-20220514110218782](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e857cea66bb740d186c45f460fa61b70~tplv-k3u1fbpfcp-zoom-1.image)



再添加`   collapse-tags-tooltip`属性，还能实现，悬浮在` +X`上方时，显示全部选中的选项。

![image-20220514110356573](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8da9fe9c52646bc9b1987e4440ee926~tplv-k3u1fbpfcp-zoom-1.image)



代码：

```html
<el-select
  v-model="form.ages"
  placeholder="请选择"
  multiple
  collapse-tags
  collapse-tags-tooltip
>
  <el-option
    v-for="item in ageList"
    :key="item"
    :label="item"
    :value="item"
  ></el-option>
</el-select>
```



```js
import { reactive, toRefs } from "vue";

const state = reactive({
  ageList: ["全部", 19, 20, 21, 22],
  form: {
    ages: [],
  },
});

const { ageList, form } = toRefs(state);
```



### 全选互斥的实现

这个主要就是依靠` disabled`属性来实现，只不过属性值变成一个返回` boolean`值的函数了。

```html
  <el-select
    v-model="form.ages"
    placeholder="请选择"
    multiple
    collapse-tags
    collapse-tags-tooltip
  >
    <el-option
      v-for="item in ageList"
      :key="item"
      :label="item"
      :value="item"
      :disabled="checkAge"
    ></el-option>
  </el-select>
```

```js
const checkAge = () => {
  return true;
};
```

![image-20220514111132828](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad32d35961344186a13590c467b578e1~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，当绑定的` checkAge`返回` true`的时候，全部选项都不能选了。



明白原理后，我们便只需要理清思路就行了。

首先，我们绑定的` checkAge`应该要把选中项(` item`)作为参数传给` checkAge`，这样子才能得到选中的项。

接着，就是思路了。我们禁选的情况就两种：

1. 选择了`全部`，此时禁选`非全部的选项`
2. 选择了`非全部的选项`，此时`禁选全部`

也就是说，只有这两个情况返回` true`，其他时候返回` false`

```js
const checkAge = (item) => {
  if (form.value.ages.includes("全部") && item !== "全部") {
    // 选择了`全部`，此时禁选`非全部的选项`
    return true;
  } else if (!form.value.ages.includes("全部") && item === "全部") {
    // 选择了`非全部的选项`，此时`禁选全部`
    return true;
  }

  return false;
};
```

是不是很简单，但是还没完，上面那样子还会有小问题。

![image-20220514112808070](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/397fedff8cd34121b2b49c5a19fc3036~tplv-k3u1fbpfcp-zoom-1.image)

我们什么都没有选择的时候，`全部选项`不能选。这是因为上面选择非全部选项时的判断，写成了没有选择`全部`的时候，所以一开始确实没有选择`全部`，那么就不能选择了。所以在一开始应该判断有没有已经选中的，如果没有，就返回`` false`

```js
const checkAge = (item) => {
  if (form.value.ages.length === 0) {
    return false;
  }
  if (form.value.ages.includes("全部") && item !== "全部") {
    return true;
  } else if (!form.value.ages.includes("全部") && item === "全部") {
    return true;
  }

  return false;
};
```

![select](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1d40521b8064444b135f82b1d3eecbe~tplv-k3u1fbpfcp-zoom-1.image)


## 多个下拉框互斥

多个下拉框不能同时选择同样的选项。

```html
  <el-select v-model="hobbys.hobby1" placeholder="请选择爱好">
    <el-option
      v-for="item in hobbyList"
      :key="item"
      :label="item"
      :value="item"
      :disabled="checkHobby(item)"
    ></el-option>
  </el-select>
```

有三个上面的下拉框，依次是` hobby1`， ` hobby2`， ` hobby3`

```js
import { reactive, toRefs } from "vue";

const state = reactive({
  hobbyList: ["听歌", "动漫", "前端"],
  hobbys: {
    hobby1: "",
    hobby2: "",
    hobby3: "",
  },
});

const { hobbyList, hobbys } = toRefs(state);
```



老样子，通过给` disabled`属性绑定方法，把选中的值传过去。

多个下拉框互斥的实现就比较简单了，只需要遍历选中的值，是不是等于要选的值，等于的话就禁止选择(`return true`)。如果能遍历完，即该选项没有被其他下拉框选中过，那么就能选择(` return false`)。

```js
const checkHobby = (item) => {
  for (const hobbyKey in hobbys.value) {
    // 如果已经有选中过该选项的下拉框，则禁止再次选择
    if (item === hobbys.value[hobbyKey]) {
      return true;
    }
  }

  return false;
};
```

![select](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8540c876da941e497fbb3f66a36b205~tplv-k3u1fbpfcp-zoom-1.image)

## 一般全选的实现

什么是一般全选？其实只是为了区分上面的全选互斥。就是常见的点击全选复选框，就会选中全部选项。

```html
<el-select
  v-model="form.ages"
  placeholder="请选择"
  multiple
  collapse-tags
  collapse-tags-tooltip
>
  <el-checkbox v-model="checked" />全选
  <el-option
    v-for="item in ageList"
    :key="item"
    :label="item"
    :value="item"
  ></el-option>
</el-select>
```



```js
import { reactive, toRefs } from "vue";

const state = reactive({
  ageList: [19, 20, 21, 22],
  form: {
    ages: [],
  },
  checked: false,
});

const { ageList, form, checked } = toRefs(state);
```

![image-20220514153047297](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81c1f16210d24415ab605bb3aab03575~tplv-k3u1fbpfcp-zoom-1.image)



这个时候，全选和下面的选项是互不关联的，所以我们可以通过添加` change`事件，但复选框状态变化时，去修改下面的选项的选中与否。

```html
<el-checkbox v-model="checked" @change="handleCheckAllChange" />全选
```

```js
const handleCheckAllChange = () => {
  if (checked.value) {
    form.value.ages = ageList.value;
  } else {
    form.value.ages = [];
  }
};
```



到这一步的时候，我们就能够做到点击全选复选框，能同时修改下面选项的选中状态了，但是，还不能实现选中下面全部选项时，同时修改全选复选框为选中状态。



可以通过添加侦听器，侦听选中结果，如果发生变化，就会触发侦听器，并根据选中结果的长度和选项总长度对比。

```js
watch(
  () => form.value.ages,
  (newValue) => {
    checked.value = newValue.length === ageList.value.length;
  }
);
```

![select](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5104a5c8b77046de916cd24255f9e6d2~tplv-k3u1fbpfcp-zoom-1.image)



如果想要加个中间态的话，就需要用到` element-plus`复选框的` indeterminate`属性。

这时候，复选框的状态不再是只依靠` checked`了，而是` indeterminate`和` v-model`同时作用。

* ` indeterminate`为` false`，`v-model`为` true`时，状态为` √`
* ` indeterminate`为` false`，`v-model`为` false`时，状态为空
* ` indeterminate`为` true`时，状态为` -`



所以要实现中间态，只需要当选中的选项的个数比总选项的个数少，且选中的选项的个数不为0时,` indeterminate`的值为` true`即可。

```html
<el-checkbox
  v-model="checked"
  :indeterminate="
    form.ages.length < ageList.length && form.ages.length !== 0
  "
  @change="handleCheckAllChange"
/>全选
```
