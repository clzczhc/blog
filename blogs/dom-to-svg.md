---
title: DOM 节点转 Svg
categories:
  - 前端
  - 小技能
date: 2024-04-21 13:11:00
tags:
  - Svg
---

# DOM 节点转 Svg

## 前言

> 使用 Canvas 实践拾色器时候接触到 dom 节点转 Svg 的功能，发现 Svg 的一些特点。[domvas](https://github.com/pbakaus/domvas)，作者 2012 年写的工具，[dom-to-image](https://github.com/tsayen/dom-to-image)这个比较多 star 的仓库也是基于这个实现的。而且看这个作者的推特介绍，妥妥一个大佬。

> 这篇博客更像是读后感，看`domvas`源码后的读后感。里面有挺多有意思的点。

## foreignObject

核心实际上就是 Svg 的 foreignObject 元素。

> SVG 中的 <foreignObject> 元素允许包含来自不同的 XML 命名空间的元素。在浏览器的上下文中，很可能是 XHTML / HTML。

由 mdn 的介绍，可以知道使用`foreignObject`就可以在 Svg 中使用 HTML 了。

[更多](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/foreignObject)

简单实践一下。

html

```html
<style>
  .container {
    width: 100px;
    height: 100px;
    background-color: pink;
  }

  h2 {
    color: red;
  }
</style>
<div class="container">
  <h2>h2</h2>
  <p style="color: blue">
    p
    <strong style="color: purple">strong</strong>
  </p>
</div>
```

js

```js
function domToSvg(originalElem) {
  const el = originalElem.cloneNode(true);
  const domString = new XMLSerializer().serializeToString(el); // 获取DOM序列化后的字符串

  // <svg>的xmlns属性就是一个标记，告诉浏览器是Svg。类似于html的<!DOCTYPE html>
  const dataUrl =
    "data:image/svg+xml," +
    `<svg xmlns='http://www.w3.org/2000/svg' width='${originalElem.offsetWidth}' height='${originalElem.offsetHeight}'>` +
    `<foreignObject width='100%' height='100%'>${domString}</foreignObject>` +
    `</svg>`;

  const img = new Image();
  img.src = dataUrl;
  document.body.appendChild(img);
}

domToSvg(document.querySelector(".container"));
```

> 代码很简单，就是把指定元素转化成字符串，再塞到`foreignObject`里面。`XMLSerializer`这个的用法是看到`domvas`里面用到的，挺有意思，可以很简单的把 DOM 序列化成字符串

![](https://www.clzczh.top/CLZ_img/images/202404202228125.png)

> DOM 节点转 Svg 就这么实现了。上面因为是 base64 编码，把`<svg>`前面的东西去掉就是 Svg 的内容。不过从图片效果就能发现，内联样式可以保留在 Svg 中，所以还需要将样式都变成内联样式

## getComputedStyle

将样式变成内联样式，就可以使用`getComputedStyle`来获取计算后的样式。
[更多](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getComputedStyle)

```js
function inlineStyle(el, originalElem) {
  const computedStyle = getComputedStyle(originalElem);

  for (const item in computedStyle) {
    // 这里的判断改用判断是不是自身属性，getComputedStyle得到的对象，自身属性都是计算样式。
    // domvas的做法是类似黑名单，但是个人感觉用hasOwnProperty就够了。
    //  isNaN(parseInt(prop, 10)) && typeof computedStyle[prop] !== "function" && !/^(cssText|length|parentRule)$/.test(prop)
    if (isNaN(parseInt(item, 10)) && computedStyle.hasOwnProperty(item)) {
      // isNaN是因为computedStyle有数字key，数字key需要忽略。
      el.style[item] = computedStyle[item];
    }
  }
}

function inlineStyles(el, originalElem) {
  // 这里的children实际上并不只是子节点，而是子孙节点。所以不能使用children属性。
  const children = el.querySelectorAll("*");
  const originalChildren = originalElem.querySelectorAll("*");

  inlineStyle(el, originalElem);

  [...children].forEach((child, index) => {
    inlineStyle(child, originalChildren[index]);
  });
}

function domToSvg(originalElem) {
  const el = originalElem.cloneNode(true);
  inlineStyles(el, originalElem);

  // ...
}
```

![](https://www.clzczh.top/CLZ_img/images/202404202323979.png)
会有一些瑕疵，这种 margin 边距溢出盒子这种问题可以通过编写 css 来避免。

## fetch 和 FileReader 获取图片的 base64 码

```js
async function getImgDataUrl(url) {
  return new Promise((resolve, reject) => {
    fetch(url).then(async (res) => {
      const blob = await res.blob();

      const fileReader = new FileReader();
      fileReader.onloadend = () => {
        resolve(fileReader.result);
      };

      fileReader.readAsDataURL(blob);
    });
  });
}
```

然后在遍历 dom 的时候把图片的地址切换成`getImgDataUrl`得到的值即可。

```js
async function inlineStyle(el, originalElem) {
  // ...

  if (el instanceof HTMLImageElement) {
    const imgDataUrl = await getImgDataUrl(el.src);
    el.src = imgDataUrl;
  }

  const background = el.style.getPropertyValue("background");
  const backgroundImage = el.style.getPropertyValue("background-image");

  if (background || backgroundImage) {
    const URL_REGEX = /url\(['"]?([^'"]+?)['"]?\)/g;

    const matchBackground = URL_REGEX.exec(background);
    const matchBackgroundImage = URL_REGEX.exec(backgroundImage);

    // 这里只考虑一张背景图的情况。如果考虑多张，还需要循环match
    if (matchBackground) {
      const backgroundDataUrl = await getImgDataUrl(matchBackground[1]);

      const newBackground = background.replace(
        URL_REGEX,
        `url('${backgroundDataUrl}')`
      );
      el.style.setProperty("background", newBackground);
    }

    if (matchBackgroundImage) {
      const backgroundDataUrl = await getImgDataUrl(matchBackgroundImage[1]);

      console.log(backgroundImage);

      const newBackground = backgroundImage.replace(
        URL_REGEX,
        `url('${backgroundDataUrl}')`
      );

      console.log(newBackground);
      el.style.setProperty("background-image", newBackground);
    }
  }
}
```

> 代码写的比较随意（手动狗头）

## 完整代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>DOM 节点转 Svg</title>

    <link rel="stylesheet" href="./index.css" />
  </head>

  <body>
    <div style="margin-bottom: 20px">
      <div class="container">
        <h2>h2</h2>
        <p style="color: blue">
          p
          <strong style="color: purple">strong</strong>
        </p>

        <div class="bg"></div>

        <img src="./imgs/kurumi.png" alt="" />
      </div>
    </div>

    <script>
      async function getImgDataUrl(url) {
        return new Promise((resolve, reject) => {
          fetch(url).then(async (res) => {
            const blob = await res.blob();

            const fileReader = new FileReader();
            fileReader.onloadend = () => {
              resolve(fileReader.result);
            };

            fileReader.readAsDataURL(blob);
          });
        });
      }

      async function inlineStyle(el, originalElem) {
        const computedStyle = getComputedStyle(originalElem);

        for (const item in computedStyle) {
          // 这里的判断改用判断是不是自身属性，getComputedStyle得到的对象，自身属性都是计算样式。
          // domvas的做法是类似黑名单，但是个人感觉用hasOwnProperty就够了。
          //  isNaN(parseInt(prop, 10)) && typeof computedStyle[prop] !== "function" && !/^(cssText|length|parentRule)$/.test(prop)
          if (isNaN(parseInt(item, 10)) && computedStyle.hasOwnProperty(item)) {
            // isNaN是因为computedStyle有数字key，数字key需要忽略。
            el.style[item] = computedStyle[item];
          }
        }

        if (el instanceof HTMLImageElement) {
          const imgDataUrl = await getImgDataUrl(el.src);
          el.src = imgDataUrl;
        }

        const background = el.style.getPropertyValue("background");
        const backgroundImage = el.style.getPropertyValue("background-image");

        if (background || backgroundImage) {
          const URL_REGEX = /url\(['"]?([^'"]+?)['"]?\)/g;

          const matchBackground = URL_REGEX.exec(background);
          const matchBackgroundImage = URL_REGEX.exec(backgroundImage);

          // 这里只考虑一张背景图的情况。如果考虑多张，还需要循环match
          if (matchBackground) {
            const backgroundDataUrl = await getImgDataUrl(matchBackground[1]);

            const newBackground = background.replace(
              URL_REGEX,
              `url('${backgroundDataUrl}')`
            );
            el.style.setProperty("background", newBackground);
          }

          if (matchBackgroundImage) {
            const backgroundDataUrl = await getImgDataUrl(
              matchBackgroundImage[1]
            );

            console.log(backgroundImage);

            const newBackground = backgroundImage.replace(
              URL_REGEX,
              `url('${backgroundDataUrl}')`
            );

            console.log(newBackground);
            el.style.setProperty("background-image", newBackground);
          }
        }
      }

      async function inlineStyles(el, originalElem) {
        // 这里的children实际上并不只是子节点，而是子孙节点。所以不能使用children属性。
        const children = el.querySelectorAll("*");
        const originalChildren = originalElem.querySelectorAll("*");

        await inlineStyle(el, originalElem);

        const childrenArr = [...children];

        for (let i = 0; i < children.length; i++) {
          await inlineStyle(children[i], originalChildren[i]);
        }
      }

      async function domToSvg(originalElem) {
        const el = originalElem.cloneNode(true);
        await inlineStyles(el, originalElem);

        const domString = new XMLSerializer().serializeToString(el); // 获取DOM序列化后的字符串

        console.log(domString);

        // <svg>的xmlns属性就是一个标记，告诉浏览器是Svg。类似于html的<!DOCTYPE html>
        const dataUrl =
          "data:image/svg+xml," +
          `<svg xmlns='http://www.w3.org/2000/svg' width='${originalElem.offsetWidth}' height='${originalElem.offsetHeight}'>` +
          `<foreignObject width='100%' height='100%'>${domString}</foreignObject>` +
          `</svg>`;

        const img = new Image();
        img.src = dataUrl;
        document.body.appendChild(img);
      }

      domToSvg(document.querySelector(".container"));
    </script>
  </body>
</html>
```

index.css

```css
.container {
  width: 400px;
  height: 400px;
  background-color: pink;
}

h2 {
  color: red;
  margin: 0;
}

.bg {
  width: 100px;
  height: 100px;
  background: url("./imgs/kurumi.png") no-repeat;
  background-size: cover;
  margin-bottom: 20px;
}

.container img {
  width: 100px;
  height: 120px;
  object-fit: cover;
}
```

> 多说一嘴：`canvas`的`toDataURL`这个方法可以用来实现把图片转化为其他格式，默认为 png。只需要把上面 DOM 节点转 Svg 的图片画到画布上，即可使用。[toDataURL](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toDataURL)

## 参考

[domvas](https://github.com/pbakaus/domvas)

[dom-to-image](https://github.com/tsayen/dom-to-image)
