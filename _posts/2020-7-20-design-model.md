---
layout: post
title: Java：设计模式总结
tags: java
---


> 记录Java中的各种设计模式。

##  目录
* 目录
{:toc}
在进行看框架源码时候，必须要知道各种的设计模式才能更好的理解其设计理念。

# 单例模式

一种最常用的模式，在spring中大量使用，默认的Bean就是一个单例的，所谓单例也就是在获取对象的时候返回的都是同一个对象，不会创建第二个对象，类比于现实生活中的一个班级班长只有一个。

## 一、懒汉加载单例

该模式的特点是类加载时没有生成单例，只有当第一次调用 getlnstance 方法时才去创建这个单例。

```java
public class LazySingleton
{
    private static volatile LazySingleton instance=null;    //保证 instance 在所有线程中同步
    private LazySingleton(){}    //private 避免类在外部被实例化
    public static synchronized LazySingleton getInstance()
    {
        //getInstance 方法前加同步
        if(instance==null)
        {
            instance=new LazySingleton();
        }
        return instance;
    }
}
```

注意：如果编写的是多线程程序，则不要删除上例代码中的关键字 volatile 和 synchronized，否则将存在线程非安全的问题。如果不删除这两个关键字就能保证线程安全，但是每次访问时都要同步，会影响性能，且消耗更多的资源，这是懒汉式单例的缺点。

## 二、饿汉加载单例

该模式的特点是类一旦加载就创建一个单例，保证在调用 getInstance 方法之前单例已经存在了。

```java
public class HungrySingleton
{
    private static final HungrySingleton instance=new HungrySingleton();
    private HungrySingleton(){}
    public static HungrySingleton getInstance()
    {
        return instance;
    }
}
```

# 工厂模式

工程模式也是一种很常用的模式，对于工厂来说最重要的就是解耦和好扩展，顾名思义，对于消费者来说，当想生产一辆车的时候，只需要向工厂申请就好，细节不需要知道，而且使用工厂模式的话，还比较好扩展，因为是运用的是接口编程思想。在Spring中大量使用工厂模式进行创建Bean。

**实例：**
创建一个可以绘制不同形状的绘图工具，可以绘制圆形，正方形，三角形，每个图形都会有一个draw()方法用于绘图，不看代码先考虑一下如何通过该模式设计完成此功能

```java
public interface Shape {
    void draw();
}
```

圆形

```java
public class CircleShape implements Shape {

    public CircleShape() {
        System.out.println(  "CircleShape: created");
    }

    @Override
    public void draw() {
        System.out.println(  "draw: CircleShape");
    }

}
```

正方形

```java
public class RectShape implements Shape {
    public RectShape() {
       System.out.println(  "RectShape: created");
    }

    @Override
    public void draw() {
       System.out.println(  "draw: RectShape");
    }

}
```

工厂方法

```java
public class ShapeFactory {
    public static final String TAG = "ShapeFactory";
    
    public static Shape getShape(String type) {
        Shape shape = null;
        if (type.equalsIgnoreCase("circle")) {
            shape = new CircleShape();
        } else if (type.equalsIgnoreCase("rect")) {
            shape = new RectShape();
        }
        return shape;
    }
}
```

客户端使用

```java
Shape shape= ShapeFactory.getShape("circle");
shape.draw();

Shape shape= ShapeFactory.getShape("rect");
shape.draw();
```

这样封装的话就比较优雅，而Spring中的工厂方法则是封装的更加的复杂，使用工厂方法的还有一个好处就是，如果要扩展三角形什么的，可以直接的修改工厂方法，而客户端的源码就不需要在更改，编程解耦的思想就是在更改需求的时候，尽量更改最少的代码，维护代码的健壮性，可以简单的理解为尽量减少直接的使用new关键字，不然在进行维护的时候更改使用类时，需要更改太多的代码，即耦合性很大。

# 建造者模式

其实也就是builder模式，有点不太理解？

