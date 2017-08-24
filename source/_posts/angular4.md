---
title: angular4 笔记
date: 2017-08-15 11:22:00
tags:
- javascript
- angular
---

　　`cnpm`的坑，安装都不会有问题，但允许`npm test`就会报错：

```shell
jasmineRequire is not defined
```

　　覆盖率和正确率报告出不过来，解决办法只要使用`npm install`或者使用`yarn`

https://github.com/angular/angular-cli/issues/6136

https://github.com/taras42/karma-jasmine-html-reporter/issues/24



　　[https://angular.cn/](https://angular.cn/)文档非常棒，不过有些API已经过时了，需要回到官方文档查询。在编写的过程中经常会遇到`RxJS`的操作符，需要查[文档](https://www.gitbook.com/book/btroncone/learn-rxjs/details)，英文的看着吃力没关系，[这里](https://rxjs-cn.github.io/learn-rxjs-operators/)还有中文文档。

　　部署这里对于初学者稍有一些复杂