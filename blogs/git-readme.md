---
title: github添加README文件后导致的问题
date: 2022-03-17 14:09:30
categories: 小技能
tags:
  - git
---

# github 添加 README 文件后导致的问题

github 添加` README.md`文件后，` git push origin main`报错，` git pull origin main`后再推也无济于事。

这是因为` github`处添加` README`文件导致历史不一样。

通过` git pull`指令后添加` --allow-unrelated-histories`选项解决问题

```shell
git pull origin main --allow-unrelated-histories
```

该选项可以合并两个独立启动仓库的历史。
