---
title: css3图片去色
date: 2017-08-04
tags: css
---

　　近日遇到这样一个场景，菜单中的图片是后台管理员上传的纯色`PNG`图片，效果看似`iconfonts`，选中状态只是改变字的颜色，效果如下：

<iframe height='265' scrolling='no' title='filter' src='//codepen.io/xbl/embed/OjbbKy/?height=265&theme-id=0&default-tab=result,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/xbl/pen/OjbbKy/'>filter</a> by javascript.xie (<a href='https://codepen.io/xbl'>@xbl</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

　　突然有一天，一个部门的同事说这个选中状态不够明显，要做一个明显反选中的状态，我擦，这可是图片啊大哥...吐槽归吐槽，但问题还是要解决的，总不能让管理员在后台再上传一张选中状态的图片吧，首先想到`CSS3`中的`filter`可以将图片变成黑白，我们看看[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/filter)还有哪些属性。

　　在文档中我们可以看到`brightness`(亮度)这个属性似乎可以帮助我们解决这个问题，但是写上去发现我们的图片有点虚，`filter`允许我们写多个属性，例如：

```css
filter: contrast(175%) brightness(3%);
```

​	我们再修改一下对比度试试:

<iframe height='265' scrolling='no' title='css3-filter2' src='//codepen.io/xbl/embed/PKbmqV/?height=265&theme-id=0&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/xbl/pen/PKbmqV/'>css3-filter2</a> by javascript.xie (<a href='https://codepen.io/xbl'>@xbl</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

　　

