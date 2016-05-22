---
title: React 学习笔记 - 2
date: 2016-05-17 10:39:14
tags: [JavaScript,React,React-router,Redux]
---

## 简介
本节中包含`React-router`控制React SPA路由。

关于`React-router`的详细使用，请查阅官方文档。

## 示例代码
``` js
import React from 'react';
import { render } from 'react-dom';

import {Router, Route, IndexRoute, Link} from 'react-router';

class App extends React.Component {
  render() {
    return (
      <div>
        <h1>Welcome to React!</h1>
        <ul>
          <li><Link to="/">Index</Link></li>
          <li><Link to="/about">About</Link></li>
        </ul>

        {this.props.children}
      </div>
    );
  }
}

class Index extends React.Component {
  render() {
    return (
    	<div>
        <h2>Index</h2>
    	</div>
    );
  }
}

class About extends React.Component {
  render() {
    return (
    	<div>
        <h2>About</h2>
    	</div>
    );
  }
}

render( (
	<Router>
	  <Route path="/" component={App}>
        <IndexRoute component={Index} />
        <Route path="about" component={About} />
	  </Route>
	</Router> 
), document.getElementById('content') );
```

运行以上代码能正常运行，URL地址为`http://hostname/#/pageRoute?_k=xxxx`，并在Console中看到以下警告
```
Warning: [react-router] `Router` no longer defaults the history prop to hash history. Please use the `hashHistory` singleton instead. http://tiny.cc/router-defaulthistory
```
根据以上提示信息，给Router Component添加history属性 `<Router history={hashHistory}>`，*使用hashHistory前需要先在react-router模块中引入*。

#### URL格式
1. 想要使用`http://hostname/pageRoute`格式的URL地址，只需用 `browserHistory` 替换掉hashHistory即可。
2. 想要使用`http://hostname/projectName/pageRoute`格式的URL地址，需要引入 `useRouterHistory` 和 `history`模块的 `createHistory`，然后添加如下代码
``` js
const history = useRouterHistory(createHistory)({
  basename: '/projectName'
})
```
## 多级路由
本节需到的API：
- IndexRoute from react-router

多级路由需要将次级对应的`<Route />`节点作为父级的字节点
``` js
  <Router>
    <Route path="/" component={App}>
        <IndexRoute component={Index} />
        <Route path="about" component={About}>
          <Route path="author" component={Author} />
        </Route>
    </Route>
  </Router> 
```

并在父级`component`中输出`{this.props.children}`
``` js
class About extends React.Component {
  render() {
    return (
      <div>
        <h2>About</h2>
        {this.props.children}
      </div>
    );
  }
}
```
`About Component`的文字出现在了`/about/author`中，怎么才能让`/about/author`不显示`About Component`的内容呢？
这时候就要用到`react-router`的另外一个Component `IndexRoute`，使用方法和 `Route` 相同，修改后的代码如下
``` js
<Route path="about" component={About}>
  <IndexRoute component={AboutIndex} />
  <Route path="author" component={Author} />
</Route>
```
``` js
class About extends React.Component {
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    );
  }
}

class AboutIndex extends React.Component {
  render() {
    return (
      <div>
        <h2>About</h2>
      </div>
    );
  }
}

class Author extends React.Component {
  render() {
    return (
      <div>
        <h2>Author: Gavin</h2>
      </div>
    );
  }
}
```

## 可变路由

当需要路由`/user/gavin` `/user/david`等这类有规律且对应的Component是同一个的时候，可设置路由规则为`/user/:username`
示例如下
``` js
<Route path="user" component={User}>
  <IndexRoute component={UserIndex} />
  <Route path=":username" component={UserCenter} />
</Route>
```
``` js
class User extends React.Component {
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    )
  }
}

class UserIndex extends React.Component {
  render() {
    return (
      <div>
        <h2>User Index</h2>
      </div>
    )
  }
}

class UserCenter extends React.Component {
  render() {
    return (
      <div>
        <h2>User Name: {this.props.params.username}</h2>
      </div>
    )
  }
}
```

可以看出`:username`对应的值在Component中通过`this.props.params.username`获取。

## Link
应用内部链接需要使用`<Link />`代替`<a href="">Link</a>`，具体使用方法参考官方文档。

## 生命周期
在1.x中提供了Mixin Lifecycle的方法，可以添加方法`routerWillLeave`，当离开当前Component触发，2.x版本已不再建议。
在2.x中相同需求的实现方法如下
``` js
const RouteComponent = React.createClass({
  contextTypes: {
    router: React.PropTypes.object.isRequired
  },
  componentDidMount() {
    const { route } = this.props
    const { router } = this.context
    router.setRouteLeaveHook(route, this.routerWillLeave)
  }
});
```

*Tips: 这里用 `React.createClass`, 而不是继承React.Component将在下一节中学习。*


## 其他

React Router 还提供了一些有用的Component/Function：
- `<Redirect/>` 路由跳转以便兼容旧路由
- `RouterContext` 通过`context.router`访问router的方法（具体的实现原理将在下一节中学习）
- `withRouter` 通过`this.props.router`访问router的方法

*Tips: router 类似于1.x的history，2.x不再推荐history和Mixins，后续版本可能会移除*

## 参考链接
相关API参考 [React Router Document](https://github.com/reactjs/react-router/blob/master/docs/API.md)

示例代码地址 [Github](https://github.com/germanyt/react-study-example/tree/master/example/2)