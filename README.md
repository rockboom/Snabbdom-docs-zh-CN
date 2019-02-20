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

这个部分描述核心模块。所有的模块都是可选的。

### class模块

class模块提供了一种简单的方式来动态地切换元素的类名。class模块期望`class`数据属性是一个对象。该对象应该把类名映射成布尔值，用来指明这个类名是否应该在vnode上保留或者是删除。

```javascript
h('a', {class: {active: true, selected: false}}, 'Toggle');
```

### props模块

用来设置DOM元素的属性。

```javascript
h('a', {props: {href: '/foo'}}, 'Go to Foo');
```

### attributes模块

和props模块相同，不过是用来设置DOM的属性而不是用来设置DOM特性。

```javascript
h('a', {attrs: {href: '/foo'}}, 'Go to Foo');
```

用`setAttribute`对attributes进行添加和更新。如果先前添加/设置的属性在`attrs`对象中不再存在，则使用`removeAttribute`把该属性从DOM元素的属性列表中删除。

如果是布尔属性(比如`disabled`，`hidden`，
`selected`等 )，意味着不再依赖属性值(`true`或`false`)，而是取决于该属性自身在DOM元素中的存在/缺失。这些属性会被不同的模块处理：如果布尔属性被设置成[falsy值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)(`0`, `-0`, `null`, `false`,`NaN`, `undefined`, 或者空字符串(`""`))，name这个属性将会被从DOM元素的属性列表中删除。

### dataset模块

可以设置DOM元素的自定义数据属性(`data-*`)。这些自定义属性可以通过[HTMLElement.dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset)属性访问。

```javascript
h('button', {dataset: {action: 'reset'}}, 'Reset');
```

### style模块

style模块用于让HTML看起来平滑，并且流畅的执行动画。style模块的核心允许在DOM元素上设置CSS属性。

```javascript
h('span', {
  style: {border: '1px solid #bada55', color: '#c0ffee', fontWeight: 'bold'}
}, 'Say my name, and every colour illuminates');
```

值得注意的是，如果style属性被当做style对象的属性移除时，style模块并不会移除style属性。要想移除一个样式，你应该把对应的属性值设为空字符串。

```javascript
h('div', {
  style: {position: shouldFollow ? 'fixed' : ''}
}, 'I, I follow, I follow you');
```

#### 自定义属性（CSS变量）

支持CSS的自定义属性（aka CSS属性），属性必须带有前缀`--`

```javascript
h('div', {
  style: {'--warnColor': 'yellow'}
}, 'Warning');
```

### 延时属性

你也可以指明延时属性。无论什么时候更改延时属性，都会直到下一帧后才会应用更改。

```javascript
h('span', {
  style: {opacity: '0', transition: 'opacity 1s', delayed: {opacity: '1'}}
}, 'Imma fade right in!');
```

这使得声明式地为元素的输入设置动画变得容易。

#### 为`remove`设置属性

一旦元素将要被从DOM中移除，Styles中设置的`remove`属性就会执行。应用的样式应使用CSS过渡进行动画处理。只有在完成所有样式动画后，才会从DOM中删除该元素。

```javascript
h('span', {
  style: {opacity: '1', transition: 'opacity 1s',
          remove: {opacity: '0'}}
}, 'It\'s better to fade out than to burn away');
```

这使得声明式地为元素的移除设置动画变得容易。

#### 为`destroy`设置属性

```javascript
h('span', {
  style: {opacity: '1', transition: 'opacity 1s',
          destroy: {opacity: '0'}}
}, 'It\'s better to fade out than to burn away');
```

### 事件监听模块

事件监听模块为绑定事件监听器提供了强大的功能。

通过给`on`提供一个对象，对象的属性就是你想监听的事件名，就可以给vnode的事件绑定函数。当事件执行时，函数就会被调用，并且该函数会被传递给属于它的事件对象。

```javascript
function clickHandler(ev) { console.log('got clicked'); }
h('div', {on: {click: clickHandler}});
```

但是，很多时候你并不会在意事件对象本身。通常你会有一些与触发事件的元素相关联的数据，并且希望传递那些数据。

想象一个有三个按钮的计数器应用，一个将计数器增加1，一个将计数器增加2，一个将计数器增加3。事实上你并不会关心哪一个按钮被点击了。你在意的是哪个数字和被点击的按钮关联。事件监听模块可以通过在命名的事件属性中提供一个数组来表达这样的意思。数组的第一个元素是事件执行时被调用的函数，函数的调用的值就是数组的第二个元素。

```javascript
function clickHandler(number) { console.log('button ' + number + ' was clicked!'); }
h('div', [
  h('a', {on: {click: [clickHandler, 1]}}),
  h('a', {on: {click: [clickHandler, 2]}}),
  h('a', {on: {click: [clickHandler, 3]}}),
]);
```

每个处理函数被调用时不仅使用指定的参数，也会使用添加到参数列表中的当前事件和vnode。事件监听模块也支持为每个事件绑定多个处理函数，需要为事件指定一个处理函数的数组：
```javascript
stopPropagation = function(ev) { ev.stopPropagation() }
sendValue = function(func, ev, vnode) { func(vnode.elm.value) }

h('a', { on:{ click:[[sendValue, console.log], stopPropagation] } });
```

Snabbdom也允许在渲染器之间互换事件处理函数。这种情况发生时并没有实际地接触附加到DOM上的事件处理函数。

但是，要注意，**在虚拟节点之间共享事件处理函数时应该小心**，因为这个事件处理模块使用的技术禁止在DOM上重新绑定事件处理函数（并且通常来说，在虚拟节点之间共享数据并不能确保能够起作用，因为模块允许改变给定的数据）。

