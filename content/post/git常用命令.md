+++
title = "Git常用命令"
date = 2019-02-18
tags = ["Git"]
categories = ["Git"]
+++

![](http://qiniu.xiaocm.com/20210316200434.png)
### 一、新建代码库
> <font color=gray>\# 在当前目录新建一个Git代码库</font>  
> $ git init  
>
> <font color=gray>\# 新建一个目录，将其初始化为Git代码库</font>  
> $ git init [project-name]  
>
> <font color=gray>\# 下载一个项目和它的整个代码历史</font>  
> $ git clone [url]  
### 二、配置
Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）  
> <font color=gray>\# 显示当前的Git配置</font>  
> $ git config --list  
>
> <font color=gray>\# 编辑Git配置文件</font>  
> $ git config -e [--global]  
>
> <font color=gray>\# 设置提交代码时的用户信息</font>  
> $ git config [--global] user.name "[name]"  
> $ git config [--global] user.email "[email   address]"
### 三、增加/删除文件
> <font color=gray>\# 添加指定文件到暂存区</font>  
> $ git add [file1] [file2] ...  
>
> <font color=gray>\# 添加指定目录到暂存区，包括子目录</font>  
> $ git add [dir]  
>
> <font color=gray>\# 添加当前目录的所有文件到暂存区</font>  
> $ git add .  
>
> <font color=gray>\# 添加每个变化前，都会要求确认</font>  
> <font color=gray>\# 对于同一个文件的多处变化，可以实现分次提交</font>  
> $ git add -p
>
> <font color=gray>\# 删除工作区文件，并且将这次删除放入暂存区</font>  
> $ git rm [file1] [file2] ...  
>
> <font color=gray>\# 停止追踪指定文件，但该文件会保留在工作区</font>  
> $ git rm --cached [file]  
>
> <font color=gray>\# 改名文件，并且将这个改名放入暂存区</font>  
> $ git mv [file-original] [file-renamed]  
### 四、代码提交
> <font color=gray>\# 提交暂存区到仓库区</font>  
> $ git commit -m [message]  
>
> <font color=gray>\# 提交暂存区的指定文件到仓库区</font>  
> $ git commit [file1] [file2] ... -m [message]  
>
> <font color=gray>\# 提交工作区自上次commit之后的变化，直接到仓库区</font>  
> $ git commit -a  
>
> <font color=gray>\# 提交时显示所有diff信息</font>  
> $ git commit -v  
>
> <font color=gray>\# 使用一次新的commit，替代上一次提交</font>  
> <font color=gray>\# 如果代码没有任何新变化，则用来改写上一次commit的提交信息</font>  
> $ git commit --amend -m [message]  
>
> <font color=gray>\# 重做上一次commit，并包括指定文件的新变化</font>  
> $ git commit --amend [file1] [file2] ...  
### 五、分支
> <font color=gray>\# 列出所有本地分支</font>  
> $ git branch  
>
> <font color=gray>\# 列出所有远程分支</font>  
> $ git branch -r  
>
> <font color=gray>\# 列出所有本地分支和远程分支</font>  
> $ git branch -a  
>
> <font color=gray>\# 新建一个分支，但依然停留在当前分支</font>  
> $ git branch [branch-name]  
>
> <font color=gray>\# 新建一个分支，并切换到该分支</font>  
> $ git checkout -b [branch]  
>
> <font color=gray>\# 新建一个分支，指向指定commit</font>  
> $ git branch [branch] [commit]  
>
> <font color=gray>\# 新建一个分支，与指定的远程分支建立追踪关系</font>  
> $ git branch --track [branch] [remote-branch]  
>
> <font color=gray>\# 切换到指定分支，并更新工作区</font>  
> $ git checkout [branch-name]  
>
> <font color=gray>\# 切换到上一个分支</font>  
> $ git checkout -  
>
> <font color=gray>\# 建立追踪关系，在现有分支与指定的远程分支之间</font>  
> $ git branch --set-upstream [branch] [remote-branch]  
>
> <font color=gray>\# 合并指定分支到当前分支</font>  
> $ git merge [branch]  
>  
> <font color=gray>\# 选择一个commit，合并进当前分支</font>  
> $ git cherry-pick [commit]  
>
> <font color=gray>\# 删除分支</font>  
> $ git branch -d [branch-name]  
>
> <font color=gray>\# 删除远程分支</font>  
> $ git push origin --delete [branch-name]  
> $ git branch -dr [remote/branch]  
### 六、标签
> <font color=gray>\# 列出所有tag</font>  
> $ git tag  
>
> <font color=gray>\# 新建一个tag在当前commit</font>  
> $ git tag [tag]  
>
> <font color=gray>\# 新建一个tag在指定commit</font>  
> $ git tag [tag] [commit]  
>
> <font color=gray>\# 删除本地tag</font>  
> $ git tag -d [tag]  
>
> <font color=gray>\# 删除远程tag</font>  
> $ git push origin :refs/tags/[tagName]  
>
> <font color=gray>\# 查看tag信息</font>  
> $ git show [tag]  
>
> <font color=gray>\# 提交指定tag</font>  
> $ git push [remote] [tag]  
>
> <font color=gray>\# 提交所有tag</font>  
> $ git push [remote] --tags  
>
> <font color=gray>\# 新建一个分支，指向某个tag</font>  
> $ git checkout -b [branch] [tag]  
### 七、查看信息
> <font color=gray>\# 显示有变更的文件</font>  
> $ git status  
>
> <font color=gray>\# 显示当前分支的版本历史</font>  
> $ git log  
>
> <font color=gray>\# 显示commit历史，以及每次commit发生变更的文件</font>  
> $ git log --stat  
>
> <font color=gray>\# 搜索提交历史，根据关键词</font>  
> $ git log -S [keyword]  
>
> <font color=gray>\# 显示某个commit之后的所有变动，每个commit占据一行</font>  
> $ git log [tag] HEAD --pretty=format:%s  
>
> <font color=gray>\# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件</font>  
> $ git log [tag] HEAD --grep feature  
>
> <font color=gray>\# 显示某个文件的版本历史，包括文件改名</font>  
> $ git log --follow [file]  
> $ git whatchanged [file]  
>
> <font color=gray>\# 显示指定文件相关的每一次diff</font>  
> $ git log -p [file]  
>
> <font color=gray>\# 显示过去5次提交</font>  
> $ git log -5 --pretty --oneline  
>
> <font color=gray>\# 显示所有提交过的用户，按提交次数排序</font>  
> $ git shortlog -sn  
>
> <font color=gray>\# 显示指定文件是什么人在什么时间修改过</font>  
> $ git blame [file]  
>
> <font color=gray>\# 显示暂存区和工作区的代码差异</font>  
> $ git diff  
>
> <font color=gray>\# 显示暂存区和上一个commit的差异</font>  
> $ git diff --cached [file]  
>
> <font color=gray>\# 显示工作区与当前分支最新commit之间的差异</font>  
> $ git diff HEAD  
>
> <font color=gray>\# 显示两次提交之间的差异</font>  
> $ git diff [first-branch]...[second-branch]  
>
> <font color=gray>\# 显示今天你写了多少行代码</font>  
> $ git diff --shortstat "@{0 day ago}"  
>
> <font color=gray>\# 显示某次提交的元数据和内容变化</font>  
> $ git show [commit]  
>
> <font color=gray>\# 显示某次提交发生变化的文件</font>  
> $ git show --name-only [commit]  
>
> <font color=gray>\# 显示某次提交时，某个文件的内容</font>  
> $ git show [commit]:[filename]  
>
> <font color=gray>\# 显示当前分支的最近几次提交</font>  
> $ git reflog  
>
> <font color=gray>\# 从本地master拉取代码更新当前分支：branch 一般为master</font>  
> $ git rebase [branch]  
### 八、远程同步
> $ git remote update  --更新远程仓储  
> <font color=gray>\# 下载远程仓库的所有变动</font>  
> $ git fetch [remote]  
>  
> <font color=gray>\# 显示所有远程仓库</font>  
> $ git remote -v  
>
> <font color=gray>\# 显示某个远程仓库的信息</font>  
> $ git remote show [remote]  
>
> <font color=gray>\# 增加一个新的远程仓库，并命名</font>   
> $ git remote add [shortname] [url]  
>
> <font color=gray>\# 取回远程仓库的变化，并与本地分支合并</font>  
> $ git pull [remote] [branch]  
>
> <font color=gray>\# 上传本地指定分支到远程仓库</font>  
> $ git push [remote] [branch]  
>
> <font color=gray>\# 强行推送当前分支到远程仓库，即使有冲突</font>  
> $ git push [remote] --force  
>
> <font color=gray>\# 推送所有分支到远程仓库</font>  
> $ git push [remote] --all  
### 九、撤销  
> <font color=gray>\# 恢复暂存区的指定文件到工作区</font>  
> $ git checkout [file]   
>
> <font color=gray>\# 恢复某个commit的指定文件到暂存区和工作区</font>  
> $ git checkout [commit] [file]  
>  
> <font color=gray>\# 恢复暂存区的所有文件到工作区</font>  
> $ git checkout .  
>
> <font color=gray>\# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变</font>  
> $ git reset [file]  
>
> <font color=gray>\# 重置暂存区与工作区，与上一次commit保持一致</font>  
> $ git reset --hard
>
> <font color=gray>\# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变</font>  
> $ git reset [commit]  
>
> <font color=gray>\# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致</font>  
> $ git reset --hard [commit]
>
> <font color=gray>\# 重置当前HEAD为指定commit，但保持暂存区和工作区不变</font>  
> $ git reset --keep [commit]  
>
> <font color=gray>\# 新建一个commit，用来撤销指定commit  
\# 后者的所有变化都将被前者抵消，并且应用到当前分支</font>  
> $ git revert [commit]  
>
> <font color=gray>\# 暂时将未提交的变化移除，稍后再移入</font>  
> $ git stash  
> $ git stash pop  
### 十、其他
> <font color=gray>\# 生成一个可供发布的压缩包</font>  
> $ git archive
