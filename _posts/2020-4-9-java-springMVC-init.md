---
layout: post
title: SpringMVC:使用idea创建项目
tags: java  
---


> 记录使用maven创建springMVC项目的过程，并使用tomcat服务器进行运行项目。

##  目录
* 目录
{:toc}
## 1.创建maven项目

创建spring项目的时候，尽量都要使用maven进行创建，这样就不需要进行导包测试，就会很方便。以下是创建过程：  

1.按照图片上的123进行操作。首先新建一个maven工程，然后选择Create from archetype，这样maven便会自动帮你配置成一个web项目，点击next按钮。  

[![maven1.png](https://pic.tyzhang.top/images/2020/04/09/maven1.png)](https://pic.tyzhang.top/image/dCsT)

2.点击下一步，进行取包名和项目名字，然后点击next。  

[![maven2.png](https://pic.tyzhang.top/images/2020/04/09/maven2.png)](https://pic.tyzhang.top/image/d95n)

3.在这里按照可以设置让maven更快的下载依赖，首先点击1的加号，然后输入图片中的内容，然后点击ok下一步，这样便能在新建项目的时候依赖不会慢慢的下载，很快就能建立好项目。  

```java
archetyoeCatalog
internal
```

[![maven3.png](https://pic.tyzhang.top/images/2020/04/09/maven3.png)](https://pic.tyzhang.top/image/dDvS)

4.在next以后，便会到了finish界面，此时如果项目存储路径对的话，就直接finish即可。    

5.建好的项目界面。此时需要进行更改项目目录，以下为新建好项目的原始样貌。    

[![maven5.png](https://pic.tyzhang.top/images/2020/04/09/maven5.png)](https://pic.tyzhang.top/image/dMn9)

6.对比上图需要创建java和resources目录，并且分别对其右键将java文件夹更改为source root 、将resources文件夹更改为resources root目录。如下图所示。这样整个的目录结构便完成。  

[![mavenfile2.png](https://pic.tyzhang.top/images/2020/04/09/mavenfile2.png)](https://pic.tyzhang.top/image/dT9p)  

[![mavenfile.png](https://pic.tyzhang.top/images/2020/04/09/mavenfile.png)](https://pic.tyzhang.top/image/d3XH)

## 2.配置文件

首先需要更改pom.xml文件，添加项目所需要的依赖，以后需要的依赖也是直接从这里面进行添加。在没有maven之前都是复制jar包，现在直接更改pom文件，系统会自动的下载。直接更改dependencies标签下面的东西。

```xml
<dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.9.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.1.9.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.1.9.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
     <groupId>javax.faces</groupId>
      <artifactId>jsf-api</artifactId>
      <version>2.0.3</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
```

因为是springMVC项目，所以需要新建一个springMVC配置文件，也就是整个springMVC的入口文件，其目录在自己新建的文件夹下resources/springmvc.xml，以后项目中需要的东西也是在这里进行配置。  

另外，配置文档其实也就是在进行新建对象，即处理对象之间的依赖和调用关系，另外配置文档也可以通过java代码实现。  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd 
        http://www.springframework.org/schema/context 
        https://www.springframework.org/schema/context/spring-context.xsd">
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

然后配置tomcat项目的配置文件，即webapp/WEB-INF/web.xml，这个是tomcat web项目的入口文件。

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
  
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <!--去加载springMVC配置文件。-->
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath*:springmvc.xml</param-value>
    </init-param>
    <!--启动的时候就要开始创建。相当于优先级。-->
    <load-on-startup>1</load-on-startup>
  </servlet>

  <!--拦截所有的请求。-->
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

到此为止，所有的配置文件已经配置完成，整体流程就是，先配置pom文件加载到需要的jar包，即依赖。然后配置springMVC框架的配置文件springmvc.xml，然后配置tomcat项目的配置文件web.xml。其中web.xml文件设置springmvc配置文件的入口。

## 3.配置tomcat运行环境

因为新建的配置为一个项目，需要有对应的tomcat容器进行运行。  

首先点击add configuration 按钮，跳转到新增界面，然后点击左上角加号，找到对应的tomcat配置界面，并选择local。如下所示： 图标1表示可以更改tomcat运行环境的名字，图标2表示选择自己本地下载的tomcat运行程序，没有的需要点击config进行配置，建议下载到项目的相同目录，直接下载解压好就行。  

[![tomcat.png](https://pic.tyzhang.top/images/2020/04/09/tomcat.png)](https://pic.tyzhang.top/image/d52N)

点击上图的3，可以进入到部署界面，tomcat需要进行部署才能正常的运行项目。如下图所示，首先点击下图的1，新增加部署，然后选择2表示将本项目部署到tomcat根目录下webapp文件夹中，因为每次tomcat运行的项目都是在它自己根目录下webapp中的程序，所有需要部署，部署即IDE自动的将开发的程序复制到tomcat对应文件夹中。点击ok即可。

[![tomcat2.png](https://pic.tyzhang.top/images/2020/04/09/tomcat2.png)](https://pic.tyzhang.top/image/dJxt)

下图有个application contex选项，这个选项的目的就是在tomcat运行时候，url路径是否带上项目文件名。主要是因为tomcat可以支持同时运行多个web项目，他们是根据文件路径进行区分的，如果只有一个项目建议选择1，表示直接localhost:8080/ 表示该项目的根目录。否则就是localhost:8080/spring-demo-war/为访问的路径。然后点击OK，表示项目创建完成。

[![tomcat3.png](https://pic.tyzhang.top/images/2020/04/09/tomcat3.png)](https://pic.tyzhang.top/image/diVZ)

## 4. 测试helloworld

只需要设置三个文件便能够测试成功输出helloworld。

首先创建一个controller控制器，路径为java/cn.dwj.controller.HelloController

```java
package cn.dwj.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * Describe: 返回success.jsp界面。
 * @author Seven on 2020/4/9
 */
@Controller
public class HelloController {

    @RequestMapping(path = "/hello")
    public String sayHello(){
        System.out.println("hello");
        return "success";
    }
}
```

新建success.jsp文件。路径webapp/WEB-INF/pages/success.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h3>跳转成功</h3>
</body>
</html>
```

在index.jsp中设置跳转连接 。路径webapp/index.jsp。也是项目的入口文件。

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h3>
    入门程序
    <a href="hello">入门程序</a>
</h3>
</body>
</html>

```

运行程序以后便能点击访问到对应的success界面。  

## 5. 流程解析

服务器启动以后，便开始加载配置文件web.xml，加载springmvc配置文件，生成对象，然后处理请求。过程如下所示。

[![flow.jpg](https://pic.tyzhang.top/images/2020/04/09/flow.jpg)](https://pic.tyzhang.top/image/d0a8)

