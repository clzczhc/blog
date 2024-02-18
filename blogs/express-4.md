---
title: Express实战(二)    登录验证、身份认证、增删改查
date: 2022-03-06 13:20:02
keywords: Express
categories: "后端"
tags:
  - Express
---

# Express 实战(二) 登录验证、身份认证、增删改查

最终结果：[realworld-api-express-practise- ](https://github.com/13535944743/realworld-api-express-practise-)

## 1. 数据验证(登录验证)

validate \ user.js

```js
exports.login = [
  validate([
    body("user.email").notEmpty().withMessage("邮箱不能为空"),
    body("user.password").notEmpty().withMessage("密码不能为空"),
  ]),
  validate([
    // 只有上面的验证通过才会执行，利用的是中间件的机制
    body("user.email").custom(async (email, { req }) => {
      // 这里参数的req解构是官网文档用法
      const user = await User.findOne({
        email,
      }).select(["password", "username", "email", "bio", "image"]); // 这里需要获取密码的话，因为用户密码的模式设计那里设置了select: false，即通过查找不能查到密码，此时需要通过select()实现能查出密码
      if (!user) {
        return Promise.reject("用户不存在");
      }

      // 将数据挂载到请求对象上，这样子后续的中间件也可以直接使用
      req.user = user;
    }),
  ]),
  validate([
    body("user.password").custom(async (password, { req }) => {
      if (md5(password) !== req.user.password) {
        return Promise.reject("密码错误");
      }

      console.log(req.user);
    }),
  ]),
];
```

user 的路由那里也要加上

router \ user.js

![image-20220212175651983](https://s2.loli.net/2022/03/06/VOkFwbflWv32tqx.png)

![image-20220212175753306](https://s2.loli.net/2022/03/06/Q72rvg8PFGHahKW.png)

![image-20220212175808931](https://s2.loli.net/2022/03/06/QeX7S6bxAN98OCP.png)

## 2. 基于 JWT 的身份认证

[JSON Web Tokens - jwt.io](https://jwt.io/)

JWT 原理：服务器认证之后，生成一个 JSON 对象，类似下面

```js
{
    "姓名": "clz",
    "角色": "admin",
    "到期时间": "2022-02-28"
}
```

以后，用户和服务端通信，都要发回这个 JSON 对象，服务器只靠这个对象确认用户身份。为了防止用户篡改数据，服务器在生成这个对象时，会加上签名。

实际 JWT：

![image-20220212181436946](https://s2.loli.net/2022/03/06/e6YlBAa9SKiLIyP.png)

JWT 的三个部分：

- Header（头部）
- Payload（负载）
- Signature（签名）

Header.Payload.Signature

### 2.1 Header

Header 部分是一个 JSON 对象，描述 JWT 的元数据

```js
{
    "alg": "HS256",		// 表示签名的算法，默认是HMAC SHA256
    "typ": "JWT"		// 表示令牌（token）的类型，ＪＷＴ令牌写为ＪＷＴ
}
```

最后通过` Base64URL`算法将上面的ＪＳＯＮ对象转成字符串

### 2.2 Payload

Payload 也是一个 JSON 对象，用来存实际需要传的数据。JWT 规定了 7 个官方字段

- iss(issuer)：签发人
- exp(expiration time)：过期时间
- sub(subject)：主题
- aud(audience)：受众
- nbf(Not Before)：生效时间
- iat(Issued At)：签发时间
- jti(JWT ID)：编号

除了官方字段，还可以定义私有字段

```js
{
    "sub": "134567890",
    "name": "clz",
    "admin": true
}
```

<b style="color: red">JWT 默认是不加密的，所以需要保密的信息不应该放在这部分</b>

最后通过` Base64URL`算法将上面的 JSON 对象转成字符串

### 2.3 Signature

Signature 是对前两部分的签名，防止数据篡改

首先，需要指定一个密钥(<b style="color: red">这个密钥只有服务器知道，不能泄露给用户</b>)。然后，使用 Header 里面指定的签名算法，按以下公式产生签名。

```js
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret // 私钥
);
```

得到签名后，将 Header、Payload、Signature 三个部分拼接成一个字符串，用` .`分隔，可以返回给用户

<b style="Color: red">在 JWT 中，消息体是透明的，使用签名可以保证消息不被篡改，但不能实现数据加密功能</b>

将 Header 和 Payload 串型化的算法是` BaseURL`，和` Base64`算法类似，但有一些不同。

JWT 作为一个令牌(token)，有时候需要放到 URL 中(如 api.example.com/?token=xxx)。

- Base64 中的三个字符` +`, ` /`, ` =` ，在 URL 中有特殊意义
- Base64URL：` =`被省略，` +`替换成` -`，` /`替换成` _`

### 2.4 JWT 的使用方式

客户端收到服务器返回的 JWT，可以存在 Cookie 里，也可以存在 localStorage 中。之后，客户端与服务器通信，都要带上这个 JWT，可以将 JWT 放在 Cookie 里自动发送，不过这样子不能跨域。<b style="color: red">更好的做法是：放在 HTTP 请求头的` Authorization`字段里面</b>

```
Authorization: Bearer <token>
```

### 2.5 使用 jsonwebtoken

[jsonwebtoken 仓库](https://github.com/auth0/node-jsonwebtoken)

了解 jsonwebtoken 的使用

先安装，` npm install jsonwebtoken`

```js
const jwt = require("jsonwebtoken");

// 生成jwt：jwt.sign
// // 同步方式:
// const token = jwt.sign({ foo: 'bar' }, 'hello');
// console.log(token)

// 异步方式：就只是加多一个回调函数
const token = jwt.sign(
  {
    foo: "bar",
  },
  "hello",
  (err, token) => {
    if (err) {
      return console.log("生成token失败");
    }
    console.log(token);
  }
);

// 验证jwt：jwt.verify
// 同步方式：
// const result = jwt.verify('eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.\
// eyJmb28iOiJiYXIiLCJpYXQiOjE2NDQ2NjY1NDd9.\
// 0Vy596XulYTCxeTrBp27U2T4BMh93IPN5l2b0GqxAMY', 'hello')

// console.log(result)

// 异步方式：
jwt.verify("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.\
eyJmb28iOiJiYXIiLCJpYXQiOjE2NDQ2NjY1NDd9.\
0Vy596XulYTCxeTrBp27U2T4BMh93IPN5l2b0GqxAMY", "hello", (err, ret) => {
  if (err) {
    return console.log("验证token失败");
  }
  console.log(ret);
});
```

#### 2.5.1 生成 token

util \ jwt.js

```js
const jwt = require("jsonwebtoken");
const { promisify } = require("util"); // 将回调函数转换成Promise形式

exports.sign = promisify(jwt.sign);

exports.verify = promisify(jwt.verify);

exports.decode = promisify(jwt.decode); // 不验证，直接解析
```

config \ config.default.js

```js
module.exports = {
  dbURL: "mongodb://localhost:27017/realworld", // MongoDB默认端口17017
  jwtSecret: "c06eddf5-78eb-494f-b2c6-4a6d45b56cd5", // uuid随机生成(直接搜索uuid)
};
```

controller userController.js（只改登录部分）

```js
// 类前面引入
const jwt = require('../util/jwt')
const { jwtSecret } = require('../config/config.default')

// 用户登录
async login(req, res, next) {
  try {
    // 1. 数据验证
    // 2. 生成token
    const user = req.user.toJSON()

    const token = await jwt.sign({
      userId: user._id    // 生成token不需要全部user信息，只要_id即可
    }, jwtSecret)

    // 3. 发送成功响应(包含token的用户信息)
    delete user.password
    res.status(200).json({
      ...user,
      token
    })
  } catch (err) {
    next(err)
  }
}
```

![image-20220212204150260](https://s2.loli.net/2022/03/06/7E9rMZ3c8QU42sS.png)

### 2.6 中间件统一处理 JWT 身份认证

middleware \ authorization.js

```js
const { verify } = require("../util/jwt");
const { jwtSecret } = require("../config/config.default");
const { User } = require("../model/index");

module.exports = async (req, res, next) => {
  // 1. 从请求头获取token
  let token = req.headers["authorization"];

  token = token ? token.split("Bearer ")[1] : null;

  if (!token) {
    return res.status(401).end("请求头无token或token格式不对");
  }

  try {
    // 2. 验证token是否有效
    //    无效 ==> 响应401状态码
    //    有效 ==> 把用户信息读取出来，并挂载到req请求对象中，继续往后执行
    const decodedToken = await verify(token, jwtSecret);
    req.user = await User.findById(decodedToken.userId);

    next();
  } catch (err) {
    return res.status(401).end("token无效");
  }
};
```

![image-20220213193413017](https://s2.loli.net/2022/03/06/QsY2dEJHM48ISLB.png)

![image-20220213193952500](https://s2.loli.net/2022/03/06/bqK4VkZ9XQsWIyH.png)

### 2.7 JWT 过期时间

![image-20220213194228528](https://s2.loli.net/2022/03/06/6lFVXrutyLmdkOo.png)

设置为 15 秒，体验下过期

![jwt](https://s2.loli.net/2022/03/06/XBpAb8rQvGM9qDS.gif)

### 2.8 Postman 自动添加 token

![image-20220213195824827](https://s2.loli.net/2022/03/06/ES316tO8CoMA9BY.png)

![image-20220213200015365](https://s2.loli.net/2022/03/06/2c8tgln6ekvhGYq.png)

## 3. 新增文章

**和注册类似**

### 3.1 数据验证

validate \ article.js

```js
const validate = require("../middleware/validate");
const { body } = require("express-validator");

exports.createArticle = validate([
  body("article.title").notEmpty().withMessage("文章标题不能为空"),
  body("article.description").notEmpty().withMessage("文章摘要不能为空"),
  body("article.body").notEmpty().withMessage("文章内容不能为空"),
]);
```

### 3.2 文章模型

model \ article.js

```js
const mongoose = require("mongoose");

const baseModel = require("./base-model");

const Schema = mongoose.Schema;

// 创建文章模型
const articleSchema = mongoose.Schema({
  ...baseModel,
  title: {
    type: String,
    required: true,
  },
  description: {
    type: String,
    required: true,
  },
  body: {
    type: String,
    required: true,
  },
  tagList: {
    type: [String],
    default: null,
  },
  favoritesCount: {
    type: Number,
    default: 0,
  },
  author: {
    type: Schema.Types.ObjectId,
    ref: "User", // 存用户id，之后映射到用户模型去
    required: true,
  },
});

module.exports = articleSchema;
```

<b style="Color: red">ref 中的值需要时，model \ index.js 中导出的模型类中启用的名字</b>

![image-20220214134231860](https://s2.loli.net/2022/03/06/LgHpIeY5b8FUqdj.png)

### 3.3 文章相关路由

<b style="color: red">新增文章部分加上了 JWT 身份认证和数据验证</b>

router \ article.js

```js
const express = require("express");

const articleController = require("../controller/articleController");
const authorization = require("../middleware/authorization");
const articleValidate = require("../validate/article");

const router = express.Router();

// 获取所有文章(可增加条件筛选)
router.get("/", articleController.listArticles);

// 获取关注用户的所有文章(可增加条件筛选)
router.get("/feed", articleController.feedArticles);

// 获取单篇文章
router.get("/:slug", articleController.getArticle); // slug类似id，用于确定特定文章

// 新增文章
router.post(
  "/",
  authorization,
  articleValidate.createArticle,
  articleController.createArticle
);

// 更新文章
router.put("/:slug", articleController.updateArticle);

// 删除文章
router.delete("/:slug", articleController.deleteArticle);

// 增加一篇文章的评论
router.post("/:slug/comments", articleController.addComments);

// 获取一篇文章的所有评论
router.get("/:slug/comments", articleController.getComments);

// 删除文章的一条评论
router.delete("/:slug/comments/:id", articleController.deleteComment);

// 喜欢一篇文章
router.post("/:slug/favorite", articleController.likeArticle);

// 取消喜欢一篇文章
router.delete("/:slug/favorite", articleController.unlikeArticle);

module.exports = router;
```

### 3.4 处理请求

controller \ articleController.js

```js
const { Article } = require("../model/index");

class articleController {
  // 获取所有文章(可增加条件筛选)
  async listArticles(req, res, next) {
    try {
      res.send("获取所有文章");
    } catch (err) {
      next(err);
    }
  }

  // 获取关注用户的所有文章(可增加条件筛选)
  async feedArticles(req, res, next) {
    try {
      res.send("获取关注用户的所有文章");
    } catch (err) {
      next(err);
    }
  }

  // 获取单篇文章
  async getArticle(req, res, next) {
    try {
      res.send("获取单篇文章");
    } catch (err) {
      next(err);
    }
  }

  // 新增文章
  async createArticle(req, res, next) {
    try {
      const article = new Article(req.body.article);

      article.author = req.user._id; // 作者在数据库中只存一个用户id，通过id去获取用户
      article.populate("author"); // 简单来说，就是通过populate()以及文章模型中的ref: 'User'，可以通过id把用户信息放到author中

      // article.populate('author').execPopulate()

      await article.save();
      res.status(201).json({
        article,
      });
    } catch (err) {
      next(err);
    }
  }

  // 更新文章
  async updateArticle(req, res, next) {
    try {
      res.send("更新文章");
    } catch (err) {
      next(err);
    }
  }

  // 删除文章
  async deleteArticle(req, res, next) {
    try {
      res.send("删除文章");
    } catch (err) {
      next(err);
    }
  }

  // 增加一篇文章的评论
  async addComments(req, res, next) {
    try {
      res.send("增加一篇文章的评论");
    } catch (err) {
      next(err);
    }
  }

  // 获取一篇文章的所有评论
  async getComments(req, res, next) {
    try {
      res.send("获取一篇文章的评论");
    } catch (err) {
      next(err);
    }
  }

  // 删除文章的一条评论
  async deleteComment(req, res, next) {
    try {
      res.send("删除文章的一条评论");
    } catch (err) {
      next(err);
    }
  }

  // 喜欢一篇文章
  async likeArticle(req, res, next) {
    try {
      res.send("喜欢一篇文章");
    } catch (err) {
      next(err);
    }
  }

  // 取消喜欢一篇文章
  async unlikeArticle(req, res, next) {
    try {
      res.send("取消喜欢一篇文章");
    } catch (err) {
      next(err);
    }
  }
}

module.exports = new articleController();
```

<b style="Color: red">疑点：老师说查询时不需要 execPopulate()，new 出来时需要，相当于执行一次查询。但是个人试验时发现都不需要 execPopulate()，加上反而会出错</b>，类似` "article.populate(...).execPopulate is not a function"`

可能是时代变了，现在 new 出来的时候，也执行了

![image-20220214134928849](https://s2.loli.net/2022/03/06/ekR29q7nSZfCylB.png)

## 4. 查询文章

### 4.1 数据验证

model \ article.js（部分）

```js
exports.getArticle = validate([
  param("slug").custom(async (value) => {
    if (!mongoose.isValidObjectId(value)) {
      return Promise.reject("文章ID类型错误");
    }
  }),
]);
```

### 4.2 路由

router \ article.js（部分）

```js
// 获取单篇文章
router.get("/:slug", articleValidate.getArticle, articleController.getArticle); // slug类似id，用于确定特定文章
```

### 4.3 处理请求

controller \ articleController.js（部分）

```js
// 获取单篇文章
async getArticle(req, res, next) {
  try {
    const article = await Article.findById(req.params.slug).populate('author')

    if (!article) {
      return res.status(404).end()
    }

    res.status(200).json(
      article
    )
  } catch (err) {
    next(err)
  }
}
```

## 5. 获取所有文章

controller \ articleController.js（部分）

```js
// 获取所有文章(可增加条件筛选)
  async listArticles(req, res, next) {
    try {
      const {
        offset = 0,// offset默认为0
        limit = 20,
        tag,
        author
      } = req.query

      const filter = {}   // 用于筛选

      if (tag) {
        filter.tagList = tag
      }

      if (author) {
        const user = await User.findOne({ username: author })
        filter.author = user ? user._id : null  // 如果有这个作者，则获取作者的id用于筛选。没有则为null
      }

      const articlesCount = await Article.countDocuments()

      const articles = await Article.find(filter)   // 筛选出有这个标签的文章
        .skip(Number.parseInt(offset))       // 跳过多少条
        .limit(Number.parseInt(limit))      // 取多少条
        .sort({       // 排序， -1代表倒叙，1代表正序
          createdAt: -1
        })

      res.status(200).json({
        articles,
        articlesCount
      })
    } catch (err) {
      next(err)
    }
  }
```

## 6. 更新文章

### 6.1 封装验证 ID 是否有效

修改 validate 中间件

middle \ validate.js

```js
const { validationResult, buildCheckFunction } = require("express-validator");
const { isValidObjectId } = require("mongoose");

exports = module.exports = (validations) => {
  return async (req, res, next) => {
    await Promise.all(validations.map((validation) => validation.run(req)));

    const errors = validationResult(req);
    if (errors.isEmpty()) {
      return next();
    }

    res.status(400).json({ errors: errors.array() });
  };
};

exports.isValidObjectId = (location, fields) => {
  // 第一个参数是验证的数据的位置，第二个参数是验证数据字段
  return buildCheckFunction(location)(fields).custom(async (value) => {
    if (!isValidObjectId(value)) {
      return Promise.reject("ID不是有效的ObjectID");
    }
  });
};
```

### 6.2 修改 article 的验证以及添加更新文章的验证

validate \ article.js

```js
const validate = require("../middleware/validate");
const { body, param } = require("express-validator");
const { Article } = require("../model");

exports.createArticle = validate([
  body("article.title").notEmpty().withMessage("文章标题不能为空"),
  body("article.description").notEmpty().withMessage("文章摘要不能为空"),
  body("article.body").notEmpty().withMessage("文章内容不能为空"),
]);

exports.getArticle = validate([
  validate.isValidObjectId(["params"], "slug"), // 第一个参数是验证的数据的位置，第二个参数是验证数据字段

  // param('slug').custom(async value => {
  //   if (!mongoose.isValidObjectId(value)) {

  //     return Promise.reject('文章ID类型错误')
  //   }

  // })
]);

exports.updateArticle = [
  validate([
    validate.isValidObjectId(["params"], "slug"), // 第一个参数是验证的数据的位置，第二个参数是验证数据字段
  ]),
  async (req, res, next) => {
    // 校验文章是否存在
    const articleId = req.params.slug;
    const article = await Article.findById(articleId);
    req.article = article; // 把article挂载到req上

    if (!article) {
      // 要修改的文章不存在
      return res.status(404).end();
    }
    next(); // 文章存在，下一个中间件处理
  },
  async (req, res, next) => {
    // 判断文章作者是否是登录用户(禁止修改别人的文章)

    if (req.user._id.toString() !== req.article.author.toString()) {
      // ObjectId是一个对象，不能直接比较
      return res.status(403).end();
    }
    next();
  },
];
```

### 6.3 增加 article 的路由——更新文章

route \ article.js

```js
// 更新文章
router.put(
  "/:slug",
  authorization,
  articleValidate.updateArticle,
  articleController.updateArticle
);
```

### 6.4 处理请求（更新文章）

controller \ articleController.js（部分）

```js
// 更新文章
  async updateArticle(req, res, next) {
    try {
      const article = req.article
      const bodyArticle = req.body.article

      article.title = bodyArticle.title || article.title  // 有修改的则用修改的，否则用原来的
      article.description = bodyArticle.description || article.description
      article.body = bodyArticle.body || article.body

      await article.save()
      res.status(200).json({
        article
      })
    } catch (err) {
      next(err)
    }
  }
```

## 7. 删除文章

### 7.1 数据验证

middle \ validate.js（部分）

```js
exports.deleteArticle = exports.updateArticle;
```

### 7.2 路由

route \ article.js（部分）

```js
// 删除文章
router.delete(
  "/:slug",
  authorization,
  articleValidate.deleteArticle,
  articleController.deleteArticle
);
```

### 7.3 处理请求

controller \ articleController.js（部分）

```js
// 删除文章
async deleteArticle(req, res, next) {
  try {
    const article = req.article
    await article.remove()
    res.status(204).end()
  } catch (err) {
    next(err)
  }
}
```
