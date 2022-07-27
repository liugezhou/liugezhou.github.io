---
layout: post
title: Babel有关基础内容
date: 2021-09-09
updated: 2022-05-04
description: 本文，抽丝剥茧，彻底的搞懂babel，并测试、实践修改babe的配置。
toc: true
categories:
- 前端小技
tags: 
    - JavaScript
---
<font color=blue>更新说明：对文章排版以及内容格式做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

## 背景
项目中babel的设置一团遭，确实有必要搞懂这个babel。有的项目中直接在配置文件.babelrc中配置好，有的在main.js中全局import 这个polyfill，有的是在webpack中配置，有的引入了大量的第三方babel插件，这样可不行啊，如果不对babel有个全面的认知，在面对这些一系列满眼的配置下去修改，就有点力不从心。
本文，抽丝剥茧，彻底的搞懂babel，并测试、去实践修改babe的配置。

## 一、babel基础介绍
1. babel是什么？  
[Babel](https://babeljs.io/) 是一个广泛使用的 ES6 转码器，可以将 ES6 代码转为 ES5 代码，从而在老版本的浏览器执行。这意味着，你可以用 ES6 的方式编写程序，又不用担心现有环境是否支持。
2. Babel的基本工作原理
主要分为 parsing、transforming、printing三个阶段
- parsing为解析阶段，将ES6代码进行词法分析和语法分析转换成抽象语法树  
- transforming为转换阶段，将抽象语法树进行变换操作  
- printing为生成阶段，通过babel-generator生成对应的代码。

3. 配置文件.babelrc的基本格式
首先，我们先对babel配置的基本格式有个浅显的了解：他是个对象、他的两个属性都是数组，他们分别是presets和plugins。

```javascript
{
  "presets": [],
  "plugins": []
}
```
接着我们在介绍这两个属性前，了解下babel的插件与预设的概念。 

## 二、babel插件与预设
在继续上面的问题之前，这里我们需要深刻的晓得：babel的核心是**插件**，babel的所有工作都是由插件完成的。然后我们去查看babel的插件列表：[https://babeljs.io/docs/en/plugins-list](https://babeljs.io/docs/en/plugins-list)，部分截图如下

![babel](https://cdn.jsdelivr.net/gh/liugezhou/image@master/img/babel.5z62x05kn300.webp)

其中arrow-functions(也就是ES6的箭头函数)对应的插件是@babel/plugin-transform-arrow-functions
如果要使用这个插件，需要安装：npm install --save-dev @babel/plugin-transform-arrow-functions
这个插件作用就是将ES6的箭头函数转化为ES5写法。
![babel](https://cdn.jsdelivr.net/gh/liugezhou/image@master/img/babel.1pbhb3j4a5s0.webp)

进入到这个方法的详情页，如上图，我们看到：**This plugin is included in @babel/preset-env**
也就是说这个插件以及其他各种单个插件都放在了@babel/preset-env这个包中。之所以要**included到一个里面**，是因为我们总不能一个一个插件的安装引入，官方就提供了一个包： 
**嘿，兄弟，我把一些你们都会用到的一系列的插件包在一块，你们就不用单个安装了，只需要执行下面的命令**：
npm install --save-dev @babel/preset-env'

 此时，这个preset-env(一系列插件的集合)，安装好后，我们就可以去配置文件里面配置了。
【这个presets叫做“预设”：预设可以看成是一组插件或者配置项的集合。】

```javascript
{
  "plugins": [],
  "presets": [
    "@babel/preset-env"
  ]
}
```
 
## 三、✨✨✨垫片-polyfill的使用
垫片就是我们平时早已知道的babel-polyfill。
为什么引入了babel还需要引入babel-polyfill呢。   
这是因为babel将ES6代码是分成了两种情况进行转换的：**语法层和API层**。   
- 语法层就是let const 箭头函数 class这些      
- API层就是 includes Promise  map这些，因此上面的preset-env预设只是对语法层进行了转换，需要对低版本的ES6上的API进行转换就需要polyfill了。


之前的项目中，我们的使用方式如下：
首先，我们需要安装它：
- npm install @babel/polyfill -S

关于项目中使用polyfill，有好多种写法，我们这里需要对他们的每个写法都了然于胸，才会 知道如何真正优化配置。

**type1:**
> 在 node/React/Vue中使用,可能需要在入口文件最上层引入：
> - require('@babel/polyfill') 
> - import '@babel/polyfill"

**type2:**
> 在webpack中可以通过配置文件加入：
> module.exports = {
>   entry: ["@babel/polyfill", "./app/js"],
> };

```javascript
{
  "plugins": [],
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns":"usage",
        "modules": false
      }
    ]
  ]
}
```
这样引入polyfill，存在的一个问题是打出的包非常的大。
这里有一个很重要的配置 useBuiltIns : usage  
它的含义是当我们在做polyfill填充的时候，在一些低版本不存在的特性的时候，并不会把全部加载，只是根据业务代码进行加载==》按需加载  

## 四、@babel/plugin-transform-runtime
看到一些资料说，引入的polyfill会污染全局变量，意思是别人如果使用了我们的类库，就可能造成这种问题，这里还未搞懂为什么会污染全局变量(暂且记下这个结论)，按我的理解，如果我们是开发自己的项目，不会对外提供，那么是不是就不需要去解决这个问题？

总之，babel为了解决这个**污染全局变量**以及**代码冗余**的问题，为我们提供了一个插件@babel/plugin-transform-runtime 
- npm install --save-dev @babel/plugin-transform-runtime
- npm install --save-dev @babel/runtime

添加到配置文件中代码如下：

```javascript
{
  "plugins": [
    ["@babel/plugin-transform-runtime",{
      "corejs":3
    }]
  ],
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns":"usage",
        "modules": false,
        "corejs":3
      }
    ]
  ]
}
```

## 五、babel的几个包  
最后我们总结一下有关babel的几个包 
- @babel/cli 
一般情况下，我们在项目中是不会安装@babel/cli这个包的，因为这个包的作用是：如果我们想在命令行使用才需要安装，即只是一个终端cli工具。   
- @babel/core 
这个是babel的核心包，拆分成了三个模块:parser/traverse/generator，如果需要以编程的方式来使用 Babel，可以使用 babel-core 这个包。 
若是想要用babel-loader把es6的代码转换成为es5的代码，那么需要对应版本的babel-core。  
- @babel/preset-env
预设：预先设定一组插件，方便我们对插件集成配置管理。    
- @babel/polyfill 
 为浏览器不支持的API提供对应的兼容性代码，很明显如果你不提供，如果在不支持的浏览器里面使用了新API，那么你的页面就会挂掉,@babel-polyfill带来的一些问题以及介绍后续再说。
- @babel/plugin-* 
babel插件机制、方便扩展、集成。   
- @babel/plugin-transform-runtime 
，解决转译语法层时出现的代码冗余、解决转译api层出现的全局变量污染 

## 六、小结
Babel 是一个工具链，主要用于将采用ECMAScript 2015+ 语法编写的代码转换为向后兼容的JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。
这里的重点是这个 + 号，差不多的理解为：  所以现在的项目有些不会使用polyfiil，但不会出现问题，是因为写的ES代码几乎都没有什么新的特性，浏览器差不多都支持了，为了保险起见后续新API的标准推出，有的项目才会装这个polyfill     吧。

