+++
title = "Redis哨兵"
date = 2020-07-07
tags = ["- Redis"]
categories = ["Redis"]
+++

### Sentinel的分布式特征
Redis Sentinel是一个分布式系统：
Sentinel本身的设计是在为多个Sentinel进程协同合作的配置中运行。

- 当多个哨兵就给定的主机不再可用的事实达成共识时，将执行故障检测，降低了误报的可能性。
- 即使不是所有的Sentinel进程都在工作，Sentinel仍可正常工作，从而使系统能够应对故障。毕竟，拥有故障转移系统本身就是一个单点故障，

### 运行哨兵

```
redis-sentinel /path/to/sentinel.conf
```
`redis-sentinel`是`redis-server`的一个软链接，所以也可以使用以下方法：

```
redis-server /path/to/sentinel.conf --sentinel
```
两种方法的工作原理相同。

但是，再运行`Sentinel`时**必须**指定配置文件，因为系统将使用此文件来保存当前状态，以便在重新启动时重新加载。如果未指定文件，则会启动失败。

Sentinels默认情况下会**监听TCP端口26379连接**，因此必须打开26379端口。

### Sentinel的基础知识

1. 一个健壮的集群至少需要三个Sentinel实例
2. 应该将三个Sentinel实例部署在不同的机器上
3. 因为Redis使用的是异步复制，所以不能保证在故障转移期间保证数据的写入。


### 配置哨兵

Redis的源码包包含一个sentinel.conf文件可用于配置Sentinel，典型的最小配置如下所示：

```
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1
```

sentinel monitor含义如下：

```
sentinel monitor <master-group-name> <ip> <port> <quorum>
```
`<master-group-name>` 指主节点名称，`<ip>` IP地址 `<port>`端口号，重点说一下`<quorum>`：
假如有5个Sentinel进程，并且给定主服务器的quorum置为2，则将发生以下情况：

- 如果有两个哨兵同时发现主节点不可访问，则其中一个哨兵将尝试启动故障转移。
- 如果有三个哨兵同时发现主节点不可访问，则将启动故障转移。

```
sentinel down-after-milliseconds
```
`down-after-milliseconds`是指Sentinel在指定的时间内没有获得实例的响应，则认为实例已关闭。

```
sentinel parallel-syncs
```

`parallel-syncs` 设置每次可以对几个副本进行同步数据

### Redis 哨兵部署示例

#### 经典三节点最小部署

```
                           +----+
                           | M1 |
                           | S1 |
                           +----+
                              |
                    +----+    |    +----+
                    | R2 |----+----| R3 |
                    | S2 |         | S3 |
                    +----+         +----+

                   Configuration: quorum = 2
```
如果主M1发生故障，则S2和S3将达成协议，并能够开启故障转移，从而使客户端能够继续使用。

#### 模拟发生网络分区

```
                             +----+
                             | M1 |
                             | S1 | <- C1 (writes will be lost)
                             +----+
                                |
                                /
                                /
                    +------+    |    +----+
                    | [M2] |----+----| R3 |
                    | S2   |         | S3 |
                    +------+         +----+

```

在这种情况下，网络分区隔离了旧的主数据库M1，因此副本R2被提升为主数据库。但是，与旧主服务器位于同一分区中的客户端（例如C1）可能会继续向旧主服务器写入数据。该数据将永远丢失，因为当分区恢复正常时，主服务器将被重新配置为新主服务器的副本，从而造成数据丢失。


使用以下Redis复制功能可以缓解此问题，如果主服务器检测到副本数量没有达到指定的副本数据，则停止接受数据写入。

```
# 最小副本数量
min-replicas-to-write 1
# 发送异步确认的最大时间
min-replicas-max-lag 10
```









