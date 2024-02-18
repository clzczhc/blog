---
title: git常用命令
date: 2021-11-20 19:38:11
keywords: git
categories: "小技能"
tags:
  - git
---

# git 常用命令

1. git clone 远程仓库链接

如下图

```
git clone git@gitee.com:czh0318/git_test.git
```

![](https://pic.imgdb.cn/item/6198d73f2ab3f51d9187f67d.jpg)

![](https://pic.imgdb.cn/item/6198d8482ab3f51d91885ccf.jpg)

2. 进入项目文件夹，进行操作，比如增加一个文件 test.html

![](https://pic.imgdb.cn/item/6198d8e82ab3f51d91889bf7.jpg)

3. git add .(把修改放到暂存区)

4. git commit -m "提示信息"(提交到本地仓库)

![](https://pic.imgdb.cn/item/6198d9b82ab3f51d9188f150.jpg)

5. git remote 查看远程仓库的别名(如果克隆仓库的话，会有一个 origin)

![](https://pic.imgdb.cn/item/6198da942ab3f51d91894614.jpg)

6. git remote add 添加新的远程仓库

![](https://pic.imgdb.cn/item/6198db3b2ab3f51d9189849c.jpg)

7. git push 提交到远程仓库

语法：git push 远程仓库名 分支(gitee 默认 master, github 默认 main)

![](https://pic.imgdb.cn/item/6198dbb72ab3f51d9189b274.jpg)

![](https://pic.imgdb.cn/item/6198dca32ab3f51d918a2660.jpg)

8. git pull 拉取代码

语法：git pull 远程仓库名 分支名

![](https://pic.imgdb.cn/item/6198dd442ab3f51d918a71e9.jpg)

团队协作部分：请移至[远程仓库小技能](https://13535944743.github.io/2021/10/20/git-skills/)
