---
title: GitHub Pages + Hexo - 搭建博客
date: 2021-07-14 20:14:58
categories: "小技能"
tags:
  - hexo
  - blog
---

## 1. 准备

安装[git](http://git-scm.com/)
安装[Node.js](https://nodejs.org/en/)

## 2. GitHub Pages

    1. 注册GitHub
    2. 登录GitHub
    3. 新建仓库

![Snipaste_2021-06-21_12-08-29.png](https://img-blog.csdnimg.cn/img_convert/c12733a9a85d5898fd728ddc77b792bb.png) 4. 配置 SSH-Key
[参考步骤](https://jingyan.baidu.com/article/a65957f4e91ccf24e77f9b11.html)
[扩展：git 配置多个 ssh-key](https://blog.csdn.net/qq_36992688/article/details/102665783)

## 3. 安装 Hexo

    1. 创建文件夹并进入
    2. git Bash
    3. 执行命令

`npm install -g hexo-cli` 4. 初始化框架
`hexo init blog`
blog 是装博客的文件夹 5. 进入 blog 文件夹
`cd blog` 6. 执行
`npm install` 7. 执行
`hexo server` 或者 `hexo s`
成功的话，能在 http://localhost:4000， 能看到下图页面
![blog.png](https://img-blog.csdnimg.cn/img_convert/949b947b54217805f7289c28f243f565.png)

## 4. 关联 GitHub

1. 修改配置文件
   找到本地 blog 文件夹下\_config.yml，打开后滑到最后， 修改成下图样子, 冒号后都要有空格

![1.png](https://img-blog.csdnimg.cn/img_convert/1502530d461a1980f83689f85658853d.png)<br>
<font color = "#dd001b">注意：2020 年 10 月 1 日起，所有"master"分支都改名为"main"，而网上的教程大部分是"master"</font>
题外话：原因：使这家公司摆脱任何提及奴隶制的印象，换成不会有误解的包容性术语 2. blog 目录下执行
`npm install hexo-deployer-git --save` 3. 执行`hexo generate` 和 `hexo deploy`指令或者
`hexo g -d`指令
其中，`hexo generate` 作用是生成静态文件
`hexo deploy` 作用是实现部署到远程站点 4. 发布新帖子
`hexo new "My New Post`
会在 blog\source_posts 下创建新文件 My-New-Post.md 5. 执行`hexo g`
可以在 blog\source_posts\日期文件夹中发现生成了新文件夹 My-New-Post 6. 执行`hexo d`
![image.png](https://img-blog.csdnimg.cn/img_convert/f5c543454c7ee0c674e5334a1cf162ba.png)

## 5. 主题

1. 下载主题[hexo 主题](https://hexo.io/themes/)
2. 安装教程可查看对应主题的安装教程
3. 修改主题文件夹的配置文件（\_config.yml）添加想要的功能和取消不想要的功能
4. 修改样式
   ① 电脑端浏览器打开博客，右键选择检查
   ② 点击下图红框框，记录好想修改的部分的 class 名、id 名，用 vscode 打开 theme 文件夹中的 css 文件夹的 css 文件，ctrl+f 查找 class 名、id 名，修改；
   ![hexo_theme.png](https://img-blog.csdnimg.cn/img_convert/a7f6be8ce7c7197b161716832cde7c7f.png)
   ③ 搜索不到：打开空 css 文件或者直接在原 css 文件最下面添加，修改后可能没有变化，可以在后面添加!important 覆盖原本的样式（这个方法有可能会有问题，暂未遇到）

参考链接：https://blog.csdn.net/guoxiaorui666/article/details/99623023
