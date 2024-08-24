+++
title = "Redis安装与配置"
date = 2018-07-06
tags = ["- Redis"]
categories = ["Redis"]
+++


### 下载

1. 从[Redis官网](http://www.redis.io)下载redis 最新稳定版本
2. 解压到 /usr/local/ 目录下

### 安装

```
make && make test && make install
```
make时报如下错误:

```
zmalloc.h:50:31: error: jemalloc/jemalloc.h: No such file or directory
zmalloc.h:55:2: error: #error "Newer version of jemalloc required"
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/data0/src/redis-2.6.2/src'
make: *** [all] Error 2
```

原因是jemalloc重载了Linux下的ANSI C的malloc和free函数。
解决办法：

```
make MALLOC=libc
```
### redis 生产环境启动方案

1. redis utils 目录下，有个redis_init_script脚本
2. 将redis_init_script脚本拷贝到linux的/etc/init.d 目录中，将redis_init_script重命名为redis_6379,   6379是我们希望redis 的实例端口号
3. 创建两个目录 /etc/redis (存放redis 配置的目录), /var/redis/6379 (存放redis的持久文件)
4. 修改redis.conf配置文件（默认在根目录下），拷贝到/etc/redis目录下，修改名称为6379.conf
5. 修改redis.conf中的部分配置为生产环境

| 参数   | 值   |  说明  |
| --- | --- | --- |
| daemonize | yes   |   让redis以daemon进程运行 |
| pidfile   |  /var/run/redis_6379.pid | 设置redis的pid文件位置 |
| port      | 6379  | 设置redis的监听端口号 |
| dir       | /var/redis/6379 | 设置持久化文件的存储位置 |

6. 启动redis:

```
cd /etc/init.d 
chmod 777 redis_6379 
./redis_6379 start
```

7. 确认redis 进程是否启动: `ps -ef  |  grep redis`
8. 让redis跟随系统启动自动启动，在redis_6379 脚本中，最上面加入两行注释

```
# chkconfig:    2345 90 10
# description: Redis is a persistent key-value database
chkconfig redis_6379 on   
```

### 主从架构（主节点和从节点最好保证一致的版本）

1. 在 slave node 上配置:

```
slaveof 192.168.1.1 6379
```
2. 强制读写分离

基于主从复制架构，实现读写分离
redis slave node 只读，默认开启，`slave-read-only`
开启了只读的redis slave node，会拒绝所有的写操作，这样可以强制搭建成读写分离架构

3. 集成安全认证
master 上启用安全认证策略，`requirepass`
master 连接口令 `masterauth`

4. 读写分离架构的测试
先启动主节点，再启动从节点，在搭建生产环境的时候，不要忘记修改一个配置：

```
bind 127.0.0.1
```
每个redis.conf 中的bind `127.0.0.1` -> bind自己的ip，并且防火墙打开6379端口

###  redis-cli的使用
| 命令  |  说明  |
| --- | --- |
| redis-cli shutdown |连接本机的6379端口停止redis进程 |
|redis-cli -h 127.0.0.1 -p 6379 |指定要连接的ip和端口号|
|redis-cli ping | ping redis的端口，看是否正常|
|redis-cli |进入交互式命令行|
|set k1 v1| 设置一个key:value|
|get k1 | 获取key的值|


