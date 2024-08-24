+++
title = "Linux常用命令"
date = 2020-09-27
tags = ["Linux"]
categories = ["Linux"]
+++


- 查看配置文件只匹配不是以“#”开头的行
```
 cat .zshrc | grep -v "^#" > profile.txt
```
- 使用`tail`查看指定行以后的N行日志
```
tail -n +358653 catalina.2021-01-12.out|head -n 20
```
