---
layout: post
title: week30-脚手架发布模块云构建系统开发
author: liugezhou
date: 2021-07-10
category: 
    Web架构之脚手架
tags:
    Web架构之脚手架
---
### 第一章 本周导学


---

#### 1-1 本周整体内容介绍和学习方法
> - **云构建**原理、架构和实现
> - **WebSocket**入门到实战
> - **Redis**入门实战

### 
### 第二章 云架构模块架构设计

---


#### 2-1 详细分析为什么需要设计云构建系统


**为什么需要云构建**
> - 减少发布过程中的重复劳动
>    - 打包构建
>    - 上传静态资源服务器
>    - 上传CDN
> - 避免不同环境造成的差异
> - 提升构建性能
> - **对构建过程进行统一管控**
>    - 发布前代码统一规则检查
>    - 封网日统一发布卡口


#### 2-2 云构建系统架构设计
[点击查看【processon】](https://www.processon.com/embed/61127e8907912940a9654a57)

### 第三章 WebSocket 快速入门

---


#### 3-1 WebSocket基本概念及同HTTP协议对比
> WebSocket概念
> - HTTP：请求响应的单向。
> - WebSocket：只需发起一次请求，双向发起请求，双向接收响应。常用为聊天工具、云构建请求。
> - 客户端开发WebSocket与浏览器开发WebSocket是不同的。
> - 如何通过NodeJs搭建一个WebSocket服务。

相关基本概念：[https://www.runoob.com/html/html5-websocket.html](https://www.runoob.com/html/html5-websocket.html)

#### 3-2 egg集成WebSocket服务

**基础介绍**
> 基础教程：[https://eggjs.org/zh-cn/tutorials/socketio.html](https://eggjs.org/zh-cn/tutorials/socketio.html)
> demo仓库：[https://github.com/eggjs/egg-socket.io/tree/master/example](https://github.com/eggjs/egg-socket.io/tree/master/example)


> 代码在lesson01分支基础上继续开发:[cloudscope-cli-server/leeson01](https://github.com/liugezhou/cloudscope-cli-server/tree/lesson01)
> 在lesson01分支基础上，新建分支 lesson30，本周最终代码在[lesson30](https://github.com/liugezhou/cloudscope-cli-server/tree/lesson30)分支上。

**WebSocket服务开发流程**
> 1. 安装依赖
> - cnpm i -S egg-socket.io
> 2. 更新配置文件

```javascript
// config.default.js
config.io = {
  namespace: {
    '/': {
      connectionMiddleware: ['auth'],
      packetMiddleware: ['filter'],
    },
    '/chat': {
      connectionMiddleware: ['auth'],
      packetMiddleware: [],
    },
  },
};

// plugin.js
exports.io = {
  enable: true,
  package: 'egg-socket.io',
};

```
> 3. 修改路由配置

```javascript
// router.js

// app.io.of('/')
app.io.route('chat', app.io.controller.chat.index);

// app.io.of('/chat')
app.io.of('/chat').route('chat', app.io.controller.chat.index);

```
> 4. 开发middleware

```javascript
// app/io/middleware/auth.js
'use strict';

module.exports = () => {
  return async (ctx, next) => {
    const say = await ctx.service.user.say();
    ctx.socket.emit('res', 'auth!' + say);
    await next();
    console.log('disconnect!');
  };
};

//app/io/middleware/filter.js
'use strict';

module.exports = () => {
  return async (ctx, next) => {
    console.log(ctx.packet);
    const say = await ctx.service.user.say();
    ctx.socket.emit('res', 'packet!' + say);
    await next();
    console.log('packet response!');
  };
};
```
> 5. 开发controller

```javascript
// app/io/controller/chat.js
'use strict';

module.exports = app => {
  class Controller extends app.Controller {
    async index() {
      const message = this.ctx.args[0];
      console.log('chat :', message + ' : ' + process.pid);
      const say = await this.ctx.service.user.say();
      this.ctx.socket.emit('res', say);
    }
  }

  return Controller;
};

```

#### 3-3 WebSocket客户端开发
> 代码分支在 cloudscope-cli/lesson30上。
> - lerna create cloudbuild models/
> - cd models/cloudbuild
> - npm i -S socket.io-client
> - cloudbuild/lib/index.js

```javascript
'use strict';

// or http://127.0.0.1:7001/chat
const socket = require('socket.io-client')('http://127.0.0.1:7001');

socket.on('connect', () => {
  console.log('connect!');
  socket.emit('chat', 'hello world!');
});

socket.on('res', msg => {
  console.log('res from server: %s!', msg);
}); 
```

#### 3-4 WebSocket客户端与服务端交互流程分析

以日志的打出分析流程
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1628635057269-41f4e637-7c4f-4b00-aeae-d13e3959deb4.png#clientId=uf661c060-e03a-4&from=paste&height=115&id=ue8dd3dff&margin=%5Bobject%20Object%5D&name=image.png&originHeight=346&originWidth=1256&originalType=binary&ratio=1&size=41485&status=done&style=none&taskId=u4167e09a-1bcd-4394-b772-fdfed654da3&width=418.6666666666667)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1628635078101-6b51c68f-8ad1-460d-aabf-b5e30fc2569f.png#clientId=uf661c060-e03a-4&from=paste&height=90&id=u62864eef&margin=%5Bobject%20Object%5D&name=image.png&originHeight=270&originWidth=1628&originalType=binary&ratio=1&size=57699&status=done&style=none&taskId=u8f5317b2-fb1d-46eb-a800-ac81356d35c&width=542.6666666666666)
> - 首先服务端(cloudscope-cli-server)启动服务:**npm run dev**
> - 客户端启动服务：**node models/cloudbuild/lin/index.js**
> - 客户端启动后：
>    - socket on 连接成功，监控connect事件：打印出日志** connect！**
>    - 接着客户端emit chat事件：**socket.emit('chat', 'hello world!')**;
> - 服务端接收到chat事件后
>    - 首先会现在服务端config.default.js中找到 / 的 connectMiddleWare的  auth.js去执行
>       - auth.js中触发res事件
>       - 客户端监听res事件，打印日志：res from server:auth! Hello Man!!
>    - 接着服务端在config.default.js中 找到 / 的packetMiddleWare 的 filter.js去执行
>       - 服务端打印 ctx.packet日志：['chat','hello world!']
>       - filter.js触发res事件
>       - 客户端监听res事件，打印日志：res from server:packet! Hello Man!!
>    - 服务端通过route匹配到 chat，于是去Controller中找 chat.js
>       - 服务端打印日志：chat:hello world! : 10826
>       - 接着服务端调用service服务，拿到值，触发 res事件
>       - 客户端监听到res事件，打印日志：res from server:Hello Man!!
>    - 服务端等待 next执行完毕后，最后在 filter.js中，打印出：packet response！


### 第四章 Redis 快速入门

---


#### 4-1 redis基本概念+安装方法+基本命令
> - 该项目应用redis是要：存储任务信息
> - redis安装
> - 常用命令redis-cli[进入终端服务]和redis-server[启动redis服务]


#### 4-2 阿里云redis服务配置和远程连接方法讲解
我这里实在腾讯云领了一个月的redis免费试用版本,下面记录为课程的讲解，腾讯云相关redis见读书笔记。
> - 购买完数据库后，第一个设置是白名单设置，0.0.0.0/0 如果不设置，会出现远程无法连接的问题
> - 创建账号：使用默认账号或创建账号连接
> - 连接成功后 AUTH <password>

#### 4-3 egg集成redis方法讲解
redis为使用本地
> - 首先在npm官网上查看 egg-redis这个插件
> - 在server安装：npm i egg-redis  --save
> - 根据npm官网上关于egg-redis的代码讲解，分别在plugin.js和config.default.js中添加相关代码。
> - 添加一个新路由：/redis/test，并在project的controller中测试
> - 添加 await app.redis.get(key)获取key值

```javascript
async getRedis(){
  const { ctx, app } = this;
  const num = await app.redis.get('number')
  console.log(num)
  ctx.body = 'hello redis'
}


//config.defualt.js
 config.redis = {
   client: {
     port: REDIS_PORT,
     host: REDIS_HOST, 
     password:REDIS_PWD,
     db: 0,
   },
   
   //plugin.js
   exports.redis = {
   enable: true,
   package: 'egg-redis',
 	};
```
### 
### 第五章 云构建初始化流程开发

---

#### 5-1 CloudBuild类开发
> 根据第二章 架构图，本节主要代码为 CloudBuild类的创建与引用，最终传入git对象

> 3-3节已经创建了cloudbuild，修改这里的代码为

```javascript
'use strict';

class CloudBuild {
  constructor(git, props){
      console.log('cloudbuild',git)
  }
  // or http://127.0.0.1:7001/chat
// const socket = require('socket.io-client')('http://127.0.0.1:7001');

// socket.on('connect', () => {
//   console.log('connect!');
//   socket.emit('chat', 'hello world!');
// });

// socket.on('res', msg => {
//   console.log('res from server: %s!', msg);
// }); 
}
 module.exports = CloudBuild
```

> 在modes/git下引入@cloudscope-cli/cloudbuild，并新建publish方法 

```javascript
async publish(){
  await this.preparePublish()
  const buildCmd = ''
  const cloudBuild = new CloudBuild(this,{
    buildCmd
  })
  }

async preparePublish(){
  console.log('preparePublish')
}
```
> 在commands/publish/lib/index.js 文件中，接着上一周的 自动化提交代码，接着调用 git.publsih方法。
> - 这样从 commands/publish/lib/index.js中调用 git.publish()方法
> - 在 models/git/lib/index.js中开发publish方法，publish中会生成一个cloudbuild实例
> - 这个cloudbild实例为我们在models下新建的一个包，这样本节就形成了一个闭环。
> - 下节开始就是cloudbuild实例的开发，以及publish流程。


#### 5-2 生成构建命令+构建命令检查开发
> 本节主要内容是，用于定制build命令，通过 --buildCmd参数，如果用户传入build命令，那么使用传入的build命令打包，如果不是则传入默认的打包命令。
> 1. 首先在 core/cli/lib/index.js中传入 --buildCmd参数

```javascript
program
  .command('publish')
  .option('--refreshServer','强制更新远程Git仓库类型和token')
  .option('--refreshOwner','强制更新远程Git仓库用户类型')
  .option('--buildCmd <buildCmd>','构建命令')
  .action(exec)
```
> 2. 然后在commands/publish/lib/index.js中接收这个参数

```javascript
class PublishCommand extends Command {
    init(){
        // 处理参数
        log.verbose('publish init',this._argv)
            this.options = {
                refreshServer:this._argv[0].refreshServer,
                refreshOwner:this._argv[0].refreshOwner,
                buildCmd:this._argv[0].buildCmd 
            }
    }
  …………
}
```
> 3. 此时Git类接收并赋值该参数，然后在上节我们写的 **[models/git/lib/index.js]** publishPrepare中见检查这个命令

```javascript
async preparePublish(){
  if(this.buildCmd){
    const buildCmdArray = this.buildCmd.split(' ')
    if(!Object.is(buildCmdArray[0],'npm') && !Object.is(buildCmdArray[0],'cnpm')){
      throw new Error('Build命令非法，必须使用npm或cnpm！')
    }
  }else{
    this.buildCmd = 'npm run build'
  }
  console.log(this.buildCmd)
}
```
> - 最后测试一下：
> 
终端命令：
> **cloudscope-cli publish --targetPath /Users/liumingzhou/Documents/imoocCourse/Web前端架构师/cloudscope-cli/commands/publish --buildCmd 'anpm run build:prod'**
> **抛出异常：**Build命令非法，必须使用npm或cnpm



#### 5-3 通过CloudBuild创建WebSocket连接
> 我们在第三章的学习当中已经大略的知道了 前后端如何建立起socket连接，本节就是对服务端代码修改以及客户端代码开发-传递git.repo参数到服务端。
> 
> - 服务端
>    - 将与service有关的代码先删掉
>    - 将routes.js中与chat相关的修改为 build：app.io.route('build', app.io.controller.build.index);
>    - 服务端学习的核心代码为：const query = socket.handshake.query

```javascript
//app/controller/io/controller/build.js
'use strict';

module.exports = app => {
  class Controller extends app.Controller {
    async index() {
      console.log('Controller build!')
    }
  }
  return Controller;
};

//app/controller/io/middleware/auth.js

'use strict';

module.exports = () => {
  return async (ctx, next) => {
    const {socket,logger} = ctx
    const query = socket.handshake.query
    logger.info(query)
    console.log('connect!')
    await next();
    console.log('disconnect!');
  };
};
```
> - **客户端**
>    - **Cloudbuild实例的init方法传入了一个参数：git.remote**

```javascript
// models/cloudbuild/lib/index.js
'use strict';

const io = require('socket.io-client');
const TIME_OUT = 5* 60
const WS_SERVER = 'http://liugezhou.com:7001'
class CloudBuild {
  constructor(git, options){
     this.git = git
     this.buildCmd = options.buildCmd
     this.timeout = TIME_OUT
  }

  init(){
    const socket = io(WS_SERVER,{
      query:{
        repo:this.git.remote
      }
    })
    socket.on('connect', () => {
    console.log('connect!');
    // socket.emit('chat', 'hello world!');
  });
  }
}
```
> **最后测试一下：**
> - 最后测试一下，将服务端启动，在客户端执行命令：
> 
终端命令：
> **cloudscope-cli publish --targetPath /Users/liumingzhou/Documents/imoocCourse/Web前端架构师/cloudscope-cli/commands/publish --buildCmd 'npm run build:prod'**
> - 在客户端与服务端分别打印出日志，测试正确

#### 
#### 5-4 WebSocket超时自动断开连接逻辑开发
> 本节比较简单，是解决连接不上服务端的问题--连接超时，抛出异常，部分代码如下：

```javascript

const CONNET_TIME_OUT = 5*1000
class CloudBuild {
  constructor(git, options){
    this.git = git
     this.buildCmd = options.buildCmd
     this.timeout = TIME_OUT
  }

  doTtimeout(fn,timeout){
    this.timer && clearTimeout(this.timer)
    log.info('设置任务超时时间：',`${timeout/1000}秒`)
    this.timer = setTimeout(fn,timeout);
  }
  init(){
    const socket = io(WS_SERVER,{
      query:{
        repo:this.git.remote
      }
    })
    const disconnect = ()=>{
      clearTimeout(this.timer)
      socket.disconnect()
      socket.close()
    }
   this.doTtimeout(()=>{
    log.error('云构建服务连接超时，自动终止')    
    disconnect()
    }, CONNET_TIME_OUT);
   
  }
}

```
#### 5-5 WebSocket客户端和服务端通信优化

> 在整个websocket连接成功之后，是可以发送一些标准化日志的。
> - 友好实现socket连接后的标准化日志，服务端改造
>    - 第一个改造点，cloudscope-cli-server中auth.js添加try catch异常捕获
>    - 第二个改造点：新建app/extend/helper.js，统一的返回格式

```javascript
// app/extend/helper.js
'use strict';

module.exports = {
    parseMsg(action,payload={},metadata={}){
        const meta = Object.assign({},{
            timestamp:Date.now()
        },metadata);

        return {
            meta,
            data:{
                action,
                payload
            }
        }
    }
}

// app/controller/io/middleware/auth.js

const {socket,logger,helper} = ctx
socket.emit(id,helper.parseMsg('connect',{
  type:'connect',
  message:'云构建服务连接成功'
}))
```

客户端
```javascript
const get =require('lodash/get')

function parseMsg(msg){
  const action = get(msg,'data.action')
  const message = get(msg,'data.payload.message')
  return {
    action,
    message
  }
}


socket.on('connect', () => {
  clearTimeout(this.timer)
  const {id} = socket
  socket.on(id,msg=>{
    const parsedMsg = parseMsg(msg)
    log.success(parsedMsg.action,parsedMsg.message,`任务ID：${id}`)
  })
});

socket.on('disconnect',()=>{
  log.success('disconnect','云构建任务断开')
  disconnect();
})

socket.on('error',(err)=>{
  log.error('error','云构建错误',err)
  disconnect()
})
```

#### 5-6 云构建任务写入Redis
> 本节主要内容就是将云构建任务写入到redis中去【服务端】,核心代码如下：

```javascript
// app/controller/io/middleware/auth.js
const REDIS_PREFIX = 'cloudbuild'

const {app} = ctx
const {redis} = app
// 判断redis任务是否存在
let hashTask = await redis.get(`${REDIS_PREFIX}:${id}`)
if(!hashTask){
  await redis.set(`${REDIS_PREFIX}:${id}`,JSON.stringify(query))
}
hashTask = await redis.get(`${REDIS_PREFIX}:${id}`)
logger.info(hashTask)

```

#### 5-7 创建云构建任务功能开发
客户端添加build方法
```javascript
// models/cloudbuild/lib/index.js   
	build(){
    return new Promise((resolve,reject)=>{
      this.socket.emit('build')
      this.socket.on("build",msg=>{
        const parsedMsg = parseMsg(msg)
        log.success(parsedMsg.action,parsedMsg.message)
      })
      this.socket.on('building',msg=>{
        const parsedMsg = parseMsg(msg)
        log.success(parsedMsg.action,parsedMsg.message)
      })
      
 //commands/publish/lib/index.js
   //publish方法中 
      await cloudBuild.init()
      await cloudBuild.build()
```
在服务端，我们在已经建好的 app/controller/io/controller/build.js中
```javascript
'use strict';

const REDIS_PREFIX = 'cloudbuild'
const CloudBuildTask = require('../../models/CloudBuildTask')
async function createCloudBuildTask(ctx,app){
  const { socket,helper } = ctx
  const { redis } = app
  const client = socket.id
  const redisKey = `${REDIS_PREFIX}:${client}`
  const redisTask = await redis.get(redisKey)
  const task = JSON.parse(redisTask)
  socket.emit('build',helper.parseMsg('create task',{
    message:'创建云构建任务'
  }))
  return new CloudBuildTask({
    repo:task.repo,
    name:task.name,
    version:task.version,
    branch:task.branch,
    buildCmd:task.buildCmd
  }) 
}
module.exports = app => {
  class Controller extends app.Controller {
    async index() {
      //创建云构建任务
      const {ctx,app} = this
      const cloudBuildTask = await createCloudBuildTask(ctx,app)
    }
  }
  return Controller;
};

// app/models/CloudBuildTask.js
'use strict';

class CloudBuildTask {
    constructor(options){
       console.log(options)
    }
}

module.exports = CloudBuildTask
```

测试，可在服务端打印出options。

### 第六章 云构建执行流程开发

---

#### 6-1 云构建任务初始化流程开发
> 服务端云构建的初始化流程，主要内容为CloudBuildTask这个类
> - npm i -S user-home simple-git fs-extra

```javascript
// 注意CloudBuildTask类需要传入的第二个参数为ctx
'use strict';

const path = require('path')
const fse = require('fs-extra')
const userHome = require('user-home')
const Git = require('simple-git')
class CloudBuildTask {
    constructor(options,ctx){
        this._ctx = ctx
        this._name = options.name //项目名称
        this._version = options.version //项目版本号
        this._repo = options.repo
        this._branch = options.branch 
        this._buildCmd = options.buildCmd //构建命令
        this.logger = this._ctx .logger
        // 服务器的用户主目录
       this._dir = path.resolve(userHome,'.cloudscope-cli','cloudbuild',`${this._name}@${this._version}`) //缓存目录
       this._sourceCodeDir = path.resolve(this._dir,this._name) //缓存源码目录
       this.logger.info('_dir',this._dir)
       this.logger.info('_sourceCodeDir',this._sourceCodeDir)
    }

    async prepare(){
        fse.ensureDirSync(this._dir)
        fse.emptyDirSync(this._dir)
        this._git = new Git(this._dir)

    }
}

module.exports = CloudBuildTask
```

#### 6-2 云构建任务交互日志开发
> 本节在CloudBuildTask类中，还未进行开发前，现对错误日志，进行了升级或者说是友好的异常抛出。
> 首先在CloudBuildTask这个类中，对于返回的格式进行了统一
> 

```javascript
// app/models/CloudBuildTask.js

const {SUCCESS,FAILED} = require('../constant') //定义了常量

success(msg,data){
  return this.response(SUCCESS,msg,data)
}
fail(msg,data){
  return this.response(FAILED,msg,data)
}
response(code,message,data){
  return {
    code,
    message,
    data
  }
```
> 然后在 app/io/controller/build.js中。

```javascript
async function prepare(cloudBuildTask,socket,helper) {
  socket.emit('build',helper.parseMsg('prepare',{
    message:'开始执行构建前的准备工作'
  }))
  const prepareRes = await cloudBuildTask.prepare()
  if(!prepareRes || Object.is(prepareRes.code,FAILED)){
    const msg = prepareRes ? prepareRes.message : 'prepare返回为undefined'
    socket.emit('build',helper.parseMsg('prepare failed',{
       message: `执行云构建准备工作失败，失败原因：${msg}`
    }))
    return
  }
  socket.emit('build',helper.parseMsg('prepare',{
    message:'构建前准备工作成功'
  }))
}
```

> 然后在客户端的models/cloudbuild/lib/index.js，监听的build方法中做了一个优化

```javascript
 build(){
    return new Promise((resolve,reject)=>{
      this.socket.emit('build')
      this.socket.on("build",msg=>{
        const parsedMsg = parseMsg(msg)
        if(FAILED_CODE.indexOf(parsedMsg.action)>-1){
          log.error(parsedMsg.message)
          clearTimeout(this.timer)
          this.socket.disconnect()
          this.socket.close()
        }
        log.success(parsedMsg.action,parsedMsg.message)
      })
      this.socket.on('building',msg=>{
        const parsedMsg = parseMsg(msg)
        log.success(parsedMsg.action,parsedMsg.message)
      })
    })
  }
```
#### 
#### 6-3 服务端源码下载 + 切换分支
> 在服务端 app/io/controller/build.js下，
> - 我们完成了 await prepare(cloudBuildTask,socket,helper)，
> - 接着我们继续实现  await download(cloudBuildTask,socket,helper)

```javascript
// app/io/controller/build.js

async function download(cloudBuildTask,socket,helper){
  socket.emit('build',helper.parseMsg('download repo',{
    message:'开始下载源码'
  }))
  const downloadRes = await cloudBuildTask.download()
  if(!downloadRes || Object.is(downloadRes.code,FAILED)){ //  downlod下载失败
    socket.emit('build',helper.parseMsg('download failed',{
      message:'源码下载失败'
    }))
    return 
  }else{
    socket.emit('build',helper.parseMsg('download repo',{
      message:'源码下载成功'
    }))
  }
}

// app/models/CloudBuildTask.js
 async download(){
   await this._git.clone(this._repo)  //clone仓库
   this._git = new Git(this._sourceCodeDir)  //将git地址更改，生成新的simple git
   // 切换分支  git checkout -b dev/1.0.2  origin/dev/1.0.2
   await this._git.checkout(['-b',this._branch,`origin/${this._branch}`])
   return fs.existsSync(this._sourceCodeDir) ? this.success() : this.failed()
 }
```
> 然后在客户端执行命令：cloudscope-cli publish --targetPath /Users/liumingzhou/Documents/imoocCourse/Web前端架构师/cloudscope-cli/commands/publish
> 
> 经过测试，在/User/username/.cloudscoe-cli/cloudbuild/test@1.0.2/test下安装了源码，执行git branch 看到切换到了 开发分支。

#### 
#### 6-4 服务端源码依赖安装+命令执行功能封装

与上一节的download流程一样
```javascript

// app/io/controller/build.js
await install(cloudBuildTask,socket,helper)

async function install(cloudBuildTask,socket,helper) {
  socket.emit('build',helper.parseMsg('install',{
    message:'开始安装依赖'
  }))
  const installRes = await cloudBuildTask.install()
  if(!installRes || Object.is(installRes.code,FAILED)){ //  downlod下载失败
    socket.emit('build',helper.parseMsg('install failed',{
      message:'安装依赖失败'
    }))
    return 
  }else{
    socket.emit('build',helper.parseMsg('install',{
      message:'安装依赖成功'
    }))
  }
}

// app/models/CloudBuildTask.js
async install(){
  let res = true
  res && (res = await this.execCmd('npm install --registry=https://registry.npm.taobao.org'))
  return res ? this.success(): this.failed()
}

execCmd(cmd){
  // npm install ->['npm','install']
  let command = cmd.split(' ')
  if(command.length === 0){
    return null
  }
  const firstCommand = command[0]
  const leftCommand = command.slice(1) || []
  return new Promise((resolve,reject)=>{
    const p = exec(firstCommand,leftCommand,{
      cwd:this._sourceCodeDir
    },{stdio:'pipe'})
    p.on('error',e=>{
      this._ctx.logger.error('build error',e)
      resolve(fasle)
    })
    p.on('exit',c=>{
      this._ctx.logger.info('build exit',c)
      resolve(true)
    })
    p.stdout.on('data',data => {
      this._ctx.socket.emit('building',data.toString())
    })
    p.stderr .on('data', data =>{
      this._ctx.socket.emit('building',data.toString())
    })
  })
}

function exec(command,args,options){
  const win32 = process.platform === 'win32';
  const cmd = win32 ? 'cmd': command
  const cmdArgs = win32  ?  ['/c'].concat(command,args) : args;
  return require('child_process').spawn(cmd, cmdArgs,options || {})
}
```

#### 6-5 云构建任务执行逻辑开发
```javascript

// app/io/controller/build.js

await build(cloudBuildTask,socket,helper)

async function build(cloudBuildTask,socket,helper) {
  socket.emit('build',helper.parseMsg('build',{
    message:'开始启动云构建'
  }))
  const buildRes = await cloudBuildTask.build()
  if(!buildRes || Object.is(buildRes.code,FAILED)){ //  downlod下载失败
    socket.emit('build',helper.parseMsg('build failed',{
      message:'云构建任务执行失败'
    }))
    return 
  }else{
    socket.emit('build',helper.parseMsg('build',{
      message:'云构建任务执行成功'
    }))
  }
}

// app/models/CloudBuildTask.js
 async build(){
   let res = true
   if(checkCommand(this._buildCmd)){
     res = await this.execCmd(this._buildCmd)
   }else{
     res = false
   }
   return false ? this.success():this.failed()
 }

function checkCommand(command){
  if(command){
    const commands = command.split(' ')
    if(commands.length === 0 || ['npm','cnpm'].indexOf(commands[0])<0){
      return false
    }
    return true
  }     
  return false  
}
```
> 最后在客户端经过测试，看到 build的dist目录没有构建成功：这是因为我的test源码不是一个可以打包的项目。
> 于是重写创建项目，并且publish 的时候输入参数 --refreshServer  --refreshOwner ，构建成功。








