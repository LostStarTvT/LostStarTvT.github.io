---
layout: post
title: Java Annotation
tags: java
---


> java Annotation 底层原理实现记录。

##  目录
* 目录
{:toc}
# 总述

注解只是个元数据并且没有任何的业务逻辑(**Annotations are only metadata and do not contain any business logic**)，自定义注解主要是和反射进行一起使用，即正常的创建对象的方式是获取不到定义在类中或者方法中的注解的，必须使用反射或则是动态代理的方式进行获取到注解然后根据注解进行相应的操作。  

参考链接：[java注解详解](https://www.jianshu.com/p/596d389282a0)  [How Do Annotations Work in Java?](https://dzone.com/articles/how-annotations-work-java)   [Java注解](https://www.runoob.com/w3cnote/java-annotation.html)

# 一、元数据

元数据是关于数据的数据。在编程语言上下文中，元数据是添加到程序元素如方法、字段、类和包上的额外信息。对数据进行说明描述的数据。  

**为什么需要元数据？**  

通俗的含义也就是将配置的内容进行读取出来，就比如说一个人，有很多的属性，对应的属性需要配置很多的值，这些值虽然可以直接的在new的时候进行配置，这样耦合性就很高，所以可以通过元数据去配置，即将属性的值抽取出来，写在xml中，进行去读，这样的话就可以xml也就是表示元数据。这是这之前，但是之后就是使用注解来进行配置元数据。更高级的用法就是使用注解来配置元数据。

以下说明了XML vs. Annotation.  

> *Suppose, you want to set some application-wide constants/parameters. In this scenario, XML would be a better choice because this is not related to any specific piece of code. If you want to expose some method as a service, an annotation would be a better choice as it needs to be tightly coupled with that method and developer of the method must be aware of this.*

# 二、怎么写一个注解

首先是@Override注解的源码，如下所示

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
    
}
```

它的作用就是检查是否在父类中定义了一个方法，子类在重写的过程中格式是否正确，多用于继承和实现接口中。平时的写代码可以发现，即使不写`@Override`代码也是可以正常的运行，这也就说明了注解其实对于代码的逻辑是没有影响的。 

从上面的注释可以看出，其中`@Target`表示这个注解只能用在METHOD方法上，它的作用域为SOURCE源码中，表示在运行的时候就把这个注解丢掉，不会影响程序的运行效果。  

java注解主要分为元注解+内建注解+自定义注解。以下分别介绍：

## 1. java元注解

元注解即为java提供的最基础的注解，其他的注解都是基于这些进行实现的。

1. `@Documented`: 被其修饰的注解将会被javadoc工具提取成文档。
2. `@Inherited`: 被其修饰的注解将具有继承性。即控制注解是否影响子类。
3. `@Retention`: 定义被修饰注解的作用域，使用RetentionPolicy进行获取。
   1. SOURCE: 仅存在Java源文件，经过编译器后便丢弃相应的注解。
   2. CLASS: 存在Java源文件，以及经编译器生成的CLass字节码文件，但是在运行JVM时不再保留注释。
   3. RUNTIME:存在源文件、编译生成的class字节码文件，以及保留在运行时JVM中，可以通过反射进行读取注解。
4. `@Target`:表示该注解类型的所适用的程序元素类型。使用ElementType进行获取。
   1. `ANNOTATION_TYPE`: 注解类型声明。
   2. `CONSTRUCTOR` :构造方法声明。
   3. `FIELE`: 字段声明(包括枚举类型)。
   4. `LOCAL_VARIABLE`: 局部变量声明。
   5. `METHOD` : 方法声明。
   6. `PACKAGE`：包声明。
   7. `PARAMETER`: 参数声明。
   8. `TYPE`: 类、接口（包括注解类型）或枚举类型声明。

## 2. 内建注解

Java提供了多种内建的注解，下面接下几个比较常用的注解：`@Override`、`@Deprecated`、`@SuppressWarnings`以及`@FunctionalInterface`这4个注解。内建注解主要实现了元数据的第二个作用：**编译检查**。

1. `@Override` :主要是告知编译器来进行检查当前重写的方法与父类的方法是否一致。
2. `@Deprecated`:告诉编辑器此方法不建议使用了。即对于该方法出现一条横线。
3. `@SuppressWarnings`: 用于告知编译器忽略特定的警告信息，例在泛型中使用原生数据类型，编译器会发出警告，当使用该注解后，则不会发出警告。
4. `@FunctionalInterface`: 告知编译器，检查这个接口，保证该接口是函数式接口，即只能包含一个抽象方法，否则就会编译出错。

即内建注解就是基于元注解来实现的java自己定义的注解。那么问题就来了，如何用户自定义注解？ 即类比于spring定义了大量的注解，那么自己定义注解是如何的？

## 3. 自定义注解

自定义注解前需要知道的是三个类，即注解主干类，自定义注解中主要使用的就是组合三个类。

```java
//所有注解的父类，自定义注解都会继承这个接口。
package java.lang.annotation;
public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    String toString();

    Class<? extends Annotation> annotationType();
}

