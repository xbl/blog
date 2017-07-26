---
title: Docker 初探一
date: 2016-07-01 
tags: 
---
## 简介

>​	本来不太想写这段（因为不一定对...），介于有些同学之前没太了解过，所以就按照我自己的理解简单介绍一下，docker是一个虚拟化工具，可以根据自己的需求构建出一个最小化可运行的镜像(Image)，镜像的好处就是所有的对外运行方式都是统一的，运维同学不再为运行环境和安装软件版本等等问题困扰，测试也不会出现环境不一致而引发的bug。

## 构建镜像-编写Dockerfile

```shell
# 拉取一个基础的镜像
FROM alpine
# 系统更新
RUN apk update && apk upgrade
# 安装nodejs
RUN apk add nodejs
# 设置工作目录
WORKDIR /app
# 将当前系统目录添加到镜像中
ADD . /app
# 运行shell命令，执行安装npm
RUN npm install
# 端口映射，只起到记录作用，告诉用户应该映射哪个端口
EXPOSE 8080
# 运行时执行的命令
ENTRYPOINT [ "node", "server.js" ]

```

## Build 镜像

```shell
docker build -t <tag name>:<版本号> . 
docker build -t test/node:1.0 . 
# -t 创建tag
# . 当前的Dockerfile
```

## 启动容器

```shell
# 最简单的启动
docker run <tag>:<版本号>
```

常用参数：

* --add-host 添加host映射，如：` --add-host cn.bing.com:172.16.254.25`，可以添加多个host，如：`docker run -it --add-host svca:10.0.0.9 --add-host svcb:10.0.0.9 ubuntu cat /etc/hosts`
* -p 端口映射，<host端口:容器端口>，如：`-p 8080:8080`
* -v 文件路径映射，<host路径:容器路径>，如：`-v /usr/local/app:/app`
* -d 后台运行
* --name 指定容器名称，如：`--name hello`
* 更多参数查看`docker run --help` 或者[官方文档](https://docs.docker.com/engine/reference/run/)

## 容器运行状态

```shell
# 只查看运行中的
docker ps
# 查看所有的容器，包括已经停止的
docker ps -a
# 只查看停止的
docker ps -q
```

## 停止/重启容器

```shell
docker stop <容器id|容器名字>
docker start <容器id|容器名字>
docker restart <容器id|容器名字>
```

## 删除容器

```shell
# 删除已经停止的容器
docker rm <容器id|容器名字>
# 删除正在运行的容器
docker rm -f <容器id|容器名字>
```

### 微镜像 Alpine

> ​	微镜像的好处之一是我们的应用程序只需要一个很小的运行环境即可，其他并不是应用所需要的，所以没必要将机器的性能消耗在系统上，好处之二，当系统的软件越多，安全性就会越低，我们镜像上可能压根就没有ssh，即便是有了安全漏洞也与镜像没关系。

http://www.alpinelinux.org/

https://pkgs.alpinelinux.org/packages

## 云服务

[时速云](https://www.tenxcloud.com/)

[AWS](https://aws.amazon.com/cn/)

等...

## 总结

​	今天讲的内容只是冰山一角，可能连Docker的十分之一都不到，所以并不保证以上内容都是对的，哈哈！

​	有了Docker我们可以做的事情就非常的多，实现快速部署非常的容易，我们也可以根据当前的运行状态动态启动容器，再做一个服务发现机制，服务就完全自动化起来了。