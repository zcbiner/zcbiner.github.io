---
title: Git实践宝典
date: 2018-08-15 20:24:30
tags: other


---

总结平时使用Git的过程中，好用但是经常忘记的命令；记录使用Git中遇到的问题及解决方案。

<!--more-->

ps：以下命令中，`()`内的指需要根据实际替换的内容，比如分支名。

### 关于分支

本地分支与远程分支进行关联：

```shell
git branch --set-upstream-to=origin/(branchName)
```

推送本地新建分支到远端分支：

```shell
# 推送时指定分支名
git push origin (localBranch):(originBranch)
# 直接将本地分支推送到远端同名分支，远端没有分支会新建，有则会将本地修改同步
git push origin HEAD
# 同上，加了--force表示强制将本地修改同步，适合于本地执行了amend操作的情况下
git push origin HEAD --force
```

删除远端分支：

```shell
git push origin --delete (branchName)
```

查看本地分支关联的远程分支：

```shell
git branch -vv
```

分支重命名：

```shell
git branch -m (oldName) (newName)
```

删除本地版本库上那些失效的远程追踪分支:

```shell
git remote prune origin --dry-run
```

### 工作区、暂存区之间的切换

有的时候习惯性``git add .``将所有的文件都提交到了暂存区，可是有部分文件并不想提交。怎么办呢？这里总结下`git`各个区之间的切换。

1.在工作区放弃修改

当你修改了文件，但发现改错了，可使用以下命令：

```shell
git checkout -- filepathname //放弃修改某个文件，例如： git checkout -- readme.md
git checkout .  //放弃所有修改的文件
git restore . //放弃所有修改的文件
```

2.在暂存区放弃修改

当你想要提交时，发现不想提交的文件都在暂存区，那么可以使用如下命令：

```shell
git reset HEAD filepathname //恢复某个文件到工作区，例如： git reset HEAD readme.md
git reset HEAD . //恢复所有文件到工作区
git reset  //恢复所有文件到工作区
```

注意：这里只是恢复到了工作区，如果想放弃修改的代码还需要执行步骤1（工作区）中的操作

### HEAD~和HEAD^

`~`和`^` 让我傻傻分不清楚，直到我看到了这个：[What's the difference between HEAD^ and HEAD~ in Git?](https://stackoverflow.com/questions/2221658/whats-the-difference-between-head-and-head-in-git)

简单的说：

1. `~`是在当前分支上回溯，后面的数字表示回溯几步。
2. 如果分支没有merge发生，`~`和`^`是一样的效果。但如果发生了merge，那么`^`表示是哪条merge过来的分支的第一个提交。

从下面的图示可以更好的理解：

```
G   H   I   J
 \ /     \ /
  D   E   F
   \  |  / \
    \ | /   |
     \|/    |
      B     C
       \   /
        \ /
         A
A =      = A^0
B = A^   = A^1     = A~1
C = A^2
D = A^^  = A^1^1   = A~2
E = B^2  = A^^2
F = B^3  = A^^3
G = A^^^ = A^1^1^1 = A~3
H = D^2  = B^^2    = A^^^2  = A~2^2
I = F^   = B^3^    = A^^3^
J = F^2  = B^3^2   = A^^3^2
```

另外有一个命令，可以让git显示详细的merge、多分支情况：

```shell
git log --oneline --graph --all
```

### 使用Worktree

项目工程很庞大，如果我重新`clone`一份代码都要半个多小时以上。而经常会有在本地做开发或者调试时，突然要看另外一个较紧急的问题，此时就想要多开一个项目工程。`worktree`可以很好的解决这个问题。

git worktree非常适合大型项目又需要维护多个分支，想要避免来回切换的场景。其有以下几个优点：

- 单独 clone 项目相比，节省了硬盘空间，又因为 git worktree 使用 hard link 实现，要远远快于 clone
- 提交可以在同一个项目中共享
- 可以快速进行并行开发，同一个项目多个分支同时并行演进

添加一个`worktree`：

```shell
# 添加时需要指定一个目录，这个目录用于存放备份的工程，目录必须是空的。需要指定切换到worktree的分支名。
git worktree add (path) (branchName)
# 一次实际案例：在backUp目录切换到远程master分支。后续可以直接用ide打开backUp目录工程进行开发
git worktree add ../backUp origin/master
```

查看当前的worktree：

```shell
git worktree list
```

删除一个worktree：

```shell
git worktree remove (path)
```

### Fork仓库同步原仓库代码

> 有的时候从原仓库fork出了一个新仓库，这个新仓库做了自己的修改。可是原仓库呢也进行了更新，比如修复了bug，增加了新特性之类的。这个时候想要把原仓库代码同步过来。

#### 原理

把原仓库的代码拉到本地，然后通过git merge把原仓库分支代码合到自己的分支代码。

#### 执行

**1.先拉取原仓库代码到本地**

```shell
git remote add upstream (填写你仓库git地址)
git fetch upstream
```

这样就把原仓库代码拉到本地了，而且upstream跟原仓库进行了绑定。（类似origin的绑定，upstream也可以命名为其它的名称）

此时执行`git remote`，会看到有两个：

```
origin
remote
```

**2.合并分支**

切换到fork仓库的分支，可以新建一个合并分支。然后执行命令：

```shell
git merge upstream/master
```

分支名不一定得是master，也可以是其它的原仓库分支。

如果有冲突就要解决冲突，解决完冲突后，执行下命令即可：

```shell
git merge --continue
```

最后，如果完成了，可以执行命令移除这个绑定。

```shell
git remote remove upstream
```

### 遇到的问题

#### 1.Encountered 1 file(s) that should have been pointers, but weren't

git lfs的问题，通过以下两个命令修复，参考：https://github.com/git-lfs/git-lfs/issues/1939 。

```shell
git rm .gitattributes
git reset --hard HEAD
```

#### 2.error: cannot lock ref 'refs/remotes/origin

直接执行`git pull`时发现的问题，导致无法更新代码。有的解决方法是直接删除项目`.git`下`refs/remotes/origin`整个目录，试过确实是ok的，但这样还是比较复杂的，每次都需要去找相关目录。找到一个命令可以修复这种情况，暂时还不知道导致该问题的原因。

```shell
git remote prune origin
```

上面的命令，每次`pull`完想要重新`pull`时，还是会出问题，也就是说要多次执行。

还可以执行以下命令，该命令比较耗时:

```shell
git gc --prune=now
```

可参考：[Git and nasty “error: cannot lock existing info/refs fatal”](https://stackoverflow.com/questions/6656619/git-and-nasty-error-cannot-lock-existing-info-refs-fatal)
