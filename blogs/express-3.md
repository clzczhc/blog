---
title: Express实战(一)	项目结构搭建及验证、加密
date: 2022-03-06 13:19:56
keywords: Express
categories: "后端"
tags:
  - Express
---

# Express 实战(一) 项目结构搭建及验证、加密

开源项目：

- [github 仓库](https://github.com/gothinkster/realworld)

- [接口文档](https://realworld-docs.netlify.app/docs/specs/backend-specs/endpoints)

<br>

[RESTful 接口设计规范](https://clz.vercel.app/2022/02/28/RESTful/)

最终结果：[realworld-api-express-practise- ](https://github.com/13535944743/realworld-api-express-practise-)

## 1. 创建项目

```shell
mkdir realworld-api-express
cd realworld-api-express
npm init -y
npm install express
```

目录结构：

```
.
├── node_modules npm安装的第三方包目录，使用 npm 装包会自动创建
├── config	# 配置文件
|	├── config.default.js
├── controller	# 解析用户的输入，处理后返回相应的结果
├── model	# 数据持久层
├── middleware	# 中间件
├── router	# 配置URL路由
├── util	# 工具模块
├── app.js 服务端程序入口文件，执行该文件会启动我们的 Web 服务器
├── package.json 项目包说明文件，存储第三方包依赖等信息
└── package-lock.json npm的包锁定文件，用来锁定第三方包的版本和提高npm下载速度
```

## 3. 配置常用中间件

- **解析请求体**
  - express.json()
  - express.urlencoded()
- **日志输出**
  - morgan()
- **提供跨域资源请求**
  - cors()

```js
const express = require("express");
const morgan = require("morgan");
const cors = require("cors");

const app = express();

// 日志输出
app.use(morgan("dev"));

// 解析请求体的中间件
app.use(express.json());
app.use(express.urlencoded());

// 提供跨域资源请求
app.use(cors());

const PORT = process.env.PORT || 3000; // process.env.PORT读取当前目录下环境变量port的值，若没有则默认端口为3000

app.post("/", (req, res) => {
  res.send(req.body);
});

app.listen(PORT, () => {
  console.log(`Server is running at http://localhost:${PORT}`);
});
```

![image-20220211002712124](https://s2.loli.net/2022/03/06/1LCVTXJmcyZiSKA.png)

## 4. 路由设计

[接口文档](https://realworld-docs.netlify.app/docs/specs/backend-specs/endpoints)

用户相关路由(user.js)：

```js
const express = require("express");

const router = express.Router();

// 用户登录
router.post("/users/login", async (req, res, next) => {
  try {
    res.send("用户登录");
  } catch (err) {
    next(err);
  }
});

// 用户注册
router.post("/users", async (req, res, next) => {
  try {
    res.send("用户注册");
  } catch (err) {
    next(err);
  }
});

// 获取当前用户
router.get("/user", async (req, res, next) => {
  try {
    res.send("获取当前用户");
  } catch (err) {
    next(err);
  }
});

// 更新用户
router.put("/user", async (req, res, next) => {
  try {
    res.send("更新用户");
  } catch (err) {
    next(err);
  }
});

module.exports = router;
```

index.js

```js
const express = require("express");

const router = express.Router();

// 用户相关路由
router.use(require("./user"));

// 用户资料相关路由
router.use("/profiles", require("./profile"));

module.exports = router;
```

app.js 挂载路由级别中间件

![image-20220211125351747](https://s2.loli.net/2022/03/06/GjxSF28w5asHzKA.png)

其他路由做法类似

## 5. 提取控制器模块

简单来说，就是把路由处理的回调函数单独抽出来，放到另一个地方，方便维护等操作。

示例(userController)：

```js
// 用户登录
exports.login = async (req, res, next) => {
  try {
    res.send("用户登录");
  } catch (err) {
    next(err);
  }
};
```

![image-20220211185051313](https://s2.loli.net/2022/03/06/n9uJZeIOog8tMlU.png)

再优化一下(将控制器封装成类)

```js
class userController {
  // 用户登录
  async login(req, res, next) {
    try {
      res.send("用户登录");
    } catch (err) {
      next(err);
    }
  }

  // 用户注册
  async register(req, res, next) {
    try {
      res.send("用户注册");
    } catch (err) {
      next(err);
    }
  }

  // 获取当前用户
  async getCurrentUser(req, res, next) {
    try {
      res.send("获取当前用户");
    } catch (err) {
      next(err);
    }
  }

  // 更新用户
  async updateUser(req, res, next) {
    try {
      res.send("更新用户");
    } catch (err) {
      next(err);
    }
  }
}

module.exports = new userController();
```

## 6. 配置错误处理中间件

error-handler

```js
module.exports = () => {
  return (err, req, res, next) => {
    // 错误处理中间件必须要四个参数
    res.status(500).json({
      error: err.message,
    });
  };
};
```

![image-20220211210724530](https://s2.loli.net/2022/03/06/lckJVbijDN7ShK8.png)

<b style="color: red">错误处理中间件应该在最后才挂载</b>

测试：![image-20220211210925234](https://s2.loli.net/2022/03/06/UwrY8Ha2xVnSPBX.png)

​ ![image-20220211210949033](https://s2.loli.net/2022/03/06/fV9Y3abCJ1WjneS.png)

## 7. 统一管理测试接口(postman)

![image-20220211212812413](https://s2.loli.net/2022/03/06/Ewk4XzUAx3MFjyS.png)

选择对应环境(Development 或 Production)，没有则点击 No Environment，然后点击小眼睛，添加环境

设置基础路径(选择环境后，点击小眼睛进行编辑，如下图所示)

![image-20220211213132330](https://s2.loli.net/2022/03/06/zwvo7CjGmUr2euN.png)

![image-20220211213615327](https://s2.loli.net/2022/03/06/23lmkYXwPHbdRr5.png)

## 8. 使用 mongodb 数据库

### 8.1 安装 mongodb

[MongoDB Community Download | MongoDB](https://www.mongodb.com/try/download/community)

先打开 MongoDB Compass

![image-20220212132611027](https://s2.loli.net/2022/03/06/zwLaTcpu3FEs6RD.png)

### 8.2 连接 Mongodb 数据库

首先，需要安装 Mongoose，` npm install mongoose`

[Mongoose 5.0 中文文档](http://www.mongoosejs.net/docs/)

model \ index.js

```js
const mongoose = require("mongoose");

const { dbURL } = require("../config/config.default");

// 连接MongoDB数据库
mongoose.connect(dbURL);

var db = mongoose.connection;

// 数据库连接失败
db.on("error", (err) => {
  console.log("MongoDB 数据库连接失败", err);
});

// 数据库连接成功
db.once("open", function () {
  console.log("MongoDB 数据库连接成功");
});
```

数据库地址在配置中(便于上线等操作时直接更换地址)

config \ config.default.js

```js
module.exports = {
  dbURL: "mongodb://localhost:27017/realworld", // MongoDB默认端口17017
};
```

运行(` nodemon app.js`)

![image-20220212133236905](https://s2.loli.net/2022/03/06/mr56Ij931dMCGvQ.png)

### 8.3 增加数据模块(以 user 为例)

根据接口文档，确定属性

[接口文档](https://realworld-docs.netlify.app/docs/specs/backend-specs/endpoints)

![image-20220212133538143](https://s2.loli.net/2022/03/06/1U2MkDzV4AmBZHb.png)

model \ base-model.js（**存放共有的属性**，如创建时间，更新时间等）

```js
module.exports = {
  createdAt: {
    type: Date,
    default: Date.now,
  },
  updatedAt: {
    type: Date,
    default: Date.now,
  },
};
```

model \ user.js

```js
const mongoose = require("mongoose");

const baseModel = require("./base-model");

// 创建用户模型
const userSchema = mongoose.Schema({
  ...baseModel,
  username: {
    type: String,
    required: true,
  },
  email: {
    type: String,
    required: true,
  },
  password: {
    type: String,
    required: true,
  },
  bio: {
    // 简介
    type: String,
    default: null,
  },
  image: {
    // 头像
    type: String,
    default: null,
  },
});

module.exports = userSchema;
```

module \ index.js

```js
const mongoose = require("mongoose");

const { dbURL } = require("../config/config.default");

// 连接MongoDB数据库
mongoose.connect(dbURL);

var db = mongoose.connection;

// 数据库连接失败
db.on("error", (err) => {
  console.log("MongoDB 数据库连接失败", err);
});

// 数据库连接成功
db.once("open", function () {
  console.log("MongoDB 数据库连接成功");
});

// 组织导出模型类
module.exports = {
  User: mongoose.model("User", require("./user")),
  Article: mongoose.model("Article", require("./article")),
};
```

### 8.4 注册，操作数据库

controller \ userController.js

![image-20220212140140573](https://s2.loli.net/2022/03/06/TDULJdzurGaHR9e.png)

![image-20220212134443019](https://s2.loli.net/2022/03/06/hxuJtTKnz2U61mE.png)

## 9. 验证

首先，mongodb 添加模式时的` required: true`可以实现一点验证是否缺必需参数。但是，还远远不够，以下提供两个验证的库。

- [awesome-nodejs](https://github.com/sindresorhus/awesome-nodejs)
- [express-validator](https://github.com/express-validator/express-validator)

### 9.1 基本使用

具体使用可查看[文档](https://express-validator.github.io/docs/)

```
npm install express-validator
```

router \ user.js

![image-20220212150822951](https://s2.loli.net/2022/03/06/B7hdIYKL5UibsrD.png)

```js
// 用户注册
router.post(
  "/users",
  [
    // 1. 配置验证规则
    body("user.username").notEmpty().withMessage("用户名不能为空"), // withMessage()定制错误提示消息
    body("user.email")
      .notEmpty()
      .withMessage("邮箱不能为空")
      .isEmail()
      .withMessage("邮箱格式不正确")
      .bail() // 前面验证失败则不会往后执行
      .custom(async (email) => {
        // 自定义验证
        const user = await User.findOne({ email });
        if (user) {
          // 有相同邮箱的用户存在
          return Promise.reject("邮箱已存在");
        }
      }),
    body("user.password").notEmpty().withMessage("密码不能为空"),
  ],
  (req, res, next) => {
    // 2. 判断验证结果
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    next(); // 这个一定要加，不然通过验证的情况就会卡住
  },
  userController.register
); // 3. 通过验证
```

![image-20220212150745557](https://s2.loli.net/2022/03/06/SVXpFAc5r1bZGCz.png)

### 9.2 提取验证中间件模块

首先根据官方文档，增加验证中间件

![image-20220212151331241](https://s2.loli.net/2022/03/06/CuIfyMkhGjaBiOF.png)

middleware \ validate.js

```js
const { validationResult } = require("express-validator");

module.exports = (validations) => {
  return async (req, res, next) => {
    await Promise.all(validations.map((validation) => validation.run(req)));

    const errors = validationResult(req);
    if (errors.isEmpty()) {
      return next();
    }

    res.status(400).json({ errors: errors.array() });
  };
};
```

新增文件夹 validate(验证业务逻辑代码)

validate \ user.js(用户的验证逻辑)

```js
const validate = require("../middleware/validate");
const { body, validationResult } = require("express-validator");

const { User } = require("../model");

exports.register = validate([
  // 1. 配置验证规则
  body("user.username").notEmpty().withMessage("用户名不能为空"), // withMessage()定制错误提示消息
  body("user.email")
    .notEmpty()
    .withMessage("邮箱不能为空")
    .isEmail()
    .withMessage("邮箱格式不正确")
    .bail() // 前面验证失败则不会往后执行
    .custom(async (email) => {
      // 自定义验证
      const user = await User.findOne({ email });
      if (user) {
        // 有相同邮箱的用户存在
        return Promise.reject("邮箱已存在");
      }
    }),
  body("user.password").notEmpty().withMessage("密码不能为空"),
]);
```

修改` router \ user.js`

![image-20220212151657837](https://s2.loli.net/2022/03/06/6tmrvNaL92XyxDs.png)

## 10. 密码加密处理

开始前，先了解一下，MD5 的使用

```js
const crypto = require("crypto");

// 获取crypto支持的散列算法
console.log(crypto.getHashes());

const ret = crypto
  .createHash("md5") // 必须是crypto支持的散列算法
  .update("456")
  .digest("hex"); // hex代表生成的序列是十进制的

console.log(ret);
```

![image-20220212152639731](https://s2.loli.net/2022/03/06/rbVPspS6oA781fJ.png)

开搞。

1. 封装 md5 模块

   util \ md5.js

   ```js
   const crypto = require("crypto");

   module.exports = (str) => {
     // str是明文，即要加密的数据
     return crypto
       .createHash("md5") // 必须是crypto支持的散列算法
       .update(str) // 可以凭借字符串，达到混淆效果，如hunxiao + str
       .digest("hex");
   };
   ```

2. 使用 md5 模块

   ![image-20220212155626831](https://s2.loli.net/2022/03/06/wg6r2EeWGoXsM3d.png)

   ![image-20220212155939195](https://s2.loli.net/2022/03/06/uaKD8TVZwJgAyoR.png)

3. 上面返回给用户的数据中，密码也给返回了，所以有点危险

   用户模型修改：

   ```js
   password: {
       type: String,
       required: true,
       set: value => md5(value),    // 对密码赋值时，会调用md5进行加密
       select: false    // 查询数据时，不显示出来

     }
   ```

   结果还是不对，因为注册的用户是新 new 出来的对象，而不是查询出来的

   直接删除

   ```js
     // 用户注册
     async register(req, res, next) {
       try {
         let user = new User(req.body.user)
         await user.save()

         user = user.toJSON()    // 因为user是mongodb的对象，不能直接delete，所以先把它转成JSON
         delete user.password

         // 发送成功响应
         res.status(201).json({
           user
         })
       } catch (err) {
         next(err)
       }
     }
   ```

   ![image-20220212162415623](https://s2.loli.net/2022/03/06/c1PziKea9bGo2Iu.png)

视频教程：[Node.js 系列教程之 Express\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1mQ4y1C7Cn)
