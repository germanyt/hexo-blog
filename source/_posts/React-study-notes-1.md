---
title: React 学习笔记 - 1
date: 2016-05-16 17:39:37
tags:
---

## 文档
1. React [Document](https://facebook.github.io/react/docs/getting-started.html)
2. React-router [Document](https://github.com/reactjs/react-router/tree/master/docs)
3. Redux [Document](https://github.com/reactjs/redux)
4. Webpack [Document](http://webpack.github.io/)



## 简介
React SPA 开发，需要引入`React-router`控制路由，这里选择了`Redux`管理数据层，构建工具选择`Webpack`。

本节中包含React SPA 开发的相关配置和一个React的简单示例。

*Tips ：文章中所有示例代码使用`ES6`。*


## 目录结构

```
.
├──app
|   ├──js
|   |   ├──action
|   |   ├──components
|   |   ├──containers
|   |   ├──reducer
|   |   ├──store
|   |   └──app.js
|   └──index.html
├──package.json
├──server.js
└──webpack.config.js
```

## Layout

``` html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>react example</title>
</head>
<body>
	<div id="content"></div>
	<script src="/build/app.js"></script>
</body>
</html>
```
*Tips：示例中的样式使用bootstrap，在js中import，然后webpack构建*

## Webpack配置
1. webpack 配置写入 webpack.config.js 文件
2. 本地开发使用webpack-server启动web服务，监控文件变化

#### webpack.config.js
``` js
var path = require('path');
var webpack = require('webpack');

module.exports = {

  devtool: 'inline-source-map', // map 实际开发中，可使用NODE_ENV配置

  entry: path.join(__dirname, 'app', 'js', 'app.js'),

  output: {
    path: __dirname + '/build/',
    filename: 'app.js',
    chunkFilename: '[id].chunk.js',
    publicPath: '/build/'
  },

  module: {
    loaders: [
      { test: /\.(js|jsx)$/, exclude: /node_modules/, loader: 'babel', query: {
          presets: ['es2015','react']
        }
      }, //使用babel把 jsx和es6 转换为 es5
      { test: /\.css$/, loader: 'style!css' },
      { test: /\.(woff2?|svg)$/, loader: 'url?limit=10000' },
      { test: /\.(ttf|eot)$/, loader: 'file' },
    ]
  },

  resolve: {
    extensions: ['', '.js', '.jsx'] //处理文件扩展名
  },

  plugins: [
    // new webpack.optimize.CommonsChunkPlugin('shared.js'),
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
      __DEV__: JSON.stringify(JSON.parse(process.env.DEBUG || 'false'))
    })
  ]

}
```

#### webpack-server
``` js
var path = require('path')
var express = require('express')
var rewrite = require('express-urlrewrite')
var webpack = require('webpack')
var webpackDevMiddleware = require('webpack-dev-middleware')
var WebpackConfig = require('./webpack.config')

var app = express()

app.use(webpackDevMiddleware(webpack(WebpackConfig), {
  publicPath: '/build/',
  stats: {
    colors: true
  }
}))

app.use(express.static(__dirname))

app.get('/*', function (request, response){
  response.sendFile(path.resolve(__dirname, 'app', 'index.html'))
})

app.listen(8080, function () {
  console.log('Server listening on http://localhost:8080, Ctrl+C to stop')
})
```
## 依赖模块

``` json
{
  "devDependencies": {
    "babel-cli": "^6.7.5",
    "babel-core": "^6.7.6",
    "babel-eslint": "^5.0.4",
    "babel-loader": "^6.2.4",
    "babel-plugin-add-module-exports": "^0.1.2",
    "babel-plugin-dev-expression": "^0.2.1",
    "babel-polyfill": "^6.7.4",
    "babel-preset-es2015": "^6.6.0",
    "babel-preset-es2015-loose": "^7.0.0",
    "babel-preset-es2015-loose-native-modules": "^1.0.0",
    "babel-preset-react": "^6.5.0",
    "babel-preset-stage-0": "^6.5.0",
    "babel-register": "^6.7.2",
    "bootstrap": "^3.3.6",
    "bundle-loader": "^0.5.4",
    "codecov.io": "^0.1.6",
    "coveralls": "^2.11.9",
    "css-loader": "^0.23.1",
    "eslint": "^1.10.3",
    "eslint-config-rackt": "^1.1.1",
    "eslint-plugin-react": "^3.16.1",
    "expect": "^1.18.0",
    "express": "^4.13.4",
    "file-loader": "^0.8.5",
    "gzip-size": "^3.0.0",
    "history": "^2.1.0",
    "isomorphic-fetch": "^2.2.1",
    "isparta-loader": "^2.0.0",
    "pretty-bytes": "^3.0.1",
    "qs": "^6.1.0",
    "react": "^15.0.0",
    "react-addons-css-transition-group": "^15.0.0",
    "react-addons-test-utils": "^15.0.0",
    "react-bootstrap": "^0.29.3",
    "react-dom": "^15.0.0",
    "react-redux": "^4.4.5",
    "react-router": "^2.4.0",
    "react-router-bootstrap": "^0.23.0",
    "redux": "^3.5.2",
    "redux-logger": "^2.6.1",
    "redux-thunk": "^2.1.0",
    "rimraf": "^2.5.2",
    "style-loader": "^0.13.1",
    "url-loader": "^0.5.7",
    "webpack": "^1.12.14",
    "webpack-dev-middleware": "^1.6.1"
  }
}
```
到这里配置部分已经结束，接下来写一个小demo 来试试

``` js
import React from 'react';
import { render } from 'react-dom';

class App extends React.Component {
  render() {
    return (
      <div>
        <h1>Welcome to React!</h1>
      </div>
    );
  }
}

render( ( <App /> ), document.getElementById('content') );
```

示例代码地址 [Github](https://github.com/germanyt/react-study-example/tree/master/example/1)