---
title: npm 私有模块的3种方法
date: 2018-03-15 13:48:51
tags:
- npm
- nodejs
- 前端
- 私有模块
- 伺服
---



## 方法一：git + npm link



支持 npm install git 仓库

支持几种协议： `git` ,  `git+ssh` ,  `git+http` ,  `git+https` , or `git+file`.

``` shell
npm install git+<https://xxx.com/private-package.git>
```



#### 使用 Tag 控制版本

```shell
npm install github:<githubname>/<githubrepo>[#<commit-ish>]
```



参考官方文档：https://docs.npmjs.com/cli/install

## 方法二：CNPM



```shell
# clone
git clone https://github.com/cnpm/cnpmjs.org.git
# 进入目录
cd cnpmjs.org
# 执行docker-componse
docker-compose up -d
```



## 方法三：Nexus 私服



```shell
docker run -d --restart=unless-stopped  --name nexus \
-p 8081:8081 -p 5000:5000 -p 5001:5001 -p 5002:5002 -p 5003:5003 -p 5004:5004 \
--ulimit nofile=90000:90000 \
-e INSTALL4J_ADD_VM_PARAMS="-Xms2g -Xmx2g" \
sonatype/nexus3:3.8.0
```





参考：https://blog.sonatype.com/using-nexus-3-as-your-repository-part-2-npm-packages