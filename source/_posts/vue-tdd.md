---
title: 前端如何做测试驱动开发-vue版
date: 2019-06-21 16:00:47
tags:
---

最近和测试杠上了，写了的文章都和测试相关。当然，这里的「测试」并不是具体的某个角色，而是验证程序正确性的工作。曾经前端如何 TDD 困扰了我很久。随着时间的推移，前端框架开始成熟，我对前端测试有了更深刻的理解，把我做前端 TDD 的方法分享给大家。

# 理论篇

测试驱动开发，英文全称 Test-Driven Development（简称 TDD），是由Kent Beck 先生在极限编程（XP）中倡导的开发方法。以其倡导先写测试程序，然后编码实现其功能得名。

TDD 能从技术的角度帮我们提高代码质量，使代码执行结果正确，容易理解。由于有测试的保障，开发者可以更放心的重构自己的代码。

当有变更时，测试同学最关心变更的影响范围，然而开发者也很难精准确定变更所带来的影响。虽然我们不追求测试覆盖率，但足够的测试覆盖总是能够给我们更多的信心，TDD 则是增加测试覆盖的唯一途径。

在面对一个完全没有思路的算法的时候，TDD 则变成了测试驱动设计（Test-Driven Design）。选一个最简单的用例，用最简单的代码通过测试。逐渐增加测试 Case、通过测试 、重构来驱动出设计。

### TDD 的步骤

