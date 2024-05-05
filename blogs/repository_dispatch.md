---
title: b 仓库触发 a 仓库的流水线
categories: 小技能
date: 2024-05-05 19:30:00
tags:
  - Github
---

# b 仓库触发 a 仓库的流水线

通过`repository_dispatch`方式，并且添加`types`参数，b 仓库通过`POST`方式添加参数`event_type`实现。

a 仓库流水线

```yml
name: a
on:
workflow_dispatch: # 手动触发
repository_dispatch: # 其他仓库触发（API 触发）
types: [webhook]
jobs:
webhook:
runs-on: ubuntu-latest
steps: - name: console
run: echo Hello, world!
```

API 触发流水线调用格式：

```yml
curl -X POST https://api.github.com/repos/:owner/:repo/dispatches \
 -H "Accept: application/vnd.github.everest-preview+json" \
 -H "Authorization: token TRIGGER_TOKEN" \
 --data '{"event_type": "TRIGGER_EVENT"}'
```

```yml
`ower`：用户名
`repo`：仓库
`TRIGGER_TOKEN`：token
`TRIGGER_EVENT`：自定义事件（即上面 a 仓库的 repository_dispatch types 里的内容）
```

b 仓库流水线

```yml
name: b
on:
  workflow_dispatch:
  push:
    branches:
      - dev
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: WebHook
        run: |
          curl -X POST https://api.github.com/repos/clzczhc/a/dispatches \
               -H "Accept: application/vnd.github.everest-preview+json" \
               -H "Authorization: token ${{ secrets.TOKEN }}" \
               --data '{"event_type": "webhook"}'
```

## token 生成及配置

> settings -> Developer Settings -> Tokens -> Generate new token

![](https://www.clzczh.top/CLZ_img/images/202405051911014.png)

![](https://www.clzczh.top/CLZ_img/images/202405051911235.png)

复制 token，应该是`ghp_xxx`这种形式。去 b 仓库添加 Secret
![](https://www.clzczh.top/CLZ_img/images/202405051912784.png)

结果：
![](https://www.clzczh.top/CLZ_img/images/202405051912418.png)

参考：
https://cloud.tencent.com/developer/article/1868010

## 通过第三方 action 实现

[repository-dispatch](https://github.com/marketplace/actions/repository-dispatch)

b 仓库新流水线：

```yml
name: b
on:
  workflow_dispatch:
  push:
    branches:
      - dev
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Repository Dispatch
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.TOKEN }}
          repository: clzczhc/a
          event-type: webhook
```

## 传递 b 仓库的分支信息

> 一些特殊场景可能会需要传递仓库触发分支信息，比如 b 仓库推了个分支，想要跑流水线测试，b 仓库触发 a 仓库的流水线的话，默认是不会传递当前分支信息的。

首先是传递信息，通过上面第三方 action 提供的参数`client-payload`传递。

```yml
- name: Repository Dispatch
  uses: peter-evans/repository-dispatch@v3
  with:
    token: ${{ secrets.TOKEN }}
    repository: shimo-open/shimo-doc
    event-type: gen_doc
    client-payload: '{"ref": "${{ github.ref }}"}'
```

a 仓库流水线添加

```yml
export REF=${{ github.event.client_payload.ref }}
[[ ! -n $REF ]] && export REF=dev
echo "REF: ${REF}"
export BRANCH=${REF#refs/heads/}
echo "${BRANCH}"

git checkout ${BRANCH}
```

> `export BRANCH=${REF#refs/heads/}`这部分是因为`GITHUB_REF`是`refs/heads/main`这种结构，利用 shell 命令得到后面的分支命令
> ![](https://www.clzczh.top/CLZ_img/images/202405051928865.png)
