---
title: github.io出现的问题及解决方案
date: 2021-07-21 11:09:18
categories: "小技能"
tags:
  - blog
---

# github.io 遇到的问题

## 1. 你的连接不是专用连接

放假回家后打开自己的博客，发现无法打开博客，一开始以为是调样式时不小心搞坏了，打开别人的 githunb.io 博客发现都会出问题，并且用手机不连接 wifi 可以正常打开
解决办法：

### 方法一：

1. 首先调整键盘为英文；
2. 鼠标点击当前页面任意位置，然后输入 thisisunsafe
   输入完成后页面会自动刷新
   注意是直接输入，不要地址栏输入
   这个方法和输入的暗示一样，是方便但不安全的，而且也会定期更改，从 badidea 更改为 thisisunsafe

### 方法二：

1. 进入<a>chrome://net-internals/#hsts</a>页面
2. 点击 Domain Security Policy
3. 在下图区域输入域名，点击"Delete"按钮
   ![](https://img-blog.csdnimg.cn/img_convert/83bcac2264e44f0a7a69b89f7d9ebc3d.png)
4. 在下图区域输入域名，点击"Query"按钮，结果应为"Not found".
   ![](https://img-blog.csdnimg.cn/img_convert/900c6ee399ec5bb42f9cb6f6f67920aa.png)

## 2. 访问 xxx.github.io 被拒绝

原因：国内运营商 DNS 污染，域名指向不正确的 ip 地址
拓展：DNS(Domain Name System) 域名系统
在网络上访问网站，通过 DNS 服务器，把域名转换成 ip 地址
（1）系统会先查找本地的 hosts 文件，确认是否有与域名对应的 ip 地址，若有，则直接访问 ip 地址对应的域名服务器
（2）本地 hosts 文件没有与域名对应的 ip 地址，向已知的 DNS 服务器提出域名解析
[更多](https://www.cnblogs.com/yihr/p/9720715.html)

### 方法 1：

1. 查找 ip 地址，通过<a>http://tool.chinaz.com/dns</a>,复制 ip 地址
   ![](https://img-blog.csdnimg.cn/img_convert/cdb1543eb19b805bb91d38356fd8a713.png)
2. 打开 hosts 文件，路径：C:\Windows\System32\drivers\etc
   按下图示例把 ip 地址和域名输入后保存，需要管理员权限
   ![](https://img-blog.csdnimg.cn/img_convert/04229be8d57810ba531f3072bd6650cb.png)
   ![](https://img-blog.csdnimg.cn/img_convert/a8d3540d896b6427d57061d8d088b2f1.png)
3. 重新访问，正常
   注意：该方法只能使对应的域名可以正常访问，其他没有添加进去 hosts 文件的域名还是会出现问题

### 方法 2：

使用[dev-sidecar 工具](https://gitee.com/docmirror/dev-sidecar)  
功能：

1. dns 优选
2. github 加速
3. npm 加速
   等等
   可以实现解决所有的访问 github.io 被拒绝的问题，非常好用的工具

### 方法 3：

手动修改 DNS，尝试过很多个 DNS 解析服务，只有首选 DNS 服务器设置为 114.114.114.114，备用设置为 208.67.222.222 成功了一个下午，之后莫名打回原形， 使用[DNS jupmper](https://www.sordum.org/7952/dns-jumper-v2-2/)一键设置也没有成功，猜测：DNS 服务器不稳定，未找到解决方案。

参考链接：<a>https://www.itranslater.com/qa/details/2582179179139695616</a>
<a>https://blog.csdn.net/milk_aquarium/article/details/113766559</a>