![TDD 步骤](https://tva1.sinaimg.cn/large/006y8mN6ly1g6ni262pvkj30xq0r6kjl.jpg)

1. 写一个失败的测试
2. 写一个刚好让测试通过的代码
3. 重构上面的代码

每一个步骤都是带上一个角色的帽子🎩 ，让你更专注当下这个角色应该做的事。

### TDD 的三原则

1. 没有测试之前不要写任何功能代码 
2. 一次只写一个刚好失败的测试，作为新加功能的描述
3. 不写任何多余的产品代码，除⾮它刚好能让失败的测试通过 

TDD 写出的代码的验证逻辑针对的是独立的代码块，可能不是系统中的业务完整功能。用测试先行的方法写出的漂亮的代码也可能做出的功能不是客户想要的（*因为需求理解的错误所导致*）。因此，使用 「验收驱动测试开发(ATDD)」是很有必要。

### 验收驱动测试开发——ATDD(Acceptance Test Driven Development) 

ATDD 通过名字就可以看出和 TDD 有着某种神秘的联系， ATDD 是 TDD 的延伸。

在传统做法中，要给系统添加新的特性，开发人员会按照文档开发，测试，最后交给客户验收。ATDD 则有些不同：在编码前先明确新特性的验收标准，将验收标准转换成测试用例（代码），再编写代码让测试通过，当所有的验收条件被满足，也就意味着这个功能完整的实现。

2003 年左右的时候 Kent Beck 曾对 ATDD 提出质疑，时间太早不好查证，我个人猜测原因是 验收条件做为一个 测试 Case 在某些时候会比较大。

举个例子：

> **如果**用户购物车里勾选了可以购买的商品，**当**用户点击下单，**则**系统为其创建了一个包含勾选商品的一个订单。

可以看出这个验收条件可能要写上一大堆的功能点（如：锁定库存、创建订单等等）才能满足，开发人员必须将 Case 再进行拆分。

创建订单的伪代码：

```java
public Order create(List<Product> products) {
  // 锁定库存
  // 创建订单
  // 创建定时任务(以便超时支付，而取消订单、释放库存)
  // ...
  return null;
}
```

将注释的地方写出相应的函数，如：`lockStock()`、`createOrder()`、`createOrderTimeoutTask() `...并为这些函数编写单元测试再去实现。当然，你并不需要一次全部完成，而是一次只实现一个任务。

尽管 Kent Beck 曾对 ATDD 提出过质疑，但却为 2012 年出版的[《ATDD by Example》](https://book.douban.com/subject/10813193/) 写了推荐序，ATDD 也早成为公认的做法。相比后端，前端更适合使用 ATDD。

#### 测试条件格式

测试条件通常遵循以下形式：

>Given (如果)
>​	指定的状态，通常是给出的条件；
>When (当)
>​	触发一个动作或者事件；
>Then (则)
>​	对期望结果的验证；

写 JavaScript 同学要比 Java 同学幸福很多，不需要把函数名写的老长：


```javascript
it('Given a = 1 And b = 2，When execute add()，Then result is 3', () => {
  // ...
});

// 如果团队更习惯用中文，可以这样：
it('如果： a = 1 并且 b = 2，当：执行 add()，则：结果是 3', () => {
  // ...
});
```

# 实践篇

以往的示例都是拿算法来实践，搞的同学们以为 TDD 只适合做一些算法题，所以这次我们不拿算法来做实例，使用一个相对真实且简单的需求——登录。

## 安装环境



```shell
vue create vue-tdd-demo
```

![手动选择](https://i.loli.net/2019/06/21/5d0c8fe246f1c40706.png)

![选择 Unit Testing](https://i.loli.net/2019/06/21/5d0c906d0cbaf88756.png)

勾选 Unit Testing （单元测试），后面按照自己喜好来选择。

![选择Jest.jpg](https://i.loli.net/2019/06/21/5d0c91452e1f778243.jpg)

这里选择 Jest 作为测试框架。

### 验证环境

安装好之后运行 `npm run test:unit` ，刚安装的项目就会报错，真让人惆怅！看看如何解决：

![运行测试报错.png](https://i.loli.net/2019/06/21/5d0c9295afe8d27395.png)

jest.config.js 或者 package.json 中找到 `transformIgnorePatterns` 这个配置

```json
// ...
transformIgnorePatterns: [
	'/node_modules/',
	'/node_modules/(?!vue-awesome)', // 添加此行
],
```

再运行 `npm run test:unit`

![测试成功.png](https://i.loli.net/2019/06/21/5d0c93aa18c2c56887.png)



## 演练实例

### 需求

![需求](http://ww1.sinaimg.cn/large/006tNc79ly1g4dijx6oi7j30ha0beq37.jpg)

> 页面包含用户名、密码输入框和提交按钮，提交之后成功服务端返回状态为 200 然后跳转到 Home 首页，失败则 `alert()` 文字提示。
>
> 在用户名密码为空时不能提交。

通常前端功能可分为两部分，一部分是接口、另一部分是页面。

我们先来实现接口部分：

### 接口 Service 测试

请求模块使用 [axios](https://github.com/axios/axios) ，[axios-mock-adapter](https://www.npmjs.com/package/axios-mock-adapter) 是一个辅助模块，帮助我们验证接口调用是否正确。

#### 任务拆分

> 登录 Service.login 方法，接受 user 对象(username、password) ，使用 axios 发送请求，URL 为 `/users/tokens`，使用 `POST` 方式，返回一个Response 的 Promise 对象；
>
> 当输入调用 `Service.login({username: '谢小呆', password: '123'});`  ，则 axios post 的数据则与参数相同；



#### 安装依赖

```shell
npm install axios
npm install axios-mock-adapter --save-dev
# or
yarn add axios
yarn add axios-mock-adapter -D
```



#### 编写测试

```javascript
import axios from 'axios';
import MockAdapter from 'axios-mock-adapter';
import Service from '@/login/service';

describe('Login service', () => {
  it('Given 登录信息为 谢小呆,123, When 执行 Service.login() 时，Then 请求参数为 谢小呆,123 的用户', async () => {
    const mock = new MockAdapter(axios);
    const expectedResult = { username: '谢小呆', password: '123' };
    mock.onPost('/users/token').reply(200);

    await Service.login(expectedResult);
    expect(mock.history.post.length).toBe(1);
    expect(mock.history.post[0].data).toBe(JSON.stringify(expectedResult));
  });
});

```

这里验证了传入的「参数」与 `POST ` 的数据是否一致，我们并没有真正去发网络请求，也没有必要。毕竟我们并不关心接口「此时」是否能通，只要后端按照我们的接口约定给出特定的返回即可。对单元测试不了解的同学可以参考[这里](https://mp.weixin.qq.com/s/XdojsXmeDGOae00WwhrqDg)。

此时运行 `yarn run test:unit` 缺少 service.js 文件。

创建 src/login/service.js 文件

```javascript
import axios from 'axios';

const login = user => axios.post('/users/token', user);

export default {
  login,
};

```

再次运行 `yarn run test:unit` ，测试通过！

### 页面、组件测试

对于 Vue 来说页面和组件是同一个东西，Vue 提供了一个很方便的单元测试工具 [vue-test-units](https://vue-test-utils.vuejs.org/zh/) ，这里就不过多赘述其用法，参考官方文档即可。

###### 任务：当用户访问页面时可以看到用户名、密码输入框和提交按钮，所以页面中只要包含这 3 个元素即可。

```javascript
import { mount } from '@vue/test-utils';
import Login from '@/login/index.vue';

describe('Login Page', () => {
  it('When 用户访问登录页面，Then 看到用户名、密码输入框和提交按钮', () => {
    const wrapper = mount(Login);
    expect(wrapper.find('input.username').exists()).toBeTruthy();
    expect(wrapper.find('input.password').exists()).toBeTruthy();
    expect(wrapper.find('button.submit').exists()).toBeTruthy();
  });
});

```

运行测试报错，缺少 `@/login/index.vue` 文件

```vue
<template>
  <ul>
    <li>用户名：<input type="text" class="username" v-model="user.username"></li>
    <li>密码：<input type="text" class="password" v-model="user.password"></li>
    <li><button class="submit" @click="onSubmit">提交</button></li>
  </ul>
</template>

```

再次运行 `yarn run test:unit`，通过！添加新的 case。

###### 任务：实现双向绑定，在input 中输入用户名为 谢小呆，密码为 123，vue 的 vm.user 为 {username: '谢小呆', password: '123'};

```javascript
import { mount } from '@vue/test-utils';
import Login from '@/login/index.vue';

describe('Login Page', () => {
  it('When 用户访问登录页面，Then 看到用户名、密码输入框和提交按钮', () => {
    const wrapper = mount(Login);
    expect(wrapper.find('input.username').exists()).toBeTruthy();
    expect(wrapper.find('input.password').exists()).toBeTruthy();
    expect(wrapper.find('button.submit').exists()).toBeTruthy();
  });

  it('Given 用户访问登录页面，When 用户输入用户名为 谢小呆, 密码为 123，Then 页面中的 user 为 {username: "谢小呆", password: "123"}', () => {
    const wrapper = mount(Login);
    wrapper.find('input.username').setValue('谢小呆');
    wrapper.find('input.password').setValue('123');

    const expectedResult = { username: '谢小呆', password: '123' };
    expect(wrapper.vm.user).toEqual(expectedResult);
  });
});

```

运行测试，报错：

```shell
  Expected value to equal:
      {"password": "123", "username": "谢小呆"}
  Received:
      undefined
```

来添加需要绑定的对象：

```vue
<template>
  <ul>
    <li>用户名：<input type="text" class="username" v-model="user.username"></li>
    <li>密码：<input type="text" class="password" v-model="user.password"></li>
    <li><button class="submit" @click="onSubmit">提交</button></li>
  </ul>
</template>

<script>
export default {
  data: () => ({
    user: {
      username: '',
      password: '',
    },
  }),
};
</script>

```

测试通过！

###### 任务：事件绑定，点击提交按钮，调用 onSubmit 方法，验证 onSubmit 是否被调用。

验证方法被调用需要使用 [sinon](https://sinonjs.org/) 库。

```shell
npm install sinon --save-dev
# or
yarn add sinon -D
```



```javascript
import sinon from 'sinon';
import { mount } from '@vue/test-utils';
import Login from '@/login/index.vue';

describe('Login Page', () => {
  // ...

  it('Given 用户访问登录页面，When 用户点击 submit，Then onSubmit 方法被调用', () => {
    const wrapper = mount(Login);
    const onSubmit = sinon.fake();
    wrapper.setMethods({ onSubmit });
    wrapper.find('button.submit').trigger('click');

    expect(onSubmit.called).toBeTruthy();
  });
});


```

trigger 点击事件后调用 `onSubmit` 方法。通过 sinon 制作了一个替身，通过 [setMethods](https://vue-test-utils.vuejs.org/zh/api/wrapper/#setmethods) 方法，替换了 vm 的 onSubmit 函数。

运行测试，报错：


```shell
 FAIL  tests/unit/login/index.spec.js
  ● Login Page › Given 用户访问登录页面，When 用户点击 submit，Then onSubmit 方法被调用

    expect(received).toBeTruthy()

    Received: false

      26 |     wrapper.find('button.submit').trigger('click');
      27 | 
    > 28 |     expect(onSubmit.called).toBeTruthy();
         |                             ^
      29 |   });
      30 | });
      31 | 

      at Object.toBeTruthy (tests/unit/login/index.spec.js:28:29)
```

修改代码，添加绑定事件和函数：

```vue
<template>
  <ul>
    <li>用户名：<input type="text" class="username" v-model="user.username"></li>
    <li>密码：<input type="text" class="password" v-model="user.password"></li>
    <li><button class="submit" @click="onSubmit">提交</button></li>
  </ul>
</template>

<script>
export default {
  data: () => ({
    user: {
      username: '',
      password: '',
    },
  }),
  methods: {
    onSubmit() {

    },
  },
};
</script>

```

运行测试，通过！

###### 任务：增加验证，当用户名、密码为空时，提交按钮为 disabled 状态

```javascript
import sinon from 'sinon';
import { mount } from '@vue/test-utils';
import Login from '@/login/index.vue';

describe('Login Page', () => {
  // ...

  it('Given 用户访问登录页面，When 用户点击 submit，Then onSubmit 方法被调用', () => {
    const wrapper = mount(Login);
    const onSubmit = sinon.fake();
    wrapper.setMethods({ onSubmit });
    wrapper.find('button.submit').trigger('click');

    expect(onSubmit.called).toBeTruthy();
  });

  it('Given 用户访问登录页面，When 用户未输入登录信息，Then submit 按钮为 disabled And 点击 submit 不会调用 onSubmit', () => {
    const wrapper = mount(Login);
    const onSubmit = sinon.fake();
    wrapper.setMethods({ onSubmit });
    const submitBtn = wrapper.find('button.submit');
    submitBtn.trigger('click');

    expect(submitBtn.attributes('disabled')).toEqual('disabled');
    expect(onSubmit.called).toBeFalsy();
  });
});

```

运行测试，报错：

```shell
● Login Page › Given 用户访问登录页面，When 用户未输入登录信息，Then submit 按钮为 disabled And 点击 submit 不会调用 onSubmit

    expect(received).toEqual(expected)

    Expected value to equal:
      "disabled"
    Received:
      undefined

    Difference:

      Comparing two different types of values. Expected string but received undefined.

      36 |     submitBtn.trigger('click');
      37 | 
    > 38 |     expect(submitBtn.attributes('disabled')).toEqual('disabled');
         |                                              ^
      39 |     expect(onSubmit.called).toBeFalsy();
      40 |   });
      41 | });

      at Object.toEqual (tests/unit/login/index.spec.js:38:46)
```

修改代码，使测试通过：

```vue
<template>
  <ul>
    <li>用户名：<input type="text" class="username" v-model="user.username"></li>
    <li>密码：<input type="text" class="password" v-model="user.password"></li>
    <li><button class="submit" @click="onSubmit" :disabled="!validate">提交</button></li>
  </ul>
</template>

<script>
export default {
  data: () => ({
    user: {
      username: '',
      password: '',
    },
  }),
  computed: {
    validate() {
      return this.user.username && this.user.password;
    },
  },
  methods: {
    onSubmit() {

    },
  },
};
</script>

```

运行！

```shell
 FAIL  tests/unit/login/index.spec.js
  ● Login Page › Given 用户访问登录页面，When 用户点击 submit，Then onSubmit 方法被调用

    expect(received).toBeTruthy()

    Received: false

      26 |     wrapper.find('button.submit').trigger('click');
      27 | 
    > 28 |     expect(onSubmit.called).toBeTruthy();
         |                             ^
      29 |   });
      30 | 
      31 |   it('Given 用户访问登录页面，When 用户未输入登录信息，Then submit 按钮为 disabled And 点击 submit 不会调用 onSubmit', () => {

      at Object.toBeTruthy (tests/unit/login/index.spec.js:28:29)
```

唉？又报错了！报错的 case 不是刚刚的这个，而是上一个 case 报错了。因添加了 validate 验证之后，在没有数据的情况下，点击按钮不会调用 onSubmit 方法。

修改 case ，添加登录信息：

```javascript
it('Given 用户访问登录页面 And 用户输入用户名、密码，When 点击 submit，Then onSubmit 方法被调用', () => {
    const wrapper = mount(Login);
    const onSubmit = sinon.fake();
    wrapper.setMethods({ onSubmit });
    wrapper.find('input.username').setValue('谢小呆');
    wrapper.find('input.password').setValue('123');
    wrapper.find('button.submit').trigger('click');

    expect(onSubmit.called).toBeTruthy();
  });
```

再运行，测试通过！

###### 任务：用户输入登录信息后提交，返回状态应该为 200，并且调用 loginSuccess() 方法。

由于我们之前已经测试过 Service.login，所以我们这里不再进行测试，而是将 Service.login 制作一个替身，让它返回 200。

```javascript
import Vue from 'vue';
import sinon from 'sinon';
import { mount } from '@vue/test-utils';
import Login from '@/login/index.vue';
import Service from '@/login/service';

describe('Login Page', () => {
  // ...

  it('Given 用户访问登录页面 And 用户输入用户名、密码，When 点击 submit，Then 调用 Service.login() 后返回 200 And 调用 loginSuccess 方法', async () => {
    const stub = sinon.stub(Service, 'login');
    stub.resolves({ status: 200 });

    const wrapper = mount(Login);
    const loginSuccess = sinon.fake();
    wrapper.setMethods({ loginSuccess });

    const expectedUser = { username: '谢小呆', password: '123' };
    wrapper.find('input.username').setValue(expectedUser.username);
    wrapper.find('input.password').setValue(expectedUser.password);
    wrapper.find('button.submit').trigger('click');

    await Vue.nextTick();

    expect(loginSuccess.called).toBeTruthy();
    expect(stub.alwaysCalledWith(expectedUser)).toBeTruthy();
    stub.restore();
  });
});

```

因为是异步请求，所以需要 `Vue.nextTick` 来等待堆栈处理完成，再进行验证。断言验证 loginSuccess 这个函数是否调用，以及期望的的登录信息与我们输入的信息相同。

运行测试，失败！篇幅关系就不再贴失败结果。

修改代码：

```vue
<template>
  <ul>
    <li>用户名：<input type="text" class="username" v-model="user.username"></li>
    <li>密码：<input type="text" class="password" v-model="user.password"></li>
    <li><button class="submit" @click="onSubmit" :disabled="!validate">提交</button></li>
  </ul>
</template>

<script>
import Service from './service';

export default {
  data: () => ({
    user: {
      username: '',
      password: '',
    },
  }),
  computed: {
    validate() {
      return this.user.username && this.user.password;
    },
  },
  methods: {
    async onSubmit() {
      const response = await Service.login(this.user);
      if (response.status === 200) {
        this.loginSuccess();
      }
    },
    loginSuccess() {
      this.$router.push({ name: 'home' });
    },
  },
};
</script>

```

添加完整的功能，运行测试，通过！

###### 任务：用户登录提交后返回错误状态，调用 loginFailure() 方法。

```javascript
import Vue from 'vue';
import sinon from 'sinon';
import { mount } from '@vue/test-utils';
import Login from '@/login/index.vue';
import Service from '@/login/service';

describe('Login Page', () => {
  // ...

  it('Given 用户访问登录页面 And 用户输入用户名、密码，When 点击 submit，Then 调用 Service.login() 后返回不等于 200 And 调用 loginFailure 方法', async () => {
    const stub = sinon.stub(Service, 'login');
    stub.resolves({ status: 404 });

    const wrapper = mount(Login);
    const loginFailure = sinon.fake();
    wrapper.setMethods({ loginFailure });

    const user = { username: '谢小呆', password: '123' };
    wrapper.find('input.username').setValue(user.username);
    wrapper.find('input.password').setValue(user.password);
    wrapper.find('button.submit').trigger('click');

    await Vue.nextTick();

    expect(loginFailure.called).toBeTruthy();
    stub.restore();
  });
});

```

运行测试，失败！

修改代码：

```vue
<template>
  <ul>
    <li>用户名：<input type="text" class="username" v-model="user.username"></li>
    <li>密码：<input type="text" class="password" v-model="user.password"></li>
    <li><button class="submit" @click="onSubmit" :disabled="!validate">提交</button></li>
  </ul>
</template>

<script>
import Service from './service';

export default {
  data: () => ({
    user: {
      username: '',
      password: '',
    },
  }),
  computed: {
    validate() {
      return this.user.username && this.user.password;
    },
  },
  methods: {
    async onSubmit() {
      const response = await Service.login(this.user);
      if (response.status === 200) {
        this.loginSuccess();
        return;
      }
      this.loginFailure();
    },
    loginSuccess() {
      this.$router.push({ name: 'home' });
    },
    loginFailure() {
      alert('用户名、密码错误，请稍后再试！');
    },
  },
};
</script>

```

再运行测试，通过！

我们完成了主要功能的测试覆盖，但是 loginSuccess 和 loginFailure 方法并没有测试，原因是这部分调用了 路由，通常不会出错，所以测试的意义不大。

当然，如果要测试仍然可以。

###### 任务：当执行 loginSuccess 时，路由应为首页，即：'/'。

```javascript
import Vue from 'vue';
import sinon from 'sinon';
import VueRouter from 'vue-router';
import { createLocalVue, mount } from '@vue/test-utils';
import Login from '@/login/index.vue';
import Service from '@/login/service';
import router from '@/router';

describe('Login Page', () => {
  // ...

  it('When 执行 loginSuccess()，Then $route.path 为 /', async () => {
    const localVue = createLocalVue();
    localVue.use(VueRouter);

    const wrapper = mount(Login, {
      localVue,
      router,
    });

    wrapper.vm.loginSuccess();
    expect(wrapper.vm.$route.path).toEqual('/');
  });
});

```

这里需要对默认的 router.js 做改造：

```javascript
- import Vue from 'vue';
import Router from 'vue-router';
import Home from './views/Home.vue';

- Vue.use(Router);

export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home,
    },
    {
      path: '/about',
      name: 'about',
      // route level code-splitting
      // this generates a separate chunk (about.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import(/* webpackChunkName: "about" */ './login/index.vue'),
    },
  ],
});

```

接着修改 main.js

```javascript
import Vue from 'vue';
+ import Router from 'vue-router';
import App from './App.vue';
import router from './router';

Vue.config.productionTip = false;
+ Vue.use(Router);

new Vue({
  router,
  render: h => h(App),
}).$mount('#app');

```

运行测试通过！

虽然测试覆盖率不是我们所追求的，但是知道覆盖率是在上升还是在下降可以反应团队在这期间的质量关注度。

### 测试覆盖率

修改配置 jest.config.js 或者 package.json

主要关注 `coverage` 这几个配置的修改：

```javascript
module.exports = {
  moduleFileExtensions: [
    'js',
    'jsx',
    'json',
    'vue',
  ],
  transform: {
    '^.+\\.vue$': 'vue-jest',
    '.+\\.(css|styl|less|sass|scss|svg|png|jpg|ttf|woff|woff2)$': 'jest-transform-stub',
    '^.+\\.jsx?$': 'babel-jest',
  },
  transformIgnorePatterns: [
    '/node_modules/',
    '/node_modules/(?!vue-awesome)',
  ],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  snapshotSerializers: [
    'jest-serializer-vue',
  ],
  testMatch: [
    '**/tests/unit/**/*.spec.(js|jsx|ts|tsx)|**/__tests__/*.(js|jsx|ts|tsx)',
  ],
  testURL: 'http://localhost/',
  watchPlugins: [
    'jest-watch-typeahead/filename',
    'jest-watch-typeahead/testname',
  ],
+  collectCoverage: true,
+  collectCoverageFrom: [
+    '**/*.{js,vue}',
+    '!**/node_modules/**',
+    '!**/App.vue',
+    '!**/main.js',
+    '!**/router.js',
+    '!*.config.js',
+    '!.eslintrc.js',
+  ],
+  coverageReporters: [
+    'html',
+    'text-summary',
+  ],
+  coveragePathIgnorePatterns: [
+    '<rootDir>/coverage',
+    '<rootDir>/tests',
+    'babel.config.js',
+  ],
};

```

运行测试：

![文本测试覆盖率.jpg](https://i.loli.net/2019/06/25/5d11c6634c5bd95109.jpg)

Html 报告：

![html 测试覆盖率报告.png](https://i.loli.net/2019/06/25/5d11d10039ef532760.png)

点开可以看到每一个文件的覆盖，以及是否有逻辑分支忘记测试。

注意：添加测试报告之后运行测试的速度会变慢。

## 总结

经过上面的练习，相信大家能对前端如何做 TDD 有一个基本的掌握。即便不使用 TDD，前端的测试也仍然有意义。当然，相信会有一部分同学会对本文产生质疑，国内写前端测试的人 (或者公司) 都很少，更不要说前端 TDD，这样做很花时间，真的值得吗？写完页面让测试同学点一遍不就可以了？为什么一定要去写这么多代码来去验证呢？

本文并不打算去说服你去写测试，以往的工作中没有写过测试，很难通过一篇文章就能让你心动。这如同人们知道健身对自己有好处，但都懒得锻炼。只能采用逼迫方式，让自己动起来，直到有一天发现自己精神饱满、身材健硕，还能 1 个打 8 个，这时才能体会到健身给自己带来的好处。TDD 亦是如此，唯有不断的练习，让行为改变思维，在这个过程中发现它的价值，胜过无数的文章。



## 资料

https://www.agilealliance.org/glossary/atdd/

[《测试驱动开发的艺术》](https://book.douban.com/subject/5326182/)