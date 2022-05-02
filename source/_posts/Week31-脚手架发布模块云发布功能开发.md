---
layout: post
title: Week31-脚手架发布模块云发布功能开发
author: liugezhou
date: 2021-08-15
category: 
    Web架构之脚手架
tags:
    Web架构之脚手架
---
### 第一章 本周导学
---

#### 1-1 本周介绍和学习方法
> - 云发布原理、架构和实现
> - OSS API(OSS接入指南)
> - 上线发布流程(支持Hash和History模式发布)
> - 附赠：Node高分库分享(awesome-nodejs)

### 
### 第二章 云发布模块架构设计

---


#### 2-1 前端发布OSS架构设计
> - CloudBuild实例添加参数：prod（是否为正式版本）
> - 添加准备阶段 ：获取OSS文件，询问是否覆盖

#### 2-2 云发布架构和流程设计
[点击查看【processon】](https://www.processon.com/embed/61127e8907912940a9654a57)

### 第三章 云发布功能开发

---

> 本章内容更新代码为分支 [lesson31](https://github.com/liugezhou/cloudscope-cli/tree/lesson31)


#### 3-1 实现云发布前的预检查逻辑
> 在上一节中，mdoels/git/lib/index.js中，有preparePublish方法，之前检查命令是否为npm或cnpm
> 在上一节基础上，这里添加了，执行构建命令是否为 build，代码如下
> 

```javascript
// models/git/lib/index.js


async preparePublish(){
  log.info('开始进行云构建前代码检查')
  …………
  const pkg = this.getPackageJson()
  const buildCmdArray = this.buildCmd.split('')
  const buildLastCmd = buildCmArrat.slice(-1).toString()
  if(!pkg.scripts || !Object.keys(pkg.scripts).includes(lastCmd)){
    throw new Error(`${this.buildCmd}命令不存在`)
  }
  log.success('云构建代码预检查通过')
}

getPackageJson(){
	const pkgPath = path.resolve(this.dir,'package.json')
  if(!fs.existSync(pkgPath)){
  	throw new Error(`package.json不存在,源码目录:${this.dir}`)
  }
  return fse.readJsonSync(pkgPath)
}
```

#### 3-2 静态资源服务器类型选择逻辑开发
> - 上一节我们检查了build这个命令
> - 接着，我们需要选择上传资源服务器的类型，也就是OSS
>    - 这里的代码是为了后续如果要修改资源服务器类型，可以进行代码的再开发--添加其他资源服务器类型。
>    - 与检查git_token或者git_server等一样，本节需要去新建或者检查.git_publish文件，代码实现如下
> - 最后，cloudscope-cli publish这个命令添加了 --prod这个参数,以及.git_publish文件内容，传入到 Cloudbuild类中去

```javascript
// models/git/lib/index.js
const GIT_PUBLISH_FILE='.git_publish'
const GIT_PUBLISH_TYPE=[{
    name:'OSS',
    value:'oss'
}]

 async preparePublish(){
   log.info('开始进行云构建前代码检查')
   ………………
   log.success('云构建代码预检查通过')

   const gitPublishPath = this.createPath(GIT_PUBLISH_FILE)
   let gitPublish = readFile(gitPublishPath)
   if(!gitPublish){ // 如果没有读取到.git-publish文件中的内容
     gitPublish = await this.choicePublish(gitPublishPath)
     log.success('git publish 写入成功',`${gitPublish} -> ${gitPublishPath}`)
   }else{ // 如果读取到了 内容
     log.success('git publish 文件读取成功！')
   }
   this.gitPublish = gitPublish
 }

async choicePublish(gitPublishPath){
  const gitPublish = (await inquirer.prompt({
    type:'list',
    name:'gitPublish',
    message:'请选择你想要上传代码的平台',
    choices:GIT_PUBLISH_TYPE
  })).gitPublish;
  writeFile(gitPublishPath,gitPublish)
  return gitPublish
}
```
> 关于 --prod参数的传入，与--refreshToken等一样，这里就不做演示了。


#### 3-3 云发布服务端预检查逻辑实现
> 本节是服务端相关的代码实现，同样代码为分支 [ lesson31](https://github.com/liugezhou/cloudscope-cli-server/tree/lesson31)
> 主要是拿到云构建结果的dist或者build目录

```javascript
// app/io/controller/build.js
 await prePublish(cloudBuildTask,socket,helper)

async function prePublish(cloudBuildTask,socket,helper){
  socket.emit('build',helper.parseMsg('pre-publish',{
    message:'开始发布前检查'
  }))
  const prePublishRes = await cloudBuildTask.prePublish()
  if(!prePublishRes || Object.is(prePublishRes.code,FAILED)){ 
    socket.emit('build',helper.parseMsg('pre-publish failed',{
      message:'发布前检查失败,失败原因：' + (prePublishRes && prePublishRes.message ? prePublishRes.message:'未知')
    }))
    throw new Error('发布终止')
  }else{
    socket.emit('build',helper.parseMsg('pre-publish',{
      message:'发布前检查通过'
    }))
  }
}

//app/models/cloudBuildTask.js
async prePublish(){
  //获取构建结果
  const buildPath = this.findBuildPath()
  //检查构建结果
  if(!buildPath){
    return this.failed('未找到构建结果，请检查')
  }
  this._buildPath = buildPath
  return this.success()
}



findBuildPath(){
  const buildDir =['build','dist']
  const buildPath = buildDir.find(dir=>fs.existsSync(path.resolve(this._sourceCodeDir,dir)))
  this._ctx.logger.info('buildPath',buildPath)
  if(buildPath){
    return path.resolve(this._sourceCodeDir,buildPath)
  }
  return null
}
```

#### 3-4 创建OSS bucket+OSS实例化开发(服务端)
> - cnpm install -S ali-oss
> - 客户端传递prod参数到服务端，服务端根据prod参数，获取不同环境的OSS
> 

```javascript
// app/models/CloudBuildTask.js
const OSS = require('./OSS')
const config = require('../../config/db')
class CloudBuildTask(options,ctx){
	constructor(){
  	  this._prod = options.prod === 'true' ? true : false
  }
}

async prepare(){
  fse.ensureDirSync(this._dir)
  fse.emptyDirSync(this._dir)
  this._git = new Git(this._dir)
  if(this._prod){ //生产准备OSS
    this.oss =new OSS(config.OSS_PROD_BUCKET)
  }else{//测试
    this.oss =new OSS(config.OSS_DEV_BUCKET)
  }
  console.log(this.oss)
  return this.success()
}

// app/models/OSS.js
'use strict';
const OSS = require('ali-oss');
const config = require('../../config/db')
class OSS{
    constructor(bucket){
        this.oss = new OSS({
            region:config.OSS_REGION,
            accessKeyId: config.OSS_ACCESS_KEY,
            accessKeySecret: config.OSS_ACCESS_SECRET_KEY,
            bucket,
        })
    }
}

module.exports = OSS

//app/config/db.js
const fs = require('fs')
const path = require('path')
const userHome = require('user-home')
const OSS_ACCESS_KEY = ''
const OSS_ACCESS_SECRET_KEY = fs.readFileSync(path.resolve(userHome,'.cloudscope-cli','oss_access_secret_key')).toString()
const OSS_PROD_BUCKET=''
const OSS_DEV_BUCKET=''
const OSS_REGION=''
```
> 以上关于OSS Key等的配置，可去阿里云免费试用一个月OSS对象存储功能，填入相应的值。

#### 3-5 云发布核心流程：上传OSS功能开发
代码逻辑如下
> - npm i -S glob

```javascript
// app/io/controller/build.js

await publish(cloudBuildTask,socket,helper)

async function publish(cloudBuildTask,socket,helper){
  socket.emit('build',helper.parseMsg('publish',{
    message:'开始发布'
  }))
  const publishRes = await cloudBuildTask.publish()
  if(!publishRes || Object.is(publishRes.code,FAILED)){ //  downlod下载失败
    socket.emit('build',helper.parseMsg('publish failed',{
      message:'发布失败'
    }))
    return 
  }else{
    socket.emit('build',helper.parseMsg('publish',{
      message:'发布成功'
    }))
  }
}

// app/models/CloudBuildTask.js
  async publish(){
    return new Promise((resolve,reject) =>{
      glob('**',{
        cwd:this._buildPath,
        nodir:true,
        ignore:'**/node_modules/**'
      },(err,files) =>{
        if(err){
          resolve(false)
        }else{
          Promise.all(files.map(async file=>{
            console.log(file)
            const filePath = path.resolve(this._buildPath,file)
            const uploadOSSRes = await this.oss.put(`${this._name}/${file}`,filePath) 
            return uploadOSSRes
          })).then(()=>{
            resolve(true)
          }).catch(err=>{
            this._ctx.logger.error(err)
            resolve(false)
          })
        }
      })
    })
  }

// app/models/OSS.js

 async put(object, localPath, options = {}) {
   await this.oss.put(object, localPath, options);
 }
```
> 最后经过测试，在阿里云控制台看到相关的OSS文件已上传。


#### 3-6 OSS域名绑定 + CDN绑定

- 域名绑定
- CDN绑定

### 第四章 云发布流程完善

---


#### 4-1 获取OSS API开发
**服务端**
> - **router.js中添加路由 router.get('/project/oss', controller.project.getOSSProject);**

本节主要是获取OSS上传的文件，使用oss的
```javascript
// app/controller/project.js
const { failed } = require("../utils/request")
const OSS = require('../models/OSS')
const config = require('../../config/db')

async getOSSProject(){
  const { ctx } = this
	const ossProjectType = ctx.query.type
  const ossProjectName = ctx.query.name
  if(!ossProjectName){
    ctx.body = failed('项目名称不存在)
    return
  }
  if(!ossProjectType){
  	ossProjectType = 'prod'
  }
  let oss;
  if(Object.is(ossProjectType,'prod')){
  	oss = new OSS(config.OSS_DEV_BUCKET)
  }else{
    oss = new OSS(config.OSS_PROD_BUCKET)
	}
  const ossList = await oss.list(ossProjectName)
  ctx.body = ossList
}
  
  
  //app/utils/request.js
  'use strict'
  function success(message,data){
  	return {
    		code:0,
      	message,
      	data
    }
  }
  function failed(message,data){
  	return {
    		code:-1,
      	message,
      	data
    }
  }
  
  module.exports = {
  	success,
    failed
  }


//app/models/OSS.js
async list({prefix}){
	const ossFileList = await this.oss.list({prefix})
  if(ossFileList && ossFileList.objects){
  	return ossFileList.objects
  }
  return []
}
```

#### 4-2 覆盖发布逻辑开发
> 服务端提供了获取OSS文件列表的接口，接着要在客户端去请求OSS上是否有文件，且是否发布。
> 代码如下：

```javascript
// models/cloudbuild/index.js
const getProjectOSS = require('./getProjectOSS')
async prepare(){
  // 判断是否处于正式发布
  if(this.prod){
    // 1.获取OSS文件 
    const name = this.git.name
    const type = this.prod ? 'prod':'dev'
    const ossProject = await getProjectOSS({name,type})
    console.log(ossProject)
    //2.判断当前项目OSS文件是否存在
    if(Object.is(ossProject.code,0) && ossProject.data.length>0){
      // 3.询问客户是否进行覆盖安装
      const cover = (await inquirer.prompt({
        type:'list',
        defaultValue:true,
        name:'cover',
        message:`OSS已存在[${name}]项目，是否强行覆盖发布？`,
        choices:[{
          name:'覆盖发布',
          value:true
        },{
          name:'放弃发布',
          value:false
        }]
      })).cover
      if(!cover){ //放弃发布
        throw new Error('发布终止')
      }
    }
  }
}

// models/cloudbuild/cloudbuild.js
const request = require('@cloudscope-cli/request')

module.exports = function (data) {
  return request({
    url:'/project/oss',
    method:'get',
    params:data
  })
}
```
#### 4-3 服务端缓存文件清除功能实现
本节主要是对redis以及缓存文件进行清除,并在build结束后，断开链接
```javascript
// app/io/middleware/auth.js
//清除缓存文件
const cloudBuildTask = await createCloudBuildTask(ctx,app)
await cloudBuildTask.clean()

// app/models/CloudBuildTask 
async clean(){
  if (fs.existsSync(this._dir)) {
    fse.removeSync(this._dir);
  }
  const { socket } = this._ctx;
  const client = socket.id;
  const redisKey = `${REDIS_PREFIX}:${client}`;
  await this._app.redis.del(redisKey);
}

// app/io/controller/build.js
 socket.emit('build',helper.parseMsg('build success',{
   message:`云构建成功，访问链接：https://${cloudBuildTask._prod ? 'cloudscope-cli' : 'cloudscope-cli-dev'}.liugezhou.online/${cloudBuildTask._name}/index.html`
 }))
