---
title: POSTMAN API 测试工具
date: 2017-03-15 18:44:48
tags: 
---

​	API测试的工具有很多，之所以选择[POSTMAN](https://www.getpostman.com/)是因为使用简单，非常容易上手，包含了基本的断言和测试脚本。

## 安装

​	[官网](https://www.getpostman.com/)下载即可，建议使用桌面程序，Chrome插件虽然也可以但cookie使用还需要其他插件；





## 调试

​	呼出控制台需要使用快捷键`cmd + alt +c`(`ctrl + alt +c` Windows)，我们同样可以使用`console.log`来输出log；

## 导入数据文件

参考：http://blog.getpostman.com/2014/10/28/using-csv-and-json-files-in-the-postman-collection-runner/



## 内置的lib



参考：https://www.getpostman.com/docs/sandbox



## 命令行测试

https://www.npmjs.com/package/newman

```shell
# 帮助
newman run -h
# 执行
newman run ./集合文件.json -d ./iterations文件.json
```

