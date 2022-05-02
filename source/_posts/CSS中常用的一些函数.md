---
layout: post
title:  CSS中常用的一些函数
author: liugezhou
date: 2021-11-12
category: 
    CSS
---
### calc用法

> - calc()函数支持加减乘除四种运算。
> - 书写上一定要注意啊，**加减号两边一定要有空格**
> - 任何长度值都可以使用calc函数进行计算:% vh vw px em等
> - calc函数使DOM元素更加灵活的响应视口改变，还可以通过calc函数实现元素的绝对剧中定位(postion:absoute;top(50vh-Xpx))


### min函数(max函数)

> min()函数支持一个或多个表达式，每个表达式用逗号分隔，将最小的值作为返回值


### clamp函数

> 语法为clamp(MIN,VAL,MAX)：作用是返回一个区间范围内的值，首选值为VAL，大于MAX使用MAX，小与MIN，使用MIN。
相当于min(MIN,min(max,MAX))


### **变量使用(var函数)**

> - CSS声明变量，需要在声明的变量前加两根连接线：`--`,需要注意变量名大小写敏感。且注意如果变量值有单位，不能写成字符串。
> - var函数用于读取变量，var函数还可以接收第二个参数，表示变量的默认值，即如果变量不存在，就使用默认值。
> - 利用css动态根据页面变化导致的规则变化，可以在响应式布局中使用media声明全局变量，使得不同的屏幕宽度有不同的变量值。


### line-gradient

> - 用于创建一个线性渐变的图像，需要设置起点的方向和渐变的起始颜色。
> - 未设置方向，默认是从上到下。
> - 语法 line-gradient`[<angel> | <side-or-corner>],stopColor1,stopColor2`


### radial-gradient

> - 与line-gradient类似，语法为`[shape size position],stopColor1,stopColor2`
> - shape:指明径向渐变的形状，可以为circle和ellipse.
> - size:代表径向渐变范围的半径大小，shape为ellipse时需要设置两个值、、shape为circle时不能设置百分比。
> - postion:默认值是中心点，支持关键字，支持距离左上角的角标位置(px或百分比单位)


```
  .radial_1 {
    /*最简单的渐变：由中心到四周，由蓝色到黄色*/
    background-image: radial-gradient(blue, yellow);
  }
  .radial_2 {
    /*半椭圆形渐变：由左侧中心点到四周，有蓝色到黄色*/
    background-image: radial-gradient(ellipse 100% 50% at left center, blue, yellow);
  }
  .radial_3 {
    /*左上角到右下角的发散式渐变*/
    background-image: radial-gradient(circle farthest-corner at left top, blue, yellow);
  }
  .radial_4 {
    /*指定颜色渐变范围*/
    background-image: radial-gradient(ellipse 50% 30%, blue 30%, yellow 70%);
  }
```
