---
title: Vue 源码 Observe Array 分析
date: 2016-03-30
tags: 
---
> ​	Vue 已经火的不行不行啦，读代码的同学也越来越多，不过好多同学讲的都是getter/setter ，对于监听数组变化代码说的就比较少，所以今天来分析一下，自己做一个记录，代码和注释根据我自己的理解有过修改，不喜勿喷哈！

## 原理

​	数组对象不可以通过`Object.defineProperty`来监听变化，Vue的做法是重写`push`、`pop`、`shift`、`unshift`、`splice`、`sort`、`reverse`这些方法，当调用上面方法的时候先调用Array的原生方法，随后触发`notice`方法，通过`Object.create(Array.prototype)`创建一个新的`prototype对象`，再将这个`prototype`重新绑定到数组对象上；

## 复制数组protoype

``` javascript
var arrayProto = Array.prototype;
var arrayMethods = Object.create(arrayProto);
```

## 重写数组方法

``` javascript
[
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
].forEach(function (method) {
  // 缓存Array的方法
  var original = arrayProto[method]; 
  Object.defineProperty(arrayMethods, method, {
    value: function () {
      // avoid leaking arguments:
      // http://jsperf.com/closure-with-arguments
      var i = arguments.length;
      var args = new Array(i);
      while (i--) {
        args[i] = arguments[i];
      }
      // 执行Array的方法，并返回结果
      var result = original.apply(this, args);
      // 新插入的对象
      var inserted;
      switch (method) {
          // 向数组后添加
        case 'push':
          inserted = args;
          break;
          // 向数组前添加
        case 'unshift':
          inserted = args;
          break;
          // 删除数组元素，同时添加新元素
        case 'splice':
          // 只截取新添加元素
          inserted = args.slice(2);
          break;
      }
      // if(inserted) 
      // 	console.log('将更新的对象转成Observe对象');
      console.log('调用方法：', method, '添加元素：', inserted, '返回值：', result);
      return result;
    },
    enumerable: true,
    writable: true,
    configurable: true
  });
});
	
```



## 添加$set、$remove方法

``` javascript
/**
 * 扩展数组$set 方法
 * Swap the element at the given index with a new value
 * and emits corresponding event.
 *
 * @param {Number} index
 * @param {*} val
 * @return {*} - 返回被替换的元素
 */
Object.defineProperty(arrayProto, '$set', {
  value: function (index, val) {
    // 当下标大于等于当前数组的长度则修改数组长度为index + 1
    if (index >= this.length) {
      this.length = Number(index) + 1;
    }
    // 修改下标元素替换成新的val，
    // 此处调用this.splice而不是使用this[index] = val
    // 可以得到利用变形的splice来触发notice，同时兼容未扩展的原生数组
    return this.splice(index, 1, val)[0];
  }
});

/**
 * Manual indexOf because it's slightly faster than
 * native.
 *
 * @param {Array} arr
 * @param {*} obj
 */
function indexOf (arr, obj) {
  var i = arr.length;
  while (i--) {
    if (arr[i] === obj) return i;
  }
  return -1;
};

/**
 * 扩展数组$remove方法
 * 
 * @param {*} item
 */
Object.defineProperty(arrayProto, '$remove', {
  value: function (item) {
    if(!this.length)
      return ;
    var index = indexOf(this, item);
    if(index > -1) {
      return this.splice(index, 1);
    }
  }
});
```

## 将新prototype绑定到数组对象上

``` javascript
/**
 * 将Array的方法重新bind到arr这个变量上
 */
function copyAugment (arr, proto) {
  // 
  var keys = Object.getOwnPropertyNames(proto);
  for (var i = 0, l = keys.length; i < l; i++) {
    var key = keys[i];
    Object.defineProperty(target, key, {
      value: src[key]
    });
  }
}
	
```

## 完整代码

``` javascript
(function () {
	'use strict';
	var arrayProto = Array.prototype;
	var arrayMethods = Object.create(arrayProto);
	
	[
		'push',
		'pop',
		'shift',
		'unshift',
		'splice',
		'sort',
		'reverse'
	].forEach(function (method) {
		// 缓存Array的方法
		var original = arrayProto[method]; 
		Object.defineProperty(arrayMethods, method, {
			value: function () {
				// 将arguments 转成数组
				// var args = Array.apply(null, arguments);
				// 为什么这个方法不好使呢？？？谁能告诉我！！！
				// http://jsperf.com/aurgments2array2-1
				
				// avoid leaking arguments:
				// http://jsperf.com/closure-with-arguments
				var i = arguments.length;
				var args = new Array(i);
				while (i--) {
					args[i] = arguments[i];
				}
				// 执行Array的方法，并返回结果
				var result = original.apply(this, args);
				// 新插入的对象
				var inserted;
				switch (method) {
					// 向数组后添加
					case 'push':
						inserted = args;
						break;
					// 向数组前添加
					case 'unshift':
						inserted = args;
						break;
					// 删除数组元素，同时添加新元素
					case 'splice':
						// 只截取新添加元素
						inserted = args.slice(2);
						break;
				}
				// if(inserted) 
				// 	console.log('将更新的对象转成Observe对象');
				console.log('调用方法：', method, '添加元素：', inserted, '返回值：', result);
				return result;
			},
			enumerable: true,
			writable: true,
			configurable: true
		});
	});
	
	/**
	 * 扩展数组$set 方法
	 * Swap the element at the given index with a new value
	 * and emits corresponding event.
	 *
	 * @param {Number} index
	 * @param {*} val
	 * @return {*} - 返回被替换的元素
	 */
	Object.defineProperty(arrayProto, '$set', {
		value: function (index, val) {
			// 当下标大于等于当前数组的长度则修改数组长度为index + 1
			if (index >= this.length) {
				this.length = Number(index) + 1;
			}
			// 修改下标元素替换成新的val，
			// 此处调用this.splice而不是使用this[index] = val
			// 可以得到利用变形的splice来触发notice，同时兼容未扩展的原生数组
			return this.splice(index, 1, val)[0];
		}
	});
	
	/**
	 * Manual indexOf because it's slightly faster than
	 * native.
	 *
	 * @param {Array} arr
	 * @param {*} obj
	 */
	function indexOf (arr, obj) {
		var i = arr.length;
		while (i--) {
			if (arr[i] === obj) return i;
		}
		return -1;
	};
	
	/**
	 * 扩展数组$remove方法
	 * 
	 * @param {*} item
	 */
	Object.defineProperty(arrayProto, '$remove', {
		value: function (item) {
			if(!this.length)
				return ;
			var index = indexOf(this, item);
			if(index > -1) {
				return this.splice(index, 1);
			}
		}
	});
	
	/**
	 * 将Array的方法重新bind到arr这个变量上
	 */
	function copyAugment (arr, proto) {
		// 
		var keys = Object.getOwnPropertyNames(proto);
		for (var i = 0, l = keys.length; i < l; i++) {
			var key = keys[i];
			Object.defineProperty(arr, key, {
				value: proto[key]
			});
		}
	}
	
	
	var arr = [1, 2, 3];
	// 将Array的方法重新bind到arr这个变量上
	copyAugment(arr, arrayMethods);
	
	
	arr.push(4);
	console.log(arr);
	arr.splice(3, 1, 6);
	console.log(arr);
})();
```



## 结束语

​	以上代码是根据Vue 1.0.20版本摘抄而来，感谢Vue作者，人长的帅，又写得一手好代码，希望我的粗浅分析能够给其他同学带来帮助；

[Vue源码](https://github.com/vuejs/vue)

[Vue 官网](http://vuejs.org.cn/)