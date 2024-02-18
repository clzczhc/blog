---
title: el-upload上传文件
categories: 前端
date: 2022-05-29 23:14:25
tags:
  - Vue
  - Element
---

# el-upload上传文件

## 前言

公司和学校项目都用到了上传文件的功能，记录一下。

## 准备

### express实现的上传接口

```js
const express = require('express');

// 文件上传模块
const multiparty = require('multiparty')

// 提供跨域资源请求
const cors = require('cors')

// 文件操作
const fs = require('fs')

const app = express();

app.use(cors());

app.get('/policy', (req, res) => {
  res.json({
    code: 200,
    msg: '成功',
    url: '/upload',
    filename: 'clz'
  })
});

app.post('/upload', (req, res) => {
  let form = new multiparty.Form()

  // 配置文件存储路径
  form.uploadDir = './upload'

  form.parse(req, (err, fields, files) => {
    // 解析formData

    for (const file of files.file) {
      const newPath = form.uploadDir + '/' + file.originalFilename

      // renameSync(oldPath, newPath)：同步重命名文件，oldPath使用默认上传路径，newPath则是想要保存到的路径
      fs.renameSync(file.path, newPath);
    }

    res.json({
      code: 200,
      msg: '上传成功'
    })
  })
});

app.listen(8088, () => {
  console.log('http://localhost:8088/');
});
```

如果需要使用，自行查阅`express`用法，也可以到[本人博客](https://www.clzczh.top/)中查看简单教程。

## 开始

### 简单使用版本

```js
<template>
  <el-upload
    action="http://localhost:8088/upload"
    :show-file-list="true"
    :on-success="handleSuccess"
    :on-error="handleError"
  >
    <el-button type="primary">上传图片</el-button>
  </el-upload>
</template>

<script setup>
import { ElMessage } from "element-plus";

const handleSuccess = (res, file, files) => {
  console.log(res);
  console.log(file, files);

  new ElMessage({
    type: "success",
    message: "上传成功",
  });
};

const handleError = (error, file, files) => {
  console.log(error);
  console.log(file, files);
  ElMessage.error("上传失败");
};
</script>

<style lang="less" scoped>
</style>

```

解释下上面的属性：

-   `action`：请求URL

-   `show-file-list`：是否展文件列表(如果没上传成功，则会闪现一下，再消失)

-   `on-success`：文件上传成功钩子

    -   参数：
    -   res：后端返回的成功响应数据(响应状态为成功时)
    -   file：上传的文件
    -   files：成功上传的文件列表

-   `on-success`：文件上传失败钩子

    -   参数：
    -   error：错误对象，内容是后端返回的响应数据(响应状态为失败时，如状态码为500)
    -   file：上传的文件
    -   files：成功上传的文件列表

![file](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e58edc4653c54f5ab9e64e70a4098d6f~tplv-k3u1fbpfcp-zoom-1.image)

接下来，去后端设置的路径去看看有没有成功保存上传的文件。

![image-20220508120156380](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c21055f85a304475b52533a145869b83~tplv-k3u1fbpfcp-zoom-1.image)

### 添加token

这个比较简单，因为`element-plus`也封装好了，只需要使用`headers`属性，去设置请求头即可

```html
<el-upload
  action="http://localhost:8088/upload"
  :headers="{ token: '12345' }"
>
  <el-button type="primary">上传图片</el-button>
</el-upload>
```

![image-20220508120607213](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/913b816fa4c341baa16f7ce43f23a765~tplv-k3u1fbpfcp-zoom-1.image)

### 上传前获取签名再上传

有时候并不是直接上传就可以的，比如一开始并没有上传路径，需要调用获取签名接口来获取上传路径。这个时候就可以使用我们的上传文件之前的钩子`before-upload`。在上传前调用获取签名的接口，用拿到的`url`去修改，上传路径，就能够上传了。

```html
<template>
  <el-upload :action="url" :before-upload="getPolicy">
    <el-button type="primary">上传图片</el-button>
  </el-upload>
</template>

<script setup>
import axios from "axios";
import { reactive, ref } from "vue";

const url = ref("");

const getPolicy = () => {
  return new Promise((resolve, reject) => {
    axios
      .get("http://localhost:8088/policy")
      .then((res) => {
        const { data } = res;

        if (data.code === 200) {
          url.value = `http://localhost:8088${data.url}`;

          resolve();
        }
      })
      .catch((error) => {
        reject(error);
      });
  });
};
</script>

<style lang="less" scoped>
</style>
```

![image-20220508134523663](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40043fd7261d4055af018d53fce8b352~tplv-k3u1fbpfcp-zoom-1.image)

### 手动上传

我们上面的例子都是选中文件后，就会上传，但是有时候我们会有点击按钮才去上传的需求，这个时候就需要结合`auto-upload`和`submit`来实现手动上传了。先设置`auto-upload`为`false`，取消自动上传，这个时候选中图片后就没有上传了，所以我们在按钮的点击事件中，还得使用DOM去调用`submit`方法去手动上传。

```html
<template>
  <el-upload
    ref="upload"
    action="http://localhost:8088/upload"
    :auto-upload="false"
  >
    <el-button type="primary">上传图片</el-button>
  </el-upload>

  <el-button type="primary" @click="confirm">确定</el-button>
</template>

<script setup>
import { getCurrentInstance } from "vue";

const { proxy } = getCurrentInstance();

const confirm = () => {
  proxy.$refs.upload.submit();
};
</script>

