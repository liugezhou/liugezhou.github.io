---
layout: post
title: Babel
date: 2021-09-09
updated: 2022-05-04
description: 本文，抽丝剥茧，彻底的搞懂babel，并测试、去实践修改babe的配置。
toc: true
categories:
- 前端小技
tags: 
    - Babel
---
<font color=blue>更新说明：对文章排版以及内容格式做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

## 背景
> 项目中babel的设置一团遭，确实有必要搞懂这个babel。有的项目中直接在配置文件.babelrc中配置好，有的在main.js中全局import 这个polyfill，有的是在webpack中配置，有的引入了大量的第三方babel插件，这样可不行啊，如果不对babel有个全面的认知，在面对这些一系列满眼的配置下去修改，就有点力不从心。
> 本文，抽丝剥茧，彻底的搞懂babel，并测试、去实践修改babe的配置。

## 一、babel基础介绍
> 1. babel是什么？
> 
[Babel](https://babeljs.io/) 是一个广泛使用的 ES6 转码器，可以将 ES6 代码转为 ES5 代码，从而在老版本的浏览器执行。这意味着，你可以用 ES6 的方式编写程序，又不用担心现有环境是否支持。
> 
> 2. 使用第一步需要做什么？
> - 使用babel的第一步就是配置babel的配置文件。
> - 这个配置文件放在项目的根目录。
> - 这个配置文件可以是.babelrc，可以是 babel.config.js,也可以是babel.config.json
> - 这个配置文件是什么时候调用加载的❓
> 
> 3. 配置文件基本格式
> 
首先，我们先对babel的基本格式，存入心中：他是个对象、他的两个属性都是数组，他们分别是presets和plugins。

```javascript
{
  "presets": [],
  "plugins": []
}
```
> 4. presets和plugins是干嘛用的？


## 二、babel插件与预设
> 在继续上面的问题之前，这里我们需要深刻的晓得：babel的核心是**插件**，babel的所有工作都是由插件完成的。然后我们去查看babel的插件列表：[https://babeljs.io/docs/en/plugins-list](https://babeljs.io/docs/en/plugins-list)，部分截图如下
> ![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1631153922607-35bd71a6-dcaf-42bd-b2e9-336ef43dc495.png#clientId=u90972840-f7c3-4&from=paste&height=571&id=u72c58d17&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1712&originWidth=1118&originalType=binary&ratio=1&size=144065&status=done&style=none&taskId=u25964189-5d96-4681-96e9-5c4d670690a&width=372.6666666666667)
> 其中arrow-functions(也就是ES6的箭头函数)对应的插件是@babel/plugin-transform-arrow-functions
> 如果要使用这个插件，需要安装：npm install --save-dev @babel/plugin-transform-arrow-functions
> 这个插件作用就是将ES6的箭头函数转化为ES5写法。
> ![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1631154073171-462689b0-84b9-467c-9f58-e72e8d2b419a.png#clientId=u90972840-f7c3-4&from=paste&height=177&id=iomqr&margin=%5Bobject%20Object%5D&name=image.png&originHeight=530&originWidth=1726&originalType=binary&ratio=1&size=56423&status=done&style=none&taskId=ucbb483d7-f1f3-44aa-ae62-d4bfa378105&width=575.3333333333334)
> 
> 进入到这个方法的详情页，如上图，我们看到：**This plugin is included in @babel/preset-env**
> 也就是说这个插件以及其他各种单个插件都放在了@babel/preset-env这个包中。之所以要**included **到一个里面**，**是因为我们总不能一个一个插件的安装引入，官方就提供了一个包：'嘿，兄弟，我把一些你们都会用到的一系列的插件包在一块，你们就不用单个安装了，只需要执行下面的命令：
> npm install --save-dev @babel/preset-env'
> 
>  此时，这个preset-env(一系列插件的集合)，安装好后，我们就可以去配置文件里面配置了。
> 【这个presets叫做“预设”：预设可以看成是一组插件或者配置项的集合。】

```javascript
{
  "plugins": [],
  "presets": [
    "@babel/preset-env"
  ]
}
```
## 三、✨✨✨垫片-polyfill的使用
> 垫片就是我们平时早已知道的babel-polyfill。
> 为什么引入了babel还需要引入babel-polyfill呢，因为babel只能处理语法上的转化，而一些新的API他是没有办法处理的，比如Promise这些，这就需要babel-polyfill去处理这些新的API。babel-polyfill其实是corejs和regenerater的集合，现在已经被废弃了，只需要装上面两个就可以了。
> 
> 之前的项目中，我们的使用方式如下：
>  首先，我们需要安装它：
> - npm install @babel/polyfill -S
> 
关于项目中使用polyfill，有好多种写法，我们这里需要对他们的每个写法都了然于胸，才会 知道如何真正优化配置。

**type1:**
> 在 node/webpack/rollup中使用,可能需要在入口文件最上层引入：
> - require('@babel/polyfill') 
> - import '@babel/polyfill"

**type2:**
> 在webpack中可以通过配置文件加入：
> module.exports = {
>   entry: ["@babel/polyfill", "./app/js"],
> };

**type3:**
> babelrc配置文件中搭配@babel/presets-env使用，这样就不需要引入polyfill了。
> - 设置modules为false，这只是一个摆设？
> - useBuiltsIns这个设置也没搞明白！按需加载吗？

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
**type4：**
> 在浏览器环境中使用scripts引入，不推荐使用。


## 四、@babel/plugin-transform-runtime
> 看到一些资料说，引入的polyfill会污染全局变量，意思是别人如果使用了我们的类库，就可能造成这种问题，这里还未搞懂为什么会污染全局变量(暂且记下这个结论)，按我的理解，如果我们是开发自己的项目，不会对外提供，那么是不是就不需要去解决这个问题？
> 
> 总之，babel为了解决这个污染全局变量的问题，为我们提供了一个插件@babel/plugin-transform-runtime 
> - npm install --save-dev @babel/plugin-transform-runtime
> - npm install --save-dev @babel/runtime
> 
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

## 五、小结
> Babel 是一个工具链，主要用于将采用ECMAScript 2015+ 语法编写的代码转换为向后兼容的JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。
> 
> 
> 这里的重点是这个 + 号，差不多的理解为：  所以现在的项目有些不会使用polyfiil，但不会出现问题，是因为写的ES代码几乎都没有什么新的特性，浏览器差不多都支持了，为了保险起见后续新API的标准推出，有的项目才会装这个polyfill     吧？




