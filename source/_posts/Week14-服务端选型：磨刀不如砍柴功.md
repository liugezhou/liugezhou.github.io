---
layout: post
title: Week14-服务端选型：磨刀不如砍柴功
author: liugezhou
date: 2021-03-09
category: 
    Web架构之脚手架
tags:
    Web架构之脚手架
---
### 第一章 周介绍

---

#### 1-1 本周介绍
> - 服务端选型：所有技术为业务服务
> - nodejs框架选型：Koa2
> - 数据库：Mysql  Mongodb Redis
> - 登录校验：JWT
> - 单元测试与接口测试：Jest
> - 线上服务：PM2 + nginx


### 第二章 选择nodejs框架

---

#### 2-1 nodejs框架选型-开始
> - 所用常见的nodejs框架中，Koa2是最简单、最小的
> - 目的扩充广度，让你了解有这门技术
> - Koa2和Express
> - eggs.js
> - Nest.js

#### 2-2 介绍koa2和express
> - [koa2](https://koa.bootcss.com/):  基于Node.js平台的下一代web框架
> - [express](https://expressjs.com/zh-cn/starter/generator.html):node平台web框架，koa2基于express。

#### 2-3 介绍egg.js
> [egg.js](https://eggjs.org/zh-cn/):阿里开源，基于Koa2封装。

#### 2-4 介绍nest.js
> [nest.js](https://docs.nestjs.cn/)：也是一个框架，默认基于express封装，比较小众。
> 使用ts语法，大量使用装饰品，学习成本高。


### 第三章 数据库使用 Mysql Mongodb 和 Redis

---

#### 3-1 章开始
> 这一章会介绍：
> Mysql和Sequelize
> Mongodb和Mongogoose
> Mysql和Mongodb的区别
> Redis。

#### 3-2 回顾数据结构设计
> 对第一周内容做了个简单回顾

#### 3-3 Mysql 和 Sequelize 1
学习这节之前，先将代码clone到本地，代码地址：[https://github.com/liugezhou/lego_node_server](https://github.com/liugezhou/lego_node_server)

> 1. mysql是Web应用中最常见的关系型数据库
> 1. 本地安装mysql：Navicate Premium
> 1. 本地新建数据库 imooc_lego_course,使用mysql2测试数据库连接。新建路由 /api/db-check，用于展示结果。
> 
a.修改src/conf/envs/dev.js中的mysqlConf为本地

```javascript
module.exports = {
	mysqlConf : {
  	host: 'localhost',
    user: 'root',
    password: 'liugezhou1205',
    port: '3306',
    database: 'imooc_lego_course',
  }
}
```
> b. 测试：node src/db/mysql2.js

```javascript
const mysql = require('mysql2/promise')
const { mysqlConf } = require('../config/dev')   //这里加载dev

async function testMysqlConn () {
	const conn = await mysql.createConnection(mysqlConf)
  const [rows] = await conn.execute('select now()')
  return rows
}

testMysqlConn().then(res => console.log(res)
```
#### 

#### 3-4 Mysql 和 Sequelize 2
> Sequelize:最常用的ORM框架，它让开发者不用写繁琐的SQL语句，通过API即可操作数据库。
> 1. npm i -S sequelize  require-all  simple-git
> 1. 数据库连接测试文件：

```javascript
// src/db/seq/utils/conn-test.js
const seq = require('../seq')

seq.authenticate()
	.then(()=>{
		console.log('ok')
	})
	.catch(()=>{
		console.log('fail')
	})
	.finally(()=>{
		process.exit()
	})
```
```javascript
// src/db/seq/seq.js
const Sequelize = require('sequelize')
const {mysqlConf } = require('../config/dev') 
const { isPrd, isTest } = require('../../utils/env')

//连接配置
const { database, user, password, host, port } = mysqlConf
const conf = {
	host,
  port,
  dialect:'mysql'
}

//测试环境不打印日志
if(isTest){
	 conf.logging = () => {} // 默认是 console.log
}

// 线上环境用 链接池
if (isPrd) {
    conf.pool = {
        max: 5, // 连接池中最大连接数量
        min: 0, // 连接池中最小连接数量
        idle: 10000, // 如果一个线程 10 秒钟内没有被使用过的话，那么就释放线程
    }
}

// 创建连接
const seq = new Sequelize(database, user, password, conf)

module.exports = seq
```
> 3. 数据库模型：models

```javascript
// userModel
// src/models/UserModel.js
const seq = require('../db/seq/seq')
const { STRING, DATE, BOOLEAN } = require('../db/seq/types')

const User = seq.define('user', {
    username: {
        type: STRING,
        allowNull: false,
        unique: 'username', // 不要用 unique: true, 
        comment: '用户名，唯一',
    },
    password: {
        type: STRING,
        allowNull: false,
        comment: '密码',
    }
})

module.exports = User

```
> 4. 模型和数据表的同步
> 
该代码逻辑在  bin/www中，通过www代码我们直到，数据表同步功能在sync-alter中

```javascript
#!/usr/bin/env node

………………

const syncDb = require('../src/db/seq/utils/sync-alter')

………………

// 先同步 mysql 数据表
syncDb().then(() => {
    // 再启动服务
    server.listen(port)
    ………………
})

```

> src/db/seq/utils/sync-alter

```javascript

const path = require('path')
const simpleGit = require('simple-git')
const seq = require('../seq')
const { isDev } = require('../../../utils/env')

// 获取所有 seq model
require('require-all')({
    dirname: path.resolve('src', 'models'), // src/models 中可能会有 mongoose 的 model ，不过这里获取了也没关系
    filter: /\.js$/,
    excludeDirs: /^\.(git|svn)$/,
    recursive: true, // 递归
})

// 同步数据表
async function syncDb() {
    let needToSyncDb = true
    // 只适用于开发环境！！！
    if (isDev) {
      ………………
        if (fileChanged.length) {
          …………
            if (!changedDbFiles) needToSyncDb = false
        }
    }

    if (needToSyncDb) {
        await seq.sync({ alter: true })
    }
}

module.exports = syncDb

```
> 上面代码的一些逻辑总结为一句话：seq.sync({ alter: true })


#### 3-5 Mongodb和Mongoose
> - Mongodb是Web应用中最常见的NoSQL应用。
> - 本地在mongodb数据库中新建imooc_lego_course数据库，以及集合work。
> - 然后，修改代码配置
> - 接着，连接测试

```javascript
// src/config/envs/dev.js

mongodbConf: {
  host: 'localhost',
  port: '27017',
  dbName: 'imooc_lego_course',
},
```
```javascript

const mongoose = require('mongoose')
const { mongodbConf } = require('../config/index')

const { host, port, dbName, user, password } = mongodbConf

// 拼接连接字符串
let url = `mongodb://${host}:${port}` // dev 环境
if (user && password) {
    url = `mongodb://${user}:${password}@${host}:${port}` // prd 环境
}

mongoose.set('useCreateIndex', true)
mongoose.set('useFindAndModify', false)

// 开始连接（ 使用用户名和密码时，需要 `?authSource=admin` ）
mongoose.connect(`${url}/${dbName}?authSource=admin`, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
})

// 连接对象
const db = mongoose.connection

db.on('error', err => {
    console.error('mongoose connect error', err)
})

 // 演示注释掉即可
 db.once('open', () => {
     // 用以测试数据库连接是否成功
     console.log('mongoose connect success')
 })


```

> 再接着，新建数据库模型model--- work，[通过Schema生成一个model]

```javascript
/**
 * @description 作品内容 Model ，存储到 Mongodb
 * @author liugezhou
 */

const mongoose = require('../db/mongoose')

// 两个 model 公用一个 schema
const contentSchema = mongoose.Schema(
    {
        // 页面的组件列表
        components: [Object],
        // 页面的属性，如页面背景图片
        props: Object,
        // 配置信息，如微信分享配置
        setting: Object,
    },
    { timestamps: true }
)

// 未发布的内容
const WorkContentModel = mongoose.model('workContent', contentSchema)

// 发布的内容
const WorkPublishContentModel = mongoose.model('workPublishContent', contentSchema)

module.exports = {
    WorkContentModel,
    WorkPublishContentModel,
}

```
> 最后，我们在进行mysql与mongoose的测试的时候，在routes/index.js中将有关redis的内容暂时注释，
> 然后执行：npm run start，出现下面则测试成功![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1615281020589-db065d89-095b-4962-9dd5-17c296d3a937.png#height=310&id=xapI9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=620&originWidth=1054&originalType=binary&size=112212&status=done&style=none&width=527)
> 这里，讲师再次推荐了自己的一个课程，鉴于此次购买课程自己不是很满意，这里，我觉得自己补充mongoose的基础知识就够了，总结至：[https://www.yuque.com/liugezhou/gofftg/zanx0w](https://www.yuque.com/liugezhou/gofftg/zanx0w)


#### 3-6 Date 和时区
> new Date()直接打印，会显示世界标准时间，和北京差8个时区，要想获得当前时间，只需要toString()即可。


> 在Docker虚拟机里，默认没有时区，需要在Dockerfile里面进行配置

```javascript
# Dockerfile
FROM node:14
WORKDIR /app
COPY . /app

# 设置时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone

CMD npm i && npm run prd-dev && npx pm2 log

```
#### 
#### 3-7 Mysql和Mongodb的区别
> Mysql:关系型数据库，用于存储表格形式，格式规整的数据
> Mongodb：文件数据库，用于存储文件，格式零散的数据。


#### 3-8 介绍Redis

> 在项目中：npm i -S redis
> 然后根据前面Mysql以及Mongodb的调试方法，调试出本地的redis显示。


> 课程中关于redis的其它内容依旧是给出实战课让自己去学习，其它的什么也没说，而我本地也是安装过redis的，但是不记得如何启动了，于是我的步骤是这么展开的：
> 
> - 第一步：首先看本地的redis是否已删除，即查找本地安装redis证据
>    - brew info redis:本地显示not install
>    - 接着查看/usr/local/etc/下没有redis.conf文件
>    - 结论：我本地的redis已经被我删除了 [其实，没有删除，后面才清楚]
> - 第二步：安装redis
>    - brew install redis
>    - 在/usr/local/etc下多了两个配置文件：redis.conf和redis-sentinel.conf
>    - 启动redis：brew services start redis （这个命令会在后台启动redis服务，并且每一次登录系统，都会自动重启）
>    - 假如不需要后台启动服务，可以配置文件启动：redis-server /usr/local/etc/redis.conf
> 
> 我这里使用 redis-server /usr/local/etc/redis.conf的方式启动redis，
> 然后出现报错：
> ![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1615358249479-0170c9ee-507a-41f4-8ee9-1ee828c20a4b.png#height=196&id=H6RiQ&margin=%5Bobject%20Object%5D&name=image.png&originHeight=392&originWidth=1446&originalType=binary&size=113007&status=done&style=none&width=723)
> 接着查找错误，原因为配置错误,没有深究下去。
> 但是尝试了另一个启动命令  redis-server:
> ![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1615358605688-4e0632af-d62d-460f-b605-f5ec4beae02c.png#height=604&id=atM2B&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1208&originWidth=2456&originalType=binary&size=766762&status=done&style=none&width=1228)
> 成功了！
> 如图显示这个版本是5.0.8的，也就是说我之前电脑上其实是有redis的，我新安装的这个6.0.9的并没有用上。
> 
> 然后，我继续查看目录，发现我之前安装的5.0.8的版本,其实在  /usr/local/redis-5.0.8下面,而且我不是使用的brew安装的
> 
> 因此，我又把刚刚安装的redis删除：
> brew uninstall redis
> rm -rf /usr/local/etc/redis.conf
> rm -rf /usr/local/etc/redis-sentinel.conf


> 折腾了这么一趟，其实我开始的时候直接redis-server启动就可以了。
> 此时在第三章3-3 clone的代码基础上，加入了redis配置后，执行npm run dev  发现redis连接成功了！


#### 3-9 章总结
> 啰里八嗦后一句话：大家自己去学习基础知识，从头开始讲不可能。


### 第四章 登录校验并使用JWT

---

#### 4-1 开始
> 选择JWT，放弃Session。
> - Cookie和Session
> - JWt
> - SSO和OAuth2


#### 4-2 介绍 Session 登录
Cookie做登录校验的过程
> - 前端传入用户名密码，传给后端
> - 后端验证成功，返回信息时set-cookie
> - 接下来所有接口访问，都自动带上cookie

Session
> - cookie只存储用户userid，不暴露用户信息，session存储用户信息。
> - Session原理简单、易于学习
> - 用户信息存储在服务端，可以快速封禁某个登录的用户
> - 但是： 占用服务端内存、多进程、多服务、跨域传递cookie


#### 4-3 介绍JWT登录
> JWT -- Json Web Token


JWT过程
> - 前端输入用户名密码，传给后端。
> - 后端验证成功，返回一段token字符串----将用户信息加密得到。
> - 前端获取token之后，存储起来。
> - 以后访问接口，都在header中带上token。

优缺点
> 优点：不占用服务器内存、多进程,多服务器,不受影响、不受跨域限制
> 缺点：无法快速封禁登录的用户。

区别
> Session用户信息存储在服务端
> JWT用户信息存储在客户端

代码演示
> 首先需要第三方库：koa-jwt 和 jsonwebtoken
> 然后，简单对jwt以及loginCheck中间价进行了一个介绍,下面是jwt代码演示，loginCheck不贴了。

```javascript
/**
 * @description 封装 jwt 插件
 * @author liugezhou
 */

const jwtKoa = require('koa-jwt')
const { JWT_SECRET, JWT_IGNORE_PATH } = require('../config/constant')

module.exports = jwtKoa({
    secret: JWT_SECRET,
    cookie: 'jwt_token', // 使用 cookie 存储 token
}).unless({
    // 定义哪些路由忽略 jwt 验证
    path: JWT_IGNORE_PATH,
})

```
#### 4-4 SSO和OAuth2
> SSO:单点登录
> OAuth2第三方鉴权的常用方式

**使用Cookie实现**
> 简单的，如果业务系统都在同一主域名下，比如wenku.baidu.com  tieba.baidu.com，就好办了。
> 可以直接把cookie domain设置为主域名 baidu.com。

**SSO**
> 复杂一点，滴滴同时拥有 didichuxing.com xiaojukeji.com didiglobal.com等域名
> 各种cookie是完全绕不开的。

**OAuth2验证**
> 上述SSO是oauth的实际案例，其他常见的还有微信登录、github登录。即，当涉及到第三方用户登录校验时，都会用到OAuth2.0标准。


#### 4-5 章总结
> Cooike/Session/Jwt/OSS/OAuth2


### 第五章 单元测试选择 Jest

---

#### 5-1 开始
> 保证软件质量：单元测试和接口测试。
> - Jest 和Mocha
> - 单元测试为何难以落实
> - supertest接口测试
> - 测试驱动开发TDD

#### 5-2 介绍Jest和Mocha
> Jest官网：[https://jestjs.io/zh-Hans/docs/getting-started](https://jestjs.io/zh-Hans/docs/getting-started)
> Mocha官网：[https://mochajs.cn/#getting-started](https://mochajs.cn/#getting-started)

代码演示
> - 安装jest:npm i -S jest
> - 配置package.json

```javascript
 "scripts": {
 	 "test:local": "cross-env NODE_ENV=test_local jest --runInBand  --passWithNoTests --colors --forceExit",
 }
```
> - __test__/demo.test.js

```javascript
describe('test demo', () => {
    test('1+1 = 2', () => {
        expect(1 + 1).toBe(2)
    })
})
```
允许命令 npm run test:local
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1615380075667-3717ed7a-d4df-4fb3-811c-a54405a40a68.png#height=170&id=TKiJF&margin=%5Bobject%20Object%5D&name=image.png&originHeight=340&originWidth=1202&originalType=binary&size=58146&status=done&style=none&width=601)
#### 5-3 为何单元测试难以落实
> **使用方式不合理**：混淆了单元测试和集成测试，导致单元测试代码中有太多Mock。如果需要服务器启动才能执行的代码，就不是单元测试了。
> **现状：**研发流程不规范

#### 5-4 supertest接口测试
> supertest接口测试的目的是让所有接口稳起来。
> 本地测试： jest + supertest
> 远程测试： jest + axios
> 接口测试和单元测试，代码都放在 __test__下，但两者概念要区分开。

代码演示：
> 安装 supertest axios
> package.json中添加 test:remote配置（远程才用到）
> 接口测试目录：__test__/api/


### 第六章：线上服务使用PM2和nginx

---

#### 6-1 pm2和nginx-章开始
> 线上服务：稳定和高效

#### 6-2 pm2配置和使用
> 根据我之前的学习理解：pm2其实就是一个后台服务常驻的一个工具，我们平时在npm run dev后如果按Ctrl + c 停止后，服务就停止了，如果我们使用 pm2来启动，那么即使停止，我们的项目还是能够继续运行。
> 
> 另外，我本地正在开发一个vue项目，如果我想后台常驻，那么我可以直接执行：pm2 start npm -- run serve
> 我直接这么执行的话，那本地肯定会产生log日志文件，我在/Users/liumingzhou/.pm2目录下找到了日志打印。

**特点：**
> - 进程守护--稳定
> - 多进程--高效
> - 日志记录--问题可追溯

**安装**
> npm i -g pm2

**基本使用**
> - pm2 start xxx.js
> - pm2 restart <id/name>
> - pm2 reload
> - pm2 list
> - pm2 logs <id/name>
> - pm2 stop <id/name>
> - pm2 delete <id/name>
> - pm2 monit

**配置**
```javascript
const os = require('os')
const cpuCoreLength = os.cpus().length // CPU 几核

module.exports = {
    apps: [
      {
        name: 'your-server-name',
        script: 'bin/www',
        // watch: true, // 无特殊情况，不用实时监听文件，否则可能会导致很多 restart
        ignore_watch: ['node_modules', '__test__', 'logs'],
        // instances: cpuCoreLength, // 线上环境，多进程
        instances: 1, // 测试环境，一个进程即可
        error_file: './logs/err.log',
        out_file: './logs/out.log',
        log_date_format: 'YYYY-MM-DD HH:mm:ss Z', // Z 表示使用当前时区的时间格式
        combine_logs: true, // 多个实例，合并日志
        max_memory_restart: '300M', // 内存占用超过 300M ，则重启,可使用 pm2 monit查看初始内存占用，然后根据初始设置
      }
    ],
}

```
> package.json配置：
> "prd-dev": "cross-env NODE_ENV=dev pm2 start bin/pm2-prd-dev.config.js"
> 运行：npm run prd-dev


#### 6-3 pm2日志拆分
> 日志拆分的原因为：日志庞大，不利于检索且占用内存太大。
> 日志拆分的方式有按天拆分或者小时等。
> 
> 我们这里日志拆分使用的是：**pm2-logrotate**
> 安装：pm2 install pm2-logrotate -g
> 运行 pm2 list 即可看到 pm2-logrotate的进程



**默认配置如下：**
$ pm2 set pm2-logrotate:max_size 10M # 日志文件最大 10M
$ pm2 set pm2-logrotate:retain 30 # 保留 30 个文件，多了就自动删掉
$ pm2 set pm2-logrotate:compress false # gzip 压缩文件
$ pm2 set pm2-logrotate:dateFormat YYYY-MM-DD_HH-mm-ss
$ pm2 set pm2-logrotate:workerInterval 30 # 单位 s ，日志检查的时间间隔
$ pm2 set pm2-logrotate:rotateInterval 0 0 * *_ _ * # 定时规则
$ pm2 set pm2-logrotate:rotateModule true # 分割 pm2 模块的日志
可修改配置 pm2 set pm2-logrotate: <key><value>
**rotateInterval规则[crontab]：**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1615385209912-7aa07171-1c40-44b6-8472-653b309df450.png#height=216&id=o3KKF&margin=%5Bobject%20Object%5D&name=image.png&originHeight=431&originWidth=1206&originalType=binary&size=84782&status=done&style=none&width=603)
#### 6-4 nginx配置和日志拆分
> - 静态服务
> - 反向代理
> - 负载均衡
> - access log

常用命令
> nginx
> nginx -s reload
> nginx -s stop
> nginx -t
> nginx -c xxx.conf


### 第七章 周总结

---

**开发环境配置**
> - eslint premitter
> - pre-commit
> - commit规范
> - validator
> - cors


### 第八章 nodejs框架：express

---

#### 8-1 安装
> 通过脚手架安装：express-generator
> - npm i express-generator -g
> - express express-test
> - cd express-test
> - npm i 
> - npm run start
> - 为了方便改代码后不用重启，我们使用  npm i nodemon cross-env --save-dev

#### 8-2 ｜8-3 介绍app-js
> - 各个插件的作用
>    - http-errors:错误页处理
>    - express
>    - cookie-parse：只要经过这个中间件处理，我们纠结可以非常轻松的使用req.cookie()去访问所有cookie
>    - morgan:记录access log
>    - app.use(express.json()):post请求传入的数据直接在route中使用req.body获取
>    - app.use(express.urlencoded({ extended: false }));：请求参数为application/x-www-form-urlencoded
> 
> - 处理get和post请求
>    - res.json()


#### 8-4 使用中间件
> app.use()
> next参数作用。


### 第九章：nodejs框架：koa2
#### 
#### 9-1 介绍koa2
> - npm install koa-generator -g
> - koa2 koa2-test
> - npm install && npm run dev

### 
### 第十章 mysql和Sequelize
> 关于表的外键：表关联，有一些外键的设置，我发现之前的后端表中都没有对外键盘做一个级联操作，于是在回头查看一些表结构的时候，就不容易看出来一些表的关联关系，如果我们在新建表的时候就去设置外键表的关联，首先表结构一目了然，且在新增(外键关联的主键没有值得时候)会有错误提示，删除主键表的时候，关联的主键内容也会删掉。

> select * from blogs inner join users on users.id =blogs.userid
> select blogs.*,users.username,users.nickname from blogs inner join users on users.id =blogs.userid

> sequelize:mysql链接：const seq = new  Sequeslize('koa2_weibo_db','root','liugezhou1205',{host:'localhost',dislect:'mysql'})
> 建模：
> 
> 数据库建表字段长度为255，varchar为可变长度，并不是会占用这么多的空间，数据库会自动计算缩短空间


### 第十一章 mongodb基础学习

---


#### 11-1 mongodb是文档数据库
> - Mongodb是一个文档数据库
> - Mongodb和Mysql Redis的对比
> - 如何选择？举例说明


##### 文档数据库
> - Mysql 以表格形式存储数据
> - Redis以 key-value形式存储数据
> - Mongodb是以文档形式存储数据，格式像JSON

##### 对比
| Mysql | 关系型    ｜表格存储               ｜ SQL操作 ｜ 硬盘 |
| --- | --- |
| Redis | 非关系型 ｜ key-value形式存储 ｜ NoSQL   ｜ 内存 |
| Mongodb | 非关系型 ｜文档存储               ｜ NoSQL   ｜ 硬盘 |

   
#### 11-2 安装mongodb--介绍
> - 安装mongodb服务端
> - 安装mongodb客户端
> - 启动和连接


#### 11-3 安装mongodb-mac-安装homebrew
> - 安装 homebrew
> - 用homebrew安装 mongodb
> - 安装客户端 compass

> 1. 安装brew官网：
> 
  /bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"](https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)")
> 2. 验证 brew --version
> 2. 切换源：查找资料即可(我本地未切换)


#### 11-4 安装mongodb-mac-安装mongodb
> - brew install mongodb-community
> - 启动：brew services start mongodb-community
> - 验证启动：mongo > 1 + 1 mongo
> - 停止：brew services start mongodb-community


#### 11-5 安装mongodb-mac-安装compass
> mongodb官网下载 composs


#### 11-6 compass操作mongodb
> 数据库--集合--文档

#### 
#### 11-7 用命令行操作mongodb
> - show dbs
> - use myblogs --新建或者使用数据库
> - show collections
> - db.blogs.insert({"title":"标题1","content":"内容1","author":"liugezhou"})
> - db.blogs.find()
> - db.blogs.find({"author":"liugezhou"})
> - db.blogs.update({"title":"标题1"},{$set:{"author":"lisi"}})
> - db.blogs.find().sort({_id:-1})

### 
#### 11-8 mongodb的几个重要概念
> - databse:一个应用对应多个数据库服务
> - collection
> - document
> - bson：类JSON格式，Binary JSON  二进制类型的JSON
> - NoSQL：无需sql语句查询

### 
#### 11-9 nodejs连接mongodb
> - mkdir mongodb-test
> - cd mongodb-test
> - npm init -y
> - npm i mongodb --save
> 

```javascript
const MongoClient = require('mongodb')

const url =  'mongodb://localhost:27017'
const dbName = 'myblog'

MongoClient.connect(
    url,
    {
        //配置
         useUnifiedTopology: true 
    },
    (err,client) =>{
        if(err){
            console.error('mongodb  connect error',err)
            return
        }
        // 连接成功
        console.log('mongodb connect success')

        // 切换数据库
        const db = client.db(dbName)
        //关闭连接
        client.close()
    }
)
```
#### 11-10 nodejs操作mongodb
```javascript
const MongoClient = require('mongodb')

const url =  'mongodb://localhost:27017'
const dbName = 'myblog'

MongoClient.connect(
    url,
    {
        //配置
         useUnifiedTopology: true 
    },
    (err,client) =>{
        if(err){
            console.error('mongodb  connect error',err)
            return
        }
        // 连接成功
        console.log('mongodb connect success')

        // 切换数据库
        const db = client.db(dbName)

        //集合
        const userCollection = db.collection('users')

        //删除
        userCollection.deleteOne({
            username:'mongodb'
        },(err,result)=>{
                if(err){
                    console.log('user delete error', err)
                    return
                }
                console.log(result)
                client.close()
            }
        )
        //修改
        userCollection.updateOne({
            username:'mongodb'
        },{$set:{username:'mongoose'}},(err,result)=>{
            if(err){
                console.log('user update error', err)
                return
            }
            console.log(result)
            client.close()
        })

        // 新增
        userCollection.insertOne({
            username:'mongodb',
            password:'abc',
            nickname:'mongoose'
        },(err,result) =>{
            if(err){
                console.log('user insert error', err)
                return
            }
            console.log(result)
            client.close()
        })
        // 文档--查询
        userCollection.find().toArray((err,result) => {
            if(err){
                console.log('user find error', err)
                return
            }
            console.log(result)
              //关闭连接
            client.close()
        })
    }
)
```

#### 11-11 使用mongoose连接mongodb服务
> - Schema定义数据格式的规范
> - 以Model规范Collection
> - 规范数据操作的APi

```javascript
const mongoose = require('../db')

// 用Schema定义数据规范
const UserSchema = mongoose.Schema({
    username: {
        type:String,
        required:true,
        unique:true
    },
    password: String,
    nickname: String
})

// Model对应 collection
const User = mongoose.model('user',UserSchema)

module.exports = User
```
#### 11-12 mongoose操作mongodb
```javascript
const Blog = require('../models/Blog')

!(async  ()=>{
    // 创建博客
    // const blog = await Blog.create({
    //     title:'liugezhou4',
    //     content:'内容4',
    //     author:'mongoose'
    // })
    // console.log(blog)

    // 查询
//    const blogList =  await Blog.find({author:'mongoose'}).sort({_id:-1})
//    console.log(blogList)

   //修改
   const res = await Blog.findOneAndUpdate(
       { title:'liugezhou'  },
       {content:'new content'},
       {new:true}    
   )
   console.log(res)
})()
```

### 第十二章 redis

---

#### 12-1 介绍redis基本使用
> - 内存数据库：比硬盘快很多很多。
> - 公共数据可以使用redis做缓存
> - 登录信息
> - brew install redis
> - 启动：redis-server
> - 客户端启动：redis-cli
> - set name 'liugezhou'
> - get name


#### 12-2 介绍redis-nodejs操作redis-1
```javascript
const redis = require('redis')
const { REDIS_CONF } = require('../conf/db')

// 创建客户端
const redisClient = redis.createClient(REDIS_CONF.port,REDIS_CONF.host)

redisClient.on('error', err=>{
    console.log('redis error', err)
})
```
#### 
#### 12-3 介绍redis-nodejs操作redis-2
> 没什么印象深刻的


- 服务器--如何查看redis安装在哪个目录
