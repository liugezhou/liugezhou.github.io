---
layout: post
title:  CSS-伪类和伪元素
author: liugezhou
date: 2021-07-10
updated: 2021-07-10
category: 
    CSS
---
## 背景
> 写了这么多年代码，对CSS中的伪类和伪元素竟然没有细致的进行过学习总结，由此可见在实际代码开发中，用的也确实不多，也就用过一些:first-child,:hover之类的吧，其它的连before什么的都没用过，于是迫切需要大于伪元素与伪类进行一个系统整体的学习。

## 伪类和伪元素
> - **伪类：**是以一个冒号作为前缀，被添加到选择器的末尾，当你希望在**特定状态**下(:hover)才被呈现到指定元素时，可以往元素的选择器后面加上伪类。
> - **伪元素**：用于创建一些**不在文档树中的元素**，并为其添加样式。比如::before是指得元素前添加文本，且为文本添加样式，虽然用户可以看到这些文本，但文本实际不在DOM结构中。


## 常用的伪类和伪元素

> 伪类可以从状态类伪类、结构类伪类、其它伪类和表单相关伪类进行分类。
> - 状态类伪类：** :hover、:link、:active、:visited、:focus**
> - 结构类伪类：** :first-child、:last-child、:nth-child(n)**
> - 其它伪类：   :fullscreen全屏显示、:lang()匹配指定语言
> - 表单相关伪类：  :checked选中、:disabled禁用、:required必填、:read-only只读
> 
 
> 伪元素：**::before、::after、::first-letter、::first-line、::selection、::placeholder**


## 伪元素::berfore与::after的用法

> 在被选中元素的之前和之后插入内容，其中content属性指定要加入的内容，content必须有值(空值也可以)，其默认伪inline，可以通过display:block改变.
	content属性：**空值、字符串、url、attr、counter。**
>  
> - 空值可以清除浮动


```css
.clearFix::after{
  clear:both,
    display:block;
  content:'',
    height:0;
  overflow:hidden
}
```

> - 字符串可以直接添加内容，不赘述。
> - url：引用媒体文件(比如图片)
> - attr:可以调用当前元素内的某个属性(比如a标签的href属性)
> - counter计数器：可以不使用列表元素实现序号功能。


```css
<body>
<h1>Good Coder</h1>
<h2>Liu</h2>
<h2>Zhou</h2>
<h1>Bad Coder</h1>
<h2>Lazy</h2>
<h2>Not Attentioin</h2>
body{
  counter-reset:section
}  
h1{
  counter-reset:subsection
}
h1::before{
  counter-increment:section;
  content:counter(section)'\';
}
h2::before{
  counter-increment:subsection;
  content:counter(section)'.'counter(subsection);
}
</body>
```
