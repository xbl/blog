---
title: H5 复制到剪切板
date: 2018-04-10 14:18:26
tags:
- clipboard
- h5
---

　　最近在项目中遇到一个在【移动端网页中复制内容到剪切板】的需求，首先想到 H5 新的 API 似乎增加了 Clipboard 的支持，先来看看[兼容情况](https://caniuse.com/#search=clipboard)，这一看真是老泪纵横啊，时隔多年浏览器的支持竟然还这么的差。

{% asset_img 01.png can i use clipboard %}

　　这条路只能放弃，同事找到了一个第三方库-[clipboardjs](https://clipboardjs.com/) ，测试了一下可以完全兼容手机和 PC 。很好奇它是怎么做到的，拜读了源码，原来是重点是 **[document.execCommand](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/execCommand)** 。曾经在做富文本编辑器的时候使用过这个 API，竟然没有想起来，真是惭愧。再来看一下它的兼容性：

{% asset_img 02.png can i use document.execCommand %}

　　看着这片绿色就让人心情好，我们利用 [document.execCommand](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/execCommand) 先来自己写一个小demo：



<iframe height='465' scrolling='no' title='Clipboard1' src='//codepen.io/xbl/embed/pLGeGB/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/xbl/pen/pLGeGB/'>Clipboard1</a> by javascript.xie (<a href='https://codepen.io/xbl'>@xbl</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



　　这里的 demo 只有30行的 js 代码，就可以完成对 `Textarea` 标签的内容进行复制，需要注意的一点，移动端中只有通过事件触发才可以选中文本。当然，这还不够，我们需要更复杂的功能，能够复制所有元素的文本，我们再对 demo 改进一下：

<iframe height='565' scrolling='no' title='Clipboard2' src='//codepen.io/xbl/embed/pLGrEO/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/xbl/pen/pLGrEO/'>Clipboard2</a> by javascript.xie (<a href='https://codepen.io/xbl'>@xbl</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>



　　增加了20几行的代码，来讲一下思路：点击【Copy】时先获得元素的文本，再动态创建了一个 `Textarea` 标签，将元素的文本赋值给 `Textarea` 然后选中，执行 `document.execCommand('copy')` 复制到剪切板，之后删除 `Textarea` 标签，是不是 So Easy！

