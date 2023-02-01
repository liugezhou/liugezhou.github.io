---
layout: post
title: Webpack全基础
author: liugezhou
date: 2021-05-21
updated: 2023-01-31
toc: true
description: Webpack相关内容
categories:
  - 前端小技
tags:
  - JavaScript
---

## 背景

> 1. 随着前端项目工程化、越来越复杂，Webpack 出现了。它是用来实现前端项目模块化的一个静态模块打包工具。
> 1. 所谓静态指的是开发阶段。
> 1. webpack 的作用一是：编译代码能力、提高效率，解决浏览器兼容问题(ES6->ES5)
> 1. webpack 的作用二是：模块整合能力，提高性能，解决了浏览器频繁请求文件的问题
> 1. webpack 的作用三是：项目维护性增强了，支持不同种类的前端模块类型。

## 构建流程简单认识

**从代码角度看：**
传统做法(vue2.5 之前没有 vue.config.js 时)是：将 webpack.base.config.js 中各个配置对象拷贝一份(基础配置)。然后根据不同的环境 merge 不同的配置。
比如测试环境独有的代理 devServer、sourcemap、热更新 HotModuleReplacementPlugin。
比如正式环境独有的 UglifyJsPlugin、extract-text-webpack-plugin、optimize-css-assets-webpack-plugin、html-webpack-plugin 等等。

**base.config.js**
webpack.base.config.js 文件其实就足足的表达了这个 webpack 的构成到底是有哪些部分组成。
由于配置较多，我们这里支只对其中重要的几个配置做简要概述。

- mode: production(代码压缩),development(代码未压缩)
- entry: 打包入口文件，模块构建的起点，一个入口文件最后生成一个 chunk
- output：输出文件，模块构建的终点，可以设置多个输出文件和输出路径
- resolve：文件路径的指向，比如别名配置等，这个配置可以加快打包过程
- modules：这里面主要就是配置的一些 loader
- plugins：这里面主要配置的就是一些基础 plugin

## loader

> 我们必须知道的是 webpack 她只认识 js，loader 就是用来将不是 js 的文件经过**函数**处理成 js。
> 然后这个 loader 的配置，如上所示我们通常写在**modules.rules**属性中。
> 最后，需要注意的是 loader 支持链式调用(每个 loader 可以处理之前已经处理过的资源)，到这对于 loader 的掌握已经算快及格了。
> 接着看完 loader 和 plugin 之后，我们就及格了。

**常用 loader**
**样式 loader**

> - scss-loader:将 scss 文件转换为 css 文件，在 vue 的模板使用中直接安装 node-sass 和 sass-loader 即可使用，但是需要注意版本的问题，版本过高可能会引起报错
> - less-loader：将 less 文件转换为 css 文件,使用时需要安装 less 和 less-loader
> - stylus-loader:stylus 样式写法，使用时需要安装 stylus 和 stylus-loader
> - css-loader：用来处理文件中的@import 和 url()、require 等引入
> - postcss-loader:处理 css3 的前缀等，autoprefixer-loader 已被废弃
> - style-loader:创建一个 style 标签将 css 文件嵌入到 html 当中去,通过 dom 操作 css

**编译 loader**

> - vue-loader:这个 loader 的作用是将扩展名为.vue 的单文件组件转换成 js 模块
> - babel-loader：将 ES6 转换为 ES5 代码
> - ts-loader：将 ts 转为 js
> - awesome-typescript-loader：将 ts 文件转换为 js，性能优于 ts-loader

**文件 loader**

> - raw-loader：可以将文件以字符串的形式返回
> - file-loader：分发文件到 output 目录并返回相对路径
> - url-loader：与 file-loader 类似，但可以将小于配置 limit 大小的文件转换成内敛 Data Url 的方式，减少请求。
> - html-minify-loader:压缩 html

## plugin

