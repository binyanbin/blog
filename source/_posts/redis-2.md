---
title: redis学习2 Pub/Sub
date: 2016-3-2
desc: redis Pub/Sub 发布及订阅
---
Pub/Sub功能（means Publish,Subscribe）即发布及订阅功能.在Redis中，你可以设定对某一个key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这让我想到了一种模式:观察者模式。定义了一种一对多的依赖关系，让多个观察者对象同时监听某一发布者（主题对象或目标对象），在发布者的状态发生变化时，会通知所有观察者对象。
常见的应用场景:构建实时消息系统，比如普通的即时聊天，群聊等功能。消息队列功能.
<!-- more -->
## Redis-cli
* PSUBSCRIBE
* PUBLISH
* PUNSUBSCRIBE
* SUBSCRIBE
* UNSUBSCRIBE

### 一个客户端进行订阅操作(SUBSCRIBE)。
``` bash
$ redis-cli
127.0.0.1:6379>subscribe first second
Reading messages....
1) "subscribe"
2) "first"
3) (integer) 1
1) "subscribe"
2) "second"
3) (integer) 2
```
订阅first及second两个频道。

### 另一个客户端发布订阅消息(PUBLISH):
``` bash
$ redis-cli
127.0.0.1:6379>publish first 1
```
订阅客户端收到消息:
``` bash
$ redis-cli
127.0.0.1:6379>subscribe first second
Reading messages....
1) "subscribe"
2) "first"
3) (integer) 1
1) "subscribe"
2) "second"
3) (integer) 2
1) "message"
2) "first"
3) "1"
```

### 退订(UNSUBSCRIBE)
``` bash
$ redis-cli
127.0.0.1:6379>unsubsribe  first 
Reading messages....
1) "unsubsribe"
2) "first"
3) (integer) 0
```

### 按模式订阅和退订(PSUBSCRIBE和PUNSUBSCRIBE)
每个模式以 * 作为匹配符，比如 it* 匹配所有以 it 开头的频道( it.news 、 it.blog 、 it.tweets 等等)。 news.* 匹配所有以 news. 开头的频道( news.it 、 news.global.today 等等)，诸如此类