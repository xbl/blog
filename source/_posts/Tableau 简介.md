---
title: Tableau 简介
date: 2017-11-20
tags:
- Tableau
- 数据可视化
---

　　在以往的项目中，我们自己做数据可视化都需要一些图表库，如：[ECharts](http://echarts.baidu.com/)、[Highcharts](https://www.highcharts.com/) 等等，由前端同学吭哧吭哧开发，切不论开发过程是否会出现bug，就单说开发周期和人员成本，以及后期的维护成本都相当可观。有没有一些更好的办法呢？[Tableau](https://www.tableau.com/) 是一个非常强大的数据的可视化工具，可以帮助我们通过**拖拽**的方式快速制作出一张图表，但它是一个商业软件。

Tableau 分为几个产品：

* **Tableau Desktop**

  用于将数据制作成可视化图表的工具，可运行在 Windows 和 Mac 上。可创建各种类型数据源，如：Excel，JSON，数据库等等，链接数据源后通过进行拖拽完成图表制作。

* **Tableau Server**

  用于共享图表和数据，将 Tableau Desktop 制作好的图表发布到 Tableau Server 即可完成共享。Tableau Server 目前只能安装在 Windows ，可以在 Tableau Server 配置用户权限，以方便管理。

* *Tableau Online*

  Tableau Online 是 Tableau Server 的 SAAS 版本，不需要我们自己购买硬件环境。

* *Tableau Mobile*

  可以连接到 Tableau Server 和 Tableau Online，提供在移动端查看图表。

* Tableau Public 

  Tableau Desktop 免费版本，支持的数据源较少，只包含文件系统，不支持数据库。可将制作好的图表发布到 Tableau Public 的 SAAS 平台上。




## 入门演示

> 为了大家能有一个初步的认识，我们做一个入门演示。

首先打开 Tableau Desktop，选择数据源链接：

![](/assets/images/tableau/01.png)

我们可以看到数据源中包含的所有表，将表拖拽至右侧：

![](./tableau/2.jpg)

也可以拖拽多个表，Tableau 会自动为我们将多张表建立**联接**，当两张表没有相同字段时，Tableau 会提示我们选择两张表的关联字段，以完成联接。

![](/assets/images/tableau/3.jpg)

此时会的得到两张表关联后的数据，点击左下角的【工资表】或者是 【Sheet】可以看到 工作表视图：

![](./tableau/4.jpg)

在【度量】上找到我们关注的指标双击，或者拖拽至**行**，再从【维度】中找到需要关注的维度，如: Date 拖拽至**列**：

![](/assets/images/tableau/5.jpg)

我们可以在【智能显示】中选择自己想要看到的图表。

## 总结

　　Tableau 制作一个图表相当简单，在快速迭代产品，得到用户反馈的角度 Tableau 算是一个不错的选择。相比自己开发要快的多，但作为商业软件它的价格的确不菲。



## 资料

[免费培训视频](https://www.tableau.com/zh-cn/learn/training)

[Tableau 帮助](https://www.tableau.com/zh-cn/support/help)

