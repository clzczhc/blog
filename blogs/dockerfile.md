---
title: dockerfile
categories: 运维
date: 2022-11-01 10:27:34
tags:
    - Docker
    - Dockerfile
---

# 编写Dockerfile文件

Dockerfile作用就是制作镜像，保持开发，测试，生产环境的一致性。

## Dockerfile文件

\color{red}Dockerfile文件没有后缀，和hosts文件类似

```dockerfile
# Version 0.1 

# 基础镜像
FROM ubuntu:latest 

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com 

# 镜像操作命令
RUN apt-get -yqq update && apt-get install -yqq apache2 && apt-get clean 

# 容器启动命令
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Dockerfile一般包含下面几个部分：

- **基础镜像**：以哪个镜像作为基础进行制作。`FROM 基础镜像`

- **维护者信息**：写下改Dockerfile编写人的姓名/邮箱。`MAINTAINER 姓名/邮箱`

- **镜像操作命令**：对基础镜像要进行的改造命令，如安装新的软件，进行哪些特殊配置等。常见的是RUN命令
  
  ```
  RUN命令默认使用/bin/sh Shell执行，默认为root权限。
  如果命令过长需要换行，需要在行末尾加\。
  ```

- **容器启动命令**：当基于改镜像的容器启动时需要执行哪些命令，常见的是CMD命令和ENTRYPOINT命令
  
  ```
  CMD命令也是默认在/bin/sh中执行，并且默认只能有一条，
  如果是多条CMD命令则只有最后一条执行。
  用户也可以在docker run命令创建容器时指定新的CMD命令来覆盖Dockerfile里的CMD
  ```

## 创建镜像

**docker build** 命令用于使用 Dockerfile 创建镜像。

`-t：镜像名以及Tag`

去到Dockerfile所在目录，执行命令`docker build -t shiyanlou:0.1 .`

\color{red}后面有`.`，表示使用当前目录的Dockerfile创建镜像

也可以通过参数`-f`来指定要使用的Dockerfile路径。

## 创建并启动容器

`docker run -p 5000:80 --name web shiyanlou:0.1`

将容器的80端口映射给主机的5000端口，之后就能通过5000端口访问

## Dockerfile编写常用命令

> 做个记录，以后可能会用到

1. 指定容器运行的用户：`USER clzczh`。该用户会作为之后的`RUN`命令执行的用户。

2. 指定后续命令的执行目录：`WORKDIR /var/www/html`。将启动后的工作目录切换到`/var/www/html`目录

3. 对外连接端口号：`EXPOSE 80`。将内部服务的80端口暴露出来，提供给容器间互联使用

4. 设置容器主机名：`ENV HOSTNAME web`。设置由该镜像创建的容器的主机名为web。

5. 向镜像中增加文件
   
   - `COPY test.txt /var/www/html`，将txt文件拷贝到容器里的/var/ww/html目录中。这里的txt文件是相对路径，指的是相对于Dockerfile的路径。如果Dockerfile在/usr/local目录下，则test.txt的绝对路径就是`/usr/local/test.txt`
   
   - `ADD html.tar /var/www`：将tar包添加到容器指定目录，压缩包会被自动解压为目录。

6. CMD和ENTRYPOINT
   
   > ENTRYPOINT 与CMD的区别是不可以被docker run覆盖，会把docker run后面的参数当作传递ENTRYPOINT指令的参数。Dockerfile中只能指定一个ENTRYPOINT，如果指定了很多，只有最后一个有效。docker run命令的-entrypoint参数可以把指定的参数继续传递给ENTRYPOINT。

7. 挂载数据卷：`VOLUME ["/var/log/apche2"]`。
   
   将apche访问的日志数据存储到宿主机可以访问的数据卷中

8. 设置容器内的环境变量：如`ENV APACHE_RUN_DIR /var/run/apache2`
   
   使用`ENV`设置apache启动的环境变量（个人感觉JS里声明变量类似）

### 使用 Supervisord

因为CMD只能执行一条指令，所以就没法实现在一个容器中运行多个服务。这个时候就可以使用`Supervisord`来进行进程管理，方法就是将多个启动命令放入到一个启动脚本中，然后CMD运行该脚本。

安装`Supervisord`：

```dockerfile
RUN apt-get install -yqq supervisor
RUN mkdir -p /var/log/supervisor
```

然后还需要拷贝配置文件到指定的目录。

```dockerfile
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
```

文件内容如下：

```
[supervisord]
nodaemon=true
[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2ctl -D FOREGROUND"
```

如果有多个服务需要启动可以在文件后继续添加[program:xxx]，比如如果有ssh服务，可以增加[program:ssh]。

修改CMD命令，启动Supervisord：

```dockerfile
CMD ["/usr/bin/supervisord"]
```

## 完整版Dockerfile

```dockerfile
# Version 0.2
# 基础镜像
FROM ubuntu:latest
# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com
# 镜像操作命令
RUN apt-get -yqq update && apt-get install -yqq apache2 && apt-get clean
RUN apt-get install -yqq supervisor
RUN mkdir -p /var/log/supervisor

VOLUME ["/var/log/apche2"]
ADD html.tar /var/www
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR /var/www/html
ENV HOSTNAME shiyanloutest
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apche2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apche2

EXPOSE 80
# 容器启动命令
CMD ["/usr/bin/supervisord"]
```

supervisord配置文件

```
[supervisord]
nodaemon=true
[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2ctl -D FOREGROUND"
```

在指定的映像上创建一个容器，然后使用指定的命令启动它。

```docker
docker run -d -p 5000:80 --name web2 web:0.2
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202210201547227.png)
