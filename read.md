## [Vue.js 技术揭秘 & 源码阅读教程](https://ustbhuangyi.github.io/vue-analysis/)

## 第一天 (熟悉 vue 的源码工程)
## vue是使用什么工具来构建?有什么优势?

  vue源码基于 rollup 构建,是一个 js 模块打包器一般是用来打包 js 库;
```
  1.兼容 ES6 模块;
  2.能排除未实际使用的代码;
  3.代码打包后能控制较小的体积;
  4.兼容 CommonJS;
  ```
## vue使用什么工具来规范代码?有什么优势?

  flow 是 js 静态类型检查工具, 他的作用是为了规避 js 作为动态语言的劣势与隐患;

```
1.类型推断: 通过变量的使用上下文来推断出变量的类型,然后根据这些推断来检查类型.
2.类型注释: 事先注释号我们期待的类型,flow 会基于这些注释来判断.
```
## 了解vue目录结构

```
src
├── compiler        # 编译相关 
├── core            # 核心代码 
├── platforms       # 不同平台的支持
├── server          # 服务端渲染
├── sfc             # .vue 文件解析
├── shared          # 共享代码
```

### compiler
compiler 目录包含 Vue.js 所有编译相关的代码。它包括把模板解析成 ast [(抽象语法树)](https://segmentfault.com/a/1190000016231512) ，ast 语法树优化，代码生成等功能。

编译的工作可以在构建时做（借助 webpack、vue-loader 等辅助插件）；也可以在运行时做，使用包含构建功能的 Vue.js。显然，编译是一项耗性能的工作，所以更推荐前者——离线编译。

### core
core 目录包含了 Vue.js 的核心代码，包括内置组件、全局 API 封装，Vue 实例化、观察者、虚拟 DOM、工具函数等等。

这里的代码可谓是 Vue.js 的灵魂，也是我们之后需要重点分析的地方。

### platform
Vue.js 是一个跨平台的 MVVM 框架，它可以跑在 web 上，也可以配合 weex 跑在 native 客户端上。platform 是 Vue.js 的入口，2 个目录代表 2 个主要入口，分别打包成运行在 web 上和 weex 上的 Vue.js。

我们会重点分析 web 入口打包后的 Vue.js，对于 weex 入口打包的 Vue.js，感兴趣的同学可以自行研究。

### server
Vue.js 2.0 支持了服务端渲染，所有服务端渲染相关的逻辑都在这个目录下。注意：这部分代码是跑在服务端的 Node.js，不要和跑在浏览器端的 Vue.js 混为一谈。

服务端渲染主要的工作是把组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将静态标记"混合"为客户端上完全交互的应用程序。


### sfc
通常我们开发 Vue.js 都会借助 webpack 构建， 然后通过 .vue 单文件来编写组件。

这个目录下的代码逻辑会把 .vue 文件内容解析成一个 JavaScript 的对象。

### shared
Vue.js 会定义一些工具方法，这里定义的工具方法都是会被浏览器端的 Vue.js 和服务端的 Vue.js 所共享的。


## vue的大致构建过程?

构建的入口 JS 文件 scripts/build.js

build函数读取builds配置项 scripts/config.js

Runtime Only VS Runtime + Compiler
通常我们利用 vue-cli 去初始化我们的 Vue.js 项目的时候会询问我们用 Runtime Only 版本的还是 Runtime + Compiler 版本。下面我们来对比这两个版本。

Runtime Only
我们在使用 Runtime Only 版本的 Vue.js 的时候，通常需要借助如 webpack 的 vue-loader 工具把 .vue 文件编译成 JavaScript，因为是在编译阶段做的，所以它只包含运行时的 Vue.js 代码，因此代码体积也会更轻量。

Runtime + Compiler
我们如果没有对代码做预编译，但又使用了 Vue 的 template 属性并传入一个字符串，则需要在客户端编译模板，如下所示：

```
// 需要编译器的版本
new Vue({
  template: '<div>{{ hi }}</div>'
})

// 这种情况不需要
new Vue({
  render (h) {
    return h('div', this.hi)
  }
})
```
## 入口
在 web 应用下，我们来分析 Runtime + Compiler 构建出来的 Vue.js，它的入口是 src/platforms/web/entry-runtime-with-compiler.js

在这个入口 JS 的上方我们可以找到 Vue 的来源：import Vue from './runtime/index'，我们先来看一下这块儿的实现，它定义在 src/platforms/web/runtime/index.js 中

继续发现在 src/core/instance/index.js 中用一个 Vue 的函数去实现;

```
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```
使用者引入 vue.js 然后 new Vue();

## 为何 Vue 不用 ES6 的 Class 去实现呢？

  1.Vue 的 prototype 上扩展一些方法

  2.Vue 按功能把这些扩展分散到多个模块中去实现，而不是在一个模块里实现所有，这种方式是用 Class 难以实现的。

## 总结
它本质上就是一个用 Function 实现的 Class，然后它的原型 prototype 以及它本身都扩展了一系列的方法和属性，



## 第二天
- new Vue() 发生了什么?

  ```
  1.Vue是一个函数,new Vue()的时候,是对 vue 的实例化;
    在 core/instance/index.js 中;
  2.先是调用 this._init(options) 函数;
    _init函数是在 Vue构造函数中,通过原型继承的方式在prototype对象中定义,是在 core/instance/init.js 中定义;
  Vue 初始化主要就干了几件事情，合并配置，初始化生命周期，初始化事件中心，初始化渲染，初始化 data、props、computed、watcher 等等。
  ```
- Vue 实例挂载如何实现?
- render 函数的作用?
- vue 中的Virtual DOM是如何实现?
- vue VNode创建与更新?

- [额外的任务,原型相关](https://blog.csdn.net/cc18868876837/article/details/81211729)