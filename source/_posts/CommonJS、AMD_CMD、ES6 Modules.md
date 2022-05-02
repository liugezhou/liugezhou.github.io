---
layout: post
title: CMD、AMD、ES6NEXT
author: liugezhou
date: 2021-01-25 09:14:00
category: 
    前端小技
---

### CommonJS

---
> - CommonJS 一般用于服务端，Nodejs 与 webpack 就是 CommonJS 的主要实践者。
>
> - 四个重要的变量为模块化提供支持：module、exports、require、global。
>
> - 用 module.exports=value 或者 exports.xxx = value 来定义当前模块对外输出的接口，使用 require 加载模块。
>
> - 在 CommonJS 中，一个文件就是一个模块，每个文件拥有单独的作用域，不会污染全局作用域。
>
> - CommonJS 不适用浏览器是因为：此规范是同步加载模块，对于服务器端来说，所有的模块都是在本地磁盘，等待模块时间就是硬盘读取文件时间，很小，但对浏览器而言，设计到网速、代理等原因，同步加载会造成阻塞，浏览器处于“假死”状态，所以浏览器端出现了 AMD 规范。

###

### AMD(异步模块定义) & CMD(通用模块定义)
---
> - AMD 和 CMD 都是用来解决浏览器异步加载的问题。
>
> - AMD 是 Asynchronous Module Definition 的缩写，即“异步模块定义”，它采用异步方式加载模块(模块的加载不影响它后面语句的运行),且会提前加载。
>
> - AMD 同 CommonJS 一样也是使用 require 加载模块，不同的是，AMD 要求两个参数 require([Module],callback)
>
> - CMD 与 AMD 最大的不同就是：CMD 推崇依赖就近，延迟执行。可以在代码的任意一行写入依赖：

### ES6 Module
---

> ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，旨在成为浏览器和服务器通用的模块解决方案。其模块功能主要由两个命令构成：export 和 import.
>
> export 命令用于规定模块的对外接口，import 命令用于输入其他模块提供的功能。

> ES6 还提供了 export default 命令，为模块指定默认输出，对应的 import 语句不需要使用大括号.
