https://github.com/kubernetes/minikube

## [安装minibube](https://github.com/kubernetes/minikube#installation)

安装时需要翻墙，会访问 https://storage.googleapis.com

[Quickstart](https://github.com/kubernetes/minikube#quickstart)

下载需要翻墙

下载成功，启动失败，干了一件蠢事，删除了vm的镜像

又执行了几次`minikube stop` `minikube delete`

重新`minikube start` 又下载了

正要准备尝试[此方法](https://github.com/kubernetes/minikube/issues/459#issuecomment-239157063)，还没有试



启动起来了，但是不能 run

停掉了vpn

重启一下试试

遇到这种问题你一定会找到这个[链接](https://github.com/kubernetes/minikube/issues/1224)，然而并没什么用

**失败了**

更新了 VirtualBox 的版本，竟然好了？？！！！

```shell
VBoxManage --version
5.2.6r120293
```

又尝试了另一种方式

~~需要安装 [xhyve driver](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#xhyve-driver)~~

```shell
minikube start --vm-driver=hyperkit --registry-mirror=https://registry.docker-cn.com
```

~~使用 xhyve 的成功率更好，也ok了！！！~~

这玩意有bug，指定了 vm-driver 不起作用，让我误以为使用 xhyve 是成功的...



## [安装 istio](https://istio.io/docs/setup/kubernetes/quick-start.html)

基本可以安装文档来操作，[中文文档](http://istio.doczh.cn/docs/setup/kubernetes/quick-start.html)有些过时，因为这东西比较新，所有的翻译或者个人的笔记都可能随时过时，建议大家看官方文档。

要注意一点执行

```shell
kubectl apply -f install/kubernetes/istio.yaml
```

之前一定要保证 istioctl 以及添加到环境变量中，否则运行不起来，不过可以通过卸载重装：

```shell
kubectl delete -f install/kubernetes/istio.yaml
```

启动 Docker with Consul 

``` shell
docker-compose -f samples/bookinfo/consul/bookinfo.yaml up -d
```

执行此脚本时会提示找不到 consul_istiomesh 网络，不过不用担心，后面提示出了相应的脚本，告诉我们如何创建一个 consul_istiomesh 网络：

```shell
docker network create consul_istiomesh
```

