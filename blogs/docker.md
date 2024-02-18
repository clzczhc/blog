---
title: Docker简单入门
categories: 运维
date: 2022-11-01 10:27:21
tags:
    - Docker
---

# Docker简单入门

## 简介

Docker有两个比较重要的概念**镜像**、**容器**。

- **镜像**：一种文件存储形式。一个磁盘的数据拷贝到另一个磁盘中，另一个磁盘中的这份数据就是镜像。

- **容器**：利用开源的容器引擎，让开发者打包应用以及依赖到一个可移植的镜像中，然后部署到服务器上。

$\color{red}容器是实例化后的镜像$。

**Docker作用**：将应用程序与该程序的依赖，打包在一个文件里，运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。$\color{red}避免环境不统一的问题$。`Docker` 也支持跨平台。

镜像还能有另一个功能：内网依赖提供给外网用。（不确定合不合理，学校和tx的合作项目，tx导师就是这么干的）

这里顺带提一下，开发时一般会弄两条流水线的原因：

> 一条用于推上去测试用，另一条在小组合并分支的时候自动触发（测试用的会减少检测分支命名规则、发送邮件通知评审人两个环节）

Docker只是初步了解，多多包涵。有问题还希望评论提醒。

## Docker常用操作

### 查看命令用法

`docker`：查看Docker的命令用法

`docker COMMAND --help`：查看指定命令详细用法

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210191601619.png)

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210191602309.png)

### 镜像操作

1. 查找镜像：`docker search 关键词`
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210191604594.png)

2. 下载镜像：`docker pull 镜像名:Tag`（Tag表示版本，`latest`，表示最新版本）
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210191641971.png)

3. 查看镜像：`docker images`（查看本地所有镜像）
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210191700047.png)

4. 查看镜像的详细信息：`docker inspect 镜像ID或镜像名:Tag`
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210191711746.png)

5. 删除镜像：`docker rmi -f 镜像ID或镜像名:Tag`（`rmi`的`rm`是删除，而`i`是镜像，因为`docker rm`是删除容器的命令，`-f`表示强制删除）
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210191926682.png)

### 容器操作

1. 创建容器，并运行一个命令：`docker run 镜像ID或镜像名:Tag`
   
   ```js
   // 可加参数
   --name：指定容器名，不指定自动命名
   -i：以交互模式运行容器
   -t：分配一个伪终端
   -p：指定映射端口，将主机端口映射到容器内的端口。-p 5000:80就是将容器的80端口映射给主机的5000端口，之后就能通过5000端口访问
   -d：后台运行容器
   -v：指定挂载主机目录，默认是读写模式`rw`，`ro`表示只读
   ```
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210191953218.png)
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210191955964.png)
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192010518.png)

2. 查看容器列表：`docker ps`（不加参数表示查看正在运行的容器，`-a`表示查看所有容器，`-q`表示只查看容器ID）
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192015638.png)

3. 启动容器：`docker start 容器ID或容器名`
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192153769.png)

4. 停止容器：`docker stop 容器ID或容器名`
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192159516.png)

5. 删除容器：`docker rm -f 容器ID或容器名`)
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192034351.png)

6. 查看日志：`docker logs 容器ID或容器名`
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192205115.png)

7. 进入**正在运行**的容器：`docker exec -it 容器ID或容器名 /bin/bash`
   
   ```js
   /bin/bash是固定写法。
   因为docker后台必须运行一个进程，否则容器就会退出。
   在这里表示启动容器后启动bash。
   ```
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192212284.png)

8. 拷贝文件
   
   主机中文件（文件夹）拷贝到容器中：
   
   ```bash
   docker cp 主机文件路径 容器ID或容器名:容器路径
   ```
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192305751.png)
   
   容器中文件（文件夹）拷贝到主机中：

```bash
docker cp 容器ID或容器名:容器路径 主机文件路径
```

文件：

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192252283.png)

文件夹：

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192258791.png)

9. 获取容器详细信息：`docker inspect 容器ID或容器名`

10. 更新镜像：`docker commit -m="描述信息" -a="作者" 容器ID或容器名 镜像:Tag`
    
    ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210192320850.png)

## 容器和虚拟机区别

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202211011029778.png)

缺点：

  和虚拟机相比，Docker隔离性更弱，属于进程之间的隔离，而虚拟机可以实现系统级别的隔离。

  Docker的安全性也更弱，Docker的租户root和宿主机的root等同，一旦容器内的用户从普通用户权限提升为root权限，它就直接具备了宿主机的root权限，进而可进行无限制的操作
