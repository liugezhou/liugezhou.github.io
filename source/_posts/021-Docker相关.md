---
layout: post
title: Docker
author: liugezhou
date: 2021-09-05
updated: 2022-12-14
toc: true
description: Docker应用部署
category: 服务端
tags: 服务端
---

## 一、快速命令一览
- `docker -v`: 查看 docker 版本
- `docker images`: 查看本地安装的所有镜像
- `docker images -q`: 查看本地安装的所有镜像ID
- `docker search liugezhou`: 搜索公共镜像
- `docker pull something`: 下载某个镜像
- `docker rmi something`: 删除某个镜像

---

- `docker ps`: 查看正在运行的容器
- `docker stop id`: 停止正在运行的容器
- `docker start id`: 再次启动停止的容器
- `docker restart id`: 重新启动已经启动的容器
- `docker rm id`: 删除已经停止运行的容器
- `docker rm id -f`: 直接删除正在运行中的容器
- `docker run -it --name=nginx nginx /bin/bash`: 此启动方式会给出一个命令终端
  - **-i**: 表示容器运行
  - **-t**: 表示分配一个终端
  - **-d**: 表示守护后台运行
  - **-p**: 表示修改默认的端口
  - **--name**: 给容器起一个名字
- `docker exec  -it id /bin/bash` : 可以通过这个命令进入到容器中去
- 在容器中退出而不停止，使用 `ctrl+p ctrl+q`


## 二、Docker 应用部署实践

### 2.1 MySQL

1. 搜索 mysql 镜像

```yml
docker search mysql
```

2. 拉取 mysql 镜像

```yml
docker pull mysql:5.6
```

3. 创建容器，设置端口映射、目录映射

```yml
# 在/root目录下创建mysql目录用于存储mysql数据信息
mkdir ~/mysql
cd ~/mysql
```

```yml
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
  - **-v $PWD/data:/var/lib/mysql** ：将主机当前目录下的 data 目录挂载到容器的 /var/lib/mysql 。数据目录
  - **-e MYSQL_ROOT_PASSWORD=123456：**初始化 root 用户的密码。

4. 进入容器，操作 mysql

```yml
docker exec –it c_mysql /bin/bash
```

5. 使用外部机器连接容器中的 mysql

### 2.2 Tomcat

1. 搜索 tomcat 镜像

```yml
docker search tomcat
```

2. 拉取 tomcat 镜像

```yml
docker pull tomcat
```

3. 创建容器，设置端口映射、目录映射

```yml
# 在/root目录下创建tomcat目录用于存储tomcat数据信息
mkdir ~/tomcat
cd ~/tomcat
```

```yml
docker run -id --name=c_tomcat \
-p 8080:8080 \
-v $PWD:/usr/local/tomcat/webapps \
tomcat
```

- 参数说明：
  - **-p 8080:8080：**将容器的 8080 端口映射到主机的 8080 端口
    **-v $PWD:/usr/local/tomcat/webapps：**将主机中当前目录挂载到容器的 webapps

4. 使用外部机器访问 tomcat

### 2.3 Nginx

1. 搜索 nginx 镜像

```yml
docker search nginx
```

2. 拉取 nginx 镜像

```yml
docker pull nginx
```

3. 创建容器，设置端口映射、目录映射

```yml
# 在/root目录下创建nginx目录用于存储nginx数据信息
mkdir ~/nginx
cd ~/nginx
mkdir conf
cd conf
# 在~/nginx/conf/下创建nginx.conf文件,粘贴下面内容
vim nginx.conf
```

```yml

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

```yml
docker run -id --name=c_nginx \
-p 80:80 \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/usr/share/nginx/html \
nginx
```

- 参数说明：
  - **-p 80:80**：将容器的 80 端口映射到宿主机的 80 端口。
  - **-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf**：将主机当前目录下的 /conf/nginx.conf 挂载到容器的 :/etc/nginx/nginx.conf。配置目录
  - **-v $PWD/logs:/var/log/nginx**：将主机当前目录下的 logs 目录挂载到容器的/var/log/nginx。日志目录

