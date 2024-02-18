---
title: Github Actions实现项目的CI/CD
categories: 小技能
date: 2023-04-02 11:20:29
tags:
    - CICD
---

# Github Actions实现项目的CI/CD

## 介绍

> 当我们想要部署一个项目，需要将开发好的代码打包好，然后把打包后的文件传输到服务器上，并且配置好nginx并且启动nginx。每次我们部署都需要不断重复上面所说的步骤，但是，实际上可以通过一些CI/CD工具来帮忙简化这个过程。

`GitHub Actions`是`GitHub`推出的CI/CD服务，它给我们提供了虚拟的服务器资源，让我们可以基于它完成自动化测试、集成、部署等操作。

## 基本概念

- `Workflows（工作流）`：一个工作流就是一个完整的流程

- `Job（任务）`：一个工作流由一个或多个任务组成

- `Step（步骤）`：一个任务会包含一个或多个步骤，步骤会依次被执行

- `Action（动作）`：一个步骤会包含一个或多个动作，比如一些指令（`npm install`）

## Github Pages初尝Github Actions

### Github Pages

首先，使用vite创建一个web项目，修改一下配置文件，避免遇到打包后，出现白屏的问题。

vite.config.js

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  base: './'    // 默认是根路径，修改为相对路径。否则部署github pages时，会去https://xxx.github.io/这个路径下找资源，结果会找不到。
})
```

然后将打包后的dist文件夹的内容作为build分支push到github上，而主分支main则是实际的项目代码。根据build分支开启Github Pages。

![](https://www.clzczh.top/CLZ_img/images/202303282231302.png)

效果：

![](https://www.clzczh.top/CLZ_img/images/202303282233119.png)

因为我有用自己的域名，所以效果稍稍不一样。一般来说，会是github.io形式的域名。

这个时候就能稍微看到Github Actions的风采了，我们点击项目下的Actions选项，就能看到有一个工作流里，这个就是Github Pages的工作流，当每次推送到`build`分支时，就会重新部署。

![](https://www.clzczh.top/CLZ_img/images/202303291539549.png)

![](https://www.clzczh.top/CLZ_img/images/202303291540728.png)

可以看到Github Page工作流有三个Job：`build`、`report-build-status`、`deploy`。点击对应的Job，能看到里面的steps，并且每个step还都可以展开看具体的`Actions`。（当然，并不仅仅是Actions，还有一行执行命令的输出之类的）

![](https://www.clzczh.top/CLZ_img/images/202303291544636.png)

### Github Actions

那么，接下来便是Github Actions的重头戏了。

依次点击下图中红框内容，创建新的工作流。

![](https://www.clzczh.top/CLZ_img/images/202303290002226.png)

workflow 文件采用 YAML 格式，上面的操作会在`/.github/workflows/`下新建一个yml文件，也就是通过该文件来确定工作流的具体内容。

![](https://www.clzczh.top/CLZ_img/images/202303290006850.png)

```yml
name: CI Github Pages
on:
  #监听push操作
  push:
    branches:
      - main # 这里只配置了main分支，所以只有推送main分支才会触发以下任务
jobs:
  # 任务ID
  build-and-deploy:
    # 运行环境
    runs-on: ubuntu-latest
    # 步骤
    steps:
      # 官方action，将代码拉取到虚拟机
      - name: Checkout  ️ 
        uses: actions/checkout@v3

      - name: Install and Build   # 安装依赖、打包，如果提前已打包好无需这一步
        run: |
          npm install
          npm run build

      - name: Deploy   # 部署
        uses: JamesIves/github-pages-deploy-action@v4.3.3
        with:
          branch: build # 部署后提交到那个分支
          folder: dist # 这里填打包好的目录名称
