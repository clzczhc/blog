---
title: 让代码能访问外网
categories: 小技能
date: 2023-05-09 14:55:42
tags:
    - 代理
---

# 让代码能访问外网

众所周知，国内是没有办法直接访问外网的，而我们查东西很多时候都会用Google来进行搜索（百度的SEO不太友好，每次查对应内容，官网经常会在挺后面），比如使用v2ray等工具。

但是使用工具的话，代码也没法fq，即使配置了系统代理。

> 这个还是写一个爬虫小工具时，看的一个Python爬虫库的原理，后面慢慢尝试才知道的。顺带一提，Python比较特殊，可以直接访问外网。

## 问题复现（pipe流式数据处理）

首先，这边使用`https`模块访问`www.baidu.com`，看看实际效果。

```js
import https from 'https';
import fs from 'fs';

const endpoint = 'https://www.baidu.com';

https.get(endpoint, (res) => {
  res.pipe(process.stdout);
  res.pipe(fs.createWriteStream('./index.html'));
})
```

> 上面使用`pipe`方法的原因：
> 
> `https`模块的回调函数中的`res`对象是一个可读流，可以通过流式数据的处理方式处理响应体。而`pipe`方法是`Nodejs`中用于流式数据处理的一个方法，\color{red}{将可读流中的数据直接传递给可写流进行处理}。
> 
> `res.pipe(process.stdout)`则是将向响应体直接传递给标准输出流，即直接将响应体输出到命令行窗口中。`process.stdout`是一个可写流，可以将数据输出到终端。

效果：

![](https://cdn.jsdelivr.net/gh/13535944743/CLZ_img@master/images/202304081053283.png)

而将百度换成谷歌就会出现问题：（已经开了系统代理）

![](https://cdn.jsdelivr.net/gh/13535944743/CLZ_img@master/images/202304081055947.png)

## socks-proxy-agent

因为我用的是v2ray来代理的，所以就需要一个工具来实现使用socks代理来发送请求，这个时候就可以使用`socks-proxy-agent`库。

详细内容可以查看官方介绍：[socks-proxy-agent](https://www.npmjs.com/package/socks-proxy-agent)

```js
import https from 'https';
import fs from 'fs';
import url from 'url';
import { SocksProxyAgent } from 'socks-proxy-agent';

const endpoint = 'https://www.google.com';
const opts = url.parse(endpoint);     // 将url字符串转换成 { protocol: 'https:', hostname: 'www.google.com', ... }的对象形式

const proxy = 'socks://127.0.0.1:10808';    // Socks代理连接到连接到电脑上开的v2ray代理
const agent = new SocksProxyAgent(proxy);
opts.agent = agent;

https.get(opts, (res) => {
  res.pipe(process.stdout);
  res.pipe(fs.createWriteStream('./index.html'));
})
```

> 因为我们需要配置代理（在代理里），所以就不能够像前面一样，`https.get`的第一个参数应该是一个`{ protocol: 'https:', hostname: '[www.google.com'](http://www.google.com'), ... }`形式的对象，这样子配置代理只需要给这个对象添加`agent`属性即可，所以用到了`url`模块将url字符串转换。
> 
> 而`socks-proxy-agent`的应用也很简单，引入`SocksProxyAgent`构造函数，并且将电脑上开的代理地址作为参数传给构造函数即可。因为我是本地开的代理，所以ip会是`127.0.0.1`，而端口则可以自己查看。`设置 -> 参数设置`
> 
> ![](https://cdn.jsdelivr.net/gh/13535944743/CLZ_img@master/images/202304081119620.png)

效果：

![](https://cdn.jsdelivr.net/gh/13535944743/CLZ_img@master/images/202304081110692.png)

使用webpack的解决方案：(没测试过，感觉以后用得上，其他框架配置也类似)

[解决webpack-dev-server由于网络问题出现ETIMEDOUT-CSDN博客](https://blog.csdn.net/lwenwei/article/details/127552205)

## fetch版本

```js
import fetch from 'node-fetch';
import { SocksProxyAgent } from 'socks-proxy-agent';
import fs from 'fs';
// 创建代理对象
const proxy = 'socks://127.0.0.1:10808';;
const agent = new SocksProxyAgent(proxy);

// 设置 fetch 请求选项
const url = 'https://www.google.com';
const options = {
  method: 'GET',
  agent: agent
};

// 发起 fetch 请求
fetch(url, options)
  .then(res => {
    res.body.pipe(process.stdout);
    res.body.pipe(fs.createWriteStream('./index.html'));
  })
  .catch(err => console.error(err));
```

> `fetch`版本的`res`对象并不是可读流，`res.body`才是，可以从`stream`中引入`Readable`测试。
> 
> ![](https://cdn.jsdelivr.net/gh/13535944743/CLZ_img@master/images/202304081710966.png)

## Node18内置 fetch

Node18以及之后的版本中，都内置了fetch api。但是，在浏览器环境下，`fetch`是不支持直接设置代理的，因为浏览器限制了跨域请求和安全性问题，不允许通过脚本设置底层网络层信息。而Node18之后内置的`fetch` API也不支持直接设置代理。

> 可以使用`https`模块或者使用`node-fetch`来设置。

浅尝一下内置的`fetch` API。

```js
import { Readable } from 'stream';

fetch('https://www.clzczh.top')
  .then(res => {
    console.log(res instanceof Readable);         // false
    console.log(res.body instanceof Readable);    // false

    return res.text();
  })
  .then(data => {
    console.log(data);
  })
```

![](https://cdn.jsdelivr.net/gh/13535944743/CLZ_img@master/images/202304081734866.png)

> 需要注意：原生`fetch`的`res`和`res.body`都不是可读流，所以需要用`fetch`的`res.text()`那一套来获取响应体。

## 更多

[流 - 权威指南](https://web.dev/i18n/zh/streams/#%E5%B0%86%E5%8F%AF%E8%AF%BB%E6%B5%81%E9%80%9A%E8%BF%87%E7%AE%A1%E9%81%93%E4%BC%A0%E8%BE%93%E5%88%B0%E5%8F%AF%E5%86%99%E6%B5%81)(貌似是浏览器端的，但是感觉挺有用的，记录一下。)

[(4条消息) 解决webpack-dev-server由于网络问题出现ETIMEDOUT-CSDN博客](https://blog.csdn.net/lwenwei/article/details/127552205)