> webpack 的 plugin 要比 loader 强大，可以在 webpack 运行到某个时刻的时候，帮你做一些事情 类似与 vue 中的生命周期函数(比如html-webpack-plugin会在打包结束后自动生成html文件)。  
> 通过之前的学习，我们也知道需要哪些插件，我们就直接引入，然后以 new 对象的形式传入 plugins 配置对象中去就可以。
> 然后，如果再深入一点点的话，我们看 plugin 的本质：其实就是具有一个**apply**方法的 JS 对象。这个 apply 方法是会被 webpack 的 compiler 调用的。并且在整个编译生命周期都可以访问 compiler 对象。

**_内置插件_**

- uglifyJsPlugin:压缩和混淆代码。
- CommonsChunkPlugin:提高打包效率，将第三方库和业务代码分开打包
- HotModuleReplacementPlugin:热更新
- DefinePlugin:编译时配置全局变量，这对开发模式和发布模式的构建允许不同行为非常有用

**其它插件**

- html-webpack-plugin：可以根据模板自动生成 html 代码，并自动引用 css 和 js 文件
- ProvidePlugin：自动加载模块，代替 require 和 import
- extract-text-webpack-plugin:将 js 文件中引用的样式单独抽离成 css 文件
- optimize-css-assets-webpack-plugin:不同组件中重复的 css 可以快速去重
- clean-webpack-plugin:打包生成 dist 目录下的覆盖

## loader 与 plugin 的区别，以及如何自定义

**区别**

> - loader 本身就只是一个函数，在该函数中对接收到的内容进行转换。它是个翻译官，它在 modules 的 rules 中配置，内部包含 test、loader 和 options 属性。
> - Plugin 就是插件，基于事件流。Webpack 在运行当中会去广播一些事件，plugin 去监听这些事件，然后干活。plugin 单独配置，通过构造函数传入参数生效。

**自定义 loader**

> - loader 本质上是一个函数
> - 因为函数中的 this 作为上下文会被 webpack 填充，因此不能将 loader 设为一个箭头函数
> - 该函数接受一个参数，这个参数是 webpack 传递给 loader 的文件源内容

**自定义 Plugin**

> - webpack 编译会创建两个核心对象：compiler 和 compilation
> - compiler：包含了 webpack 环境的所有配置消息，包括 options、loader 和 plugin，以及 webpack 整个生命周期相关的钩子
> - compilation：作为 Plugin 内置事件回调函数的参数，包含了当前的模块资源、编译生成资源、变化的文件以及被跟踪依赖的状态信息。当检测到一个文件变化，一次新的 compilation 将被创建。

```javascript
// 导出一个函数，其中source为webpack传递给loader的输入参数--文件源内容
module.exports = function (source) {
  const content = doSomething2JsString(source)
  // 如果loader配置了options对象，那么this.query将指向options
  const options = this.query
  this.fallback(null, content) //异步
  return content //同步
}
```

自定义 Plugin，需要遵循的规范是：

> - 插件必须是一个函数或是包含 apply 方法的对象，这样才能访问 compiler 实例

```javascript
class MyPlugin {
  //Webpack会调用MyPlugin实例的apply方法给插件实例传入compiler对象
  apply(compiler) {
    // 找到合适的事件钩子，实现自己的插件
    compiler.hooks.emit.tap('MyPlugin', (compilation) => {
      //do something
    })
  }
}
```

## 热更新

webpack 的热更新又称为热替换(Hot Module Replacement) -- HMR:这个机制可以做到不用刷新浏览器而将变更的模块替换掉。

HMR 的核心就是：客户端从服务端拉去更新后的文件(他们直接维护了一个**websocket**)，当本地资源发生变更后，客户端进行资源对比，然后增量更新。  
开启 HMR，要在 webpack 配置文件的 devServer 中设置 hot 为 true 即可。

## SourceMap 设置

- 项目打包后，如果关闭 sourceMap 的配置，在浏览器打开项目后，看到的 js 代码为打包后的代码，不利于查找代码错误。
- sourceMap 是一个映射关系，他可以知道在 dist 打包后的 main.js 错误的代码对应在未经打包的代码的位置。