```

简单解释一下上面的一些字段：

- `name`：workflow名称

- `on`：触发woekflow条件，通常是事件，也可以是事件的数组。而上面就是指**只有`main`分支发生`push`事件时，才会触发workflow**

- `jobs`：表示要执行的任务（一个或多个）。每一个任务都应该有一个任务的`job_id`，比如上面的`build-and-deploy`。

- `runs-on`：指定运行时需要用到的虚拟机环境。如上面的`ubuntu-latest`。

- `steps`：指定每个Job的步骤
  
  - `name`：步骤的名称
  
  - `uses`：表示该步骤用的第三方Actions。Github有专门的Actions市场：[GitHub Marketplace · Actions to improve your workflow · GitHub](https://github.com/marketplace?type=actions)
  
  - `run`：该步骤运行的命令。如果有多条命令需要用`&&`连接，或者像上面一样，使用`|`符号。否则，上面的指令就会变成：`npm install npm run build`
    
    ![](https://www.clzczh.top/CLZ_img/images/202303291712342.png)
  
  - `with`：使用第三方Actions时的传参。比如上面的例子中，就是将打包后的`dist`目录的文件，部署到指定分支。所以需要目录字段和分支字段。

添加好工作流后，在本地修改代码，`push`到github，就能看到我们的配置的Actions起作用了。

![](https://www.clzczh.top/CLZ_img/images/202303291721662.png)

![](https://www.clzczh.top/CLZ_img/images/202303291722771.png)

## 部署服务器版本

上面的例子是通过github pages来实现的CICD。但是，我们开发完的项目更多是通过服务器来部署的。下面就来搞一波自动部署服务器。

首先，先配置好nginx，并且将打包后的项目放到服务器上，看看有没有正常部署。

![](https://www.clzczh.top/CLZ_img/images/202303291752459.png)

![](https://www.clzczh.top/CLZ_img/images/202303291758475.png)

正常部署成功了，那些接下来就只需要修改一下workflow就行了。实际上和Github Pages类似，只不过不再是将打包后的文件部署到另一个分支了，而是部署到服务器。

所以，在[Actions市场](https://github.com/marketplace?type=actions)找一波文件传输的Actions。个人最终采用的是`ssh-scp-ssh-pipelines`，可以通过密码登录，也可以通过SSH公钥登录。

接下来按它的例子来写`steps`即可。

```yml
- name: ssh scp ssh pipelines
  uses: cross-the-world/ssh-scp-ssh-pipelines@latest
  env:
    WELCOME: "ssh scp ssh pipelines"
    LASTSSH: "Doing something after copying"
  with:
    host: ${{ secrets.CLZ_HOST }}
    user: ${{ secrets.CLZ_USER }}
    pass: ${{ secrets.CLZ_PASS }}
    port: 22
    connect_timeout: 10
    first_ssh: |
      rm -rf /var/www/html/action_practise/*
    scp: |
      './dist/*' => /var/www/html/action_practise/
```

- `first-ssh`：传输文件前的命令，上面的例子中，则是删除指定目录下的文件/夹

- `scp`：文件传输，`=>`左边是本地目录，右边是服务器目录。上面的例子就是将`dist`文件夹下的文件/夹都传输到指定路径下。

传输文件到服务器，自然就需要ip地址，用户名、密码或者ssh私钥。但是这些内容属于机密，那就不应该直接填写，而是通过`${{ secrets.*** }}`的形式来占位。之后再到仓库的Setting下添加Actions secrets。

![](https://www.clzczh.top/CLZ_img/images/202303291903574.png)

修改完workflow后，修改项目代码，push查看效果。

![](https://www.clzczh.top/CLZ_img/images/202303291906636.png)

> 有可能会因为权限问题导致传输失败，比如用root用户创建的文件夹，但是workflow的用户不是root，那删除文件/夹时可能就会权限报错。可以用对应用户来创建一个文件夹，然后将文件夹移动到需要root权限的地方，这样子，就有权限对移动的文件夹进行操作了。

### 完整workflow

```yml
name: CICD
on:
  #监听push操作
  push:
    branches:
      - main # 这里只配置了main分支，所以只有推送main分支才会触发以下任务
jobs:
  # 任务ID
  build-and-deploy:
    # 运行环境
    runs-on: ubuntu-latest
    # 步骤
    steps:
      # 官方action，将代码拉取到虚拟机
      - name: Checkout  ️ 
        uses: actions/checkout@v3

      - name: Install and Build   # 安装依赖、打包，如果提前已打包好无需这一步
        run: |
          npm install
          npm run build

      - name: ssh scp ssh pipelines
        uses: cross-the-world/ssh-scp-ssh-pipelines@latest
        env:
          WELCOME: "ssh scp ssh pipelines"
          LASTSSH: "Doing something after copying"
        with:
          host: ${{ secrets.CLZ_HOST }}
          user: ${{ secrets.CLZ_USER }}
          pass: ${{ secrets.CLZ_PASS }}
          port: 22
          connect_timeout: 10
          first_ssh: |
            rm -rf /var/www/html/action_practise/*
          scp: |
            './dist/*' => /var/www/html/action_practise/
```

### ssh公私钥对版本

生成ssh密钥：`ssh-keygen -o`。

在`C:\Users\用户名\.ssh`下，找一对以 `id_dsa` 或 `id_rsa` 命名的文件，其中一个带有 `.pub` 扩展名。 `.pub` 文件是你的公钥，另一个则是与之对应的私钥。

将本地生成的公钥`id_rsa.pub`上传到服务器中，路径的话是`/home/用户名/.ssh/`，并且将文件名修改为`authorized_keys`，无后缀。

![](https://www.clzczh.top/CLZ_img/images/202303291928009.png)

可以用电脑ssh远程连接服务器试试看：`ssh -p 22 username@xxx.xxx.xxx.xxx`。

@之前是用户名，之后是服务器IP地址。

![](https://www.clzczh.top/CLZ_img/images/202303291932283.png)

修改参数：

```yml
- name: ssh scp ssh pipelines
  uses: cross-the-world/ssh-scp-ssh-pipelines@latest
  env:
    WELCOME: "ssh scp ssh pipelines"
    LASTSSH: "Doing something after copying"
  with:
    host: ${{ secrets.CLZ_HOST }}
    user: ${{ secrets.CLZ_USER }}
    key: ${{ secrets.CLZ_KEY }}
    port: 22
    connect_timeout: 10
    first_ssh: |
      rm -rf /var/www/html/action_practise/*
    scp: |
      './dist/*' => /var/www/html/action_practise/
```

然后将私钥`id_rsa`的内容（全部），作为CLZ_KEY的值，存出Actions secrets中。

效果：

![](https://www.clzczh.top/CLZ_img/images/202303291946147.png)

> 但是实际感觉和用密码的方式差不多，也不能减少参数个数。

## Express后端部署

Express的部署采用比较简单的方案：直接clone git项目到服务器，然后通过`nodemon app.js`启动项目，直接push代码的时候，触发workflow，将项目传输到服务器，然后因为使用的`nodemon`，所以可以直接更新。

> 但是，上面说的方法有两个大问题：
> 
> 1. 添加新的依赖模块时，不会更新
> 
> 2. 用xshell连接服务器，启动express服务后，如果关掉xshell，服务也会停止

最后采用`pm2`方案来管理node进程，以及修改`workflow`来解决上面的问题。而且node.js 是单进程，报错后后整个服务就寄了，所以需要进程管理工具。（需要使用`npm`全局安装）

简单说一下可能会用到的命令：

- `pm2 start app.js`：启动。
  
  - `--watch`表示以监控的方式启动，app.js文件有变动时，pm2会自动`reload`。
  
  - `--name mynode`：启动一个进程并命名为`mynode`

- `pm2 list`：显示全部进程信息

- `pm2 stop mynode`：停止名字为`mynode`的进程。（实际也可以是id的值）

- `pm2 restart mynode`：重启名字为`mynode`的进程

查看更多：[PM2 命令使用方法总结 - 掘金](https://juejin.cn/post/6889300755539312653)

### workflow

```yml
name: CICD
on:
  #监听push操作
  push:
    branches:
      - main # 这里只配置了main分支，所以只有推送main分支才会触发以下任务
jobs:
  # 任务ID
  build-and-deploy:
    # 运行环境
    runs-on: ubuntu-latest
    # 步骤
    steps:
      # 官方action，将代码拉取到虚拟机
      - name: Checkout  ️ 
        uses: actions/checkout@v3

      - name: ssh scp ssh pipelines
        uses: cross-the-world/ssh-scp-ssh-pipelines@latest
        env:
          WELCOME: "ssh scp ssh pipelines"
          LASTSSH: "Doing something after copying"
        with:
          host: ${{ secrets.CLZ_HOST }}
          user: ${{ secrets.CLZ_USER }}
          pass: ${{ secrets.CLZ_PASS }}
          port: 22
          connect_timeout: 10
          first_ssh: | # 删除进程以及后端文件
            export NVM_DIR=~/.nvm
            source ~/.nvm/nvm.sh
            pm2 delete backend
            rm -rf /home/ubuntu/association_backend/*
          scp: |
            './*' => /home/ubuntu/association_backend/
          last_ssh: |  # 更新服务器上的后端后，安装依赖和开启进程
            export NVM_DIR=~/.nvm
            source ~/.nvm/nvm.sh
            cd /home/ubuntu/association_backend
            npm install
            pm2 start app.js --name backend
```

> 简单讲一下：`first_ssh`是在传输文件前执行的命令，在传输文件前把后端进程以及文件都删除掉（可能文件没必要删，预防万一）。`last_ssh`是在传输文件后执行的命令，包括安装依赖，启动node进程等。

> `first_ssh`和`last_ssh`开头都有两个命令好像是因为我是通过`nvm`来使用node的原因。在一些ssh-action仓库里找到类似的issues。就是上面的解决方案：配置nvm_dir，并更新配置。并且在`first_ssh`和`last_ssh`下还不互通，所以都需要添加那两行命令，添加后才能用node（包括用node全局安装的pm2）

### 小问题

上面的workflow已经能够搞定express项目的CICD了，但是还有一个小瑕疵。如果我们手动删除了进程，再执行CICD的时候就会报错，因为这个时候没有进程能删了。

![](https://www.clzczh.top/CLZ_img/images/202304021111084.png)

而在workflow中出现这种情况，整个流水线就直接寄了（不会执行后续的操作）。

所以需要判断一下有没有要删除的进程存在，有的话，就删，没有就算了。

```shell
PM2_EXIST=$(if pm2 list 2> /dev/null | grep -q appname; then echo "Yes" ; else echo "No" ; fi)

if [ $PM2_EXIST = Yes ] ; then
    pm2 restart appname
    echo "Restart appname."
else
    pm2 start file.js --name appname
    echo "Started appname."
fi
```

在stackoverflow上找到的解决方案：[node.js - pm2 - How to start if not started, kill and start if started - Stack Overflow](https://stackoverflow.com/questions/34734975/pm2-how-to-start-if-not-started-kill-and-start-if-started)

但是workflow中每个命令一行，所以没法像上面一个格式化的很好。

### 完整workflow

```yml
name: CICD
on:
  #监听push操作
  push:
    branches:
      - main # 这里只配置了main分支，所以只有推送main分支才会触发以下任务
jobs:
  # 任务ID
  build-and-deploy:
    # 运行环境
    runs-on: ubuntu-latest
    # 步骤
    steps:
      # 官方action，将代码拉取到虚拟机
      - name: Checkout  ️ 
        uses: actions/checkout@v3

      - name: ssh scp ssh pipelines
        uses: cross-the-world/ssh-scp-ssh-pipelines@latest
        env:
          WELCOME: "ssh scp ssh pipelines"
          LASTSSH: "Doing something after copying"
        with:
          host: ${{ secrets.CLZ_HOST }}
          user: ${{ secrets.CLZ_USER }}
          pass: ${{ secrets.CLZ_PASS }}
          port: 22
          connect_timeout: 10
          first_ssh: | # 删除进程以及后端文件
            export NVM_DIR=~/.nvm
            source ~/.nvm/nvm.sh
            PM2_EXIST=$(if pm2 list 2> /dev/null | grep -q backend; then echo "Yes" ; else echo "No" ; fi)
            if [ $PM2_EXIST = Yes ] ; then pm2 delete backend; echo "delete backend"; else echo "backend not exists"; fi
            rm -rf /home/ubuntu/association_backend/*
          scp: |
            './*' => /home/ubuntu/association_backend/
          last_ssh: |  # 更新服务器上的后端后，安装依赖和开启进程
            export NVM_DIR=~/.nvm
            source ~/.nvm/nvm.sh
            cd /home/ubuntu/association_backend
            npm install
            pm2 start app.js --name backend
            pm2 save
```

> 启动进程那里还稍微改了一下，加了`pm2 save`来保存进程列表，网上的说法是这样子重启pm2（比如重启服务器），就可以通过`pm2 resurrect`来启动所有的node应用程序。

![](https://www.clzczh.top/CLZ_img/images/202304021124309.png)

![](https://www.clzczh.top/CLZ_img/images/202304021125130.png)

# 参考链接（更多内容）

[GitHub Actions 自动部署前端 Vue 项目 - 掘金](https://juejin.cn/post/7189510686760779833)

[GitHub Actions 入门教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

[关于工作流程 - GitHub 文档](https://docs.github.com/zh/actions/using-workflows/about-workflows)

之后玩：

> [Github Actions + 腾讯云函数实现微信推送天气、课表，上课提醒，每日晚安心语_github微信推送_无奈清风吹过的博客-CSDN博客](https://blog.csdn.net/qq_29711355/article/details/126808897)
> 
> [玩转 GitHub Actions - 掘金](https://juejin.cn/post/7134990079407161357)

`GitHub Actions`是`GitHub`推出的CI/CD服务，它给我们提供了虚拟的服务器资源，让我们可以基于它完成自动化测试、集成、部署等操作。

## 基本概念

- `Workflows（工作流）`：一个工作流就是一个完整的流程

- `Job（任务）`：一个工作流由一个或多个任务组成

- `Step（步骤）`：一个任务会包含一个或多个步骤，步骤会依次被执行

- `Action（动作）`：一个步骤会包含一个或多个动作，比如一些指令（`npm install`）

## Github Pages初尝Github Actions

### Github Pages

首先，使用vite创建一个web项目，修改一下配置文件，避免遇到打包后，出现白屏的问题。

vite.config.js

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  base: './'    // 默认是根路径，修改为相对路径。否则部署github pages时，会去https://xxx.github.io/这个路径下找资源，结果会找不到。
})
```

然后将打包后的dist文件夹的内容作为build分支push到github上，而主分支main则是实际的项目代码。根据build分支开启Github Pages。

![](https://www.clzczh.top/CLZ_img/images/202303282231302.png)

效果：

![](https://www.clzczh.top/CLZ_img/images/202303282233119.png)

因为我有用自己的域名，所以效果稍稍不一样。一般来说，会是github.io形式的域名。

这个时候就能稍微看到Github Actions的风采了，我们点击项目下的Actions选项，就能看到有一个工作流里，这个就是Github Pages的工作流，当每次推送到`build`分支时，就会重新部署。

![](https://www.clzczh.top/CLZ_img/images/202303291539549.png)

![](https://www.clzczh.top/CLZ_img/images/202303291540728.png)

可以看到Github Page工作流有三个Job：`build`、`report-build-status`、`deploy`。点击对应的Job，能看到里面的steps，并且每个step还都可以展开看具体的`Actions`。（当然，并不仅仅是Actions，还有一行执行命令的输出之类的）

![](https://www.clzczh.top/CLZ_img/images/202303291544636.png)

### Github Actions

那么，接下来便是Github Actions的重头戏了。

依次点击下图中红框内容，创建新的工作流。

![](https://www.clzczh.top/CLZ_img/images/202303290002226.png)

workflow 文件采用 YAML 格式，上面的操作会在`/.github/workflows/`下新建一个yml文件，也就是通过该文件来确定工作流的具体内容。

![](https://www.clzczh.top/CLZ_img/images/202303290006850.png)

```yml
name: CI Github Pages
on:
  #监听push操作
  push:
    branches:
      - main # 这里只配置了main分支，所以只有推送main分支才会触发以下任务
jobs:
  # 任务ID
  build-and-deploy:
    # 运行环境
    runs-on: ubuntu-latest
    # 步骤
    steps:
      # 官方action，将代码拉取到虚拟机
      - name: Checkout  ️ 
        uses: actions/checkout@v3

      - name: Install and Build   # 安装依赖、打包，如果提前已打包好无需这一步
        run: |
          npm install
          npm run build

      - name: Deploy   # 部署
        uses: JamesIves/github-pages-deploy-action@v4.3.3
        with:
          branch: build # 部署后提交到那个分支
          folder: dist # 这里填打包好的目录名称
```

简单解释一下上面的一些字段：

- `name`：workflow名称

- `on`：触发woekflow条件，通常是事件，也可以是事件的数组。而上面就是指**只有`main`分支发生`push`事件时，才会触发workflow**

- `jobs`：表示要执行的任务（一个或多个）。每一个任务都应该有一个任务的`job_id`，比如上面的`build-and-deploy`。

- `runs-on`：指定运行时需要用到的虚拟机环境。如上面的`ubuntu-latest`。

- `steps`：指定每个Job的步骤
  
  - `name`：步骤的名称
  
  - `uses`：表示该步骤用的第三方Actions。Github有专门的Actions市场：[GitHub Marketplace · Actions to improve your workflow · GitHub](https://github.com/marketplace?type=actions)
  
  - `run`：该步骤运行的命令。如果有多条命令需要用`&&`连接，或者像上面一样，使用`|`符号。否则，上面的指令就会变成：`npm install npm run build`
    
    ![](https://www.clzczh.top/CLZ_img/images/202303291712342.png)
  
  - `with`：使用第三方Actions时的传参。比如上面的例子中，就是将打包后的`dist`目录的文件，部署到指定分支。所以需要目录字段和分支字段。

添加好工作流后，在本地修改代码，`push`到github，就能看到我们的配置的Actions起作用了。

![](https://www.clzczh.top/CLZ_img/images/202303291721662.png)

![](https://www.clzczh.top/CLZ_img/images/202303291722771.png)

## 部署服务器版本

上面的例子是通过github pages来实现的CICD。但是，我们开发完的项目更多是通过服务器来部署的。下面就来搞一波自动部署服务器。

首先，先配置好nginx，并且将打包后的项目放到服务器上，看看有没有正常部署。

![](https://www.clzczh.top/CLZ_img/images/202303291752459.png)

![](https://www.clzczh.top/CLZ_img/images/202303291758475.png)

正常部署成功了，那些接下来就只需要修改一下workflow就行了。实际上和Github Pages类似，只不过不再是将打包后的文件部署到另一个分支了，而是部署到服务器。

所以，在[Actions市场](https://github.com/marketplace?type=actions)找一波文件传输的Actions。个人最终采用的是`ssh-scp-ssh-pipelines`，可以通过密码登录，也可以通过SSH公钥登录。

接下来按它的例子来写`steps`即可。

```yml
- name: ssh scp ssh pipelines
  uses: cross-the-world/ssh-scp-ssh-pipelines@latest
  env:
    WELCOME: "ssh scp ssh pipelines"
    LASTSSH: "Doing something after copying"
  with:
    host: ${{ secrets.CLZ_HOST }}
    user: ${{ secrets.CLZ_USER }}
    pass: ${{ secrets.CLZ_PASS }}
    port: 22
    connect_timeout: 10
    first_ssh: |
      rm -rf /var/www/html/action_practise/*
    scp: |
      './dist/*' => /var/www/html/action_practise/
```

- `first-ssh`：传输文件前的命令，上面的例子中，则是删除指定目录下的文件/夹

- `scp`：文件传输，`=>`左边是本地目录，右边是服务器目录。上面的例子就是将`dist`文件夹下的文件/夹都传输到指定路径下。

传输文件到服务器，自然就需要ip地址，用户名、密码或者ssh私钥。但是这些内容属于机密，那就不应该直接填写，而是通过`${{ secrets.*** }}`的形式来占位。之后再到仓库的Setting下添加Actions secrets。

![](https://www.clzczh.top/CLZ_img/images/202303291903574.png)

修改完workflow后，修改项目代码，push查看效果。

![](https://www.clzczh.top/CLZ_img/images/202303291906636.png)

> 有可能会因为权限问题导致传输失败，比如用root用户创建的文件夹，但是workflow的用户不是root，那删除文件/夹时可能就会权限报错。可以用对应用户来创建一个文件夹，然后将文件夹移动到需要root权限的地方，这样子，就有权限对移动的文件夹进行操作了。

### 完整workflow

```yml
name: CICD
on:
  #监听push操作
  push:
    branches:
      - main # 这里只配置了main分支，所以只有推送main分支才会触发以下任务
jobs:
  # 任务ID
  build-and-deploy:
    # 运行环境
    runs-on: ubuntu-latest
    # 步骤
    steps:
      # 官方action，将代码拉取到虚拟机
      - name: Checkout  ️ 
        uses: actions/checkout@v3

      - name: Install and Build   # 安装依赖、打包，如果提前已打包好无需这一步
        run: |
          npm install
          npm run build

      - name: ssh scp ssh pipelines
        uses: cross-the-world/ssh-scp-ssh-pipelines@latest
        env:
          WELCOME: "ssh scp ssh pipelines"
          LASTSSH: "Doing something after copying"
        with:
          host: ${{ secrets.CLZ_HOST }}
          user: ${{ secrets.CLZ_USER }}
          pass: ${{ secrets.CLZ_PASS }}
          port: 22
          connect_timeout: 10
          first_ssh: |
            rm -rf /var/www/html/action_practise/*
          scp: |
            './dist/*' => /var/www/html/action_practise/
```

### ssh公私钥对版本

生成ssh密钥：`ssh-keygen -o`。

在`C:\Users\用户名\.ssh`下，找一对以 `id_dsa` 或 `id_rsa` 命名的文件，其中一个带有 `.pub` 扩展名。 `.pub` 文件是你的公钥，另一个则是与之对应的私钥。

将本地生成的公钥`id_rsa.pub`上传到服务器中，路径的话是`/home/用户名/.ssh/`，并且将文件名修改为`authorized_keys`，无后缀。

![](https://www.clzczh.top/CLZ_img/images/202303291928009.png)

可以用电脑ssh远程连接服务器试试看：`ssh -p 22 username@xxx.xxx.xxx.xxx`。

@之前是用户名，之后是服务器IP地址。

![](https://www.clzczh.top/CLZ_img/images/202303291932283.png)

修改参数：

```yml
- name: ssh scp ssh pipelines
  uses: cross-the-world/ssh-scp-ssh-pipelines@latest
  env:
    WELCOME: "ssh scp ssh pipelines"
    LASTSSH: "Doing something after copying"
  with:
    host: ${{ secrets.CLZ_HOST }}
    user: ${{ secrets.CLZ_USER }}
    key: ${{ secrets.CLZ_KEY }}
    port: 22
    connect_timeout: 10
    first_ssh: |
      rm -rf /var/www/html/action_practise/*
    scp: |
      './dist/*' => /var/www/html/action_practise/
```

然后将私钥`id_rsa`的内容（全部），作为CLZ_KEY的值，存出Actions secrets中。

效果：

![](https://www.clzczh.top/CLZ_img/images/202303291946147.png)

> 但是实际感觉和用密码的方式差不多，也不能减少参数个数。

## Express后端部署

Express的部署采用比较简单的方案：直接clone git项目到服务器，然后通过`nodemon app.js`启动项目，直接push代码的时候，触发workflow，将项目传输到服务器，然后因为使用的`nodemon`，所以可以直接更新。

> 但是，上面说的方法有两个大问题：
> 
> 1. 添加新的依赖模块时，不会更新
> 
> 2. 用xshell连接服务器，启动express服务后，如果关掉xshell，服务也会停止

最后采用`pm2`方案来管理node进程，以及修改`workflow`来解决上面的问题。而且node.js 是单进程，报错后后整个服务就寄了，所以需要进程管理工具。（需要使用`npm`全局安装）

简单说一下可能会用到的命令：

- `pm2 start app.js`：启动。
  
  - `--watch`表示以监控的方式启动，app.js文件有变动时，pm2会自动`reload`。
  
  - `--name mynode`：启动一个进程并命名为`mynode`

- `pm2 list`：显示全部进程信息

- `pm2 stop mynode`：停止名字为`mynode`的进程。（实际也可以是id的值）

- `pm2 restart mynode`：重启名字为`mynode`的进程

查看更多：[PM2 命令使用方法总结 - 掘金](https://juejin.cn/post/6889300755539312653)

### workflow

```yml
name: CICD
on:
  #监听push操作
  push:
    branches:
      - main # 这里只配置了main分支，所以只有推送main分支才会触发以下任务
jobs:
  # 任务ID
  build-and-deploy:
    # 运行环境
    runs-on: ubuntu-latest
    # 步骤
    steps:
      # 官方action，将代码拉取到虚拟机
      - name: Checkout  ️ 
        uses: actions/checkout@v3

      - name: ssh scp ssh pipelines
        uses: cross-the-world/ssh-scp-ssh-pipelines@latest
        env:
          WELCOME: "ssh scp ssh pipelines"
          LASTSSH: "Doing something after copying"
        with:
          host: ${{ secrets.CLZ_HOST }}
          user: ${{ secrets.CLZ_USER }}
          pass: ${{ secrets.CLZ_PASS }}
          port: 22
          connect_timeout: 10
          first_ssh: | # 删除进程以及后端文件
            export NVM_DIR=~/.nvm
            source ~/.nvm/nvm.sh
            pm2 delete backend
            rm -rf /home/ubuntu/association_backend/*
          scp: |
            './*' => /home/ubuntu/association_backend/
          last_ssh: |  # 更新服务器上的后端后，安装依赖和开启进程
            export NVM_DIR=~/.nvm
            source ~/.nvm/nvm.sh
            cd /home/ubuntu/association_backend
            npm install
            pm2 start app.js --name backend
```

> 简单讲一下：`first_ssh`是在传输文件前执行的命令，在传输文件前把后端进程以及文件都删除掉（可能文件没必要删，预防万一）。`last_ssh`是在传输文件后执行的命令，包括安装依赖，启动node进程等。

> `first_ssh`和`last_ssh`开头都有两个命令好像是因为我是通过`nvm`来使用node的原因。在一些ssh-action仓库里找到类似的issues。就是上面的解决方案：配置nvm_dir，并更新配置。并且在`first_ssh`和`last_ssh`下还不互通，所以都需要添加那两行命令，添加后才能用node（包括用node全局安装的pm2）

### 小问题

上面的workflow已经能够搞定express项目的CICD了，但是还有一个小瑕疵。如果我们手动删除了进程，再执行CICD的时候就会报错，因为这个时候没有进程能删了。

![](https://www.clzczh.top/CLZ_img/images/202304021111084.png)

而在workflow中出现这种情况，整个流水线就直接寄了（不会执行后续的操作）。

所以需要判断一下有没有要删除的进程存在，有的话，就删，没有就算了。

```shell
PM2_EXIST=$(if pm2 list 2> /dev/null | grep -q appname; then echo "Yes" ; else echo "No" ; fi)

if [ $PM2_EXIST = Yes ] ; then
    pm2 restart appname
    echo "Restart appname."
else
    pm2 start file.js --name appname
    echo "Started appname."
fi
```

在stackoverflow上找到的解决方案：[node.js - pm2 - How to start if not started, kill and start if started - Stack Overflow](https://stackoverflow.com/questions/34734975/pm2-how-to-start-if-not-started-kill-and-start-if-started)

但是workflow中每个命令一行，所以没法像上面一个格式化的很好。

### 完整workflow

```yml
name: CICD
on:
  #监听push操作
  push:
    branches:
      - main # 这里只配置了main分支，所以只有推送main分支才会触发以下任务
jobs:
  # 任务ID
  build-and-deploy:
    # 运行环境
    runs-on: ubuntu-latest
    # 步骤
    steps:
      # 官方action，将代码拉取到虚拟机
      - name: Checkout  ️ 
        uses: actions/checkout@v3

      - name: ssh scp ssh pipelines
        uses: cross-the-world/ssh-scp-ssh-pipelines@latest
        env:
          WELCOME: "ssh scp ssh pipelines"
          LASTSSH: "Doing something after copying"
        with:
          host: ${{ secrets.CLZ_HOST }}
          user: ${{ secrets.CLZ_USER }}
          pass: ${{ secrets.CLZ_PASS }}
          port: 22
          connect_timeout: 10
          first_ssh: | # 删除进程以及后端文件
            export NVM_DIR=~/.nvm
            source ~/.nvm/nvm.sh
            PM2_EXIST=$(if pm2 list 2> /dev/null | grep -q backend; then echo "Yes" ; else echo "No" ; fi)
            if [ $PM2_EXIST = Yes ] ; then pm2 delete backend; echo "delete backend"; else echo "backend not exists"; fi
            rm -rf /home/ubuntu/association_backend/*
          scp: |
            './*' => /home/ubuntu/association_backend/
          last_ssh: |  # 更新服务器上的后端后，安装依赖和开启进程
            export NVM_DIR=~/.nvm
            source ~/.nvm/nvm.sh
            cd /home/ubuntu/association_backend
            npm install
            pm2 start app.js --name backend
            pm2 save
```

> 启动进程那里还稍微改了一下，加了`pm2 save`来保存进程列表，网上的说法是这样子重启pm2（比如重启服务器），就可以通过`pm2 resurrect`来启动所有的node应用程序。

# 参考链接（更多内容）

[GitHub Actions 自动部署前端 Vue 项目 - 掘金](https://juejin.cn/post/7189510686760779833)

[GitHub Actions 入门教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

[关于工作流程 - GitHub 文档](https://docs.github.com/zh/actions/using-workflows/about-workflows)

[玩转 GitHub Actions - 掘金](https://juejin.cn/post/7134990079407161357)
