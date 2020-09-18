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

# 1. 单例模式

一种最常用的模式，在spring中大量使用，默认的Bean就是一个单例的，所谓单例也就是在获取对象的时候返回的都是同一个对象，不会创建第二个对象，类比于现实生活中的一个班级班长只有一个。

模式定义： 保证一个类只有一个实例，并且提供一个全局访问点。

应用场景： 重量级对象不需要多个实例，比如线程池对象，数据库连接池，还有就是类加载器。

![image-20200915163506360](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915163506360.png)

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

上面这种每次获取都加锁性能明显不可取，因为只是为了在防止多创建，如果已经创建那么在获取的时候就不需要加锁，直接获取就好,双重检查模式。

```java
class LazySingleton{

    private static volatile LazySingleton singleton; // 是为了创建好对象以后，别的线程能够立刻的获取到。并且能够防止指令重拍。

    private LazySingleton(){}
    public static LazySingleton getSingleton() {
        
        if (singleton == null){
            synchronized (LazySingleton.class){ // 因为可能有两个线程同时竞争加锁， 当一个线程创建好对象以后，另外一个线程会直接的获取到锁进行执行下面的代码。 
                if (null == singleton){ // 如果不进行检测会出现多个对象的情况。
                    singleton = new LazySingleton();
                    // 因为这个new指令被分为：
                    // 1. 分配空间
                    // 3 引用赋值
                    // 2 初始化
                    // 加了volatile 之后肯定是 1 2 3 正常。
                    // 即在因为分配空间是必须先进行，但是 2  3 可以被指令重排位32 那么就可能
                    // 出现 先对singleton进行引用赋值，但是还没有初始化，那么其他线程在访问的时候就可能出现空指针异常
                    //因为外面时直接检测singleton 不为空就直接取值了，可能会出现异常。 我们必须保持先初始化
                    // 然后在进行引用赋值。
                }
            }
        }
        return singleton;
    }
}
```

注意：如果编写的是多线程程序，则不要删除上例代码中的关键字 volatile 和 synchronized，否则将存在线程非安全的问题。如果不删除这两个关键字就能保证线程安全，但是每次访问时都要同步，会影响性能，且消耗更多的资源，这是懒汉式单例的缺点。

为什么需要volatile 关键字？ 因为避免指令重排序，还有就是内存可见，因为其他线程会先检测singleton是否为空，所以需要使用volatile保持内存可见。

可能出现的问题：

1. 线程安全问题。
2. double check加锁优化
3. 编译器，CPU有可能对指令进行重排序，导致使用到尚未实例化的实例，可以通过添加volatile关键字进行修饰，对于volatile修饰的字段，可以防止指令重排。

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

这种主要是利用了类加载机制，因为类会在初始化的时候就对类的静态变量进行赋值，此时就会实例化一个类。



## 三、静态内部类的方式

本质上是利用类的加载机制来保证线程安全，因为对于类加载来说肯定是线程安全的。而且也是只会在使用的时候才会被加载，是一种懒加载的形式。

```java
class InnerClassSingleton{
    // 也是一种懒加载的方式。
    private static class InnerClassHolder{
        private static  InnerClassSingleton instance = new InnerClassSingleton();
    }

    private InnerClassSingleton(){}

    public static InnerClassSingleton getInstance(){
        return InnerClassHolder.instance;
    }
}
```

对于通过反射进行新建的单例对象与直接调用静态方法获取到的对象不是同一个对象。称之为反射攻击。

抽象类的中所有的抽象方法子类都必须重写，包括构造方法。

枚举类不能被方式构造。



## 其他一些事项

如果将单例对象记性序列化与反序列化以后，此时生成的对象与使用单例获取到的对象不是同一个对象，这样就出现了单例多对象的错误。

```java
class InnerClassSingleton implements Serializable {
    static final long serialVersionUID = 42L; // 序列化一定要加上版本号，不然更改属性以后，版本号就会改变。
    // 也是一种懒加 载的方式。
    private static class InnerClassHolder{
        private static  InnerClassSingleton instance = new InnerClassSingleton();
    }

    private InnerClassSingleton(){
        if (InnerClassHolder.instance != null) throw new RuntimeException("单例不允许多个实例");
    }

    public static InnerClassSingleton getInstance(){
        return InnerClassHolder.instance;
    }
	
    // 加上这个方法能够保证反序列化 也是同一个对象，即会在反序列生成对象的时候先判断jvm有没有已经存在，存在就将当返回当前单例对象。
    Object readResolve() throws ObjectStreamException{
        return  InnerClassHolder.instance;
    }
}
```

