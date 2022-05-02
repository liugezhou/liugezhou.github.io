---
layout: post
title: Week1-需求分析与架构设计
author: liugezhou
date: 2020-12-21
category: 
    Web架构之脚手架
tags:
    Web架构之脚手架
---
### week1:需求分析与架构设计

#### 第一章 课程导学

---

##### 1-1 课程导学

> - 课程目标对标大厂P7
> - 课程准备从招聘、提高自身实力、分析未来发展展开
> - 课程内容：5大核心系统、12个代码仓库、4万行代码等构成一个体系完整的闭关系统项目。
> - 项目预览、体系关系图、知识点图谱等。
> - 收获：具有web前端架构师能力、亲身精力实战研发过程、拥有架构师思维。
> - 对前置知识：TS/Vue/React/Webpack/nodejs等自行查漏补缺。
> - 多实践、记录笔记。


#### 第二章 周介绍

---

##### 2-1 周介绍

> - 本周内容：需求和架构设计
> - 收获：研发流程规范化、熟悉产品需求、以架构师思维分析理解需求、《整体技术方案设计》文档、学会如何写技术方案设计。


#### 第三章 需求分析

---

> 脱离业务的架构就是耍流氓 、架构师必须深入理解需求、参与需求、看透需求背后的业务本质。


##### 3-1 产品研发流程

> 公司起步-> 项目启动 ->需求 -> 技术方案设计 -> 开发 -> 联调 -> 测试 -> 上线(版本升级) -> 项目总结 -> 年度总结


##### 3-2 以架构师的思维分析需求

> 一道面试题开始--要对业务本身以及延伸展开考虑


##### 3-3 项目浅层需求

> 简言之就是从项目前端业务展示看整个项目的流程

##### 
##### 3-4 深度需求

> 不容易一眼看出来，却很重要的需求
> - 作品管理(删除恢复、转增、复制)
> - 作品统计(通过统计看结果，分渠道统计)
> - 作品发布(url不变、支持多渠道)
> - H5分享(对业务增长服务)
> - 后台管理(数据全局把控)


##### 3-5 需求总结
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1611107891584-9ad7cb88-e1ca-44e6-8b13-9ec7bcb056e3.png#align=left&display=inline&height=490&margin=%5Bobject%20Object%5D&name=image.png&originHeight=980&originWidth=1474&size=291129&status=done&style=none&width=737)

#### 第四章 架构设计

---

##### 4-1 整体架构设计--章介绍

> - 任何看似复杂的架构，都是让整个系统变得简单
> - 学会如何写技术方案设计
> - 看整体、考虑扩展性、可行性、多调研、莫为设计而设计、用最简单实现方式。


##### 4-2 分析需要多少个项目

> B端编辑器(前端、后端)、H5、管理后台(前端、后端)
> ![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1611109401048-36d88b06-e12a-4593-919f-9b09e8918036.png#align=left&display=inline&height=504&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1008&originWidth=2498&size=846837&status=done&style=none&width=1249)


##### 4-3 需要自研统计服务

> 需要自研统计服务的原因：第三方贵且不能很好满足需求。
> ![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1611109127676-121109ff-7320-4e44-9091-c4adf26e8b43.png#align=left&display=inline&height=807&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1614&originWidth=3192&size=1638978&status=done&style=none&width=1596)


##### 4-4 各个项目之间的关系图
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1611109490984-9356c610-ac8e-419d-9783-7962c50a1abc.png#align=left&display=inline&height=450&margin=%5Bobject%20Object%5D&name=image.png&originHeight=900&originWidth=1518&size=532197&status=done&style=none&width=759)

##### 4-5 作品数据结构设计
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1611110762637-d0746925-b374-40ad-a96e-dc72dfc102c8.png#align=left&display=inline&height=177&margin=%5Bobject%20Object%5D&name=image.png&originHeight=354&originWidth=1436&size=66758&status=done&style=none&width=718)

##### 4-6 数据流转关系图
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1611110798922-dae08982-b324-4eed-9532-f2bced6bef01.png#align=left&display=inline&height=921&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1842&originWidth=1524&size=219286&status=done&style=none&width=762)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1611110940620-b88500a2-b27b-4a80-a0c8-b9e3b738dec8.png#align=left&display=inline&height=529&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1058&originWidth=1554&size=105617&status=done&style=none&width=777)

##### 4-7 技术方案文档的重要性
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1611111650292-8e1fbf19-0323-43a2-8408-c197e18016eb.png#align=left&display=inline&height=506&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1012&originWidth=1566&size=135823&status=done&style=none&width=783)

##### 4-8 写架构设计文档

> 通过此节内容，整理一个架构设计文档的范本，方便以后在写架构设计文档的时候有个demo参考。
> [整体架构设计范本](https://www.yuque.com/liugezhou/jiagou/qs4mtc)


##### 4-9 本周总结以及下一步操作

![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1611113097236-d89fa8ed-74b1-42b9-907f-b97b22061aac.png#align=left&display=inline&height=479&margin=%5Bobject%20Object%5D&name=image.png&originHeight=958&originWidth=1560&size=121338&status=done&style=none&width=780)
#### 第五章 本周总结

---

##### 5-1 本周总结

> 以架构师思维分析需求、理解需求，写整体技术方案设计。


##### 5-2 关于作业和打卡

> [homework.imooc-lego.com](homework.imooc-lego.com)









