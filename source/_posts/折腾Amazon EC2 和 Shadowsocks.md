---
title: 折腾Amazon EC2 和 Shadowsocks
date: 2016-05-26
tags: 
---
> ​	一直都懒得折腾，始终用着`Lantern`，无奈`Lantern`的速度实在是无法忍受，而且经常连不上，所以只好靠自己啦；

## Amazon EC2

> ​	大家都知道的，介绍和注册我就不多说了，两年前写过一个[文章](http://my.oschina.net/xbl/blog/287218)可以看一下。要注意一点选择服务的地址是**【东京】**，经过测试**东京**要比首尔或者其他的更快。

### 创建密钥&创建实例

​	进入【EC2 控制面板】后可以看到【启动实例】按钮，但是先别着急，在创建实例之前我们要先在左侧的菜单【网络与安全】->【密钥对】，然后点【创建密钥对】，创建好后下载一个密钥文件，这个需要保持好，SSH登录的时候会用到。

![ec2start](https://cloud.githubusercontent.com/assets/1027448/15568349/fcaa4604-235d-11e6-80f4-ed61b5003015.png)

​	然后我们就可以创建实例了，这里选择免费套餐，选择自己熟悉的Linux，我这里选择的是【RedHat】

![ec2os](https://cloud.githubusercontent.com/assets/1027448/15568369/1767d4f2-235e-11e6-93e3-2e8c983fb498.png)

​	接下来就选择我们刚刚创建的密钥对，这时就启动我们的实例了，进入【实例面板】点击【连接】按钮上面写了如何通过SSH来连接到我们的实例上，通常即便实例已经启动也需要等个几分钟才可以连接上

### 修改root密码

​	默认我们是用ec2-user用户来登录的，而且不需要密码，但是安装其他软件需要用到root的密码，所以我们来设置一个：

```she
sudo passwd root
```

### 安装shadowsocks Server

​	[shadowsocks](https://shadowsocks.org/en/download/servers.html)可以有很多种安装方式，这里选择的是`shadowsocks-libev`也可参考[官方文档](https://github.com/shadowsocks/shadowsocks-libev#install-from-repository-1)

1. 下载[repo](https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/)文件到`/etc/yum.repos.d/`
2. 使用yum安装：

```shell
su -c 'yum update'
su -c 'yum install shadowsocks-libev'
```

### 配置shadowsocks

```shell
sudo vi /etc/shadowsocks/config.json
```

```json
{
  "server":"0.0.0.0",
  "server_port":8388,    // 服务器端口
  "local_port":1080,
  "password":"barfoo!",  // 认证密码
  "timeout":600,
  "method":"aes-256-cfb" // 加密方式，推荐使用aes-256-cfb
}
```

### 重启shadowsocks服务

```shell
sudo /etc/init.d/shadowsocks stop
sudo /etc/init.d/shadowsocks start
```

###  配置安全组

​	其实就是防火墙，默认我们的实例只开通了ssh的接口，是不允许其他接口访问的，所以我们要在这里加上一条配置，点击实例面板下方【安全组】进入安全组面板，选择【入站】选项卡，我这里偷懒就选择了所有流量，大家可以选择自己的类型和端口号，这样更安全：

![netgate](https://cloud.githubusercontent.com/assets/1027448/15598754/d0d08b12-240f-11e6-9121-238d9bdcfdc7.png)

### 结束	

​	到这里服务就已经安装完了，剩下的就是客户端了，客户端的安装都比较简单，**值得注意的一点当chrome安装了【SwitchySharp】插件的同学，一定要选择【使用系统代理设置】否则会一直不起作用**。
![switchysharp](https://cloud.githubusercontent.com/assets/1027448/15568386/31078b64-235e-11e6-8bdf-7f17e0d6ce4c.png)