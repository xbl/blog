---
title: Crosswalk 跨平台开发
date: 2016-08-27 18:44:48
tags: 
---
​	这个话题早已经不是什么新闻，[Cordova](http://cordova.apache.org/) 已经到了6.x的版本，cordova 是基于的WebView开发，门槛对于咱们前端来说非常的低，但对于低版本的android WebView 能够支持的CSS和ES5的新API仍然是不确定的，而且WebView的处理能力依然很弱，所以就引出了今天的主角-[Crosswalk](https://crosswalk-project.org/index_zh.html)。

## 概述

​	Crosswalk是基于[开源的Chromium](https://www.chromium.org/Home)进行开发，是一款为HTML应用提供运行时环境的开源项目，同时它也扩展了一些Web平台的新特性，也就是说我们就像在Chrome一样做Native开发，不需要考虑低版本的兼容性，通过使用Crosswalk项目开发者可以：

* 使用主流和新兴的Web标准。
* 使用主流浏览器无法获取的试验性API。

## 开始-Android

​	其实[Crosswalk官方的文档](https://crosswalk-project.org/documentation/android_zh.html)已经足够好了，这篇完全是为了充数才写的文章，还是以Android为例。

### Java JDK

Oracle JDK下载安装地址：[http://www.oracle.com/technetwork/java/javase/downloads/](http://www.oracle.com/technetwork/java/javase/downloads/)

### Apache Ant

编译工具Apache Ant下载地址： [http://www.apache.org/dist/ant/binaries/](http://www.apache.org/dist/ant/binaries/) （版本1.9.3可以正常工作）

### 下载Android SDK

* Android Studio下载地址： [http://developer.android.com/sdk/index.html](http://developer.android.com/sdk/index.html)，[这里](http://www.androiddevtools.cn)是墙内链接。

* 下载后配置环境变量

  ```shell
  # 打开.bash_profile
  vim ~/.bash_profile

  # 配置ant 和 android 环境变量
  export PATH=$PATH:/Users/xbl/Documents/source/apache-ant-1.9.7/bin;
  export ANDROID_HOME=/Users/xbl/Documents/source/android-sdk-macosx
  export PATH=$ANDROID_HOME/platform-tools:$PATH
  export PATH=$ANDROID_HOME/tools:$PATH

  # 保存后执行source 使配置生效
  source ~/.bash_profile
  ```

* 启动SDK Manager

  ```shell
  # 执行android 命令打开SDK Manager
  android
  ```

  腾讯提供了一个SDK镜像([https://dsx.bugly.qq.com/](https://dsx.bugly.qq.com/))，里面有使用方法，速度非常快。

* 安装Nodejs

  安装适合您系统的node.js:[https://nodejs.org/en/download/](https://nodejs.org/en/download/). 其中附带安装npm。

* 安装crosswalk-app-tools

  ```shell
  sudo npm install -g crosswalk-app-tools
  ```

### 验证环境

```shell
crosswalk-app check android
```

## 创建应用

```shell
# 执行命令会帮助我们创建一个工程模板，也可以手动创建，第一次有些慢
crosswalk-app create com.baobao.game
# 编译应用
crosswalk-pkg com.baobao.game
```

会生成一个app目录，其中`manifest.json` 文件很重要，内如如下：

```shell
{
   "name": "应用名字", // 应用名字
   "xwalk_app_version": "0.1", // 版本号
   "start_url": "index.html", // 入口文件
   "xwalk_package_id": "com.haihai.game", // 包名
   "icons": [
     {
         "src": "icon.png",
         "sizes": "72x72"
     }
   ]
}
```

权限等等都需要在该文件上添加，详情请见[manifest说明](https://crosswalk-project.org/documentation/manifest_zh.html)。

### 编译应用

```shell
# 第一次编译同样会非常慢...
crosswalk-pkg <含有manifest文件的目录>
```

这样就会编译好arm和x86两个平台的APK，点击[这里](https://crosswalk-project.org/documentation/crosswalk-app-tools_zh.html)参考其他参数。

## USB调试程序

首先连接手机，在手机中打开【设置>开发者选型>打开USB调试】，执行如下命令：

```shell
# 查看是否有设备列表
adb devices
```

### 安装应用

```shell
adb install -r com.baobao.game-0.1-debug.armeabi-v7a.apk
```

`-r` 代表"reinstall"，为了调试重新安装。

### Chrome 调试

运行应用后，打开chrome在地址栏中输入`chrome://inspect/#devices`

会出现我们的应用，点击`inspect`就可以调试了。

## 总结

​	Crosswalk 并不是只能自己运行，可以作为Cordova的插件与运行，也可以在Native应用中代替WebView，好处还是蛮多的，除了折腾环境麻烦了一点，编写起来几乎没有门槛。随着Chromium不断添加新的特性，通过Crosswalk编译后的APK大小都在20MB以上，官方提供了[Crosswalk Lite](https://crosswalk-project.org/documentation/crosswalk_lite_zh.html) 可以减小应用体检，当然也会丢失一些功能。

​	Crosswalk 优缺点很明显，Css3 的动画在pc上如果性能不高，在手机上更是惨不忍睹，阿里的[Weex](http://alibaba.github.io/weex/index.html) 正是解决这个问题，下次我们一起玩玩Weex。