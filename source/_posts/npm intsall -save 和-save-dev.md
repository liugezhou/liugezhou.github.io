---
title: npm intsall -save 和-save-dev
date: 2018-06-28
categories:
- 前端小技
tags: 
- npm install
---
关于npm在安装的时候，有个小的知识点，一定要小小的补一下，那就是npm install 后面带参数的问题。
相信不少在开始接触node的时候，我们肯定都使用过这样的命令
`npm install -save` | `npm install -save-dev` | `npm install -g [--global]`
但是这些`-save` | `-save-dev` | `-g`的一些小小的区别是什么呢?
> <!-- more -->
###### npm install moduleName 命令
>1.安装模块到项目node_modules目录下。
>2.不会将模块依赖写入devDependencies或dependencies 节点。
>3.运行 npm install 初始化项目时不会下载模块。

###### npm install -g moduleName 命令
>1.安装模块到全局，不会在项目node_modules目录中保存模块包。
>2.不会将模块依赖写入devDependencies或dependencies 节点。
>3.运行 npm install 初始化项目时不会下载模块。

###### npm install -save moduleName 命令
>1.安装模块到项目node_modules目录下。
>2.会将模块依赖写入dependencies 节点。
>3.运行 npm install 初始化项目时，会将模块下载到项目目录下。
>4.运行npm install --production或者注明NODE_ENV变量值为production时，会自动下载模块到node_modules目录中。

###### npm install -save-dev moduleName 命令
>1.安装模块到项目node_modules目录下。
>2.会将模块依赖写入devDependencies 节点。
>3.运行 npm install 初始化项目时，会将模块下载到项目目录下。
>4.运行npm install --production或者注明NODE_ENV变量值为production时，不会自动下载模块到node_modules目录中。

总结
>devDependencies 节点下的模块是我们在开发时需要用的，比如项目中使用的 gulp ，压缩css、js的模块。这些模块在我们的项目部署后是不需要的，所以我们可以使用 -save-dev 的形式安装。像 express 这些模块是项目运行必备的，应该安装在 dependencies 节点下，所以我们应该使用 -save 的形式安装。
