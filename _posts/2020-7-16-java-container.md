---
layout: post
title: Java：容器
tags: java
---


> 记录学习java基础，java常用的容器总结。

##  目录
* 目录
{:toc}
# HashCode方法

## 为什么要有 hashCode

**我们先以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode：** 当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与该位置其他已经加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 `equals()`方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自我的 Java 启蒙书《Head first java》第二版）。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

以上可以看出：`hashCode()` 的作用就是**获取哈希码**，也称为散列码；它实际上是返回一个 int 整数。这个**哈希码的作用**是确定该对象在哈希表中的索引位置。**`hashCode()`在散列表中才有用，在其它情况下没用**。在散列表中 hashCode() 的作用是获取对象的散列码，进而确定该对象在散列表中的位置。

对于集合Set和HashMap来说，其在进行存储的时候都需要使用对象的hash值作为key来进行确定对象的位置，其值是由Java中的本地方法进行实现。

[Java基础](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.md#251-string-stringbuffer-%E5%92%8C-stringbuilder-%E7%9A%84%E5%8C%BA%E5%88%AB%E6%98%AF%E4%BB%80%E4%B9%88-string-%E4%B8%BA%E4%BB%80%E4%B9%88%E6%98%AF%E4%B8%8D%E5%8F%AF%E5%8F%98%E7%9A%84) 

以下是Object对象中的实现。

```java
    /**
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@code equals(Object)}
     *     method, then calling the {@code hashCode} method on each of
     *     the two objects must produce the same integer result.
          如果两个对象使用equals方法判断相同，那么HashCode必须相同。
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link java.lang.Object#equals(java.lang.Object)}
     *     method, then calling the {@code hashCode} method on each of the
     *     two objects must produce distinct integer results.  However, the
     *     programmer should be aware that producing distinct integer results
     *     for unequal objects may improve the performance of hash tables.
     * </ul>
     */
    public native int hashCode();
	
	// 默认判断两个对象是否是同一个对象。 内部调用的就是 ==  即地址是否相同。
    public boolean equals(Object obj) {
        return (this == obj);
    }
	
	// 默认打印对象输出的值。
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

> Whenever it is invoked on the same object more than once during an execution of a Java application, the {@code hashCode} method **must** **consistently return the same integer**, provided no information  used in {@code equals} comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application. If two objects are equal according to the {@code equals(Object)} method, then calling the {@code hashCode} method on each of  the two objects must produce the same integer result.

从上面的英文解释来说，对于一个对象来说，从出生到死亡其哈希值都不能进行改变，否则在进行HashMap或者Set操作的时候就会出错。而且其是使用本地方法进行实现的，那么如何进行保持一致呢？Java的做法就是将哈希值保存在每个对象头中，跟随对象一生。另外从toString()方法可以看出，我们在输出对象时候，其实也是在输出对象名+上对象的hash值。如下所示：

```java
public static void main(String[] args) {

    Student student = new Student();
    Student student1 = new Student();

    System.out.println("student: " + student.hashCode());
    System.out.println("student: " + student);

    System.out.println("student1: " +student1.hashCode());
    System.out.println("student1: " + student1);
}
```

```shell
student: 460141958
student: HashDemo.Hash$Student@1b6d3586
student1: 1163157884
student1: HashDemo.Hash$Student@4554617c
```

其中460141958的十六进制表示为1b6d3586。另外有的博客说`System.out.println("student1: " + student1);`这样输出的就是对象的地址，感觉应该不是，参考[Object::hashCode的返回值是不是对象的内存地址？](https://developer.aliyun.com/article/575705) 这篇文章主要说明的是，如果使用地址基准作为hash值的计算依据，那么在JVM进行GC的时候，进行复制移动那么地址肯定会改变，相应的Hash值是否也要改变？如果Hash值也要改变，那么使用Hash的集合便会出现内存泄露。虽然计算过后的hash值会存储在对象头一直保存下来，但是如果会变的话，那么也是不唯一的。 以下是查看h1和h2两个对象在JVM中的地址。

> hsdb再次连上，查看数据，发现预期一样写入了对应的位： 
>
> 0x000000 **6f2b958e** 01  // h1的markwork哈希值
>
> 0x000000 **1eb44e46** 01 // h2的markwork哈希值
>
> ```
> # Hash h1
> hsdb> mem 0x000000010b33d690 3
> 0x000000010b33d690: 0x0000006f2b958e01  
> 0x000000010b33d698: 0x000000010c000578 
> 0x000000010b33d6a0: 0x0000000000001234 
> # Hash h2
> hsdb> mem 0x000000010b33d6a8 3
> 0x000000010b33d6a8: 0x0000001eb44e4601 
> 0x000000010b33d6b0: 0x000000010c000578 
> 0x000000010b33d6b8: 0x0000000000005678 
> ```
>
> 再让程序执行到第三个断点，程序输出 `after gc, h2.hashCode=1eb44e46` ，hash code没变。理论上此时h1被回收，h2被copy到old gen，地址变化了。于是使用OQL再次查询h2的地址为0x000000010b5ea220，查看内存如下
>
> ```
> # Hash h2
> hsdb> mem 0x000000010b5ea220 3
> 0x000000010b5ea220: 0x0000001eb44e4601 
> 0x000000010b5ea228: 0x000000010c000578 
> 0x000000010b5ea230: 0x0000000000005678
> ```
>
> 对象数据不变，所以还是能从MarkWord 0x000000 **1eb44e46** 01 中取出生成过的hash code。

从上面可以看出，在经过gc以后，h2对象的哈希值没有没有变，但是保存对象头的地址已经变了，0x000000010b5ea220 不等于0x000000010b33d6a8。所以说`System.out.println("student1: " + student1);`输出的只是对象头中的哈希值，JVM将对象的地址给屏蔽掉，对于程序员是透明的。

对象头的细节：

![head.png](https://pic.tyzhang.top/images/2020/07/07/head.png)

>细心的读者看到这里会发现一个问题：当对象进入偏向状态的时候，Mark Work大部分的控件都用于存储持有锁的线程ID了，这部分空间占用了原有存储对象哈希码的位置，那原来对象的哈希码怎么办？
>
>在Java语言里面一个对象如果计算过哈希码，就应该一直保持不变（强烈推荐但是不强制，因为用户可重载hashCode()方法按自己的意愿放回哈希码），否则很多依赖对象哈希码的API可能出在出错的风险。而作为绝大多数哈希码来源的Objec::hashCode()方法，返回的是对象的一致性哈希码(Identity Hash Code)，这个值是能够强制保证不变的，他通过在对象头中存储计算结果来保证第一次计算之后，再次调用该方法取到的哈希值永远不会发生改变。因此，当一个对象已经计算过一致性哈希码后，它就再也无法进入偏向锁状态了；而当一个对象当前正处于偏向锁状态，而又收到需要计算其一致性哈希码请求时，它的偏向锁状态会被立刻撤销，并且锁会膨胀为重量级锁。在重量级锁的实现中，对象头指向了重量级锁的位置，代表重量级锁的ObjectMonitor类里有字段可以记录费加锁状态(标志位为“01”)下的Mark Word，其中自然可以存储原来的哈希值。

即Mark Work会根据对象锁的转态不断的进行改变，详情见Synchronize锁底层实现。

[Java中的hashCode 真的是地址吗？](https://blog.csdn.net/topc2000/article/details/79454064) 这篇博文中说java底层的hashCode函数有五种的实现方式，实现的方式有自增序列，随机数，内存地址，应该是可以自主的选择使用哪种方式。但是肯定不是对象的具体虚拟地址，但是可以是通过虚拟地址进行转变过来，但是只调用一次保证不变。

# Equal方法

从原生API的判断可以看出，判断两个对象是否相等， 就是简单的调用 == ，也就是比较地址是否是同一个对象。

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

## 证明“==”是比较地址

当定义两个对象，然后对比是否相同的时候，如下：此时没有重写toString()方法，

```java
public static void main(String[] args) {

    Student student = new Student();
    Student student1 = new Student();
   
    System.out.println(student.equals(student1)); // false
}
```

debug的情况如下：可以看出地址不同而且HashCode不同。但比较的也是前面的@477和@487。

![equals89037894e8f2a185.png](https://pic.tyzhang.top/images/2020/07/16/equals89037894e8f2a185.png)

当重写了toString()方法以后，在进行调用equals()方法，如下：

```java
    public static void main(String[] args) {

        Student student = new Student();
        Student student1 = new Student();
        student.setName("aaa");
		student.setName("aaa");
        System.out.println(student.equals(student1)); // false
    }
```

虽然后面的值是一样，但是其地址@477和@488还是不一样，仍然返回false。

![equalString.png](https://pic.tyzhang.top/images/2020/07/16/equalString.png)

以上分析便说明了，默认情况下调用equal判断是不是同一个对象，当且仅当是同一个对象才会返回True，而同一个对象那么HashCode肯定相同，因为保存在对象头中。而我们需要的是调用equal的时候判断对象内容是否相同，此时是两个不同的对象进行比较，不同对象头中的默认调用hashCode必定不相同，所以必须要重写HashCode方法，使得两个内容相同的对象的HashCode一致，一种方法就是对对象的内容进行HASH散列，参考String的实现。

在有些情况下，程序设计者在设计一个类的时候为需要重写equals方法，比如String类，但是千万要注意，**在重写equals方法的同时，必须重写hashCode方法。** 

> 　“设计hashCode()时最重要的因素就是：无论何时，对同一个对象调用hashCode()都应该产生同样的值。如果在讲一个对象用put()添加进HashMap时产生一个hashCdoe值，而用get()取出时却产生了另一个hashCode值，那么就无法获取该对象了。所以如果你的hashCode方法依赖于对象中易变的数据，用户就要当心了，因为此数据发生变化时，hashCode()方法就会生成一个不同的散列码”。

以上也可以说，如果两个对象的HashCode相同，对象可能不相同，但是如果两个对象相同，那么hashCode一定相等。这个原则是在HashMap和Set中的运用准则，即程序会首先判断对象的HashCode是否相同，如果相同则再次调用equals方法判断是否相同，如果都相同则是相同对象，否则为不同对象。

[浅谈Java中的hashcode方法](https://www.cnblogs.com/dolphin0520/p/3681042.html)  参考以上博文。

## hashCode()与 equals()的相关规定

以下的规定为**重写equals时需要满足的条件，**默认也是满足的，因为只有当且仅当`System.out.println(student.equals(student));` 即自己和自己比较时才会返回True。默认情况下内容相同的对象返回False，所以要重写。

1. 如果两个对象相等，则 hashcode 一定也是相同的
2. 两个对象相等,对两个对象分别调用 equals 方法都返回 true
3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

[Java hashCode() 和 equals()的若干问题解答](https://www.cnblogs.com/skywang12345/p/3324958.html)  

那么为什么**在重写equals方法的同时，必须重写hashCode方法？** 主要是因为java提供的API是满足上面所说的条件，但当重写了equal后，如果不重写HashCode方法，可能会出现两个对象相同但是HashCode不同，违反了java设计原则。此时在使用HashMap等API便会出错。

# HashMap

