---
title: React 学习笔记 - 4
date: 2016-05-20 15:28:33
tags:
---


## 前言
前边几节中学习了组件和路由控制，已经能够做一个静态的SPA了。但是对于一个富应用来说，我们只学习了View，接下来我们将要学习数据处理模块，在正式开始学习之前，先来介绍下这位新朋友。

## Redux
Redux 是 JavaScript 状态容器，提供可预测化的状态管理。可以让你构建一致化的应用，运行于不同的环境（客户端、服务器、原生应用），并且易于测试。Redux 由 Flux 演变而来，但受 Elm 的启发，避开了 Flux 的复杂性。
Redux 规定，将模型的更新逻辑全部集中于 `reducer` 层，不允许程序直接修改数据，而是用一个叫作 `action` 的普通对象来对更改进行描述。

## 三个基本原则
1. 单一数据源
	整个应用只存在一个 Object tree，并且 Object tree 只存在于唯一的 store 中。
2. State 是只读的
	唯一改变 state 的方法就是触发 action
3. 使用纯函数来执行修改
	为了描述 action 如何改变 state tree ，你需要编写 reducers。

## Action
`action`是把数据从应用传到 Store 的载体，是 Store 数据的唯一来源。

- 示例Action：
``` js
const ACTION_TEST = 'ACTION_TEST';

{
  type: ACTION_TEST,
  data
  // 其它数据
}
```

这里约定，action 内必须使用一个字符串类型的 type 字段来表示将要执行的动作。建议在单独的模块或文件中使用常量定义，以方便维护。

#### Action 创建函数
Action 创建函数，是一个返回action的简单函数，这样做方便移植和测试。
- 示例
``` js
function actionTest(data) {
  return {
    type: ACTION_TEST,
    data
  };
}
```
Redux 中只需把 action 创建函数的结果传给 `dispatch()` 方法即可发起一次 dispatch 过程。