//枚举类型的 ElementType，用于表明注解的作用域
package java.lang.annotation;
public enum ElementType {
    TYPE,               /* 类、接口（包括注释类型）或枚举声明  */

    FIELD,              /* 字段声明（包括枚举常量）  */

    METHOD,             /* 方法声明  */

    PARAMETER,          /* 参数声明  */

    CONSTRUCTOR,        /* 构造方法声明  */

    LOCAL_VARIABLE,     /* 局部变量声明  */

    ANNOTATION_TYPE,    /* 注释类型声明  */

    PACKAGE             /* 包声明  */
}

//枚举类型的声明周期。
package java.lang.annotation;
public enum RetentionPolicy {
    SOURCE,            /* Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了  */

    CLASS,             /* 编译器将Annotation存储于类对应的.class文件中。默认行为  */

    RUNTIME            /* 编译器将Annotation存储于class文件中，并且可由JVM读入 */
}
```

创建自定的注解与创建接口的方法有些类似，但是必须要使用`@interface`进行标记，表明隐式的继承了`Annotation `接口

```java
@Documented
@Target(ElementType.METHOD)
@Inherited
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotataion{
    String name();
    String website() default "hello";
    int revision() default 1;
}
```

上面的注解就是表明，这个注解会被javadoc工具提取成文档，然后只能使用在METHOD方法上面，是可以被继承的，并且会被将会被jvm读取到，其中里面定义了三个属性，在使用时是可以被赋值的，但是里面的值只能被使用反射获取到，所以这也是为什么注解会大量的被运用在框架中，因为框架天生就是需要使用反射动态代理，所以说天生能够读取到注解上的东西。

使用方法：类比于spring中的注解的使用方法，里面也是可以直接的定义属性并且赋值的，另外数组也是可以使用的。

```java
public class AnnotationDemo {
    @MyAnnotation(name="lvr", website="hello", revision=1)
    public static void main(String[] args) {
        System.out.println("I am main method");
    }

    @SuppressWarnings({ "unchecked", "deprecation" })
    @MyAnnotation(name="lvr", website="hello", revision=2)
    public void demo(){
        System.out.println("I am demo method");
    }
}
```

获取注解中的数据：

```java
public class AnnotationParser {
   public static void main(String[] args) throws ClassNotFoundException {
       //定义对应的包名 
       String clazz = "com.github.yeecode.easyrpc.client.demo.AnnotationDemo";
       //通过反射获取到类中定义的方法
       Method[]  demoMethod = AnnotationParser.class.getClassLoader().loadClass(clazz).getMethods();
		//遍历出来具有注解的方法，然后获取上面的注解进行解析，虽然只有几个方法，但是因为都是继承object也会有很多的方法可以遍历。
       for (Method method : demoMethod) {
           if (method.isAnnotationPresent(MyAnnotation.class)) {
               MyAnnotation annotationInfo = method.getAnnotation(MyAnnotation.class);
               System.out.println("method: "+ method);
               System.out.println("name= "+ annotationInfo.name() +
                                  " , website= "+ annotationInfo.website()
                                  + " , revision= "+annotationInfo.revision());
           }
       }
    }
}
```

运行结果：遍历出来了两个有自定义的注解，然后还获取到了对应的注解里面的值，也就是spring底层注解实现的原理。

注解+反射控制，即注解作为标记，注解里面的值作为需要的元数据，然后进行遍历操作。

```shell
method: public static void com.github.yeecode.easyrpc.client.demo.AnnotationDemo.main(java.lang.String[])
name= lvr , website= hello , revision= 1
method: public void com.github.yeecode.easyrpc.client.demo.AnnotationDemo.demo()
name= lvr , website= hello , revision= 2
```

