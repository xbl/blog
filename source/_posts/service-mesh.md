---
title: Service Mesh 简介
date: 2018-07-17 10:21:12
tags:
---

　　在 2016 年和 2017 年微服务和容器化得到迅速普及，成为时下最火的技术。以 Spring cloud 为代表的侵入式微服务框架占据主流地位，一度成为微服务的代名词。

　　这里我们需要留意一下，似乎没有听说过其他语言的微服务框架，例如 PHP，这个问题我们后面再讨论。

　　微服务的好处自然是不用多说：

* 独立的可扩展性
* 跨语言
* 故障和资源隔离
* 独立运行、发布
* 软件架构与组织架构一致
* ...

　　好处如此之多，但是我们观察到真正将微服务落地的企业并不多，我们来看一下微服务的现状。

...

　　传统企业采用互联网技术，对现有技术转型。无论是 Spring Cloud 还是阿里的 Dubbo 使用门槛都很高，这还仅仅是 Java 技术栈，其他语言落地恐怕更难。其次是传统企业缺乏互联网技术的基因，缺乏有相关技术的人才。

## 微服务的困难

　　当我们要实践微服务的时候我们要考虑做哪些事情？

- 服务注册、发现
- 链路跟踪
- 负载均衡
- 超时
- 熔断
- 限流
- 健康检查
- 版本控制
- 日志收集
- …

　　抛开分布式事务我们仍然要这么多的工作，根据社区落地经验，大概会有十分之一的人员投入在这些工作上的。更困难的是作为一个刚入职的新员工想要了解这些技术全貌需要花费大量的时间。

　　Spring cloud 提供了很多的组件列表可供使用，看起来就像使用乐高的积木搭建飞船一样，复杂的让人难以下手，而且还没有乐高有趣。

...

　　搭建微服务并不是我们的目的，对吧，我们的目的是更快更好的实现业务价值。我们真正需要是一个可用时间使用的"飞船"。

## Service Mesh 的诞生

　　于是在世界的一个角，有一个叫做 William Morgan 的基础设施工程师想明白了，再也不能这样活。于是创办了 Buoyant 公司专门研发 Service Mesh。那个时候 Service Mesh 这个词汇还只在内部使用。在2017年4月发表博文，并给 Service Mesh 一个权威的定义：

> A service mesh is a dedicated **infrastructure layer** for handling service-to-service communication. It’s responsible for the reliable delivery of requests through the complex topology of services that comprise a modern, cloud native application. In practice, the service mesh is typically implemented as an array of lightweight network proxies that are deployed alongside application code, without the application needing to be **aware**. 

>Service Mesh 是一个**基础设施层**，用于处理服务间通信。云原生应用有着复杂的服务拓扑，Service Mesh 负责在这些拓扑中**实现请求的可靠传递**。在实践中，Service Mesh 通常实现为一组轻量级的网络代理，它们与应用程序一起部署，而对**应用程序透明。** 

　　我们看到刚刚的那一个巨长的组件列表，都做为微服务的基础设施层，我们的开发者不再需要花精力关心，可以更专注于业务价值。这个基础设施对应用是无感知的。

### Sidecar 模式

　　这里不得不提的是 Sidecar 模式，我们年纪大一点的同学见过下面图片的挎斗摩托车，当然啦，我只在电视上见过。在二战时期被大量使用，通常配置一个骑手和一个射击手，骑手不会直接发起攻击，只与射击手通讯，而射击手负责对外发起攻击。而且骑手与射击手拥有共同的生命周期。

...

　　Sidecar 模式就是如此，应用程序并不直接对外通讯，通过 Sidecar 与外部进行通讯。当应用销毁 Sidecar 也一同被销货。

### Service Mesh 都做了什么？

...

　　图中的 Linkerd 作为 Sidecar 与应用一同部署，所有服务间的调用都通过 Sidecar 来完成，Sidecar 拥有服务发现的能力。

...

　　来看一个稍微实际一点的例子。

...

　　当我们部署大量的服务就形成了网格。与Sidecar不同，不单单只是一个组件，而是由代理互相连接形成的网络。看到这张图我们就能够理解为什么它被叫做”Service Mesh“。

## Service Mesh的发展

　　我们看来看看 Service Mesh 的发展：

### Linkerd

　　前面提到了 Buoyant 公司，Service Mesh 名词的创造者，创造了第一个 Service Mesh 产品 Linkerd，使用 Scala 进行编写，2016年1月15 发布 0.0.7 版本。

### Envoy

　　来自 Lyft 公司，使用 C++ 编写，在2016年9月13日发布1.0版本。由于C++语言的优势，Envoy 在性能方面要强过 Linkerd。最近已经有很多公司在使用 Envoy 代替 Nginx，足以证明其性能和稳定性。

以上算是第一代的Service Mesh

### Istio

　　是由 Google 和 IBM 联手打造，同时使用 Envoy 作为 Sidecar，使用 Golang 编写，2017年5月24发布0.1版本，Google 和 IBM 高调宣讲，社区反响热烈。Istio的初期版本只支持K8s，在 0.3 版本开始对非 K8s平台支持。

### Conduit

　　Linkerd 被 Istio 的冲击已经先驱变先烈了，但是我们要注意，Buoyant 就是为了 Service Mesh 而生的，如果它不在 Service Mesh 一条路走到黑，恐怕也没有其他选择。于是Buoyant 绝地反击，在2017年12月5日发布 0.1.0 版，借鉴了 Istio 的架构设计，增加了控制面板。之前踩过性能的坑，Conduit 跳过了 Golang，直接选择 [Rust](https://www.rust-lang.org/zh-CN/)。所以后面的发展暂时还不可预见，有待观望。



## Istio 成为巨星

　　出身名门的 Istio 自然成为了这场战争中的赢家，优秀的架构设计，增加了控制面板