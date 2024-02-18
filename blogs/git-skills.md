---
title: 远程仓库小技能
date: 2021-10-20 14:25:30
categories: "小技能"
tags:
  - git
---

# 远程仓库小技能

## 1. github_dev

这个功能 gitee**好像**没有，突然发现的工具，可以在网页直接编辑仓库文件，而且有 vscode 支持，和直接用 vscode 差不多

1. 进入要修改的仓库，按下键盘.(句号那个键)

   ![](https://pic.imgdb.cn/item/616d9f642ab3f51d911fce14.jpg)

2. 等一小会，进入以下界面

   ![](https://pic.imgdb.cn/item/616d9fcb2ab3f51d91203540.jpg)

3. 直接开始修改代码，这个 网页版的 vscode 会实时保存，所以，当你修改后，在下图红框框中会出现小标，当你手动恢复原状时，小标又会消失

![](https://pic.imgdb.cn/item/616da0362ab3f51d91209c0f.jpg)

4. 点击小标后，按顺序点击下图的 1， 2，和 vscode 类似，比 vscode 简单，相当于没有远程库了，因为你在网页上打开的就是 github 上的库，所以只需要执行 git add . 和 git commit -m "add test3"就行，1 和 2 分别对应这两个步骤

   ![](https://pic.imgdb.cn/item/616da1742ab3f51d9121bc0b.jpg)

5. 回到仓库，会发现已经修改成功

   ![](https://pic.imgdb.cn/item/616da30b2ab3f51d9123332a.jpg)

6. 删除文件也是同理，通过这个工具再也不用为了删除一个无关文件，而把整个库克隆到本地，再修改提交到远程仓库了

## 2. gitee 项目仓库流程

1. 管理员建仓库

2. 邀请成员

   ![](https://pic.imgdb.cn/item/616fab6e2ab3f51d918e5798.jpg)

   ![](https://pic.imgdb.cn/item/616fabb62ab3f51d918e8108.jpg)

3. 设置保护分支，防止项目成员不小心误推

   「保护分支」是 Gitee 针对团队协作中代码权限管理的功能，即为了减小成员误操作带来的损失，对一些关键的分支进行保护，防止被破坏。保护以后，只有仓库的管理员才能对这个分支进行修改、合并等操作。(转自 gitee)

   ![](https://pic.imgdb.cn/item/616fad072ab3f51d918f4c08.jpg)

4. 设置保护分支规则(比如谁可以推之类的)

   ![](https://pic.imgdb.cn/item/616fadb32ab3f51d918fc252.jpg)

   ![](https://pic.imgdb.cn/item/616fad882ab3f51d918fa574.jpg)

5. 既然添加了保护分支规则，那就肯定不是所有项目成员都可以直接 push 到仓库的了，这里就需要先 fork 仓库，再 push 到自己的仓库，然后发起 pull request 请求

   ![](https://pic.imgdb.cn/item/616fb3062ab3f51d91930d9a.jpg)

   ![](https://pic.imgdb.cn/item/616fb36f2ab3f51d9193512d.jpg)

   ![](https://pic.imgdb.cn/item/616fb4612ab3f51d9193d0fb.jpg)

6. 等待管理员合并代码

   tips: git 使用 https 协议，每次 pull,push 都要输入密码，使用 git 协议，使用 ssh 密钥可以省去每次输密码
