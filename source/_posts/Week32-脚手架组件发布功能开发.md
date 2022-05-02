---
layout: post
title: Week32-脚手架组件发布功能开发
author: liugezhou
date: 2021-08-22
category: 
    Web架构之脚手架
tags:
    Web架构之脚手架
---
### 第一章 周介绍

---

#### 1-1 整体内容介绍
> - 前端组件演讲过程和大厂前端物料体系介绍
> - 组件复用体系架构设计
> - 脚手架组件创建+发布全流程实现


### 第二章 大厂物料体系介绍和前端组件平台架构设计

---

#### 2-1 大厂前端物料体系介绍
#### 2-2 组件平台架构设计
[点击查看【processon】](https://www.processon.com/embed/612322001efad44bb279c2f2)


### 第三章 脚手架组件创建和预览项目开发

---

#### 3-1 新的组件库模板开发
** **
** 1. 发布npm包**

- 下载源至某处：[https://git.imooc.com/class-110/lego-bricks](https://git.imooc.com/class-110/lego-bricks)
- 新建文件 cloudscope-cli-components
- cd cloudscope-cli-components
- npm init -y 
- 将源码移动到  cloudscope-cli-components下,并将lego-bricks改名为template
- cloudscope-cli-components/package.json下，添加publishConfig
```json
{
  "name": "cloudscope-cli-components",
  "version": "1.0.2",
  "description": "",
  "keywords": [],
  "author": "",
  "license": "ISC",
  "publishConfig": {
    "access": "public"
  }
}
```

- cloudscope-cli-components/template/package.json下，添加files
```json
"files": [
    "build",
    "src",
    "tests",
    "dist",
    ".browserslistrc",
    ".eslintrc.js",
    ".gitignore",
    ".travis.yml",
    "babel.config.js",
    "jest.config.js",
    "README.md",
    "tsconfig.json"
  ],
```

- npm publish

2. **维护mongodb,添加一条数据**
```json
{
    "_id" : ObjectId("612473d8e1694b2c0191177f"),
    "name" : "通用的Vue3组件库模版",
    "npmName" : "cloudscope-cli-components",
    "version" : "1.0.1",
    "type" : "normal",
    "installCommand" : "npm install --registry=https://registry.npm.taobao.org",
    "startCommand" : "npm run serve",
    "tag" : [ 
        "component"
    ],
    "ignore" : [ 
        "**/public/**", 
        "**.png"
    ]
}
```

3. **新建components-test目录**
> cloudscope-cli init -tp /Users/liumingzhou/Documents/imoocCourse/Web前端架构师/cloudscope-cli/commands/init


#### 3-2 组件库预览项目开发
> 这节听的有点懵逼，上一节，通过cloudscope-cli init了一个命令后，
> - npm run build
> - 根目录下继续新建examples目录,cd examples
> - vue create examples
> - cd examples中

```javascript
//main.js
import LegoComponents from "../../../dist/component-test.esm";
createApp(App).use(LegoComponents).mount('#app')

//App.vue

<template>
  <div id="app">
      <l-image src="https://img.mukewang.com/5d5e51f900011f0c06550501-100-100.jpg"></l-image>
  </div>
</template>

//vue.config.js
module.exports = {
	publicPath:'./'
}
```
> 最后继续 npm run build,打开dist目录下的index.html,查看页面效果


#### 3-3 组件多预览模式开发

> 有点小懵逼，组件库就开发完毕了

```javascript
module.exports = {
    publicPath:'./',
    pages:{
        index:{
            entry:'./src/main.js',
            templates:'./public/index.html'
        },
        index2:{
            entry:'./src/main2.js',
            templates:'./public/index.html'
        },
    }
}
```
#### 
#### 3-4 将预览功能集成到组件库模板
#### 3-5 组件初始化时自动生成配置文件
#### 3-6 组件库命名优化


### 第四章 脚手架组件发布流程开发

---

####  4-1 组件仓库初始化流程优化
cloudscope-cli 本章相关代码提交至 [cloudscope-cli/lesson32](https://github.com/liugezhou/cloudscope-cli/tree/lesson32)
> 本节主要为对仓库名称带有·@·进行一个合法的名字修改

```javascript
// cloudscope-cli/models/git/lib/index.js
class Git {
    constructor({name, version, dir},{
      ^^^^^^^
      if(name.startsWith('@') && name.indexOf('/')>0){
            const nameArray = name.split('/')
            this.name = nameArray.join('_').replace('@','')
        }else{
            this.name = name    //发布项目名称
        }
    }
}
```

#### 4-2 组件上传前预检查流程开发
cloudscope-cli 本章相关代码提交至 [cloudscope-cli/lesson32](https://github.com/liugezhou/cloudscope-cli/tree/lesson32)
```javascript
// cloudscope-cli/models/git/lib/index.js
async prepare(){
 	await this.checkComponent()  // 组件合法性检查
}
 async checkComponent(){
   let componentFile = this.isComponent()
   if(componentFile){
     log.info('开始检查build结果')
     if(!this.buildCmd){
       this.buildCmd = 'npm run build'
     }
     require('child_process').execSync(this.buildCmd,{
       cwd:this.dir
     })
     const buildPath = path.resolve(this.dir,componentFile.buildPath)
     if(!fs.existsSync(buildPath)){
       throw new Error(`构建结果：${buildPath}不存在！`)
     }
     const pkg = this.getPackageJson()
     if(!pkg.files || !pkg.files.includes(componentFile.buildPath)){
       throw new Error(`packages.json中files属性未添加构建结果目录：[${componentFile.buildPath}],请在packag.json中收到添加`)
     }
     log.success('build结果检查通过！')
   }

 }

isComponent(){
  const componentFilePath = path.resolve(this.dir,COMPONENT_FILE);
  return fs.existsSync(componentFilePath) && fse.readJsonSync(componentFilePath)
}
```

#### 4-3 组件发布前准备工作开发
cloudscope-cli 本章相关代码提交至 [cloudscope-cli/lesson32](https://github.com/liugezhou/cloudscope-cli/tree/lesson32)
```javascript
async publish(){
  let ret = false;
  if(this.isComponent()){
    log.info('开始发布组件')
    await this.saveComponentToDB()
  }else{
    ………………
  }
  if (this.prod && ret) {
    await this.uploadComponentToNpm()    
    this.runCreateTagTask();
  }
  
async saveComponentToDB(){
  // 将组件信息上传至数据库-RDS

  // 将组件多预览页面上传至OSS
}
  async uploadComponentToNpm(){
    // 完成组件上传至npm
  }
```

#### 4-4 创建RDS组件表+后端MySQL插件集成
cloudscope-cli-server 本章相关代码提交至 [cloudscope-cli-server/lesson32](https://github.com/liugezhou/cloudscope-cli-server/tree/lesson32)

> 本章主要是在服务端链接mysql数据库，并做相关测试
> 编写相关代码前：cnpm i -S egg-mysql

```javascript
// app/config/db.js
const MYSQL_HOST = 'liugezhou.com'
const MYSQL_PORT = 3306
const MYSQL_USER = 'root' 
const MYSQL_PWD = fs.readFileSync(path.resolve(userHome,'.cloudscope-cli','mysql_pwd')).toString().trim()
const MYSQL_DB ='imooc_web_architect_cli'

//app/config/plugin.js
exports.mysql = {
  enable:true,
  package: 'egg-mysql'
}

//app/config/config.default.js
config.mysql = {
    client: {
      host: MYSQL_HOST, 
      port: MYSQL_PORT,
      user:MYSQL_USER,
      password:MYSQL_PWD,
      database:MYSQL_DB
    },
    app:true,
    agent:false
  }


//相关测试代码
// app/app/router.js
router.get('/mysql',controller.project.mysqlTest)

// app/controller/project.js
async mysqlTest(){
  const {ctx, app } = this
  const list = await app.mysql.select('component')
  ctx.body = JSON.stringify(list)
}
```

#### 4-5 组件上传数据库准备工作开发
cloudscope-cli 本章相关代码提交至 [cloudscope-cli/lesson32](https://github.com/liugezhou/cloudscope-cli/tree/lesson32)
```javascript
// models/git/lib/index.js
const ComponentRequest = require('../lib/ComponentRequest');
 async saveComponentToDB(){
   // 将组件信息上传至数据库-RDS
   log.info('上传组件信息至OSS+写入数据库')
   const componentFile = this.isComponent()
   let componentExamplePath = path.resolve(this.dir,componentFile.examplePath);
   let dirs = fs.readdirSync(componentExamplePath)
   if(dirs.includes('dist')){
     componentExamplePath = path.resolve(componentExamplePath,'dist')
     dirs = fs.readdirSync(componentExamplePath)
     componentFile.examplePath = `${componentFile.examplePath}/dist`
   }
   dirs = dirs.filter(dir => dir.match(/^index(\d)*.html$/))
   componentFile.exampleList = dirs
   componentFile.exampleRealPath = componentExamplePath
   const data = await ComponentRequest.createComponent({
     component:componentFile,
     git: {
       type: this.gitServer.type,
       remote: this.remote,
       version: this.version,
       branch: this.branch,
       login: this.login,
       owner: this.owner,
     },
   })
   if (!data) {
     throw new Error('上传组件失败');
   }
   // 2.将组件多预览页面上传至OSS
   return true;
 }

// models/git/lib/ComponentRequest.js
const axios = require('axios');
const log = require('@cloudscope-cli/log');

module.exports = {
  createComponent: async function(component) {
    try {
      const response = await axios.post('http://liugezhou.com:7001/api/v1/components', component);
      log.verbose('response', response);
      const { data } = response;
      if (data.code === 0) {
        return data.data;
      }
      return null;
    } catch (e) {
      throw e;
    }
  },
};
```
#### 4-6 组件上传restful api开发
cloudscope-cli-server 本章相关代码提交至 [cloudscope-cli-server/lesson32](https://github.com/liugezhou/cloudscope-cli-server/tree/lesson32)
```javascript
// app/constant.js
module.exports = {
    STATUS: {
      ON: 1,
      OFF: 0,
    },
}

// app/router.js
router.resources('components', '/api/v1/components', controller.v1.components);

// app/Controller/v1/Components.js
'use strict'

const Controller = require("egg").Controller;
const constant = require('../../constant');
const ComponentService = require('../../service/ComponentService');

class ComponentsController extends Controller {

    // api/v1/components
    async index() {
        const { ctx } = this;
        ctx.body = 'get component'
    }

    // api/v1/components/:id
    async show() {
        const { ctx } = this;
        ctx.body = 'get single component'
    }

    // post data
    async create() {
        const { ctx, app } = this;
        const { component, git } = ctx.request.body;
        const timestamp = new Date().getTime()
          // 1. 添加组件信息
        const componentData = {
            name: component.name,
            classname: component.className,
            description: component.description,
            npm_name: component.npmName,
            npm_version: component.npmVersion,
            git_type: git.type,
            git_remote: git.remote,
            git_owner: git.owner,
            git_login: git.login,
            status: constant.STATUS.ON,
            create_dt: timestamp,
            create_by: git.login,
            update_dt: timestamp,
            update_by: git.login,
        };
        const componentService = new ComponentService(app);
        ctx.body = 'create component'
    }
}

module.exports = ComponentsController
```

#### 4-7 组件上传数据库逻辑开发
```javascript
// app/service/ComponentService.js
'use strict'

class ComponentService {
    constructor(app){
        this.app = app,
        this.name = 'component'
    }

    async queryOne(query){
        const data = await this.app.mysql.select(this.name, {
            where: query,
          });
          if (data && data.length > 0) {
            return data[0];
          }
          return null;
    }

    async insert(data) {
        const res = await this.app.mysql.insert(this.name, data);
        return res.insertId;
    }
}

module.exports = ComponentService

//app/service/versionService.js
'use strict'

class VersionService {
    constructor(app) {
        this.app = app;
        this.name = 'version'
    }
    async queryOne(query) {
        const data = await this.app.mysql.select(this.name, {
            where: query,
        });
        if (data && data.length > 0) {
            return data[0];
        }
        return null;
    }

    async insert(data) {
        const res = await this.app.mysql.insert(this.name, data);
        if (res.affectedRows > 0) {
            return true;
        }
        return false
    }

    async update(data, query) {
        const res = await this.app.mysql.update(this.name, data, {
            where: query
        })
        if (res.affectedRows > 0) {
            return true;
        }
        return false
    }
}

module.exports = VersionService

//app/controller/v1/Components.js
async create() {
        const { ctx, app } = this;
        const { component, git } = ctx.request.body;
        const timestamp = new Date().getTime()
        // 1. 添加组件信息
        const componentData = {
            name: component.name,
            classname: component.className,
            description: component.description,
            npm_name: component.npmName,
            npm_version: component.npmVersion,
            git_type: git.type,
            git_remote: git.remote,
            git_owner: git.owner,
            git_login: git.login,
            status: constant.STATUS.ON,
            create_dt: timestamp,
            create_by: git.login,
            update_dt: timestamp,
            update_by: git.login,
        };
        const componentService = new ComponentService(app);
        const haveComponentInDB = await componentService.queryOne({
            className: component.className
        })
        let componentId;
        if (!haveComponentInDB) {
            componentId = await componentService.insert(componentData);
        } else {
            componentId = haveComponentInDB.id
        }
        if (!componentId) {
            ctx.body = failed('添加组件失败')
        }
        // 2.添加组件版本信息
        const versionData = {
            component_id: componentId,
            version: git.version,
            build_path: component.buildPath,
            example_path: component.examplePath,
            example_list: JSON.stringify(component.exampleList),
            status: constant.STATUS.ON,
            create_dt: timestamp,
            create_by: git.login,
            update_dt: timestamp,
            update_by: git.login,
        }
        const versionService = new VersionService(app);
        const haveVersionInDB = await versionService.queryOne({
            component_id: componentId,
            version: git.version
        });
        if (!haveVersionInDB) {
            const versionRes = await versionService.insert(versionData)
            if (!versionRes) {
                ctx.body = failed('添加组件失败')
            }
        } else {
            const updateData = {
                build_path: component.buildPath,
                example_path: component.examplePath,
                example_list: JSON.stringify(component.exampleList),
                update_dt: timestamp,
                update_by: git.login,
            };
            const versionRes = await versionService.update(updateData, {
                component_id: componentId,
                version: versionData.version,
            });
            if (!versionRes) {
                ctx.body = failed('更新组件失败');
                return;
            }
        }

    }
```

#### 4-8 组件NPM发布逻辑开发

> 本节主要是调试过多，重要几行代码为在脚手架 cloudscope-cli中添加NPM发布逻辑

```javascript
async uploadComponentToNpm() {
  // 完成组件上传至npm
  if (this.isComponent()) {
    log.info('开始发布组件NPM')
    require('child_process').execSync('npm publish', {
      cwd: this.dir
    })
    log.success('组件NPM发布成功')
  }
}
```

#### 4-9 组件自动生成远程仓库Tag问题解决

> End

