---
title: Docker 初探二
date: 2016-07-02
tags: 
---
## 容器之间的互联

> 上周我们讲过可以通过`--add-host`来添加容器host达到动态访问外部服务的目的，其实容器之间的东西可以有更方便的方式`--link`

```shell
# 先运行起一个容器hello
docker run --name hello:1.0
# 在运行一个需要link的容器
docker run -it --link hello:hello  alpine cat /etc/hosts
```

会发现输出的结果会自动将hello的host添加上

```shell
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	hello 641b0f8bd474
172.17.0.3	3d0b57a24fd1
```

## 在已经运行容器上执行命令

> 刚发现的一个命令，还是蛮有用的：

```shell
# 查看正在运行的容器
docker ps
# 查看容器的hosts
docker exec -it c8a95d04b606 cat /etc/hosts
```

