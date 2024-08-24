+++
title = "Git修改commit信息"
date = 2020-10-23
tags = ["Git"]
categories = ["Git"]
+++


#### 修改commit信息主要有这几种情况:
1. 刚刚commit，还没有push，使用`git commit --amend`
2. 刚刚push，要修改最近一个push的commit信息，使用`git commit --amend`
3. 修改历史push的commit信息，使用`git rebase -i HEAD~n`【其中的n为记录数】，配合2中的命令

> **注意：**
其中1、2两种情况的修改方式是一样的，但是git log的记录是不同的,第三种方式也是把需要修改的记录调整为最新的提交，然后使用2的方式修改。