配置项为：

- devtool:'source-map'---会在 dist 目录下生成一个.map 的映射文件。
  - 如果为'inline-source-map'，则不会生成.map 文件，直接在原 main.js 文见中添加注释以映射(位置在底部)。
  - 如果为'cheap-inline-source-map' :与 inline 不同，只告诉是哪行代码出错，效率会高一些。
  - 如果为"cheap-module-source-map':不管是业务代码，但是依赖的第三方模块，都会显示出出错的地方。
  - eval 是打包效率最高的方式。
- 如果是开发环境，建议使用`cheap-module-eval-source-map`这种方式。
- 如果是生产环境，一般不用设置 devtool 的配置。如果要配置，推荐使用`cheap-module-source-map`。

## 代理

**配置**

> webpack 中提供服务器的工具为 webpack-dev-server，只适用与开发阶段
> 配置核心为：devServer -> proxy

**原理**

> Proxy 工作原理实际上利用**http-proxy-middleware**这个 http 中间件，实现请求转发给其他服务器。

```javascript
const express = require('express')
const proxy = require('http-proxy-middleware')
const app = new express()
app.use('/api',proxy({target:'http://liugezhou.online'},changeOrigin:true))
// http://localhost:3000/api/foo/bar ->http://liugezhou.online/aoi/foo/bar
```

**跨域**

> 过程：
>
> - webpack-dev-server 在本地开发时启动了一个服务器，我们开发的应用运行在这个服务器上
> - 后端服务运行在另一个服务上
> - 这个时候由于**浏览器的同源策略**，访问后端服务就会出现跨域现象
> - 然后使用 devServer-proxy 配置，相当于开了一个代理服务器
> - 于是交互变成：本地发生请求、代理服务器接受请求、代理服务器将请求发生给目标服务器，然后再倒叙顺序返回
> - 由于服务器与服务器直接请求数据不会发生跨域行文，所以上面的流程就跑通了(跨域行为是浏览器的同源策略导致的)

## TreeShaking

**要解决的问题**
在 math.js 这个模块中有两个方法 add 和 minus，在 index 中只调用 add 方法，去打包的时候，会将 math 中的两个方法均打包，这样做是没有必要，且会使得打包文件变大，Tree Shaking 就是为了解决这个问题的。

- 开发环境下：默认没有TreeShaking功能。
- 生成环境下：默认配置好，但是仍然需要对 package.json 中的 sideEffects 进行配置。

## Code Splitting
Webpack的代码分割与webpack无关，只是对代码进行分割，提高代码的执行效率和性能。  
比如引入了第三方包lodash，在entry中设置将业务代码和引入的包分别单独打包。

## Chunk指什么
chunk 指的是整个项目完成打包后，dist 下面有几个 js 文件就是指几个 chunk。

## 借助 webpack 优化性能

> - JS 代码压缩 -> uglifyJsPlugin/terserPlugin
> - CSS 代码压缩 ->optimize-assets-css-webpack-plugin/css-minimizer-webpack-plugin
> - HTML 文件代码压缩 ->htmlwebpackplugin 设置 minify 属性进行优化
> - 文件大小压缩 -> compression-webpack-plugin
> - 图片压缩 -> url-loader/
> - Tree Shaking ->
> - 代码分离 -> splitChunkPlugin
> - 内联 chunk ->

## 提搞 webpack 的构建速度

**优化 loader 配置**

> 使用 loader 时，可以通过配置 include、exclude、test 等属性来匹配文件

**合理使用 resolve.extensions**
**优化 resolve.modules**

> 项目构建时，可以通过指明存放第三模块的绝对路径来减少寻找的时间

**优化 resolve.alias**

> 别名使用

## 除了 webpack，其他模块管理工具

**rollup**

> 相比 webpack，rollup 要小巧很多，当下的 vue、react、three.js 都是使用 rollup 打包

**vite**

> - 快速冷启动
> - 即时热更新
> - 真正的按需编译

**parcel**
