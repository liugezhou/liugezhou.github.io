---
layout: post
title: Week29-脚手架发布模式git自动化流程开发
author: liugezhou
date: 2021-07-05
category: 
    Web架构之脚手架
tags:
    Web架构之脚手架
---
### 第一章 本周导学

---

#### 1-1 本周整体介绍和学习方法
> - GitFlow实战
> - 通过simple-git操作git命令
> - Github和Gitee openAPI接入
> - 本周加餐：Node最佳实践分享

##### 主要内容
> - Git仓库初始化(利用Github和Gitee OpenAPI)
> - 本地Git初始化
> - GitFlow流程实现(代码自动提交)

##### 附赠内容
> Node项目最佳实践：
> - 项目结构最佳实践
> - 异常处理最佳实践
> - 测试最佳实践
> - 发布上线最佳实践
> - 安全最佳实践

### 第二章 Git Flow 模块架构设计

---

#### 2-1 GitFlow模块架构设计
[点击查看【processon】](https://www.processon.com/embed/610bf5d81e0853337b1eb8bd)
#### 2-2 GitFlow流程回顾

[点击查看【processon】](https://www.processon.com/embed/60f2487c1efad41bbea86894)
### 第三章 Github&Gitee API 接入

---

#### 3-1 创建Git类
> 1. 首先创建一个新的package--git，用来管理与git相关的所有内容
> - lerna create git models/git
> 2. 在上周代码写完this.prepare()之后（commands/publish/lib/index.js中），我们就需要去调用这个新建的git包，实例化出来一个对象，并将projectInfo的信息传递进去
> - const Git = require('@cloudscope-cli/git')
> - const git = new Git()
> 3. 我们在新建的 models/git/lib/index.js中新建一个Git类，本节代码内容为：

```javascript
'use strict';

const SimpleGit = require('simple-git')

class Git {
    constructor({name, version, dir}){
        this.name = name
        this.version = version
        this.dir = dir
        this.git = SimpleGit(dir)
        this.gitServer = null
    }

    prepare(){

    }
    init(){
        console.log('Git init')
    }
}

module.exports = Git;
```

#### 3-2 用户主目录检查逻辑开发

> 本节的主要代码为在Class Git中编写prepare方法，检查用户主目录是否存在

```javascript
'use strict';

const path = require('path')
const fs = require('fs')
const SimpleGit = require('simple-git')
const userHome = require('user-home')
const log = require('@cloudscope-cli/log')
const fse = require('fs-extra')

const DEFAULT_CLI_HOME = '.cloudscope-cli'
class Git {
    constructor({name, version, dir}){
        this.name = name
        this.version = version
        this.dir = dir
        this.git = SimpleGit(dir)
        this.gitServer = null
        this.homePath = null
    }

    async prepare(){
        this.checkHomePath();// 检查缓存主目录
    }

    checkHomePath(){
        if(!this.homePath){
            if(process.env.CLI_HOME_PATH){
                this.homePath = process.env.CLI_HOME_PATH
            }else{
                this.homePath = path.resolve(userHome,DEFAULT_CLI_HOME)
            }
        }
        log.verbose('home:',this.homePath )
        fse.ensureDirSync(this.homePath);
        if(!fs.existsSync(this.homePath)){
            throw new Error('用户主目录获取失败！')
        }
    }
    init(){
        console.log('Git init')
    }
}

module.exports = Git;
```
> 本节对path/fs/user-home/fs-extra进行简单回顾。
> - path：node内置，path模块提供了处理文件和目录的路径的实用工具。
>    - **path.dirname(path)** ,返回something(文件或文件夹)所在的文件目录。
>    - **path.extname(path)**:返回最后一次出现**.**字符到字符串的结尾
>    - **path.isAbsolute(path)**:确定这个path路径是否为绝对路径。
>    - **path.join([...paths])**:路径拼接
>    - **path.parse(path):**将路径返回成一个对象{root,dir,name,ext,base}
>    - **path.resolve(path):**将路径或路径片段的序列解析为**绝对路径**，给定的路径序列从右向左处理
>    - **path.seq:**window上是\，Mac上是/
> - fs：node内置，文件系统
>    - **fs.existsSync(path):**如果路径存在返回true，不存在返回false
> - user-home\fs-extra:第三方


#### 3-3 选择远程Git仓库逻辑开发
> - 上一节的内容总结一句大白话就是：获取 /User/username/.cloudscope-cli目录，如果没有就创建。
> - 本节的内容一句话总结就是实现了一个**checkGitServer**方法，这个方法的主要功能是检查在上文提到的.cloudscope-cli下是否有.git/.git-server文件，没有的话通过 inquirer询问创建
> - 且在 @cloudscope-cli/utils下新建了** readFile**和writeFile方法
> - 并且添加了一个参数：refreshServer，如果有这个参数就判断是否重写.git-server文件
> 
本节新开发添加代码如下：

```javascript
const { readFile,writeFile } = require('@cloudscope-cli/utils')
const inquirer = require('inquirer')

const DEFAULT_CLI_HOME = '.cloudscope-cli'
const GIT_ROOT_DIR = '.git'
const GIT_SERVER_FILE = '.git_server'

const GITHUB = 'github'
const GITEE ='gitee'
const GIT_SERVER_TYPE = [{
    name:'Github',
    value: GITHUB
},{
    name: 'Gitee',
    value: GITEE
}]  

 async prepare(){
        this.checkHomePath();// 检查缓存主目录
        await  this.checkGitServer();//检查用户远程仓库类型
 }

async checkGitServer(){
        const gitServerPath = this.createPath(GIT_SERVER_FILE)
        let gitServer = readFile(gitServerPath)
        if(!gitServer){ // 如果没有读取到.git-server文件中的内容
            gitServer = await this.choiceServer(gitServerPath)
            log.success('git server 写入成功',`${gitServer} -> ${gitServerPath}`)
        }else{ // 如果读取到了 内容
            if(this.refreshServer){ // 是否重写标识
                const refresh = (await inquirer.prompt([{
                    type:'confirm',
                    name:'ifContinue',
                    default:false,
                    message:'当前.git-server目录已存在，是否要重写选择托管平台？'
                }])).ifContinue
                if(refresh){
                    gitServer = await this.choiceServer(gitServerPath)
                    log.success('git server 重写成功',`${gitServer} -> ${gitServerPath}`)
                }else{
                    log.success('git server 获取成功 ', gitServer)
                }
            }else{ //不重写，直接读取
                log.success('git server 获取成功 ', gitServer)
            }
        }
        this.gitServer = this.createServer(gitServer)
    }

  async choiceServer(gitServerPath){
        const gitServer = (await inquirer.prompt({
            type:'list',
            name:'server',
            message:'请选择你想要托管的Git平台',
            default: GITHUB,
            choices:GIT_SERVER_TYPE
        })).server;
        writeFile(gitServerPath,gitServer)
        return gitServer
    }

    createPath(file){
        const rootDir = path.resolve(this.homePath,GIT_ROOT_DIR)
        const serverDir = path.resolve(rootDir,file)
        fse.ensureDirSync(rootDir)
        return serverDir
    }
```
> @cloudscope-cli/utils 下的readFile和writeFile方法实现为使用node自带的fs.readFileSync(path)和fs.writeFileSync(path)方法

#### 
#### 3-4 创建GitServer类
> 本节主要创建了一个GitServer类，并新建Github类和Gitee类分别继承GitServer。

```javascript
function error(methodName) {
    throw new Error(`${methodName} must be implemented!` )
}

class GitServer {
    constructor(type,token){
        this.type= type
        this.token = token
    }

    setToken(){
        error('setToken')
    }

    createRepo(){
        error('createRepo')
    }

    createOrgRepo(){
        error('createOrgRepo')
    }

    getRemote(){
        error('getRemote')
    }
    getuser(){
        error('getuser')
    }
    getOrg(){
        error('getOrg')
    }
}

module.exports = GitServer
```
#### 3-5 生成远程仓库Token逻辑开发

> 本节课的主要内容为生成远程仓库Token。
> - 首先在类Github.js和Gitee.js中分别实现获取帮助文档和相应token的文档方法

```javascript
//Github.js
    getSSHKeysUrl(){
          return 'https://gitee.com/profile/sshkeys';
    }

//Gitee.js
		getSSHKeysUrl(){
        return 'https://gitee.com/profile/sshkeys';
    }
    getTokenHelpUrl(){
        return 'https://gitee.com/help/articles/4191'
    }
```
> 然后在models/git/index.js中实现获取Giteetoken的方法
> - 安装了  terminal-link库，且版本号为2.1.1，功能是直接在ternimal中点击跳转链接

```javascript
	const terminalLink = require('terminal-link')	
	const GIT_TOKEN_FILE = '.git_token'

	async checkGitToken(){
        const tokenPath = this.createPath(GIT_TOKEN_FILE)
        let token = readFile(tokenPath)
        if(!token || this.refreshServer){
            log.warn(this.gitServer.type + ' token未生成,请先生成！' + this.gitServer.type + ' token,'+terminalLink('链接',this.gitServer.getTokenHelpUrl())) ;
            token = (await inquirer.prompt({
                type:'password',
                name:'token',
                message:'请将token复制到这里',
                default:'',
            })).token
            writeFile(tokenPath,token)
            log.success('token 写入成功',` ${tokenPath}`)
        }else{
            log.success('token获取成功',tokenPath)
        }
        this.token  = token
        this.gitServer.setToken(token)
    }
```

#### 3-6 Gitee API接入+获取用户组织信息功能开发

> 本章节主要是Gitee API的接入：获取用户信息和组织信息
> - 使用第三方库有 axios
> - 新建的文件有 GitServerRequest.js

```javascript
const axios = require('axios')
const BASE_URL = 'https://gitee.com/api/v5'

class GiteeRequest {
    constructor(token){
        this.token = token
        this.service = axios.create({
            baseURL:BASE_URL,
            timeout:5000
        })
        this.service.interceptors.response.use(
            response =>{
                return response.data
            },
            error =>{
                if(error.response && error.response.data){
                    return error.response
                } else {
                    return Promise.reject(error)
                }
            }
        )
    }

    get(url,params,headers){
        return this.service({
            url,
            params:{
                ...params,
                access_token:this.token
            },
            method:'get',
            headers,
        })
    }
}

module.exports = GiteeRequest
```
> index.js主代码主要实现的方法是getUserAndOrgs：

```javascript
async getUserAndOrgs(){
        this.user = await this.gitServer.getUser()
        if(!this.user){
            throw new Error('用户信息获取失败')
        }
        this.orgs = await this.gitServer.getOrg(this.user.login)
        if(!this.orgs){
            throw new Error('组织信息获取失败')
        }
        log.success(this.gitServer.type + ' 用户和组织信息获取成功')
    }
```

#### 3-7 Github API接入开发
> 本节代码类似于Gitee API，不同点在于，Github API需要在headers中传入token：_config_.headers['Authorization'] =`token ${**this**.token}`
> 然后，基础BASE_URL更换，获取组织URL更换，改动部分代码即可。


#### 3-8 远程仓库类型选择逻辑开发
> 之前的章节我们选择了Git托管类型，生成了相关托管平台的token，获取到了个人和组织的信息，然后这节我们将要继续选择：确认远程仓库的类型(是个人还是组织)，如果我们拿到的信息只有个人，那么就不显示组织选项。
> 同样，我们要将拿到的个人或者组织登录名(login)以及类型(owner)写入到缓存文件中.
> 

```javascript
const GIT_OWN_FILE = '.git_own'
const GIT_LOGIN_FILE = '.git_login'

const REPO_OWNER_USER = 'user'
const REPO_OWNER_ORG = 'org'
const GIT_OWNER_TYPE = [{
    name:'个人',
    value: REPO_OWNER_USER
},{
    name: '组织',
    value: REPO_OWNER_ORG
}]
const GIT_OWNER_TYPE_ONLY = [{
    name:'个人',
    value: REPO_OWNER_USER
}]

………………
await this.checkGitOwner();//确认远程仓库类型
………………

async checkGitOwner(){
        const ownerPath =this.createPath(GIT_OWN_FILE) ;
        const loginPath =this.createPath(GIT_LOGIN_FILE) ;
        let owner = readFile(ownerPath)
        let login = readFile(loginPath)
        if(!owner || !login || this.refreshOwner){
            owner = (await inquirer.prompt({
                type:'list',
                name:'owner',
                message:'请选择远程仓库类型',
                default: REPO_OWNER_USER,
                choices:this.orgs.length > 0 ? GIT_OWNER_TYPE : GIT_OWNER_TYPE_ONLY
            })).owner
            if(owner === REPO_OWNER_USER){
                login = this.user.login
            }else{
                login = (await inquirer.prompt({
                    type:'list',
                    name:'login',
                    message:'请选择',
                    choices:this.orgs.map(item =>({
                        name:item.login,
                        value: item.login,
                    }))
                })).login
            }
            writeFile(ownerPath,owner)
            writeFile(loginPath,login)
            log.success('owner 写入成功',`${owner} -> ${ownerPath}`)
            log.success('login 写入成功',`${login} -> ${loginPath}`)
        }else{
            log.success('owner 读取成功',`${owner} -> ${ownerPath}`)
            log.success('login 读取成功',`${login} -> ${loginPath}`)
        }
        this.owner = owner
        this.login = login
    }
```

### 第四章 GitFlow 初始化流程开发

---

#### 4-1 Gitee获取和创建仓库API接入
> - 这节的代码，出了一个小bug，调试到了天亮,bug方法的实现为下面示例50行：this.handleResponse(response)，课程代码讲到在获取一个仓库的API时没有status参数，经测试是有的。
> - 本节**主要**完成的功能有：
>    -  检查并创建远程仓库 **checkRepo **方法实现
>    - GiteeRequest添加post请求
>    - Gitee类实现 **createRepo()**和 **getRepo()** 方法

```javascript
await this.checkRepo(); //  检查并创建远程仓库

//models/git/lib/index.js
async checkRepo(){
  let repo = await this.gitServer.getRepo(this.login,this.name)
  log.verbose('repo',repo)
  if(!repo){ //如果远程仓库不存在，就去创建
    let spinner = spinnerStart('开始创建远程仓库')
    try {
      if(this.owner === REPO_OWNER_USER){
        repo = await this.gitServer.createRepo(this.name)
        log.success('用户个人远程仓库创建成功！')
      }else{
        this.gitServer.createOrgRepo(this.name,this.login)
        log.success('用户组织远程仓库创建成功1')
      }
    } catch (error) {
      log.error(error)
    }finally {
      spinner.stop(true)
    }
    if(!repo){
      throw new Error('远程仓库创建失败')
    }
  }else{
    log.success('远程仓库已存在且获取成功！')
  }
  this.repo = repo
}

//models/git/lib/GiteeRequest.js
 post(url,data,headers){
   return this.service({
     url,
     params:{
       access_token:this.token,
     },
     data,
     method:'POST',
     headers
   })
 }

//models/git/lib/Gitee.js
getRepo(login,name){
  //GET https://gitee.com/api/v5/repos/{owner}/{repo}
  return this.request
    .get(`/repos/${login}/${name}`)
    .then(response =>{
    return this.handleResponse(response)
  })
}
createRepo(name){
  // POST https://gitee.com/api/v5/user/repos
  return this.request.post('/user/repos',{
    name,
  })
}
```

#### 4-2 Github获取和创建仓库API接入
> 与Gitee获取和创建仓库API类似。GithubRequest同样实现了post方法。
> 类Github同样实现了getRepo和createRepo方法。

#### 
#### 4-3 Github&Gitee组织仓库创建API接入
> 本节内容较为简单，实现了远程创建组织仓库API

```javascript
createOrgRepo(name,login){
  return this.request.post(`/orgs/${login}/repos`,{
    name
  },{
    accept:'application/vnd.github.v3+json'
  })
```
#### 4-4 gitignore文件检查
> 提交准备工作：有些项目没有默认创建.gitignore,因此会引发提交大量无用或无关代码。
> 因此，我们需要检查并创建.gitignore文件的方法
> 这里需要注意的是安装的.gitignore安装目录为当前执行文件，而不是缓存文件

```javascript
const GIT_IGNORE_FILE='.gitignore'

this.checkGitIgnore();//检查并创建.gitignore文件

checkGitIgnore(){
        const gitIgnorePath = path.resolve(this.dir,GIT_IGNORE_FILE)
        console.log(gitIgnorePath)
        if(!fs.existsSync(gitIgnorePath)){
            writeFile(gitIgnorePath,`.DS_Store
node_modules
/dist


# local env files
.env.local
.env.*.local

# Log files
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Editor directories and files
.idea
.vscode
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?
`)
            log.success(`自动写入${GIT_IGNORE_FILE}文件成功！`)
        }
    }
```
#### 4-5 git本地仓库初始化和远程仓库绑定

> 本节主要完成的功能为本地的仓库的初始化：即执行git init方法和git addRemote方法。

```javascript
await this.init(); //完成本地仓库初始化

 async init(){
   if(await this.getRemote()){
     return
   }
   await this.initAndAddRemote();
 }

async initAndAddRemote(){
  log.info('执行git初始化')
  await this.git.init(this.dir)
  log.info('添加git remote')
  const remotes = await this.git.getRemotes();
  console.log('git remotes',remotes)
  if(!remotes.find(item => item.name === 'origin')){
    await this.git.addRemote('origin',this.remote)
  }
}
async getRemote(){
  const gitPath = path.resolve(this.dir,GIT_ROOT_DIR)
  this.remote = this.gitServer.getRemote(this.login,this.name)
  if(fs.existsSync(gitPath)){
    log.success('git已完成初始化')
    return true
  }
}
```

#### 4-6 git自动化提交功能开发
> 上一节的流程在本地实现了两个操作
> - **git init** 
> - **git remote add origin 'git@github.com:${login}/${name}.git'**
> 
紧接着这一节按照本地的操作，我们应该实现  git add. / git commit -m'的操作。
> 本节实现initCommit()方法：
> - 首先检查是否有代码冲突
> - 然后检查代码是否有未提交
> - 然后判断远程分支是否已存在
>    - 不存在的话直接push代码
>    - 存在的话就需要使用git pull去拉取代码，且使用 '--allow-unrelated-histories:all':null 参数。
> 

```javascript
 async initCommit(){
   await this.checkConflicted(); //检查代码冲突
   await this.checkNotCommitted();//检查代码未提交
   if(await this.checkRemoteMaster()){ //判断远程仓库master分支是否已存在
     await this.pullRemoteRepo('master',{
       '--allow-unrelated-histories':null
     })
   } else {
     await this.pushRemoteRepo('master')  //如果不存在直接push代码
   }

 }
async checkConflicted(){
  log.info('代码冲突检查')
  const status = await this.git.status()
  if(status.conflicted.length > 0 ){
    throw new Error('当然代码存在冲突，请手动处理合并后再试')
  }
  log.success('代码冲突检查通过')
}

async checkNotCommitted(){
  const status = await this.git.status()
  if(status.not_added.length >0 || 
     status.created.length >0 ||
     status.deleted.length>0 ||
     status.modified.length>0 ||
     status.renamed.length>0
    ){
    log.verbose('status',status)
    await this.git.add(status.not_added)
    await this.git.add(status.created)
    await this.git.add(status.deleted)
    await this.git.add(status.modified)
    await this.git.add(status.renamed)
    let message;
    while (!message) {
      message = (await inquirer.prompt({
        type:'text',
        name:'message',
        message:'请输入commit信息'
      })).message
    }
    await this.git.commit(message)
    log.success('本次commit提交成功！')
  }
}
async checkRemoteMaster(){
  // git ls-remote
  return (await this.git.listRemote(['--refs'])).indexOf('refs/heads/master') >=0
}
async pushRemoteRepo(branchName){
  log.info(`推送代码至${branchName} 分支`)
  await this.git.push('origin',branchName)
  log.success('推送代码成功！')
}
async pullRemoteRepo(branchName,options){
  log.info(`同步远程${branchName}分支代码`)
  await this.git.pull('origin',branchName,options)
    .catch(err=>{
    log.error(err.message)
  })
}
```

### 第五章 GitFlow本地仓库代码自动提交

---

#### 5-1 自动生成开发分支原理讲解1
> 第一遍：这节课听的有点懵逼。
> 

[点击查看【processon】](https://www.processon.com/embed/610f8ef87d9c082be335ac23)

#### 5-2 自动生成开发分支功能开发

> 本节主要实现为 获取远程发布分支列表(git ls-remote --refs)和获取远程最新发布分支号(通过正则匹配release分支，并排序获取最新分支)，详细代码如下：

```javascript
const semver = require('semver')

const VERSION_RELEASE = 'release'
const VERSION_DEVELOP = 'dev'

 async commit(){
        // 1.生成开发分支
        await this.getCorrectVersion()
        // 2.在开发分支上提交代码

        // 3.合并远程开发分支

        //4.推送开发分支
    }

    async getCorrectVersion(type){
         // 1.获取远程发布分支
         // 规范：release/x.y.z ,dev/x.y.z
         // 版本号递增规范：major/minor/patch
         log.info('获取远程仓库代码分支')
         const remoteBranchList = await this.getRemoteBranchList(VERSION_RELEASE)
         let releaseVersion = null;
         if(remoteBranchList && remoteBranchList.length>0){
             releaseVersion = remoteBranchList[0]
         }
         log.verbose('releaseVersion',releaseVersion)
    }

    async getRemoteBranchList(type){
        const remoteList = await this.git.listRemote(['--refs'])
        let reg;
         if(type === VERSION_RELEASE ){
            reg = /.+?refs\/tags\/release\/(\d+\.\d+\.\d+)/g
         }else{
         }
        return remoteList.split('\n').map(remote =>{
            const  match = reg.exec(remote)
            reg.lastIndex = 0
            if(match &&semver.valid(match[1]) ){
                return match[1]
            }
        }).filter(_ => _ ).sort((a,b) => {
            if(semver.lte(b,a)){
                if(a===b) return 0;
                return -1
            }
            return 1
        })
        
    }
```

#### 5-3 高端操作：自动升级版本号功能开发
> 根据5-1图示，上两节我们完成的部分为：获取远程发布分支号列表、获取远程最新发布分支号，并在上节代码中经过处理，拿到了最新的远程发布的版本号，接下来我们实现
> - 判断最新发布版本号是否存在
>    - 不存在：生成本地开发分支
>    - 存在：与本地开发分支版本号通过semver对比
>       - 本地分支小于远程最新发布分支版本号
>          - 通过inquirer询问选择本地版本的升级方式
>          - 获取选择升级的版本号
>          - 重新写入到本地package.json中的version中去
>       - 本地分支大于远程最新发布分支版本号

```javascript
  this.branch = null //本地开发分支


//接着上一节的代码，在getCorrectVersion方法中继续：
 //2.生成本地开发分支
const devVersion = this.version
if(!releaseVersion){  // 不存在远程发布分支
  this.branch = `${VERSION_DEVELOP}/${devVersion}`
}else if(semver.gt(this.version,releaseVersion)){ //本地分支大于远程发布分支
  log.info('当前版本大于线上最新版本',`${devVersion} >= ${releaseVersion}`)
  this.branch = `${VERSION_DEVELOP}/${devVersion}`
}  else {
  log.info('当前线上版本大于本地版本',`${releaseVersion} > ${devVersion}`)
  const incType = (await inquirer.prompt({
    type:'list',
    name:'incType',
    message:'自动升级版本，请选择升级版本',
    default:'patch',
    choices:[{
      name:`小版本(${releaseVersion} -> ${semver.inc(releaseVersion,'patch')})`,
      value:'patch'
    },{
      name:`中版本(${releaseVersion} -> ${semver.inc(releaseVersion,'minor')})`,
      value:'minor'
    },{
      name:`大版本(${releaseVersion} -> ${semver.inc(releaseVersion,'major')})`,
      value:'major'
    }]
  })).incType
  const incVersion = semver.inc(releaseVersion,incType)
  this.branch = `${VERSION_DEVELOP}/${incVersion}`
  this.version = incVersion
}
log.verbose('本地开发分支',this.branch)
//3.将version同步到package.json
this.syncVersionToPackageJson()


 syncVersionToPackageJson(){
   const pkg = fse.readJsonSync(`${this.dir}/package.json`)
   if(pkg && pkg.version!== this.version){
     pkg.version = this.version
     fse.writeJsonSync(`${this.dir}/package.json`,pkg,{spaces:2})
   }
 }
```
#### 
5-4 GitFlow代码自动提交流程梳理+stash区检查功能开发

[点击查看【processon】](https://www.processon.com/embed/610fe95c6376891eb94d7e38)

> 本地执行git status 有未提交的代码时，执行 git stash将未提交的代码缓存在stash区当中。
> 然后通过git status命令发现，没有代码可提交
> 这里温习了git stash的个命令：git stash / git stash list / git stash pop

本节代码实现
```javascript
async checkStash(){
  //1. 检查stash list
  const stashList = await this.git.stashList()
  if(stashList.all.length >0){
    await this.git.stash['pop']
    log.success('stash pop成功')
  }
}
```

#### 5-5 代码冲突处理+Git代码删除后还原方法讲解
> 本节以及上一节听的有些懵逼，需要第二遍重新学习


#### 5-6 自动切换开发分支+合并远程分支代码+推送代码功能开发
> 先暂时略过笔记。



### 第六章 本周加餐：Node编码最佳实践

---

#### 6-1 Node最佳实践学习说明
> - 通过[https://github.com/goldbergyoni/nodebestpractices/blob/master/README.chinese.md](https://github.com/goldbergyoni/nodebestpractices/blob/master/README.chinese.md)这个库学习。
> - 需要有node的实践。


#### 6-2 Node项目架构最佳实践
#### 6-3 Node异常处理最佳实践

**代码示例: 捕获 unresolved 和 rejected 的 promise**
```javascript
process.on('unhandledRejection', (reason, p) => {
  //我刚刚捕获了一个未处理的promise rejection, 因为我们已经有了对于未处理错误的后备的处理机制（见下面）, 直接抛出，让它来处理
  throw reason;
});
process.on('uncaughtException', (error) => {
  //我刚收到一个从未被处理的错误，现在处理它，并决定是否需要重启应用
  errorManagement.handler.handleError(error);
  if (!errorManagement.handler.isTrustedError(error))
    process.exit(1);
});


```
#### 6-4 Node编码规范最佳实践
#### 6-5 Node测试+安全最佳实践













