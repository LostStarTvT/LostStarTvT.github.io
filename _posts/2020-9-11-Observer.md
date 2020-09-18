---
layout: post
title: Java：观察者模式
tags: java
---


> 总结并记录观察者模式的实现

##  目录
* 目录
{:toc}
之前感觉比较新奇的接口回调其实也是一种观察者模式的实现，但是这种只是种一对一的观察者模式，传统的观察者模式是一对多的实现，即一个subject，对应多个observer，当subject中一个属性变化的时候，便会将变化通知到所有的观察者。具体的实现：

创建subject类：

```java
import java.util.ArrayList;
import java.util.List;
 
public class Subject {
   
   // 所有观察者的集合。 需要存储所有观察者的地址，也相当于接口回调中的注册。
   private List<Observer> observers 
      = new ArrayList<Observer>();
   private int state;
 
   public int getState() {
      return state;
   }
   
   // 当有人调用更新主题属性的时候，便将更新通知到所有的观察者。
   public void setState(int state) {
      this.state = state;
      notifyAllObservers();
   }
 
   public void attach(Observer observer){
      observers.add(observer);      
   }
 	
   // 通知所有的观察者事件变化。
   public void notifyAllObservers(){
      for (Observer observer : observers) {
         observer.update();
      }
   }  
}
```

创建Observer类：

```java
public abstract class Observer {
   protected Subject subject; // 里面有主题的类
   public abstract void update();
}
```

创建具体的观察者：

```java
// 可以发现，在初始化构造方法中，需要把主题对象的地址传递进来，然后调用主题的attach()方法进行注册自己。
public class BinaryObserver extends Observer{
 
   public BinaryObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }
   
   // 然后注册自己想知道的通知，其实和接口回调是完全一致的。
   @Override
   public void update() {
      System.out.println( "Binary String: " 
      + Integer.toBinaryString( subject.getState() ) ); 
   }
}

public class OctalObserver extends Observer{
 
   public OctalObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }
 
   @Override
   public void update() {
     System.out.println( "Octal String: " 
     + Integer.toOctalString( subject.getState() ) ); 
   }
}


public class HexaObserver extends Observer{
 
   public HexaObserver(Subject subject){
      this.subject = subject;
      this.subject.attach(this);
   }
 
   @Override
   public void update() {
      System.out.println( "Hex String: " 
      + Integer.toHexString( subject.getState() ).toUpperCase() ); 
   }
}
```

测试程序：

```java
public class ObserverPatternDemo {
   public static void main(String[] args) {
      Subject subject = new Subject();
 
      new HexaObserver(subject);
      new OctalObserver(subject);
      new BinaryObserver(subject);
 
      System.out.println("First state change: 15");   
      subject.setState(15);
      System.out.println("Second state change: 10");  
      subject.setState(10);
   }
}
```

输出结果如下：

```shell
First state change: 15
Hex String: F
Octal String: 17
Binary String: 1111
Second state change: 10
Hex String: A
Octal String: 12
Binary String: 1010
```
类比于接口回调，正常情况下是A调用B的接口，然后进行输出，现阶段我们想让B调用A中的接口，那么就需要一点面向接口编程进行实现，即首先在B中定一个接口和接口的注册方法，然后在B中具体的业务中调用接口方法。在A中首先实现B中接口方法，然后获取B对象的地址，向B中注册该接口方法，那么A中在调用接口方法的时候，便会调用改接口，既然可以调用一个方法，那么我用一个数组存储下来所有注册接口的方法，那么就成为了观察者模式，这样也是Netty异步实现的基础。即当主题进行数据更新后，变化调用观察者的更新方法。