特别是不能像这样做：

```javascript
// Does not work
var sharedHandler = {
  change: function(e){ console.log('you chose: ' + e.target.value); }
};
h('div', [
  h('input', {props: {type: 'radio', name: 'test', value: '0'},
              on: sharedHandler}),
  h('input', {props: {type: 'radio', name: 'test', value: '1'},
              on: sharedHandler}),
  h('input', {props: {type: 'radio', name: 'test', value: '2'},
              on: sharedHandler})
]);
```

大多数这样的例子可以使用基于数组的处理函数代替（上面的内容有描述）。另外，还要确保每一个节点只有唯一的`on`值：

```javascript
// Works
var sharedHandler = function(e){ console.log('you chose: ' + e.target.value); };
h('div', [
  h('input', {props: {type: 'radio', name: 'test', value: '0'},
              on: {change: sharedHandler}}),
  h('input', {props: {type: 'radio', name: 'test', value: '1'},
              on: {change: sharedHandler}}),
  h('input', {props: {type: 'radio', name: 'test', value: '2'},
              on: {change: sharedHandler}})
]);
```

## 辅助函数

### SVG

当使用`h`函数创建虚拟节点时，SVG才会起作用。SVG元素会自动在合适的命名空间下创建。

```javascript
var vnode = h('div', [
  h('svg', {attrs: {width: 100, height: 100}}, [
    h('circle', {attrs: {cx: 50, cy: 50, r: 40, stroke: 'green', 'stroke-width': 4, fill: 'yellow'}})
  ])
]);
```

可以查看[SVG示例](https://github.com/snabbdom/snabbdom/tree/master/examples/svg)和[SVG Carousel示例](https://github.com/snabbdom/snabbdom/tree/master/examples/carousel-svg)

#### 在SVG元素中使用类名

某些浏览器（比如IE<=11）[不支持SVG元素的`classList`属性](http://caniuse.com/#feat=classlist)。因此，_class_ 模块（内部使用`classList`属性）在这些浏览器内就不能运行。

SVG元素的类选择器从0.6.7版本开始就能良好的运行。

对于这些例子你可以使用 _attributes_ 模块给SVG元素添加动态的类名，像下面展示的数组一样：

```js
h('svg', [
  h('text.underline', { // 'underline'是一个类选择器，在渲染器之间也不会改变。
      attrs: {
        // 'active' 和 'red' 是动态类名，会在渲染器之间发生改变。
        // 因此我们需要把他们放在class属性当中去。
        // (通常我们使用class 模块，但是在SVG内部不起作用)
        class: [isActive && "active", isColored && "red"].filter(Boolean).join(" ")
      }
    },
    'Hello World'
  )
])
```

### Thunks

`thunks`函数接受一个选择器，一个指明thunk的key，一个返回vnode的函数，以及一个数量可变的状态参数。如果发生调用，这个渲染函数将会接受状态参数。

`thunk(selector, key, renderFn, [stateArguments])`

`key`是可选的。当`selector`在thunks的子节点中不唯一时，才会提供`key`。这样就能确保当有不同时总能正确地匹配。

当一个元素用一成不变的数据处理时，Thunks是一个可以使用的优化策略。

想象一个基于数字创建虚拟节点的简单函数。

```js
function numberView(n) {
  return h('div', 'Number is: ' + n);
}
```

视图仅依赖于`n`。这意味着如果`n`不发生改变，那么创建虚拟DOM节点和修补旧的虚拟节点就是浪费。我们可以使用`thunk`辅助函数禁止上面的行为。

```js
function render(state) {
  return thunk('num', numberView, [state.number]);
}
```

实际上并不会调用`numberView`函数，只会在虚拟数中放置一个假的虚拟节点。当Snabbdom对前一个vnode修补这个虚拟节点时，他将会比较`n`的值。如果`n`不改变，那他就会简单的使用旧的虚拟节点。这样就会禁止重新创建数字视图和差异过程。

这里的视图函数只是一个例子。如果你在渲染一个复杂的视图，并且该视图会浪费大量的计算时间生成，实际上thunks才是相关的。

## 虚拟节点

**属性**
 - [sel](#sel--string)
 - [data](#data--object)
 - [children](#children--array)
 - [text](#text--string)
 - [elm](#elm--element)
 - [key](#key--string--number)

#### sel : String

虚拟节点的`.sel`属性是在创建期间传递给[`h()`](#snabbdomh)的CSS选择器。例如：`h('div#container',
{}, [...])`将会创建一个把`div#container`作为它的`.sel`属性的虚拟节点。

#### data : Object

虚拟节点的`.data`属性用来添加信息的位置，当虚拟节点被创建时，[模块](#模块文档)通过`.data`属性来访问以及操纵真实的节点；添加样式，CSS类名，属性，等等。

数据对象是[`h()`](#snabbdomh)中的第二个参数（可选）

比如 `h('div', {props: {className: 'container'}}, [...])` 将会生成一个虚拟节点，并用

```js
{
  "props": {
    className: "container"
  }
}
```

作为它的`.data`对象。

#### children : Array<vnode>

虚拟节点的`.children`属性是[`h()`](#snabbdomh)创建时的第三个参数。`.children` 只是一个虚拟节点的数组，这些虚拟节点应该在创建之前被添加到父DOM节点的子节点中去。

例如`h('div', {}, [ h('h1', {}, 'Hello, World') ])`将会创建一个虚拟节点，并把

```js
[
 {
   sel: 'h1',
   data: {},
   children: undefined,
   text: 'Hello, World',
   elm: Element,
   key: undefined,
 }
]
```

作为它的`.children`属性。