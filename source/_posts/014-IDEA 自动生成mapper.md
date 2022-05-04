---
layout: post
title: IDEA 自动生成mapper
date: 2021-05-28
updated: 2022-05-04
toc: true
description: IDEA 自动生成mapper
categories:
- 前端小技
tags: 
- Webpack
---
<font color=blue>更新说明：对文章排版以及内容格式做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

## 第一步：
> 需要两个jar包(aymsp-alipay下备份)：
> -  mybatis-generator-core-1.4.0.jar
> -  mysql-connector-java-5.1.31.jar (数据库驱动)

## 第二步：
> 配置  generatorConfig.xml文件
> 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
 
<generatorConfiguration>
 
  <!--MySQL连接驱动-->
  <classPathEntry location="mysql-connector-java-5.1.31.jar" />
  
  <!--数据库链接URL，用户名、密码 -->
  <context id="MySQL" targetRuntime="MyBatis3">

    <commentGenerator>  
            <property name="suppressDate" value="true"/>  
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->  
            <property name="suppressAllComments" value="true"/>  
    </commentGenerator> 
    
    
    <!----------------- 第一个需要修改的地方 ----------------->
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
        connectionURL="jdbc:mysql://localhost:3306/aymspdb"
        userId="root"
        password="pass">
    </jdbcConnection>
 
    
    
    
    <!--是否启用java.math.BigDecimal-->
    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>
    
    <!----------------- 第二个需要修改的地方 ----------------->
    <!-- 生成模型的包名和位置--> 
    <javaModelGenerator targetPackage="org.ayfoundation.api.impl.app.alipay.entity.model" targetProject="output">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>
    
    
     <!----------------- 第三个需要修改的地方 ----------------->
    <!-- 生成映射文件的包名和位置-->
    <sqlMapGenerator targetPackage="test.xml"  targetProject="output">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>
    
    
     <!----------------- 第四个需要修改的地方 ----------------->
    <!-- 生成DAO的包名和位置--> 
    <javaClientGenerator type="XMLMAPPER" targetPackage="org.ayfoundation.api.impl.app.alipay.entity.dao"  targetProject="output">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>
    
    
     <!----------------- 第五个需要修改的地方 ----------------->
    <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名-->
    <table tableName="app_alipay_downloadbill" domainObjectName="AlipayBillInfo" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="true" selectByExampleQueryId="true">
    </table>
    
  </context>
</generatorConfiguration>

```
## 第三步：
> java -jar mybatis-generator-core-1.3.5.jar -configfile generatorConfig.xml

