---
layout: post
title: scss在项目实战中的使用
author: liugezhou
date: 2021-11-15
updated: 2021-11-15
category: 
    CSS
tags:
    scss
---
> SCSS在项目中的使用比较常用的有几下几种：

## 1. 变量使用
> - 全局使用：使用$varaible格式定义变量，比如全局的主题色，可在common.scss中定义，通过@import的方式引用即可
> - 局部使用：在本文件中创建变量$themeColor = red,然后直接使用，存在块级作用域。

> CSS原生可通过定义	`-- 变量名`结合var函数的方式来达到这一目标。

## 2. 混合使用(mixins)
> - 可在common.scss中使用@mixin varibaleName{}的方式定义 多次重复使用的样式，通过@include的方式应用。 
> - 还可以使用@mixin varibaleName(varib1 varib2 varib3){} 的方式传入自定义的属性，进行代码复用，比如可以将 flex布局使用mixin的形式，传入变量使用。

## 3. 嵌套
> 嵌套功能避免了重复输入父选择器，令复杂的CSS结果更易于管理。

## 4. 导入
> - @import 导入，文件扩展名为.scss或.sass
> - 可同时导入多个文件 @import 'bar','foo';

## 5. &使用
> - 在嵌套 CSS 规则时，有时也需要直接使用嵌套外层的父选择器，比如hover、first-child等使用。
> - &还有一个使用情况是: .main{ &-content{}}，这里经过编译后就是 .main-content.

