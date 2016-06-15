---
title: redis学习7 主从复制
date: 2016-3-9
desc: redis  主从复制
---
redis集群有二种方式，一种分区，一种主从复制。
redis的主从复制功能非常强大，一个master可以拥有多个slave，而一个slave又可以拥有多个slave，如此下去，形成了强大的多级服务器集群架构。下面是关于redis主从复制的一些特点：
* master可以有多个slave
* 除了多个slave连到相同的master外，slave也可以连接其他slave形成图状结构
* 主从复制不会阻塞master。也就是说当一个或多个slave与master进行初次同步数据时，master可以继续处理client发来的请求。相反slave在初次同步数据时则会阻塞不能处理client的请求。
* 主从复制可以用来提高系统的可伸缩性,我们可以用多个slave专门用于client的读请求，比如sort操作可以使用slave来处理。也可以用来做简单的数据冗余
*可以在master禁用数据持久化，只需要注释掉master配置文件中的所有save配置，然后只在slave上配置数据持久化。
<!-- more -->
## 配置
* 把安装好的redis做master,然后copy一份当slave。然后修改slave配置
port 6379  修改为port 6380
slaveof 127.0.0.1 6379  (映射到主服务器上)
* 配置完成以后可以在二个服务中分别进行set和get操作来看效果,master可以get和set操作,slave上能get操作不能set，也就是说master可读可写，slave只能读。你在master上set数据，slave上可以查询得到。
* 在master和slave分别执行info命令，查看结果如下：
![master](/img/info1.png)
![slave](/img/info2.png)
