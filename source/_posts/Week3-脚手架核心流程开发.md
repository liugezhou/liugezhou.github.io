---
layout: post
title: Week3-脚手架核心流程开发
date: 2021-01-25
updated: 2022-05-04
description: 脚手架核心流程开发
toc: true
category: 
    Web架构之脚手架
tags:
    Web架构之脚手架
---

<font color=blue>更新说明：对文章目录排版做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

## 第一章：本周导学
**本周整体内容介绍和学习方法**
**标题**
> 脚手架需求分析和架构设计，核心流程开发

**收获**
> - 架构设计和技术方案设计全过程
> - 脚手架核心流程和commander框架
> - 如何让Node项目支持ES Module

**主要内容**
> - 脚手架需求分析和架构设计
> - 脚手架模块拆分策略和core模块技术方案
> - 脚手架执行准备过程实现
> - 脚手架命令注册实现(基于commander)

**加餐**
> Node项目如何支持ES Module

## 第二章：脚手架整体架构设计

**2-1 大厂是如何做项目的**
![3-1](https://cdn.jsdelivr.net/gh/liugezhou/image@master/imooc-course/3-1.6id7710dpwk0.webp)

**2-2 前端研发过程中的痛点和需求分析**
![3-2](https://cdn.jsdelivr.net/gh/liugezhou/image@master/imooc-course/3-2.6ic0nprs1e00.webp)

**2-3 加餐：大厂的git操作规范是怎样的？**

> - clone下来的项目 master分支是不做开发的，我们会新建一个dev分支，上线以后会新建一个release分支。

![3-3](https://cdn.jsdelivr.net/gh/liugezhou/image@master/imooc-course/3-3.4sujpjgc1p40.webp)

**2-4 高端操作：脚手架架构设计+绘制架构图 /  2-5 架构设计图绘图技巧分享**

![3-4](https://cdn.jsdelivr.net/gh/liugezhou/image@master/imooc-course/3-4.3xuy7m03daw0.webp)

## 第三章 脚手架模块拆分策略和core模块技术方案

**3-1 脚手架模块拆分策略**

- 核心流程：core
- 命令：      commands  【初始化、发布、清除缓存】
- 模型层：   models       【Command命令 、 Project项目 、 Component组件 、 Npm模块 、 Git仓库】
- 支撑模块：utils           【Git操作 、 云构建 、 工具方法 、 API请求、 Git API】

**3-2 core模块技术方案**

准备阶段：
![3-5](https://cdn.jsdelivr.net/gh/liugezhou/image@master/imooc-course/3-5.1i5lq7mn8h5s.webp)

## 第四章：脚手架执行准备过程实现

**4-1 脚手架框架代码拆包+import-local应用**
> - mkdir cloudscope-cli
> - cd cloudscope-cli
> - npm init -y
> - 添加 .gitignore文件
> - lerna init 
> - lerna create cli & lerna create utils
> - mkdir  commands & mkdir models & mkdir core  & mkdir utils
> - 将packages下core移到外层core、utils移到外层utils,删除packages目录
> - 修改lerna.json文件：

```javascript
{
  "packages": [
    "core/*",
    "commands/*",
    "models/*",
    "utils/*"
  ],
  "version": "1.0.4"
}

```

- 在cloudscope-cli/core/cli下新建bin/index.js文件
- 修改  cloudscope-cli/core/cli下的package.json配置文件，添加如下代码
```javascript
……
 "bin": {
    "cloudscope-cli": "bin/index.js"
  },
    
 ……
 "dependencies": {
    "@cloudscope-cli/utils": "file:../../utils/utils",
    "import-local": "^3.0.2",
    "npmlog": "^4.1.2"
  },
```
> - cd core/cli
> - npm install
> - 在cloudscope-cli/core/cli/bin/index.js下添加如下代码：

```javascript
#! /usr/bin/env node

const importLocal = require('import-local')

if(importLocal(__filename)){
  require('npmlog').info('cli','正在使用 cloudscope-cli 本地版本')
}else{
  require('../lib')(process.argv.slice(2))
}
```

> - 在cloudscope-cli/core/cli/lib/index.js 文件中添加一行日志文件
> - npm link
> - cloudscope-cli:即可看到输出日志

git代码提交扩展
> 我将此代码通过分支的形式，对每一次的代码提交，都在一个特殊的分支进行代码提交。
> 本节提交到该分支：[ lesson01](https://github.com/liugezhou/cloudscope-cli/tree/lesson01)
> 
> 下面是分支提交小记录。
> - git checkout -b lesson01
> - git add . && git commit -m 'lesson01 for init ' && git push origin lesson01
> - git checkout main
> - git merge lesson01
> - git push
> - 删除本地分支  git branch -D lesson01
> 
> - 如果后面我们想要修改该分支代码并提交到该分支，我们就可以：
> - git checkout -b temp origin/lesson01

**4-2 检查版本号功能开发（require加载资源类型讲解+npmlog封装)**

从本节开始本地新建分支 lesson02，最后将此分支代码推送至远程仓库[cloudscope-cli](https://github.com/liugezhou/cloudscope-cli)。

1. 本节代码开发过程中在命令行用到的命令：
> - utils下新建log包： lerna create @cloudscope-cli/log utils
> - log下安装npmlog包： lerna add npmjs utils/log 

2. 其它记录的小知识点
- require加载资源的类型：
> - .js      ： 必须输出一个module.exports || exports. 
> - .json   :  会通过JSON.parse()对该文件进行解析，并输出一个对象
> - .node  ： c++的一个插件，它的实现原理是通过 process.dlopen去打开一个c++插件，通常不使用。
> - any     ：会通过.js引擎进行解析

3. npmlog源码：
> - default level:  info，可以通过传入的参数进行level定制。
> - 源码中存在的 log.addLevel()才可以打印，我们也可以通过这个方法定制自己的打印设置。（可以设置名字、value值大小，字体色、背景色等）
> - 可以通过 log.heading定制输日志输出的前置。同时可以通过headingStyle做样式修改。

**4-3 最低Node版本检查功能开发**

检查Node版本号的原因以及解决办法：
> - 这是因为一些低版本的Node API在低版本是不支持的，因此要设置一个最低的Node版本号。
> - 拿到本地版本号的方法为：process.version
> - 版本号比对：第三方库 semver。
> - 抛出异常颜色输出：第三方库 colors:引用'colors/safe',使用：colors.red('')

**4-4 root账号启动检查和自动降级功能开发**

检查账号权限原因以及解决方法：
> - 如果是使用root权限，一些文件就没有可读或者修改权限，因此需要对用户进行查询与降级处理
> - 通过process.geteuid() 获取登录用户的ID ,501为普通用户，0 为超级管理员(root)
> - 检查第三方库：root-check。使用方法引入一下调用即可降级。
> - root-check实现原理：调用downgrade-root 库 -> 判断是否为root权限 -> 若是通过process.env.SUDO_UID或者默认 defaultUid() 获取各个操作系统的uid

**4-5 用户主目录检查功能开发**

> - user-home:可以实现跨操作系统获取用户主目录的功能。
> - path-exists:判断文件目录是否存在

> - user-home实现：调用os-homedir库，再调用os库，若os库有homedir直接返回，若没有直接拿process.env.home(),还是没有就拼接 ‘/Users/'+process.env.USER
> - path-exists实现：直接调用fs的accessSync(path)方法。

**4-6 入参检查和debug模式开发**

> 这里就进行如参检查，是要判断是否进入调试模式，如果带有 --debug参数，我们要进行log的level设置。

> - 实现方式：使用minimist第三方库。
> - 查看是否包含debug参数，直接：require('minimist')(process.argv.slice(2)).debug即可。
> - 若上值为true，直接修改log.level即可。

**4-7 环境变量检查功能开发**

> - 检查环境变量，我们使用第三方库：dotenv。
> - 用法：require('dotenv').config({ path: '' }) ：若不传参数，我们在当前目录下拿到.env文件中的变量，之后就可以直接在process.env中使用了。支持传入path变量。
> - 环境变量其实就是一个全局变量,如果我们有很多的环境变量需要使用，可以直接在.env文件宏进行配置

**4-8 通用npm API模块封装 | **4-9 npm全局更新功能开发****
> 准备阶段的最后一个功能：检查我们的这个脚手架是否为最新版本

步骤：
> 1. 获取当前版本号与模块名: pkg.version | pkg.name
> 1. 调用npm API获取所有模版号：
> 
npm提供了这样一个API: https://registry.npmjs.org/cloudscope-cli/core ,可以获得该包的所有版本号,要从这里拿到所有版本号，我们需要使用第三方库 axios,同时我们也需要添加一个用来url拼接的库：url-join，可以帮助我们进行多参数的拼接，以及我们进行版本对比的第三方库 semver。
> 3. 获取所有版本号，比对哪些版本号是大于当前版本号
> 3. 获取最新的版本号，提示用户更新到此版本。

将以上代码提交支仓库远程cloudscope-cli的分支 [lesson02](https://github.com/liugezhou/cloudscope-cli/tree/lesson02)，并合并至main分支。
**
## 第五章：脚手架命令注册实现(基于commander）
**5-1 快速实现一个commander脚手架 ｜ 5-2 commander脚手架全局配置**
> 之前在学习命令注册的时候，使用的是yrags，本节使用另一个库 commander去实现命令注册
> 本节代码提交至：[liugezhou-yargs-demo](https://github.com/liugezhou/liugezhou-yargs-demo)
> 其中 bin/yargs.js是之前学习yargs的demo代码。
> bin/commander.js是本节关于commander的代码。
> 
> 我们在这个库的基础上，学习commander的简单用法.
> - 首先，安装npm i -S commander
> - 然后，在bin/index.js中：

```javascript
#!/usr/bin/env node

const commander = require('commander')
const pkg = require("../package.json")

// 获取commander的单例
// const {program} = commander

// 手动实例化一个Command示例
const program = new commander.Command()

program
  .name(Object.keys(pkg.bin)[0])
  .usage('<command> [options]')
  .version(pkg.version)
  .option('-d, --debug', '是否开启调试模式', false)
  .option('-e, --envName  <envName>' , '环境变量')
  .parse(process.argv);

  const options = program.opts()
  console.log(options.debug)
  program.outputHelp()   //打印出帮助信息
```
> - liugezhou-test -V,即可看到输入版本号。
> - liugezhou-test -d  // true
**5-3 commander脚手架命令注册的两种方法**

课程所讲内容：commander命令注册有两种方式：
> 1. comman API注册命令
> 1. addCommand API 注册命令

现在默认安装commander时，已更新到7.0.0，sam老师写法还是6.X，可参考 [commader for git](https://github.com/tj/commander.js/blob/HEAD/Readme_zh-CN.md) 配置。本节内容就是对官方文档更新用法之后的学习笔记：

1. Commander.js :
> 完整的node.js命令行解决方法

2. 声明commander变量
> 为简化声明，Commander提供了一个全局对象

```javascript
const { program } = require('commander')
program.version('0.0.1')
```
> 若程序较为复杂，用户需要以多种方式来使用Commander，如单元测试等，可采用创建本地Commander对象的方法

```javascript
const {Command } = require('commander')
const program = new Command();
program.version('0.0.1')
```

3. 选项
```javascript
program
//Commander 使用.option() 方法来定义选项，同时可以附加选项的简介
	.option('-d,--debug','booelan') 
//选项可以设置一个默认值。
	.option('-c, --cheese <type>', 'add the specified type of cheese', 'blue');
//必填选项
	.requiredOption('-n, --name <type>', 'name must have cheese');
//通过program.parse(arguments)方法处理参数
program.parse(process.argv)
const ret = program.opts()
const debug = ret.debug
```

4. 命令
```javascript
program.
//参数支持必须<>,可选[]
	command('clone <source> [destination]')

//在参数名后加上...来声明可变参数，且只有最后一个参数支持这种用法
program
    .command('rmdir <dirs...>')
    .action(function (dirs) {
      dirs.forEach((dir) => {
        console.log('rmdir %s', dir);
      });
    });
```
**5-4 commander注册命令的两种高级用法 ｜5-5 再讲3条commander的高级用法**

> 1. 监听所有命令输入 : arguments
> 1. 脚手架串行使用
```javascript
   program
      .command('install [name]','install package',{
        executableFile:'vue'，
     		isDefault:true,
 	      hidden:true //command的隐藏
      })
      .alias('i')
```
> 3. 高级定制help信息
> -  program._outputHelp()_
> - _ program.helpInformation() = function(){}_
> - _program.on('--help'),function(){ console.log('your help information'}_
> 4. 高级定制debug模式

```javascript
program.on('optins:debug',function(){
  if(program.debug){
  process.env.LOG_LEVEL = 'verbose'
  }
}
```
> 5. 对未知命令监听二：
```javascript
program.on('command:*',function(obj){
      console.log(obj)
      console.log('未知的命令：' + obj[0])
      const avaliableCommands = program.commands.map( cmd => cmd.name())
      console.log('可用命令：'+ avaliableCommands.join(','))
  })
```
## 第六章 Node项目如何支持ES Module【加餐】

**6-1 通过webpack完成ES Module资源构建**
**模块化**
> - CMD/AMD/require.js
> - CommonJS: 加载：require(), 输出：module.exports() || exports.x
> - ES Module: 加载： import, 输出 export.default || export function || export const


**实现**
> - npm i -D webpack webpack-cli
> - touch webpack-config.js

```javascript
//webpack-config.js
const path = require('path')
module.exports = {
  entry: './bin/core.js',
  output:{
    path:path.join(__dirname,'/dist'),
    filename:'core.js'
  },
  mode:'development'
}

// index.js
#! /usr/bin/env node

require('./core')


// core.js
import utils from './utils'

utils()

// utils
module.exports = function(){
  console.log('kskaksk')
}
```
> - 在package.json中添加两个script，分别为 "build":"webpack"和"dev":"webpack -w"
> - 上面代码通过 npm run build 打包后，将上面index.js中的require修改为 require("../dist/core.js")
> - 执行 liugezhou-test 看到构建成功。

**6-2 通过webpack target属性支持Node内置库**

- webpack的target使用
```javascript
// npm i -S path-exists
// bin/utils.js
import pathExists from 'path-exists'
export function exists(p){
  return pathExists.sync(p)
}

// bin/core.js
import path from 'path'
import {exists} from './utils'

console.log(path.resolve('.'))
console.log(exists(path.resolve('.')))

// webpack.config.js
...
	target:"node"
```

- core.js代码添加
```javascript
(async function(){
    await new Promise(resolve => setTimeout( resolve,1000));
    console.log('ok')
})()
```
**6-3 webpack loader配置babel-loader支持低版本node**

> 配置一个最简单babel-loader,需要安装的库
> npm i -D 
> babel-loader @babel/core  @babel/preset-env  @babel/plugin-transform-runtime  @babel/runtime-corejs3

```javascript
const path = require('path')
module.exports = {
  entry:'./bin/core.js',
  output:{
    path: path.join(__dirname,'/dist'),
    filename:'core.js'
  },
  mode:'development', //开发模式
  // target: 'web'//默认
  target:'node', // 识别内置库
  module:{
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets:['@babel/preset-env'],
            plugins: [
              [
                '@babel/plugin-transform-runtime',
                {
                  corejs:3,
                  regenerator:true,
                  useESModules:true,
                  helpers: true
                }
              ]
            ]
          }
        }
      }
    ]
  }
}
```
**6-4 通过Node原生支持ES Module**

将node版本升级到14.x，代码中将引用的文件，改写后缀名为 .mjs即可。




