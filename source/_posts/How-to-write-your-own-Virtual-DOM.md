---
title: 如何自己实现虚拟DOM
date: 2016-06-02 14:21:30
tags: [Translate,JavaScript]
---

原文地址：[medium.com @deathmood](https://medium.com/@deathmood/how-to-write-your-own-virtual-dom-ee74acc13060#.8eontjd52)

自己实现虚拟DOM需要知道两件事情。你不需要深入研究 React 源码或者其它的实现源码，他们非常庞大和复杂，但是真正实现虚拟DOM部分的代码不到50行。50行代码！！！

这里有两个概念：
- 虚拟DOM是任何一种真实DOM的表现形式
- 改变虚拟DOM树中的东西，我们将获得一个新的虚拟DOM树。通过算法来比较这两个树（新的和旧的虚拟树），找出不同点。然后对应虚拟树在真实DOM树中做最小的必要改变。

就是这样！让我们一起来深入理解每个概念。

## 表示 DOM 树
首先需要以某种方式将DOM树存储在内存中。我们可以用原生 Object 来做这件事。假设我们有一个这样的树：
``` html
<ul class="list">
  <li>item 1</li>
  <li>item 2</li>
</ul>
```
看起来很简单，不是吗？我们怎样只用JS对象来表示它呢？
``` js
{ type: 'ul', props: { 'class': 'list' }, children: [
  { type: 'li', props: {}, children: ['item 1'] },
  { type: 'li', props: {}, children: ['item 2'] }
] }
```
这里我们需要注意两点：
- 我们用对象表示DOM元素
``` js
{ type: '…', props: { … }, children: [ … ] }
```
- 我们用原生JS字符串表示DOM文本节点

但是用这样的方法写一个大树是很困难的。所以我们写一个辅助函数，她能让我们更容易理解结构：
``` js
function h(type, props, …children) {
  return { type, props, children };
}
```

现在，我们可以像这样来写DOM树：
``` js
h('ul', { 'class': 'list' },
  h('li', {}, 'item 1'),
  h('li', {}, 'item 2'),
);
```

它看起来简洁多了，不是吗？但是我们可以更进一步。你听说过 JSX 吗？是的，在这里我想用它。它是怎么工作的？

如果你读过[这里](https://babeljs.io/docs/plugins/transform-react-jsx/)的官方 Babel JSX 文档，你将会知道，Babel 编译这段代码：
``` html
<ul className="list">
  <li>item 1</li>
  <li>item 2</li>
</ul>
```
像这样：
``` js
React.createElement('ul', { className: 'list' },
  React.createElement('li', {}, 'item 1'),
  React.createElement('li', {}, 'item 2'),
);
```

注意到有一些相似？是的是的。如果我们能将 `React.createElement(…)` 替换成我们的 `h(…)`。事实上我们可以这样做——通过用叫做 `jsx pragma` 的东西。我们只需要在文件顶部写一行类注释的内容：
``` jsx
/** @jsx h */
<ul className="list">
  <li>item 1</li>
  <li>item 2</li>
</ul>
```

它告诉 Babel “嘿，编译 JSX 但是要用 `h` 代替 `React.createElement`。” 在这你可以用任意一个函数代替 `h` 进行编译。

所以，像我前面说的那样，我们将用这种方法写DOM：
``` js
/** @jsx h */
const a = (
  <ul className="list">
    <li>item 1</li>
    <li>item 2</li>
  </ul>
);
```
它将会被 Babel 编译成这样：
``` js
const a = (
  h('ul', { className: 'list' },
    h('li', {}, 'item 1'),
    h('li', {}, 'item 2'),
  );
);
```

当函数 `h` 执行的时候，它会返回一个原生JS对象——我们虚拟DOM表示：
``` js
const a = (
  { type: 'ul', props: { className: 'list' }, children: [
    { type: 'li', props: {}, children: ['item 1'] },
    { type: 'li', props: {}, children: ['item 2'] }
  ] }
);
```

开始吧！在 JSFiddle 中尝试（不要忘记设置语言为 Babel ）：
{% jsfiddle 5qyLubt4 'js,resources,html,css,result' %}

## 应用我们的 DOM 表示法
好了，现在我们已经用自己的结构的原生对象表示了我们的 DOM 树。这非常酷，但是我们需要用它以一种方式创建真实的DOM树，我们不能只将我们的表示法附加到 DOM 中。

首先让我们做一些假设和定义一些术语：
- 所有的真实 DOM 节点（元素和文本节点）变量将会用 `$` 开头——因此 `$parent` 将会是一个真实的 DOM 元素
- 虚拟节点表示法将指定变量名为 `node`
- 像在 React 中，你只能有一个根节点——所有的其他节点都在里面

好了，像这样说的，让我们写一个能将虚拟 DOM 节点转化成真实 DOM 节点的函数 `createElement(…)` 。现在先忽略 `props` 和 `children` ——我们以后将创建它：
``` js
function createElement(node) {
  if (typeof node === 'string') {
    return document.createTextNode(node);
  }
  return document.createElement(node.type);
}
```

因为我们有文本节点（原生字符串）和 Elements(含有 type 的JS对象）像这样：
``` js
{ type: '…', props: { … }, children: [ … ] }
```
因此，我们能传递虚拟的文本节点和虚拟的 element 节点，它都能工作。
现在，让我们一起来思考子节点——他们每一个不是文本节点就是 Element 节点。所以它们也能被 `createElement(…)` 函数创建。耶，你感觉到了吗？它感觉像是递归。所以我们能为每个元素子节点调用 `createElement(…)` ，然后像这样 `appendChild()` 它们到我们的元素中：
``` js
function createElement(node) {
  if (typeof node === ‘string’) {
    return document.createTextNode(node);
  }
  const $el = document.createElement(node.type);
  node.children
    .map(createElement)
    .forEach($el.appendChild.bind($el));
  return $el;
}
```

哇哦，看起来很不错。现在我们