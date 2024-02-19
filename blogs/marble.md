---
title: matterjs 实现弹珠台
categories: 前端
date: 2023-12-21 22:48:45
tags:
  - 动画
---

# matterjs 实现弹珠台

## webpack 构建

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      html,
      body {
        padding: 0;
        margin: 0;
        overflow: hidden;
      }

      .container {
        margin: 20px;
      }
    </style>
  </head>

  <body>
    <div class="container"></div>
    <script src="./bundle.js"></script>
  </body>
</html>
```

webpack.config.js

```js
const path = require("path");

module.exports = {
  entry: path.join(__dirname, "src", "index.js"),
  mode: "development",
  output: {
    filename: "bundle.js",
    path: path.join(__dirname),
  },
  devServer: {
    hot: true,
    static: {
      directory: path.join(__dirname),
    },
  },
};
```

安装一些会用到的依赖：

package.json

```json
{
  "name": "marble",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "npx webpack server"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^5.89.0",
    "webpack-cli": "^5.1.4"
  },
  "dependencies": {
    "matter-attractors": "^0.1.6",
    "matter-js": "^0.19.0",
    "pathseg": "^1.2.1",
    "poly-decomp": "^0.3.0",
    "svgpath": "^2.6.0",
    "webpack-dev-server": "^4.15.1"
  }
}
```

## 初始化并添加墙壁等物块

位置啥的都是自己根据别人的效果图，初略调的

```js
import Matter from "matter-js";

let engine;

const WIDTH = 500;
const HEIGHT = 640;

function init() {
  engine = Matter.Engine.create();

  const render = Matter.Render.create({
    element: document.querySelector(".container"),
    engine: engine,
    options: {
      width: WIDTH,
      height: HEIGHT,
      wireframes: false,
    },
  });
  Matter.Render.run(render);

  const runner = Matter.Runner.create();
  Matter.Runner.run(runner, engine);

  const mouse = Matter.Mouse.create(render.canvas);
  const mouseConstraint = Matter.MouseConstraint.create(engine, {
    mouse: mouse,
    constraint: {
      stiffness: 0.2,
      render: {
        visible: false,
      },
    },
  });

  Matter.Composite.add(engine.world, mouseConstraint);
}

function addBoundries() {
  Matter.Composite.add(engine.world, [
    boundary(0, HEIGHT / 2, 40, HEIGHT),
    boundary(WIDTH, HEIGHT / 2, 40, HEIGHT),
    boundary(WIDTH / 2, HEIGHT, HEIGHT, 40),

    wall(150, 100, 18, 40),
    wall(230, 100, 18, 40),
    wall(320, 100, 18, 40),

    circle(100, 180, 20),
    circle(225, 180, 20),
    circle(350, 180, 20),

    circle(160, 260, 20),
    circle(290, 260, 20),

    wall(440, 420, 20, 450),

    wall(120, 380, 20, 110),
    wall(320, 380, 20, 110),

    wall(60, 400, 20, 150),
    wall(88, 485, 20, 88, {
      angle: -0.95,
    }),

    wall(380, 400, 20, 150),
    wall(352, 485, 20, 88, {
      angle: 0.95,
    }),

    // reset(0, 50),
    reset(225, 64),
    reset(465, 32),
  ]);
}

function boundary(x, y, width, height) {
  return Matter.Bodies.rectangle(x, y, width, height, {
    isStatic: true,
    render: {
      fillStyle: "#495057",
    },
  });
}

function circle(x, y, radius) {
  const circle = Matter.Bodies.circle(x, y, radius, {
    isStatic: true,
    render: {
      fillStyle: "#495057",
    },
  });

  return circle;
}

function wall(x, y, width, height, options) {
  return Matter.Bodies.rectangle(x, y, width, height, {
    isStatic: true,
    angle: options && options.angle ? options.angle : 0,
    chamfer: {
      radius: 10,
    },
    render: {
      fillStyle: "#495057",
    },
  });
}

function reset(x, width) {
  return Matter.Bodies.rectangle(x, 620, width, 2, {
    label: "reset",
    isStatic: true,
    render: {
      fillStyle: "#fff",
    },
  });
}

function load() {
  init();
  addBoundries();
}

