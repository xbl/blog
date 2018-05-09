---
title: 为小程序框架 WePY 添加单元测试
date: 2018-03-22 18:49:16
tags:
- 小程序
- wepy
- unit test
- 单元测试
---

　　现在微信小程序越来越火，受到了各大企业和开发者的青睐，越来越多的开发者投入精力学习小程序开发，小程序的开发框架也层出不穷，像美团的 [mpvue](http://mpvue.com/) 、腾讯的 [WePY](https://tencent.github.io/wepy/) ，还有很多个人开发者开源出的优秀作品。但由于 [mpvue](http://mpvue.com/) 刚刚推出不久还不敢轻易投入生产，而出身名门又根正苗红的 [WePY](https://tencent.github.io/wepy/) 成了大多数企业和个人开发者的选择，我们也是其中之一。

　　在使用 [WePY](https://tencent.github.io/wepy/) 过程中好的就不说了，唯一不足的是官方没有提供单元测试的方法，[问过](https://github.com/Tencent/wepy/issues/1110)官方开发团队短期应该不会提供对测试的支持，作为一名有情怀的程序员这并不能阻挡我们写单元测试。

### WePY 的技术背景

　　了解过  [WePY](https://tencent.github.io/wepy/) 的同学一定知道，它的文件写法与 [Vuejs](https://cn.vuejs.org/) 一致，都是在一个文件中使用 `template` 、`style` 、`script` 来完成 Page 或者 Component 的开发，官方甚至推荐使用在VSCode 中 使用 vue 的语法高亮，只是后缀不同而已。在使用 [Vuejs](https://cn.vuejs.org/) 开发时需要 webpack + vue-loader，而 [WePY](https://tencent.github.io/wepy/) 则是自己开发了一个 [wepy-cli](https://github.com/Tencent/wepy/blob/2.0.x/packages/wepy-cli/README.md) 工具来进行.wpy 构建和解析，其中 loader 的部分还不是一个独立模块，cli 似乎也只是做静态编译，并不需要运行 `.wpy` 文件。

### 添加测试的思路

了解了技术背景，我们先来写一段我们期望的测试代码：

**index.spec.js**

``` javascript
import IndexPage from './IndexPage.wpy'

test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});

test('index.data.a will be 1', () => {
  const index = new IndexPage();
  expect(index.data.a).toBe(1);
});

```

要想成功运行这段测试代码，摆在我们面前的有几个问题：

1. 选择一个测试框架
2. **测试框架需要支持非 js 文件扩展**
3. **写一个.wpy 文件的 loader**

　　选择一个测试框架，这个简单啦，js 世界里选项特别多，比如：[Mocha](https://mochajs.org/) 、[Jest](https://facebook.github.io/jest/)、[AVA](https://github.com/avajs/ava) 等等，Mocha 使用的最多，但是需要再使用外部断言库，太麻烦，Jest 和 AVA 都还比较简单。要支持非 js 文件扩展 ，这个需求好像没在 Mocha 中见过，不太了解，Jest 和 AVA 似乎都支持。AVA 使用我个人不太熟悉，Jest 的文档更完整，而且更熟悉，所以 Jest 胜出。

#### 安装 Jest

``` shell
npm i jest babel-jest -D
```

修改 package.json

```json
"scripts": {
  ...
  "test": "jest ./src/**/*.spec.js",
  "test:watch": "jest ./src/**/*.spec.js --watch"
},
...
"babel": {
  "presets": [
    "env"
  ]
},
"jest": {
  "moduleFileExtensions": [
    "js"
  ],
  "transform": {
    "^.+\\.js$": "<rootDir>/node_modules/babel-jest",
  }
},
...
```

然后运行测试：

``` shell
npm test
```

报错啦！当然，我们还没有对 `.wpy` 文件支持。

#### 对非 js 文件扩展

　　首先来看一下 Jest 的[文档](https://facebook.github.io/jest/docs/en/tutorial-react.html#custom-transformers)，Jest 允许我们自定义 transform ，只要在 process 函数中返回 **字符串** 或者 **包含 code 字段的对象**就 ok 了，所以先把示例代码 Copy 过来再说！

```javascript
'use strict';

const babel = require('babel-core');
const jestPreset = require('babel-preset-jest');

module.exports = {
  process(src, filename) {
    if (babel.util.canCompile(filename)) {
      return babel.transform(src, {
        filename,
        presets: [jestPreset],
      });
    }
    return src;
  },
};
```

　　接下来我们要对 `.wpy` 文件进行解析，我们先设一个小目标：**只解析 wpy 文件的 script 标签内容**，参考 [wepy-cli](https://github.com/Tencent/wepy/blob/2.0.x/packages/wepy-cli/README.md) 的思路，我们把 `.wpy` 文件当做 xml 解析就可以了，同时还有支持 es2015，代码如下：

```javascript
'use strict';

const babel = require('babel-core');
const jestPreset = require('babel-preset-jest');
const stage1 = require('babel-preset-stage-1');
const es2015 = require('babel-preset-es2015');
const DOMParser = require('xmldom').DOMParser;

module.exports = {
  process(src, filename) {
    const doc = new DOMParser().parseFromString(src);
    let code = '';
    [].slice.call(doc.childNodes || []).forEach((child) => {
        const nodeName = child.nodeName;
        if (nodeName === 'script') {
          [].slice.call(child.childNodes || []).forEach((c) => {
              code += c.toString();
          });
        }
    });
    if (code) {
      return babel.transform(code, {
        filename,
        presets: [jestPreset, es2015, stage1],
      });
    }
    return src;
  },
};
```

然后再修改 package.json

```javascript
...
"jest": {
  "moduleFileExtensions": [
    "js",
    "wpy"
  ],
  "transform": {
    "^.+\\.js$": "<rootDir>/node_modules/babel-jest",
    ".*\\.(wpy)$": ".../wepy-jest"
  }
},
...
```

再次运行测试：

```shell
npm test
```

输出：
```shell
 PASS  src/pages/index.spec.js
  ✓ two plus two is four (3ms)
  ✓ index.data.a will be 1
```

**成功啦！**

### 总结

　　我们成功的迈出一小步，虽然还不能支持模板的编译，但调用 Page 和 Component 的methods 和 data 就可以验证我们的大部分逻辑，只要数据是正确的，就可以把错误率降到最低。

　　[wepy-jest](https://github.com/xbl/wepy-jest) 完整代码已经放到了github ，未来会对它做更多的扩展，欢迎大家 fork or star ，如果文中出现错误，欢迎大家批评指正。