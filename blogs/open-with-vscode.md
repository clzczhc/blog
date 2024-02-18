---
title: 右键打开VScode
date: 2022-02-21 21:44:31
categories: "小技能"
tags:
  - VSCode
---

# 右键打开 VScode

## 右键打开文件

1. ` win + R`，输入` regedit`，打开注册表编辑器

2. 在` HKEY_CLASSES_ROOT \ * \ shell`下新建项` VSCode`

3. 在` VSCode`下新建项` command`

   ![image-20220221211953677](https://s2.loli.net/2022/02/21/YMrKETtlfQXkcRs.png)

4. 点击` command`项，双击右边的` (默认)`，数值数据变为 VSCode 的安装路径 + "%1"

   ![image-20220221212426269](https://s2.loli.net/2022/02/21/dvuhkpO2tYjHyJQ.png)

5. 点击` VSCode`项，双击右边的` (默认)`，数值数据变为你想要的提示信息，如` Open with Code`

6. 在` VSCode`下新建` 可扩充字符串值`，命名为` Icon`，更改数值数据为 VSCode 的安装路径

   ![image-20220221213111724](https://s2.loli.net/2022/02/21/BmC2pDZLawGPRn7.png)

## 右键打开文件夹

同理，就是位置不一样而已，在` HKEY_CLASSES_ROOT\Directory\shell`下

![image-20220221213828789](https://s2.loli.net/2022/02/21/hIrp9VEPWwsuizZ.png)

另外，` command`项的数据数值后的参数` "%1"`变为` "%V"`

## 右键文件夹空白处，打开文件夹

同理，就是位置不一样而已，在` HKEY_CLASSES_ROOT\Directory\Background\shell`下

![image-20220221214201643](https://s2.loli.net/2022/02/21/uKi2AJ7Z1moXwHv.png)

另外，` command`项的数据数值后的参数也是` "%V"`

其实，有个快捷方法的，我之前就是跟着网上的教程一次操作，三个都配置好了，只不过我忘记了，也没记下来，又没收藏，真是悔不当初啊。知道的人可以说一下。
