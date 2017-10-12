---
title: react redux入门三（路由）
date: 2017-09-20 15:22:30
tags:
- react
- router
- lazyload
- code splitting
- javascript
---

　　因为没有历史包袱，这里使用的最新的[React-Router4](https://reacttraining.com/react-router/)

```shell
npm install react-router-dom
# or
yarn add react-router-dom
```

我们来创建一个带有路由组件：

```jsx
import React from 'react';
import {
	BrowserRouter as Router,
	Route,
	Link
} from 'react-router-dom';

import App from './App';
import About from './components/About';

export const RootApp = () => (
	<Router>
		<div>
			<ul>
				<li><Link to="/">Home</Link></li>
				<li><Link to="/about">About</Link></li>
			</ul>

			<hr/>
			<Route exact path="/" component={App}/>
			<Route path="/about" component={About}/>
		</div>
	</Router>
)
```

　　运行效果类时候[这样](https://codepen.io/xbl/full/eGZMZB)，与以往对路由的认识可能不同，通常我们的路由都是使用对象的方式配置，这里是组件，对于React这样组件化的库来说其实是更加合理的，而且这样做还有另一个好处，我们可以动态创建路由，组件中可以拥有自己的路由，不再需要一个单独路由配置文件。

## 子路由

我们来创建一个带有子路由的组件：

```jsx
import React from 'react';
import {
	Route,
	Link
} from 'react-router-dom';

export const Topic = ({ match }) => (
	<div>
		<h3>{match.params.topicId}</h3>
	</div>
)

export const Topics = ({ match }) => (
	<div>
		<h2>Topics</h2>
		<ul>
			<li>
				<Link to={`${match.url}/rendering`}>
				Rendering with React
				</Link>
			</li>
			<li>
				<Link to={`${match.url}/components`}>
				Components
				</Link>
			</li>
			<li>
				<Link to={`${match.url}/props-v-state`}>
				Props v. State
				</Link>
			</li>
		</ul>
		<Route path={`${match.url}/:topicId`} component={Topic}/>
		<Route exact path={match.url} render={() => (
			<h3>Please select a topic.</h3>
		)}/>
	</div>
)

export default Topics;
```



```jsx
import React from 'react';
import {
	BrowserRouter as Router,
	Route,
	Link
} from 'react-router-dom';

import App from './App';
import About from './components/About';
import Topics from './components/Topics';

export const RootApp = () => (
	<Router>
		<div>
			<ul>
				<li><Link to="/">Home</Link></li>
				<li><Link to="/about">About</Link></li>
				<li><Link to="/topics">Topics</Link></li>
			</ul>

			<hr/>
			<Route exact path="/" component={App}/>
			<Route path="/about" component={About}/>
			<Route path="/topics" component={Topics}/>
		</div>
	</Router>
)
```

这是官方提供的例子，相对简单，但也足够啦，我们的子路由组件甚至不需要知道自己所在的路径，添加到入口的路由即可~

## lazyload / Code Splitting

　　路由的基础功能完成，接下来我们玩点高级的，也是大家都会去做的事情，将代码根据路由分割~

我们来创建一个异步组件：

```jsx
import React, { Component } from "react";

export default function asyncComponent(importComponent) {
	class AsyncComponent extends Component {
		constructor(props) {
			super(props);

			this.state = {
				component: null
			};
		}

		async componentDidMount() {
			const { default: component } = await importComponent();

			this.setState({
				component: component
			});
		}

		render() {
			const C = this.state.component;

			return C ? <C {...this.props} /> : null;
		}
	}

	return AsyncComponent;
}
```

再来修改一下RootApp.js

```jsx
import React from 'react';
import {
	BrowserRouter as Router,
	Route,
	Link
} from 'react-router-dom';

import asyncComponent from './components/AsyncComponent'
// 异步组件
const AsyncTopics = asyncComponent(() => import('./components/Topics'));
const AsyncApp = asyncComponent(() => import('./App'));
const AsyncAbout = asyncComponent(() => import('./components/About'));

export const RootApp = () => (
	<Router>
		<div>
			<ul>
				<li><Link to="/">Home</Link></li>
				<li><Link to="/about">About</Link></li>
				<li><Link to="/topics">Topics</Link></li>
			</ul>

			<hr/>
			<Route exact path="/" component={AsyncApp}/>
			<Route path="/about" component={AsyncAbout}/>
			<Route path="/topics" component={AsyncTopics}/>
		</div>
	</Router>
)
```

这样我们就完成了改造，是不是有点太简单了~

**注意**：这里并不是按照[路由官方文档](https://reacttraining.com/react-router/web/guides/code-splitting)来实现的，因为项目使用[create-react-app](https://github.com/facebookincubator/create-react-app#create-react-app-)创建，很多模块都已经集成好的。

### 更好的方式

　　现在我们非常容易的实现了lazyload，有一个小问题，当加载的组件时间过长或者失败的时候怎么处理呢？或者你也许想预加载某些组件。例如，用户在登录页面的时候你一定会想预加载主页。

　　上面提到这些边缘的情况有一个很好的组件来帮助我们完成，它就是：[react-loadable](https://github.com/thejameskyle/react-loadable)

首先我们需要安装它：

```shell
npm install --save react-loadable
# or
yarn add react-loadable
```

使用它来代替上面的`asyncComponenet`

```javascript
import React from 'react';
import {
	BrowserRouter as Router,
	Route,
	Link
} from 'react-router-dom';
import Loadable from 'react-loadable';

import Loading from './components/Loading';

const AsyncAbout = Loadable({
	loader: () => import('./components/About'),
	loading: Loading
});
const AsyncApp = Loadable({
	loader: () => import('./App'),
	loading: Loading
});
const AsyncTopics = Loadable({
	loader: () => import('./components/Topics'),
	loading: Loading
});

// perload 
AsyncTopics.preload();

export const RootApp = () => (
	<Router>
		<div>
			<ul>
				<li><Link to="/">Home</Link></li>
				<li><Link to="/about">About</Link></li>
				<li><Link to="/topics">Topics</Link></li>
			</ul>

			<hr/>
			<Route exact path="/" component={AsyncApp}/>
			<Route path="/about" component={AsyncAbout}/>
			<Route path="/topics" component={AsyncTopics}/>
		</div>
	</Router>
)
```

还有Loading组件：

```jsx
import React from 'react';

const Loading = ({isLoading, error}) => {
	// Handle the loading state
	if (isLoading) {
		return <div>Loading...</div>;
	}
	// Handle the error state
	else if (error) {
		return <div>Sorry, there was a problem loading the page.</div>;
	}
	else {
		return null;
	}
};

export default Loading;
```

搞定！So Easy！

## 参考

参考这篇文章[https://serverless-stack.com/chapters/code-splitting-in-create-react-app.html](https://serverless-stack.com/chapters/code-splitting-in-create-react-app.html)

[react-loadable官网](https://github.com/thejameskyle/react-loadable)