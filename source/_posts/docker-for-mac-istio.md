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


　　Service Mesh 在过去的一年的迅猛发展，各大厂商都在投入精力开发适合自己的产品。而对于小厂来说虽不能自己开发，但选择也有很多，[Linkerd](https://linkerd.io/)、[Envoy](https://www.envoyproxy.io/)、[Istio](https://istio.io/) 、[Conduit](https://conduit.io/) （Linkerd 2.0），甚至是 [Consul](https://www.consul.io/) 都在开发 Service Mesh。当中 Istio 因为出身名门和优秀的设计在众星之中脱颖而出，社区纷纷站队表示支持，尤其是发布 1.0 版本以后，更是引来众多关注。

　　Istio 虽然可以脱离 Kubernetes 运行，但从官方投入的精力和社区上的资料，都是基于 Kubernetes，如果不想采坑，还是老老实实的折腾 Kubernetes 吧。蚂蚁金服的 Jimmy song 创建了一个 [kubernetes-vagrant-centos-cluster](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster) 项目，可以帮助我们很容易的启动 Kubernetes 集群。唯一不足是启动时会比较耗资源，而 Docker 的新版本也同样支持了 Kubernetes，于是便有了这篇文章。

## 安装 Kubernetes

　　在 Docker 18.06.0 的增加对 Kubernetes 的正式支持（在之前的版本也有支持，只是非正式版本）

![docker支持kubernetes](http://upload-images.jianshu.io/upload_images/5588689-ca3d4fe3a3f022ad.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里调整了 docker 的内存为 4GB，之前默认 2GB 运行 Kubernetes 感觉会很吃力。

![调整内存设置](http://upload-images.jianshu.io/upload_images/5588689-410edb46c7ac6d8d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

找到 Kubernetes 选项，勾选 Enable 选择 Kubernetes，然后执行 Apply

![开启kubernetes](http://upload-images.jianshu.io/upload_images/5588689-bb3757788b04ccd8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![启动中...](http://upload-images.jianshu.io/upload_images/5588689-2cbbff841e81461a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们会看到 Kubernetes 一直在 starting... ，此时 docker 正在下载镜像，殊不知在遥远的东方有一堵“墙”，下载需要的镜像越过墙才可以。

![无奈](http://upload-images.jianshu.io/upload_images/5588689-38ce4b0cf71199ff?imageMogr2/auto-orient/strip)

　　要相信这个世界上总会有人与你一样遇到相同的问题，于是这个人就写了一个 [github](https://github.com/maguowei/k8s-docker-for-mac) 仓库。按照文档所说，我们需要配置一下国内的代理，然后执行下载镜像脚本，再重新启动 Kubernetes ，Kubernetes 就这样奇迹般的启动起来了。

### 安装 kubectl

kubectl 是 Kubernetes 的客户端

``` shell
brew install kubernetes-cli
# 或者更新
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

![Dashboard](http://upload-images.jianshu.io/upload_images/5588689-a430ac08b26f868d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

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

如果 Helm 版本小于 2.10.0 ，请通过 kubectl apply 安装 Istio，并等待几秒钟，以便在kube-apiserver 中提交CRD：

``` shell
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
kubectl apply -f install/kubernetes/helm/istio/charts/certmanager/templates/crds.yaml
```

[官方文档](https://istio.io/docs/setup/kubernetes/helm-install/)提供了安装几种方式，Option 1 使用 `helm template` 安装，可选的东西比较少。所以我们选择  **Option 2**。

**注意：这2个选项是互斥的，只能二选一哦。**

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

*PS: kiali pod 部署时会无法正常启动，不影响使用，后面我们会再提到。*

## 部署 Bookinfo

我们来部署一个官方的 [Bookinfo Examples](https://istio.io/docs/examples/bookinfo/)，进入 istio 的目录

``` shell
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
```

确认一下 service 和 pod 是否正确启动了

```shell
kubectl get services
```

![get services](http://upload-images.jianshu.io/upload_images/5588689-6bd897f7cdfd6412.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```shell
kubectl get pods
```

![get pods](http://upload-images.jianshu.io/upload_images/5588689-6758514dfb359e95.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们来创建网关

```shell
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

再 check 一下

```shell
istioctl get gateway
```

![gateway](http://upload-images.jianshu.io/upload_images/5588689-d1e089e7d7a81957.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**注意！** 后面会和官方文档不太一样啦，官方会去获取 ingress 的 ip 和端口，我们使用的 Docker for Mac 不需要查看映射端口，在 Dashboard 上找到 namespace 选择为 istio-system ，就可以看到我们映射的端口。

![查看 gateway 端口](http://upload-images.jianshu.io/upload_images/5588689-5c830a006ccb23bf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

激动人心的时刻到啦，访问这里：[http://localhost/productpage](http://localhost/productpage)

就可以看到 Bookinfo 的demo啦！

![Bookinfo demo](http://upload-images.jianshu.io/upload_images/5588689-fb81580eb6e2c72c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 卸载

可以直接参考[官方文档](https://istio.io/docs/examples/bookinfo/#uninstall-from-kubernetes-environment)啦！

## 分布式跟踪-Jaeger

开启 Jaeger 网络映射

```shell
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686:16686 &
```

访问 http://127.0.0.1:16686/

![Jaeger UI](http://upload-images.jianshu.io/upload_images/5588689-6ecad2732e7beae9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以点开具体的一次Trace来查看链路情况

![Trace](http://upload-images.jianshu.io/upload_images/5588689-aa7903b9d01c11b0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

更多好玩的东西请参考[官方文档](https://istio.io/docs/tasks/telemetry/distributed-tracing/)



## 使用Grafana 查询指标

先看来看我们的 Prometheus和 Grafana 是否正常

```shell
kubectl -n istio-system get svc prometheus
kubectl -n istio-system get svc grafana
```

![Prometheus和 Grafana 状态](http://upload-images.jianshu.io/upload_images/5588689-6d46e49ad694b224.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

开启 Grafana 网络映射

```shell
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &
```

访问 http://localhost:3000/d/LJ_uJAvmk/istio-service-dashboard?refresh=10s&orgId=1

![Grafana](http://upload-images.jianshu.io/upload_images/5588689-910d34300c27e5b6.gif?imageMogr2/auto-orient/strip)

更多好玩的东西请参考[官方文档](https://istio.io/docs/tasks/telemetry/using-istio-dashboard/)



##  kiali

[kiali](https://www.kiali.io/) 目前还在开发当中，所以不能用于生产，在Istio 默认不被开启，不过玩玩还是可以的。前面提到 kiali 在部署的时候无法启动，查看了一下原因是拉取的镜像为`docker.io/kiali/kiali:istio-release-1.0`，而Docker hub 中根本没有这个 Tag ...

![docker hub](http://upload-images.jianshu.io/upload_images/5588689-1cf2570e512de3b9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以抱着试试看的态度，在 Dashboard 手动改一下 tag 为 latest ，更新！
![修改tag](http://upload-images.jianshu.io/upload_images/5588689-ee8a400e67f7e35c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
开启映射网络端口

```shell
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001 &
```

访问：[http://localhost:20001/](http://localhost:20001/) 
账号密码：admin/admin

![kiali UI](http://upload-images.jianshu.io/upload_images/5588689-0da755ffe73454f2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



看起来还是蛮酷的，更多细节可以参考 [Kiali 官方文档](https://www.kiali.io/gettingstarted/)



## 停止Kubernetes

![停止Kubernetes](http://upload-images.jianshu.io/upload_images/5588689-78a53befe761a795.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在不需要的时候我们可以将Kubernetes 停止，以保证我们 Mac 的性能，在安装了太多的组件后会比较耗电。





## 一切都可以重来...

![重新开始](http://upload-images.jianshu.io/upload_images/5588689-52e98ad0743082cf?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Docker for Mac 还提供了一个非常人性的功能——**Reset**

![一切都可以重来...](http://upload-images.jianshu.io/upload_images/5588689-7ce0596f08c4a60f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

无论我们是安装过程出了问题还是需要做各种测试，只需要轻轻点击 【Reset Kubernetes cluster】一切就重新开始。



## 总结

　　Istio 的 example 还有很多可以玩的，比如限流、故障注入、retry 等等，后面有机会再和大家分享。教程类的文章总有时效性，尤其像发展迅猛的Istio ，所以如果有安装失败的同学可以给我留言，反正我也不会改的。
![见笑啦](http://upload-images.jianshu.io/upload_images/5588689-b47d91ed299b4fff?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
今天就到这里啦，谢谢大家。