+++
title = "'git commit之后，撤销commit'"
date = 2020-08-21
tags = ["Git"]
categories = ["Git"]
+++


### 场景

一般情况下，我们写完代码后会执行:
```
git add .
git commit -m "xxx"
```

但是执行完后，想撤回怎么办？可以执行以下命令:

```
git reset --soft HEAD^
```
这样就可以撤回你的提交，并且不会丢失提交前修改的内容.

### 理解

HEAD^ 代表上一个版本，同等于HEAD~1，如果进行了两次提交，可以写成HEAD~2

**参数**

> -\-mixed

默认参数，不删除工作空间改动的代码，只撤回提交，并且撤回`git add .`操作

> -\-soft

不删除工作空间改动的代码，撤销提交，但是不撤回`git add .` 操作

> -\-hard

删除工作空间改动的代码，撤销commit，撤销`git add .`，直接回退到上次`commit`

### 最后

如果commit注释写错了，只是想改一下注释，只需要：

```
git commit --amend
```
此时会进入默认vim编辑器，修改注释完毕后保存就好了。
