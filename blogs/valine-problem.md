---
title: LeanCloud国际版遇到的问题及解决方案
date: 2022-03-07 12:13:09
categories: 小技能
tags:
  - blog
---

# LeanCloud 国际版遇到的问题及解决方案

首先，为什么要用` LeanCloud国际版`呢？就是因为设置邮件提醒功能时，需要绑定访问域名来唤醒 leancloud，而国际版提供免费域名，国内的需要备案域名。

![image-20220307123838756](https://s2.loli.net/2022/03/07/52vX34VkWodfMUj.png)

<br />只能说弄这个博客，真的是非常能感受到迭代的快(虽然遇到的不是技术点上的)：

- 搭建博客时，刚好遇上 github 默认分支从` master`变为` master`，然后网上的教程都还是` master`
- 这次的问题也是因为 us.avoscloud.com 这个域名被弃用了，然而报错提示的确实跨域问题

![img](https://s2.loli.net/2022/03/07/zpYQtBL8f2Fq5ng.png)

这一次属于是长教训了，不看公告，一个月前的事情现在才知道

![image-20220307122010510](https://s2.loli.net/2022/03/07/gLp8f4vniyNwt6D.png)

最后通过到 leancloud 社区直接询问，通过[shifuchen 大佬](https://forum.leancloud.cn/users/shifuchen)的回答解决问题

那么怎么解决这个问题呢？

- 首先，登录[LeanCloud](https://console.leancloud.app/apps)，进入自己的应用。然后进入` 设置 -> 应用凭证`，复制**REST API 服务器地址**

  ![image-20220307122544336](https://s2.loli.net/2022/03/07/jFlm6HG42sVtfwM.png)

  ![image-20220307122650723](https://s2.loli.net/2022/03/07/owuWH5fLhX9T7nm.png)

- 然后，回到你的博客的主题文件夹中，找到使用 valine 部分，[Matery](https://github.com/blinkfox/hexo-theme-matery)主题的就在` layout \ _partial \ valine.ejs`中

  ![image-20220307123020590](https://s2.loli.net/2022/03/07/uaCetFqbYPLDXpf.png)

- 新建`Valine`实例时，添加` serverURLs`属性，值为刚刚复制的地址

  ![image-20220307123226134](https://s2.loli.net/2022/03/07/xJrIPAlsfVc18i5.png)

成功：[contact | 赤蓝紫](https://clz.vercel.app/contact/)

![image-20220307123328264](https://s2.loli.net/2022/03/07/zlyf1ZWcmFYhNOX.png)
