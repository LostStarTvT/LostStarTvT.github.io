---
layout: post
title: SpringMVC 框架学习
tags: java  
---


> 看Java SpringMVC视频学习笔记总结

## 0.说明

[视频链接](https://www.bilibili.com/video/BV1d4411g7tv?p=320) b站的spring springMVC MyBatis视频课程，这个课程其实比较老，而且最新版的spring源码已经更改，但是其思想还是没有变的，学习基础的话还是可以看。  

## 1.MVC架构模式

MVC提倡，每一层都只编写自己的东西，不写任何其他的代码。主要作用就是为了分层，而分层的目的是为了解耦，解耦是为了维护的方便的分工合作。MVC架构模型如下所示：

[![MVC.png](https://pic.tyzhang.top/images/2020/04/07/MVC.png)](https://pic.tyzhang.top/image/d8Hc)

而在MVC中，对应的关系如下：

- M: Model模型：封装和映射数据(JavaBean)
- V: View视图：界面的显示工作(.jsp)
- C: Controller控制器：控制整个网站的跳转逻辑(Servlet)，相当于路由:
  - 调用业务逻辑处理
  - 跳转到某个页面

对应的包的分类：  

- com.diaowenjie.entities（实体）
- com.diaowenjie.dao
- com.diaowenjie.service
- com.diaowenjie.controller

## 2.SpringMVC概述

- Spring 为展现层提供了基于MVC的设计里面的优秀WEB框架，
- 通过一套MVC注解，让POJO成为处理请求的控制器，而不需要实现任何接口
- 支持REST风格的URL请求
- 采用松散耦合的可插拔组件结构，比其他MVC框架更具有扩展性和灵活性。

另外与spring完美切合，即spring为springMVC提供后台的支持。  

另外springMVC的MVC实现思想：

[![springMVC.png](https://pic.tyzhang.top/images/2020/04/07/springMVC.png)](https://pic.tyzhang.top/image/ddyA)

## 3.springMVC配置文件

### 3.1配置文件

resources/springmvc.xml。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd 			http://www.springframework.org/schema/context  https://www.springframework.org/schema/context/spring-context.xsd">
    <!--开启注解扫描扫描-->
    <context:component-scan base-package="cn.dwj"/>

    <!--视图解析器-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--即返回视图的前缀拼接-->
        <property name="prefix" value="/WEB-INF/pages/"/>
        <!--返回视图的后缀拼接-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--开启springMVC框架注解支持-->
    <mvc:annotation-driven/>
</beans>
```



