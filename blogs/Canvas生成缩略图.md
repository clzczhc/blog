---
title: Canvas生成缩略图
categories: 前端
date: 2022-06-18 12:03:02
tags:
  - JavaScript
  - Canvas
---

# Canvas生成缩略图

## 前言

> 个人博客的图片太大了，想换成缩略图，正好学了点Canvas，发现用Canvas画出来的图片就有点缩略图的感觉，于是就开始搞起来了

## 利用canvas实现绘制图片

先通过`canvas.getContext('2d')`获取2D绘图上下文，然后调用绘图上下文的`drawImage`方法，实现图片的绘制。详情可查看之前写的Canvas的文章。

```html
<canvas id="mycanvas" width="640" height="360"></canvas>
```



```js
 const mycanvas = document.getElementById('mycanvas')

const context = mycanvas.getContext("2d");

// 获取图像
const img = new Image();
img.src = "./big.png";

// 图片加载完成后，绘制图片到canvas上
img.onload = () => {
    context.drawImage(img, 0, 0, 640, 360);
}
```

![image-20220521184441072](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181203588.png)

## 下载图片

上面我们就已经绘制图片了，但是下载图片比较麻烦，得通过右键浏览器自带的另存方法。

所以我们可以通过Canvas的`toDataURL`方法，将绘制的图片转成`base64`编码，然后通过`a链接`方式下载。具体的下载方法的实现同样可以查看以前写的文章。(偷懒，不想贴地址，手动狗头)

```js
const mycanvas = document.getElementById('mycanvas')

const context = mycanvas.getContext("2d");

// 获取图像
const img = new Image();
img.src = "./big.png";

// 图片加载完成后，绘制图片到canvas上
img.onload = () => {
    context.drawImage(img, 0, 0, 640, 360);


    const a = document.createElement('a')
    a.href = mycanvas.toDataURL()

    // 获取源图片的名字
    a.download = img.src.split('/')[img.src.split('/').length - 1]

    a.click()
}
```

![image-20220521185207447](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/220bf3f107554ab49e131c83dc1a2054~tplv-k3u1fbpfcp-zoom-1.image)

刷新页面，我们就能发现自动下载了。接下来，我们就来看看，是不是我们想要的效果。

![image-20220521185605337](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181203556.png)

美滋滋



## 去掉html结构中的canvas

实际上，我们可以自己生成Canvas，然后设置宽高，这样就不需要上DOM树，也能实现下载图片。

```js
const w = 640
const h = 360

const mycanvas = document.createElement('canvas')
mycanvas.width = w
mycanvas.height = h

const context = mycanvas.getContext("2d");

// 获取图像
const img = new Image();
img.src = "./big.png";

// 图片加载完成后，绘制图片到canvas上
img.onload = () => {
    context.drawImage(img, 0, 0, w, h);


    const a = document.createElement('a')
    a.href = mycanvas.toDataURL()

    // 获取源图片的名字
    a.download = img.src.split('/')[img.src.split('/').length - 1]

    a.click()
}
```



## 使用`input:file`实现生成多张缩略图

因为安全的关系，网页中的js访问文件夹下的图片会有很多限制。所以选了一个比较简单地方法，利用`input:file`来实现。通过`input:file`的`files`属性可以访问选择的文件列表。



下面为了方便，使用了`form`元素，通过` document.form[0]`方式访问`form`元素，通过`form.elements[0]`访问`input`元素。

```html
<form action="#">
    <input type="file" multiple>
</form>
<button onclick="generatePreview()">生成缩略图</button>
```



```js
const w = 640
const h = 360

function generatePreview() {

    const files = document.forms[0].elements[0].files
    console.log(files)
}
```

![image-20220521191518818](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181203559.png)

这样子就能够获取选中的文件信息了，但是实际上我们只需要文件数量信息和文件名信息即可。



遍历图片列表，下载图片。(图片的`src`属性需要改成对应文件名，生成的图片也需要更改名字。)

```js
a.download = fileName.replace(/(\w+)/, '$1_preview')
```

这里我使用的是`正则表达式+replace`方法去修改名字(简单版本，勿喷)

* `\w`：匹配一个单字字符（字母、数字或者下划线）。等价于 `[A-Za-z0-9_]`
* `+`：匹配前面一个表达式 1 次或者多次
* `$1`：捕获匹配项，前面的正则表达式中，使用括号包住了`\w+`，所以`$1`实际上就是这部分



完整代码：(**`html`文件要放到图片文件夹下才有效**)

```js
<body>
  <form action="#">
    <input type="file" multiple>
  </form>
  <button onclick="generatePreview()">生成缩略图</button>

  <script>
    const w = 640
    const h = 360

    function generatePreview() {

      const files = document.forms[0].elements[0].files

      for (const file of files) {
        const fileName = file.name
        console.log(file)

        const mycanvas = document.createElement('canvas')
        mycanvas.width = w
        mycanvas.height = h

        const context = mycanvas.getContext("2d");

        // 获取图像
        const img = new Image();
        img.src = `./${fileName}`;

        // 图片加载完成后，绘制图片到canvas上
        img.onload = () => {
          context.drawImage(img, 0, 0, w, h);


          const a = document.createElement('a')
          a.href = mycanvas.toDataURL()

          // 获取源图片的名字
          a.download = fileName.replace(/(\w+)/, '$1_preview')

          a.click()
        }
      }
    }
  </script>

</body>
```

![image-20220521193533209](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f603307987840eca895bb6ba1dcd3e9~tplv-k3u1fbpfcp-zoom-1.image)

![image-20220521193331550](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181203510.png)




## 问题
另外，直接本地打开，执行会报错，因为没有服务器环境，本地的html网页，本地的图片

本地的位置是没有域名的，所以会认为是跨域，会报错，可以使用`VSCode`的`Live Server`插件，然后右键，点击`Open with Live Server`就能够以服务器环境打开。

![image-20220521200601567](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181203701.png)
