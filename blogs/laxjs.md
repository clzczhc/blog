---
title: laxjs使用
date: 2024-09-22 23:42:00
categories: 动效
tags:
  - 动效
---

# laxjs使用

## 前言
> 体验一下各种动画库的使用。

## 使用

laxjs跟一些常见的动画库(`animejs`、`gsap`)的使用方式差异较大。`animejs`它们主要是获取dom元素，然后设置终点的样式，来进行动画。

如：
```js
anime({
  targets: '.css-selector-demo .el',
  translateX: 250
});
```

而laxjs则是通过`addDriver`添加驱动，通过添加的驱动来实现动画。
> 驱动的值是任意的，不过更常见的是通过滚动驱动（laxjs官方介绍就是`create smooth & beautiful animations when you scroll`）

简单使用：
```tsx
import { useLayoutEffect } from "react";
import lax from "lax.js";
import "./App.css";

function App() {
  useLayoutEffect(() => {
    lax.init();

    lax.addDriver('scrollY', () => {
      return window.scrollY;
    })

    lax.addElements(".box", {
      scrollY: {
        translateX: [
          [0, 200],
          [0, 400]
        ],
        translateY: [
          [0, 200],
          [0, 400]
        ],
        opacity: [
          [0, 200],
          [1, 0]
        ]
      }
    })
  });

  return <div className="box">
  </div>;
}

export default App;
```

