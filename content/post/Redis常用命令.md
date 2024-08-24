+++
title = "Redis常用命令"
date = 2020-10-28
tags = ["Redis"]
categories = ["工作"]
+++


### 工作中常用到的Reids命令

- 模糊匹配*key*并批量删除

```
redis-cli -u redis://password@ip:port/database keys "business:data:10103*" | xargs redis-cli -u redis://password@ip:port/database del
```
