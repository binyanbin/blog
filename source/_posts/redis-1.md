---
title: redis学习1 环境及配置
date: 2016-2-28
desc: redis 配置
---
最近工作需要使用redis，现在只能边学习边总结。Redis的介绍就不说了，网上一搜一大把。

## 环境搭建
官方不提供windows版redis，[微软开源技术的github](https://github.com/MSOpenTech)上有提供，可以[下载](https://github.com/MSOpenTech/redis/releases)。下载解决之后，我们就开始可以搭建开发环境。
<!-- more -->

### 配置
* maxheap
这个是一个强限制，maxheap的大小包括文件存储大小及内存存储大小。如果超过这个限制，服务就会结束。
* maxmemory
maxheap必需要比maxmemory大，一般设置为maxmemory的1.5倍。
* 文件系统大小
redis官方提供的公式:
``` 
(size of physical memory) + (2 * size of maxheap)
```
如果你有一台机器内存为8G,maxheap设置为8G，那么你至少要有这么多空闲硬盘空间:
``` 
(8GB) + (2 * 8GB) = 24GB
```
* maxmemory-policy
如果运行中达到了maxmemory，redis将根据这个设置清除一些存储数据。
* requirepass 
设置服务密码。
* heapdir
内存映射文件路径
* timeout
连接超时时间

### 运行
启动服务
``` cmd
redis-server redis.windows.conf
```
启动客户端
``` cmd
redis-cli -h 127.0.0.1 -p 6379 -a password
```
### 基本命令
``` cmd
set key value 
get key
del key
exists key
expire key 100
keys *
db size
ttl key
info
flushdb 
ping
```
