---
layout: post
title: SpringBoot入门
tags:  java

---
> 本篇主要介绍使用SpringBoot的基础知识，进行搭新的项目的和项目的解析。

## Spring boot文件目录

其中pom.xml文件就是整个项目的配置文件，相当于安卓的gradle配置文件。 也即是可以在这里面添加依赖文件等其他的东西。但是在进行更新  。

默认生成的Spring boot项目：

- 主程序已经生成好，我们只需要自己的逻辑即可
- resources 文件夹目录结构
  - static:保存所有的静态资源：css js images
  - templates：保存所有的模板界面；（Spring Boot默认的jar包使用嵌入式的Tomcat，默认不支持jsp页面）；可以使用模板引擎(freemarker thymeleaf)。
  - application.properties  更改Spring boot的默认配置。eg server.port = 8081  则更改了用户端口

