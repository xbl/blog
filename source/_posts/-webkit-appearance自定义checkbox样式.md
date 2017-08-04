---
title: -webkit-appearance自定义checkbox样式
date: 2017-08-04
tags: css
---

　　在工作中美术老师给我们前端的设计稿是这样的:

![](/assets/images/checkbox/1.png)

　　而实际的效果却是这样的：

![](/assets/images/checkbox/2.png)

　　接下来你会见到一向平易近人的美术老师和你说：“我擦！怎么会是这个样子，丑哭了有没有，为什么不按我的设计稿去做？？？”

　　通常遇到这样的情况我们首选想到的是自己写一个`js`模拟`checkbox`，这对于我们大前端来说根本不什么难事，可是这样增加了程序的复杂度，原本只是多选框就可以搞定问题，我们却用js去实现，有没有只用css去实现的办法呢？那么就是我们今天要分享的一个私有属性：[-webkit-appearance](https://developer.mozilla.org/zh-CN/docs/Web/CSS/-moz-appearance)。

　　先看看MDN怎么说的：

> *请不要将这个属性用于网页中：不仅是因为这个属性不是W3C标准, 也是因为这个属性在各个浏览器中的表现不一致。甚至在同一个元素上将这个属性的值设为none，也会在不同浏览器中产生差别。并且一些浏览器也不支持这个属性*

这根本就是不能用嘛.../(ㄒoㄒ)/~~，再看看[Can I use](http://caniuse.com/#search=appearance) 的结果：

![](/assets/images/checkbox/3.png)

　　只需要带上`-webkit`前缀，将这个属性设置为`none`，在移动设备上支持的比较好，已经满足了我们的需求，然后我们再设置上宽高和边框：

<iframe height='265' scrolling='no' title='custom-checkbox-1' src='//codepen.io/xbl/embed/vJyXVd/?height=265&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/xbl/pen/vJyXVd/'>custom-checkbox-1</a> by javascript.xie (<a href='https://codepen.io/xbl'>@xbl</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
接下来的事情一定难不住你，完整代码：

```css
input[type=checkbox]{ 
    border: #e0e0e0 solid 1px;
    border-radius: 3px;
    -webkit-appearance: none;
    appearance: none;
    width: 17px;
    height: 17px;
    outline: 0;
    position: relative;
    vertical-align: middle;
    background: #fff;
}
input[type=checkbox]:checked {
    background: #fff;
}
/* 选中状态使用before来模拟 */
input[type=checkbox]:checked:before {
    content: '';
    background: #ff660c;
    text-align: center;
    width: 60%;
    height: 60%;
    display: inline-block;
    border-radius: 3px;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
}
```
<iframe height='265' scrolling='no' title='NvbRmV' src='//codepen.io/xbl/embed/NvbRmV/?height=265&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/xbl/pen/NvbRmV/'>NvbRmV</a> by javascript.xie (<a href='https://codepen.io/xbl'>@xbl</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

　　虽然以上已经完成了我们的需求，但毕竟不是W3C的标准，说不定在未来的某一天就会被废除掉，使用这个属性还是要**慎重!**

