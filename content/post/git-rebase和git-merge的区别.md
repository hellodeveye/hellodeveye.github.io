+++
title = "git rebase和git merge的区别"
date = 2020-08-19
tags = ["Git"]
categories = ["Git"]
+++


### Description
`git rebase` 和 `git merge` 一样都是用于从一个分支获取并且合并到当前分支，但是他们采取不同的工作方式，以下面的一个工作场景说明其区别.

如图所示：你在一个feature分支进行新特性的开发，与此同时，master 分支的也有新的提交。

![http://qiniu.xiaocm.com/b454bf1d01ec3dee808830b24dd87c2e.png](http://qiniu.xiaocm.com/b454bf1d01ec3dee808830b24dd87c2e.png)
为了将master 上新的提交合并到你的feature分支上，你有两种选择：`merging or rebasing`

### merge

执行以下命令:

```
git checkout feature
git merge master
```
或者执行更简单的：
`git merge master feature`

那么此时在feature上git 自动会产生一个新的commit(merge commit)
look like this：

![http://qiniu.xiaocm.com/2cebea59e5f82803cb35f99f85b6653d.png](http://qiniu.xiaocm.com/2cebea59e5f82803cb35f99f85b6653d.png)

**merge 特点：**自动创建一个新的commit,如果合并的时候遇到冲突，仅需要修改后重新commit
**优点：**记录了真实的commit情况，包括每个分支的详情
**缺点：**因为每次merge会自动产生一个merge commit，所以在使用一些git 的GUI tools，特别是commit比较频繁时，看到分支很杂乱。

### rebase
本质是变基,执行以下命令：
```
git checkout feature
git rebase master
```
![http://qiniu.xiaocm.com/245938aba30e1d7ff14f759eea81eb37.png](http://qiniu.xiaocm.com/245938aba30e1d7ff14f759eea81eb37.png)
**rebase 特点：**会合并之前的commit历史
**优点：**得到更简洁的项目历史，去掉了merge commit
**缺点：**如果合并出现代码问题不容易定位，因为re-write了history
合并时如果出现冲突需要按照如下步骤解决

- 修改冲突部分
- git add
- git rebase --continue
- （如果第三步无效可以执行 git rebase --skip）

不要在git add 之后习惯性的执行 git commit命令
***The Golden Rule of Rebasing rebase*** 的黄金法则:
**never use it on public branches(不要在公共分支上使用)**
比如说如下场景：如图所示
![http://qiniu.xiaocm.com/ac37304e85f5ddf56f1fc302b9e42781.png](http://qiniu.xiaocm.com/ac37304e85f5ddf56f1fc302b9e42781.png)

如果你rebase master 到你的feature分支：
rebase 将所有master的commit移动到你的feature 的顶端。问题是：其他人还在original master上开发，由于你使用了rebase移动了master，git 会认为你的主分支的历史与其他人的有分歧，会产生冲突。
所以在执行git rebase 之前 问问自己，

会有其他人看这个分支么？
IF YES 不要采用这种带有破坏性的修改commit 历史的rebase命令
IF NO OK，随你便，可以使用rebase

### Summary 总结

如果你想要一个干净的，没有merge commit的线性历史树，那么你应该选择git rebase
如果你想保留完整的历史记录，并且想要避免重写commit history的风险，你应该选择使用git merge

### 参考资料

> https://www.atlassian.com/git/tutorials/merging-vs-rebasing/conceptual-overview
> https://git-scm.com/book/zh/v2/Git-分支-变基
> https://git-scm.com/book/zh/v2/Git-分支-分支的新建与合并#_basic_merging
