---
title: WebP与Nginx实践
date: 2016-09-01 18:44:48
tags: 
---
​	WebP 是谷歌推出的适合于 Web 使用的图像格式，在保持同样质量的情况下，可比 JPG 减小 40% 的体积，好处一搜一大堆，这里就不做过多介绍啦。至于问题你肯定也早有耳闻，那就是兼容性比较差，除了Google自己的产品支持以外最近只有Opera在支持，[点这里查看结果](http://caniuse.com/#search=webp)。

​	兼容性不好我们就放弃吗？我们能吗？我们能吗？我们能吗？**不能！**我们还是仍然可以在Android和Chrome上使用，并期待着IOS未来的支持，你肯定会想这一定需要很多的工作量去调整代码，放心我肯定不会写那种劳民伤财的文章，Nginx就可以帮助我们很简单的解决此问题而且不需要修改一句代码，一句代码！！！

```nginx
# http config block
map $http_accept $webp_ext {
	default "";
	"~*webp" ".webp";
}
# server 配置
# 尝试webp
location ~ ^/static/.+\.(jpg|jpeg|png)$ {
	add_header Vary Accept;
	try_files $uri$webp_ext $uri =404;
	root /hhapp/www/;
}
```

当支持`webp`的时候我们就在nginx上添加.webp后缀，不支持的就返回原有的文件，这里准备了两个文件，为了看出效果这两个文件的内容并不相同：

http://yuntest.haihaigame.com/static/images/1.png

http://yuntest.haihaigame.com/static/images/1.png.webp

打开1.png的链接在支持WebP设备你会看到webp的图片，而不支持的设备我们看到的仍然是png

#### 总结

这次的分享实在是过于简单，只不过网上的方案实在过于复杂，所以才写了这篇文章。这样做的好处就是对现有的代码根本无需更改，只需将图片再生成一份.webp格式即可，但没办法用在CDN上大量使用，除非那个CDN厂商能够提供支持，对于内部系统倒是个不错的选择。
后记：有个同事研究了一个另类的方法，在前端构建时生成两套css，通过应用服务器做了判断加载不同的css，当然这也只能限定是我们前端项目中的图片资源，而非用户上传的图片资源。