socket.disconnect()
```
#### 4-4 自动提交代码 BUG 修复
> 本地未出错

#### 4-5 history模式发布原理讲解
> 使用 createWebHistory发布代码时，在OSS服务器，改变url地址，再刷新的话，会显示404，在nginx中有try_files的配置，而我们这里没有，因此除了将createWebhistory改为createWebHashHistory模式外，本节课讲述history模式发布解决这个问题---通过**本地**方式演示。
> - 首先，history模式，需要在nginx上做配置
> - 然后，分两个步骤实现
>    - index.html放到nginx服务器指定位置,配置好这个nginx
>    - css/js等文件上传到OSS服务器上

1. **vue.config.js中配置OSS路径publicPath**
> module.exports= {
> publicPath:'https://xxxx.online/vue-router-demo/'
>  }

2. **本地打包文件dist下目录手动上传至oss服务器   ：vue-router-demo目录下。**
2. **将本地dist/index.html下文件复制到Desktop/vue-router/index.html中。**
4. **配置您选配置文件**
```javascript
server {
  listen  8081;
  server_name resource;
  root /Users/liumingzhou/Desktop/ ;
  autoindex on;
  location / {
    add_header  Access-Control-Allow-Origin *;

  }
  location /vue-router-demo {
    try_files  $uri  $uri/  /vue-router-demo/index.html;
  }
  add_header   Cache-Control "no-cache, must-revalidate";
}
```

5. **这个时候启动本地nginx服务器，访问8081端口，再刷新页面，histpry路由模式就不会报404了**



#### 4-6 history模式远程发布原理讲解
> - 登录ESC服务器
> - 配置nginx
> - 本地文件上传到服务器 
>    -  scp -r /User/liugezhou/Desktop/vue-router-demo/index.html  root@liugezhou:upload/test



#### 4-7 脚手架自动上传模板逻辑开发
#### 4-8 获取 OSS 文件 API 开发
#### 4-9 上传模板功能实现
> 通过三个新的参数：sshUser 、sshIp、sshPath继续下一步，具体代码就不贴了，这几章就只是流程的问题了。

#### 
#### 4-10 自动打tag+合并代码至master分支流程开发
> **本节课直接看着仓库代码，具体的流程还需要后续深度学习。**



### 第五章 本周加餐：node常用三方库介绍

---

#### 5-1 Node高分库：PDF文件生成工具——PDFKit
> [awesome-nodejs](https://github.com/sindresorhus/awesome-nodejs)

本节sam老师，主要是讲解了这个[pdfkit](https://github.com/foliojs/pdfkit)库。

- 创建一个nodejs项目:npm init -y
- npm install -S pdfkit
- 新建kit/index.js,官方仓库代码copy如下：
```javascript
const PDFDocument = require('pdfkit');
const fs = require('fs');

