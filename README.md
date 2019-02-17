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

## 为什么要使用虚拟DOM

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