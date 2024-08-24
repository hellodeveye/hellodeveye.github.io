+++
title = "Redis复制"
date = 2018-07-07
tags = ["- Redis"]
categories = ["Redis"]
+++


### Replication
Redis使用主从复制非常简单，它允许从实例成为主实例的副本，每当链接断开时，从节点将自动重新连接到主服务器上。并且无论主服务器发生什么情况，从节点都尝试完全复制所有数据。

### Redis主从复制使用以下三种机制：

- 当主从实例连接良好时，主节点向从节点发送**命令流**来进行更新，是为了复制由于Key过期或回收等其他操作对数据产生影响。
- 当主从实例断开时，从节点重新连接后只会尝试获取链接断开期间错开的命令流。
- 如果无法进行部分重新同步，则会要求完全重新同步。

> 默认情况下，Redis会使用***异步复制***，Redis从节点会异步地确认接收的数据。因此，主节点不会等待从节点处理完成。但是，如果需要知道从节点都处理了那些命令，也可以选择同步复制。

### Redis 复制的几个特征：

1. Redis是使用异步复制
2. 一个主节点可以有多个从节点
3. 从节点还可以有子节点，从Redis 4.0开始，所有子节点将从主节点接收完全相同的复制流。 
4. Redis主从复制是无阻塞的，意味着在进行复制或同步是仍然可以进行数据查询
5. 复制既可以用于可伸缩性，也可以用于只读查询的多个副本（读写分离）
6. 可以配置主节点不进行数据保存或只启用AOF，然后将数据保存到副本，但是注意:当服务器重启后，从节点服务器重新同步数据时，同时会清空从节点数据。

### Redis 复制的原理：

每个Redis Master 节点都有一个replication ID:这是一个较大的伪随机字符串，用于标记当前数据集。每个Master 还有一个偏移量，该偏移量会针对复制流中要发送到副本的每个字节的增量而增加。
以便更新副本的状态。即使未连接任何副本，复制偏移也会增加。

```
# Replication
role:master
connected_slaves:1
slave0:ip=39.105.157.176,port=6379,state=online,offset=61238,lag=0
# 主 Replication ID
master_replid:649255ffb2183786d00203aa51715169b06f4f47
# 副 Replication ID
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:61238
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:61238
```

当从节点连接主节点时，会使用`PSYNC`命令来发送当前旧的主节点的Replication ID和offset，这样主节点只会同步相差的数据量，如果主节点缓冲区的副本引用（Replication ID）不存在，则会进行全量同步。

#### 全量同步的工作流程如下：
主节点开始后台生成RDB文件。同时，它开始缓冲从客户端收到的所有新写入命令。后台保存完成后，主数据库将数据库文件传输到从节点，从节点将其保存在磁盘上，然后将其加载到内存中。然后，主服务器将所有缓冲的命令发送到副本。

### Replication ID 说明：

如果两个实例具有相同的Replication ID和offset，则它们具有完全相同的数据。但是为什么会有两个Replication ID(主和副)？
<p style="text-indent:2em">因为两个实例A和B具有相同的Replication ID，但一个实例的offset为1000，另一个实例的offset为1023，则意味着第一个实例缺少应用于数据集的某些命令。这也意味着，仅通过应用一些命令，A即可达到与B完全相同的状态。</p>
<p style="text-indent:2em">Redis实例具有两个Replication ID的原因是由于副本被提升为主副本。故障转移后，升级后的副本仍需要记住其过去的Replication ID，因为该Replication ID是以前的主副本之一。这样，当其他副本将与新的主副本同步时，它们将尝试使用旧的主Replication ID执行部分重新同步。因为将副本提升为主副本时，它会将其辅助ID设置为其主ID，并记住发生此ID切换时的offset。稍后它将选择一个新的随机Replication ID，因为新的历史记录开始了。处理新的副本连接时，主机将其ID和offset与当前ID和辅助ID相匹配（为安全起见，直到给定的偏移量）。简而言之，这意味着在故障转移后，连接到新提升的主服务器的副本不必执行完全同步。</p>

### 无盘复制

通常，完全重新同步需要在磁盘上创建RDB文件，然后从磁盘重新加载相同的RDB，以便为副本提供数据。
但是对于硬盘速度慢，但是在内网环境下，可以采用无盘复制，这样可以直接通过Socket将RDB发送给从节点，而无需使用硬盘作为中间存储。

### 配置

```
replicaof 192.168.1.1 6379
```
也可以使用`REPLICAOF`命令进行同步。
可以使用`repl-diskless-sync`配置参数启用无盘复制。在`repl-diskless-sync-delay `参数控制第一个副本之后，为了等待更多副本到达而开始传输的延迟。

```
masterauth <password>
```
设置主节点的密码，也可以使用redis-cli输入`config set masterauth <password>`进行设置。

```
# 在当前至少有N个副本连接到主服务器时才接受写查询
min-replicas-to-write <number of replicas>
# 在有N个副本延迟少于M秒时，则接受写入
min-replicas-max-lag <number of seconds>
```
