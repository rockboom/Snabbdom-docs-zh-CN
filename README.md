# Snabbdom

Snabbdom是一个虚拟DOM库，专注于简单性，模块化，具有强大的功能和性能。


[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](https://opensource.org/licenses/MIT) [![npm version](https://badge.fury.io/js/snabbdom.svg)](https://badge.fury.io/js/snabbdom) [![npm downloads](https://img.shields.io/npm/dm/snabbdom.svg)](https://www.npmjs.com/package/snabbdom)

[![Join the chat at https://gitter.im/paldepind/snabbdom](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/paldepind/snabbdom?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## 目录

* [简介](#简介)
* [功能](#功能)
* [内联示例](#内联示例)
* [示例](#示例)
* [核心文档](#核心文档)
* [模块文档](#模块文档)
* [辅助函数](#辅助函数)
* [虚拟节点](#虚拟节点)
* [构建应用](#构建应用)

## 背景

虚拟DOM非常有用。它允许我们展示我们的应用程序的视图作为它状态的一个功能。但现有的解决办法太过臃肿，太缓慢，功能不足，而偏向于OOP and/or 的API又缺乏我需要的功能。

## 简介

Snabbdom由一个非常简单，高性能，可扩展的核心组成，核心的大小 ≈ 200 SLOC。它提供了具有丰富功能的模块化架构，可通过自定义模块进行扩展。为了维持核心的简洁，所有非必须的功能都被委派给模块。

你可以把Snabbdom构建成你想要的任何形式。挑选并定义你想要的功能。或者你也可以使用默认的扩展，得到一个高性能，体积小的虚拟DOM。虚拟DOM的所有特点都列在下面。

## 特点

* 核心部分的特点
  * 大约200SLOC - 你可以轻易地阅读所有核心部分，并充分理解他是如何工作的。
  * 可通过模块扩展
  * 有一组丰富的钩子函数，每一个vnode和全局的模块都可以使用，可以在diff和patch进程的任意部分进行挂载。
  * 高性能。在[Virtual DOM Benchmark](http://vdom-benchmark.github.io/vdom-benchmark/)中，Snabbdom是最快的虚拟DOM库之一
  * Patch函数的签名功能等效于reduce/scan函数。允许更轻易地和FRP库集成。
  * `h`函数就可以轻松创建虚拟DOM节点
  * [SVG 只有和`h`辅助器配合 _才能工作_](#svg)
  * 可以执行复杂的CSS动画。
  * 强大的事件监听功能
  * [Thunks](#thunks)优化diff和patch进程，甚至更快
* 第三方库的特点
  * 由于[snabbdom-pragma](https://github.com/Swizz/snabbdom-pragma)，支持JSX。
  * 由[snabbdom-to-html](https://github.com/acstll/snabbdom-to-html)提供服务端的HTML输出。
  * 使用[snabbdom-helpers](https://github.com/krainboltgreene/snabbdom-helpers)创建简洁的虚拟DOM。
  * 使用[snabby](https://github.com/jamen/snabby)支持模板字符串。
  * 使用[snabbdom-looks-like](https://github.com/jvanbruegge/snabbdom-looks-like)声明虚拟DOM。

## 行内示例
```javascript
var snabbdom = require('snabbdom');
var patch = snabbdom.init([ // 选择模块来初始化patch函数
  require('snabbdom/modules/class').default, // 方便切换类
  require('snabbdom/modules/props').default, // 设置DOM元素的属性
  require('snabbdom/modules/style').default, // 处理支持动画元素的样式
  require('snabbdom/modules/eventlisteners').default, // 添加监听事件
]);
var h = require('snabbdom/h').default; // 用来创建虚拟节点的辅助函数

var container = document.getElementById('container');

var vnode = h('div#container.two.classes', {on: {click: someFn}}, [
  h('span', {style: {fontWeight: 'bold'}}, 'This is bold'),
  ' and this is just normal text',
  h('a', {props: {href: '/foo'}}, 'I\'ll take you places!')
]);
// 修补空的DOM元素-这会把DOM修改为副作用
patch(container, vnode);

var newVnode = h('div#container.two.classes', {on: {click: anotherEventHandler}}, [
  h('span', {style: {fontWeight: 'normal', fontStyle: 'italic'}}, 'This is now italic type'),
  ' and this is still just normal text',
  h('a', {props: {href: '/bar'}}, 'I\'ll take you places!')
]);
// 第二个 `patch` 调用
patch(vnode, newVnode); // Snabbdom 会高效地把旧视图更新到新的状态
```

## 示例
* [元素的动画渲染](http://snabbdom.github.io/snabbdom/examples/reorder-animation/)
* [英雄过渡](http://snabbdom.github.io/snabbdom/examples/hero/)
* [SVG传送带](http://snabbdom.github.io/snabbdom/examples/carousel-svg/)

## 核心部分的文档

Snabbdom的核心部分只提供最重要的功能。它被设计得尽可能简单，同时还很快速并且可扩展。

### `snabbdom.init`

核心只暴露一个单一函数 `snabbdom.init`。这个`init`获取一个模块列表，并返回一个使用指定模块集合的`patch`函数。

```javascript
var patch = snabbdom.init([
  require('snabbdom/modules/class').default,
  require('snabbdom/modules/style').default,
]);
```

### `patch`

`init`返回的`patch`函数有两个参数。第一个参数是呈现当前视图的DOM元素或vnode。第二个参数是呈现新的更新视图的vnode。

如果被传入的DOM元素有一个父元素，`newVnode` 将会被转换成DOM节点，并且传递的元素将会被创建的元素替换。如果传入一个旧节点，Snabbdom将会高效地修改它来匹配新节点中的描述。

被传入的任何一个旧节点必须是之前调用`patch`的结果vnode。这很有必要，因为Snabbdom把信息存储在vnode内。这样就会让它有可能实现一个更简单，更高性能的架构。这样也会禁止一个新旧vnode树的创建。

```javascript
patch(oldVnode, newVnode);
```

### `snabbdom/h`

推荐使用`snabbdom/h`创建vnode。`h`接受一个字符串格式的标签或选择器，一个可选的数据对象，一个可选的字符串或子节点数组。

```javascript
var h = require('snabbdom/h').default;
var vnode = h('div', {style: {color: '#000'}}, [
  h('h1', 'Headline'),
  h('p', 'A paragraph'),
]);
```

### `snabbdom/tovnode`

把DOM节点转换成vnode。特别适合修补预先存在的服务端生产的内容。

```javascript
var snabbdom = require('snabbdom')
var patch = snabbdom.init([ // 选择模块来初始化补丁函数
  require('snabbdom/modules/class').default, // 更方便的切换类
  require('snabbdom/modules/props').default, // 设置DOM元素上的属性
  require('snabbdom/modules/style').default, // 处理支持动画的元素的样式
  require('snabbdom/modules/eventlisteners').default, // 添加事件监听器
]);
var h = require('snabbdom/h').default; // 创建vnode的辅助函数
var toVNode = require('snabbdom/tovnode').default;

var newVNode = h('div', {style: {color: '#000'}}, [
  h('h1', 'Headline'),
  h('p', 'A paragraph'),
]);

patch(toVNode(document.querySelector('.container')), newVNode)

```

### 钩子函数

钩子函数是嵌入DOM节点生命周期的一种方式。Snabbdom提供了丰富的钩子函数集合。钩子函数常被模块用来扩展Snabbdom，而在普通代码中，钩子函数则用来执行虚拟节点周期中期望点的任意代码。

#### 概览

| 名称         | 触发时间                                            | 回调参数   |
| ----------- | --------------                                     | ----------------------- |
| `pre`       | 补丁进程开始                                         | none                     |
| `init`      | 虚拟节点被添加时                                      | `vnode`                 |
| `create`    | 基于vnode的DOM节点被创建时                            | `emptyVnode, vnode`     |
| `insert`    | 元素被插入DOM节点时                                   | `vnode`                 |
| `prepatch`  | 元素将要被修补时                                      | `oldVnode, vnode`       |
| `update`    | 元素将要被更新时                                      | `oldVnode, vnode`       |
| `postpatch` | 元素已经被修补时                                      | `oldVnode, vnode`       |
| `destroy`   | 元素直接或间接的被删除时                               | `vnode`                 |
| `remove`    | 一个直接被从DOM中删除时                                | `vnode, removeCallback` |
| `post`      | 补丁进程结束时                                        | none                    |

下面的钩子函数用于模块：`pre`， `create`，
`update`， `destroy`， `remove`，`post`。

个人元素的`hook`属性提供了下面的钩子函数：`init`， `create`， `insert`， `prepatch`， `update`，
`postpatch`， `destroy`， `remove`。

#### 使用

要使用钩子函数，需要把他们作为一个对象传入数据对象参数的`hook`字段。

```javascript
h('div.row', {
  key: movie.rank,
  hook: {
    insert: (vnode) => { movie.elmHeight = vnode.elm.offsetHeight; }
  }
});
```

#### `init`钩子

当一个新的虚拟节点被发现时，会在修补过程中调用这个钩子。在Snabbdom已经用任意一种方式处理节点之前，这个函数就会被调用。例如基于vnode创建DOM节点之前。

#### `remove`钩子

一旦DOM元素插入文档，这个钩子函数就会被调用，_并且_ 修补周期的剩余部分已经完成。这意味着你可以使用在这个钩子函数内安全地测量DOM尺寸（比如使用[getBoundingClientRect](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect），并且测量之后，那些能够影响插入元素位置的元素都不会被改变。

#### `remove`钩子

删除元素时嵌入这个函数。一旦一个vnode被从一个DOM中删除时，就会调用这个函数。处理函数接受一个vnode和一个回调函数。你可以通过回调控制并延缓删除。一旦这个钩子函数完成了自己的任务，就会立即调用回调函数，并且仅当所有的`remove`钩子都调用完他们的回调时，这哥元素才会被删除。

当一个元素被从它的父元素中删除时，这个钩子函数就会被触发-而不是它的子元素被删除时触发。为此，请查看`destroy`钩子。

#### `destroy`钩子

当虚拟节点的DOM元素被从DOM中移除时，或者虚拟节点的父元素将要被从DOM移除时，这个钩子就会被虚拟节点调用。

对于这个钩子和`remove`钩子之间的区别，参考下面的例子。

```js
var vnode1 = h('div', [h('div', [h('span', 'Hello')])]);
var vnode2 = h('div', []);
patch(container, vnode1);
patch(vnode1, vnode2);
```

例子中的`destroy`会被内部的`div`元素和它包含的`span`元素触发。从另一方面来说，`remove`只会被`div`元素触发，因为它是唯一和它的父元素分离的元素。

例如当一个元素被移除时你可以使用`remove`触发动画，也可以用`destroy`为被删除元素的子元素添加动画。

### 创建模块

通过给[钩子函数](#钩子函数)注册全局的监听函数，模块就能够使用。模块只是把钩子名称映射到函数的字典。

```javascript
var myModule = {
  create: function(oldVnode, vnode) {
    // 新的虚拟节点被创建时调用
  },
  update: function(oldVnode, vnode) {
    // 虚拟节点被更新时调用
  }
};
```

通过这样的机制，你能够轻松的增加Snabbdom的行为。相关说明请查看默认模块的实现。

## 模块文档