JDK中Runtime实现用饿汉式单例模式。

# 2. 工厂模式

简单工厂模式：

抽象工厂模式： 提供一个创建一系列相关或互相依赖对象的接口，而无需指定它们之间具体的类。 面向接口编程。真的很好用，很符合开闭原则，即面向扩展开，面向更改关。

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

# 3. 建造者模式

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

# 4. 观察者模式

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

# 5. 适配器模式

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

# 6. 门面模式

门面模式其实也就是为一些复杂的功能提供了一简单统一的API，而屏蔽底层的复杂逻辑，其实每个web就是一个门面模式的实例，因为封装了统一的API，然后前端只需要进行调用就行。

![image-20200915102336927](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915102336927.png)

从上面可以看出来，其实就是为一个复杂的子系统构建了一个门面而已。

《设计模式》给出了门面模式的使用环境：
1) 当你要为一个复杂子系统提供一个简单接口时。在上面已经描述了原因。
2) 客户程序与抽象类的实现部分之间存在着很大的依赖性。引入 facade 将这个子系统与客户以及其他的子系统分离，可以提高子系统的独立性和可移植性（上面也提到了）。
3) 当你需要构建一个层次结构的子系统时，使用 facade 模式定义子系统中每层的入口点。如果子系统之间是相互依赖的，你可以让它们仅通过 facade 进行通讯，从而简化了它们之间的依赖关系。
以下是它的优点：
1) 它对客户屏蔽子系统组件，因而减少了客户处理的对象的数目并使得子系统使用起来更加方便。
2) 它实现了子系统与客户之间的松耦合关系，而子系统内部的功能组件往往是紧耦合的。松耦合关系使得子系统的组件变化不会影响到它的客户。 Facade 模式有助于建立层次结构系统，也有助于对对象之间的依赖关系分层。 Facade 模式可以消除复杂的循环依赖关系。这一点在客户程序与子系统是分别实现的时候尤为重要。在大型软件系统中降低编译依赖性至关重要。在子系统类改变时，希望尽量减少重编译工作以节省时间。用Facade 可以降低编译依赖性，限制重要系统中较小的变化所需的重编译工作。 Facade模式同样也有利于简化系统在不同平台之间的移植过程，因为编译一个子系统一般不需要编译所有其他的子系统。
3) 如果应用需要，它并不限制它们使用子系统类。因此你可以让客户程序在系统易用性和通用性之间加以选择。  

# 7. 享元模式

其中String的设计就是享元模式，即对于相同的字符串引用的是同一个字符串。如果不同则是新建一个字符串，不会更改原来的字符串。

模式定义：运用**共享技术**有效的支持大量细粒度的对象。（一般都是不可变的对象，所以是线程安全）

优点：如果系统游大量的类似对象，可以节省大量的内存以及CPU资源。

那么享元模式与单例模式之间的区别呢？ 一个最主要的区别就是享元模式下的对象不可变，而单例模式只是单纯的表示JVM中只有一个对象，表明单一元素。比如说游戏中一个地图只有

# 8. 装饰者模式

模式定义： 在不改变原有对象的基础上，将功能附加到对象上。

![image-20200915104612302](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915104612302.png)

相当于是红线上面的是原始的功能，而下面是需要新加上的功能，使用Decorator在不更改上面代码的前提下，通过持有component对象的前提下，新增了下面的operation方法，相当于是扩展功能，符合程序设计的开闭原则，进行了解耦，类似于，为一个相机上了美颜滤镜的效果，美颜滤镜是后面加上，只需要获取相机数据流就可以直接的进行操作。