// Create a document
const doc = new PDFDocument();

// Pipe its output somewhere, like to a file or HTTP response
// See below for browser usage
doc.pipe(fs.createWriteStream('output.pdf'));

// Embed a font, set the font size, and render some text
doc
  .font('fonts/PalatinoBold.ttf')
  .fontSize(25)
  .text('Some text with an embedded font!', 100, 100);

// Add an image, constrain it to a given size, and center it vertically and horizontally
doc.image('path/to/image.png', {
  fit: [250, 300],
  align: 'center',
  valign: 'center'
});

// Add another page
doc
  .addPage()
  .fontSize(25)
  .text('Here is some vector graphics...', 100, 100);

// Draw a triangle
doc
  .save()
  .moveTo(100, 150)
  .lineTo(100, 250)
  .lineTo(200, 250)
  .fill('#FF3300');

// Apply some transforms and render an SVG path with the 'even-odd' fill rule
doc
  .scale(0.6)
  .translate(470, -380)
  .path('M 250,75 L 323,301 131,161 369,161 177,301 z')
  .fill('red', 'even-odd')
  .restore();

// Add some text with annotations
doc
  .addPage()
  .fillColor('blue')
  .text('Here is a link!', 100, 100)
  .underline(100, 100, 160, 27, { color: '#0000FF' })
  .link(100, 100, 160, 27, 'http://google.com/');

// Finalize PDF file
doc.end();
```

#### 5-2 Node Excel处理库讲解
[https://github.com/dtjohnson/xlsx-populate](https://github.com/dtjohnson/xlsx-populate)

#### 5-3 命令行交互库Listr讲解

- [np](https://github.com/sindresorhus/np)：Better npm publish
- np核心库：[listr](https://www.npmjs.com/package/listr)
#### 5-4 利用Listr对项目自动创建Tag逻辑进行优化
```javascript
const Listr = require('listr')
const { Observable } = require('rxjs')

const tasks = new Listr([
    {
        title:'Task1',
        task:()=>new Listr([{
            title:'Task1-1',
            task:()=>{
                return new Observable(o=>{
                    o.next('Task-1-1')
                    setTimeout(() => {
                        o.next('Task-1-1-1')
                        o.complete()
                    }, 1000);
                    
                })
            }
        }])
    },
    {
        title:'Task2',
        task:()=>{
            throw new Error('error')
    },
}
])
tasks.run()
process.on('unhandledRejection',(e)=>{
    console.log(e.message)
})
```