load();
```

![](https://www.clzczh.top/CLZ_img/images/202312172312294.png)

## 添加三角形物块

```js
const PATHS = {
  // leftArrow: 'M 0 0 L 40 60 L 0 100 L 0 0',
  leftArrow: "M 0 0 L 40 60 L 0 100 L 0 0",
  // 最高的那个点要作为出发点（不知道具体原因）
  rightArrow: "M 40 -60 L 40 40 L 0 0 L 40 -60",
  leftBottom: "M 0 0 L 0 -140 L 180 0 L 0 0",
  rightBottom: "M 0 -140 L 0 0 L -180 0 L 0 -140",
};

function path(x, y, path) {
  const vertices = Matter.Vertices.fromPath(path);

  return Matter.Bodies.fromVertices(x, y, vertices, {
    isStatic: true,
    render: {
      fillStyle: "pink",
      strokeStyle: "pink",
      lineWidth: 1,
    },
  });
}

function addBoundries() {
  Matter.Composite.add(engine.world, [
    // ...
    path(35, 260, PATHS.leftArrow),
    path(416, 280, PATHS.rightArrow),

    path(80, 580, PATHS.leftBottom),
    path(370, 580, PATHS.rightBottom),
  ]);
}
```

![](https://www.clzczh.top/CLZ_img/images/202312172313187.png)

PATHS 里的路径可以在[SvgPathEditor](https://yqnn.github.io/svg-path-editor/)中使用 SVG 语法来绘制。

![](https://www.clzczh.top/CLZ_img/images/202312172312474.png)

需要注意的是：**最高的那个点要作为出发点（不知道具体原因）**

去掉`isStatic: true`后：

`M 0 -140 L 0 0 L -180 0 L 0 -140`：最高点为出发点。

![](https://www.clzczh.top/CLZ_img/images/202312182353699.gif)

`M 0 0 L -180 0 L 0 -140 L 0 0`：最高点不是出发点（和上面的形状是一样的

![](https://www.clzczh.top/CLZ_img/images/202312190004820.gif)

可以看到，这时候墙壁基本就是虚设。（具体原因不清楚，但是有可能是因为 svg 路径的凸凹啥的，之前听过一丢丢

## 添加半圆物块

半圆物块没有采用上面的办法，因为按上面的办法的话，半圆还得得到很多点才行。（SVG 语法的 A 命令之类的不生效，基本只能用 L 语法去得到点，即需要很多点来弄出一个半圆）

采用的方案是通过[SvgPathEditor](https://yqnn.github.io/svg-path-editor/)绘制半圆，并得到对应的 SVG，然后再通过`Matter.Svg.pathToVertices`来将 SVG 路径转成物块。

1. 绘制

   ![](https://www.clzczh.top/CLZ_img/images/202312190019465.png)

   ![](https://www.clzczh.top/CLZ_img/images/202312190031686.png)

2. 加载 SVG，并将 path 元素通过`Matter.Svg.pathToVeritices(path, 36)`将 path 转成顶点 Vertices。

   ```js
   const loadSvg = function (url) {
     return fetch(url)
       .then(function (response) {
         return response.text();
       })
       .then(function (raw) {
         // 把加载的svg转成document对象
         return new window.DOMParser().parseFromString(raw, "image/svg+xml");
       });
   };

   function svg(x, y, svgName) {
     loadSvg(`/svg/${svgName}.svg`).then((root) => {
       const vertices = [...root.querySelectorAll("path")].map((path) => {
         return Matter.Svg.pathToVertices(path, 36);
       });

       Matter.Composite.add(
         engine.world,
         Matter.Bodies.fromVertices(x, y, vertices, {
           isStatic: true,
           render: {
             fillStyle: "#495057",
             lineWidth: 1,
           },
         })
       );
     });
   }
   ```

3. 通过`Matter.Bodies.fromVertices`根据顶点生成物块，并添加到物理世界中。

4. 调用方法`svg`

   ```js
   function addBoundries() {
     svg(236, 80, "dome");
     // ...
   }
   ```

5. 安装`pathseg`和`poly-decomp`。

   `pathseg`：因为`Matter.Svg.pathToVertices`不支持复杂路径，所以需要`pathseg`来 polyfill。

   `poly-decomp`：将 2D 多边形分解为凸块（matter-js 不支持凹顶点）\color{red}{凹凸顶点有机会再了解}

   ```js
   import "pathseg";

   function init() {
     Matter.Common.setDecomp(require("poly-decomp"));
     // ...
   }
   ```

![](https://www.clzczh.top/CLZ_img/images/202312192359405.png)

## 添加操作浆

> 原本看了别人的博客实现方式，觉得一个浆，加两个停止器就行了。但是实现起来才发现效果不行，浆会有很高频的抖动效果。最后有参考（抄）别人的思路。

![](https://www.clzczh.top/CLZ_img/images/202312202302959.png)

上面的蓝色物块就是觉得没必要，后面才发现很有必要的。添加好后，设置`visible`为`false`即可。

1. 添加键盘事件

   ```js
   let isLeftPaddleUp = false;
   let isRightPaddleUp = false;
   function addEvents() {
     window.addEventListener("keypress", (e) => {
       if (e.key.toLowerCase() === "a") {
         isLeftPaddleUp = true;
       } else {
         isLeftPaddleUp = false;
       }

       if (e.key.toLowerCase() === "d") {
         isRightPaddleUp = true;
       } else {
         isRightPaddleUp = false;
       }
     });

     window.addEventListener("keyup", (e) => {
       if (e.key.toLowerCase() === "a") {
         isLeftPaddleUp = false;
       }

       if (e.key.toLowerCase() === "d") {
         isRightPaddleUp = false;
       }
     });
   }
   ```

   function load() {
   init();
   addBoundries();
   addEvents();
   }

````
2.  使用`matter-attractors`，创建stopper物块。

```js
// 1.
import MatterAttractors from 'matter-attractors';


