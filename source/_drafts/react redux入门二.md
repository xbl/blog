---
title: react redux入门二
date: 2017-09-03 15:22:30
tags:
- react
- router
- javascript
---

　　路由

```shell
npm install react-router-dom
// or
yarn add react-router-dom
```

　　之前`componentDidMount()` 调用`dispatch`， 每次切换到路由都调用`componentDidMount()`

```jsx
<Route exact path="/" component={App}/>
```