效果：
![](https://www.clzczh.top/CLZ_img/images/202409221900956.gif)

> lax.addElements第一个参数不支持直接传dom元素，所以在react中没法使用`useRef`来操作dom元素。

lax使用主要分为三个步骤：
1. 初始化。`lax.init`
2. 添加驱动，第一个参数是驱动名称，第二个参数是一个返回数字的函数，该函数每帧都会被调用，值用于动画。`lax.addDriver`
3. 添加元素，第一个参数是选择器，第二个参数是一个对象，key为驱动名称，value是样式映射的一个对象。

比如上面的:
```js
opacity: [
  [0, 200],
  [1, 0]
]
```
就意味着，当`window.scrollY`为0时，opacity的值为1，`window.scrollY`为200时，`opacity`的值为0。opacity跟`window.scroolY`的关系是线性关系，即当`window.scrollY`为100时，`opacity`的值会是`0.5`。

当值映射有多个值时，线性关系也会分段。（下面的0-200为一段，200-800为另一段）
官方例子：
```js
[0, 200, 800]  // Driver value map
[0, 10,  20]   // Animation value map

// Result

| In  | Out |
| --- | --- |
| 0   | 0   |
| 100 | 5   |
| 200 | 10  |
| 500 | 15  |
| 800 | 20  |
```

还可以使用lax内置的一些常用值：
![](https://www.clzczh.top/CLZ_img/images/202409221957479.png)

## lax官网动效分析

看完laxjs的官方文档后，对lax的使用也有了一些了解，再加上lax本身就有一个比较简单的官网动效，而且代码并没有经过压缩混淆，所以可以分析一下官网动效，来稍微加深一下对lax的理解（有一些地方可以做了些许调整）

共同代码（初始化、驱动程序）：
```js
lax.init();

lax.addDriver('scrollY', () => {
  return window.scrollY;
})
```

### logo文字

logo部分的html结构
```html
<div className="title">
  <div className="letter-area">
    <img className="letter" id="c" src={CSvg} />
  </div>
  <div className="letter-area">
  <img className="letter" id="l" src={LSvg} />
  </div>
  <div className="letter-area">
  <img className="letter" id="z" src={ZSvg} />
  </div>
</div>
```

js
```js
lax.addElements('#c', {
  scrollY: {
    translateX: [[0, "screenHeight"], [0, 400]],
    opacity: [[0, "screenHeight/2"], [1, 0]]
  }
})
lax.addElements('#l', {
  scrollY: {
    translateX: [[0, "screenHeight"], [0, -400]],
    opacity: [[0, "screenHeight/2"], [1, 0]]
  }
})
lax.addElements('#z', {
  scrollY: {
    translateY: [[0, 100], [ 0, 100]],
    scale: [[100, "screenHeight"], [1, 10]],
    opacity: [[0, "screenHeight/2", "screenHeight"], [1, 1, 0]],
  }
})
```

> 图片换成在iconfont上切的图片了

C和L的动效比较浅显易懂，开始滚动的时候，透明度逐渐变小，到屏幕一半的时候完全隐藏，与此同时，C向右偏移，L向左偏移。（文字都是`fixed`定位，所以不会出现往上走的情况）

Z的动效就有一点点特殊。滚动开始到100像素的时候，Z向下偏移，然后开始逐渐放大。这里的透明度还做了一个分段处理：滚动到一半这个周期，透明度不发生变化，一半之后到首屏消失，透明度逐渐减小为0

### scroll down文字
整体跟logo文字差不多，不过，处理的样式是`letter-spacing`，跟`translate`系列不一样，需要额外参数`cssUnit: "px"`来设置属性

```js
lax.addElements(".scrolldown", {
  scrollY: {
    "letter-spacing": [
      [0, "screenHeight"],
      [0, 150],
      {
        cssUnit: "px"
      }
    ],
    opacity: [["screenHeight*0.25", "screenHeight*0.75"], [1, 0]],
  }
})
```

### 漂移文字
```js
lax.addElements(".haha", {
  scrollY: {
    translateX: [["elInY", "elOutY"], [0, "screenWidth-200"]]
  }
});

lax.addElements(".hoho", {
  scrollY: {
    translateX: [["elInY", "elOutY"], [0, "-screenWidth-200"]]
  }
});

lax.addElements(".pass", {
  scrollY: {
    translateX: [["elInY", "elOutY"], [-400, "screenWidth+100"]],
    skewX: [["elInY", "elOutY"], [40, -40]],
  }
});
```

> 整体还是跟logo文字一样（万变不离其宗🐕），用到了`lax`内置的一些特殊值。当`haha`这个元素出现的时候（`window.scrollY === elInY`），开始偏移，当它消失的时候（`window.scrollY === elOutY`）时，偏移达到`screenWidth-200`。`pass`元素还额外处理了一下`skewX`属性，会有一个出现到消失，倾斜从向左逐渐变成向右。

### 旋转纸片
```js
lax.addElements(".bubble", {
  scrollY: {
    translateY: [
      ["screenHeight/4", "screenHeight * 3"],
      ["Math.random()*screenHeight", "Math.random()*screenHeight*3"],
    ],
    opacity: [
      ["screenHeight/4", "screenHeight/2"],
      [0, 1],
    ],
    translateX: [[0], ["index*(screenWidth/25)-50"]],
    transform: [
      [0, 4000],
      [0, "(Math.random() + 0.8) * 1000"],
      {
        cssFn: function (val: number) {
          return `rotateX(${val % 360}deg)`;
        },
      },
    ],
    rotate: [
      [0, 4000],
      [0, "(Math.random() - 0.5) * 1000"],
    ],
  },
});
```

这里的动效因为是针对多个元素的，所以上面会有比较多的`Math.random()`

属性较多，按属性分析：
1. 
```js
translateY: [
  ["screenHeight/4", "screenHeight * 3"],
  ["Math.random()*screenHeight", "Math.random()*screenHeight*3"],
],
```
> 设置初始值为`Math.random()*screenHeight`，就避免了所有元素一开始都在同一起跑线，而第二个`random`则是为了在滚动过程中会有变化，才会有"超车"这种情况发生。

2. 
```js
translateX: [[0], ["index*(screenWidth/25)-50"]],
```

> 水平线上的位置只有一个值，所以不会有变化。可以通过`index`来设置初始值，比如当`screenWidth`为1000px时，第一个元素的`translateX`为`-50`，第二个元素的`translateX`为`1*(1000/25)-50 = -10`，依次类推。(实际上就是`screenWidth/25`的一个等差数列)

 3. 
 ```js
 transform: [
  [0, 4000],
  [0, "(Math.random() + 0.8) * 1000"],
  {
    cssFn: function (val: number) {
      return `rotateX(${val % 360}deg)`;
    },
  },
],
rotate: [
  [0, 4000],
  [0, "(Math.random() - 0.5) * 1000"],
],
```

这两个都是旋转，区别是`rotate`是二维的，而`rotateX`是三维的，`rotateX`可以有种纸片在正对我们旋转的感觉。上面因为`transform`只需要额外设置`rotateX`，所以用到了`cssFn`处理样式，还加了一下取余做一些优化。

### 最终效果
![](https://www.clzczh.top/CLZ_img/images/202409222340696.gif)

### 完整代码

tsx
```tsx
import { useLayoutEffect } from "react";
import "./App.css";
import CSvg from "./assets/c.svg";
import LSvg from "./assets/l.svg";
import ZSvg from "./assets/z.svg";
import lax from "lax.js";

function App() {
  useLayoutEffect(() => {
    lax.init();

    lax.addDriver("scrollY", () => {
      return window.scrollY;
    });
    lax.addElements("#c", {
      scrollY: {
        translateX: [
          [0, "screenHeight"],
          [0, 400],
        ],
        opacity: [
          [0, "screenHeight/2"],
          [1, 0],
        ],
      },
    });
    lax.addElements("#l", {
      scrollY: {
        translateX: [
          [0, "screenHeight"],
          [0, -400],
        ],
        opacity: [
          [0, "screenHeight/2"],
          [1, 0],
        ],
      },
    });
    lax.addElements("#z", {
      scrollY: {
        translateY: [
          [0, 100],
          [0, 100],
        ],
        scale: [
          [100, "screenHeight"],
          [1, 10],
        ],
        opacity: [
          [0, "screenHeight/2", "screenHeight"],
          [1, 1, 0],
        ],
      },
    });

    lax.addElements(".scrolldown", {
      scrollY: {
        "letter-spacing": [
          [0, "screenHeight"],
          [0, 150],
          {
            cssUnit: "px",
          },
        ],
        opacity: [
          ["screenHeight*0.25", "screenHeight*0.75"],
          [1, 0],
        ],
      },
    });

    lax.addElements(".bubble", {
      scrollY: {
        translateY: [
          ["screenHeight/4", "screenHeight * 3"],
          ["screenHeight", "Math.random()*screenHeight*3"],
        ],
        opacity: [
          ["screenHeight/4", "screenHeight/2"],
          [0, 1],
        ],
        translateX: [[0], ["index*(screenWidth/25)-50"]],
        transform: [
          [0, 4000],
          [0, "(Math.random() + 1) * 1000"],
          {
            cssFn: function (val: number) {
              return `rotateX(${val % 360}deg)`;
            },
          },
        ],
        rotate: [
          [0, 4000],
          [0, "(Math.random() - 0.5) * 1000"],
        ],
      },
    });

    lax.addElements(".haha", {
      scrollY: {
        translateX: [
          ["elInY", "elOutY"],
          [0, "screenWidth - 00"],
        ],
      },
    });
    lax.addElements(".hoho", {
      scrollY: {
        translateX: [
          ["elInY", "elOutY"],
          [0, "-screenWidth-200"],
        ],
      },
    });
    lax.addElements(".pass", {
      scrollY: {
        translateX: [
          ["elInY", "elOutY"],
          [-400, "screenWidth+100"],
        ],
        skewX: [
          ["elInY", "elOutY"],
          [40, -40],
        ],
      },
    });
  });

  return (
    <div className="container">
      <div className="title">
        <div className="letter-area">
          <img className="letter" id="c" src={CSvg} />
        </div>
        <div className="letter-area">
          <img className="letter" id="l" src={LSvg} />
        </div>
        <div className="letter-area">
          <img className="letter" id="z" src={ZSvg} />
        </div>
      </div>

      <h2 className="scrolldown">scroll down</h2>

      <div className="bubbles">
        <div className="bubble blue"></div>
        <div className="bubble red"></div>
        <div className="bubble yellow"></div>
        <div className="bubble red"></div>
        <div className="bubble blue"></div>
        <div className="bubble yellow"></div>
        <div className="bubble blue"></div>
        <div className="bubble red"></div>
        <div className="bubble yellow"></div>
        <div className="bubble red"></div>
        <div className="bubble blue"></div>
        <div className="bubble blue"></div>
        <div className="bubble red"></div>
        <div className="bubble yellow"></div>
        <div className="bubble red"></div>
        <div className="bubble blue"></div>
        <div className="bubble yellow"></div>
        <div className="bubble blue"></div>
        <div className="bubble red"></div>
        <div className="bubble yellow"></div>
        <div className="bubble red"></div>
        <div className="bubble blue"></div>
        <div className="bubble yellow"></div>
        <div className="bubble blue"></div>
        <div className="bubble red"></div>
      </div>

      <div className="haha">哈哈</div>

      <div className="hoho">吼吼</div>

      <div className="pass">路过</div>
    </div>
  );
}

export default App;
```

css
```css
body {
  height: 400vh;
  color: #fff;
  background-color: #242224;
  overflow-x: hidden;
}

.title{
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 100px;
}

.letter-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.letter {
  position: fixed;
  width: 200px;
}

#c {
  top: 100px;
}

#l {
  top: 300px;
}

#z {
  top: 500px
}

.scrolldown {
  position: fixed;
  bottom: 100px;
  display: flex;
  width: 100%;
  justify-content: center;
  font-size: 40px;
  text-align: center;
}

.bubbles {
  top: -100vh;
  position: fixed;
  transform: translate3d(0, 0, 0);
  z-index: 5;
}

.bubble {
  width: 140px;
  height: 140px;
  opacity: 1;
  position: absolute;
}

.bubble.red {
  background: #a94fe4;
}

.bubble.blue {
  background: #68e4f1;
}

.bubble.yellow {
  background: #ffe773;
}

.haha {
  position: absolute;
  top: 150vh;
  left:0;
  font-size: 150px;
}

.hoho {
  position: absolute;
  top: 200vh;
  right:0;
  font-size: 150px;
}

.pass {
  position: absolute;
  top: 270vh;
  left:0;
  font-size: 150px;
}
```