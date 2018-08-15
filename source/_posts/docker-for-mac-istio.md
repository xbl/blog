---
title: Docker for mac 安装 Istio
date: 2018-08-15 10:36:54
tags:
- Istio
- service mesh
- docker
- kubernetes
- k8s 
---

　　Service Mesh 在过去的一年的迅猛发展，各大厂商都在投入精力开发适合自己的产品。而对于小厂来说虽不能自己开发，但选择也有很多，[Linkerd](https://linkerd.io/)、[Envoy](https://www.envoyproxy.io/)、[Istio](https://istio.io/) 、[Conduit](https://conduit.io/) ，甚至是 [Consul](https://www.consul.io/) 都在开发 Service Mesh。当中 Istio 因为出身名门和优秀的设计在众星之中脱颖而出，社区纷纷站队表示支持，尤其是发布 1.0 版本以后，更是引来众多关注。

　　Istio 虽然可以脱离 Kubernetes 运行，但从官方投入的精力和社区上的资料，都是基于 Kubernetes，如果不想采坑，想玩玩还需要一定的门槛。蚂蚁金服的 Jimmy song 创建了一个 [kubernetes-vagrant-centos-cluster](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster) 项目，可以帮助我们很容易的启动 Kubernetes 集群。唯一不足是启动时会比较耗资源，而 Docker 的新版本也同样支持了 Kubernetes，于是便有了这篇文章。

## 安装 Kubernetes

　　在 Docker 18.06.0 的增加对 Kubernetes 的正式支持（在之前的一些版本也有支持，只是非正式版本）

![docker支持kubernetes](https://ws4.sinaimg.cn/large/006tNbRwly1fua6p3zwgjj312w0ykguy.jpg)

这里调整了 docker 的内存为 4GB，之前默认 2GB 运行 Kubernetes 感觉会很吃力。

![调整内存设置](https://ws2.sinaimg.cn/large/006tNbRwly1fua70gky5qj30dw0ct0tx.jpg)

找到 Kubernetes 选项，勾选 Enable 选择 Kubernetes，然后执行 Apply

![开启kubernetes](https://ws2.sinaimg.cn/large/006tNbRwly1fua6r4btrhj30rs0lan16.jpg)

![启动中...](https://ws4.sinaimg.cn/large/006tNbRwly1fua6sdxfmpj30rs0la0wo.jpg)

我们会看到 Kubernetes 一直在 starting... ，此时 docker 正在下载镜像，殊不知在遥远的东方有一堵“墙”，下载需要的镜像越过墙才可以。



　　要相信这个世界上总会有人与你一样遇到相同的问题，于是这个人就写了一个 [github](https://github.com/maguowei/k8s-docker-for-mac) 仓库。按照文档所说，我们需要配置一下国内的代理，然后执行下载镜像脚本，再重新启动 Kubernetes ，Kubernetes 就这样奇迹般的启动起来了。

### 安装 kubectl

kubectl 是 Kubernetes 的客户端

``` shell
brew install kubernetes-cli
# 更新
brew upgrade kubernetes-cli
```

### 安装 Kubernetes dashboard

```shell
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

#### 启动 proxy

```shell
kubectl proxy
```

访问这里：[http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)

![Dashboard](https://ws2.sinaimg.cn/large/006tNbRwly1fuacyriux9j31kw0z1q95.jpg)

如果是想玩玩单点的 Kubernetes 到这里就结束啦~


## 安装 Istio


先[下载 Istio 最新版本](https://github.com/istio/istio/releases/)

找个你心仪的地方解压，然后配置环境变量：

```shell
export PATH="$PATH:/解压的目录/istio-1.0.0/bin"
```

如果放在 `.bash_profile` 或者 `.zshrc` 文件中记得要 source 一下

```shell
source ~/.bash_profile
```

验证一下是否生效

```shell
istioctl version
```

### 安装 Istio 

这里为了快速简单的搭建 Istio ，使用 helm 来帮助我们。

#### [安装 Helm](https://github.com/helm/helm#install)

Helm 是 Kubernetes 的包管理器

```shell
brew install kubernetes-helm
# 验证一下
helm version
```

然后我们安装 Istio 步骤

如果 Helm 版本小于 2.10.0 ，请通过 kubectl apply 安装 Istio，并等待几秒钟，以便在kube-apiserver 中提及CRD：

``` shell
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
kubectl apply -f install/kubernetes/helm/istio/charts/certmanager/templates/crds.yaml
```

[官方文档](https://istio.io/docs/setup/kubernetes/helm-install/)提供了安装几种方式，Option 1 使用 `helm template` 安装，可选的东西比较少。所以我们选择  **Option 2**。

**注意：这2个选项是互斥的。**

1. 如果还没有为 Tiller 配置 service account，请配置一个：

   ``` shell
   kubectl create -f install/kubernetes/helm/helm-service-account.yaml
   ```

2. 使用 service account 在您的集群中安装 Tiller

   ```shell
   helm init --service-account tiller
   ```

3. 安装 Istio

   ```shell
   helm install install/kubernetes/helm/istio --name istio --namespace istio-system \
   --set tracing.enabled=true \
   --set kiali.enabled=true \
   --set grafana.enabled=true
   ```

默认 tracing 、kiali 、grafana 并不会开启，这里需要在安装时手动 `--set xxx.enabled=true` 进行开启。配置说明可查看：install/kubernetes/helm/istio/README.md

## 部署 Bookinfo

我们来部署一个官方的 [Bookinfo Examples](https://istio.io/docs/examples/bookinfo/)，进入 istio 的目录

``` shell
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
```

确认一下 service 和 pod 是否正确启动了

```shell
kubectl get services
```

![get services](https://ws4.sinaimg.cn/large/006tNbRwly1fuacfscd99j30e803fdhy.jpg)

```shell
kubectl get pods
```

![get pods](https://ws1.sinaimg.cn/large/006tNbRwly1fuach5d9gvj30d104ctaz.jpg)

然后我们来创建网关

```shell
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

再 check 一下

```shell
istioctl get gateway
```

![gateway](https://ws1.sinaimg.cn/large/006tNbRwly1fuackwypktj309001h74m.jpg)

**注意！** 后面会和官方文档不太一样啦，官方会去获取 ingress 的 ip 和端口，我们使用的 Kubernetes 不需要查看映射端口，在 Dashboard 上找到 namespace 选择为 istio-system ，就可以看到我们映射的端口。

![查看 gateway 端口](https://ws1.sinaimg.cn/large/006tNbRwly1fuacp1zvs4j31kw0fqn2z.jpg)

激动人心的时刻到啦，我们访问这里：[http://localhost/productpage](http://localhost/productpage)

就可以看到 Bookinfo 的demo啦！

![Bookinfo demo](https://ws4.sinaimg.cn/large/006tNbRwly1fuacswz8ojj30w50eqtb7.jpg)

#### 卸载

可以直接参考[官方文档](https://istio.io/docs/examples/bookinfo/#uninstall-from-kubernetes-environment)啦！

## 分布式跟踪-Jaeger

开启 Jaeger 网络映射

```shell
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686:16686 &
```

访问 http://127.0.0.1:16686/

![Jaeger UI](https://ws1.sinaimg.cn/large/006tNbRwly1fuam6k95q4j31kw12kjzj.jpg)

可以点开具体的一次Trace来查看链路情况

![Trace](https://ws4.sinaimg.cn/large/006tNbRwly1fuam7ysud5j31dq13s109.jpg)

更多好玩的东西请参考[官方文档](https://istio.io/docs/tasks/telemetry/distributed-tracing/)



## 使用Grafana 查询指标

先看来看我们的 Prometheus和 Grafana 是否正常

```shell
kubectl -n istio-system get svc prometheus
kubectl -n istio-system get svc grafana
```

![Prometheus和 Grafana 状态](https://ws3.sinaimg.cn/large/006tNbRwly1fuameeksy8j30ds02xjsc.jpg)

开启 Grafana 网络映射

```shell
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &
```

访问 http://localhost:3000/d/LJ_uJAvmk/istio-service-dashboard?refresh=10s&orgId=1

![Grafana](https://ws2.sinaimg.cn/large/006tNbRwly1fuan6h4b5cg30og0ewnk6.gif)

更多好玩的东西请参考[官方文档](https://istio.io/docs/tasks/telemetry/using-istio-dashboard/)



## 停止Kubernetes

![停止Kubernetes](https://ws1.sinaimg.cn/large/006tNbRwly1fuamks81v5j30rs0la42j.jpg)

在不需要的时候我们可以将Kubernetes 停止，以保证我们 Mac 的性能，在安装了太多的组件后会比较耗电。





## 一切都可以重来...

![一切都可以重来...](https://ws1.sinaimg.cn/large/006tNbRwly1fuajl06i7bj30rs0r2jwr.jpg)

Docker for Mac 还提供了一个非常人性的功能——**Reset**

无论我们是安装过程出了问题还是需要做各种测试，我们只需要轻轻点击 【Reset Kubernetes cluster】一切就重新开始。



## 总结

　　Istio 的 example 还有很多可以玩的，比如限流、故障注入、retry 等等。大家可以尝试写个应用玩玩，谢谢。