# 工作中的那点事之 Git

## git reset --soft

效果和`git reset --hard`类似，区别是`--soft`会把所有的 commit 都保留。所以也可以通过`--soft`来实现将多个 commit 合成一个。

## git stash apply

`git stash`可以将未提交的修改保存起来，通过`git stash apply`或`git stash pop`将保存的东西释放。

> `git stash apply`和`git stash pop`区别：`apply`不会删除 stash 记录，并且还可以通过`git stash list` + `git stash apply @x`来应用指定的记录

> 在需要短暂切分支修复紧急 bug 的时候，可以使用到。

**使用`git stash pop`风险**：会释放保存的内容，并同时清除当前应用的 stash 记录。但是不小心 pop 了多次的话，没有办法撤销回退。而`apply`还保留记录，所以还可以撤销回退。

## git commit --no-verify

类似于上面的场景：在需要切分支修复紧急 bug 的时候，但是可能并不是短时间可以修复。

> 这个时候使用`git stash`的话，有可能会出现很多分支都`stash`一下，最后找起来比较麻烦。

这个时候可以使用`git commit --no-verify`忽略限制，强行提交。比如`git commit -m "暂存" --no-verify`。

解决问题后，再回到暂存 commit 的分支，使用`git reset --soft HEAD~`把东西又吐出来。

## git revert

撤销修改。跟`git reset --hard`的区别是，它的撤销是新增加一个 commit，把修改的内容自动撤销。所以不会需要`git push --force`，不容易出意外。

![](https://www.clzczh.top/CLZ_img/images/202503262236591.png)
场景：当前有两个 commit，第一个是添加`a.txt`文件，第二个是添加`b.txt`，并且修改`a.txt`为`c.txt`。（文件内容为文件名。

现在想要撤销第二个 commit，只需要使用`git revert [第二个commitID]`即可。
![](https://www.clzczh.top/CLZ_img/images/202503262239846.png)

## git rebase -i

在团队开发中，经常会使用到`git rebase`来更新分支。

`git rebase -i`可以实现一些 commit 交换位置，删除指定 commit 等功能。

> 比如 `git rebase -i HEAD~3`对最近三个 commit 进行处理，然后点击 i，对 commit 进行处理（删除、修改位置等）。修改后，点击`ESC`退出编辑，输入`:wq`保存修改
> ![](https://www.clzczh.top/CLZ_img/images/20250311213851.png)

上面的方法并不能修改 commit 信息。需要将要修改的 commit 的`pick`改为`edit`,`:wq`保存退出。通过`git commit --amend`修改 commit 信息，修改后再执行`git rebase --continue`，知道修改完成。

## git diff 显示差异

> git diff HEAD^ --unified=0 -- ./c.txt | grep '^+[^+]' > diff.txt
> 上面的代码，可以比较当前提交 (HEAD) 与上一个提交 (HEAD^) 的差异，并将`c.txt`中新增的内容输出到`diff.txt`中。每个参数的具体含义有兴趣的话，可以自行了解。需要修改的话，可以借助强大的 AI。

应用场景：获取新增的国际化字段。
具体场景：项目迁移的时候，需要把迁移后的项目里面新增的国际化字段提取出来，再根据以前项目的国际化资源，把已有的国际化内容迁移到新项目。
