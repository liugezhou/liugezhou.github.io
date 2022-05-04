---
layout: post
title: CentOS_Docker_Nginx_Node_Jenkins
toc: true
date: 2021-06-13
updated: 2022-05-04
description: CentOS_Docker_Nginx_Node_Jenkins
categories:
- 前端小技
tags: 
- 服务端
---
<font color=blue>更新说明：对文章排版以及内容格式做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

> 此篇博客总结背景：
> [https://mp.weixin.qq.com/s/Mjh7JU4GhDva2wn7VVbq4Q](https://mp.weixin.qq.com/s/Mjh7JU4GhDva2wn7VVbq4Q)

## CentOs
> - 本地拷贝文件到远程服务器
>  scp output.txt root@xx.xx.xx.xx:/data/
>  scp /Users/liumingzhou/Descktop/demo/*	root@xx.xx.xx.xx:/data/
> - 本地连接远程CentOs服务器
>  ssh -p 22 root@xx.xx.xx.xx 
> - yum切换为阿里源
>  cd /etc/yum.repo.d/
> curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
> yum makecache
> - 将 test目录下所有文件**复制**到 newtest目录下
cp -r /root/test/* /root/newtest
> - 将 test目录下所有文件**移动**到 newtest目录下
mv /root/test/* /root/newtest
> - 检查端口被哪个进程占用
>    netstat -lnp | grep 8080   kill -9 8080
>    lsof -i:8900 	/ kill -9 PID
> - 查看当前Centos操作系统发行版信息
cat /etc/redhat-release
> - 列出该目录下的所有文件路径
>  find .

## Nginx
> - yum安装nginx
>     yum install -y nginx
>     nginx 【启动nginx】
> - ngiinx查找本地目录
>     nginx -t
> - nginx重启
>     nginx -c /usr/local/etc/nginx/nginx.conf
>     nginx -s reload
> - 反向代理配置
>     访问路径：/api/getUser
>     如下代码，proxy_pass中带着 `/`,代理到后端的路径为：[http://127.0.0.1:18081/](http://127.0.0.1:18081/)getUser
>     如果，不带 `/`时，代理到后端的路径为：[http://127.0.0.1:18081/api/getUser](http://127.0.0.1:18081/getUser)
> 

```css
server {
    listen       80;
    server_name  www.123.com;

    location /api/ {
        proxy_pass http://127.0.0.1:18081/;
    }
}
```

## Centos-Node环境
> - nvm查看Node版本列表
>     nvm list-remote
> - 安装node版本
>     nvm install 14
> - 查看当前node版本
>     nvm current  || node -v
> - npm配置淘宝源
>     npm config set registry https://registry.npm.taobao.org
> - npm查看当前源
>     npm get registry

## Centos-Docker
> - Centos上安装Docker
>    1. 需要安装 device-mapper-persistent-data [高级存储]和 lvm2 [逻辑磁盘分区]两个依赖
>       1. yum install -y yum-utils device-mapper-persistent-data lvm2
>    2. 安装完毕后，将阿里云Docker镜像添加进去，加速Docker安装
>       1. sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
>    3. 启动Docker
>       1. systemctl start docker
>       1. docker -v --查看Docker安装版本信息
>       1. chkconfig docker on --开机启动
> - Docker常见命令
>    1. docker version/docker info	--查看docker版本
>    1. docker pull <image-name>:<tag>	--下载镜像
>    1. docker images		--查看本地已经安装好的镜像
>    1. docker rmi <images-id>		-- 删除本地安装的某镜像
>    1. docker ps -a		 --显示本地启动的所有容器
>    1. docker start/stop/restart imageID   -- 容器的启动/关闭/重启
>    1. docker rm containerID	--	删除一个容器
>    1. docker instapect <container-id>	--查看容器信息
>    1. docker logs <container-id>	--查看容器日志
>    1. docker built -t <name>	--	自定义创建image容器
>    1. docker exec -it <container-id>/bin/sh	--进入控制台
>    1. docker run -p 81:80		--从image文件生成容器
>    1. docker cp [container-id]:[/Users/liu/Desktop]		--文件拷贝
>    1. docker run -it ubuntu:tag /bin/sh		--启动ubuntu 镜像
>    1.  docker run -p 81/80 -d --name nginx1 nginx
> - Docker内配置nginx容器
> 
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1623587645864-c743dce2-55da-427b-b24e-227ec7e91da5.png#clientId=u6e986a30-d22b-4&from=paste&height=516&id=ud7b803a7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1732&originWidth=1452&originalType=binary&ratio=2&size=430565&status=done&style=none&taskId=u7ae7306b-c74d-4dc3-99e3-643dd093bbd&width=433)

## Centos--Jenkins
>   因为Jenkins是Java编写的持续构建平台，所以安装Java必不可少。
> - 使用yum包管理器安装openjdk
>     yum install -y java
> - 使用国内镜像加速安装Jenkins
>    1. cd /data
>    1. wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.277.2-1.1.noarch.rpm
>    1. sudo yum install jenkins-2.277.2-1.1.noarch.rpm
> - 启动Jenkins
>    1. sevice jenkins start  （Jenkins第一次启动时间较长）
>    1. service jenkins restart restart		--重启jenkins
>    1. sevice jenkins restart stop	--停止Jenkins
> - 解锁Jenkins
>    1. 解锁界面	[cat /var/lib/jenkins/secrets/initialAdminPassword     --查看密码]
>    1. 下载插件	[更换插件源--将/var/lib/jenkins/updates/default.json 内的插件源地址替换成清华大学的源地址，将 google 替换为 baidu 即可]
> 

```css
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/lib/jenkins/updates/default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /var/lib/jenkins/updates/default.json
```
> c. 完成安装--注册管理员账户
> - 测试安装
>    点击 Jenkins 首页 -> 左侧导航 -> 新建任务 -> Freestyle project(构建一个自由风格的软件项目)
> 
后续步骤暂不整理，直接查看文首链接。

