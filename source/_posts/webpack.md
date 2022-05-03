---
layout: post
title: webpack
author: liugezhou
date: 2021-05-21
updated: 2021-05-25
category: 
    前端小技
tags:
    webpack
---
## 1、背景
> 1. 随着前端项目工程化、越来越复杂，Webpack出现了。它是用来实现前端项目模块化的一个静态模块打包工具。
> 1. 所谓静态指的是开发阶段。
> 1. webpack的作用一是：编译代码能力、提高效率，解决浏览器兼容问题(ES6->ES5)
> 1. webpack的作用二是：模块整合能力，提高性能，解决了浏览器频繁请求文件的问题
> 1. webpack的作用三是：项目维护性增强了，支持不同种类的前端模块类型。

## 2、构建流程简单认识
### 从代码角度看：
> 传统做法(vue2.5之前没有vue.config.js时)是：将webpack.base.config.js中各个配置对象拷贝一份(基础配置)。然后根据不同的环境merge不同的配置。
> 比如测试环境独有的代理devServer、sourcemap、热更新HotModuleReplacementPlugin。
> 比如正式环境独有的UglifyJsPlugin、extract-text-webpack-plugin、optimize-css-assets-webpack-plugin、html-webpack-pluin等等。

### base.config.js
> webpack.base.config.js文件其实就足足的表达了这个webpack的构成到底是有哪些部分组成。
> 由于配置较多，我们这里支只对其中重要的几个配置做简要概述。
> - entry: 入口文件，模块构建的起点，一个入口文件最后生成一个chunk
> - output：输出文件，模块构建的终点，可以设置输出文件和输出路径
> - resolve：文件路径的指向，比如别名配置等，这个配置可以加快打包过程
> - modules：这里面主要就是配置的一些loader
> - plugins：这里面主要配置的就是一些基础plugin

## 3. loader
> 我们必须知道的是webpack她只认识js，loader就是用来将不是js的文件经过**函数**处理成js。
> 然后这个loader的配置，如上所示我们通常写在**modules.rules**属性中。
> 最后，需要注意的是loader支持链式调用(每个loader可以处理之前已经处理过的资源)，到这对于laoder的掌握已经算快及格了。
> 接着看完loader和plugin之后，我们就及格了。

### 常用loader
#### 样式loader
> - scss-loader:将scss文件转换为css文件，在vue的模板使用中直接安装node-sass和sass-loader即可使用，但是需要注意版本的问题，版本过高可能会引起报错
> - less-loader：将less文件转换为css文件,使用时需要安装 less和less-loader
> - stylus-loader:stylus样式写法，使用时需要安装stylus和stylus-loader
> - css-loader：用来处理文件中的@import和url()、require等引入
> - postcss-loader:处理css3的前缀等，autoprefixer-loader已被废弃
> - style-loader:创建一个style标签将css文件嵌入到html当中去,通过dom操作css 

#### 编译loader
> - vue-loader:这个loader的作用是将扩展名为.vue的单文件组件转换成js模块
> - babel-loader：将ES6转换为ES5代码
> - ts-loader：将ts转为js
> - awesome-typescript-loader：将ts文件转换为js，性能优于ts-loader

#### 文件loader
> - raw-loader：可以将文件以字符串的形式返回
> - file-loader：分发文件到output目录并返回相对路径
> - url-loader：与file-loader类似，但可以将小于配置limit大小的文件转换成内敛Data Url的方式，减少请求。
> - html-minify-loader:压缩html

## 4. plugin
> webpack的plugin要比loader强大，这个plugin在webpack的整个生命周期活动，可以做一些在构建范围内的事情。
> 通过之前的学习，我们也知道需要哪些插件，我们就直接引入，然后以new对象的形式传入plugins配置对象中去就可以。
> 然后，如果再深入一点点的话，我们看plugin的本质：其实就是具有一个**apply**方法的JS对象。这个apply方法是会被webpack的compiler调用的。并且在整个编译生命周期都可以访问compiler对象。

#### **_内置插件_**
> - uglifyJsPlugin:压缩和混淆代码。
> - CommonsChunkPlugin:提高打包效率，将第三方库和业务代码分开打包
> - HotModuleReplacementPlugin:热更新
> - DefinePlugin:编译时配置全局变量，这对开发模式和发布模式的构建允许不同行为非常有用

#### 其它插件
> - html-webpack-plugin：可以根据模板自动生成html代码，并自动引用css和js文件
> - ProvidePlugin：自动加载模块，代替require和import
> - extract-text-webpack-plugin:将js文件中引用的样式单独抽离成css文件
> - optimize-css-assets-webpack-plugin:不同组件中重复的css可以快速去重


## 5. loader与plugin的区别，以及如何自定义
### 区别
> - loader本身就只是一个函数，在该函数中对接收到的内容进行转换。它是个翻译官，它在modules的rules中配置，内部包含test、loader和optins属性。
> - Plugin就是插件，基于事件流。Webpack在运行当中会去广播一些事件，plugin去监听这些事件，然后干活。plugin单独配置，通过构造函数传入参数生效。

### 自定义loader
> - loader本质上是一个函数
> - 因为函数中的this作为上下文会被webpack填充，因此不能将loader设为一个箭头函数
> - 该函数接受一个参数，这个参数是webpack传递给loader的文件源内容

### 自定义Plugin
> - webpack编译会创建两个核心对象：compiler和compilation
> - compiler：包含了webpack环境的所有配置消息，包括options、loader和plugin，以及webpack整个生命周期相关的钩子
> - compilation：作为Plugin内置事件回调函数的参数，包含了当前的模块资源、编译生成资源、变化的文件以及被跟踪依赖的状态信息。当检测到一个文件变化，一次新的compilation将被创建。

```javascript
// 倒出一个函数，其中source为webpack传递给loader的输入参数--文件源内容
module.exports = function(source){
  const content = doSomething2JsString(source);
  // 如果loader配置了options对象，那么this.query将指向options
  const options = this.query
  this.fallback(null,content) //异步
  return content; //同步
}
```
自定义Plugin，需要遵循的规范是：
> - 插件必须是一个函数或是包含apply方法的对象，这样才能访问compiler实例

```javascript
class MyPlugin{
  //Webpack会调用MyPlugin实例的apply方法给插件实例传入compiler对象
  apply(compiler){
    // 找到合适的事件钩子，实现自己的插件
    compiler.hooks.emit.tap('MyPlugin',compilation=>{
      //do something
    })
  }
}
```
## 5. 热更新
> webpack的热更新又称为热替换(Hot Module Replacement) -- HMR
> 这个机制可以做到不用刷新浏览器而将变更的模块替换掉。
> 
> HMR的核心就是：客户端从服务端拉去更新后的文件(他们直接维护了一个**websocket)**，当本地资源发生变更后，客户端进行资源对比，然后增量更新。
> 开启HMR，要在webpack配置文件的devServer中设置hot为true即可。

## 6.代理
#### 配置
> webpack中提供服务器的工具为webpack-dev-server，只适用与开发阶段
> 配置核心为：devServer -> proxy

#### 原理
> Proxy工作原理实际上利用**http-proxy-middleware**这个http中间件，实现请求转发给其他服务器。

```javascript
const express = require('express')
const proxy = require('http-proxy-middleware')
const app = new express()
app.use('/api',proxy({target:'http://liugezhou.online'},changeOrigin:true))
// http://localhost:3000/api/foo/bar ->http://liugezhou.online/aoi/foo/bar
```
#### 跨域
> 过程：
> - webpack-dev-server在本地开发时启动了一个服务器，我们开发的应用运行在这个服务器上
> - 后端服务运行在另一个服务上
> - 这个时候由于**浏览器的同源策略**，访问后端服务就会出现跨域现象
> - 然后使用devServer-proxy配置，相当于开了一个代理服务器
> - 于是交互变成：本地发生请求、代理服务器接受请求、代理服务器将请求发生给目标服务器，然后再倒叙顺序返回
> - 由于服务器与服务器直接请求数据不会发生跨域行文，所以上面的流程就跑通了(跨域行为是浏览器的同源策略导致的)

## 7. 借助webpack优化性能
> - JS代码压缩 		-> uglifyJsPlugin/terserPlugin
> - CSS代码压缩	->optimize-assets-css-webpack-plugin/css-minimizer-webpack-plugin
> - HTML文件代码压缩		->htmlwebpackplugin设置minify属性进行优化
> - 文件大小压缩	-> compression-webpack-plugin
> - 图片压缩 	-> url-loader/
> - Tree Shaking	->	
> - 代码分离	-> splitChunkPlugin
> - 内联chunk	->

## 8. 提搞webpack的构建速度
#### 优化loader配置
> 使用loader时，可以通过配置include、exclude、test等属性来匹配文件

#### 合理使用resolve.extensions
#### 优化resolve.modules
> 项目构建时，可以通过指明存放第三模块的绝对路径来减少寻找的时间

#### 优化resolve.alias
> 别名使用

## 9.除了webpack，其他模块管理工具
#### rollup
> 相比webpack，rollup要小巧很多，当下的vue、react、thress.js都是使用roullup打包

#### vite
> - 快速冷启动
> - 即时热更新
> - 真正的按需编译

#### parcel

