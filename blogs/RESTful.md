---
title: RESTful接口设计规范
date: 2022-02-28 10:17:45
categories: "后端"
tags:
	- 接口
---

# RESTful 接口设计规范

- **协议**：API 与用户的通信协议，尽量使用 HTTPS

- **域名**：尽量将 API 部署在专用域名下，如` https://api.example.com`。如果 API 很简单，不会有进一步的扩展，则可以放在主域名下，如` https://example.com/api/`

- **版本**：将 API 的版本号放到 URL 中，如` https://api.example.com/v1/`。或将版本号放在 HTTP 头信息中。

- **路径**：在 RESTful 架构中，每个网址代表一种资源，即网址中不能有动词，只能有名词，而且使用的名词一般和数据库的表格名对应。如` https://api.example.com/v1/users`

- **HTTP 动词**：资源的具体操作类型

  - GET(读取)：从服务器读取资源
  - POST(创建)：在服务器新建资源
  - PUT(完整更新)：在服务器更新资源(客户端提供改变后的完整资源，包括不改变的属性)
  - PATCH(部分更新)：在服务器更新资源(客户端提供改变的属性)
  - DELETE(删除)：从服务器删除资源

  不常用的两个：

  - HEAD：获取资源的元数据(和 GET 类似，只是没有响应体)

  - OPTIONS：获取信息(关于资源的哪些属性是客户端可以改变的

  ![image-20220210134816272](https://s2.loli.net/2022/02/28/IJUrwTfCnqXvF89.png)

- **过滤信息**：如果记录数量很多，API 应该提供参数，过滤返回结果。

  常见参数：

  - ?limit=10：指定返回记录的数量
  - ?offset=10：指定返回记录的开始位置
  - ?page=2$per_page=50：指定第几页，以及每页的记录数
  - ?sortby=name&order=asc：指定返回结果按哪个属性排序，以及排序是升序还是降序
  - ?id=1：指定筛选条件

- **状态码**：客户端的每一次请求，服务器都应该给出响应。响应包括 HTTP 状态码和数据两部分

- **返回结果**：API 返回的数据格式，不应该是纯文本，而应该是 JSON 对象，这样才可以放回标准的格式化数据。服务器回应的 HTTP 头部的` Content-Type`属性也应该设为` application/json`

  ![image-20220210135856605](https://s2.loli.net/2022/02/28/vDzmrRVoL3Wajsf.png)

- **错误处理**：状态码反应发生的错误，具体的错误信息放在数据体中

  ![image-20220210140039430](https://s2.loli.net/2022/02/28/CFru1mcIayXP8Nl.png)

- **身份认证**：例如基于 JWT 的接口权限认证

  - 字段名：` Authorization`
  - 字段值：` Bearer token数据`

- **跨端处理**：在服务端设置 CORS 以允许客户端跨域资源请求
