---
title: ESLint 使用
date: 2016-03-14
tags: 
---

> ​	在团队不断扩张的时候，编码规范就变的非常重要，每个程序员都有自己的代码风格，以至于代码风格不一，写法各异，虽然我们会在新同学入职的时候做编码规范培训，然后并不是很有效，所以我们需要工具来帮助我们一起规范我们的代码。

## ESLint 安装

``` shell
# 全局安装
npm install -g eslint
# 或者只安装在项目里
npm install --save-dev eslint
```

## 使用

``` shell
# 第一次初始化项目的时候可以使用init来生成配置文件
eslint --init
# 全局安装可以直接运行eslint命令
eslint test.js test2.js
# 在项目里安装的则要麻烦一点
./node_modules/.bin/eslint test.js
```

## 配置

在运行`eslint --init`会在项目中根目录生成`.eslintrc.*`文件，编辑可以看到：

``` json
{
    "rules": {
        "semi": ["error", "always"],
        "quotes": ["error", "double"]
    }
}
```

* `"off"` 或者 `0` — 关闭规则验证
* `"warn"` 或者 `1` — 只警告不报错
* `"error"` 或者 `2` — 不符合规则就报错

其他配置看 [官方文档](http://eslint.org/docs/user-guide/configuring)会有很详细的解释;



## 全局变量

​	我们经常会用到第三方库，比如`jQuery`、`angular`、`wx`等等，在我们的js中会作为全局变量出现，但eslint 并不知道，需要我们配置一下:

``` json
{
    "env": {
        "browser": true
    },
    // 定义全局变量
    "globals": {
        "wx": true,
        // 百度地图api
        "BMap": true,
        // webkitblob关键字
        "WebKitBlobBuilder": true
    },
	"rules": {...}
```





## 查看报错

查看eslint 运行结果:

``` shell
210:28  error  Expected '===' and instead saw '=='                            eqeqeq
212:2   error  Mixed spaces and tabs                        no-mixed-spaces-and-tabs
```

这两行对应信息：

| 行号:列  | 类(warn/error) | 描述                                  | 对应的规则名称                  |
| ----- | ------------- | ----------------------------------- | ------------------------ |
| 212:2 | error         | Mixed spaces and tabs               | no-mixed-spaces-and-tabs |
| 215:2 | error         | Expected '===' and instead saw '==' | eqeqeq                   |

​	这样我们就可以很方便的定位代码的问题了，这里要强调一点列的位置会忽略代码前面的缩进，所以调试时如果不能很快定位问题可以将缩进先删除，至于规则我们也可以在[规则文档](http://eslint.org/docs/rules/)里找到正确的编码规范；

## 添加忽略文件

​	与git有些类似，需要添加一个`.eslintignore`文件，将忽略的文件名写进去就ok了

``` 
HHAPI-1.1.js
HHAPI/**
```

## 插件

以angular 的插件为例：

``` shell
# 先安装插件
npm install --save-dev eslint-plugin-angular
npm install --save-dev eslint-config-angular
```

编辑`.eslintrc.*`文件

``` json
{
    "env": {
        "browser": true
    },
  	// 插件
    "extends": ["angular"],
	"rules": {...}
}
```

## 使用gulp-eslint

只用命令行运行效率就太低了，我们可以利用`gulp.watch`来提高效率：

``` shell
npm install --save-dev gulp-eslint
```

### gulpfile.js

``` javascript
var gulp = require('gulp'),
    eslint = require('gulp-eslint');

// 当js变化则运行eslint
gulp.watch(['./js/**/*.js'], function (event) {
	gulp.src(event.path)
		.pipe(eslint())
		.pipe(eslint.format());
});	
```

更多功能[点击这里](https://www.npmjs.com/package/gulp-eslint)

## 规则配置

目前项目中使用的一个配置文件：

``` json
{
    "env": {
        "browser": true
    },
    "extends": ["angular"],
    // 定义全局变量
    "globals": {
        "wx": true,
        // webkitblob关键字
        "WebKitBlobBuilder": true
    },
    "rules": {
        // 使用tab缩进
        "indent": [2, "tab", {"SwitchCase": 1}],
        "linebreak-style": [2, "unix"],
        // 字符串必须使用单引号
        "quotes": [2, "single"],
        // 分号
        "semi": [2, "always"],
        "accessor-pairs": 2,
        "array-bracket-spacing": 0,
        "block-scoped-var": 0,
        "brace-style": [2, "1tbs", { "allowSingleLine": true }],
        // 使用驼峰
        "camelcase": 2,
        // json结尾不允许有逗号
        "comma-dangle": [2, "never"],
        "comma-spacing": [2, { "before": false, "after": true }],
        "comma-style": [2, "last"],
        "complexity": 0,
        "computed-property-spacing": 0,
        "consistent-return": 0,
        "consistent-this": 0,
        "constructor-super": 2,
        "curly": [0, "multi-line"],
        "default-case": 0,
        "dot-location": [2, "property"],
        "dot-notation": 0,
        "eqeqeq": [2, "allow-null"],
        "func-names": 0,
        "func-style": 0,
        "generator-star-spacing": [2, { "before": true, "after": true }],
        "guard-for-in": 0,
        "handle-callback-err": [2, "^(err|error)$" ],
        "key-spacing": [2, { "beforeColon": false, "afterColon": true }],
        "linebreak-style": 0,
        "lines-around-comment": 0,
        "max-nested-callbacks": 0,
        "new-cap": [2, { "newIsCap": true, "capIsNew": false }],
        "new-parens": 2,
        "newline-after-var": 0,
        "no-array-constructor": 2,
        "no-caller": 2,
        "no-catch-shadow": 0,
        "no-cond-assign": 2,
        "no-constant-condition": 0,
        "no-control-regex": 2,
        "no-debugger": 2,
        "no-delete-var": 2,
        "no-div-regex": 0,
        "no-dupe-args": 2,
        "no-dupe-keys": 2,
        "no-duplicate-case": 2,
        "no-else-return": 0,
        "no-empty": 0,
        "no-empty-character-class": 2,
        "no-eq-null": 0,
        "no-eval": 2,
        "no-ex-assign": 2,
        "no-extend-native": 2,
        "no-extra-bind": 2,
        "no-extra-boolean-cast": 2,
        "no-extra-parens": 0,
        "no-extra-semi": 0,
        "no-fallthrough": 2,
        "no-floating-decimal": 2,
        "no-func-assign": 2,
        "no-implied-eval": 2,
        "no-inline-comments": 0,
        "no-inner-declarations": [2, "functions"],
        "no-invalid-regexp": 2,
        "no-irregular-whitespace": 2,
        "no-iterator": 2,
        "no-label-var": 2,
        "no-labels": 2,
        "no-lone-blocks": 2,
        "no-lonely-if": 0,
        "no-loop-func": 0,
        "no-mixed-requires": 0,
        "no-mixed-spaces-and-tabs": 2,
        "no-multi-spaces": 2,
        "no-multi-str": 2,
        "no-multiple-empty-lines": [2, { "max": 1 }],
        "no-native-reassign": 2,
        "no-negated-in-lhs": 2,
        "no-nested-ternary": 0,
        "no-new": 2,
        "no-new-func": 0,
        "no-new-object": 2,
        "no-new-require": 2,
        "no-new-wrappers": 2,
        "no-obj-calls": 2,
        "no-octal": 2,
        "no-octal-escape": 2,
        "no-param-reassign": 0,
        "no-path-concat": 0,
        "no-process-env": 0,
        "no-process-exit": 0,
        "no-proto": 0,
        "no-redeclare": 2,
        "no-regex-spaces": 2,
        "no-restricted-modules": 0,
        // 返回必须有值
        // "no-return-assign": 2,
        "no-script-url": 0,
        "no-self-compare": 2,
        "no-sequences": 2,
        "no-shadow": 0,
        "no-shadow-restricted-names": 2,
        "no-spaced-func": 2,
        "no-sparse-arrays": 2,
        "no-sync": 0,
        "no-ternary": 0,
        "no-this-before-super": 2,
        "no-throw-literal": 2,
        // 空行里是否有空格或tab
        "no-trailing-spaces": 0,
        "no-undef": 2,
        "no-undef-init": 2,
        "no-undefined": 0,
        "no-underscore-dangle": 0,
        "no-unexpected-multiline": 2,
        "no-unneeded-ternary": 2,
        "no-unreachable": 2,
        "no-unused-expressions": 0,
        "no-unused-vars": [2, { "vars": "all", "args": "none", "wx": "none" }],
        "no-use-before-define": 0,
        "no-var": 0,
        "no-void": 0,
        "no-warning-comments": 0,
        "no-with": 2,
        "object-curly-spacing": 0,
        "object-shorthand": 0,
        "operator-assignment": 0,
        "operator-linebreak": [2, "after", { "overrides": { "?": "before", ":": "before" } }],
        "padded-blocks": 0,
        "prefer-const": 0,
        "quote-props": 0,
        "quotes": [2, "single", "avoid-escape"],
        // parseInt 基数
        "radix": 0,
        "sort-vars": 0,
        // { 前面有空格
        "space-before-blocks": [2, "always"],
        // 在函数名后面与括号有空格
        "space-before-function-paren": [2, "always"],
        "space-in-parens": [2, "never"],
        "space-infix-ops": 2,
        "space-unary-ops": [2, { "words": true, "nonwords": false }],
        "spaced-comment": [2, "always", { "markers": ["global", "globals", "eslint", "eslint-disable", "*package", "!"] }],
        "use-isnan": 2,
        "valid-jsdoc": 0,
        "valid-typeof": 2,
        "vars-on-top": 0,
        "wrap-iife": [2, "any"],
        "wrap-regex": 0,
        "yoda": [2, "never"],
        // 启用严格模式
        "strict": 2,
        // "comments/no-jsdoc": 2,
        // 不允许空函数
        "no-empty-function": 2,
        "angular/module-setter": 0,
        "angular/module-getter": 0,
        "angular/log": 0,
        // 自己修改规则
        "angular/controller-name": [2, "/[A-Z].*Ctrl/"],
        // 注入方式使用数组
        "angular/di": [2, "array"],
        // controller as
        "angular/controller-as-route": 0,
        "angular/controller-as-vm": 0,
        "angular/controller-as": 0
    }
}
```