<style lang="less" scoped>
</style>
```

![file](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d098f8945fde457e82e1f5114a8377b1~tplv-k3u1fbpfcp-zoom-1.image)

### 上传的时候修改文件名

情境：调用签名接口时也给你返回一个文件名，前端在上传的时候需要把文件名改掉再上传，让服务器保存的是规范的文件名。

首先，先说一下结论：无法通过修改`File`对象的`name`属性，实现重命名

#### 在上传前钩子中修改File对象的name属性

```html
<template>
  <el-upload
    action="http://localhost:8088/upload"
    :before-upload="getPolicy"
  >
    <el-button type="primary">上传图片</el-button>
  </el-upload>
</template>

<script setup>
// 上传前钩子
const getPolicy = (file) => {
  console.log(file);

  file.name = "clz.png";    // 如果在上传前钩子中对文件的name属性进行修改，则会导致上传不了。而且不报错，难以发现问题所在。

  console.log(file.name);
};
</script>

<style lang="less" scoped>
</style>
```

![file](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54b5d42e1cab4d32844a1ae53d839210~tplv-k3u1fbpfcp-zoom-1.image)

毫无波澜。

#### 上传文件时修改

通过`http-request`属性，覆盖默认的上传行为。

```html
<template>
  <el-upload action="http://localhost:8088/upload" :http-request="httpRequest">
    <el-button type="primary">上传图片</el-button>
  </el-upload>
</template>

<script setup>
const httpRequest = ({ file }) => {
  console.log(file);
  file.name = "clz.png";
};
</script>

<style lang="less" scoped>
</style>
```

![file](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03f4aea317444c5d884436f8bf85110b~tplv-k3u1fbpfcp-zoom-1.image)

直接报错

#### 解决方案

既然不能直接修改`File`对象的`name`属性来实现重命名操作，那么应该怎么办呢？

这个时候就需要通过`new File`构造函数去再创建一个文件，创建的同时更改名字。

```html
<template>
  <el-upload action="http://localhost:8088/upload" :http-request="httpRequest">
    <el-button type="primary">上传图片</el-button>
  </el-upload>
</template>

<script setup>
const httpRequest = ({ file }) => {
  console.log(file);

  const cloneFile = new File([file], "clz.png");

  console.log(cloneFile);
};
</script>

<style lang="less" scoped>
</style>
```

**注意：如果是更改一个文件的文件名的话，File的构造函数第一个参数应该是包住`file`的数组**

![image-20220508164215907](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d4332203fae48388a03cc202a556d1e~tplv-k3u1fbpfcp-zoom-1.image)

但是这个时候，又有问题了，我们已经使用`http-request`覆盖默认的上传的行为了，所以我们还得重新实现上传。

上传文件首先需要`formData`对象，然后给`formData`添加上数据，在把`formData`通过接口发出去即可。

```html
<template>
  <el-upload action="#" :http-request="httpRequest">
    <el-button type="primary">上传图片</el-button>
  </el-upload>
</template>

<script setup>
import axios from "axios";
import { ElMessage } from "element-plus";

const httpRequest = ({ file }) => {
  console.log(file);

  const cloneFile = new File([file], "clz.png");

  const formData = new FormData();
  formData.append("file", cloneFile);

  axios.post("http://localhost:8088/upload", formData).then((res) => {
    if (res === 200) {
      new ElMessage({
        type: "success",
        message: "上传成功",
      });
    }
  });
};
</script>

<style lang="less" scoped>
</style>
```

![image-20220508165110118](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1986ac52146c48069eb4309471c1780b~tplv-k3u1fbpfcp-zoom-1.image)

#### 小贴士

上面已经说出解决方法了，但是，重命名一般不会帮你把文件后缀都给改掉。所以这个时候可以通过正则表达式把后缀给取出来。

```js
const houzhui = file.name.replace(/.+./, "");
const newFile = new File([file], filename+houzhui);
```

### 一次请求上传多个文件

`el-upload`默认一个请求上传一个文件。需要上传多个文件首先得添加`multiple`属性。

![file](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5281fd7239345f59bae1b91e310ebff~tplv-k3u1fbpfcp-zoom-1.image)

上面的例子中，我们可以发现，我们上面选中了两个文件，点击确定，上传图片时调用了两次上传接口。

既然`el-upload`默认一个请求上传一个文件，那么我们就不要使用`el-upload`的上传方法就行了。点击确定按钮时，去调用一个上传文件方法。

因为我们点击确定时，需要获取选中的文件，所以需要有`file-list`属性，保存选中的文件。

```html
<template>
  <el-upload
    ref="upload"
    action="#"
    multiple
    :file-list="fileList"
    :auto-upload="false"
  >
    <el-button type="primary">上传图片</el-button>
  </el-upload>

  <el-button type="primary" @click="confirm">确定</el-button>
</template>
```

点击确定按钮，会触发`confirm`事件，实现一个请求上传多个文件的关键就在这，这个时候创建一个`formData`对象，遍历选中的文件列表，通过`append`添加到`formData`上。最后在调用`uploadFile`函数，真正把文件上传上去。

```js
const fileList = reactive([]);

const confirm = () => {
  const formData = new FormData();

  console.log(fileList);

  for (const file of fileList) {
    formData.append("file", file.raw);
  }

  uploadFiles(formData);
};

function uploadFiles(data) {
  axios.post("http://localhost:8088/upload", data).then((res) => {
    if (res === 200) {
      new ElMessage({
        type: "success",
        message: "上传成功",
      });
    }
  });
}
```

![file](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce56ebee118446efbf2629321d3bddc8~tplv-k3u1fbpfcp-zoom-1.image)

## 小技能

### 获取图片的宽高像素

```js
// 创建Image对象
const img = new Image();

// 设置图片的src
img.src = 'https://www.clzczh.top/medias/featureimages/19.png';

// 添加load事件，图片加载完成后执行
img.onload = () => {
  // 获取图片的宽高像素
  console.log(img.width, img.height);
};
```
