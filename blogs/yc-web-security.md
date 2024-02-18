---
title: Web开发安全
date: 2022-02-06 14:39:44
keywords: 安全
categories: "前端"
tags:
  - 青训营
  - 安全
---

# Web 开发安全

参加字节跳动的青训营时写的笔记。这部分是刘宇晨老师讲的课。

## 1. 攻击

### 1.1 跨站脚本攻击(XSS)

XSS 攻击通常指的是通过利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。

如填写表单信息时，如果盲目相信用户的提交内容，那么假如用户填写了类似` <script>alert("注入恶意代码")</script>`的信息，然后直接通过 element.innerHTML 把用户提交的代码存起来的话，那么工具者就可以实现攻击了。

比较巧妙的攻击方式：

![image-20220127180108501](https://s2.loli.net/2022/02/06/pB7uNcWPvFAMtG9.png)

**XSS 特点**：

- 通常难以从 UI 上发现，因为是在暗地里执行脚本
- 可以窃取用户信息(cookie、token)
- 还可以绘制 UI(如弹窗)，诱骗用户点击

demo：

![image-20220127173357726](https://s2.loli.net/2022/02/06/KBule7Yj8ypvJXN.png)

#### 1.1.1 Stored XSS

把恶意脚本存储在被攻击网站的数据库中。当其他人访问页面时，回去读数据，然后就会执行到数据库中的恶意脚本，从而被攻击。<b style="color: red">危害最大，对全部用户可见</b>

#### 1.1.2 Reflected XSS

- 不涉及数据库
- 从 URL 上进行攻击

![image-20220127174047613](https://s2.loli.net/2022/02/06/P9ufYFds2biRKwS.png)

#### 1.1.3 DOM-based XSS

- 不需要服务器的参与
- 恶意攻击的发起、执行，都在浏览器完成

![image-20220127174625118](https://s2.loli.net/2022/02/06/5agcRdhjMuPqxD1.png)

<b style="color: red">和 Reflected XSS 很像，不过，Reflected XSS 的恶意脚本是注入到服务器中，而 DOM-based XSS 的恶意脚本是注入到浏览器中，而且攻击不需要服务器的参与</b>

#### 1.1.4 Mutation-based XSS

- 利用浏览器渲染 DOM 的特性(独特优化)
- 按浏览器进行攻击

### 2. 跨站伪造请求 CSRF

**CSRF 攻击**：在用户不知情的前提下，利用用户权限(cookie)，构造 HTTP 请求，窃取或修改用户敏感信息。

![image-20220127180459599](https://s2.loli.net/2022/02/06/L35BGtUMJaleTpj.png)

经典例子：银行转账

首先，a 为了转账给 b10000 元，于是 a 登录银行页面进行转账操作, <b style="color: red">还没有推出登录</b>，又收到中奖通知链接(假的或者是只有小额鱼饵)，a 点击链接，并点击领奖按钮。然后发现钱被转走了 100000 元了。

CSRF 攻击原理：利用<b style="Color: red">cookie 的自动携带特性</b>，在其他网站向你的网站发送请求时，如果网站中的用户没有退出登录，而发送的请求又是一些敏感的操作请求(如转账)，则会给用户带来巨大的损失。

### 3. 注入攻击 injection

#### 3.1 SQL 注入

![image-20220127202310672](https://s2.loli.net/2022/02/06/b3rvj1keGdLSUs7.png)

demo

![image-20220127202606829](https://s2.loli.net/2022/02/06/MZgyfvjwKi4d3C7.png)

### 4. Dos

通过某种方式(构造特定请求)，导致服务器资源被显著消耗，来不及响应更多请求，导致请求挤压，进而引起雪崩效应。

### 4.1 ReDoS

例子：上网找到了讲的非常好的文章

[正则表达式所引发的 DoS 攻击(ReDoS)](https://blog.csdn.net/systemino/article/details/89913551)

### 4.2 Distributed DOS(DDoS)

短时间内，来自大量僵尸设备的请求，服务器不能及时响应全部请求，导致请求堆积，进而引发雪崩效应，无法响应新请求。

攻击特点：

- 直接访问 IP
- 任意 API
- 消耗大量带宽(耗尽)

DDoS 攻击 demo：SYN Flood

原理：TCP 的三次握手

![image-20220127205207771](https://s2.loli.net/2022/02/06/a4FQpvyedsrHW3g.png)

如果客户端给服务器发送的 ACK 丢失的话，服务器不知道丢失，会等客户端的 ACK，超时后重新发送 SYN-ACK 消息给客户端，直到重试超过一定次数才会放弃。

而 SYN Flood 就是通过发送大量的 SYN，但是不给服务器发送 ACK，从而耗尽服务器的资源。

![image-20220127205705566](https://s2.loli.net/2022/01/27/wmpI8bKUcugOQnJ.png)

### 5. 中间人攻击

![image-20220127205738271](https://s2.loli.net/2022/01/27/o6qOfQANlgRWmJY.png)

## 2. 防御

### 2.1 XSS

方案：

- 永远不信任用户的提交内容
- 不把用户提交内容直接转换成 DOM

现成工具：

- 前端：
  - 主流框架默认防御 XSS
  - google-closure-library
- 服务端(Node)：
  - DOMPurify

### 2.2 CSRF

#### 2.2.1 同源政策

**浏览器的同源政策**：A 网页设置的 Cookie，B 网页不能打开，除非这两个网页"同源"。所谓"同源"指的是"三个相同"。(出自阮一峰的网络日志)

- 协议相同
- 域名相同
- 端口相同

同源政策的目的，是为了保证用户信息的安全，防止恶意的网站窃取数据。

![image-20220127230913468](https://s2.loli.net/2022/01/27/mJVtw9pxQGkoCz2.png)

#### 2.2.2 CSP

CSP(Content Security Policy)：内容安全策略

- 决定好哪些源(域名)是安全的
- 来自安全源的脚本可以执行，否则直接报错
- 对于 eval / inline script 直接禁止

两种形式：

- 服务器的响应头部：

  ![image-20220127232201451](https://s2.loli.net/2022/01/27/o7XIRZhdkUWPGx9.png)

- 浏览器 meta

  ![image-20220127232216808](https://s2.loli.net/2022/01/27/cLzb9X4HDhJx1yQ.png)

#### 2.2.3 CSRF 防御(token)

![image-20220127232639659](https://s2.loli.net/2022/01/27/1OJQuRLCISD6qig.png)

#### 2.2.4 SameSite Cookie

避免用户信息被携带

下面的部分参考自[Cookie 的 SameSite 属性](https://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)

Cookie 的 SameSite 属性用来限制第三方 Cookie，从而减少安全风险。

可以设置成三个值：

- **Strict**：最严格。完全禁止第三方 Cookie，跨站点时，任何情况都不会发送 Cookie

  ```
  Set-Cookie: CookieName=CookieValue; SameSite=Strict;
  ```

  用户体验不会很好。比如访问别人的项目网站时，有个 fork me 链接到 github，然后点击跳转不会带有 github 的 token，所以跳转过后，都会是未登录状态

- **Lax**：大多数情况不发送第三方 Cookie，但是<b style="color: red">导航到目标地址的 GET 请求</b>会发送。

  ```
  Set-Cookie: CookieName=CookieValue; SameSite=Lax;
  ```

  导航到目标地址的 GET 请求：<b style="color: red">链接、预加载、GET 表单</b>

  ![image-20220127234433512](https://s2.loli.net/2022/01/27/Hl86by42ZdqEUYm.png)

  <b style="color: red">设置了 Strict 或 Lax 之后，基本杜绝了 CSRF 攻击。</b>

- **None**：显示关闭 SameSite 属性。前提是需要同时设置 Secure 属性(Cookie 只能通过 HTTPS 协议发送)，否则无效

  ```
  Set-Cookie: widget_session=abc123; SameSite=None; Secure
  ```

  应用场景是依赖 Cookie 的第三方服务：如网站内嵌其他网站的播放器，开启 SameSite 属性后，就识别不了用户的登录态，也就发不了弹幕了

#### 2.2.5 SameSite 和 CORS 的区别

![image-20220127235222723](https://s2.loli.net/2022/01/27/MBQCtgd3Z4wA187.png)

### 2.3 Injection

![image-20220127235528020](https://s2.loli.net/2022/01/27/SaQpwdnDfh2PzOG.png)

貌似是 Java 中的，我没用过，也不好说。不过查了下资料，prepareStatement 对象防止 sql 注入的方式是**把用户非法输入的单引号用\反斜杠做了转义**，从而达到了防止 sql 注入的目的。

**在 SQL 语句外做的防御**：

![image-20220127235830052](https://s2.loli.net/2022/01/27/8JgET6bfKeABWuy.png)

### 2.4 Dos

#### 2.4.1 ReDoS

- Review 代码
- 代码扫描 + 正则性能测试
- 拒绝使用用户提供的正则

#### 2.4.2 DDoS

![image-20220128000235944](https://s2.loli.net/2022/01/28/aPq9HzvyMGAENFb.png)

### 2.5 防御中间人攻击

<b style="color: red">HTTPS 防止中间人攻击</b>

HTTPS 其实是`SSL+HTTP`的简称,当然现在`SSL`基本已经被`TLS`取代了

![image-20220128000730701](https://s2.loli.net/2022/01/28/BdbPkZ7gx3uJiLV.png)

**HTTPS 的一些特性**：

- 可靠性：加密(非明文)
- 完整性：MAC 验证(防篡改)，通过 hash 算法来实现，所以就算只有很小的改变，hash 输出结果也会变化很大
- 不可抵赖性：数字签名(确定身份)
