---
layout: post
title: Week15-服务端 CI_CD：Github 自动化
date: 2021-03-15
updated: 2022-05-04
description: 服务端 CI_CD：Github 自动化
toc: true
category: 
    Web架构之脚手架
tags:
    Web架构之脚手架
---

<font color=blue>更新说明：对文章目录排版做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

## 第一章：周介绍

**1-1 介绍**

服务端 CI/CD流程--让 github 自动化为我们服务
> CI:    Continuous Integration  持续集成
> CD:   Continuous Delivery     持续交付
> 合理全面的 CI/CD，自动化研发流程，提高研发效率，增加系统稳定性


**收获**
> - 使用 Github actions 进行 CI/CD
> - 学会 Docker 在 nodejs 中的应用
> - 搭建测试环境

关键词
> - CI/CD
> - Github actions：实现 CI/CD 的一个工具
> - Docker Docker-compose

链接：[CI/CD 介绍](https://www.redhat.com/zh/topics/devops/what-is-ci-cd)
## 第二章 Github actions

> 这一章双越讲的真的不知道讲了个啥，自己课下补吧，真是一塌糊涂。
> github actions 的学习确实很有必要啊，回头等学习完毕再来吐槽。
> 学习阮一峰大哥的这节文档：http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html

**第二遍视频笔记记录如下**
**2-1 章介绍**
> - Github actions 是github 官方发布的一个产品 。
> - 使用 Github actions 自动化构建和测试
> - 认识 Github actions
> - 注意：接口测试会依赖于测试机搭建。
> - 二八法则。
> - 疑问：为了主流程跑通，不让边角东西打扰我们主流程，难道不注释掉那些代码就不能演示吗？后面再接上，这里的我要搞明白为什么在讲课代码演示的时候，是否为了讲师自己方便注释划水讲课。
> - 疑问二：既然不是讲 Github actions 和 Docker 的一门课，又为什么抽出一周的时间来划水(老师的答案可能是后面确实是用到这个知识了，有必要了解一下，那我的疑问又来了，既然用到了，又讲到了，那肯定默认这部分内容是很重要的，作为一门架构课，是不是应该认真对待每一周每一节课的录制，即使不那么深入，起码基础的内容得讲明白，这是必须的吧)
> - 疑问三：课程是以业务为导向，不可能把全部细节都讲出来，这个无可厚非，没有毛病，那你好歹把基础的大路边的内容，起码普及个差不多吧。
> - 疑问四：这节章介绍内容，你讲解那么多二八法则是干嘛？讲解什么边际效应，是为了后续课程划水先找借口吗？

**2-2 认识 Github actions**
> - 00:00-01:00：由于中美时差造成的 Github 不稳定问题。
> - 01:00-01:50：讲解了githu被微软收购，变得更好些，有了私有化个人项目，对中小企业越来越友好
> - 01:50-02:10:讲解了课程为什么代码不公开的原因是一些数据的线上原因，此事说过好几遍，
>    - (疑问:有敏感数据，代码公开直接将重要数据脱敏这个方案是否可行？又是否因为写代码的课程录制繁琐而不公开仓库)
> - 02:10-04:15: 链接一介绍：进入一个项目，讲解如何查找 actions，以及 actions 下面的页面展示，得出的结论：帮助你在项目根目录下新建.github/workflow/xxx.yml 文件。
> - 04:16-06:20:链接二介绍，官方翻译版本，挑选了 node.js 发布中的命令 npm install,npm publish，以及 secrets 的点名。
> - 06:20-07：00 :拿出代码，介绍有这个文件，然后我们介绍这个文件前，先去看应用场景。
> - 07:00-08:50:应用场景、范围介绍，打开开源的项目，介绍有三个文件名 yml，test.yml 对应master 分支，自动化测试，dev 分支即deploy-dev.yml--自动部署到测试机，v.x.x.x格式的 tag，自动上线--deploy-pro.yml
> - 08:50- 17:13 :代码演示
>    - 08:50-09:10: 讲解注释，去掉注释，添加注释
>    - 09:10-10:15: 中心思想为讲解 name 的命名要语义化    
>       - (补充：name 可以省略，省略的话，默认以文件名命名，还有一点演示过程中，yml 文件名称改为 demo，yml 文件内容也更改为demo，会让人误以为这个 name 的命名必须以文件名字命名，其实不是，文件的命令与文件内容中 name 的命名没有关联)
>    - 10:15-12:24: on/push/branches/paths的讲解，其中 paths 讲解可以简练点，讲的啰嗦了
>       - (补充：on字段可以是事件数组比如 on:[push,pull_request]]),branches可以限定分支或**标签**）
>    - 12:24-17:13:jobs 讲解。
>       - （补充：runs-on 没什么特殊情况下直接使用 ubuntu-latest,还有可以设置的比如windows-latest，macOS-latest，steps 中 uses 中的 actions/checkout@v2,实际上代表的是github.com/actions/checkout 这个仓库，actions/xxx 中的东西，都是仓库中的内容，GitHub 官方的 actions 都放在 [github.com/actions](https://github.com/actions) 里面）

**2-3 Github actions 功能演示**
> - 00:00-00:50：run为自定义命令。如果为多个命令，则格式为 run: | 。
> - 00:50-02:00：演示 run 命令 touch、echo、cat。
> - 03:00-03:50：代码提交--将branch 改为本地代码分支，演示本地分支提交触发流程，其中关于 .docker-volumes/加到 ignore，具体干啥的留个问号。还有 commit 规范这块前面**貌似**没讲是如何控制规范的「ci 工具」？
> - 03:50-10:00:来到代码仓库-Actions，讲解 workflows。讲解内容为成功失败执行过程的状态以及 job 在 Github 上Actions 中的执行结果，结论：遇到错误看日志 。
> - 10:00-10:56 :总结回顾步骤 steps 的四种形式 (我的理解是并不是四种形式，是属于一种：steps 下面的 name属性可省略；uses 是是否有使用第三方 actions的需求，可选；run：执行脚本，可选)
> - 11:00-12:57:本章小结
>    - 认识 Github Actions
>    - 应用场景
>    - yml语法格式

**2-4 Github actions 做自动化测试**
新建 yml 文件
```javascript
name: test

on:
    push:
        branches:
            - master
        paths:
            - '.github/workflows/**'
            - '__test__/**'
            - 'src/**'

jobs:
    test:
        runs-on: ubuntu-latest

        steps:
            - uses: actions/checkout@v2
            - name: Use Node.js
              uses: actions/setup-node@v1
              with:
                  node-version: 14
            - name: lint and test
              run: |
                  npm i
                  npm run lint
                  npm run test:remote
```
> 本节讲 Github actions 做自动测试
> - 00:00-- 03:00   test.yml 代码讲解
> 
主要自动测试命令为 npm run lint 和 npm run test:remote(补充：checkout 与setup-node 是 actions 仓库比较常用的两个 actions，分别表示下载代码和安装 node)
> - 03:00-- 04:30   本地与远程接口测试
> 
pre-commit 执行本地接口测试(我的遗留问题：**关于 pre-commit 部分**)
> master push 远程接口测试
> - 04:30 -- 04:50
> 
测试 「测试部署机」部署完毕
> - 04:45 --  05:30  branches 讲解
> 
关于 branches 分支的一些注意说明
> - 05:30 --09:50  分支提交actions 讲解
>    - 这里由于直接下载的代码为开源版本，与课程内容代码出入非常大，因此需要对开源的代码进行操作
>    - 如果将 test.yml 分支改为 main，push到我们自己仓库的时候, actions日志中会发现 lin and test 出现大量错误
>    - 课程给出的开源代码一团，我们为了修正这个错误，我们要去修改、甚至删除那些相应的代码，这里非常不得劲
> 
**还是那个疑问，为什么不整个与课程同步的代码仓库？这个对学员来说究竟是不是必要的？**
> 经过笨拙的尝试，终于成功。

**2-5 Github actions 章总结**
> 没说什么新的内容

## 第三章 Docker

**3-1 Docker 章介绍**
Docker
> 基于 Docker，我们可以把开发、测试环境，一键部署到任何一台机器上。只要该机器安装了 Docker。
> 有了 Docker，就有了一切。

主要产出
> 使用 Docker 构建 nodejs 项目

主要内容
> 认识 Dcoker
> Dockerfile

注意事项
> 专业的运维工程师对 Docker还有更全面的应用：弹性扩展、微服务等。

**3-2 认识 Docker**
介绍
> Docker 就是一种虚拟机技术，比传统虚拟机(VMware、virtualBox)更加简单、轻量
> - 启动快
> - 资源占用少
> - 体积小

查找 docker 安装镜像
> [https://hub.docker.com/](https://hub.docker.com/)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1615798502926-399a6870-27e9-4c1d-9a1c-fb00cb91c9d0.png#height=770&id=EfLvb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1540&originWidth=3192&originalType=binary&size=1486827&status=done&style=none&width=1596)

**3-3 启动一个Docker容器**

**image镜像**
> - 下载镜像 docker pull <image-name>:<tag>
> - 查看所有镜像： docker images
> - 删除镜像：    docker rmi <images-id>
> - 上传镜像：    docker push <username>/<password>:<tags>
> - 如果出现REPOSITORY为 null 的情况时，使用docker image prune删除

**container**
> - 启动容器：**docker run -p xxxx:xxx -v=hostpath:containerPath -d --name <container-name> <image-name>**
>    - -p 端口映射
>    - -v 数据卷，文件映射
>    - -d 后台运行
>    - --name 定义容器名称
> - 查看所有容器 docker ps, 加 -a 显示隐藏的容器
> - 停止容器 docker stop <container-id>
> - 删除容器 docker rm <contaier-id>,加-f 是强制删除
> - 查看容器信息，如 IP 地址 docker inspect <container-id>
> - 查看容器日志 docker logs <container-id>
> - 进入容器控制台 docker exec -it <container-id> /bin/sh

**3-4 Docker容器的进一步演示**
**功能演示**
```javascript
docker run -p 81:80 -d --name nginx1 nginx
docker ps

# 访问 localhost:81 ，并查看 log

docker exec -it <container-id> /bin/sh
cd /usr/share/nginx/html
echo hello docker world index.html
exit

# 重新访问 localhost:81 ，强制刷新

docker stop <container-id>
docker rm <container-id>

```

```javascript
# 1. 新建 /Users/wfp/html/index.html ，内容自定义即可

# 2. 运行
docker run -p 81:80 -v=/Users/wfp/html:/usr/share/nginx/html  -d --name nginx1 nginx

# 3. 访问 重新访问 localhost:81 ，看是否你创建的页面？

```

**3-5 介绍 Dockerfile 语法**
> 一个简单的配置文件，描述如何构建一个新的 image 镜像
> 注意：必须是 Dockerfile 这个文件名，必须在项目的根目录

```javascript
# Dockerfile

FROM node:latest
WORKDIR /app
COPY . /app

# 设置时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone
#安装
RUN npm set registry https://registry.npm.taobao.org
RUN npm install
#启动
CMD echo #SERVER_NAME &&echo &AUTHOR_NAME && npm run dev && npx pm2 log

#环境变量

ENV SERVER_NAME ="test"
ENV AUTHOR_NAME ="liugezhou"

```

**3-6 用 Dockerfile 构建镜像**
**构建**
> docker build -t <name> .  #最后的'.'指 Dockerfile 在当前目录下。    
> docker images

**课程修改代码为(去掉routes/index.js的数据库连接以及bin/www中的数据库同步)：**
```javascript
# Dockerfile
FROM node:14
WORKDIR /app
COPY . /app

# 设置时区
RUN ln -sf /usr/share/zoneinfo/Asia/Beijing /etc/localtime && echo 'Asia/Beijing' >/etc/timezone

#安装
RUN  npm set registry https://registry.npm.taobao.org
RUN npm install

#启动
CMD echo $SERVER_NAME && echo $AUTHOR_NAME && npm run dev && npx pm2 log

#环境变量
ENV SERVER_NAME="lego-node-server"
ENV AUTHOR_NAME="liugezhou"
```

```typescript
// 步骤一：构建
docker build -t liugezhou-server .  # name为将要构建镜像的名字，. 为当前目录
// 步骤二：查看
docker images
//步骤三：启动
docker run -p 8081:3000 -d --name server1 liugezhou-server  # 创建容器，注意端口映射
//步骤四：查看启动状态
docker ps
// 步骤五 查看容器日志
docker logs <container-id>  # 需等待构建完成

# 访问 localhost:8081/api/db-check ，查看 docker logs

docker stop <container-id>
docker rm <container-id>
docker rmi <image-id>

```
## 第四章 Docker-compose

**4-1 docker-compose 章开始**
> 用的来说就是 Docker-compse就是一种组合，这章学到的内容就是这个英文单词的翻译。

**4-2 docker-compose 配置文件**
> - 文件名称必须为 **docker-compose.yml**
> - 代码演示：多个service，代表多个docker镜像
> - **image:redis **   表示引用官网的redis 镜像
> - container-name    镜像名称
> - ports    端口映射
> - enviroment    环境变量 - TZ
> - build：    docker build -t  <image-name>  . 
>    - context .  ：「当前目录」
>    - dockerfile：Dockerfile   ： 「基于Dockerfile构建」

```typescript
version: '3'
services:
    editor-server:  # service name
        build:
            context: .  # 当前目录
            dockerfile: Dockerfile  # 基于 Dockerfile 构建
        image: editor-server # 依赖于当前 Dockerfile 创建出来的镜像
        container_name: editor-server
        ports:
            - 8081:3000 # 宿主机通过 8081 访问
    editor-redis:  # service name，重要！
        image: redis  # 引用官网 redis 镜像
        container_name: editor-redis
        ports:
 # 宿主机，可以用 127.0.0.1:6378 即可连接容器中的数据库 `redis-cli -h 127.0.0.1 -p 6378`
 # 但是，其他 docker 容器不能，因为此时 127.0.0.1 是 docker 容器本身，而不是宿主机
            - 6378:6379
        environment:
            - TZ=Asia/Shanghai # 设置时区

```
**4-3 docker-compose 命令演示**
> - 00：00    --    02：55    **命令**
>    - docker-compose build <service-name>
>    - 启动所有服务器：docker-compose up -d (后台启动)
>    - 停止所有服务：    docker-compose down
>    - 查看服务：    docker-compose ps
>    - docker 与docker-compose的命令执行范围
> - 02：55    --    05：10    **安装pm2**
>    - 本地安装pm2  npm i pm2 --S,或者Dockerfile中全局安装pm2
>    - 再次强调 「阻塞控制台的命令」
> - 05：10    --    06 :30    **代码修改**
>    - 新建 docker-compose.yml
>    - 新建 config/envs/prd-dev.js
> - 06：30    --    08：16    **prd-dev.js文件**
>    - 内容为修改redis连接配置，讲解
> - 08：19    --    10：00    画图？
>    - 罗里吧嗦
> - 10：01    --    10：48    代码修改
>    - wtf，也不知道是出于什么原因，如此设计课程，观看视频体验 ：一颗星
> - 10：49    --    13：17    **build**
>    - docker-compose build  editor-server
> - 13：18    --    15：12    **演示**
>    - docker images     查看build是否成功
>    - docker-compose -d
>    - docker-compose ps
>    - docker ps
>    - 访问：localhost:8081/api/db-check
> - 15：12    --    17：17    **redis-cli**
>    - 终端输入：**redis-cli**，进入到本地redis服务,查看本地 keys *。
>       - 「执行redis-cli，我本地显示：Could not connect to Redis at 127.0.0.1:6379: Connection refused；这是因为我本地没启redis服务，于是：redis-server启动，redis-cli进入redis控制台」
>    - **redis-cli -h 127.0.0.1 -p 6378**    进入到docker容器中的redis
> - 17：18    --    18：25    **查看日志、down**
>    - docker logs <container-id>
>    - docker-compose down

**4-4 数据持久化**
**连接mysql和mongodb**
> **区别:**
> - redis无数据库，mysql与mongodb需要连接数据库
> - redis是缓存，无需数据持久化，mysql与mongodb需要**。**

> volumes:
> - '.docker-volumes/mongo/data:/data/db' # 数据持久化

**4-5 配置 mysql**
```typescript
version: '3'
services:
    editor-server:
        build:
            context: .
            dockerfile: Dockerfile
        image: editor-server # 依赖于当前 Dockerfile 创建镜像
        container_name: editor-server
        ports:
            - 8081:3000 # 宿主机通过 8081 访问
    editor-redis:
        image: redis # 引用官网 redis 镜像
        container_name: editor-redis
        ports:
            - 6378:6379 # 宿主机可以用 127.0.0.1:6378 即可连接容器中的数据库
        environment:
            - TZ=Asia/Beijing # 设置时区
    editor-mysql:
        image: mysql # 引用官网 mysql 镜像
        container_name: editor-mysql
        restart: always
        privileged: true # 高权限，执行下面的 mysql/init
        command: --default-authentication-plugin=mysql_native_password # 解决无法远程访问的问题
        ports:
            - 3305:3306 # 宿主机可以用 127.0.0.1:3305 即可连接容器中的数据库
        volumes:
            - .docker-volumes/mysql/log:/var/log/mysql # 数据持久化
            - .docker-volumes/mysql/data:/var/lib/mysql
            - ./mysql/init:/docker-entrypoint-initdb.d/ # 初始化 sql
        environment:
            - MYSQL_DATABASE=imooc_lego_course # 初始化容器时创建数据库
            - MYSQL_ROOT_PASSWORD=liugezhou1205
            # - MYSQL_USER=shuangyue #创建 test 用户
            # - MYSQL_PASSWORD=shuangyue #设置 test 用户的密码
            - TZ=Asia/Beijing # 设置时区
    editor-mongo:
        image: mongo # 引用官网 mongo 镜像
        container_name: editor-mongo
        restart: always #出错则重启
        volumes:
            - '.docker-volumes/mongo/data:/data/db' # 数据持久化
        environment:
            - MONGO_INITDB_DATABASE=imooc_lego_course
            # - MONGO_INITDB_ROOT_USERNAME=root
            # - MONGO_INITDB_ROOT_PASSWORD=123456
            - TZ=Asia/Beijing # 设置时区
        ports:
            - '27016:27017' # 宿主机可以用 127.0.0.1:27016 即可连接容器中的数据库

```

## 第五章 发布到测试机

**5-2  配置测试机**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1616483749691-5096e493-0140-498b-9141-01183dfbaba1.png#height=241&id=GkJ6b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=482&originWidth=1590&originalType=binary&size=97199&status=done&style=none&width=795)

**5-3 自动发布到测试机-讲解配置**
**5-4 自动发布到测试机-功能演示**
```typescript
# This workflow will do a clean install of node dependencies, build the source code and run tests across different versions of node
# For more information see: https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions
# github actions 中文文档 https://docs.github.com/cn/actions/getting-started-with-github-actions

name: deploy for dev

on:
push:
branches:
- 'dev' # 只针对 dev 分支
paths:
- '.github/workflows/*'
# - '__test__/**' # dev 不需要立即测试
  - 'src/**'
    - 'Dockerfile'
    - 'docker-compose.yml'
    - 'bin/*'

jobs:
  deploy-dev:
 	 runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
      - name: set ssh key # 临时设置 ssh key
      run: |
        mkdir -p ~/.ssh/
    echo "${{secrets.WFP_ID_RSA}}" > ~/.ssh/id_rsa
    # secret 在这里配置 https://github.com/imooc-lego/biz-editor-server/settings/secrets
    chmod 600 ~/.ssh/id_rsa
    ssh-keyscan "182.92.xxx.xxx" >> ~/.ssh/known_hosts
      - name: deploy # 部署
      run: |
        ssh work@182.92.xxx.xxx "
# 【注意】用 work 账号登录，手动创建 /home/work/imooc-lego 目录
# 然后 git clone https://username:password@github.com/imooc-lego/biz-editor-server.git -b dev （私有仓库，使用 github 用户名和密码）
    # 记得删除 origin ，否则会暴露 github 密码

    cd /home/work/imooc-lego/biz-editor-server;
    git remote add origin https://wangfupeng1988:${{secrets.WFP_PASSWORD}}@github.com/imooc-lego/biz-editor-server.git;
    git checkout dev;
    git pull origin dev; # 重新下载最新代码
    git remote remove origin; # 删除 origin ，否则会暴露 github 密码
    # 启动 docker
    docker-compose build editor-server; # 和 docker-compose.yml service 名字一致
    docker-compose up -d;
    "
      - name: delete ssh key # 删除 ssh key
      run: rm -rf ~/.ssh/id_rsa
```

**5-5 自动发布到测试机--章总结**
> 😔
