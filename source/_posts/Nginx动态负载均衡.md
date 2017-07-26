---
title: Nginx动态负载均衡
date: 2016-07-27 
tags: 
---
> 服务发现的一个基础功能

## 方案一

#### 安装环境需要的基础文件

```shell
yum install zlib zlib-devel openssl openssl-devel pcre pcre-devel -y
```



#### 安装LuaJIT

```shell
wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz
tar zxvf LuaJIT-2.0.4.tar.gz 
cd LuaJIT-2.0.4
make 
```



#### 安装Tengine

```shell
./configure --with-http_lua_module --with-luajit-lib=/usr/local/luajit/lib/ --with-luajit-inc=/usr/local/luajit/include/luajit-2.0/ --with-ld-opt=-Wl,-rpath,/usr/local/luajit/lib --with-http_dyups_module --with-http_dyups_lua_api --prefix=/hhapp/opt/tengine
```



```shell
# add
curl "http://127.0.0.1:8081/dynamic?upstream=zone_for_backends&add=&server=127.0.0.1:8089"
# remove
curl "http://127.0.0.1:8081/dynamic?upstream=zone_for_backends&remove=&server=127.0.0.1:8089"

# up
curl "http://127.0.0.1:8081/dynamic?upstream=zone_for_backends&server=127.0.0.1:8089&up="

# down
curl "http://127.0.0.1:8081/dynamic?upstream=zone_for_backends&server=127.0.0.1:8089&down="

# update
curl "http://127.0.0.1:8081/dynamic?upstream=zone_for_backends&server=127.0.0.1:8089&weight=15&max_fails=5&fail_timeout=5"


docker run -d -p 8088:8090 -e "server=127.0.0.1:8088" test/node:1.1
```



## 参考

https://help.aliyun.com/knowledge_detail/41336.html