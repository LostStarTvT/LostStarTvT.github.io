---
layout: post
title: SpringMVC：框架学习
tags: java  
---


> 看Java SpringMVC视频学习笔记总结

##  目录
* 目录
{:toc}
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

### 2.1 HelloWorld处理流程。

  spring架构思想：

[![springMVC.png](https://pic.tyzhang.top/images/2020/04/07/springMVC.png)](https://pic.tyzhang.top/image/ddyA)

一个简单的helloworld数据处理流程：首先用户发送请求，前端控制器接收到请求以后，将请求分发给处理器映射器，处理器映射器根据map查找到对应的处理方法，返回给前端控制器，前端控制器将方法分配器处理器适配进行执行方法体，然后handler处理器执行方法，返回方法体参数，然后前端控制器将放回的数据分发给视图解析器，将数据解析并放回view视图，然后返回给用户进行显示。

[![spinrmvcHelloWorld.png](https://pic.tyzhang.top/images/2020/04/10/spinrmvcHelloWorld.png)](https://pic.tyzhang.top/image/dOsa)

### 2.2 springMVC的九大组件：

```java
//处理九大组件的逻辑。  服务器启动的时候就会进行初始化。
protected void initStrategies(ApplicationContext context) {
    //初始化文件的解析
    this.initMultipartResolver(context);
    //本地解析化
    this.initLocaleResolver(context);
    //主题解析
    this.initThemeResolver(context);
    //处理器映射
    this.initHandlerMappings(context);
    //处理器适配器
    this.initHandlerAdapters(context);
    //handler的异常处理器
    this.initHandlerExceptionResolvers(context);
    //当处理器没有返回逻辑视图名等相关信息时，自动将请求url映射为逻辑视图名
    this.initRequestToViewNameTranslator(context);
    //视图逻辑名称转换器，即允许返回逻辑视图名称，然后它会找到真实的视图
    this.initViewResolvers(context);
    //这是一个关注flash开发的Map容器，不在介绍。
    this.initFlashMapManager(context);
}
```

上面所说的就是springMVC的核心组件，

1. MultipartResolver:文件解析器，用于支持服务器的文件上传。
2. LocaleResolver:国际化解析器，可以提供国际化功能。
3. ThemeResolver:主题解析器，类似于软件皮肤的转换功能。
4. HandlerMapping:**处理器映射器**，SpringMVC中十分重要的内容，它会包装用户提供的一个控制器的方法和对它的一些拦截器，通过调用它便能够运行控制器。
5. handlerAdapter**:处理器适配器**，因为处理器会在不同的上下文中运行，所有springMVC会先找到合适的适配器，然后运行处理器服务器方法。
6. HandlerExceptionResolver:处理器异常解析器，处理器可能出现异常，产生异常便会用这个解析器进行处理。比如，转到指定的异常界面。
7. RequestToViewNameTranslator:视图逻辑名转换器，有时候控制器放回一个视图的名称，通过它可以找到实际的视图，当处理器没有返回逻辑图名等相关信息是，自动将请求url映射为逻辑视图名。
8. ViewResolver:**视图解析器**，当控制器返回后，通过视图解析器会把逻辑视图名称进行解析，然后定位实际视图。

其中处理器映射器、处理器适配器和视图解析器为spring的三大组件。在配置文件中使用以下注解，便导入了以上三大组件。

```xml
 <mvc:annotation-driven/> 
```

## 3.springMVC配置文件

spring的入口文件，里面记录了所有需要使用对象的之前的关系。

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
       http://www.springframework.org/schema/mvc/spring-mvc.xsd 			
       http://www.springframework.org/schema/context  
       https://www.springframework.org/schema/context/spring-context.xsd">
    <!--开启注解扫描-->
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

## 4.处理请求参数

### 4.1 url中带有参数

url:http://localhost:8080/user/student/aa

```java
@RequestMapping(value = "/student/{name}")
public String addStudent(@PathVariable("name") String name){
    System.out.println(name);
    return "success";
}
```

便能够直接的获取到aa这个值。

### 4.2  请求体中带有参数，表单

如果传递过来的的参数名与方法中的一致，可以直接的获取到。例如表单传递了三个参数data,phoneid,userName，那么就可以直接使用@Requestparam注解获取到：

```java
public String insertUserData(@Requestparam("data") String data,@Requestparam("phoneId") String phoneId,@Requestparam("userName") String usrName) throws JSONException {
	
    //转换成json数组
    JSONArray jsonArray = new JSONArray(data);
    return "success";
}
```

上面如果数据过多的话，直接写一个javabean进行传递数据。前面传递的参数与之一一对应即可。

### 4.3 直接获取reques对象

springMVC也提供了servlet封装的request对象，这个我觉是可以进行debug的时候测试，即看前端发送了什么数据。然后进行测试，能够更好的理解。

```java
@ResponseBody
@RequestMapping(value = "/users/user_keyboard_pressure_store",method = RequestMethod.POST)
//通过request 这个对象可以获得用户传递过来的所有数据，然后可以进行对应的获取，其他的注解也就是简化版的此对象，不需要知道所有的数据，可以使用debug进行测试知道发来的数据类型是什么样的。
//以后这样的写法要少写，因为这个是servlet的写法，对于之后的不太好维护。
public String insertUserData(HttpServletRequest request) throws JSONException {
	
    //转换成json数组
    String data = request.getParameter("data");
    JSONArray jsonArray = new JSONArray(data);
	//获取请求体里面保存的数据。
    String phoneId = request.getParameter("phoneId");
    String usrName = request.getParameter("userName");

    return "success";
}
```

也相当于以上4.2的写法。

## 5.RESTful风格的CURD实现 

REST全名：Representational state Transfer，资源表现层状态转化。

1. 资源（resources），网络实体，或者说是网络上的一个具体信息，可以是文本，图片，歌曲等等，可以使用一个url指向，类似于本地文档的路径。
2. 表现层(Respresentation) ，把资源具体呈现出来的形式。例如html、xml、json或者是二进制。
3. 状态转化(State Transfer)，即客户端对于服务器上的资源状态的更改，增删改查，对应的关键字为get查，post新建，put更新，delete删除。

对比于传统的通过具体动作名称url进行指定，如下所示：

1. /getBook?id=1;
2. /deleteBook?id=1;
3. /updateBook?id=1;
4. /addBook ；

rest风格的操作更为简洁，可以将网络资源像本地资源一样进行操作，并且url的命名会更加的规范。  

**eg: book/1 method = post  表示为新增id为1的book。**  

实现方法：首先需要在web.xml中进行配置过滤器，因为默认的只要get和post方法，put和delete方法需要通过过滤器来进行检测到。以下为配置过滤器。  

web.xml

```xml
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <!--  注意这里  -->
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

get、post方法为内置方法，以下 对于delete和put方法进行说明，  

index.jsp

```jsp
<%--其中1为请求参数--%>
<form action="/user/book/1" method="post">
    <%--添加了一个_name 的方法--%>
    <input name="_method" value="delete">
    <input type="submit" value="删除1号图书">
</form>
<form action="/user/book/1" method="post">
    <input name="_method" value="put">
    <input type="submit" value="删除1号图书">
</form>
```

controller.java

```java
//两个虽然url是相同的，但是请求方法是不一样的。
@RequestMapping(value = "/book/{bid}",method = {RequestMethod.PUT})
@ResponseBody //不加这个会测试不通过。
public String updateBook(@PathVariable("bid") Integer id){
    System.out.println(id);
    return "success";
}

@RequestMapping(value = "/book/{bid}",method = {RequestMethod.DELETE})
@ResponseBody
public String deleteBooke(@PathVariable("bid") Integer id){
    System.out.println(id);
    return "success";
}
```

## 5.数据持久化存储（保存并存储属性参数）

有时候我们会将数据暂存在HTTP的request中去，让其可以在视图中进行展示，即在jsp中可以获取到用户传递过来的数据，那么就需要用到属性参数的保存。SpringMVC中共有三个注解可以实现：

1. @RequestAttribute：获取request对象的属性值，用来传递给控制器参数

2. @SessionAttribute: 在Session对象属性中，用来传递给控制器参数

3. @SessionAttritubes:可以给他配置一个字符串数组，这个数组对应的是数据模型对应的键值对，也就是可以存储更多的值。

## 6.处理json数据，

### 6.1 使用jquery发送ajax数据

首先需要配置springMVC不要拦截静态资源，配置如下：

```xml
<!--告诉前端控制器，哪些静态资源不进行拦截-->
<!--以下表示/web/js 文件下的内容都不进行拦截。-->
<mvc:resources location="/js/" mapping="/js/**"/>
```

然后在index.jsp中发送ajax数据测试：

```jsp
 <script src="js/jquery-3.4.1.min.js"></script>
    <script>
        $(function () {
            $("#btn").click(function () {

                $.ajax({
                   url:"user/testAjax",
                    contentType:"application/json;charaset=UTF-8",
                    data:'{"username":"hehe","age":30}',
                    dataType:"json",
                    type:"post",
                    success:function (data) {
                        alert(data);
                    }
                });
                // alert("hello");
            });
        });
    </script>

<button id="btn">发送ajax请求</button>
```

### 6.2 接受和返回json数据

使用@RequestBody注解以string对象的方法获取json数据：

```java
//模拟异步的ajax模拟
@RequestMapping(value = "/testAjax")
public void testAjax(@RequestBody String user){
    System.out.println("收到了ajax数据");
    System.out.println(user);
}

//附user bean
public class User {
    private String username;
    private String age;
    //get and set
}
```

使用jackson将json转成对象。

1.首先在pom文件中导入需要的jar包依赖。

```xml
<!--Jackson required包-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.0</version>
</dependency>
```

2.系统便会自动的将json数据转成user对象。加上ResponseBody注解，系统会将user对象转成json返回。

```java
//模拟异步的ajax模拟
@RequestMapping(value = "/testAjax")
public @ResponseBody User testAjax(@RequestBody User user){
    System.out.println("收到了ajax数据");
    //此时的数据已经将其转换成了对象。
    System.out.println(user);
    user.setAge("11");
    return user;
}

//附user bean
public class User {
    private String username;
    private String age;
    //get and set
}
```

## 8.视图和视图解析器

请求处理方法执行完成后，最终返回一个ModelAndView对象，对于那些返回String，view或ModeMap等类型的处理方法，SpringMVC也会在内部将它们转配成一个ModeAndView对象，它包含逻辑名和模型对象的实体。SpringMVC借助视图解析器得到最终的视图对象，最终的视图可以是JSP，也可以是Excel、JFreeChart等各种表现形式的视图。  

**什么叫做视图？**  

视图的作用就是渲染模型数据，将模型中的数据以某种形式展示给用户。SpringMVC中定义了一个抽象接口View，具体的实现由视图解析器将视图对象进行实例化。另外，由于视图是无状态的，所有他们不会有线程安全问题。

## 9.拦截器的使用

SpringMVC提供了拦截器机制，即允许在运行controller 方法**之前或之后**运行一些方法，作为数据的筛选或者拦截工作。其实也就是相当于安卓中的声明周期。

filter：javaWEB实现。    

HandelerInterceptor拦截器：SpringMVC实现。  

1. preHandle：在目标方法之前进行调用，return true 放行，false 不放行。
2. postHandle： 在目标方法运行之后运行。
3. afterCompletion： 在整个请求完成以后执行。

[![interceptor.png](https://pic.tyzhang.top/images/2020/04/13/interceptor.png)](https://pic.tyzhang.top/image/dVG0)

实现方法：  

首先新建一个自定义拦截器。  

```java
//每个方法都会传进来resquest和response对象，这样便能够对其进行操作。
public class MyFirstInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

进行springMVC的配置，即让容器加载这个自定义的容器。以下为配置多个拦截器的方法。其中将bean标签更改自己的class即可。  

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <!-- 多个拦截器，按顺序执行 -->        
    <mvc:interceptor>
        <mvc:mapping path="/**"/> <!-- 表示拦截所有的url包括子url路径 -->
        <bean class="ssm.interceptor.HandlerInterceptor1"/>
    </mvc:interceptor>
    
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="ssm.interceptor.HandlerInterceptor2"/>
    </mvc:interceptor>
    
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="ssm.interceptor.HandlerInterceptor3"/>
    </mvc:interceptor>
</mvc:interceptors>
```

对于多个拦截器来说，其运行是属于栈的运行近结构，如下,三个拦截器都放行。

```xml
HandlerInterceptor1….preHandle
HandlerInterceptor2….preHandle
HandlerInterceptor3….preHandle

HandlerInterceptor3….postHandle
HandlerInterceptor2….postHandle
HandlerInterceptor1….postHandle

HandlerInterceptor3….afterCompletion
HandlerInterceptor2….afterCompletion
HandlerInterceptor1….afterCompletion
```

[参考链接](https://blog.csdn.net/eson_15/article/details/51749880)  

### 10. 其他知识

对于springMVC还有其他的东西，比如说文件的上传和jsr验证，即数据的格式验证，比如说邮箱格式，密码长度等等。国际化等操作，国际化即为在更改浏览器语言的时候，所有的文字会变成英文，这些都不在赘述。还有就是网络请求的请求头session 和cookie的设置，有机会在进行更新。  

另外，就是重定向，使用关键字forward和redirect关键字。

```java
@RequestMapping(value = "/handler")
public String handler(){
  
    return "redirct:/hello.jsp";
    //return "forward:/hello.jsp";
}
```