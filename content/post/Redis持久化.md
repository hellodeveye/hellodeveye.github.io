+++
title = "Redis持久化"
date = 2018-07-06
tags = ["- Redis"]
categories = ["Redis"]
+++

## Redis的持久化机制-RDB

### 1.什么是RDB
The RDB persistence performs point-in-time snapshots of your dataset at specified intervals.(RDB持久化是以指定的时间间隔执行数据集的时间点快照。)
简单来说，RDB是每隔一段时间，会把内存中的数据写入磁盘的临时文件作为快照，恢复时把快照文件读取到内存中。如果机器宕机，那么内存中的数据就会丢失，使用RDB机制后，重启后数据会恢复。

### 2.备份与恢复

内存备份 --> <font color="#e58484">磁盘临时文件</font>
临时文件 --> <font color="#e58484">恢复到内存</font>

### 3.RDB优劣势

* **优势**
    1. 每隔一段时间全量备份
    2. 容灾简单，适合进行远程冷备份
    3. 子进程备份的时候，主进程不会有任何IO操作（不会有写入或修改），保证备份数据的完整性
    4. 相对AOF来说，当有更大的文件时候可以快速重启恢复

* **劣势**
    1. 发生故障时，可能会丢失最后一次备份数据
    2. 子进程所占用的内存会和父进程一模一样，会造成CPU负担
    3. 由于定时全量备份是重量级操作，所以对于实时备份，就无法处理了

### 4.RDB的配置

1. 保存位置可以自定义配置:

```
dir /var/redis/6379/dump.rdb
```
2. 保存机制

```
#   save <seconds> <changes>
#   900秒（15分钟）后，如果至少有 1 个Key发生改变
#   300秒（5分钟）后，如果至少有 10 个Key发生改变
#   60秒（1分钟）后，如果至少有 10000 个Key发生改变

save 900 1
save 300 10
save 60 10000
```
## Redis的持久化机制-AOF

### 1.什么是AOF
The AOF persistence logs every write operation received by the server, that will be played again at server startup, reconstructing the original dataset. Commands are logged using the same format as the Redis protocol itself, in an append-only fashion. Redis is able to rewrite the log in the background when it gets too big.

AOF持久化会记录服务器接收到的每个写入操作，这些操作将在服务器启动时再次执行，以重新构建原始的数据。命令记录的格式与Redis协议本身的格式相同，采用追加方式。当日志文件过大时，Redis可以在后台进行`rewrite`。

### 2.备份与恢复

日志备份 --> <font color="#e58484">命令追加方式</font>
日志文件 --> <font color="#e58484">根据命令日志文件重新构建</font>

### 3.AOF优劣势

* **优势**
    1. 使用AOF数据持久化更加完整
    2. 可以使用不同的fsync策略，使用默认策略fsync时，每秒的写入性能仍然很好（fsync是使用后台线程执行的，并且在没有进行fsync的情况下，主线程将尽力执行写入操作）但是会损失一秒的写入时间
    3. AOF是使用日志追加的方式，如果断电或其他原因导致日志只写了一半，可以使用`redis-check-aof`工具进行修复 
    4. Redis太大时，Redis可以在后台自动重写AOF。 Redis继续追加到旧文件时，会生成一个新的文件，其中包含创建当前数据集**所需的最少操作集**，一旦准备好第二个文件，Redis会切换这两个文件并开始追加到新的那一个
    5. 即使使用`FLUSHALL`命令清空了所有数据，也可以通过修改AOF文件进行数据恢复
* **劣势**
    1. AOF相比RDB占用空间更大
    2. 使用fsync策略后，相比RDB性能差一些
    3. 当服务器宕机后，使用AOF恢复的数据可能不完整

### 4.AOF的配置

1. 开启AOF

```
appendonly yes
```

2. 配置fsync策略

```
# If unsure, use "everysec".
# 如果不确定，推荐使用 "everysec"

# appendfsync always
appendfsync everysec
# appendfsync no
```

3. AOF rewrite配置

```
# 设置一个百分比
auto-aof-rewrite-percentage 100
# 设置AOF 的最小大小
auto-aof-rewrite-min-size 64mb
``` 

4. Redis 启动加载配置

```
# 优先加载AOF 文件
aof-use-rdb-preamble yes
```

## 两种持久化方式改如何选择

- 如果对数据可靠性要求很高，则应同时使用两种持久性方式。
- 如果在灾难情况下仍然可以承受几分钟的数据丢失，则可以仅使用RDB。
- 不推荐单独使用AOF，因为AOF在数据恢复时，有时候会出现Bugs。

