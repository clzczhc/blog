---
title: 工作中的那点事之 杂项
date: 2025-04-08 20:48:05
categories: "前端"
tags:
  - 工作中的那点事
---

# 工作中的那点事之 杂项

## shell 语法

```shell
sh -c 'echo "Hello"; echo "world"'
```

> -c 表示是命令，分号将每条命令分隔开。

### 条件判断

语法

```shell
if [条件]; then [命令]; else 命令; fi
```

使用:

```shell
sh -c '''
if [ "$EnvName" = "dev" ];
then echo "dev";
else echo "pro";
fi
'''
```

![](https://www.clzczh.top/CLZ_img/images/202504081954641.png)

## antd pro 多环境变量

> 可以通过`define`的方式提供自定义环境变量，但是在不同环境的值可能不一样，所以需要添加不同环境的配置文件。但是如果都需要上线，但是上线环境有分 dev 和 pro 的话，需要文件名需要是`config.prod.dev.ts`和`config.prod.prod.ts`.

[环境变量](https://pro.ant.design/zh-CN/docs/environment-manage/)

![](https://www.clzczh.top/CLZ_img/images/202504081958121.png)
