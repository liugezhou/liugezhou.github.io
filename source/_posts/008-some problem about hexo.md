---
title: 最近搭建博客遇到的一系列问题
date: 2019-06-11
updated: 2022-05-04
toc: true
description: 最近在重新搭博客的时候，发生了一系列问题，今天抓紧记录下。
categories:
    博客搭建
tags: 
    HEXO
---
<font color=blue>更新说明：对文章排版以及内容格式做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

> 最近在重新搭博客的时候，发生了一系列问题，今天抓紧记录下。

## 环境介绍
> 首先介绍一下我的博客部署：
> 我的博客是通过hexo搭建、上传至Github上，之前的博客访问便都是：https://liugezhou.github.io
> 但是最新搭建的这个博客，我想给自己已有的域名进行绑定：www.liugezhou.online.
> 于是，查找了一些资料，知道需要将我的域名进行解析， 需要添加 CNAME 记录。
> 在域名解析的时候，发现其实没服务器什么事，直接在解析设置上添加两条记录（并暂停其它的一些配置）：
> 
 |记录类型|主机记录|解析线路|记录值|
 |:---   |:---  |:---   |:--- |
 |A| @|默认|192.30.252.153|
 | CNAME|www|默认|liugezhou.github.io|

> 我也记不清为什么是这么配置了，总之这个时候我的个人域名便可以直接访问我的博客内容了。

## 问题出现
> 在写博客的时候，避免不了要使用图片，于是我专门在Github上建了个仓库放图片，这样我每次需要插入图片的时候，直接使用Markdown语法引入图片便好了。  
> 用这样的方法，`在github网站上访问项目博客文件`的时候，看md文件的确是可以访问到图片的。
> 
> 但是用HEXO搭建的博客网站，其中图片便死活不出来，就这样写了两篇文章后，感觉是真难受。
>
>一篇文章怎么可以没有图片呢？
>怎么可以没有自己上传的图片呢？
>怎么可以总是引用别人的图片地址呢？

##  解决图片存储问题
> 七牛云有个免费存储图片的功能--使我可以便于链接访问。

> 这里记录的我使用到它所做的简单步骤。

> 1️⃣  不管你用什么方法首先登录七牛云网站，不管你用什么方法首先有个自己的域名。

> 2️⃣  进入首页--查找产品--对象存储--立即使用--新建存储空间

> 3️⃣  绑定域名[融合 CDN 加速域名] （重点⭐⭐⭐）
> 这里又需要在域名控制台配置一下CNAME，但是我之前已经将Github与liugehzou.online的域名使用了一次CANME,于是犯了嘀咕。。  
> 
> 继续，查阅[官方资料](https://developer.qiniu.com/fusion/kb/1322/how-to-configure-cname-domain-name)、等资料，经过多次尝试：

> 绑定域名选择【普通域名】、加速域名设置为：【img.liugezhou.online】,然后到这里在七牛云上的配置便结束！

> 接着进入域名管理平台，进行域名解析：
> 同上，只需要添加一条记录即可：
> 
|记录类型|主机记录|解析线路|记录值|
 |:---   |:---  |:---   |:--- |
 | CNAME|img|默认|img.liugezhou.online.qiniudns.com|

 >这时候回到七牛云平台，点击内容管理，直接上传照片，然后查看地址便可以访问了。
 > 这期间还有一个加水印的功能，我试了一下没加上，这个应该是后台上传图片的时候才有效，也罢，到此最起码解决了图片的上传问题。

 ## 博客遗留问题
 > 之前我的博客在另一台电脑上进行的部署，后来我换电脑后，出了一些状况。
 > 就是：换了台电脑我不能很好的部署上传了，原因为文章的发布时间都错乱了，这也是导致我重新部署博客的原因。

 > 但是，如果我再换电脑是不是意味着我还有继续重新部署，博客文章归为0呢？
 > 当然不是，查阅了一些资料后，我知道情况是这样的：
 
 > 首先HEXO博客搭建上传至Github使用的命令是hexo clean && hexo g && hexo d,这时候上传到Github上的只是由HEXO生成的静态文件，而本地的关于hexo的文件其实还是在本地的，这个时候就需要在liugezhou.github.io的项目上，再建一个分支，将本地的HEXO项目上传至另一个分支，每次提交代码的时候hexo d部署到主分支，本地的文件提交到新建的分支，这样在其它电脑上再写博客的时候，只需要将新建的那个分支上的代码下载到本地即可。（这里说的比较混乱，因为这里我现在还没有新建分支提代码，这里只是记录一下思路，待有兴趣或者迫不得已的时候再去实践） 

 > 行吧，大概就是这个样子。

