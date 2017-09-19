---
title: angular4 笔记
date: 2017-08-15 11:22:00
tags:
- javascript
- angular
---

　　`cnpm`有坑！安装不会有问题，运行`npm test`会报错：

```shell
jasmineRequire is not defined
```

　　覆盖率和正确率报告出不过来，解决办法只要使用`npm install`或者使用`yarn install`就好了

https://github.com/angular/angular-cli/issues/6136

https://github.com/taras42/karma-jasmine-html-reporter/issues/24



　　[https://angular.cn/](https://angular.cn/)文档非常棒，不过有些API已经过时了，需要回到官方文档查询。在编写的过程中经常会遇到`RxJS`的操作符，需要查[文档](https://www.gitbook.com/book/btroncone/learn-rxjs/details)，英文的看着吃力没关系，[这里](https://rxjs-cn.github.io/learn-rxjs-operators/)还有中文文档。

　　部署这里对于初学者稍有一些复杂。

## 服务端渲染

在`Angular2`版本有一个专门做服务渲染的[universal](https://universal.angular.io/)项目，看样子目前还只支持`Angular2`，后来在[angular-cli]()找到了[这篇wiki](https://github.com/angular/angular-cli/wiki/stories-universal-rendering)。



## 最后

> ## Caveats
>
> - Lazy loading is not yet supported, but coming very soon. Currently lazy loaded routes aren't available for prerendering, and you will get a `System is not defined` error.
> - The bundle produced has a hash in the filename from webpack. When deploying this to a production server, you will need to ensure the correct bundle is required, either by renaming the file or passing the bundle name as an argument to your server.

我还是帮大家翻译一下：

> ## 警告
>
> - 延迟加载目前还不支持，但是很快就可以。`prerendering`路由延迟加载还不可用，你会得到一个`System is not defined`错误。
> - webpack生成的文件会带有hash，当你部署到生成服务器时要确保引入正确的bundle文件，通过重命名文件或者将bundle文件名字作为参数传给服务器。

