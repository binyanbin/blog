---
title: redis学习3 数据类型
date: 2016-3-3
desc: redis 数据类型
---
redis数据类型
* String
* List
* Hashes
* Sets
* Sorted sets
<!-- more -->
## String
存值取值:
``` bash
127.0.0.1:6379>SET yanbin redis
ok
127.0.0.1:6379>GET yanbin
"redis"
```
查询是否存在及删除
``` bash
127.0.0.1:6379>SET yanbin redis
ok
127.0.0.1:6379>exists yanbin
(integer) 1
```
设置失效时间及查询失效剩余时间
``` bash
127.0.0.1:6379>SET yanbin redis
ok
127.0.0.1:6379> expire yanbin 500
(integer) 1
127.0.0.1:6379>tll yanbin 
(integer) 495
```
## List
列表是简单的字符串列表，可以排序插入顺序,可以在头部或列表的尾部Redis的列表添加元素。
``` bash
127.0.0.1:6379>rpush yanbin a b c
(integer) 3
127.0.0.1:6379>lpush yanbin 1
(integer) 1
127.0.0.1:6379>lrange 0 -1
1)"1"
2)"a"
3)"b"
4)"c"
127.0.0.1:6379>lpop yanbin
"1"
127.0.0.1:6379>rpop yanbin
"c"
127.0.0.1:6379>lrange 0 -1
1)"a"
2)"b"
127.0.0.1:6379>llen yanbin
(integer) 2
```
## Hashes
哈希值是字符串字段和字符串值之间的映射，可以表示对象的数据类型。
其实实际应用中用json格式做数据保存，可以表示对象。
``` bash
127.0.0.1:6379>hmset user:1000 username antirez birthyear 1977 verified 1
ok
127.0.0.1:6379>hget user:1000 username
"antirez"
127.0.0.1:6379>hhgetall user:1000
1)"username"
2)"antirez"
3)"birthyear"
4)"1977"
4)"verified"
4)"1"
```

## Sets
集合是一个无序的字符串合集,且不允许重复的成员。
``` bash
127.0.0.1:6379>sadd yanbin 1 2 3
(integer) 5
127.0.0.1:6379>smembers yanbin
1)"1"
2)"2"
3)"3"
127.0.0.1:6379>scard yanbin
(integer) 3
```

## Sorted sets
有序集合是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。有序集合的成员是唯一的,但分数(score)却可以重复。
``` bash
127.0.0.1:6379>zadd yanbin 1 a 2 b 3 c
(integer) 5
127.0.0.1:6379>zrange yanbin 0 -1
1)"a"
2)"b"
3)"c"
127.0.0.1:6379>zrange yanbin 0 -1 withscores
1)"a"
2)"1"
3)"b"
4)"2"
5)"c"
6)"3"
127.0.0.1:6379>zcard yanbin
(integer) 3
```