4. 使用外部机器访问 nginx

### 2.4 Redis

1. 搜索 redis 镜像

```yml
docker search redis
```

2. 拉取 redis 镜像

```yml
docker pull redis:5.0
```

3. 创建容器，设置端口映射

```yml
docker run -id --name=c_redis -p 6379:6379 redis:5.0
```

4. 使用外部机器连接 redis

```yml
./redis-cli.exe -h 192.168.149.135 -p 6379
```

## 三、Dockerfile

### 3.1 Docker 镜像原理

> - Docker 镜像的本质是什么？
>   - 是一个分层文件系统
> - Docker 中一个 CentOS 镜像为什么只有 200MB，而一个 centos 操作系统的 iso 文件要几个 G？
>   - CentOS 的 iso 镜像文件包含 bootfs 和 rootfs，而 docker 的 centos 镜像复用了操作系统的 bootfs，只有 rootfs 和其他镜像层
> - Docker 中一个 Tomcat 镜像为什么有 680MB，而一个 tomcat 安装包只要 70 多 MB？
>   - 由于 docker 中镜像是分层的，tomcat 一个安装包虽然只有 70 多 MB，但也需要依赖与父镜像和基础镜像，所有整个对外暴露的 tomcat 镜像大小有差不多 700MB。

下面为原理讲解：
![image](https://cdn.jsdelivr.net/gh/liugezhou/image@master/img/image.7klmm4fsfyg0.webp)

- linux 文件系统由 boottfs 和 rootfs 两部分组成
  - bootfs：包含 bootloader(引导加载程序)和 kernel(内核)
  - rootfs：root 文件系统，典型的 /dev,/bin,/etc 等标准目录和文件
  - 不同的 linux 发行版，bootfs 基本一样，而 rootfs 不同，如 ubuntu，centos 等。
- Docker 镜像原理
  - Docker 镜像是由特殊的文件系统叠加而成
  - 最底端是 bootfs，并使用宿主机的 bootfs
  - 第二层是 root 文件系统 rootfs，成为 base image
  - 再往上可以叠加其他的镜像文件
  - 只读镜像不可修改，如果需要对镜像 tomcat 做修改，Docker 在最顶层提供了可加载一个读写文件系统作为容器

![image](https://cdn.jsdelivr.net/gh/liugezhou/image@master/img/image.4799tc2g20y0.webp)

### 3.2 Docker 镜像如何制作

 1. 容器转为镜像
    - docker commit 容器 id 镜像名称：版本号 【数据卷内容不会 commit 到 tar 文件中】
    - docker save -o 压缩文件名称 自定义镜像名：版本
    - docker load -i 压缩文件名称
 2. commit
    - 将容器转化为一个镜像
    - 'docker commit -m 'ubuntu with vim wget node yum' -a liugezhou tender_saha liugezhou/ubuntu:node12' 这个node12很重要，表示tag
    - 'docker push liugezhou/ubuntu:node12' 直接推送到仓库


 3. dockerfile

### 3.3 DockerFile 概念以及作用

> - DockerFile 是一个文本文件
> - 包含了一条条的指令
> - 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
> - 对于开发人员：可以为开发团队提供一个完全一致的开发环境
> - 对于测试人员：可以直接拿开发时所构建的镜像或者通过 DockerFile 文件构建一个新的镜像开始工作
> - 对于运维人员：在部署时，可以实现应用的无缝移植

### 3.4 Docker 关键字

> FROM：指定父镜像--指定 dockerfiel 基于哪个 image 构建
> MAINTAINER：作者信息--用来标明这个 dockerfile 谁写的
> LABLE：标签
> RUN: 执行 linux 命令 --默认是 /bin/sh，RUN command 或者 Run ["command","param1","param2"]
> EXPOSE：暴露端口
> CMD：容器启动命令
> COPY：复制文件--build 的时候复制文件到 image 中
> ADD：添加文件--可以来源于远程服务
> ENV：环境变量
> VOLUMN：定义外部可以挂载的数据卷--启动容器的时候用-v 绑定 volume 目录
> WORKDIR：工作目录--指定容器内部的工作目录，没有就创建

### 3.5 案例

> 案例一：定义 DockerFile，发布 spring-boot 项目
>
> - 定义父镜像：FROM java:8
> - 定义作者信息：MAINTAINER liugezhou<18231133236@163.com>
> - 将 jar 包添加到容器 ： ADD springboot.jar app.jar
> - 定义容器启动执行命令：CMD java -jar app.jar
>
> 通过 dockerfile 构建镜像： docker build -f dockerfile 文件路径 -t 镜像名称：版本

> 案例二：自定义 centos 镜像，默认登录路径为 /usr,可以使用 vim
>
> - 定义父镜像：FROM centos:7
> - 定义作者信息：MAINTAINER liugezhou<18231133236@163.com>
> - 执行安装 vim 命令： RUN yum install -y vim
> - 定义默认的工作目录 WORKDIR /usr
> - 定义容器启动执行的命令：CMD /bin/bash
>
> 操作：
>
> - cd /Users/liugezhou/Desktop/demo/dockerfiles
> - vi centos_dockerfile
> - 将上面内容写入到 centos_dockerfile 中
> - docker build -t ./centos_dockerfile mycentos:1 .
>
> 此时，查看镜像列表：docker images
> 然后，进入查看目录与 vim 命令是否生效：docker run -it --name=c4 mycentos

## 四、docker compose

### 4.1 Docker 服务编排

> 服务编排：按照一定的业务规则批量管理容器
>
> Docker Compose：Docker Compose 是一个编排多容器分布式部署的工具 ，提供命令集管理容器化应用的完整开发周期，包括服务构建、启动和停止。
> 使用步骤：
>
> 1. 利用 Dockerfile 定义运行环境镜像
> 1. 使用 docker-compose.yml 定义组成应用的各服务
> 1. 运行 docker-compose up 启动应用

### 4.2 Docker Compose 安装 & 使用

> - curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
> - chmod +x /usr/local/bin/docker-compose
> - docker-compose 卸载 **rm /usr/local/bin/docker-compose**

> - cd /Users/liumingzhou/Desktop/demo
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

## 五、Docker 私有仓库

### 5.1 搭建私有仓库

> 1、拉取私有仓库镜像
>
> - docker pull registry
>
> 2、启动私有仓库容器
>
> - docker run -id --name=registry -p 5000:5000 registry
>
> 3、打开浏览器 输入地址 http://私有仓库服务器 ip:5000/v2/\_catalog，
> 看到{"repositories":[]} 表示私有仓库 搭建成功
> 4、修改 daemon.json
>
> - vim /etc/docker/daemon.json
> - 在上述文件中添加一个 key，保存退出。此步用于让 docker 信任私有仓库地址；注意将私有仓库服务器 ip 修改为自己私有仓库服务器真实 ip
> - {"insecure-registries":["私有仓库服务器 ip:5000"]}
>
> 5、重启 docker 服务
>
> - systemctl restart docker
> - docker start registry

### 5.2 上传镜像到私有仓库

> 1、标记镜像为私有仓库的镜像
> docker tag centos:7 私有仓库服务器 IP:5000/centos:7
> 2、上传标记的镜像
> docker push 私有仓库服务器 IP:5000/centos:7

### 5.3 从私有仓库拉取镜像

> docker pull 私有仓库服务器 ip:5000/centos:7

## 六、Docker 相关概念

![image](https://cdn.jsdelivr.net/gh/liugezhou/image@master/img/image.5j5a1fna9340.webp)

> docker 容器与传统虚拟机比较
>
> - 相同：容器和虚拟机具有相似的资源隔离和分配优势
> - 不同：容器虚拟化的是操作系统，虚拟机虚拟化的是硬件；传统虚拟机可以运行不同的操作系统，容器只能运行统一类型操作系统。
>
> ![image](https://cdn.jsdelivr.net/gh/liugezhou/image@master/img/image.xisha1ny828.webp)
