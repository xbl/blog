---
title: react redux入门一
date: 2017-09-03 10:42:21
tags:
- react
- javascript
---

> 　　在学习Redux之前需要先了解React，两者虽然是业界标配却不可以同时学习，建议先从[React官方文档](https://facebook.github.io/react/docs/hello-world.html)开始，对React有完整的了解再看本文会更容易。
>
> **注意**：以下的所有代码都基于现在的新版本`15.6.1`，可能未来会不太一样~

##  React 例子

　　在React中有一个很重要的概念`state`，在每一个组件都有一个自己的`state`，相当于组件的私有变量，通过改变state会触发组件render()，我们来看一个简短的例子：
<iframe height='265' scrolling='no' title='React1' src='//codepen.io/xbl/embed/ZJPYqE/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/xbl/pen/ZJPYqE/'>React1</a> by javascript.xie (<a href='https://codepen.io/xbl'>@xbl</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
　　例子非常简单，几乎没什么可讲的，如果你第一次创建React应用建议使用[create-react-app](https://github.com/facebookincubator/create-react-app#create-react-app-)来帮助你，会省去很多繁琐的配置。

##  Redux-同步action 

　　Redux 并不包含[react-redux](https://github.com/reactjs/react-redux)绑定库，需要独立安装：

```shell
npm install --save react-redux
// or
yarn add react-redux
```
　　Redux的文档并不容易理解，`action`、`reducer`和`store`一堆的概念，你可以理解为订阅模式或者是事件模式，`store.dispatch`负责触发`action`事件，`action`可以携带数据，再由`reducer`将数据进行整理，更新对应组件的`state`。流程我们看懂了，再修改一下上面的例子：

<iframe height='265' scrolling='no' title='React-Redux-Sync2' src='//codepen.io/xbl/embed/vJPNqK/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/xbl/pen/vJPNqK/'>React-Redux-Sync2</a> by javascript.xie (<a href='https://codepen.io/xbl'>@xbl</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

　　例子中使用了一些`react-redux`的API，建议大家仔细学习一下[文档](https://github.com/reactjs/react-redux/blob/master/docs/api.md#api)。`connect()`进行了多次重载，这里只接收了一个参数`mapStateToProps`，文档中第二个参数是`mapDispatchToProps`，市面上的写法有很多，我们来一起看一下：

`mapDispatchToProps` 参数可以是**对象**或者**函数**

如果是对象，这个对象必须由Action组成，会遍历此对象合并到`props`。

```javascript
const mapDispatchToProps = {
	initData: fetchArrAction // fetchArrAction是我们的Action
};
// 或者
const mapDispatchToProps = {
 	fetchArrAction
};
```

如果是函数，第一个参数会传入一个`dispatch`，返回一个对象，会将返回的对象遍历合并到`props`，这也是最常见的写法：

 ```javascript
const mapDispatchToProps = (dispatch) => {
 	return {
 		initData: () => {
 			dispatch(fetchArr());
 		}
 	}
};
 ```


如果是函数且有两个参数，那么第一个参数仍然是`dispatch`，第二个参数容器组件的`props`，通常写作`ownProps`。

如果没有传入`mapDispatchToProps`这个参数，那么会在组件的`props`中注入`dispatch`这个方法。

>  所以上面的例子就是最后一种。

### 【容器组件】和【展示组件】 

　　例子中还引入了一个新的概念【容器组件】和【展示组件】，简单的理解【容器组件】都使用`react-redux`的`connect()`方法来生成，为展示组件提供数据和方法。

【展示组件】就直白的多，仅仅只做展示。这个概念会决定你能否设计出合理的组件结构。

## 异步的action

　　目前的`action`是同步的，必须返回一个对象，一个真正的应用必然会有网络请求，那我们如何与网络请求结合起来呢？标准的做法是使用[Redux Thunk 中间件](https://github.com/gaearon/redux-thunk)。

```shell
npm install redux-thunk --save
// or 
yarn add redux-thunk
```

　　对于Redux中间件本文不会深入讨论，大家有个认识即可。为了例子足够简单，我们使用`setTimeout`来代替网络请求：
<iframe height='265' scrolling='no' title='React-Redux-Async' src='//codepen.io/xbl/embed/JyzdxZ/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/xbl/pen/JyzdxZ/'>React-Redux-Async</a> by javascript.xie (<a href='https://codepen.io/xbl'>@xbl</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

　　我们只增加了一个`fetchArr()`异步`action`，对组件没有任何修改，是不是很happy！组件的测试不需要修改，只要增加action的测试就ok了！

> 　　真正的网络请求要比这个例子复杂的多，从【loading】到【成功】或是【失败】都需要3个`action`才能完成。

## 总结

　　本文是为了让初学`React`和`Redux`的同学有一个概念，以最简单的demo来讲解他们是如何工作的，这仅仅是冰山一角，后面我们会逐步深入更多内容。同时也建议大家在学习本文之后再仔细学习[Redux文档](http://redux.js.org/)（下方有中文版本），理解起来会更加容易和深刻。

## 参考

https://facebook.github.io/react/docs/hello-world.html

[Redux 中文文档](http://github.com/camsong/redux-in-chinese)

[译文《容器组件和展示组件》原作者：Dan Abramov](http://www.jianshu.com/p/6fa2b21f5df3)

http://cn.redux.js.org/docs/advanced/ExampleRedditAPI.html

