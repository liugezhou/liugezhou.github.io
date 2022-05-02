---
layout: post
title: Docker
author: liugezhou
date: 2021-09-05
category: 
    服务端
tags:
    项目部署
---
## 一、Docker 应用部署

---


### 1-1、部署MySQL

1. 搜索mysql镜像

```shell
docker search mysql
```

2. 拉取mysql镜像

```shell
docker pull mysql:5.6
```

3. 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建mysql目录用于存储mysql数据信息
mkdir ~/mysql
cd ~/mysql
```

```shell
docker run -id \
-p 3307:3306 \
--name=c_mysql \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.7
```

- 参数说明： 
   - **-p 3307:3306**：将容器的 3306 端口映射到宿主机的 3307 端口。
   - **-v $PWD/conf:/etc/mysql/conf.d**：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。配置目录
   - **-v $PWD/logs:/logs**：将主机当前目录下的 logs 目录挂载到容器的 /logs。日志目录
   - **-v $PWD/data:/var/lib/mysql** ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。数据目录
   - **-e MYSQL_ROOT_PASSWORD=123456：**初始化 root 用户的密码。

4. 进入容器，操作mysql

```shell
docker exec –it c_mysql /bin/bash
```

5. 使用外部机器连接容器中的mysql


### 1-2、部署Tomcat

1. 搜索tomcat镜像

```shell
docker search tomcat
```

2. 拉取tomcat镜像

```shell
docker pull tomcat
```

3. 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建tomcat目录用于存储tomcat数据信息
mkdir ~/tomcat
cd ~/tomcat
```

```shell
docker run -id --name=c_tomcat \
-p 8080:8080 \
-v $PWD:/usr/local/tomcat/webapps \
tomcat
```

- 参数说明： 
   -  **-p 8080:8080：**将容器的8080端口映射到主机的8080端口
**-v $PWD:/usr/local/tomcat/webapps：**将主机中当前目录挂载到容器的webapps 

4. 使用外部机器访问tomcat


### 1-3、部署Nginx

1. 搜索nginx镜像

```shell
docker search nginx
```

2. 拉取nginx镜像

```shell
docker pull nginx
```

3. 创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建nginx目录用于存储nginx数据信息
mkdir ~/nginx
cd ~/nginx
mkdir conf
cd conf
# 在~/nginx/conf/下创建nginx.conf文件,粘贴下面内容
vim nginx.conf
```

```shell

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

```shell
docker run -id --name=c_nginx \
-p 80:80 \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/usr/share/nginx/html \
nginx
```

- 参数说明： 
   - **-p 80:80**：将容器的 80端口映射到宿主机的 80 端口。
   - **-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf**：将主机当前目录下的 /conf/nginx.conf 挂载到容器的 :/etc/nginx/nginx.conf。配置目录
   - **-v $PWD/logs:/var/log/nginx**：将主机当前目录下的 logs 目录挂载到容器的/var/log/nginx。日志目录

4. 使用外部机器访问nginx


### 1-4、部署Redis

1. 搜索redis镜像

```shell
docker search redis
```

2. 拉取redis镜像

```shell
docker pull redis:5.0
```

3. 创建容器，设置端口映射

```shell
docker run -id --name=c_redis -p 6379:6379 redis:5.0
```

4. 使用外部机器连接redis

```shell
./redis-cli.exe -h 192.168.149.135 -p 6379
```


## 二、Dockerfile

### 2-1 Docker镜像原理
> - Docker镜像的本质是什么？
>    - 是一个分层文件系统
> - Docker中一个CentOS镜像为什么只有200MB，而一个centos操作系统的iso文件要几个G？
>    - CentOS的iso镜像文件包含bootfs和rootfs，而docker的centos镜像复用了操作系统的bootfs，只有rootfs和其他镜像层
> - Docker中一个Tomcat镜像为什么有680MB，而一个tomcat安装包只要70多MB？
>    - 由于docker中镜像是分层的，tomcat一个安装包虽然只有70多MB，但也需要依赖与父镜像和基础镜像，所有整个对外暴露的tomcat镜像大小有差不多700MB。

