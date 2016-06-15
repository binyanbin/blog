---
title: redis学习4 事务
date: 2016-3-5
desc: redis  事务
---
事务可以一次执行多个命令， 并且带有以下两个重要的保证：
* 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
* 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。
<!-- more -->
## 命令
* multi 开启事务
* discard 放弃事务
* WATCH 事务执行条件
* 事务执行

简单事务:
``` bash
127.0.0.1:6379>multi
OK
127.0.0.1:6379>set yanbin 1
QUEUED
127.0.0.1:6379>exec
1) OK
127.0.0.1:6379>get yanbin
"1"
```
放弃事务:
``` bash
127.0.0.1:6379>multi
OK
127.0.0.1:6379>set yanbin 1
QUEUED
127.0.0.1:6379>discard
OK
127.0.0.1:6379>exec
(error) ERR EXEC without MULTI
127.0.0.1:6379>get yanbin
(nil)
```
事务条件:WATCH 使得 EXEC 命令需要有条件地执行： 事务只能在所有被监视键都没有被修改的前提下执行， 如果这个前提不能满足的话，事务就不会被执行。
``` bash
127.0.0.1:6379>watch yanbin
OK
127.0.0.1:6379>set yanbin 1
OK
127.0.0.1:6379>multi
OK
127.0.0.1:6379>incr yanbin
QUEUED
127.0.0.1:6379>exec
(nil)
127.0.0.1:6379>get yanbin
"1"
```