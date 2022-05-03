---
layout: post
title: CentOS_Docker_Nginx_Node_Jenkins
author: liugezhou
date: 2021-06-13
updated: 2021-06-13
category: 
    服务端
tags:
    项目部署
---
> 此篇博客总结背景：
> [https://mp.weixin.qq.com/s/Mjh7JU4GhDva2wn7VVbq4Q](https://mp.weixin.qq.com/s/Mjh7JU4GhDva2wn7VVbq4Q)

### 
### CentOs
> 1. 本地拷贝文件到远程服务器
> 
scp output.txt root@xx.xx.xx.xx:/data/
>      scp /Users/liumingzhou/Descktop/demo/*	root@xx.xx.xx.xx:/data/
> 2. 本地连接远程CentOs服务器
> 
ssh -p 22 root@xx.xx.xx.xx 
> 3. yum切换为阿里源
> 
cd /etc/yum.repo.d/
> curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
> yum makecache
> 4. 将 test目录下所有文件**复制**到 newtest目录下
> 
cp -r /root/test/* /root/newtest
> 5. 将 test目录下所有文件**移动**到 newtest目录下
> 
mv /root/test/* /root/newtest
> 6. 检查端口被哪个进程占用
>    - netstat -lnp | grep 8080   kill -9 8080
>    -  lsof -i:8900 	/ kill -9 PID
> 7. 查看当前Centos操作系统发行版信息
> 
cat /etc/redhat-release
> 8. 列出该目录下的所有文件路径
> 
find .

### Nginx
> 1. yum安装nginx
>    1. yum install -y nginx
>    1. nginx 【启动nginx】
> 2. ngiinx查找本地目录
>    1. nginx -t
> 3. nginx重启
>    1. nginx -c /usr/local/etc/nginx/nginx.conf
>    1. nginx -s reload
> 4. 反向代理配置
>    1. 访问路径：/api/getUser
>       1. 如下代码，proxy_pass中带着 `/`,代理到后端的路径为：[http://127.0.0.1:18081/](http://127.0.0.1:18081/)getUser
>       1. 如果，不带 `/`时，代理到后端的路径为：[http://127.0.0.1:18081/api/getUser](http://127.0.0.1:18081/getUser)
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
### 

### Centos-Node环境
> 1. nvm查看Node版本列表
>    1. nvm list-remote
> 2. 安装node版本
>    1. nvm install 14
> 3. 查看当前node版本
>    1. nvm current  || node -v
> 4. npm配置淘宝源
>    1. npm config set registry https://registry.npm.taobao.org
> 5. npm查看当前源
>    1. npm get registry

### 
### Centos-Docker
> 1. Centos上安装Docker
>    1. 需要安装 device-mapper-persistent-data [高级存储]和 lvm2 [逻辑磁盘分区]两个依赖
>       1. yum install -y yum-utils device-mapper-persistent-data lvm2
>    2. 安装完毕后，将阿里云Docker镜像添加进去，加速Docker安装
>       1. sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
>    3. 启动Docker
>       1. systemctl start docker
>       1. docker -v --查看Docker安装版本信息
>       1. chkconfig docker on --开机启动
> 2. Docker常见命令
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
> 3. Docker内配置nginx容器
> 
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1623587645864-c743dce2-55da-427b-b24e-227ec7e91da5.png#clientId=u6e986a30-d22b-4&from=paste&height=516&id=ud7b803a7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1732&originWidth=1452&originalType=binary&ratio=2&size=430565&status=done&style=none&taskId=u7ae7306b-c74d-4dc3-99e3-643dd093bbd&width=433)
> 


### Centos--Jenkins
>    1. 因为Jenkins是Java编写的持续构建平台，所以安装Java必不可少。
> 1. 使用yum包管理器安装openjdk
>    1. yum install -y java
> 2. 使用国内镜像加速安装Jenkins
>    1. cd /data
>    1. wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.277.2-1.1.noarch.rpm
>    1. sudo yum install jenkins-2.277.2-1.1.noarch.rpm
> 3. 启动Jenkins
>    1. sevice jenkins start  （Jenkins第一次启动时间较长）
>    1. service jenkins restart restart		--重启jenkins
>    1. sevice jenkins restart stop	--停止Jenkins
> 4. 解锁Jenkins
>    1. 解锁界面	[cat /var/lib/jenkins/secrets/initialAdminPassword     --查看密码]
>    1. 下载插件	[更换插件源--将/var/lib/jenkins/updates/default.json 内的插件源地址替换成清华大学的源地址，将 google 替换为 baidu 即可]
> 

```css
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/lib/jenkins/updates/default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /var/lib/jenkins/updates/default.json
```
> c. 完成安装--注册管理员账户
> 5. 测试安装
>    1. 点击 Jenkins 首页 -> 左侧导航 -> 新建任务 -> Freestyle project(构建一个自由风格的软件项目)
> 
后续步骤暂不整理，直接查看文首链接。

















