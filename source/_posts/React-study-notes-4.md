---
title: React 学习笔记 - 4
date: 2016-05-20 15:28:33
tags: [JavaScript,React,Redux]
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
	要处理多个Action的时候，在switch中为每一个Action添加一个case。
	``` js
	export function message(state = initState, action) {
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

4. 多个reducer
  多个Action时，需要合理的设计state结构，例如如下结构state，可以拆分filter和message为两个reducer。建议为每个reducer对应一个文件，应用庞大时可使用文件夹分类。
  ``` js
  {
    filter: {
      key: 'Redux'
    },
    message: [
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
  现在我们在reducers文件夹下建两个reducer文件，分别命名为filter.reducer.js和message.reducer.js。
  ``` js
  // filter.reducer.js
  import { INPUT_FILTER_KEY } from '../actions/constants';

  const initState = {
    key: ''
  };

  export default function filter(state = initState, action) {
    switch(action.type) {
      case INPUT_FILTER_KEY:
        let key = action.key || '';
        return {...state, key};
      default:
        return state;
    }
  }
  ```
  ``` js
  // message.reducer.js
  import {ADD_MESSAGE, DELETE_MESSAGE} from '../actions/constants';

  const initState = []

  export default function message(state = initState, action) {
    switch(action.type){
      case ADD_MESSAGE:
        return [...state, action.data];
      case DELETE_MESSAGE:
        let index = action.index;
        if(index >= 0 && index < state.length){
          return [...state.slice(0, index), ...state.slice(index+1)];
        }

        return state;
      default:
        return state;
    }
  }
  ```
  由于应用中只能存在单一的state树，所以我们需要将上面两个reducer合并成一个，Redux 提供了 `combineReducers()` 工具类来合并多个reducer。
  现在我们在reducers文件夹下，新建一个 `index.js` 文件使用 `combineReducers()` 合并两个reducer：
  ``` js
  import { combineReducers } from 'redux';

  import message from './message.reducer';
  import filter from './filter.reducer';

  const messageApp = combineReducers( {
    message,
    filter
  } );

  export default messageApp;
  ```
  注意上面的写法和下面完全等价：
  ``` js
  import message from './message.reducer';
  import filter from './filter.reducer';

  export default function messageApp(state = {}, action){
    return {
      message: message(state.message, action),
      filter: filter(state.filter, action)
    }
  };
  ```


## Store

Store 就是把Action 和 reducer 联系到一起的对象。Store 有以下职责：
- 维持应用的 state；
- 提供 `getState()` 方法获取 state；
- 提供 `dispatch(action)` 方法更新 state；
- 通过 `subscribe(listener)` 注册监听器;
- 通过 `subscribe(listener)` 返回的函数注销监听器。

再次强调一下 *Redux 应用只有一个单一的 store*。当需要拆分数据处理逻辑时，你应该使用 reducer 组合 而不是创建多个 store。
根据已有的reducer创建store非常容易。现在我们导入前面合并后的reducer，并传递给 `ctreateStore()`。

- 创建一个store文件
``` js
import { createStore } from 'redux';

import messageApp from '../reducers';

let store = createStore(messageApp);

export default store;
```

## 发起Actions
现在我们已经建好了store，在没有页面的情况下就已经可以开始测试数据了。
- 测试store
``` js
import store from './store';
import {addMessage, deleteMessage, inputFilterKey} from './actions';

console.log(store.getState());

// 每次 state 更新时，打印日志
// 注意 subscribe() 返回一个函数用来注销监听器
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

// 发起一系列 action
store.dispatch(addMessage({text: 'Learn about actions', create_time: '2016-05-24 11:09:24'}));
store.dispatch(addMessage({text: 'Learn about reducers', create_time: '2016-05-24 11:09:24'}));
store.dispatch(addMessage({text: 'Learn about store', create_time: '2016-05-24 11:09:24'}));
store.dispatch(deleteMessage(2));
store.dispatch(deleteMessage(0));
store.dispatch(inputFilterKey('Learn'));

// 停止监听 state 更新
unsubscribe();
```

## 连接Redux
在我们已经写好页面的情况下，如何将页面组件与Store连接？
我们用 `react-redux` 提供的 `connect()` 方法，将Message Component 转化成容器组件。
- containers/message.container.js
``` js
import { connect } from 'react-redux';

import { Message } from '../components/';
import { addMessage, deleteMessage, inpuFilterKey } from '../actions';


// 哪些 action 创建函数是我们想要通过 props 获取的？
function mapDispatchToProps(dispatch) {
  return {
    onAddMessage: (data) => dispatch(addMessage(data)),
    onDeleteMessage: (index) => dispatch(deleteMessage(index)),
    onChangeFilterKey: (key) => dispatch(inputFilterKey(key))
  };
}

function filterMessage(messages, filter_key) {
  if(filter_key){
    var messageList = [];
    messages.map( (message, index) => {

      if( ~message.text.indexOf(filter_key) ){
        messageList.push({
          id: index,
          text: message.text,
          create_time: message.create_time
        });
      };
    });

    return messageList;
  } else {
    return messages;
  }
}
// 哪些 Redux 全局的 state 是我们组件想要通过 props 获取的？
// 这里我们还使用了 filterMessage 来根据filter_key 筛选出相匹配的 message
function mapStateToProps(state){
  return {
    messages: filterMessage(state.message, state.filter.key),
    filter_key: state.filter.key
  }
}

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Message);
```
到此，我们已经可以使用Redux来管理React数据了，但是当我们需要和服务器进行交互或者是需要进行非纯函数操作时，应该怎么做呢？
## 异步Action
在使用异步Action时，一般情况下每个请求都要dispatch三个不同的Action，为了区分不同的Action，我们建议为他们定义不同的type。
上面 addMessage 的create_time值都是静态数据，现在我们想动态的获取create time。
- 
``` js
{ type: 'GET_TIME_REQUEST' }
{ type: 'GET_TIME_FAILURE', error: 'error info' }
{ type: 'GET_TIME_SUCCESS', data: { ... } }
```
1. 创建Action
``` js
export function addMessage(data) {
  // return {
  //   type: ADD_MESSAGE,
  //   data
  // };

  return dispatch => {
    dispatch( getTimeRequest() );
    getTime().then(time => {
      dispatch( getTimeSuccess() );

      data.create_time = time;

      dispatch( addMessageToStore(data) );

    }).catch(error => dispatch( getTimeFailure(error && error.message || error) ));
  }

}

function addMessageToStore(data) {
  return {
    type: ADD_MESSAGE,
    data
  };
}

function getTimeRequest() {
  return {
    type: GET_TIME_REQUEST
  }
}

function getTimeFailure(error) {
  return {
    type: GET_TIME_FAILURE,
    error
  }
}

function getTimeSuccess(data) {
  return {
    type: GET_TIME_SUCCESS,
    data
  }
}

function getTime(){
  return new Promise(function(resolve, reject){
    let time = moment().format('YYYY-MM-DD HH:mm:ss');

    resolve(time);
  });
}
```
2. 创建reducer
``` js
import { GET_TIME_REQUEST, GET_TIME_FAILURE, GET_TIME_SUCCESS, GET_TIME_ERROR_CLEAR } from '../actions/constants';

const initState = {
  loading: false,
  errorInfo: ''
};

export default function getTime(state = initState, action) {
  let loading = false;
  let errorInfo = '';
  switch(action.type) {
    case GET_TIME_REQUEST:
      loading = true;
      return {...state, loading};
    case GET_TIME_FAILURE:
    case GET_TIME_SUCCESS:
      loading = false;
      
      if(action.error){
        errorInfo = JSON.stringify(action.error);
      } else {
        errorInfo = '';
      }
      return {...state, loading, errorInfo};
    case GET_TIME_ERROR_CLEAR:
      errorInfo = '';
      return {...state, errorInfo};
    default:
      return state;
  }
}
```
3. 添加Middleware
要在 Action 中使用 dispatch() 函数，需要中间件支持，我们选择使用 redux-thunk 。
``` js
import thunk from 'redux-thunk';
let store = createStore(reducers, applyMiddleware(thunk));
```

redux-thunk 中间件不是唯一的方法。也可以使用 redux-promise 或者 redux-promise-middleware 来 dispatch Promise 替代函数。当然也可以写一个自定义的middleware。

##### 现在已经可以使用redux通过同步和异步Action管理数据了。

## 参考链接
更多API参考 [Redux Document](http://redux.js.org/docs/introduction/) [中文](http://cn.redux.js.org/index.html)

示例代码地址 [Github](https://github.com/germanyt/react-study-example/tree/master/example/4)

## 推荐阅读

[從 source code 來看 Redux 更新 state 的運行機制](https://medium.com/@as790726/%E5%BE%9E-source-code-%E4%BE%86%E7%9C%8B-redux-%E7%9A%84%E9%81%8B%E8%A1%8C%E6%A9%9F%E5%88%B6-f5e0adc1b9f6#.o4l5v2hry)
[從 source code 來看 React-Redux 怎麼讓 Redux 跟 React 共舞](https://medium.com/@as790726/%E5%BE%9E-source-code-%E4%BE%86%E7%9C%8B-react-redux-%E6%80%8E%E9%BA%BC%E8%AE%93-redux-%E8%B7%9F-react-%E5%85%B1%E8%88%9E-a0777b99463a#.hslbm6a96)
*需要搬梯子*
