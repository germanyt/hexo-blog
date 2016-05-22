---
title: React 学习笔记 - 4
date: 2016-05-20 15:28:33
tags: [JavaScript,React,React-router,Redux]
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
const ADD_MESSAGE = 'ADD_MESSAGE';

{
  type: ADD_MESSAGE,
  data
  // 其它数据
}
```

这里约定，action 内必须使用一个字符串类型的 type 字段来表示将要执行的动作。建议在单独的模块或文件中使用常量定义，以方便维护。

#### Action 创建函数
Action 创建函数，是一个返回action的简单函数，这样做方便移植和测试。
- 示例
``` js
function addMessage(data) {
  return {
    type: ADD_MESSAGE,
    data
  };
}
```
Redux 中只需把 action 创建函数的结果传给 `dispatch()` 方法即可发起一次 dispatch 过程。

## Reducer
Action 描述了事情已经发生，Reducer来执行具体如何更新state。

1. 先设计state的结构，以一个消息列表为例：
	``` js
	{
	  list: [
	    {
	      text: 'Consider using Redux',
	      create_time: '2016-05-21 12:22:56'
	    },
	    {
	      text: 'Keep all state in a single tree',
	      create_time: '2016-05-21 12:22:56'
	    }
	  ]
	}
	```
2. Action 处理
	reducer 就是一个纯函数，接收旧的 state 和 action，返回新的 state。
	``` js
	import {ADD_MESSAGE} from '../actions/constants';

  const initState = {
    list: []
  };

  export function Message(state = initState, action) {
    switch(action.type){
      case ADD_MESSAGE:
        // return Object.assign({}, state, {
        //   list: [...state.list, action.data]
        // });

        let list = [...state.list, action.data];
        return { ...state, list };
      default:
        return state;
    }
	}
	```
	action参数的值是 Action对象，在这里不要在state上直接进行修改，可以使用`Object.assign({}, state)` 或 其他库的 `_.assign()` 对state新建一个副本，第一个参数必须设置为空对象。还可以使用了ES7提案中的对象展开运算符，需要使用转换编译器，如Babel。Babel中使用 [babel-plugin-transform-object-rest-spread](http://babeljs.io/docs/plugins/transform-object-rest-spread/) 插件。

	*注意：永远不要在reducer中做以下操作：*
	- 修改传入参数；
	- 执行有副作用的操作，如 API 请求和路由跳转；
	- 调用非纯函数，如 Date.now() 或 Math.random()。

3. 多个Action
	要处理多个Action的时候，只需要在switch中添加一个case
	``` js
	export function Message(state = initState, action) {
	  switch(action.type){
      case ADD_MESSAGE:
        let list = [...state.list, action.data];
        return { ...state, list };
      case DELETE_MESSAGE:
        let index = action.index;
        if(index >= 0 && index < state.list.length){
          let list = [...state.list.slice(0, index), ...state.list.slice(index+1)];
          return {...state, list};
        }

        return state;
      default:
        return state;
    }
	}
	```



