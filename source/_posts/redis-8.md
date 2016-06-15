---
title: redis学习8  Sentinel集群控制
date: 2016-3-10
desc: redis  Sentinel 集群
---
Redis-Sentinel是Redis官方推荐的高可用性(HA)解决方案，当用Redis做Master-slave的高可用方案时，假如master宕机了，Redis本身(包括它的很多客户端)都没有实现自动进行主备切换，而Redis-sentinel本身也是一个独立运行的进程，它能监控多个master-slave集群，发现master宕机后能进行自懂切换。
<!-- more -->
## Sentinel任务
* 监控:Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
* 提醒:当被监控的某个Redis服务器出现问题时,Sentinel 可以通过API向管理员或者其他应用程序发送通知。
* 自动故障迁移:当一个主服务器不能正常工作时,Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器, 并让失效主服务器的其他从服务器改为复制新的主服务器;当客户端试图连接失效的主服务器时,集群也会向客户端返回新主服务器的地址,使得集群可以使用新主服务器代替失效服务器。

## 配置Sentinel
下面是一个Sentinel的标准配置
port 26370
sentinel monitor master 127.0.0.1 6381 1
sentinel auth-pass master yanbin
sentinel down-after-milliseconds master 60000
sentinel parallel-syncs master 5
第一行配置指示Sentinel去监视一个名为master的主服务器,将这个主服务器判断为失效至少需要1个Sentinel同意。只要同意Sentinel的数量不达标,自动故障迁移就不会执行。
down-after-milliseconds 指定了Sentinel认为服务器已经断线所需的毫秒数。
parallel-syncs 执行故障转移时,最多可以有多少个从服务器同时对新的主服务器进行同步,数字越小,完成故障转移所需的时间就越长。
auth-pass 监视主服务器的密码。

## 运行Sentinel
Sentinel配置在我这个windows 2.8这个版本中是没有的，我自建了一个sentinel.conf配置文件。然后使用命令行启动Sentinel：
redis-server  sentinel.conf --sentinel


## 故障演示

集群配置最少需要启动三个服务，我启动了4个服务分别是
127.0.0.1:26370 （redis sentinel 集群监控）
127.0.0.1:6379  （redis 主）
127.0.0.1:6380  （redis 从）
127.0.0.1:6381  （redis 从）
查看网络状态
``` bash
$ redis-cli -h 127.0.0.1 -p 26380 info Sentinel
Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=master,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=1
``` 

接着我关闭一个服务

``` bash
redis-cli -h 127.0.0.1 -p 6379 shutdow
``` 
过了一会再次查看网络
``` bash
$ redis-cli -h 127.0.0.1 -p 26380 info Sentinel
#Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=master,status=ok,address=127.0.0.1:6381,slaves=1,sentinels=1
``` 
6381变成主redis,可以write操作。
再次启动6379
``` bash
$ redis-cli -h 127.0.0.1 -p 26380 info Sentinel
#Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=master,status=ok,address=127.0.0.1:6381,slaves=2,sentinels=1
``` 
6379已不是主redis，变成了从redis,不能再write操作，只能read.

