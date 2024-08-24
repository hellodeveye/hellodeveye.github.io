+++
title = "日志系统：一条SQL更新语句是如何执行的?"
date = 2022-06-06
tags = ["MySQL"]
categories = ["MySQL实战45讲"]
+++

### redo log（重做日志）

InnoDB 的 redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1GB，那么总共就可以记录 4GB 的操作。从头开始写，写到末尾就又回到开头循环写，如下面这个图所示。
![](http://qiniu.xiaocm.com/blog/img/20220606110553.png)

write pos 是当前记录的位置，一边写一边后移，写到第 3 号文件末尾后就回到 0 号文件开头。checkpoint 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。

有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为crash-safe。

### binlog（归档日志）

binlog没有crash-safe能力，只能用于归档。

这两种日志的不同点：

- redo log是InnoDB引擎特有的，binlog是MySQL的server层实现的，所有引擎都可以使用。

- **redo log是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑。**

- redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。


### 两阶段提交

```
mysql> update T set c=c+1 where ID=2;
```
执行器和 InnoDB 引擎在执行这个简单的 update 语句时的内部流程:

1. 执行器先找引擎取 ID=2 这一行。ID 是主键，引擎直接用树搜索找到这一行。如果 ID=2 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。

2. 执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。

3. 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。

4. 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。

5. 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。

流程图如下所示：
![](http://qiniu.xiaocm.com/blog/img/20220606112558.png)

### 如何让数据库恢复到半个月任意一秒？

- 首先，找到最近的一次全量备份，如果你运气好，可能就是昨天晚上的一个备份，从这个备份恢复到临时库；
- 然后，从备份的时间点开始，将备份的 binlog 依次取出来，重放到中午误删表之前的那个时刻。

### 总结

建议```innodb_flush_log_at_trx_commit``` 这个参数设置成 1 的时候，表示每次事务的 redo log 都直接持久化到磁盘，这样可以保证 MySQL 异常重启之后数据不丢失。

建议```sync_binlog ```这个参数设置成 1 ，表示每次事务的 binlog 都持久化到磁盘，这样可以保证 MySQL 异常重启之后 binlog 不丢失。

<iframe id="embed_dom" name="embed_dom" frameborder="0" style="display:block;width:715px; height:245px;" src="https://www.processon.com/embed/629f14fd07912907215003f8"></iframe>

