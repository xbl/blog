---
title: js 小片段（持续更新）
date: 2015-12-03
tags: 
- javascript
---

### 判断空对象`{}`

``` javascript
var isEmptyObject = function(obj) {
  var t;
  for(t in obj) {
  	return false;
  }
  return true;
};

// 更加简单的办法
var isEmpty = Object.keys(obj).length <= 0;
```

### 判断是否是window对象

``` javascript
var isWindow = function(obj) {
  return null != obj && obj.window === window;
};
```

> 技巧：`null == undefined` ，window对象自带window属性

### 将arguments转成数组

``` javascript
// ES5
Array.apply(null, arguments);
// ES6
Array.from(arguments);
```

### 是否是Array

``` javascript
// ES5
Array.isArray(obj);
// Polyfill
if (!Array.isArray) {
  Array.isArray = function(arg) {
    return Object.prototype.toString.call(arg) === '[object Array]';
  };
}
```

### 取整

``` javascript
~~9.9 // 9
```

### 不使用函数字符串转数字

```javascript
+'9.9' // 9.9
```

### extends 合并对象

```javascript
var extends = function(dst) {
	for (var i = 1, ii = arguments.length; i < ii; i++) {
		var obj = arguments[i];
		if (obj) {
			var keys = Object.keys(obj);
			for (var j = 0, jj = keys.length; j < jj; j++) {
				var key = keys[j];
				dst[key] = obj[key];
			}
		}
	}
	return dst;
};

// es6已经实现了
Object.assign({}, obj);
```

> 只能合并对象，不包括数组，将后面的对象统统添加到了第一个对象上

### 数组拷贝

```javascript
var copyArr = [1, 2, 3].slice();
// or
var copyArr = Array.apply(Array, [1, 2, 3]);
```

### copy对象

```javascript
var obj = {a: 1, b: 2};
// 简单粗暴，不支持函数
var copyObj = JSON.parse(JSON.stringify(obj));
// es6
var copyObj = { ...obj };
```