function init() {
  Matter.use(MatterAttractors);
  // ...
}

// 2.
function getPaddleStatus(side) {
  const isPaddleUp = side === 'left' ? isLeftPaddleUp : isRightPaddleUp;
  return isPaddleUp;
}

function stopper(x, y, side, position) {
  const judgeLabel = side === 'left' ? 'paddleLeftComp' : 'paddleRightComp';


  const options = {
    isStatic: true,
    render: {
      fillStyle: 'red'
    },
    plugin: {
      attractors: [
        function (bodyA, bodyB) {
          if (bodyB.label === judgeLabel) {
            return {
              x: (bodyA.position.x - bodyB.position.x) * 0.002 * ((getPaddleStatus(side)) ? -1 : 0.5),  // 0.5是防止松手后，吸引力太大导致变形
              y: (bodyA.position.y - bodyB.position.y) * 0.002 * ((getPaddleStatus(side)) ? -1 : 0.5)
            }
          }
        }
      ]
    }
  };

  const hadForce = position === 'bottom';
  if (!hadForce) {
    // 只有下面的stopper有引（斥）力
    Reflect.deleteProperty(options, 'plugin');
  }

  return Matter.Bodies.circle(x, y, 20, options);
}


// 3.
function addPaddles() {
  const stoperLeftTop = stopper(170, 460, 'left', 'top');
  const stoperLeftBottom = stopper(136, 580, 'left', 'bottom');
  const stoperRightTop = stopper(280, 460, 'right', 'top');
  const stoperRightBottom = stopper(300, 580, 'right', 'bottom');

  Matter.Composite.add(engine.world, [
    stoperLeftTop,
    stoperLeftBottom,
    stoperRightTop,
    stoperRightBottom
  ]);
}
````

