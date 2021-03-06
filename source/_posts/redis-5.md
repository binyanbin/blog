---
title: redis学习5 lua脚本
date: 2016-3-7
desc: redis  lua
---
redis内置的 Lua 解释器，可以使用lua对redis进行操作。使用脚本有以下好处:
* 减少网络开销:批量执行redis命令。
* 原子性操作:Redis也保证脚本会以原子性的方式执行:当某个脚本正在运行的时候，不会有其他脚本或Redis 命令被执行。
* 复用：客户端发送的脚本会永久存储在Redis中，意味着其他客户端可以复用这一脚本而不需要使用代码完成同样的逻辑。
<!-- more -->
## 使用
* lua脚本
lua是一个很容易嵌入其它语言中使用的语言。很多应用程序、游戏使用LUA作为自己的嵌入式脚本语言，以此来实现可配置性、可扩展性。查看[lua5.1在线中文用户手册](http://manual.luaer.cn/)。
* Eval
通过redis-cli客户端单独调用Lua脚本文件，格式如下:
redis-cli --eval myscript.lua [key ...] arg [arg ...]
``` lua
--[[ 
限制一定时间内的调用次数 
KEYS[1]:key 
ARGV[1]:存在时长
ARGV[2]:调用次数
]]
local times = redis.call('incr',KEYS[1])

if times == 1 then
    redis.call('expire',KEYS[1], ARGV[1])
end

if times > tonumber(ARGV[2]) then
    return 0
end
return 1
```
调用 redis-cli --eval d:\test.lua test:127.0.0.1 , 10 3

通过EVAL命令执行脚本，格式如下:
EVAL script numkeys key [key ...] arg [arg ...]
``` bash
127.0.0.1:6379>eval "local times = redis.call('incr',KEYS[1]);if times == 1 then redis.call('expire',KEYS[1], ARGV[1]);end;if times > tonumber(ARGV[2]) then return 0;end;return 1" 1 test:127.0.0.1 , 10 3
(integer) 1
127.0.0.1:6379>get test:127.0.0.1
"1"
```
redis.call lua脚本通过redis.call调用redis命令。

 总结:我们可以通过Lua来实现很多功功能:用Lua来封装复杂了Redis操作的业务;计数，统计，分析，收集数据;实现业务操作事务控制等等。

