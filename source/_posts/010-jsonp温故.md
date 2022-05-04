---
title: jsonp温故
date: 2019-08-11 20:00:00
updated: 2022-05-04
toc: true
description: 今天重新学习之前写了半截的项目，其中提到了jsonp，当时也是查了很多资料，做了很多笔记，但是最近在写一个项目的时候，竟然遗忘了很多，所以特此做个总结，在下次再遇到jsonp的时候，可以有一个清晰的认识。
categories:
- 前端小技
tags: 
    - 前端跨域
---
<font color=blue>更新说明：对文章排版以及内容格式做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

> 今天重新学习之前写了半截的项目，其中提到了jsonp，当时也是查了很多资料，做了很多笔记，但是最近在写一个项目的时候，竟然遗忘了很多，所以特此做个总结，在下次再遇到jsonp的时候，可以有一个清晰的认识。
<!--more-->
## jsonp原理介绍
>jsonp就是为了解决前端的跨域问题而进行的一项设计，jsonp之所以能实现跨域，是因为`它发送的不是ajax请求`，它`动态创建了script标签`，script标签是不受同源策略限制的，将script的src指向正式的服务器地址。

*查找资料：*
> 目前为止(2012年)最被推崇或者说首选的方案还是用JSON来传数据，靠JSONP来跨域。  
> JSON是一种数据交换格式，而JSONP是一种依靠开发人员的聪明才智创造出的一种非官方跨域数据交互协议。

## JSONP是怎么产生的：
> 1、一个众所周知的问题，Ajax直接请求普通文件存在跨域无权限访问的问题，甭管你是静态页面、动态网页、web服务、WCF，只要是跨域请求，一律不准； 
>
> 2、不过我们又发现，Web页面上调用js文件时则不受是否跨域的影响（不仅如此，我们还发现凡是拥有"src"这个属性的标签都拥有跨域的能力，比如`<script>`、`<img>`、`<iframe>`）；
>
> 3、于是可以判断，当前端如果想通过纯web端（ActiveX控件、服务端代理、属于未来的HTML5之Websocket等方式不算）跨域访问数据就只有一种可能，那就是在远程服务器上设法把数据装进js格式的文件里，供客户端调用和进一步处理；
>
> 4、恰巧我们已经知道有一种叫做JSON的纯字符数据格式可以简洁的描述复杂数据，更妙的是JSON还被js原生支持，所以在客户端几乎可以随心所欲的处理这种格式的数据；
> 
> 5、这样子解决方案就呼之欲出了，web客户端通过与调用脚本一模一样的方式，来调用跨域服务器上动态生成的js格式文件（一般以JSON为后缀），显而易见，服务器之所以要动态生成JSON文件，目的就在于把客户端需要的数据装入进去。
>
> 6、客户端在对JSON文件调用成功之后，也就获得了自己所需的数据，剩下的就是按照自己需求进行处理和展现了，这种获取远程数据的方式看起来非常像AJAX，但其实并不一样。
>
> 7、为了便于客户端使用数据，逐渐形成了一种非正式传输协议，人们把它称作JSONP，该协议的一个要点就是允许用户传递一个callback参数给服务端，然后服务端返回数据时会将这个callback参数作为函数名来包裹住JSON数据，这样客户端就可以随意定制自己的函数来自动处理返回数据了。

## 安装
> npm install jsonp --save

## [GithubAPI](https://github.com/webmodules/jsonp)
> API介绍很简洁，下面为全文。
> 
> //语法：
> **jsonp(url,opts,fn)**
```
url (String) url to fetch   
<!-- > 要获取的网址 -->
opts(Object) ,optional       
<!-- 一个参数对象   -->
·param(String) name of the query string parameter to specify the callback (default to callback)
<!-- 用于指定回调的查询字符串参数的名称 (默认为callback) -->
·timeout(Number) how long after a timeout error is emitted. 0 to disable(default to 60000)
<!-- 超时错误多长时间后出发。 0表示禁用（默认为60s） -->
·prefix(String) prefix for the global callback functions that handle jsonp responses(default to __ip)
<!-- 处理jsonp响应的全局回调函数的前缀 -->
·name(String) name of the global callback funcitions that handle jsonp responses(default to `prefix` + incremented counter)
<!-- 处理jsonp响应的全局回调函数的名称 -->
fn callback 
The callback is called with `err`,`data` parameters.
<!-- 使用`err`，`data`参数调用回调。 -->
If it times out ,the err will be an ERROR object whose message is Timeout.
<!-- 如果超时，则错误将是ERROR对象，其消息为Timeout。 -->
Return a function that ,when called,will cancel the in-progress jsonp request( `fn` wont't be called)
<!-- 返回一个函数，当出现错误时，将取消正在进行的jsonp请求（`fn`不会被调用） -->
```