---
title: React 学习笔记 - 3
date: 2016-05-18 09:51:58
tags: [JavaScript,React,React-router,Redux]
---

## 简介
本节主要学习React Component相关内容

## 目录
- create
- props
- state
- Event system
- lifecycle
- context

## create
有三种方式创建Component，实际开发中可根据实际情况选择

1. class App extends React.Component {}
这是ES6中class 继承的写法
2. const App = React.createClass( {} )
这是通过createClass函数来创建的写法
3. 无状态函数
``` js
const HelloMessage = (props) => <div>Hello {props.name}</div>;
ReactDOM.render(<HelloMessage name="Sebastian" />, mountNode);
```

(1)(2)两种方法类似，具体区别参考官方文档中的 [ES6 classes](https://facebook.github.io/react/docs/reusable-components.html#es6-classes)

(3)只适用于简单组件

## props
props的属性和方法来自于调用该 Component 时传递的内容。

#### children
JSX有两种方式render Component: `<App />` & `<App>Hello world</App>`，第一种 没有children属性，第二种children可以是 `字符串`或`DOM树`，为DOM树时children是一个和树对应的数组。

- 
``` html
// 需要先 import Parent Component
<div>
  <Parent />
  <code>&lt;Parent /&gt;</code>
</div>
<div>
  <Parent>Hello world!</Parent>
  <code>&lt;Parent&gt;字符串&lt;/Parent&gt;</code>
</div>

<div>
  <Parent>
    <h1>这是H1</h1>
    <h2>这是H2</h2>
  </Parent>
  <code>&lt;Parent&gt;DOM Tree&lt;/Parent&gt;</code>
</div>
```
- 
``` js
// Parent Component 
import React from 'react';

class Parent extends React.Component {
  render() {
    return (
      <div>
        <h3>Parent</h3>
        <p>This is children:</p>
        {this.props.children}
      </div>
    );
  }
}

export default Parent;
```

#### 手动传递
React中 this.porps的属性除children外都需要手动传递，手动传递有两种方式：
1. “不定参数“形式
``` js
import React from 'react';

import Receive from './receive.component';

class Transfer extends React.Component {
  render() {
    let props = {
      a: 1,
      b: 'show Text',
      c: ['tv', 'ico', 'co']
    }
    return (
      <div>
        <h2>Transfer</h2>
        <Receive {...props} />
      </div>
    )
  }
}

export default Transfer;
```
2. 属性方式
``` js
import React from 'react';

import Receive from './receive.component';

class Transfer extends React.Component {
  render() {
    return (
      <div>
        <h2>Transfer</h2>
        <Receive x="2002" y={['fb', 'tw', 'gl']} z={{text: 'hello', name: 'Deo'}} />
      </div>
    )
  }
}

export default Transfer;
```

*Tips: v15已不再建议通过`setProps` `replaceProps`修改props。*

## state
state用于维护当前组件的状态

初始化state, 需要用到`getInitialState`方法，但ES6 classes需要写在`constructor`中，示例如下：
1. createClass
``` js
import React from 'react';

const StateEs5 = React.createClass({
	getInitialState() {
		return {count: 0};
	},

	tick() {
		this.setState({count: this.state.count + 1});
	},

	render() {
		return (
      <div onClick={this.tick}>
        Clicks: {this.state.count}
      </div>
    );
	}
});

export default StateEs5;
```

2. ES6 classes
``` js
import React from 'react';

class State extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 0};
    // this.tick = this.tick.bind(this);
  }

  tick() {
    this.setState({count: this.state.count + 1});
  }

  render() {
    return (
      <div onClick={() => this.tick()}>
        Clicks: {this.state.count}
      </div>
    );
  }

}

export default State;
```

除了初始化state的区别外，ES6 classes 不支持自动绑定，需要在绑定时使用箭头函数 `()=>this.tick()` 或者在constructor中添加 `this.tick = this.tick.bind(this)`，两种方法任选其一。

## 事件绑定

- 绑定示例
``` html
<div onClick={this.handleClick}>
  Clicks: {this.state.count}
</div>
<input type="text" onChange={this.handleChange} />
```

支持的事件列表参看 [官方文档](https://facebook.github.io/react/docs/events.html)


## 生命周期
方法分为三类：
1. 初始化调用且只调用一次
	- componentWillMount
	- componentDidMount
2. 更新props或state时调用（初始化时不调用）
	- componentWillReceiveProps
	- shouldComponentUpdate (执行`forceUpdate`时该方法不执行)
	- componentWillUpdate
	- componentDidUpdate
3. 卸载组件时调用
	- componentWillUnmount

附：[官方文档](https://facebook.github.io/react/docs/component-specs.html#lifecycle-methods)

## Context

Context是一个高级实验功能，未来可能会有变化，请谨慎使用。

* 示例代码
``` js
import React from 'react';

const Button = React.createClass({
	contextTypes: {
		btnClass: React.PropTypes.string
	},

	render() {
		return (
      <button className={'btn ' + this.context.btnClass}>
        {this.props.children}
      </button>
    );
	}
});

const Message = React.createClass({
  render() {
    return (
      <div>
        {this.props.text} <Button>Delete</Button>
      </div>
    );
  }
});

const MessageList = React.createClass({
  childContextTypes: {
    btnClass: React.PropTypes.string
  },
  getChildContext() {
    return {btnClass: "btn-default"};
  },
  render() {
    let children = ['Context 示例', '在祖先组件定义并设置值', '在子组件取得相应值'].map(function(message, index) {
      return <Message text={message} key={index} />;
    });
    return <div>{children}</div>;
  }
});

export default MessageList;
```

## 结语
`props` `state` `context` 为Component提供了需要的数据
- state: 维护Component内部的状态
- props: 属性或方法由父组件传递到Component
- context: 方法或属性可继承祖先组件要传给后代的数据

## 参考链接
更多API参考 [React Document](https://facebook.github.io/react/docs/getting-started.html)

示例代码地址 [Github](https://github.com/germanyt/react-study-example/tree/master/example/3)

