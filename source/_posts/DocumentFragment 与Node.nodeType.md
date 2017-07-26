---
title: DocumentFragment 与Node.nodeType
date: 2016-02-28
tags: 
---
> ​	一直比较懒，懒得去写一些文章，以后尽量把学习到的东西记录一下吧，最近在读一个牛人写的框架，其中知道了好多`javascript`基础，真是惭愧做了这么多年前端，仍然有很多是自己不知道的知识点。

## DocumentFragment

​	今天先来说说`DocumentFragment`对象，其实就是一个代码片段对象，当我们要向DOM中批量插入一段`HTML`代码片段的时候就可以使用它了，我们来看个例子：

``` html
<!DOCTYPE html>
<html>
<head>
	<title></title>
	<script type="text/javascript">
		window.onload = function() {
			var ul = document.getElementById('test');
			var documentFrament = document.createDocumentFragment();

			["Internet Explorer", "Mozilla Firefox", "Safari", "Chrome", "Opera"].forEach(function(e) {
				var li = document.createElement("li");
				li.textContent = e;
				documentFrament.appendChild(li);
			});
			// 在这里才真正的添加到DOM上
			ul.appendChild(documentFrament);
		};
	</script>
</head>
<body>
	<ul id="test"></ul>
</body>
</html>
```

​	在使用`jQuery`时候通常会创建一个`div`，将其他节点添加到`div`上，使用`documentFrament`就不会多出一个`div`，可以直接被父节点添加，那么当我们假设自己也在写框架的时候，如何判断使用者给我们传过来的参数是一个真正的到`DOM`还是一个`documentFrament`对象呢？接下来我们来说说`Node.nodeType`;

## Node.nodeType

​	Node是一个接口，`Document`，`Element`，`CharacterData`，`DocumentFrament`，`DocumentType` 等等都是继承了Node接口，nodeType 是Node的一个属性，返回一个整形的常量：

| Name                        | Value |
| --------------------------- | ----- |
| ELEMENT_NODE                | 1     |
| TEXT_NODE                   | 3     |
| PROCESSING_INSTRUCTION_NODE | 7     |
| COMMENT_NODE                | 8     |
| DOCUMENT_NODE               | 9     |
| DOCUMENT_TYPE_NODE          | 10    |
| DOCUMENT_FRAGMENT_NODE      | 11    |

​	可以看到当一个Element的nodeType等于1，而documentFrament.nodeType等于11，我们要判断究竟是什么就非常容易了：

``` javascript
document.createElement('span').nodeType // 1
document.createDocumentFragment().nodeType // 11
document.createComment('哈哈哈~~~').nodeType // 8
```

今天先记录到这，欢迎各位看官指正批评！

## 参考

https://developer.mozilla.org/en-US/docs/Web/API/Document/createDocumentFragment

https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType