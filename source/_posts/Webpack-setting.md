---
title: Webpack 使用笔记
date: 2016-05-27 15:49:35
tags: [JavaScript,webpack]
---

## 前言
最近在学习React，开始使用gulp+browserify的方式，对于SPA应用来说，总是没有那么顺手。看到最近比较火的webpack，被它强大的插件库和loader惊艳到了，特别是可以在js中import css这样写组件岂不是很方便。就开始学习一下，当然要从官网Docoument开始学习。
webpack 支持命令行配置和webpack.config.js配置。命令行配置不太适用大型项目更适合小型的demo应用，在这里只学习webpack.config.js配置。

## 简单样例
- webpack.config.js
``` js
var webpack = require('webpack');

module.exports = {
    context: __dirname + "/app",
    entry: "./entry",
    output: {
        path: __dirname + "/dist",
        filename: "bundle.js"
    }
}
```

## context
为`entry`设置的基础目录绝对地址，默认 `process.cwd()`;

## entry
要构建的文件入口点，可以是三种类型：
1. `String` 一般是SPA应用的入口点
2. `Array` 配置中所有的文件都会被加载，最后一个文件将会作为出口
3. `Object` 适用于多入口页面配置，key 将会作为文件块的名称，属性值可以配置 String 或 Array

*字符串可以是路径，也可以是node模块名*

## output
输出配置，需要注意的是，多入口点的情况，只有一个输出配置。
如果要使用哈希（[hash] or [chunkhash]），必须确定一个统一的模块顺序。
#### output.filename
有单入口输出和多入口输出的区别：
1. 单入口点

{% jsfiddle 0htedLra 'js,resources,html,css,result' %}