![](https://www.clzczh.top/CLZ_img/images/202312212126937.png)

> `matter-attractors`可以对物块添加持续的力。通过键盘事件来控制力是引力还是斥力。直接添加会导致对所有物块都有影响，所以通过`label`来判断是否添加。

3. 创建浆

   ```js
   function addPaddles() {
     const paddleLeft = {};
     paddleLeft.paddle = Matter.Bodies.trapezoid(134, 512, 20, 88, 0.33, {
       label: "paddleLeft",
       angle: 1.57,
       chamfer: {},
       render: {
         fillStyle: "skyblue",
       },
     });

     paddleLeft.brick = Matter.Bodies.rectangle(134, 524, 40, 40, {
       render: {
         fillStyle: "blue",
         // visible: false
       },
     });

     // 将两个物块组装在一起
     paddleLeft.comp = Matter.Body.create({
       label: "paddleLeftComp",
       parts: [paddleLeft.paddle, paddleLeft.brick],
     });

     Matter.Composite.add(engine.world, [paddleLeft.comp]);
   }
   ```

   ![](https://www.clzczh.top/CLZ_img/images/202312212140086.png)

4. 添加约束，固定浆

   ```js
   function addPaddles() {
     // ...
     // bodyB表示给paddleLeft.comp添加约束，pointB则是相对paddleLeft.comp的坐标
     // pointA是这个物理世界的坐标
     const constraintLeft = Matter.Constraint.create({
       bodyB: paddleLeft.comp,
       pointB: {
         x: -32,
         y: -8,
       },
       pointA: {
         x: paddleLeft.comp.position.x,
         y: paddleLeft.comp.position.y,
       },
       stiffness: 0.9,
       length: 0,
       render: {
         strokeStyle: "pink",
       },
     });
     Matter.Composite.add(engine.world, constraintLeft);
   }
   ```

   ![](https://www.clzczh.top/CLZ_img/images/202312212153939.gif)

5. 右边的浆同理

   ```js
   const paddleRight = {};
   paddleRight.paddle = Matter.Bodies.trapezoid(304, 512, 20, 88, 0.33, {
     // isStatic: true,
     label: "paddleRight",
     angle: -1.57,
     chamfer: {},
     render: {
       fillStyle: "skyblue",
     },
   });

   paddleRight.brick = Matter.Bodies.rectangle(304, 524, 40, 40, {
     render: {
       fillStyle: "blue",
       // visible: false
     },
   });

   paddleRight.comp = Matter.Body.create({
     label: "paddleRightComp",
     parts: [paddleRight.paddle, paddleRight.brick],
   });

   const constraintRight = Matter.Constraint.create({
     bodyB: paddleRight.comp,
     pointB: {
       x: 32,
       y: -8,
     },
     pointA: {
       x: paddleRight.comp.position.x,
       y: paddleRight.comp.position.y,
     },
     stiffness: 0.9,
     length: 0,
     render: {
       strokeStyle: "pink",
     },
   });
   Matter.Composite.add(engine.world, constraintRight);

   Matter.Composite.add(engine.world, [paddleLeft.comp, paddleRight.comp]);
   ```

## 添加弹珠，并添加碰撞事件，实现`reset`效果

```js
function addMarble() {
  const marble = Matter.Bodies.circle(0, 0, 14, {
    render: {
      fillStyle: "green",
    },
  });
  Matter.Composite.add(engine.world, marble);

  // 设置坐标
  Matter.Body.setPosition(marble, {
    x: 464,
    y: 500,
  });

  // 设置力
  Matter.Body.setVelocity(marble, {
    x: 0,
    y: -25,
  });
}

function addEvents() {
  // ...
  Matter.Events.on(engine, "collisionStart", (e) => {
    e.pairs.forEach(function (pair) {
      const bodyA = pair.bodyA;
      const bodyB = pair.bodyB;

      if (bodyA.label === "reset") {
        Matter.Composite.remove(engine.world, [bodyB]);
        addMarble();
      }

      if (bodyB.label === "reset") {
        Matter.Composite.remove(engine.world, [bodyA]);
        addMarble();
      }
    });
  });
}
```

![](https://www.clzczh.top/CLZ_img/images/202312212206056.gif)

## 给中间的 5 个圆加弹力

> 现在的弹珠基本会掉到中间，浆根本没有用武之地。

```js
function circle(x, y, radius) {
  const circle = Matter.Bodies.circle(x, y, radius, {
    isStatic: true,
    render: {
      fillStyle: "#495057",
    },
  });

  // 直接在初始化那里加不会生效
  circle.restitution = 1.5;

  return circle;
}
```

## 设置`group`。让 stopper、marble 它们变成“一伙”

> 现在弹珠碰到`stopper`物块也会有碰撞效果。所以设置`group`，成群之后就不会再有碰撞效果了。

```js
const group = Matter.Body.nextGroup(true);


function stopper(x, y, side, position) {
  // ...
  const options = {
    isStatic true,
    render: {
      fillStyle: 'red'
    },
    collisionFilter: {
      group: group,
    }
    // ...
  };
  // ...
}

function addMarble() {
  const marble = Matter.Bodies.circle(0, 0, 10, {
    render: {
      fillStyle: 'green'
    },
    collisionFilter: {
      group: group
    }
  });
  Matter.Composite.add(engine.world, marble);

  // 设置坐标
  Matter.Body.setPosition(marble, {
    x: 464,
    y: 500,
  });

  // 设置力
  Matter.Body.setVelocity(marble, {
    x: 0,
    y: -22,  // 球变小了。所以里也应该相应减小。
  });
}
```

> \color{red}{弹珠的大小从 14 变成了 10}。后面发现会被`brick`卡住，并且没法单独给`brick`设置`group`。

## 隐藏辅助物块

```js
function stopper(x, y, side, position) {
  // ...
  const options = {
    isStatic true,
    render: {
      fillStyle: 'red',
      visible: false,
    },
    // ...
  };
  // ...
}


  paddleLeft.brick = Matter.Bodies.rectangle(134, 524, 40, 40, {
    render: {
      fillStyle: 'blue',
      visible: false
    },
  });
  // paddleRight.brick同理
```

## 效果以及完整代码

![](https://www.clzczh.top/CLZ_img/images/202312212241074.gif)

```js
import Matter from "matter-js";
import "pathseg";
import MatterAttractors from "matter-attractors";

let engine;

const PATHS = {
  leftArrow: "M 0 0 L 40 60 L 0 100 L 0 0",
  // 最高的那个点要作为出发点（不知道具体原因）
  rightArrow: "M 40 -60 L 40 40 L 0 0 L 40 -60",
  leftBottom: "M 0 0 L 0 -140 L 180 0 L 0 0",
  rightBottom: "M 0 -140 L 0 0 L -180 0 L 0 -140",
};

const WIDTH = 500;
const HEIGHT = 640;

const group = Matter.Body.nextGroup(true);

function init() {
  Matter.Common.setDecomp(require("poly-decomp"));
  Matter.use(MatterAttractors);

  engine = Matter.Engine.create();

  const render = Matter.Render.create({
    element: document.querySelector(".container"),
    engine: engine,
    options: {
      width: WIDTH,
      height: HEIGHT,
      wireframes: false,
    },
  });
  Matter.Render.run(render);

  const runner = Matter.Runner.create();
  Matter.Runner.run(runner, engine);

  const mouse = Matter.Mouse.create(render.canvas);
  const mouseConstraint = Matter.MouseConstraint.create(engine, {
    mouse: mouse,
    constraint: {
      stiffness: 0.2,
      render: {
        visible: false,
      },
    },
  });

  Matter.Composite.add(engine.world, mouseConstraint);
}

function addBoundries() {
  svg(236, 80, "dome");

  Matter.Composite.add(engine.world, [
    boundary(0, HEIGHT / 2, 40, HEIGHT),
    boundary(WIDTH, HEIGHT / 2, 40, HEIGHT),
    boundary(WIDTH / 2, HEIGHT, HEIGHT, 40),

    wall(150, 100, 18, 40),
    wall(230, 100, 18, 40),
    wall(320, 100, 18, 40),

    circle(100, 180, 20),
    circle(225, 180, 20),
    circle(350, 180, 20),

    circle(160, 260, 20),
    circle(290, 260, 20),

    wall(440, 420, 20, 450),

    wall(120, 380, 20, 110),
    wall(320, 380, 20, 110),

    wall(60, 400, 20, 150),
    wall(88, 485, 20, 88, {
      angle: -0.95,
    }),

    wall(380, 400, 20, 150),
    wall(352, 485, 20, 88, {
      angle: 0.95,
    }),

    reset(225, 64),
    reset(465, 32),

    path(35, 260, PATHS.leftArrow),
    path(416, 280, PATHS.rightArrow),

    path(80, 580, PATHS.leftBottom),
    path(370, 580, PATHS.rightBottom),
  ]);
}

let isLeftPaddleUp = false;
let isRightPaddleUp = false;
function addEvents() {
  window.addEventListener("keypress", (e) => {
    if (e.key.toLowerCase() === "a") {
      isLeftPaddleUp = true;
    } else {
      isLeftPaddleUp = false;
    }

    if (e.key.toLowerCase() === "d") {
      isRightPaddleUp = true;
    } else {
      isRightPaddleUp = false;
    }
  });

  window.addEventListener("keyup", (e) => {
    if (e.key.toLowerCase() === "a") {
      isLeftPaddleUp = false;
    }

    if (e.key.toLowerCase() === "d") {
      isRightPaddleUp = false;
    }
  });

  Matter.Events.on(engine, "collisionStart", (e) => {
    e.pairs.forEach(function (pair) {
      const bodyA = pair.bodyA;
      const bodyB = pair.bodyB;

      if (bodyA.label === "reset") {
        Matter.Composite.remove(engine.world, [bodyB]);
        addMarble();
      }

      if (bodyB.label === "reset") {
        Matter.Composite.remove(engine.world, [bodyA]);
        addMarble();
      }
    });
  });
}

function getPaddleStatus(side) {
  const isPaddleUp = side === "left" ? isLeftPaddleUp : isRightPaddleUp;
  return isPaddleUp;
}

function stopper(x, y, side, position) {
  const judgeLabel = side === "left" ? "paddleLeftComp" : "paddleRightComp";

  const options = {
    isStatic: true,
    render: {
      fillStyle: "red",
      visible: false,
    },
    collisionFilter: {
      group: group,
    },
    plugin: {
      attractors: [
        function (bodyA, bodyB) {
          if (bodyB.label === judgeLabel) {
            return {
              x:
                (bodyA.position.x - bodyB.position.x) *
                0.002 *
                (getPaddleStatus(side) ? -1 : 0.5), // 0.5是防止松手后，吸引力太大导致变形
              y:
                (bodyA.position.y - bodyB.position.y) *
                0.002 *
                (getPaddleStatus(side) ? -1 : 0.5),
            };
          }
        },
      ],
    },
  };

  const hadForce = position === "bottom";
  if (!hadForce) {
    // 只有下面的stopper有引（斥）力
    Reflect.deleteProperty(options, "plugin");
  }

  return Matter.Bodies.circle(x, y, 20, options);
}

function addPaddles() {
  const stoperLeftTop = stopper(170, 460, "left", "top");
  const stoperLeftBottom = stopper(136, 580, "left", "bottom");
  const stoperRightTop = stopper(280, 460, "right", "top");
  const stoperRightBottom = stopper(300, 580, "right", "bottom");

  Matter.Composite.add(engine.world, [
    stoperLeftTop,
    stoperLeftBottom,
    stoperRightTop,
    stoperRightBottom,
  ]);

  const paddleLeft = {};
  paddleLeft.paddle = Matter.Bodies.trapezoid(134, 512, 20, 88, 0.33, {
    label: "paddleLeft",
    angle: 1.57,
    chamfer: {},
    render: {
      fillStyle: "skyblue",
    },
  });

  paddleLeft.brick = Matter.Bodies.rectangle(134, 524, 40, 40, {
    render: {
      fillStyle: "blue",
      visible: false,
    },
  });

  // 将两个物块组装在一起
  paddleLeft.comp = Matter.Body.create({
    label: "paddleLeftComp",
    parts: [paddleLeft.paddle, paddleLeft.brick],
  });

  // bodyB表示给paddleLeft.comp添加约束，pointB则是相对paddleLeft.comp的坐标
  // pointA是这个物理世界的坐标
  const constraintLeft = Matter.Constraint.create({
    bodyB: paddleLeft.comp,
    pointB: {
      x: -32,
      y: -8,
    },
    pointA: {
      x: paddleLeft.comp.position.x,
      y: paddleLeft.comp.position.y,
    },
    stiffness: 0.9,
    length: 0,
    render: {
      strokeStyle: "pink",
    },
  });
  Matter.Composite.add(engine.world, constraintLeft);

  const paddleRight = {};
  paddleRight.paddle = Matter.Bodies.trapezoid(304, 512, 20, 88, 0.33, {
    // isStatic: true,
    label: "paddleRight",
    angle: -1.57,
    chamfer: {},
    render: {
      fillStyle: "skyblue",
    },
  });

  paddleRight.brick = Matter.Bodies.rectangle(304, 524, 40, 40, {
    render: {
      fillStyle: "blue",
      visible: false,
    },
  });

  paddleRight.comp = Matter.Body.create({
    label: "paddleRightComp",
    parts: [paddleRight.paddle, paddleRight.brick],
  });

  const constraintRight = Matter.Constraint.create({
    bodyB: paddleRight.comp,
    pointB: {
      x: 32,
      y: -8,
    },
    pointA: {
      x: paddleRight.comp.position.x,
      y: paddleRight.comp.position.y,
    },
    stiffness: 0.9,
    length: 0,
    render: {
      strokeStyle: "pink",
    },
  });
  Matter.Composite.add(engine.world, constraintRight);

  Matter.Composite.add(engine.world, [paddleLeft.comp, paddleRight.comp]);
}

function addMarble() {
  const marble = Matter.Bodies.circle(0, 0, 10, {
    render: {
      fillStyle: "green",
    },
    collisionFilter: {
      group: group,
    },
  });
  Matter.Composite.add(engine.world, marble);

  // 设置坐标
  Matter.Body.setPosition(marble, {
    x: 464,
    y: 500,
  });

  // 设置力
  Matter.Body.setVelocity(marble, {
    x: 0,
    y: -22,
  });
}

function boundary(x, y, width, height) {
  return Matter.Bodies.rectangle(x, y, width, height, {
    isStatic: true,
    render: {
      fillStyle: "#495057",
    },
  });
}

function circle(x, y, radius) {
  const circle = Matter.Bodies.circle(x, y, radius, {
    isStatic: true,
    render: {
      fillStyle: "#495057",
    },
  });

  // 直接在初始化那里加不会生效
  circle.restitution = 1.5;

  return circle;
}

function wall(x, y, width, height, options) {
  return Matter.Bodies.rectangle(x, y, width, height, {
    isStatic: true,
    angle: options && options.angle ? options.angle : 0,
    chamfer: {
      radius: 10,
    },
    render: {
      fillStyle: "#495057",
    },
  });
}

function reset(x, width) {
  return Matter.Bodies.rectangle(x, 620, width, 2, {
    label: "reset",
    isStatic: true,
    render: {
      fillStyle: "#fff",
    },
  });
}

function path(x, y, path) {
  const vertices = Matter.Vertices.fromPath(path);

  return Matter.Bodies.fromVertices(x, y, vertices, {
    isStatic: true,
    render: {
      fillStyle: "pink",
      strokeStyle: "pink",
      lineWidth: 1,
    },
  });
}

const loadSvg = function (url) {
  return fetch(url)
    .then(function (response) {
      return response.text();
    })
    .then(function (raw) {
      // 把加载的svg转成document对象
      return new window.DOMParser().parseFromString(raw, "image/svg+xml");
    });
};

function svg(x, y, svgName) {
  loadSvg(`/svg/${svgName}.svg`).then((root) => {
    console.log(root);

    const vertices = [...root.querySelectorAll("path")].map((path) => {
      return Matter.Svg.pathToVertices(path, 36);
    });

    Matter.Composite.add(
      engine.world,
      Matter.Bodies.fromVertices(x, y, vertices, {
        isStatic: true,
        render: {
          fillStyle: "#495057",
          lineWidth: 1,
        },
      })
    );
  });
}

function load() {
  init();
  addBoundries();
  addEvents();
  addPaddles();
  addMarble();
}

load();
```

> 代码放仓库了，有兴趣的话，可以查看。
>
> [GitHub - clzczhc/marble](https://github.com/clzczhc/marble)

## 参考链接

[JavaScript Physics with Matter.js / Coder's Block](https://codersblock.com/blog/javascript-physics-with-matter-js/)

[SvgPathEditor](https://yqnn.github.io/svg-path-editor/)

[Matter.js - a 2D rigid body JavaScript physics engine · code by @liabru](https://brm.io/matter-js/)

[GitHub - liabru/matter-attractors: an attractors plugin for matter.js](https://github.com/liabru/matter-attractors)
