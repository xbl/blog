---
title: 为什么前后端分离了，你比从前更痛苦？
date: 2018-10-09 09:25:12
tags:
- 契约
- mock server
- 
---



**你有没有遇到过：**

前端代码刚写完，后端的接口又变了？

接口文档永远都是不对的？

测试工作永远只能临近上线才能开始？



为什么前后端分离了，你比从前更痛苦？



**为什么接口会频繁变动？**

设计之初没有想好，

变动的成本较低。前后端同学坐在一起工作的时候效率会有提升，当后端同学接口变化时，只需要口头上通知一下即可，团队也觉得我们没有文档，我们很敏捷啊。没错，我们需要承认这样配合开发的效率会很高，但是如果频繁的变动会导致不断返工，亦是一种浪费，这种浪费是可以被减少，甚至是被消除的。

一方面提高需求的理解和接口设计能力。

另一方面提高接口变化的成本。



**为什么接口文档永远都是不对的？**

正因为接口的频繁变化，文档永远落后于实际接口，维护文档的带来了一定的成本。接口文档就如同我们定下的契约，但是当我们写完契约就没有用了，维护它似乎不能带来价值。有些公司干脆丢掉接口文档，说我们要拥抱敏捷。

原因在于契约没有给我们带来真正的价值。



**为什么测试工作永远只能临近上线才能开始？**

测试同学会感受颇深，一个需求，后端开发 4 天，前端开发 4 天，联调 4 天，留给测试同学只有2天时间，测不完只能带 bug 上线。

测试同学前期根本无法介入，前几天只能写写 Test Case 。在 “提测” 之前一切都在变，接口在变，前端也在变。



问题大致搞清楚了，解决以上问题要**让接口文档发挥价值，提高变动接口的成本，测试提前介入。**



### How ？

让接口文档发挥出应有的价值，就要赋予契约的作用，来约束我们必须按照契约来进行开发，契约是由前后端共同定义。

有了契约，测试同学就可以提前介入，依照契约文档来编写测试脚本。

当后端接口发生变化除了口头通知以外必须修改契约，前端同学和测试同学都需要各自修改。如此一来修改契约的成本变高，人们在定契约时则会更加慎重。

理论终于扯完了，说起来容易做起来难啊，需要工具来帮助我们。



接口描述的工具有很多，比较知名的 [Swagger](https://swagger.io/) 和 [Raml](https://raml.org/)，我个人更倾向于 Raml 。

![API 文档](https://ws1.sinaimg.cn/large/006tNbRwly1fw2w6al0lfg30dw0bob0f.gif)

描述工具帮助我们生成文档还不够，还要帮助我们生成 Mock Server。

### [raml-mocker](https://github.com/xbl/raml-mocker)

是一个基于 Raml 的 Mock Server 工具，在用 Raml 描述接口一个简单配置即可启动 Mock Server。

## 开始

#### 初始化项目
```shell
git clone https://github.com/xbl/raml-mocker-starter.git raml-api
cd raml-api
git remote rm origin
```
#### 安装
```shell
yarn
# or
npm install
```
#### 启动 mock server
```shell
yarn start
# or
npm start
```
#### 测试
```shell
curl -i http://localhost:3000/api/v1/users/1/books/
# or
curl -i http://localhost:3000/api/v1/users/1/books/1
```

#### 生成 API 可视化文档

```shell
yarn run build
# or
npm run build
```

此功能使用了[raml2html](https://www.npmjs.com/package/raml2html)。

### 配置 .raml-config.json

```json
{
  "controller": "./controller",
  "raml": "./raml",
  "main": "api.raml",
  "port": 3000,
  "plugins": []
}
```

* controller: controller 目录路径，在高级篇中会有更详细说明
* raml: raml 文件目录
* main: raml 目录下的入口文件
* port:  mock server 服务端口号
* plugins: 插件（*可能会有变动*）




### 入门篇：Mock Server

raml-mocker 可以不写 js 代码生成Mock Server，只需要在response 添加 example:

```yaml
/books:
  /:id:
    post:
      body:
        application/json:
          type: abc
      responses:
        200:
          body:
            application/json:
              type: song
              example: !include ./books_200.json
```

books_200.json

```json
{
  "code": 200,
  "data": [
    {
      "id": 1,
      "title": "books title",
      "description": "books desccription1"
    },
    {
      "id": 2,
      "title": "books title",
      "description": "books desccription2"
    }
  ]
}
```



### 高级篇：动态 Server

在 raml 文档中添加 `(controller)` 指令，即可添加动态的 Server，如：

```yaml
/books:
  type:
    resourceList:
  get:
    description: 获取用户的书籍
    (controller): user#getBook
    responses:
      200:
        body:
          type: song[]
          example: !include ./books_200.json
```

在文档中 `(controller)`  表示 controller 目录下 user.js 中 getBook 函数。

controller/user.js

```javascript
exports.getBook = (req, res, webApi) => {
  console.log(webApi);
  res.send('Hello World!');
}
```

Raml-mocker 是在 [expressjs](http://expressjs.com/) 基础上进行开发，req、res 可以参考 express 文档。

webApi 会返回文档中的配置：

```json
{
  "absoluteUri": "/api/:version/users/:user_id/books",
  "method": "get",
  "controller": "user#getBook",
  "responses": [
    {
      "code": "200",
      "body": "... example ...",
      "mimeType": "application/json"
    }
  ]
}

```

如此，raml-mocker 提供了更多可扩展空间，我们甚至可以在 controller 中实现一定的逻辑判断。



#### 插件

Raml-mocker 提供了插件机制，允许我们在不使用 `controller` 指令的时候对 response 的内容进行处理，例如使用 [Mockjs](http://mockjs.com/)。

.raml-config.json

```json
{
  "controller": "./controller",
  "raml": "./raml",
  "main": "api.raml",
  "port": 3000,
  "plugins": ["./plugins/mock.js"]
}

```

./plugins/mock.js

```javascript
var { mock } = require('mockjs');

module.exports = (body) => {
  try {
    return mock(JSON.parse(body));
  } catch(e) {}
  return body;
}

```



### 



[APIDoc](http://apidocjs.com/)
阿里出品的 [RAP](http://thx.github.io/RAP/index_zh.html)