下面为原理讲解：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1630824233495-7c78fd33-bf56-453c-bafc-75a494830b4c.png#clientId=ua34b126c-6bee-4&from=paste&height=235&id=u4d44d949&margin=%5Bobject%20Object%5D&name=image.png&originHeight=329&originWidth=255&originalType=binary&ratio=1&size=50532&status=done&style=none&taskId=ud113abe7-c25a-4431-afca-777456cb594&width=182.5)

- linux文件系统由boottfs和rootfs两部分组成
   - bootfs：包含bootloader(引导加载程序)和kernel(内核)
   - rootfs：root文件系统，典型的 /dev,/bin,/etc等标准目录和文件
   - 不同的linux发行版，bootfs基本一样，而rootfs不同，如ubuntu，centos等。
- Docker镜像原理
   - Docker镜像是由特殊的文件系统叠加而成
   - 最底端是bootfs，并使用宿主机的bootfs
   - 第二层是root文件系统rootfs，成为base image  
   - 再往上可以叠加其他的镜像文件
   - 只读镜像不可修改，如果需要对镜像tomcat做修改，Docker在最顶层提供了可加载一个读写文件系统作为容器

![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1630824925617-59703c72-1354-406f-b70c-c797fe604d9e.png#clientId=ua34b126c-6bee-4&from=paste&height=448&id=u00206449&margin=%5Bobject%20Object%5D&name=image.png&originHeight=582&originWidth=646&originalType=binary&ratio=1&size=169405&status=done&style=none&taskId=u43c81d39-31c6-4795-8874-00a4b0ba00b&width=497)

### 2-2 Docker镜像如何制作
> 1. 容器转为镜像
>    - docker commit 容器id 镜像名称：版本号  【数据卷内容不会commit到tar文件中】
>    - docker save -o 压缩文件名称 自定义镜像名：版本
>    - docker load -i 压缩文件名称
> 2. dockerfile

### 
### 2-3 DockerFile概念以及作用
> - DockerFile是一个文本文件
> - 包含了一条条的指令
> - 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
> - 对于开发人员：可以为开发团队提供一个完全一致的开发环境
> - 对于测试人员：可以直接拿开发时所构建的镜像或者通过DockerFile文件构建一个新的镜像开始工作
> - 对于运维人员：在部署时，可以实现应用的无缝移植


### 2-4 Docker关键字
> FROM：指定父镜像--指定dockerfiel基于哪个image构建
> MAINTAINER：作者信息--用来标明这个dockerfile谁写的
> LABLE：标签
> RUN: 执行linux命令 --默认是 /bin/sh，RUN command或者 Run ["command","param1","param2"]
> EXPOSE：暴露端口
> CMD：容器启动命令
> COPY：复制文件--build的时候复制文件到image中
> ADD：添加文件--可以来源于远程服务
> ENV：环境变量
> VOLUMN：定义外部可以挂载的数据卷--启动容器的时候用-v绑定 volume目录
> WORKDIR：工作目录--指定容器内部的工作目录，没有就创建

### 
### 2-5 案例
> 案例一：定义DockerFile，发布spring-boot项目
> 
> - 定义父镜像：FROM java:8
> - 定义作者信息：MAINTAINER liugezhou<18231133236@163.com>
> - 将jar包添加到容器 ： ADD springboot.jar app.jar
> - 定义容器启动执行命令：CMD java -jar app.jar
> 
> 通过dockerfile构建镜像： docker build -f dockerfile文件路径 -t 镜像名称：版本


> 案例二：自定义centos镜像，默认登录路径为 /usr,可以使用vim
> - 定义父镜像：FROM centos:7
> - 定义作者信息：MAINTAINER liugezhou<18231133236@163.com>
> - 执行安装vim命令： RUN yum install -y vim
> - 定义默认的工作目录 WORKDIR /usr
> - 定义容器启动执行的命令：CMD /bin/bash
> 
操作：
> - cd /Users/liugezhou/Desktop/demo/dockerfiles
> - vi centos_dockerfile
> - 将上面内容写入到 centos_dockerfile中
> - docker build -t ./centos_dockerfile mycentos:1 .
> 
此时，查看镜像列表：docker images
> 然后，进入查看目录与vim命令是否生效：docker run -it --name=c4 mycentos


## 三、docker compose

---

### 3-1 Docker服务编排
> 服务编排：按照一定的业务规则批量管理容器
> 
> Docker Compose：Docker Compose是一个编排多容器分布式部署的工具 ，提供命令集管理容器化应用的完整开发周期，包括服务构建、启动和停止。
> 使用步骤：
> 1. 利用Dockerfile定义运行环境镜像
> 1. 使用docker-compose.yml定义组成应用的各服务
> 1. 运行docker-compose up启动应用



### 3-2 Docker Compose安装 & 使用
> - curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
> - chmod +x /usr/local/bin/docker-compose
> - docker-compose 卸载 **rm /usr/local/bin/docker-compose**

> -  cd /Users/liumingzhou/Desktop/demo
> - mkdir docker-compose
> - cd docker-compose
> - vim docker-compose.yml

```javascript
version: '3'
services:
  nginx:
   image: nginx
   ports:
    - 80:80
   links:
    - app
   volumes:
    - ./nginx/conf.d:/etc/nginx/conf.d
  app:
    image: app
    expose:
      - "8080"
```
> - mkdir -p ./nginx/conf.d
> - cd nginx/conf.d
> - vim test.conf

```javascript
server {
    listen 80;
    access_log off;

    location / {
        proxy_pass http://app:8080;
    }
   
}
```
> - cd /Users/liumingzhou/Desktop/demo/docker-compose
> - docker-compose up


## 四、Docker私有仓库

---

#### 1 搭建私有仓库
> 1、拉取私有仓库镜像
> - docker pull registry
> 
2、启动私有仓库容器
> - docker run -id --name=registry -p 5000:5000 registry
> 
3、打开浏览器 输入地址http://私有仓库服务器ip:5000/v2/_catalog，
> 看到{"repositories":[]} 表示私有仓库 搭建成功
> 4、修改daemon.json
> - vim /etc/docker/daemon.json
> - 在上述文件中添加一个key，保存退出。此步用于让 docker 信任私有仓库地址；注意将私有仓库服务器ip修改为自己私有仓库服务器真实ip
> - {"insecure-registries":["私有仓库服务器ip:5000"]}
> 
5、重启docker 服务
> - systemctl restart docker
> - docker start registry

#### 4-2 上传镜像到私有仓库
> 1、标记镜像为私有仓库的镜像
> docker tag centos:7 私有仓库服务器IP:5000/centos:7
> 2、上传标记的镜像
> docker push 私有仓库服务器IP:5000/centos:7

#### 4-3 从私有仓库拉取镜像
> docker pull 私有仓库服务器ip:5000/centos:7


## 五、Docker相关概念
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1630838686450-52d7d239-b044-40aa-be45-5e91a8b160d9.png#clientId=ue6ba99a4-2430-4&from=paste&height=476&id=udd163364&margin=%5Bobject%20Object%5D&name=image.png&originHeight=621&originWidth=413&originalType=binary&ratio=1&size=124725&status=done&style=none&taskId=u85af19d6-fc2e-4c6c-8603-3982ad75ef4&width=316.65625)
> docker容器与传统虚拟机比较
> - 相同：容器和虚拟机具有相似的资源隔离和分配优势
> - 不同：容器虚拟化的是操作系统，虚拟机虚拟化的是硬件；传统虚拟机可以运行不同的操作系统，容器只能运行统一类型操作系统。
> 
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1630838866544-eedcc664-4b72-4279-bde5-df04d4bf357a.png#clientId=ue6ba99a4-2430-4&from=paste&height=226&id=u8b0a675f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=258&originWidth=755&originalType=binary&ratio=1&size=76482&status=done&style=none&taskId=u867c49dd-e341-4a2d-a70e-1bc4a027c4c&width=661.6619262695312)

