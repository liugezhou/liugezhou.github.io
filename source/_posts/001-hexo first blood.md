---
title: 搭建个人-博客极简
author: 六个周
date: 2018-07-11
updated: 2018-07-11
toc: true
description: 第一次搭建博客的极简总结
categories:
    博客搭建
tags: 
    HEXO
---
> 进入[HEXO](https://hexo.io/zh-cn/)官方文档，可以查看完全的个人博客的搭建。
> 本文只是对个人在搭建博客过程中，极简的总结，几乎涵盖搭建一个个人免费博客的全部命令，还是非常推荐去官方网站学习。
## 官网主题
在搭建个人博客本地版的时候，总共使用六个个命令就可以看到样板。
```
npm install hexo-cli -g 
cd DeskTop
hexo init liugezhou
cd liugezhou
npm install
hexo g && hexo s
```
此时，本地效果图如下：
![001hexo](https://cdn.jsdelivr.net/gh/liugezhou/image@master/img/001hexo.3c8e8gvca6g0.webp)

## 更改主题
> $ git clone https://github.com/Haojen/hexo-theme-Anisina.git themes/lmz

## 项目部署
> $  npm install hexo-deployer-git --save 
> $  hexo clean && hexo g && hexo d
## 效果查看
使用该主题，初次搭建的博客效果：https://www.liugezhou.online
PC版：
![primary](https://cdn.jsdelivr.net/gh/liugezhou/image@master/img/primary.webp)


