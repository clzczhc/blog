---
title: hexo博客自制背景动画(代码雨)
date: 2021-10-04 15:33:56
categories: "小技能"
tags:
  - hexo
  - blog
---

# hexo 博客自制背景动画(代码雨)

起因：看到[比较厉害的特效](https://pranx.com/matrix-code-rain/)，想学一下加到自己的博客中看看效果。

## 1. 首先，在单独一个 html 文件中实现动画效果

```html
<!DOCTYPE html>

<body lang="en">

    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>代码雨</title>
        <style>
            #canvas {
                position: fixed;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                z-index: -1;
            }
        </style>
    </head>

    <body>

        <canvas id="canvas"></canvas>
        <script>
            window.onload = () => {
                const canvas = document.getElementById("canvas")
                const window_screen = window.screen
                const w = (canvas.width = window_screen.width)
                const h = (canvas.height = window_screen.height)
                const fontSize = 12
                const context = canvas.getContext('2d') // 返回一个用于在画布上绘图的环境

                let col = 0 // 列数
                let drops = [] // 保存每一列当前的位置

                col = Math.floor(w / fontSize) // 得到代码雨的列数

                for (let i = 0; i < col; i++) {
                    drops.push(0)
                }

                var str = 'qwertyuiopasdfghjklzxcvbnm0123456789' //代码雨中的文字

                function rain() {
                    context.fillStyle = 'rgba(0, 0, 0, .1)' // 增加一个遮盖层
                    context.fillRect(0, 0, w, h)
                    context.fillStyle = '#00ff00' // 先为要显示的文字设置好颜色

                    for (let i = 0; i < col; i++) {
                        const index = Math.floor(Math.random() * str.length)
                        const x = i * fontSize
                        const y = drops[i] * fontSize

                        context.fillText(str[index], x, y) // 把文字写到画布上去
                        if (y >= canvas.height && Math.random() >= 0.99) {
                            // 通过random()实现列与列位置的差别
                            drops[i] = 0
                        }
                        drops[i]++
                    }
                }

                setInterval(rain, 30)
            }
        </script>
    </body>

</body>

</html>
```

[结果展示](https://codepen.io/13535944743/pen/VwWOrxx)

现在的效果可能有点不太好看，因为是看了很多代码雨的 js 代码，明白了大概如何实现之后依葫芦画瓢做出来的，待未来优化。

## 2. 实现动画效果后，把它加到 hexo 主题中去

- 在`blog\themes\hexo-theme-matery\source\js`中添加名为 digitalRain.js 的 js 文件，把之前写的 js 代码复制粘贴上去。

  这样子的话，部署(`hexo g -d`)之后可以在`blog\public\js`中发现新增的 js 文件

![](https://pic.imgdb.cn/item/615a9f742ab3f51d91819a05.jpg)

- 这个时候，js 文件到位了，但是 html 文件并没有引入 js 文件，就要使用 hexo 主题的功能了

  找到下图框框中的文件

  ![](https://pic.imgdb.cn/item/615aa0492ab3f51d9182f8b3.jpg)

  打开后，可以看到它引入了很多 js 文件，只是引入的方式有点高端，但是先普通的引入，按下图框框中引入，<b style="color: red">这个路径并不是当前路径，而是相对于`blog\public`的路径</b>。

  ![](https://pic.imgdb.cn/item/615aa0c02ab3f51d9183bb5e.jpg)

  如果路径填错了，会报一个 404 错误。这个时候可以到 github 下的博客仓库看一下，js 文件有没有在上面

  ![](https://pic.imgdb.cn/item/615aa3112ab3f51d9187710e.jpg)

引入成功的话，如下图，双击下图 js 文件时会去到 js 文件的地址

![](https://pic.imgdb.cn/item/615aa3cd2ab3f51d9188a271.jpg)

- 把 js 引入后，就需要设置一下样式了。样式可以通过 js 设置，但是这样子，样式部分和行为部分就混在一起了，之后想改进时会变得很困难。所以样式应该放到 css 文件中

  在` blog\themes\hexo-theme-matery\source\css`中原有的 css 文件中添加 canvas 的样式，可能有权重问题

![](https://pic.imgdb.cn/item/615a9ed82ab3f51d91809926.jpg)

## 3. 部署网站

如果它原本就有`<canvas id="canvas"></canvas>`，那么就可以正常实现背景动画，但是，当它原本没有 canvas 标签时，就需要微操一下 js 代码了。

```js
/****** 微操前 ************/
const canvas = document.getElementById("canvas");
const window_screen = window.screen;

/****** 微操后 ************/
const body = document.body; // 获取body节点
const canvas = document.createElement("canvas"); // 动态生成canvas标签
canvas.id = "canvas";
body.insertBefore(canvas, body.children[0]); // 插入body节点中

const window_screen = window.screen;
```

成功实现，结束。
