---
title: Etcd3 入门
date: 2017-04-14 18:44:48
tags: 
---
​	对于[Etcd](https://coreos.com/etcd/)不了解的同学可以先看一下文章最后的介绍，Etcd是一个用于配置共享和服务发现的键值对数据库，使用的Raft算法实现的分布式一致性。

## 安装并启动

​	安装参考官网文档就好了，启动起来就好了。

## API使用

​	Etcd3 版本使用了gRPC作为他的消息协议，所以我们并不需要其他lib，只要支持gRPC就ok了，同时etcd提供了命令行工具[etcdctl](https://github.com/coreos/etcd/tree/master/etcdctl) ，方便我们在命令行使用。

​	gRPC接口文档：https://github.com/coreos/etcd/blob/master/Documentation/dev-guide/api_reference_v3.md

​	首先将https://github.com/coreos/etcd/tree/master/etcdserver/etcdserverpb文件夹copy到项目中。

```javascript
const grpc = require('grpc');
 
const etcdserverpb = grpc.load('./etcdserverpb/rpc.proto').etcdserverpb;
const etcdHost = 'localhost:2379';
var insecure = grpc.credentials.createInsecure();
// kv 存储客户端
var clientKV = new etcdserverpb.KV(etcdHost, insecure);
// 监听客户端
var clientWatch = new etcdserverpb.Watch(etcdHost, insecure);
// 过期客户端
var clientLease = new etcdserverpb.Lease(etcdHost, insecure);
```

### get 、put

```javascript
// get 
clientKV.range({
	key: new Buffer('foo')
}, function (err, res) {
	if (res.kvs.length) {
		var value = res.kvs[0].value.toString();
		console.log(value);
	}
});

// put
clientKV.put({
	key: new Buffer('hello'),
	value: new Buffer('world!!!'),
}, function (err, res) {
	console.log(arguments);
});
```

### delete

```javascript
clientKV.deleteRange({
	key: new Buffer('id'),
}, function (err, resp) {
	// 0 或者 1
	console.log(~~(resp.deleted));
});
```

### 根据前缀查询

```javascript
// 根据前缀查询
var perfix = 'hello';
clientKV.range({
	key: Buffer.from(perfix),
	range_end: Buffer.alloc(perfix.length + 1, perfix)
}, function (err, res) {
	var kvs = res.kvs;
	kvs.forEach(function (obj, i) {
		console.log(obj.key + ':' + obj.value.toString());
	});
});
```

### 根据前缀删除

```javascript
// 根据前缀删除
var perfix = 'hi';
clientKV.deleteRange({
	key: Buffer.from(perfix),
	range_end: Buffer.alloc(perfix.length + 1, perfix)
}, function (err, resp) {
	console.log(~~(resp.deleted));
});
```

### 监听key

```javascript
var watcher = clientWatch.watch();
watcher.write({
	create_request: {
		key: new Buffer('hello')
	}
});
watcher.on('data', function(res) {
	var events = res.events;
	if (events && events.length) {
		var value = events[0].kv.value.toString();
		console.log(value);
	}
});
```

### 从某一历史版本监听

```javascript
// 从某一历史版本监听
var watcher = clientWatch.watch();
watcher.write({
	create_request: {
		key: new Buffer('hello'),
		start_revision: 1
	}
});
watcher.on('data', function(res) {
	var events = res.events;
	events.forEach(function (obj, i) {
		if(obj.type == 'PUT')
			console.log(obj.kv.value.toString());
	});
});
```

### 事务

```javascript
clientKV.txn({
	compare: [
		{
			// result 是条件，是否等于、大于、小于、不等于
			// CompareResult {
			// 	EQUAL = 0;
			// 	GREATER = 1;
			// 	LESS = 2;
			// 	NOT_EQUAL = 3;
			// }
			result: 'EQUAL',
			// target 比较值、版本、或者是状态
			// CompareTarget {
			// 	VERSION = 0;
			// 	CREATE = 1;
			// 	MOD = 2;
			// 	VALUE= 3;
			// }
			target: 'VALUE',
			key: new Buffer('name'),
			value: new Buffer('abc')
		}
    ],
    // 事务成功做什么操作
	success: [
		{
			request_put: {
				key: new Buffer('id'),
				value: new Buffer('789'),
				lease: 0
			}
		},
		{
			request_put: {
				key: new Buffer('uid'),
				value: new Buffer('yoyoyo'),
				lease: 0
			}
		}
    ],
    failure: []
}, function (err, resp) {
	console.log(err);
	// 如果事物成功则输出
	if(resp.succeeded) {
		console.log(resp);
	}
});
```

### 设置过期

```javascript
// 先申请一个十秒钟后过期id
clientLease.leaseGrant(10, function (err, lease) {
	console.log(lease.ID);
	// 设置需要过期的key-value
	clientKV.put({
		key: new Buffer('hello'),
		value: new Buffer('world123678!!!'),
		lease: lease.ID
	}, function (err, res) {
      	console.log(res);
	});
});

clientLease.leaseGrant(100, function (err, lease) {
	console.log(lease.ID);
	// 设置需要过期的key-value
	clientKV.put({
		key: new Buffer('hello'),
		value: new Buffer('world100!!!'),
		lease: lease.ID
	}, function (err, res) {
		console.log(arguments);
		// 将过期id释放掉，同时key-value过期
		clientLease.leaseRevoke(lease.ID, function (err, res) {
			console.log(arguments);
		});
	});
});
```

### 离线事件

```javascript
// 创建10秒钟过期
clientLease.leaseGrant(10, function (err, lease) {
	console.log(lease.ID);
	// 创建一个连接
	var stream = clientLease.leaseKeepAlive();
	// 回写过期id
	stream.write(lease.ID);

	// 设置需要过期的key-value
	clientKV.put({
		key: new Buffer('hi'),
		value: new Buffer('online!'),
		lease: lease.ID
	}, function (err, res) {
		// console.log(arguments);
	});
	// 心跳
	setInterval(function () {
		stream.write(lease.ID);
	}, 3000);
});
```







## 参考

参考资料：https://skyao.gitbooks.io/leaning-etcd3/content/documentation/

API：https://github.com/coreos/etcd/blob/master/Documentation/dev-guide/api_reference_v3.md 

ETCD3介绍：http://dockone.io/article/1494

应用场景：http://www.infoq.com/cn/articles/etcd-interpretation-application-scenario-implement-principle