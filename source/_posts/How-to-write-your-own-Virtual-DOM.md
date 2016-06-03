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

哇哦，看起来很不错。现在我们先把 `props` 放在一边，以后再讨论它。我们不需要它来理解虚拟DOM的基础概念，但是它却会增加复杂度。

现在开始让我们在 JSFiddle 中尝试它：
{% jsfiddle cL0Lc7au 'js,resources,html,css,result' %}

## 变化处理

我们已经能把虚拟DOM转换成真实DOM，现在思考虚拟DOM树之间的差别。我们需要写一个来对比两个虚拟DOM树（新和旧）并在真实DOM树中只做必要改变的算法。

怎么对比树？我们需要下面几种情况：
- 对应位置没有旧的节点——需要 `appendChild(…)` 添加节点
![对应位置没有旧的节点](https://cdn-images-1.medium.com/max/800/1*GFUWrX6pBgiDQ5Z-IvzjUw.png)
- 对应位置没有新的节点——需要 `removeChild(…)` 删除节点
![对应位置没有新的节点](https://cdn-images-1.medium.com/max/800/1*VRoYwAeWPF0jbiWXsKb2HA.png)
- 对应位置有不同的节点——需要 `replaceChild(…)` 替换节点
![对应位置没有新的节点](https://cdn-images-1.medium.com/max/800/1*6iQYEH0APjbuPvYmnD7Qlw.png)
- 节点相同——需要进一步对比子节点
![节点相同](https://cdn-images-1.medium.com/max/800/1*x1Eq-uuqgL0z9d9qn_opww.png)

好了，我们写一个 `updateElement(…) ` 函数，传递三个参数（$parent, newNode and oldNode）给它，`$parent` 是虚拟节点对应的真实节点的父节点。现在我们来看怎么处理上面描述的几种情况。

## 没有旧节点
这是非常简单的：
``` js
function updateElement($parent, newNode, oldNode) {
  if (!oldNode) {
    $parent.appendChild(
      createElement(newNode)
    );
  }
}
```

## 没有新节点
这里有个问题——如果新虚拟树的当前位置没有节点，我们应该在真实DOM中移除它，但是我们应该怎么做呢？我们知道父元素（已被传递到函数中），因此我们假设调用 `$parent.removeChild(…)` 并传递真实DOM的引用给它。但是我们没有真实DOM的引用。如果我们知道节点在父元素中的位置，我们能够通过 `$parent.childNodes[index]` 得到它的引用， `index` 就是节点在父元素中的索引。

我们假设 `index` 会传递给函数（后面你会看到它真的会被传递）。因此我们的代码会变成这样：
``` js
function updateElement($parent, newNode, oldNode, index = 0) {
  if (!oldNode) {
    $parent.appendChild(
      createElement(newNode)
    );
  } else if (!newNode) {
    $parent.removeChild(
      $parent.childNodes[index]
    );
  }
}
```

## 节点变化
首先我们需要一个对比两个节点的函数来告诉我们节点是否改变。我们应该考虑 elements 和文本节点：
``` js
function changed(node1, node2) {
  return typeof node1 !== typeof node2 ||
         typeof node1 === ‘string’ && node1 !== node2 ||
         node1.type !== node2.type
}
```

现在我们已经有了当前节点在父元素的索引 (`index`) ，我们能很容易用新创建的节点替换它：
``` js
function updateElement($parent, newNode, oldNode, index = 0) {
  if (!oldNode) {
    $parent.appendChild(
      createElement(newNode)
    );
  } else if (!newNode) {
    $parent.removeChild(
      $parent.childNodes[index]
    );
  } else if (changed(newNode, oldNode)) {
    $parent.replaceChild(
      createElement(newNode),
      $parent.childNodes[index]
    );
  }
}
```

## 比较子节点
最后，但是并非不重要的。我们应该遍历每个节点的子节点并对比它们，通过调用 `updateElement(…)` 。是的，又是递归。
但是有些写代码之前需要考虑的事情：
- 我们应该比较只有节点是 Element 的子节点（文本节点没有子节点）
- 传递当前节点的引用作为父节点
- 必须一个一个的比较子节点，即使有些地方我们只有 “undefined” 这都是没问题的，我们的函数可以处理
- 最后，`index`——它只是子节点在子节点数组中的索引

``` js
function updateElement($parent, newNode, oldNode, index = 0) {
  if (!oldNode) {
    $parent.appendChild(
      createElement(newNode)
    );
  } else if (!newNode) {
    $parent.removeChild(
      $parent.childNodes[index]
    );
  } else if (changed(newNode, oldNode)) {
    $parent.replaceChild(
      createElement(newNode),
      $parent.childNodes[index]
    );
  } else if (newNode.type) {
    const newLength = newNode.children.length;
    const oldLength = oldNode.children.length;
    for (let i = 0; i < newLength || i < oldLength; i++) {
      updateElement(
        $parent.childNodes[index],
        newNode.children[i],
        oldNode.children[i],
        i
      );
    }
  }
}
```

## 所有的代码放在一起
我们把所有的代码放在 JSFiddle， 就像我承诺的一样它的执行部分只有50行代码。我们来看看：
{% jsfiddle 0htedLra 'js,resources,html,css,result' %}
打开开发者工具，观察点击 “Reload” 按钮时变化的应用。
![](https://cdn-images-1.medium.com/max/800/1*e1s_Zc_fVxL3i0un2ZNEtg.gif)

## 结语
恭喜！我们已经做到了。我们写一个虚拟DOM的执行程序，它工作的很好。我希望通过阅读这篇文章，你能够理解虚拟DOM和React底层的运行机制。

然而有些事情我们并没有在这里介绍。（在后面的文章中我会尝试着覆盖到它们）
- 设置／对比／更新 Element 属性（props）
- 处理事件——在 Elements 上添加事件监听
- 像React一样在组件中使用虚拟DOM
- 真正的 DOM 节点引用
- 在直接改变真实 DOM 的库中使用虚拟DOM——像 jQuery 和它的插件一样
- 以及更多的内容...