建造者（Builder）模式的定义：指将一个复杂对象的构造与它的表示分离，使同样的构建过程可以创建不同的表示，这样的[设计模式](http://c.biancheng.net/design_pattern/)被称为建造者模式。它是将一个复杂的对象分解为多个简单的对象，然后一步一步构建而成。它将变与不变相分离，即产品的组成部分是不变的，但每一部分是可以灵活选择的。 就是有点像兼容的模式，有些属性可以不传递然后也能生成对应的独享，有点像极简的风格，也可以是复杂的风格，但是都能用。

其中netty中使用了很多的建造者模式。通过使用一个内存类Builder进行构建对象。

```java
public class Student {

    private String name;

    private int age;

    private int num;

    private String email;

    // 提供一个静态builder方法
    public static Student.Builder builder() {
        return new Student.Builder();
    }
    // 外部调用builder类的属性接口进行设值。
    public static class Builder{
        private String name;

        private int age;

        private int num;

        private String email;
		
        // 会先返回一个对象，然后在使用builder 进行构造函数。 其实就是一个public方法，并且返回Builder对象而已。
        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder num(int num) {
            this.num = num;
            return this;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Student build() {
            // 将builder对象传入到学生构造函数
            return new Student(this);
        }
    }
    // 私有化构造器
    private Student(Builder builder) {
        name = builder.name;
        age = builder.age;
        num = builder.num;
        email = builder.email;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", num=" + num +
                ", email='" + email + '\'' +
                '}';
    }
}
```

使用方法

```java
public static void student(){
    Student student = Student.builder()
        .name("平头哥")
        .num(1)
        .age(18)
        .email("平头哥@163.com")
        .build();
    System.out.println(student);
}
```

# 观察者模式

观察着模式有点像消息订阅，比如说服务器了消息，那么我客户端就能收到通知，相当于客户端一直在观察服务器的变化，这种也有点像对象之间的通信。而且一个服务器可以定义多个观察者，简单的观察者模式就是使用一个List进行存储观察者。

观察者（Observer）模式的定义：指多个对象间存在一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。这种模式有时又称作发布-订阅模式、模型-视图模式，它是对象行为型模式。

观察者模式是一种对象行为型模式，其主要优点如下。

1. 降低了目标与观察者之间的耦合关系，两者之间是抽象耦合关系。
2. 目标与观察者之间建立了一套触发机制。

它的主要缺点如下。

1. 目标与观察者之间的依赖关系并没有完全解除，而且有可能出现循环引用。
2. 当观察者对象很多时，通知的发布会花费很多时间，影响程序的效率。

## 简单demo

定义接口

```java
public interface AbstractSubject {
    // 添加观察者
    public void addObserver(AbstractObserver observer);
    // 删除观察者
    public void removeObserver(AbstractObserver observer);
    // 发布消息
    public void notification();
}
```

具体的通知执行者

```java
public class ConcreteSubject implements AbstractSubject {
    // 使用一个数组存储所有的观察者。 通过接口的动态性实现。 即不一定非要使用接口回调也能实现对应的功能。
    List<AbstractObserver> list = new ArrayList<AbstractObserver>();
    
    @Override
    public void addObserver(AbstractObserver observer) {
        list.add(observer);
    }
    
    @Override
    public void removeObserver(AbstractObserver observer) {
        list.remove(observer);
    }
    
    // 状态改变了，所有观察者更新自己的界面
    @Override
    public void notification() {
        // 使用一个for循环遍历通知所有的观察者。
        for (AbstractObserver abstractObserver : list) {
            // 调用所有观察者的update方法。
            abstractObserver.update();
        }
    }
}
```

如果要实现观察者就需要实现实现这个接口。

```java
public interface AbstractObserver {
    public void update();
}
```

测试类

```java
class Client {
    public static void main(String[] args) {
        // 生成一个主题角色
        AbstractSubject subject = new ConcreteSubject();
        // 为主题角色增加观察者对象，这里采用匿名内部类的方式，与AWT编程里的安装监听器类似
        subject.addObserver(new AbstractObserver() {
            @Override
            public void update() {
                System.out.println("A同学您的APP需要更新");
            }
        });
        
        subject.addObserver(new AbstractObserver() {
            @Override
            public void update() {
                System.out.println("B同学您的APP需要更新");
            }
        });
        
        subject.addObserver(new AbstractObserver() {
            @Override
            public void update() {
                System.out.println("C同学您的APP需要更新");
            }
        });
        subject.notification();
    }
}
```

- 参考链接 [Java设计模式-观察者模式](https://juejin.im/post/5cd262fcf265da039e2008e1)

在Spring框架中也扩展了很多的事件监听器，使用的就是观察者模式进行构建。参考[spring_event_exercise](https://github.com/LostStarTvT/spring_event_exercise)  

# 适配器模式

适配器模式其实也就是一种兼容的模式，类似于电脑上的转接头。以下其实也就是有点像代理模式，虽然船长不会用渔船，但是可以通过适配器进行是去调用具体方法去转换，达到了适配的模式，其实也就是java的多态性质。

**代码实例讲解**

有一个船长，只会使用划艇，不会使用渔船。首先，我们有接口`RowingBoat`和`FishingBoat`

```java

public interface RowingBoat {
  void row();
}

public class FishingBoat {
  public void sail() {
    LOGGER.info("The fishing boat is sailing");
  }
}
```

并且船长本身是会划艇的，所以船长已经实现了这个接口。

```java
public class Captain implements RowingBoat {

  private RowingBoat rowingBoat;

  public Captain(RowingBoat rowingBoat) {
    this.rowingBoat = rowingBoat;
  }

  @Override
  public void row() {
    rowingBoat.row();
  }
}
```

现在来了一个海盗，并且只有渔船可以使用，我们的船长想要逃避海盗，就只能使用渔船。所以我们需要一个适配器来帮助船长来用他操作划艇的技术来操作渔船。

```java
// 相当于定义了代理类，使用代理去调用boat对象的sail方法
public class FishingBoatAdapter implements RowingBoat {
  
  private FishingBoat boat;

  public FishingBoatAdapter() {
    boat = new FishingBoat();
  }

  @Override
  public void row() {
    boat.sail();
  }
}
```

现在，船长既可以开着渔船逃避海盗了。

```java
Captain captain = new Captain(new FishingBoatAdapter());
captain.row();
```

[参考链接](https://juejin.im/post/5b7e4964f265da4381519d9b)





