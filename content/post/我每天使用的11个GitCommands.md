+++
title = "我每天使用的11个Git Commands"
date = 2022-04-12
tags = ["Git"]
categories = ["Git"]
+++

> 本文是为我的年轻人（或任何新手）准备的，他们可以有效地使用git命令行，并简要介绍了我如何使用这些命令。
> https://medium.com/@mmpatil34/11-git-commands-i-use-daily-9bbd7590c8eb

### 1. git fetch origin
从特定的仓库拉取所有的branchs/tags, 这里的仓库是“origin”, 我每天从这个命令开始, 它可以让本地和远程仓库状态保持一致.
### 2. git status
显示当前分支自上次提交到现在的文件改动列表,在切换分支、创建新分支、进行新更改或拉取更改之前, 此命令可以检查是否有文件需要被stash.
### 3. git checkout
```git checkout -b <new branch name> origin/<source branch name>```
从特定源分支创建一个新分支,这块的“origin”代表默认仓库.
```git checkout — — <name of the file>```
当本地有文件变动时,可以使用这个命令丢弃当前改变,恢复文件到之前状态.
```git checkout <branch name>```
切换本地到指定分支.
### 4. git pull origin <branch name>
将更改从远程分支拉到本地分支，并在更改兼容的情况下调用```git merge```。
```git pull 和 git fetch```的不同之处是:``` git pull = git fetch + git merge```
### 5. git add <name of the file>
文件修改完成后,就可以使用```git add```命令将文件添加到特定提交中,使用``git status``命令可以很方便的获取到要添加指定提交的文件名.
### 6. git commit
```git commit -m "<提交内容的描述>"```
提交本地改变,并指定和提交内容相关的描述.
### 7. git push origin <branch name>
将本地提交推送到远程存储库,这里的仓库是“origin”.
### 8. git cherry-pick
```
a - b - c - d   Master
         \
           e - f - g Feature
```
现在将提交f应用到master分支。
```

# 切换到 master 分支
$ git checkout master
 
# Cherry pick 操作
$ git cherry-pick f
```
上面的操作完成以后，代码库就变成了下面的样子。
```
 a - b - c - d - f   Master
         \
           e - f - g Feature
```
```
git cherry-pick -m 1 <commit_id>
```

如果原始提交是一个合并节点，来自于两个分支的合并，那么 Cherry pick 默认将失败，因为它不知道应该采用哪个分支的代码变动。

```-m```配置项告诉 Git，应该采用哪个分支的变动。它的参数parent-number是一个从1开始的整数，代表原始提交的父分支编号。

上面命令表示，Cherry pick 采用提交commitHash来自编号1的父分支的变动。
一般来说，1号父分支是接受变动的分支（the branch being merged into），2号父分支是作为变动来源的分支（the branch being merged from）。
### 9. git revert <commit id>
引入一个新的提交来撤回已经push的提交.
### 10. git reset — soft HEAD~1
撤消一次本地提交而不会丢失文件中的变更.
### 11. git reset — hard HEAD~1
撤销一次本地提交并且丢弃文件中的变更.
希望对你有帮助. 感谢阅读😁
