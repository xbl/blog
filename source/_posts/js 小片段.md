# js 小片段

### 判断空对象

``` javascript
var isEmptyObject = function(obj) {
  var t;
  for(t in obj) {
  	return false;
  }
  return true;
};
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
}
```

> 只能合并对象，不包括数组，将后面的对象统统添加到了第一个对象上