```java
/**
 * @Description TODO 装饰者模式测试
 */
public class DecoratorDemo {
    public static void main(String[] args) {
        Component component = new concreteComponent();
        component.operation(); //进行拍照
        System.out.println("-----");
        // 使用装饰者进行调用新的方法。
        Component component1 = new ConcreteDecorator1(component);
        component1.operation();

        System.out.println("-----");
        // 添加滤镜。。 疯狂套娃 因为是同一个方法所以可以直接的加上
        Component component2 = new ConcreteDecorator2(component1);
        component2.operation();
        // 主要就是进行加上了
    }
}

// 功能接口
interface Component{
    void operation();
}

class concreteComponent implements Component{
    @Override
    public void operation() {
        System.out.println("拍照");
    }
}

// 定义装饰者
abstract  class  Decorator implements Component{
    Component component; // 获取原始的对象。

    public Decorator(Component component) {
        this.component = component;
    }
}

class ConcreteDecorator1 extends Decorator{
    // 实现父类的构造方法
    public ConcreteDecorator1(Component component) {
        super(component);
    }

    @Override
    public void operation() {
        component.operation(); // 调用父类的方法。
        System.out.println("添加美颜"); // 自己新增加的方法。
    }
}

class ConcreteDecorator2 extends Decorator{
    // 实现父类的构造方法
    public ConcreteDecorator2 (Component component) {
        super(component);
    }

    @Override
    public void operation() {
        component.operation(); // 调用父类的方法。
        System.out.println("添加滤镜"); // 自己新增加的方法。
    }
}


```

应用场景：

扩展一个类的功能或给一个类添加附加职责。

优点：

1. 不改变原有对象的情况下给一个对象添加功能，
2. 使用不同的组合可以实现不同的效果
3. 符合开闭原则。

主要也就是通过实现面向接口的编程，可以疯狂的套娃，然后调用上层的方法。

经典案例就是在 javax.servlet.http.httpServletRequestWrapper 这个类中实现了装饰者模式。

# 9. 策略模式

Strategy

模式定义： 定义了算法簇，分别封装起来，让他们之间可以互相替换，此模式的变化独立于算法的使用者。

这种很适合替换掉if switch语句，也能够很好的扩展业务逻辑，如果单纯的在if 和 switch上面扩展业务的话，很麻烦。我现在写的项目好像就很难扩展。。

JDK中比较接口就是策略模式。Comparator 就是策略的接口。即可以实现不同的排序策略。

![image-20200915141210450](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915141210450.png)

核心也就是面向接口编程。

```java
/**
 * @Classname StrategyTest
 * @Description TODO 策略模式
 * @Date 2020/9/15 13:53
 * @Created by Seven
 */
public class StrategyTest {
    public static void main(String[] args) {
        Zombie normal = new NormalZombie();
        normal.display();
        normal.move();
        normal.attack();

        normal.setAttackAble(new HitAttack()); // 更换策略。
        normal.attack(); //直接就变成打的策略了 那么以后直接进行更换就非常简单的进行更换。
    }
}

// 可以选择的策略。
interface MoveAble{
    void  move();
}

interface  AttackAble{
    void  attack();
}

// 抽象出默认的僵尸，定义好公共的属性。
abstract class Zombie{
    // 展示僵尸
    abstract  public void display();

    MoveAble moveAble;
    AttackAble attackAble;
    // 带有的属性，但是属性的值是可以变的
    abstract  void  move();
    abstract  void attack();

    // 默认构造函数
    public Zombie(MoveAble moveAble, AttackAble attackAble) {
        this.moveAble = moveAble;
        this.attackAble = attackAble;
    }

    public MoveAble getMoveAble() {
        return moveAble;
    }

    public void setMoveAble(MoveAble moveAble) {
        this.moveAble = moveAble;
    }

    public AttackAble getAttackAble() {
        return attackAble;
    }

    public void setAttackAble(AttackAble attackAble) {
        this.attackAble = attackAble;
    }
}

// 不同的策略，那么我就可以直接进行定义不同的move策略，就可以直接的更换，
// 其实还利用了java的动态绑定。
class StepByStep implements MoveAble{

    @Override
    public void move() {
        System.out.println("一步一步的移动");
    }
}

class BiteAttack implements AttackAble{

    @Override
    public void attack() {
        System.out.println("咬");
    }
}

class HitAttack implements AttackAble{

    @Override
    public void attack() {
        System.out.println("打");
    }
}

//具体的实现类。
class NormalZombie extends Zombie{

    // 默认构造函数。
    public NormalZombie() {
        super(new StepByStep(), new BiteAttack());
    }

    public NormalZombie(MoveAble moveAble, AttackAble attackAble) {
        super(moveAble, attackAble);
    }

    @Override
    public void display() {
        System.out.println("普通僵尸");
    }

    // 调用父类的属性中的接口。
    @Override
    void move() {
        moveAble.move();
    }

    @Override
    void attack() {
        attackAble.attack();
    }
}
```

