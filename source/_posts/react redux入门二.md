---
title: react redux入门二（单元测试）
date: 2017-09-18 16:25:18
tags:
- javascript
- react
- 单元测试
---

　　前面我们讲过[react redux入门的第一篇](/2017/09/03/react%20redux入门一/)，今天我们来分享一下如何写redux单元测试，如果你使用[create-react-app](https://github.com/facebookincubator/create-react-app#create-react-app-)来创建项目，那么你的测试环境已经搭建好了，项目会以[Jest](http://facebook.github.io/jest/)做为runner。

先看看我们的项目结构：

```
└── src
    └── actions
        └── index.js
    └── components
        └── List.js
    └── containers
        └── ListContainer.js
    └── reducers
        └── arr.js
	└── App.css
	└── App.js
	└── index.js
    └── ...
```

上一篇我们了解到`reducer`都是纯函数，这部分测试应该是最简单哒，我们首先运行命令：

```shell
npm run test // 运行测试
```

## reducer 测试

在`reducers`目录下创建`arr.spec.js`

```javascript
// arr.js
export default (state = [], action) => {
	switch (action.type) {
		case 'INIT_ARR':
			return state.concat(action.arr);
		default:
			return state;
	}
};

// arr.spec.js
import arr from './arr';

describe('arr reducer', () => {
	it('action is {}, should return []', () => {
		expect(arr(undefined, {}))
			.toEqual([]);
	});

	it('action.type is INIT_ARR, should return data', () => {
		const data = [
			{ name: 1},
			{ name: 2},
		];
		expect(arr(undefined, { type: 'INIT_ARR', arr: data }))
			.toEqual(data);
	});
});
```

为了方便看我将两个文件都贴在这里，对于纯函数我们只要针对不同的输入输出进行验证即可。

## 同步Action测试 

与`reducers`差不多，同步的Action也很简单：

```javascript
// index.js
export function initArr(arr) {
	return {
		type: 'INIT_ARR',
		arr
	}
}

// index.spec.js
import * as actions from './index';

describe('actions', () => {
	it('initArr() should init arr', () => {
		var arr = [{name: 1}, {name: 2}];
		expect(actions.initArr(arr))
			.toEqual({
				type: 'INIT_ARR',
				arr
			});
	});
});
```

## 异步Action测试

我们先来写一个最简单的异步action，返回一个`Promise`对象:

```javascript
export function initArr(arr) {
	return {
		type: 'INIT_ARR',
		arr
	}
}

export function fetchArr() {
	return (dispatch) => {
		return Promise.resolve('').then(() => {
			dispatch(initArr([{name: 'hi'}, {name: 'hello'}, {name: '你好!'}]));
		});
	};
}
```

这里首先需要Mock一个`store`，这里用到了[redux-mock-store](http://arnaudbenard.com/redux-mock-store/)

```Shell
npm install redux-mock-store --save-dev
# or
yarn add redux-mock-store -D
```

测试代码如下：
```javascript
import configureStore from 'redux-mock-store';
import thunkMiddleware from 'redux-thunk';
import * as actions from './index';

// ...

describe('async actions', () => {
	const middlewares = [thunkMiddleware];
	const mockStore = configureStore(middlewares);

	it('fetchArr() should init arr', () => {
		const initialState = {};
		const store = mockStore(initialState);
		const arr = [{name: 'hi'}, {name: 'hello'}, {name: '你好!'}]
	    // 通过dispatch 异步action之后触发同步的action
		store.dispatch(actions.fetchArr())
			.then(() => {
				const actionArr = store.getActions();
				expect(actionArr[0]).toEqual(actions.initArr(arr));
                expect(actionArr).toEqual(expect.arrayContaining([actions.initArr(arr)]));
				expect(actionArr.length).not.toBe(0);
				expect(actions.initArr(arr)).toEqual(actionArr[0]);
			});
	});	
});
```

这样我们就完成了一个异步action的单元测试，但这还不够，我们需要一个具有网络请求的Action。

### 使用原生的[Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)

```javascript
// 异步action
export function fetchArr() {
	return (dispatch) => {
		return fetch('http://localhost/todos')
			.then((res) => res.json())
			.then((json) => dispatch(initArr(json)))
			.catch(ex => {
				console.log(ex);
			});
	};
}
```

这里使用了`Fetch`进行网络请求，Action的代码就是这样，所以我们需要`Mock`一个`fetch`对象，我们来添加一个Fetch 的mock测试吧~

```javascript
it('fetch() mock', () => {
  let arr = [{name: 1}, {name: 2}],
      body = JSON.stringify(arr),
      resp = new window.Response(body, {
        status: 200,
        statusText: '',
        headers: {
          'Content-type': 'application/json'
        }
      });

  // https://developer.mozilla.org/en-US/docs/Web/API/Response/Response
  window.fetch = jest.fn().mockImplementation(() => Promise.resolve(resp));
  fetch('/abc.json')
    .then((resp) => resp.json())
    .then((json) => {
    expect(json).toEqual(arr);
  })
  .catch(err => console.log(err));
});
```

好了，这样我们就模拟了一个`fetch`函数，还返回了一个`Response`对象，可是...`fetch`在低版本的浏览器支持的还不够好，虽然这段Mock的代码代码可以抽离成一个单独的函数，但还是不够方便，有没有兼容性更好、更简洁的方法呢？往下看吧~

### 使用isomorphic-fetch

使用[isomorphic-fetch](https://www.npmjs.com/package/isomorphic-fetch)模块来完成兼容性的部分

```shell
npm install --save isomorphic-fetch es6-promise
# or
yarn add isomorphic-fetch es6-promise 
```

那么我们需要在代码中引入一下即可：

```javascript
import fetch from 'isomorphic-fetch';
```

Mock的部分我们使用[Nock](https://github.com/node-nock/nock)模块来搞定（**注意**：`Nock`不能Mock原生的`Fetch`）：

```shell
npm install nock --save-dev
# or
yarn add nock -D
```

我们来看一个完整的测试：

```javascript
it('fetchArr()2 should init arr', () => {
  const initialState = {};
  const store = mockStore(initialState);
  const arr = [{name: 1}, {name: 2}];

  nock('http://localhost')
    .get('/todos')
    .reply(200, arr , { 'Content-Type': 'application/json' });

  store.dispatch(actions.fetchArr())
    .then(() => {
      const actionArr = store.getActions();
      expect(actionArr).toEqual(expect.arrayContaining([actions.initArr(arr)]));
      expect(actionArr.length).not.toBe(0);
      expect(actions.initArr(arr)).toEqual(actionArr[0]);
    });
});	
```

这样是不是简洁多了呢~接下来就是组件的单元测试啦。

## Component 测试

　　Component 测试，我们需要[Enzyme](http://airbnb.io/enzyme/index.html)模块，Enzyme是airbnb开发的React组件测试工具，通过模仿jQuery的API操作DOM。

我们来看一个展示组件：

```javascript
// List.js
import React from 'react';
import PropTypes from 'prop-types';

const List = ({ arr, clickHandler }) => (
	<ul>
		{ arr.map(person => 
			<li key={person.name} onClick={ clickHandler && clickHandler.bind(this, person)}>{person.name}</li>
		)}
	</ul>
)

List.propTypes = {
	arr: PropTypes.array.isRequired
};

export default List;
```

再来看看测试是如何写的：

```javascript
// List.spec.js
import React from 'react';
import { shallow, mount } from 'enzyme';

import List from './List';

describe('List component', () => {
	it('renders without crashing', () => {
		shallow(<List arr={ [] }/>);
	});

	it('renders 2 li', () => {
		const arr = [
			{ name: 1},
			{ name: 2},
		];
		const wrapper = shallow(<List arr={ arr }/>);
		expect(wrapper.find('li').length).toEqual(2);
	});

	it('renders 2 li', () => {
		const arr = [
			{ name: 1},
			{ name: 2},
		];
		// 注意这里是mount
		const wrapper = mount(<List arr={ arr }/>);
		expect(wrapper.props().arr).toEqual(arr);
		expect(wrapper.find('li').length).toEqual(2);
		// 找到第一个li
		expect(wrapper.find('li').first().html())
			.toEqual('<li>1</li>');
		expect(wrapper.find('li').at(0).key()).toEqual('1');
		expect(wrapper.find('li').at(1).key()).toEqual('2');
		wrapper.find('li').forEach((node, index) => {
			// 验证text文本内容
			expect(node.text()).toEqual(arr[index].name.toString());
			// 验证key文本内容
			expect(node.key()).toEqual(arr[index].name.toString());
		});
	});
	
	it('renders 1 li and click Handler', () => {
		let mockPerson = { name: 'first' },
			arr = [ mockPerson ],
			clickHandler = (person) => {
				expect(mockPerson).toEqual(person);
			}
		const wrapper = shallow(<List arr={ arr } clickHandler={ clickHandler } />);
		// 触发点击
		wrapper.find('li').at(0).simulate('click');
		expect(wrapper.find('li').length).toEqual(1);
	});
});
```

其实也蛮简单的，只要找到对应的功能，对于代码一定难不倒你~

### 容器组件

　　容器组件的测试如果是使用`connect()`方法生成出来的那么我们只测试展示组件就够了，如果其中还包含一部分逻辑那么就需要将内部的组件`export`出来再进行测试，目前还没有找到更好的例子，以后找到更多的资料再进行补充吧~

## 总结

　　React并不是一个all in one的框架，所以在测试的过程当中需要不断添加第三方模块来满足我们的需求，实践的过程中也会出现多种方法供你选择，需要不断尝试才能找到一个更适合自己的方案。Angular就不需要啦，O(∩_∩)O哈哈哈~文字写的有点乱，有时间再好好整理吧！

## 参考

【官方文档，强烈建议学习】http://redux.js.org/docs/recipes/WritingTests.html

http://arnaudbenard.com/redux-mock-store/

https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch