---
title: Vue项目部署到服务器(ubuntu)
date: 2022-01-02 14:57:38
categories: "前端"
keywords: Vue, 上线
tags:
  - Vue
---

# Vue 项目部署到服务器(ubuntu)

工具：WinSCP、PuTTy(可能不是专业的工具，是本人上操作系统的课用到的软件，直接用来部署了)

1. 打包项目，` npm run build`

   执行` npm run build`命令后，会生成一个 dist 文件夹。

   这一步如果得不到预期的结果，可以把` vue.config.js`文件中的 publicPath 节点变为'./'，如果不存在，则新建文件

   ![image-20220102142328110](https://s2.loli.net/2022/01/02/RVmwxvoJdtfMY7T.png)

2. 把项目文件放到服务器上

   用 WinSCP 登录服务器后，理论上直接把本地的文件直接拖过去，就能复制过去了。但是 ubuntu 没有 root 用户，所以部分文件夹会没有权限。这个时候，就可以采用战略：先复制到不需要权限的地方，然后再通过命令行给命令 mv 添加 sudo 增加权限，把文件夹复制到需要文件的地方。

   ![image-20220102143258862](https://s2.loli.net/2022/01/02/i8jMTw3z2NAalI9.png)

   ![image-20220102143404631](https://s2.loli.net/2022/01/02/lH2tL7NMoyzmhuO.png)

   ![image-20220102143522128](https://s2.loli.net/2022/01/02/WcBbZD2XFH6iRVm.png)

   ![image-20220102143626669](https://s2.loli.net/2022/01/02/CJs3WhY2H7KmraS.png)

3. 安装 nginx, ` sudo apt-get install nginx`

   ![image-20220102143748982](https://s2.loli.net/2022/01/02/pozBZEP7flqDM9L.png)

4. 使用 PuTTy 配置 nginx， 到下图路径中，执行命令` sudo vim default`

   ![image-20220102144203777](https://s2.loli.net/2022/01/02/zmxv8BnIXSQ2uNo.png)

   ![](https://s2.loli.net/2022/01/02/nAPbz2SkHlxyRj5.png)

   <b style="color: red">这里直接在 WinSCP 中执行会出错，可能是因为 WinSCP 原本就只是用来管理传输文件的</b>

   ![image-20220102145110015](https://s2.loli.net/2022/01/02/MLpGQEFuraJUvfn.png)

5. 重启 nginx，` sudo nginx -s reload`, 打开服务器网址，就能看到效果

6. 还有个小问题，如果路由模式为 history 的话，可能会有加载不成功的资源(如图片)，本人因为在考试复习周，所以没有去搞这个配置，而是直接把路由模式改为了哈希模式(虽然有#，丑了点)

**最终效果**：

![image-20220102145654643](https://s2.loli.net/2022/01/02/VguLKHvkPifR7oG.png)
