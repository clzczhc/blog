---
title: git命令——少用但有用
date: 2022-02-19 00:07:20
keywords: git
categories: "小技能"
tags:
  - git
---

# git 命令——少用但有用

实习开始了，新环境、新电脑，git 添加公钥等操作自然也是需要重新设置，为了避免之后还要查找，自己写一下笔记，方便日后使用。

[git 常用命令](https://clz.vercel.app/2021/11/20/git-order/)

<b style="color: red">以下命令在 Git  Bash 中执行</b>

## 1. 设置用户名和 email

```shell
git config --global user.name 用户名
git config --global user.email 邮箱
```

## 2. 查看用户名和 email

```shell
git config user.name
git config user.email

# 也可以把所有信息都列出来，再找
git config --list
```

## 3. 配置 ssh 公私钥

```shell
ssh-keygen -o		# 要输入的话直接Enter
cat ~/.ssh/id_rsa.pub		# 查看并复制公钥(复制是手动复制全部)
```

github 进入`setting -> SSH and GPG keys -> New SSH key`

![image-20220218231701335](https://s2.loli.net/2022/02/18/lbWKsfAohLqZiDz.png)

输入标题(自定义)以及复制的公钥

测试

```shell
ssh -T git@github.com		# 不要改成自己的邮箱，弹出提示的话，输入yes，回车
```

![image-20220218232334720](https://s2.loli.net/2022/02/19/r6j1N7wcPkaO8gx.png)

## 4. 清空暂存区

### 4.1 git rm --cached 文件

```shell
git status		# 查看暂存区文件
git rm --cached 文件		# 一次删除，知道空。效率极低
```

### 4.2 rm .git/index

```shell
rm .git/index	# 暂存区仅仅是.git目录下的一个index文件,所以只要删除这个文件，就清空暂存区了
```

### 4.2 git reset

```shell
git reset	# 后面什么都不跟
```

## 5. 撤销提交

场景：提交完后，发现漏掉文件没有添加，或者提交信息写错了

### 5.1 修改提交信息

现在提交了一次

![image-20220218235143317](https://s2.loli.net/2022/02/19/l5Jr7X3WLiQ9kwE.png)

```shell
git commit --amend
```

进入类似 vim 的页面

- 输入`i`，进入编辑模式
- 移动光标，修改信息
- esc 退出编辑模式
- `:wq`保存

![image-20220218235618176](https://s2.loli.net/2022/02/19/GwLEyRIMX25fJ6H.png)

### 5.2 添加漏掉的文件

- 直接新增文件
- ` git add .`
- ` git commit --amend`(<b style="color: red">不修改，直接保存</b>)
- 提交记录只有一条
