---
layout: post
title: JVM：New Class
tags: java  
---


> 介绍JVM中新建一个对象的过程，是如何调用各个数据区然后开辟内存空间进行初始化。

#  目录
* 目录
{:toc}
# 初始化对象

本文主将前面所说的堆和方法区连接起来，即探讨二者是如何配合工作生成一个对象。以下面的代码为例。首先定义一个Customer类，然后使用new关键字生成一个对象。

```java
public class CustomerTest{
    public static void main(String[] args ){
        Customer cust = new Customer();
    }
}
```

```java
public class Customer{
    int id = 1001;
    String name;
    Account acct;
    
    {
        name = "匿名客户";
    }
    
    public Custome(){
        acct = new Account(); //省略
    }
}
```

当执行`Customer cust = new Customer();`是，jvm的对应的内存空间处理。

![new.png](https://pic.tyzhang.top/images/2020/06/05/new.png)

从图上可以看出，其实一个对象在堆空间主要从存储的还是属性的值，但是其中的对象头比较重要，它通过一个指针指向了自己对应的class对象，这样堆和方法区便联系了起来。也就说除了属性等其他信息都是在方法区存储：程序的运行逻辑。另外运行时元数据也值得注意，这里就是保存了垃圾回收的重要信息。从上面可以看出，代码的运行都是从线程入手，即主线程也是一个线程，然后通过局部变量表的指针访问对象。

# 创建对象的方式

1. new关键字
2. class的newInstance()
3. Constructor的newInstance(Xxxx)
4. 使用clone()
5. 使用反序列化
6. 使用第三方库Objenesis

# 创建对象的步骤

1. 判断对象的对应类是否加载、链接、初始化
2. 为对象分配内存
   1. 如果内存规整，指针碰撞
   2. 如果内存不规整：虚拟机需要维护一个列表，空闲列表分配
3. 处理并发安全问题
   1. 采用CAS配上失败重试保证更新的原子性
   2. 每个线程预先分配一块TLAB
4. 初始化分配到的内存空间：所有属性设置默认值，保证对象实例字段在不赋值的时候可以直接的使用
5. 设置对象的对象头
6. 执行init初始化方法。

虚拟机在遇到一条new指令，首先检查这个指令的参数是否能够在MetaSpace的常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载、解析和初始化。如果没有那么在双亲委派的机制下使用加载器以classloader+包名+类名为key进行查找找到对应的class文件，如果没有找到文件，则抛出Classnotfoundexception。如果找到，则进行类加载，并生成对应的class类对象。

其实堆中的数据，也就是相当于一个数组，其实对于一个类来说，只有属性是最有效的东西，属性加上属性值，其他的运行逻辑则是放在方法区，所有堆的话可以看成一个保存着属性的数组。

# 对像访问定位

## 通过句柄访问

![jubing.png](https://pic.tyzhang.top/images/2020/06/05/jubing.png)

## 直接访问模式

HostSpot采用这中模式进行访问堆中的对象。

![direct.jpg](https://pic.tyzhang.top/images/2020/06/05/direct.jpg)

对于句柄来说，坏处是 效率比较低，需要单独开辟空间保存句柄。因为要经过代理，但是好处的是因为在gc的时候对象会进行移动，所以地址就会改变， 对于句柄来说此时栈中的地址无需更改，只需要更改句柄中的对象引用， 而直接引用则需要更改，即各有利弊。其实句柄也就是代理的思想，屏蔽底层的改变。