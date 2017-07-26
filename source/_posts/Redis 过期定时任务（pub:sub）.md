---
title: Redis 过期定时任务（pub/sub）
date: 2016-10-25 18:44:48
tags: 
---

> ​	因为当爸爸了所以都没有时间一直没有自己的时间做一个完整的Demo，今天算是做一个弥补。

## 配置

因为开启键空间通知功能需要消耗一些 CPU ， 所以在默认配置下， 该功能处于关闭状态。

可以通过修改 `redis.conf` 文件， 或者直接使用 `CONFIG SET notify-keyspace-events AKE ` 命令来开启或关闭键空间通知功能：

- 当 `notify-keyspace-events` 选项的参数为空字符串时，功能关闭。
- 另一方面，当参数不是空字符串时，功能开启。

`notify-keyspace-events` 的参数可以是以下字符的任意组合， 它指定了服务器该发送哪些类型的通知：

| 字符   | 发送的通知                                    |
| ---- | ---------------------------------------- |
| `K`  | 键空间通知，所有通知以 `__keyspace@__` 为前缀          |
| `E`  | 键事件通知，所有通知以 `__keyevent@__` 为前缀          |
| `g`  | `DEL` 、 `EXPIRE` 、 `RENAME` 等类型无关的通用命令的通知 |
| `$`  | 字符串命令的通知                                 |
| `l`  | 列表命令的通知                                  |
| `s`  | 集合命令的通知                                  |
| `h`  | 哈希命令的通知                                  |
| `z`  | 有序集合命令的通知                                |
| `x`  | 过期事件：每当有过期键被删除时发送                        |
| `e`  | 驱逐(evict)事件：每当有键因为 `maxmemory` 政策而被删除时发送 |
| `A`  | 参数 `g$lshzxe` 的别名                        |

输入的参数中至少要有一个 `K` 或者 `E` ， 否则的话， 不管其余的参数是什么， 都不会有任何通知被分发。

举个例子， 如果只想订阅键空间中和列表相关的通知， 那么参数就应该设为 `Kl` ， 诸如此类。

将参数设为字符串 `"AKE"` 表示发送所有类型的通知。

## 客户端代码

​	因为简单所以使用了NodeJs作为范例：

```javascript
var redis = require("redis");
var sub = redis.createClient({host: 'localhost', port: 6379});
sub.on('connect', function () {
    // 设置key-value
    // sub.set("foo", "123", redis.print);
    // 设置过期时间
    // sub.expire('foo', 10);
    // 同时设置value和过期时间
    sub.setex('foo', 10, 'value', redis.print);
  
	// 触发订阅回调
    sub.on('psubscribe', redis.print);
    // 订阅消息回调
    sub.on('pmessage', function (channel, message, key) {
        console.log('key:', key, ', message:' + message);
    });
    // 所有事件
    // sub.psubscribe('__key*__:*');
    // 只订阅过期事件
    sub.psubscribe('__key*__:expired');
});
```

输出:

```shell
key: foo , message:__keyevent@0__:expired
```

## 总结

​	这种方法非常简单，多个客户端订阅过期事件会同时接收到消息，还需要自动选举一个客户端去执行，否则就乱套了，而且该方法不能保证消息的稳定性，如果客户端不在线消息就会丢失。

## 参考：

http://redisdoc.com/topic/notification.html

http://stackoverflow.com/questions/29833939/redis-keyspace-event-not-firing

https://github.com/NodeRedis/node_redis

https://developer.mozilla.org/en-US/docs/Mozilla/Redis_Tips

https://help.aliyun.com/document_detail/26366.html?spm=5176.doc26367.6.141.VKMr9J