---
title: 通过JS实现剪贴板操作
date: 2021-12-16 18:11:37
keywords: 前端 JS JavaScript
categories: "前端"
tags:
  - JavaScript
---

# 通过 JS 实现剪贴板操作

在网上找到很多种方法，ZeroClipboard.js、clipboard.js 插件等，但是都没有办法解决本人项目中的问题，最后发现可以通过 navigator 对象得到 clipboard，进行剪切板操作

先来一下 clipboard.js 版本的热热身。

## 1. clipboard.js

### 1.1 通过 text 的 function()来复制内容

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <button id="btn">Copy</button>

    <script src="https://cdn.bootcdn.net/ajax/libs/clipboard.js/2.0.8/clipboard.min.js"></script>
    <script>
      const clipboard = new ClipboardJS("#btn", {
        // 第一个参数是可以是类似jQuery的选择器，也可以是DOM对象
        text: function () {
          return "Hello World"; // 返回要放到剪切板的内容
        },
      });
    </script>
  </body>
</html>
```

![image-20211208003450259](https://s2.loli.net/2022/01/15/nC6YdGofaLlhIwg.png)

### 1.2 通过 target 的 function()来复制内容

类似上面的，不过创建 clipboard 对象时第二个参数的对象的属性从 text 变为了 target。简单理解的话，就是 text 就相当于是直接把要复制的内容给出来，而 target 则是把要去复制的地点给出来，在下面就相当于根据 DOM 对象，顺藤摸瓜去复制。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div class="foo">Hello</div>
    <button class="btn">Copy</button>
    <div class="foo">World</div>

    <script src="https://cdn.bootcdn.net/ajax/libs/clipboard.js/2.0.8/clipboard.min.js"></script>
    <script>
      const btn = document.getElementsByClassName("btn")[0];
      const clipboard = new ClipboardJS(btn, {
        target: function () {
          return document.getElementsByClassName("foo")[1];
        },
      });
    </script>
  </body>
</html>
```

### 1.3 通过给复制按钮添加自定义属性来复制内容

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <span id="foo">1 + 1 = 2</span>
    <button
      class="btn"
      data-clipboard-action="copy"
      data-clipboard-target="#foo"
    >
      Copy
    </button>

    <script src="https://cdn.bootcdn.net/ajax/libs/clipboard.js/2.0.8/clipboard.min.js"></script>
    <script>
      const btn = document.getElementsByClassName("btn")[0];
      const clipboard = new ClipboardJS(btn);
    </script>
  </body>
</html>
```

参数说明：

- data-clipboard-action：剪切板行为，如复制 copy，剪切 cut
- data-clipboard-target：剪切板行为的目标

不过，这个样子的复制相当于自动帮你选择，并且帮你按 CTRL+C, 复制之后，复制的内容会变蓝

## 2. 通过 document.execCommand()方法

只能说是换汤不换药。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Document</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    <input type="text" id="copy-input" value="Hello World!" />
    <button id="copy-btn">复制</button>

    <script>
      const copyInput = document.getElementById("copy-input");
      const copyBtn = document.getElementById("copy-btn");

      copyBtn.addEventListener("click", function () {
        copyInput.select();
        document.execCommand("copy");
      });
    </script>
  </body>
</html>
```

无法执行 document.execCommand('paste')方法。

