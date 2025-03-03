---
title: 图片优化：js 生成渐进式图片
date: 2025-03-03 22:00:00
categories: 前端
tags:
  - 优化
---

# 图片优化：js 生成渐进式图片

## 定义

渐进式图片：渐进式 jpeg。所以需要注意：如果是透明底的 png 图片，生成渐进式图片的时候，就不是透明底了。

一般的图片都是自上往下加载的，但是还有另一种渐进式图片（先加载模糊图片，然后逐渐清晰）

> 以下例子，使用 slow 4g、3g 效果更明显

[一般的图片](https://www.clzczh.top/demos/images/21.png)
[渐进式图片](https://www.clzczh.top/demos/images/21-sharp.jpg)

> 和笔者的博客文章的 banner 图效果差不多，不过笔者的 banner 图是用缩略图、原图，通过`load`、`transitionend`时间来实现的。
> 代码：

```js
(function () {
  const images = [...document.querySelectorAll(".card-image")].map(
    (ele) => ele.children[0]
  );

  function callback(intersectionObserverEntrys) {
    for (const i of intersectionObserverEntrys) {
      // isIntersecting属性，当目标元素在视口中为true，否则为false
      if (i.isIntersecting) {
        const img = i.target;
        const realImg = img.nextElementSibling;

        const realUrl = realImg.getAttribute("data-src");
        realImg.setAttribute("src", realUrl);

        realImg.addEventListener("load", () => {
          realImg.classList.add("show");
          img.classList.remove("show");

          realImg.addEventListener("transitionend", function animationend() {
            img.classList.add("hidden");
            realImg.removeEventListener("transitionend", animationend);
          });
        });

        // 结束观察
        observer.unobserve(img);
      }
    }
  }

  const observer = new IntersectionObserver(callback);

  for (const i of images) {
    observer.observe(i);
  }
})();
```

详细内容可查看：[渐进式 jpeg(progressive jpeg)图片及其相关](https://www.zhangxinxu.com/wordpress/2013/01/progressive-jpeg-image-and-so-on/)

## 工具

### sharp

```js
const sharp = require("sharp");

sharp("./21.png")
  .jpeg({
    progressive: true,
  })
  .toFile("./21-sharp.jpg", (err) => {
    if (err) {
      console.log(err);
    }
  });
```

### imagemin

```js
import imagemin from "imagemin";
import imageminMozjpeg from "imagemin-mozjpeg";

imagemin(["./21.png"], {
  destination: "./images",
  plugins: [
    imageminMozjpeg({
      progressive: true,
    }),
  ],
});
```

### 其他的一些压缩图片的库

1. imagemin: 通过安装各种插件的形式支持优化对应格式的图片，不过只能进行图片优化，而不能转换图片格式。`imagemin-mozjpeg`提供`progressive`选项，但是 png 图片并不会被处理成渐进式图片
2. gm: 可以支持生成渐进式图片，需要额外安装 ImageMagick，且无维护
3. jimp：纯 js 实现，目前并没有纯 Javascript 的渐进式 jpeg 编码：https://github.com/jpeg-js/jpeg-js/issues/12
4. tinyPng：压缩效果会比较好，但是不支持生成渐进式图片
