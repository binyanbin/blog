---
title: redis学习6 持久化
date: 2016-3-8
desc: redis  持久化
---
redis提供二种持久化方式:一种是RDB,另一种是AOF.
RDB持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。
AOF持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行重写（rewrite），使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。
<!-- more -->
## 配置
* rdb配置
比如说， 以下设置会让 Redis 在满足“ 60 秒内有至少有 1000 个键被改动”这一条件时， 自动保存一次数据集:
save 60 1000
* aof配置
appendonly yes  启用
appendfsync always 每次有新命令追加到 AOF 文件时就执行一次 fsync ：非常慢，也非常安全
appendfsync everysec 每秒 fsync 一次：足够快（和使用 RDB 持久化差不多），并且在故障时只会丢失 1 秒钟的数据。
appendfsync no 从不 fsync ：将数据交给操作系统来处理。更快，也更不安全的选择。
配置好，启动服务之后，在你程序的目录会出现二个文件，一个是dump.rdb,一个是appendonly.aof
* RDB和AOF 之间的相互作用
1. 当 Redis 启动时，如果RDB持久化和AOF持久化都被打开了，那么程序会优先使用 AOF 文件来恢复数据集，因为 AOF 文件所保存的数据通常是最完整的。
2. Redis为了防止两个后台(RDB和AOF)进程同时对磁盘进行大量的 I/O 操作。redis在RDB Save的过程中，不会执行 AOF RewriteAOF。反之,在AOF RewriteAOF执行的过程中，也不会执行RDB Save。