![image-20211208210003809](https://s2.loli.net/2022/01/15/ieOdwycotlq49aJ.png)

新版本 Chrome 执行`document.execCommand('paste')`会返回 false，因为读取剪切板涉及用户隐私安全，所以一定要在用户允许的情况下才可以进行操作。

## 3. 异步 Clipboard API

它的所有操作都是异步的，返回 Promise 对象，而且，它可以将任意内容(比如图片)放入剪切板。

首先，通过`navigator.clipboard` 返回 Clipboard 对象，所有操作都是通过这个对象进行的。

```js
const clipboardObj = navigator.clipboard;
```

<b style="color:red">Chrome 浏览器规定，只有 HTTPS 协议的页面才能使用这个 API</b>。不过，localhost 允许使用非加密协议。

### 3.1 Clipboard.readText()、Clipboard.writeText()

Clipboard.writeText()用于复制文本数据，Clipboard.readText()用于读取剪切板中的文本数据

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Document</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    <input type="text" placeholder="请输入要复制的内容" id="copy-ipt" />
    <button id="copy-btn">复制</button>
    <br />
    <input type="text" placeholder="会输出复制的内容的内容" id="paste-ipt" />
    <button id="paste-btn">粘贴</button>
    <script>
      const copyIpt = document.getElementById("copy-ipt");
      const copyBtn = document.getElementById("copy-btn");
      const pasteIpt = document.getElementById("paste-ipt");
      const pasteBtn = document.getElementById("paste-btn");

      copyBtn.addEventListener("click", async (e) => {
        await navigator.clipboard.writeText(copyIpt.value);
      });

      pasteBtn.addEventListener("click", async (e) => {
        pasteIpt.value = await navigator.clipboard.readText();
      });
    </script>
  </body>
</html>
```

复制不需要用户给权限， 不过，点击粘贴按钮时，更准确来说是，使用 clipboard.readText()方法时，浏览器会弹出一个对话框，询问是否允许读取剪切板。如果禁止，那么就会报错，可以通过 try...catch 结构处理报错。

![image-20211208212459449](https://s2.loli.net/2022/01/15/hl3U7s8oIjdD9Xf.png)

### 3.2 Clipboard.read()、Clipboard.write()

有点像上面两个的加强版，可以复制和粘贴任意数据，如图片

- **Clipboard.read()**：从剪切板读取数据(如图片)
- **Clipboard.write()**：写入任意数据到剪切板

**Clipboard.write()**：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Document</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>
      img {
        width: 200px;
      }
      button {
        margin-left: 70px;
      }
    </style>
  </head>
  <body>
    <img
      src="https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112082214161.png"
      alt=""
    />
    <br />
    <button id="copyImg-btn">复制图片</button>
    <script>
      const mycopy = async () => {
        const imgURL =
          "https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112082214161.png";
        const data = await fetch(imgURL); //  fetch() 方法，该方法提供了一种简单，合理的方式来跨网络异步获取资源
        console.log(data);
        const blob = await data.blob(); // Blob 对象表示一个不可变、原始数据的类文件对象。
        // 它的数据可以按文本或二进制的格式进行读取，也可以转换成 ReadableStream 来用于数据操作
        console.log(blob);

        await navigator.clipboard.write([
          new ClipboardItem({
            [blob.type]: blob,
          }),
        ]);
        alert("图片已复制");
      };
      document.getElementById("copyImg-btn").addEventListener("click", mycopy);
    </script>
  </body>
</html>
```

复制成功后，去到能粘贴图片的地方，如 Word、WPS 等粘贴，即可看到复制的图片。

**Clipboard.read()**：

```js
const mypaste = async () => {
  const clipboardItems = await navigator.clipboard.read();
  /* 该方法返回一个Promise对象，一旦对象的状态变为resolved，
   * 就可以得到一个数组，每个对象都是ClipboardItem对象的实例
   */
  for (const clipboardItem of clipboardItems) {
    for (const type of clipboardItem.types) {
      const blob = await clipboardItem.getType(type); // 得到blob对象
      document.getElementById("img").src = URL.createObjectURL(blob); // URL.createObjectURL()方法通过blob对象生成一个对应的url
    }
  }
};
document.getElementById("pasteImg-btn").addEventListener("click", mypaste);
```

复制粘贴的完整代码：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Document</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>
      img {
        width: 200px;
      }

      button {
        margin-left: 70px;
      }
    </style>
  </head>

  <body>
    <img
      src="https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112082214161.png"
      alt=""
    />
    <br />
    <button id="copyImg-btn">复制图片</button>
    <br />
    <button id="pasteImg-btn">粘贴图片</button>
    <br />
    <img src="" alt="" id="img" />
    <canvas id="canvas"></canvas>
    <script>
      const mycopy = async () => {
        const imgURL =
          "https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112082214161.png";
        const data = await fetch(imgURL); //  fetch() 方法，该方法提供了一种简单，合理的方式来跨网络异步获取资源
        console.log(data);
        const blob = await data.blob(); // Blob 对象表示一个不可变、原始数据的类文件对象。
        // 它的数据可以按文本或二进制的格式进行读取，也可以转换成 ReadableStream 来用于数据操作
        console.log(blob);

        await navigator.clipboard.write([
          new ClipboardItem({
            [blob.type]: blob,
          }),
        ]);
        alert("图片已复制");
      };
      document.getElementById("copyImg-btn").addEventListener("click", mycopy);

      const mypaste = async () => {
        const clipboardItems = await navigator.clipboard.read();
        /* 该方法返回一个Promise对象，一旦对象的状态变为resolved，
         * 就可以得到一个数组，每个对象都是ClipboardItem对象的实例
         */
        for (const clipboardItem of clipboardItems) {
          for (const type of clipboardItem.types) {
            const blob = await clipboardItem.getType(type); // 得到blob对象
            document.getElementById("img").src = URL.createObjectURL(blob); // URL.createObjectURL()方法通过blob对象生成一个对应的url
          }
        }
      };
      document
        .getElementById("pasteImg-btn")
        .addEventListener("click", mypaste);
    </script>
  </body>
</html>
```

效果：

![image-20211209004028815](https://raw.githubusercontent.com/13535944743/CLZ_img/master/imsges/202112090040916.png)

学习链接：[剪贴板操作 Clipboard API 教程 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2021/01/clipboard-api.html)
