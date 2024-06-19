title: Git 命令汇总
categories: 日志
tags: [Git]
date: 2020-12-25 14:51:00
---
《Pro Git》笔记

## 基础

### 初次运行Git前的配置

设置用户名和邮箱地址

``` shell
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

### 在已存在的目录中初始化仓库

``` shell
git init
```

### 克隆现有仓库

``` shell
git clone https://github.com/libgit2/libgit2
```

``` shell
git clone https://github.com/libgit2/libgit2 mylibgit
```

<!--more-->

### 检查当前仓库状态

``` shell
git status
```

格式更为紧凑的输出：

``` shell
git status -s
git status --short
```

### 跟踪新文件/暂存已修改的文件/合并时将有冲突的文件标记为已解决状态

``` shell
git add filename
```

### 提交更新

``` shell
git commit -m "some text"
```

### 跳过使用暂存区

把所有已经跟踪的文件暂存起来一并提交：

``` shell
git commit -a -m "some text"
```

### 移除文件

``` shell
git rm filename
```

如果要删除的文件已经修改过或放到暂存区：

``` shell
git rm -f filename
```

如果想从Git仓库中删除，但希望在工作目录中保留文件：

``` shell
git rm --cached filename
```

### 重命名文件

``` shell
git mv oldname newname
```

相当于执行了以下三个命令：

``` shell
mv oldname newname
git rm oldname
git add newname
```

### 查看提交历史

``` shell
git log
```

显示每次提交引入的差异：

``` shell
git log -p
git log --patch
```

只显示最近两次提交：

``` shell
git log -p -2
```

显示简略统计信息：

``` shell
git log --stat
```

每个提交放在一行显示：

``` shell
git log --pretty=online
```

### 改善最后一次提交

``` shell
git commit --amend
```

### 取消暂存的文件

使文件由已暂存变为未暂存（已修改）

``` shell
git reset HEAD filename
```

### 撤销对文件的修改

使用仓库中最后一次提交的版本覆盖文件

``` shell
git checkout -- filename
```

## 远程仓库

### 查看远程仓库

``` shell
git remote -v
```

### 添加远程仓库

``` shell
git remote add pb https://github.com/paulboone/ticgit
```

### 从远程仓库中拉取

``` shell
git fetch <remote>
```

拉取所有远程仓库：

``` shell
git fetch --all
```

### 推送到远程仓库

将本地的serverfix分支推送到origin远程仓库的serverfix分支：

``` shell
git push origin serverfix
```

将本地的serverfix分支推送到origin远程仓库的awesomebranch分支：

``` shell
git push origin serverfix:awesomebranch
```

### 查看某个远程仓库

``` shell
git remote show origin
```

### 重命名远程仓库

``` shell
git remote rename pb paul
```

### 移除远程仓库

``` shell
git remote remove paul
```

## 标签

### 列出标签

``` shell
git tag
git tag -l "v1.8.5*"
```

### 创建标签

轻量标签：只是某个特定提交的引用。
附注标签：是存储在Git数据库中的一个完整对象。

附注标签：

``` shell
git tag -a v1.4 -m "my version 1.4"
```

轻量标签：

``` shell
git tag v1.4-lw
```

为过去的提交打标签：

``` shell
git tag -a v1.2 9fceb02
```

### 推送标签

默认情况下`git push`并不会传送标签到远程仓库服务器上，必须显式推送：

``` shell
git push origin --tags
```

### 删除本地标签

``` shell
git tag -d v1.4-lw
```

### 删除远程标签

``` shell
git push origin :refs/tags/<tagname>
```

上面这种操作的含义是，将冒号前面的空值推送到远程标签名，从而高效地删除它。

或采用以下命令：

``` shell
git push origin --delete <tagname>
```

### 检出（checkout）标签

检出标签的同时创建分支version2，否则新的提交将不属于任何分支，并且将无法访问，除非通过确切的提交哈希值才能访问。

``` shell
git checkout -b version2 v2.0.0
```

## 分支

### 创建分支

``` shell
git branch testing
```

### 切换分支

``` shell
git checkout testing
```

### 创建分支并切换

``` shell
git checkout -b iss53
```

等价于一下两个命令：

``` shell
git branch iss53
git checkout iss53
```

### 合并分支

将hotfix分支合并到master分支：

``` shell
git checkout master
git merge hotfix
```

### 查看分支

查看所有分支：

``` shell
git branch
```

查看已合并到当前分支的分支：

``` shell
git branch --merged
```

查看未合并到当前分支的分支：

``` shell
git branch --no-merged
```

### 跟踪分支

如果`git pull`不带参数，将首先fetch当前分支关联的远程分支（跟踪分支），然后将远程分支合并到当前分支。

检出分支时将自动设置关联分支：

``` shell
git checkout -b serverfix origin/serverfix
```

等价于：

``` shell
git checkout --track origin/serverfix
```

设置当前分支的跟踪分支：

``` shell
git branch -u origin/serverfix
```

-u 等价于 --set-upstream-to

查看所有跟踪分支：

``` shell
git branch -vv
```

`git pull` 等价于 `git fetch` + `git merge`，拉取远程分支并合并到当前分支。

### 删除远程分支

从服务器上删除serverfix分支：

``` shell
git push origin --delete serverfix
```