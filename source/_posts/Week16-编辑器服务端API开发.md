---
layout: post
title: Week16-编辑器服务端API开发
date: 2021-03-23
updated: 2022-05-04
description: 编辑器服务端API开发
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
> - 需求指导设计，设计指导开发。无设计不开发。
> - 服务端技术方案设计的方法
> - B端和编辑器基本功能API
> - 技术方案设计文档

## 第二章：技术方案设计

**2-1 技术方案设计-章介绍**

> **领导技术方案设计、评审技术方案设计。**
> 主要产出：server端技术方案设计
> 主要内容：
> - 接口设计
> - 选择Restful,而不是**GraphQL**
> - 数据库设计
> - sever端整体设计
> 
> **注意**：正视技术方案设计，设计会节约时间。

**2-2 接口设计-整理所有接口**
> - 接口设计应该是在需求后的第一设计，接口设计好之后，前后端可以并行开发，互不影响。
> - 接口设计是和需求对接最亲密的部分。
> - 功能范围确认--根据功能设计接口
> 
统一的输出格式：

```typescript
{
	erron:0,
  data:{.....},
  message:''
}
```
**2-3 接口设计-关于预览和数据统计**
> - 作品统计/预览作品  单独独立
> - 发布--标识位

**2-4 介绍GraphQL的使用和特点**
> 为什么选择Restful API，而没有选择GraphQL
> - GraphQL(Graph Query Language) 图查询语言。
> - 意思是擅长处理"图"数据结构的查询，即多个数据对象，各个之间还有关联关系。
> - 核心概念：schema  rootValue

**2-5 选择Restful API 而非 GraphQL**
> 应用场景
> - 数据关系比较复杂
> - 前端查询需求多变
> - 有一个独立的数据提供方，对接很多使用方，不能一一定制开发

![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1616649572022-65c0b220-bd8b-4f81-b662-600053cb6446.png#height=962&id=Y0wTY&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1924&originWidth=1550&originalType=binary&ratio=1&size=516857&status=done&style=none&width=775)

**2-6 数据库设计-数据表结构**
> 注意，使用sequelize和mongoose，会自动创建id/createAt和updateAt,无需再自己手动创建。
> - 用户表 --讲了一下用户表的表结构
> - 作品/模版--讲了一下作品表的表结构
> - 渠道 -- 同上

**2-7 数据库设计-代码演示**
> src/models下的model设计

**2-8 server端架构设计**
![image.png](https://cdn.nlark.com/yuque/0/2021/png/358819/1623766298689-bca8dd70-10f3-4853-9ca7-cc518867810b.png#clientId=ue7f9533d-5750-4&from=paste&height=352&id=ue7c617a6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=703&originWidth=1498&originalType=binary&ratio=2&size=506157&status=done&style=none&taskId=u3ceeaae9-280e-4725-ad5d-2a5bf040151&width=749)

**2-9 技术方案设计-章总结**

> 领导技术方案设、评审技术方案设计

## 第三章：基础功能开发

**3-1 章介绍**
> - 登录功能设计
> - 用户信息接口
> - 作品接口
> - 模板接口

**3-2 登录功能设计-获取验证码**
> - 初次获取验证码
> - 再次获取验证码
> - 登录验证

**3-3 登录功能设计-划重点**
> - 缓存 -- 禁止频繁发送
> - 短信服务的提示和报警
> - 短信发送失败，不会缓存，可立即重新生成验证码
> - 短信服务挂掉，报警 

**3-4 制定开发规范**
> - 拉新分支 feature-xxx / fix-xxx/ hotfix-xxx
> - commit：参考cz-config,js
> - 往dev分之提交 pr Pull Requerst

**3-5 用户信息接口-获取登录验证码**
**接口**
> - 获取手机短信验证码
> - 登录(包含注册)
> - 获取用户信息
> - 修改用户信息

**代码演示**
> - routers/users.js
> - controller/users/
> - service/users.js
> - chache/users
> - __test__/apis/users,js

**3**-6 用户信息接口-登录
> 本节课读代码

**3-7 用户信息接口-接口测试**
> 讲的个鸡毛-依然是读代码

**3-8 作品接口-创建作品**
**接口**
> - 创建空白作品
> - 复制作品(通过模版创建)
> - 删除作品
> - 恢复作品
> - 转增作品
> - 我的作品列表(搜索 分页)
> - 我的回收列表(搜索 分页)
> - 查询单个作品信息
> - 保存作品

**代码演示**
> - routes/works.js
> - controller/works.js
> - service/works.js
> - __test__/api/works.js

**3-9 作品接口-查询、更新和复制**
> 同上代码演示

**3-10 作品接口-接口测试**
> 这个课程讲的真是垃圾。
> 你以为讲源码吗？或者课程代码是多难，拿着代码讲一遍，怎么想的？

**3-11 模版接口**
