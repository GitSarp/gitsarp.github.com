---
title: git基础——持续更新
date: 2018-08-29 23:17:48
tags: git
categories: Linux
---
###  git基础——持续更新
[文档](https://git-scm.com/docs)

[中文文档](https://git-scm.com/book/zh/v2/)

### 常用命令
![](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png)

> - Workspace：工作区
> - Index / Stage：暂存区
> - Repository：仓库区（或本地仓库）
> - Remote：远程仓库

### 起步

> ```
> # 显示当前的Git配置
> $ git config --list
> # 编辑Git配置文件
> $ git config -e [--global]
> # 设置提交代码时的用户信息
> $ git config [--global] user.name "[name]"
> $ git config [--global] user.email "[email address]"
> 
> # 将当前或新建目录初始化为Git代码库
> $ git init [project-dir]
> $ git add *
> # git add LICENSE
> $ git commit -m 'initial project version'
> 
> # 跳过暂存区
> $ git commit -a
> # 克隆一个项目和它的代码历史
> $ git clone [url]
> 
> 
> # git add可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。 将这个命令理解为“添加内容到下一次提交中”而不是“将一个文件添加到项目中”要更加合适。 
> # 显示文件状态
> $ git status -s
> 新添加的未跟踪文件前面有 ?? 标记，
> 新添加到暂存区中的文件前面有 A 标记
> 出现在右边的 M 表示该文件被修改了但是还没放入暂存区，
> 出现在靠左边的 M 表示该文件被修改了并放入了暂存区。
> # 查看未暂存的修改
> $ git diff
> # 查看已暂存的修改
> $ git diff --staged
> 
> # git ignore
> https://github.com/github/gitignore
> ```
#### 别名
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
$ git config --global alias.unstage 'reset HEAD --'
$ git config --global alias.last 'log -1 HEAD'
$ git config --global alias.visual '!gitk'外部命令
#### 远程仓库
- git remote -v查看所有远程仓库
- git remote show <shortname>查看某个仓库
- git remote add <shortname> <url> 添加仓库
- $ git fetch <shortname>拉取更新，不自动合并到当前分支
- $ git pull <shortname>拉取更新，自动合并到当前分支
- git push origin master推送到仓库
- $ git remote rename oldname newname
- $ git remote rm paul
#### 取消暂存
-  $ git commit --amend这个命令会将暂存区中的文件提交
-  $ git reset HEAD <file>... 来取消暂存。
-  撤销未暂存的修改
-  $ git checkout -- <file>...
-  将文件变踢出版本控制
-  $ git rm --cached <file>
-  删除已提交暂存区的文件，同时删除本地
-  $ git rm
-  修改文件名
-  $ git mv,如果通过外部mv，需要添加新的文件，踢出旧的文件
### 标签
- git tag
- $ git tag -l 'v1.8.5*'查看标签
- $ git tag -a v1.4 -m 'my version 1.4'打标签
- $ git show v1.4查看具体标签
- $ git tag v1.4-lw打轻量标签
- $ git tag -a v1.2 9fceb02给历史提交打标签，最后的是部分校验和，可用git log查看
- $ git push origin v1.5给远程仓库也打上标签v1.5
- $ git push origin --tags给远程仓库同步打上标签
### 查看提交日志
git log [options] [-- dir]

- -n显示最近几次的日志，不写为全部
- --stat显示哪些文件改变了，增加了几行，减少了几行统计信息
- -p显示修改的内容
- --pretty=oneline一次提交在一行显示
- --pretty=format:"%h - %an, %ar : %s"
- --graph
- --since, --after仅显示指定时间之后的提交。
- --until, --before仅显示指定时间之前的提交。
$ git log --since=2.weeks
- --author/--commiter 选项显示指定作者/提交者的提交
- --grep 选项搜索提交说明中的关键字。
- -S查看包含字符串的提交 $ git log -Sfunction_name
- --shortstat只显示 --stat 中最后的行数修改添加移除统计。
- --name-only仅在提交信息后显示已修改的文件清单。
- --name-status显示新增、修改、删除的文件清单。
- --abbrev-commit仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
- --relative-date使用较短的相对时间显示（比如，“2 weeks ago”）。
- $ git log --pretty="%h - %s" --author=gitster --since="2008-10-01" \
   --before="2008-11-01" --no-merges -- t/
   查看 Git 仓库中，2008 年 10 月期间，Junio Hamano 提交的但未合并的测试文件，

### 分支合并
1. 在当前分支上进行了修改，此时需要在原有分支上进行紧急修复
- 建立新分支
$ git checkout -b iss53
Switched to a new branch "iss53"
等同于
$ git branch iss53
$ git checkout iss53
- 修改，提交
git commit -a -m 'added a new footer [issue 53]'
- 将修改全部提交！
- 切回原有分支
$ git checkout master
Switched to branch 'master'
- 建立分支紧急修复，并提交
$ git checkout -b hotfix
Switched to a new branch 'hotfix'
$git commit -a -m 'fixed the broken email address'
- 切回主分支并合并紧急修复分支
$ git checkout master
$ git merge hotfix
- 删除紧急分支
$ git branch -d hotfix
Deleted branch hotfix (3a0874c).
- 返回修改工作
$ git checkout iss53
Switched to branch "iss53"
$ git commit -a -m 'finished the new footer [issue 53]'
你在 hotfix 分支上所做的工作并没有包含到 iss53 分支中。 如果你需要拉取 hotfix 所做的修改，你可以使用 **git merge master** 命令将 master 分支合并入 iss53 分支，或者你也可以等到 iss53 分支完成其使命，再将其合并回 master 分支。
- 合并到主分支
$ git checkout master
$ git merge iss53
- 删除修改分支
$ git branch -d iss53

#### 解决冲突
- 使用 git status 命令来查看那些因包含合并冲突而处于未合并（unmerged）状态的文件：
- 在你解决了所有文件里的冲突之后，对每个文件使用 git add 命令来将其标记为冲突已解决。 一旦暂存这些原本有冲突的文件，Git 就会将它们标记为冲突已解决。
- 图形化工具来解决冲突，你可以运行 git mergetool（mac默认opendiff ）
- git commit