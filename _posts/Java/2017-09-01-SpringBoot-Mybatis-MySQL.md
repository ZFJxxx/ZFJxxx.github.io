---
layout: post
title:   Spring boot项目中集成Mybatis和MySQL
date:   2017-09-01 19:15:10
categories:  Java
tags:  Mybatis
keywords: Mybatis
description: 
---


## 1.首先在pom.xml文件中导入Jar包

```
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot</artifactId>
	<version>1.1.1</version>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
```


## 2.在src --> main -->resources文件夹下添加 mybatis-config.xml 

**mybatis的配置文件**
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <!-- 是否需要缓存 -->
        <setting name="cacheEnabled" value="true"/>
        <!-- 读取时间的超时 -->
        <setting name="defaultStatementTimeout" value="3000"/>
        <!-- 驼峰语法 自动将数据库里的 head_url—>映射成 headUrl  -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- Allows JDBC support for generated keys. A compatible driver is required.
        This setting forces generated keys to be used if set to true,
         as some drivers deny compatibility but still work -->
        <setting name="useGeneratedKeys" value="true"/>
    </settings>

    <!-- Continue going here -->

</configuration>
```


## 3.在application.properties中添加连接

```
spring.datasource.url=jdbc:mysql://localhost:3306/myzhihu?useUnicode=true&characterEncoding=utf8&useSSL=false  //myzhihu是我连接的database名
spring.datasource.username=root  //这里输入你数据库的帐号
spring.datasource.password=123   //这里输入你数据库的的密码
mybatis.config-location=classpath:mybatis-config.xml
```