# 10. 模板方法模式

模式定义：定一个操作的算法骨架，而将一些步骤延迟到子类中，Template Method使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

![image-20200915141458155](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915141458155.png)



```java
public class TemplateMethodTest {
    public static void main(String[] args) {
        // 给个模板，然后你再去填充需要的东西。
        AbstractClass abstractClass = new subClass();
        abstractClass.operation();
    }
}

// 相当于定义一个模板然后 将公用的方法定义好，然后用户填充自己的逻辑就行

abstract class AbstractClass{

    public void operation(){
        // open
        System.out.println("pre");

        System.out.println("step1");

        System.out.println("step2");

        templateMethod(); // 用户只需要实现这个方法就好，然后
        // 公共的方法自己已经实现了， 相当于是框架的设计方案、
        //..
    }
    abstract protected void templateMethod();
}

// 用户只需要处理这个逻辑就行，spring中有这种设计方法。
class subClass extends AbstractClass{

    @Override
    protected void templateMethod() {
        System.out.println("用户自己的逻辑。 ");
    }
}
```

这种在Servlet中有使用，即使用抽象类封装了具体的连接和输出传输，但是用户必须要实现具体数据的处理逻辑、

# 11. 责任链模式

为请求创建了一个接受者对象的链。

request - > 请求频率  ---> 登录认证 ----> 访问权限 ---> 敏感词过滤 ---> --- 这就是一种连续认证的模式，也就是netty的Handler的设计思路。

责任链的模式主要就是需要咋设置责任链的时候，netty是用PipeLine进行依次的遍历handler，还有一种方法就是使用数组记性依次比那里Hander。

```java

public class ChainOfResponsibilityTest {

    public static void main(String[] args) {
        Request request = new Request(true,false);
        // 进行套娃传输， 然后就可以一直进行扩展责任链。
        RequestFrequentHandler requestFrequentHandler =  new RequestFrequentHandler( new LoginingHandler( null));
        if (requestFrequentHandler.process(request)){
            System.out.println("处理是OK的");
        }else {
            System.out.println("登录失败");
        }
    }
}

abstract class Handler{
    Handler next;

    public Handler(Handler next) {
        this.next = next;
    }

    public Handler getNext() {
        return next;
    }

    public void setNext(Handler next) {
        this.next = next;
    }

    // 只有当处理放回为true的时候，才能接续往后执行。 否则原路放回。
    abstract boolean process(Request request);
}

class RequestFrequentHandler extends Handler{

    // 需要传递进来下一个handler的地址， 当为null的时候，到达最后结点。
    public RequestFrequentHandler(Handler next) {
        super(next);
    }

    @Override
    boolean process(Request request) {
        if (request.isFrequentOK()){
            System.out.println("访问控制频率。");
            Handler next = getNext();// 调用下一个Handler
            if (null == next) return true;
            // 调用并返回下一个执行链的运行结果。
            return next.process(request);
        }
        return false;
    }
}

class LoginingHandler extends Handler{

    public LoginingHandler(Handler next) {
        super(next);
    }

    @Override
    boolean process(Request request) {

        if (request.AuthenticOk){
            System.out.println("登录成功。");
            // 已经登录
            Handler next;
            if (( next = getNext()) == null) return true;
            return next.process(request); // 返回后面执行的结果.
        }
        return false;
    }
}

class Request{

    boolean FrequentOK;
    boolean AuthenticOk;

    public Request(boolean frequentOK, boolean authenticOk) {
        FrequentOK = frequentOK;
        AuthenticOk = authenticOk;
    }

    public boolean isFrequentOK() {
        return FrequentOK;
    }

    public void setFrequentOK(boolean frequentOK) {
        FrequentOK = frequentOK;
    }

    public boolean isAuthenticOk() {
        return AuthenticOk;
    }

    public void setAuthenticOk(boolean authenticOk) {
        AuthenticOk = authenticOk;
    }
}
```



![image-20200915151119512](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915151119512.png)

![image-20200915151828982](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915151828982.png)

![image-20200915155255832](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915155255832.png)

![image-20200915162119741](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915162119741.png)

![image-20200915162447001](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200915162447001.png)

最后一个相当于框架，框架中的代码不会依赖于用户写的业务代码，