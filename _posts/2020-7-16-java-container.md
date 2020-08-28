---
layout: post
title: Java：容器
tags: java
---


> 记录学习java基础，java常用的容器总结。

##  目录
* 目录
{:toc}
# 集合框图

![collection.gif](https://pic.tyzhang.top/images/2020/07/27/collection.gif)

从图中可以看出主要是有List(数组)、Set(结合)、Queue(队列)、Map(字典)、Stack(栈)五种常用的集合。其中Vector是线程安全的数组。

对于很多java中的实现类来说，如果实现了对应Object中的方法，那么就能够直接的调用API，比如说Object提供了一个简单的equal方法，该方法只是单纯的封装了 == 符号，即只有当是同一个对象进行比较的时候才会返回true，即a.equals(a)，但是String中实现了该equals方法，把它改成了是比较内容，并不是比较对象的地址，那么也就说说明如果提供了什么API就需要看它有没有实现该方法，类比于语法糖自动拆箱装箱，Integer类是封装了int关键字，那么他也就重写了eqauls方法，也就代表了比较的是其中的内容而不是地址。也就说说Object是定义了所有类的最通用的属性。

# 一、HashCode方法

## 为什么要有 hashCode

**我们先以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode：** 当你把对象加入HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与该位置其他已经加入的对象的 hashcode 值作比较，如果没有相同的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 `equals()`方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自我的 Java 启蒙书《Head first java》第二版）。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

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

从上面的英文解释来说，对于一个对象来说，从出生到死亡其哈希值都不能进行改变，否则在进行HashMap或者Set操作的时候就会出错。并且是使用本地(native)方法进行实现的，那么如何进行保持一致呢？Java的做法就是将哈希值保存在每个对象头中，跟随对象一生。另外从toString()方法可以看出，我们在输出对象时候，其实也是在输出对象名+上对象的hash值。如下所示：

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

**其中460141958的十六进制表示为1b6d3586**。另外有的博客说`System.out.println("student1: " + student1);`这样输出的就是对象的地址，感觉应该不是，参考[Object::hashCode的返回值是不是对象的内存地址？](https://developer.aliyun.com/article/575705) 这篇文章主要说明的是，如果使用地址基准作为hash值的计算依据，那么在JVM进行GC的时候，进行复制移动那么地址肯定会改变，相应的Hash值是否也要改变？如果Hash值也要改变，那么使用Hash的集合便会出现内存泄露。虽然计算过后的hash值会存储在对象头一直保存下来，但是如果会变的话，那么也是不唯一的。 以下是查看h1和h2两个对象在JVM中的地址。

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

[Java中的hashCode 真的是地址吗？](https://blog.csdn.net/topc2000/article/details/79454064) 这篇博文中说java底层的hashCode函数有五种的实现方式，实现的方式有自增序列，随机数，内存地址，应该是可以自主的选择使用哪种方式。但肯定不是对象的具体虚拟地址，不过可以是通过虚拟地址进行转变过来，但只在第一此计算时候进行调用，计算过以后便只会从对象头中取出来，从而保证不变。

# 二、Equal方法

从原生API的判断可以看出，判断两个对象是否相等， 就是简单的调用 == ，也就是比较地址是否是同一个对象。

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

## 证明“==”是比较地址

默认toString写法：

```java
// 默认打印对象输出的值。
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

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

以下的规定为**重写equals时需要满足的条件，**默认也是满足的，因为只有当且仅当`System.out.println(student.equals(student));` 即自己和自己比较时才会返回True，对应到HashMap中就是“我找我自己，我就与我自己对比”。而默认情况下内容相同的对象返回False，所以要重写。

1. 如果两个对象相等，则 hashcode 一定也是相同的
2. 两个对象相等,对两个对象分别调用 equals 方法都返回 true
3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

[Java hashCode() 和 equals()的若干问题解答](https://www.cnblogs.com/skywang12345/p/3324958.html)  

那么为什么**在重写equals方法的同时，必须重写hashCode方法？** 主要是因为java提供的API是满足上面所说的条件，但当重写了equal后，如果不重写HashCode方法，可能会出现两个对象相同但是HashCode不同，违反了java设计原则。此时在使用HashMap等API便会出错。

# 三、HashMap

[1.7版本 Map 综述（一）：彻头彻尾理解 HashMap](https://blog.csdn.net/justloveyou_/article/details/62893086)  [ 1.8以后版本 HashMap 简介](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/HashMap.md )    

## Java对象数组

长时间使用java的List都快忘记了原始的数组使用方式。

```java
Object [] objects = new Object[5];// 初始化一个长度为5的Object对象的数组。
List<Object> objectList = new ArrayList<>(); // 初始化一个ArrayList数组。

// 新增数据 二者实现的是一样的。
objects[0] = new Object();
objectList.add(new Object());
```

对于上面的两种方式都是初始化两个数组，但是后者是使用一个更加强大的数组来进行实现。看到HashSet的内部实现竟然没看懂是怎么回事，使用`new Object[5]`的方式进行初始化数组，也就是实现一个空壳。然后可以往里面填对象。

## HashMap实现原理

Map 是 Key-Value 对映射的抽象接口，该映射不包括重复的键，即一个键对应一个值。HashMap 是 Java Collection Framework 的重要成员，也是Map族(如下图所示)中我们最为常用的一种。简单地说，HashMap 是基于哈希表的 Map 接口的实现，以 Key-Value 的形式存在，即存储的对象是 Entry (同时包含了 Key 和 Value) 。在HashMap中，其会根据hash算法来计算key-value的存储位置并进行快速存取。特别地，HashMap最多只允许一条Entry的键为Null(多条会覆盖)，但允许多条Entry的值为Null。此外，HashMap 是 Map 的一个非同步的实现。

**主要需要记住的就是初始容量为16，即只有16个桶可以放数据，并且负载因子为0.75，所以最多只能存储threadhold = 16\*0.75= 12 个元素，当HashMap中存储的元素大于12时，就需要进行扩容，并且默认扩容是按照扩大二倍进行，即16\*2。**

![HashMap.png](https://pic.tyzhang.top/images/2020/07/17/HashMap.png)

另外，**虽然 HashMap 和 HashSet 实现的接口规范不同，但是它们底层的 Hash 存储机制完全相同。实际上，HashSet 本身就是在 HashMap 的基础上实现的。**

## 底层数据分析

JDK1.8 之前 HashMap 底层是 **数组和链表** 结合在一起使用也就是 **链表散列**。**HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 `(n - 1) & hash` 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。**

**所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。**

**JDK 1.8 HashMap 的 hash 方法源码:**

JDK 1.8 的 hash方法 相比于 JDK 1.7 hash 方法更加简化，但是原理不变。

```java
static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^ ：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

对比一下 JDK1.7的 HashMap 的 hash 方法源码.

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次。

所谓 **“拉链法”** 就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

而Jdk1.8以后，在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。

类属性：

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
 
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
   
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    // 这个是在第一次使用的时候才会被初始化，另外在进行扩容的时候也会被初始化，所有都写在了resize()方法中。
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table; 
    
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    
    // 加载因子
    final float loadFactor;
}
```

**Node节点类源码** 具体的存储结构。

```java
// 首先会使用这个Node生成Table 即hash桶，
// 然后 具体的节点也是存储在这里面 K:hashCode V:Object  
// 此出构建桶就是上面所说的对象数组。

// 这个是resize()方法中进行的初始化，
// Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
// table = newTab;

// 继承自 Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
       final int hash;// 哈希值，存放元素到hashmap中时用来与其他元素hash值比较
       final K key;//键
       V value;//值
       // 指向下一个节点
       Node<K,V> next;
       Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        // 重写hashCode()方法
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        // 重写 equals() 方法
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
}
```

**树节点类源码:** 即红黑树的结点结构。

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // 父
        TreeNode<K,V> left;    // 左
        TreeNode<K,V> right;   // 右
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;           // 判断颜色
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
        // 返回根节点
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
       }
```

## HashMap源码分析

### 确定哈希桶数组索引位置

不管增加、删除、查找键值对，定位到哈希桶数组的位置都是很关键的第一步。前面说过HashMap的数据结构是数组和链表的结合，所以我们当然希望这个HashMap里面的元素位置尽量分布均匀些，尽量使得每个位置上的元素数量只有一个，那么当我们用hash算法求得这个位置的时候，马上就可以知道对应位置的元素就是我们要的，不用遍历链表，大大优化了查询的效率。HashMap定位数组索引位置，直接决定了hash方法的离散性能。先看看源码的实现(方法一+方法二):

```java
//方法一：
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

//方法二： 根据hash值找到在table中的位置
static int indexFor(int h, int length) {  //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
     return h & (length-1);  //第三步 取模运算
}
```

这里的Hash算法本质上就是三步：**取key的hashCode值、高位运算、取模运算**。

对于任意给定的对象，只要它的hashCode()返回值相同，那么程序调用方法一所计算得到的Hash码值总是相同的。我们首先想到的就是把hash值对数组长度取模运算，这样一来，元素的分布相对来说是比较均匀的。但是，模运算的消耗还是比较大的，在HashMap中是这样做的：调用方法二来计算该对象应该保存在table数组的哪个索引处。

这个方法非常巧妙，它通过h & (table.length -1)来得到该对象的保存位，而HashMap底层数组的长度总是2的n次方，这是HashMap在速度上的优化。当length总是2的n次方时，h& (length-1)运算等价于对length取模，也就是h%length，但是&比%具有更高的效率。

在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

下面举例说明下，n为table的长度。 参考： [Java8系列之重新认识HashMap](https://zhuanlan.zhihu.com/p/21673805)

![hashMap_hash.jpg](https://pic.tyzhang.top/images/2020/07/17/hashMap_hash.jpg)

### 构造方法

HashMap 中有四个构造方法，它们分别如下：

```java
 	// 默认构造函数。
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all   other fields defaulted
     }
     
     // 包含另一个“Map”的构造函数
     public HashMap(Map<? extends K, ? extends V> m) {
         this.loadFactor = DEFAULT_LOAD_FACTOR;
         putMapEntries(m, false);//下面会分析到这个方法
     }
     
     // 指定“容量大小”的构造函数
     public HashMap(int initialCapacity) {
         this(initialCapacity, DEFAULT_LOAD_FACTOR);
     }
     
     // 指定“容量大小”和“加载因子”的构造函数
     public HashMap(int initialCapacity, float loadFactor) {
         if (initialCapacity < 0)
             throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
         if (initialCapacity > MAXIMUM_CAPACITY)
             initialCapacity = MAXIMUM_CAPACITY;
         if (loadFactor <= 0 || Float.isNaN(loadFactor))
             throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
         this.loadFactor = loadFactor;
         this.threshold = tableSizeFor(initialCapacity);
     }
```

**putMapEntries方法：**

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // 判断table是否已经初始化
        if (table == null) { // pre-size
            // 未初始化，s为m的实际元素个数
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            // 计算得到的t大于阈值，则初始化阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // 已初始化，并且m元素个数大于阈值，进行扩容处理
        else if (s > threshold)
            resize();
        // 将m中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

### put方法

HashMap只提供了put用于添加元素，putVal方法只是给put方法调用的一个方法，并没有提供给用户使用。

**对putVal方法添加元素的分析如下：**

- ①如果定位到的数组位置没有元素 就直接插入。
- ②如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个树节点，如果是就调用`e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)`将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // hash值不相等，即key不相等；为红黑树结点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) { 
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
} 
```

**我们再来对比一下 JDK1.7 put方法的代码**

**对于put方法的分析如下：**

- ①如果定位到的数组位置没有元素 就直接插入。
- ②如果定位到的数组位置有元素，遍历以这个元素为头结点的链表，依次和插入的key比较，如果key相同就直接覆盖，不同就采用头插法插入元素。

```java
public V put(K key, V value)
    if (table == EMPTY_TABLE) { 
    	inflateTable(threshold); 
	}  
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) { // 先遍历
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue; 
        }
    }

    modCount++;
    addEntry(hash, key, value, i);  // 再插入
    return null;
}
```

### get方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### JDK1.7 resize()方法

1.8版本的比较复杂，一般都是使用1.7的来进行说明，实现方法，首先构建一个新的桶，然后将旧桶中的数据遍历，重新计算索引并且使用头插法将所有数据转移到新桶中。

```java
void resize(int newCapacity) {   //传入新的容量
    Entry[] oldTable = table;    //引用扩容前的Entry数组
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {  //扩容前的数组大小如果已经达到最大(2^30)了
        threshold = Integer.MAX_VALUE; //修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
        return;
    }

    Entry[] newTable = new Entry[newCapacity];  //初始化一个新的Entry数组
    transfer(newTable);                         //！！将数据转移到新的Entry数组里
    table = newTable;                           //HashMap的table属性引用新的Entry数组
    threshold = (int) (newCapacity * loadFactor);//修改阈值
}
```

调用Transfer进行扩容并且使用头插法复制链表。

```java
void transfer(Entry[] newTable) {
    Entry[] src = table;                   //src引用了旧的Entry数组
    int newCapacity = newTable.length;
    for (int j = 0; j < src.length; j++) { //遍历旧的Entry数组
        Entry<K, V> e = src[j];             //取得旧Entry数组的每个元素
        if (e != null) {
            src[j] = null;//释放旧Entry数组的对象引用（for循环后，旧的Entry数组不再引用任何对象）
            do {
                Entry<K, V> next = e.next;
                int i = indexFor(e.hash, newCapacity); //！！重新计算每个元素在数组中的位置
                e.next = newTable[i]; //标记[1]
                newTable[i] = e;      //将元素放在数组上
                e = next;             //访问下一个Entry链上的元素
            } while (e != null);
        }
    }
}
```

寻找索引的值。

```java
static int indexFor(int h, int length) {
    return h & (length - 1);
}
```

以下为使用头插法进行复制的过程，在没扩容之前表大小为0-4，如下所示。并且有两个元素在位置4处。

![resize_init.png](https://pic.tyzhang.top/images/2020/07/17/resize_init.png)

下面是扩容到0-5表中的场景，(ps，HashMap扩容应该是2倍，仅以此说明)。以下为`do...while()`循环里面的过程，可以看出，使用头插法进行扩容之后的顺序为反的，相当于逆序。

![resize_finnal.png](https://pic.tyzhang.top/images/2020/07/17/resize_finnal.png)

### JDK1.8 计算新桶中的位置

参考： [Java 8系列之重新认识HashMap](https://zhuanlan.zhihu.com/p/21673805)  

下面举个例子说明下扩容过程。假设了我们的hash算法就是简单的用key mod 一下表的大小（也就是数组的长度）。其中的哈希桶数组table的size=2， 所以key = 3、7、5，put顺序依次为 5、7、3。在mod 2以后都冲突在table[1]这里了。这里假设负载因子 loadFactor=1，即当键值对的实际大小size 大于 table的实际大小时进行扩容。接下来的三个步骤是哈希桶数组 resize成4，然后所有的Node重新rehash的过程。

![resize_all.jpg](https://pic.tyzhang.top/images/2020/07/17/resize_all.jpg)

下面我们讲解下JDK1.8做了哪些优化。经过观测可以发现，我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果。

![resize_hash.jpg](https://pic.tyzhang.top/images/2020/07/17/resize_hash.jpg)

元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![resize_hash_result.png](https://pic.tyzhang.top/images/2020/07/17/resize_hash_result.png)

### JDK1.8 resize()方法

进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize。另外第一次进行put的时候也会调用这个方法，即进行初始化。

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else { 
        // signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

# 四、ArrayList

[本文参考链接](https://github.com/LostStarTvT/JavaGuide/blob/master/docs/java/collection/ArrayList.md)  [Fail-Fast](https://juejin.im/post/5cb683d6518825186d65402c#heading-0) 

ArrayList 的底层是数组队列，相当于动态数组。与 Java 中的数组相比(`int [] array = new int[];`)，它的容量能动态增长。在添加大量元素前，应用程序可以使用`ensureCapacity`操作来增加 ArrayList 实例的容量。这可以减少递增式再分配的数量。与原生的数组相比，ArrayList 可以更加方便的进行操作，原生的数组和C语言中的数据是类似的，另外在进行扩容的时候，大量使用的就是复制技术`System.arraycopy()`底层方法进行相应的操作。

它继承于 **AbstractList**，实现了 **List**, **RandomAccess**, **Cloneable**, **java.io.Serializable** 这些接口。

在我们学数据结构的时候就知道了线性表的顺序存储，插入删除元素的时间复杂度为**O(n)**,求表长以及增加元素，取第 i 元素的时间复杂度为**O(1)**

ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。

ArrayList 实现了**RandomAccess 接口**， RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是支持**快速随机访问**的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。

ArrayList 实现了**Cloneable 接口**，即覆盖了函数 clone()，**能被克隆**。

ArrayList 实现**java.io.Serializable 接口**，这意味着ArrayList**支持序列化**，**能通过序列化去传输**。

和 Vector 不同，**ArrayList 中的操作不是线程安全的**！所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择 Vector 或者 CopyOnWriteArrayList。

## 核心源码

核心源码中主要涉及到的点：

1. 数组默认的长度为10，每次扩容都是扩容当前数组的1.5倍长。
2. 核心的扩容方法为grow()方法。
3. 核心的保存数组为`transient Object[] elementData;` 使用Object数组进行数据的保存。
4. 其他常用的 删除 remove、 增加 add、长度size、等方法。
5. 其中`ensureCapacity()`方法比较重要，List进行扩容的入口方法，默认情况下在进行add时候，会进行检测List的空间是否够用，不够用便会进行扩容。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 空数组（用于空实例）。
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

     //用于默认大小空实例的共享空数组实例。
      //我们把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 保存ArrayList数据的数组
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList 所包含的元素个数
     */
    private int size;

    /**
     * 带初始容量参数的构造函数。（用户自己指定容量）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     *默认构造函数，DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为0.初始化为10，也就是说初始其实是空数组 当添加第一个元素的时候数组容量才变成10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
     */
    public ArrayList(Collection<? extends E> c) {
        //
        elementData = c.toArray();
        //如果指定集合元素个数不为0
        if ((size = elementData.length) != 0) {
            // c.toArray 可能返回的不是Object类型的数组所以加上下面的语句用于判断，
            //这里用到了反射里面的getClass()方法
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 用空数组代替
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    /**
     * 修改这个ArrayList实例的容量是列表的当前大小。 应用程序可以使用此操作来最小化ArrayList实例的存储。 
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
	//下面是ArrayList的扩容机制
	//ArrayList的扩容机制提高了性能，如果每次只扩充一个，
	//那么频繁的插入会导致频繁的拷贝，降低性能，而ArrayList的扩容机制避免了这种情况。
    /**
     * 如有必要，增加此ArrayList实例的容量，以确保它至少能容纳元素的数量
     * @param   minCapacity   所需的最小容量
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)? 0: DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
   //得到最小扩容量
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取默认的容量和传入参数的较大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
  	//判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容，调用此方法代表已经开始扩容了
            grow(minCapacity);
    }

    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //再检查新容量是否超出了ArrayList所定义的最大容量，
        //若超出了，则调用hugeCapacity()来比较minCapacity和 MAX_ARRAY_SIZE，
        //如果minCapacity大于MAX_ARRAY_SIZE，则新容量则为Interger.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    //比较minCapacity和 MAX_ARRAY_SIZE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /**
     *返回此列表中的元素数。 
     */
    public int size() {
        return size;
    }

    /**
     * 如果此列表不包含元素，则返回 true 。
     */
    public boolean isEmpty() {
        //注意=和==的区别
        return size == 0;
    }

    /**
     * 如果此列表包含指定的元素，则返回true 。
     */
    public boolean contains(Object o) {
        //indexOf()方法：返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1 
        return indexOf(o) >= 0;
    }

    /**
     *返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1 
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                //equals()方法比较
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此列表中指定元素的最后一次出现的索引，如果此列表不包含元素，则返回-1。.
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此ArrayList实例的浅拷贝。 （元素本身不被复制。） 
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            //Arrays.copyOf功能是实现数组的复制，返回复制后的数组。参数是被复制的数组和复制的长度
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // 这不应该发生，因为我们是可以克隆的
            throw new InternalError(e);
        }
    }

    /**
     *以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。 
     *返回的数组将是“安全的”，因为该列表不保留对它的引用。 （换句话说，这个方法必须分配一个新的数组）。
     *因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）; 
     *返回的数组的运行时类型是指定数组的运行时类型。 如果列表适合指定的数组，则返回其中。 
     *否则，将为指定数组的运行时类型和此列表的大小分配一个新数组。 
     *如果列表适用于指定的数组，其余空间（即数组的列表数量多于此元素），则紧跟在集合结束后的数组中的元素设置为null 。
     *（这仅在调用者知道列表不包含任何空元素的情况下才能确定列表的长度。） 
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 新建一个运行时类型的数组，但是ArrayList数组的内容
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
            //调用System提供的arraycopy()方法实现数组之间的复制
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    // Positional Access Operations

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 返回此列表中指定位置的元素。
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * 用指定的元素替换此列表中指定位置的元素。 
     */
    public E set(int index, E element) {
        //对index进行界限检查
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        //返回原来在这个位置的元素
        return oldValue;
    }

    /**
     * 将指定的元素追加到此列表的末尾。 
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }

    /**
     * 在此列表中的指定位置插入指定的元素。 
     *先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
     *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //arraycopy()这个实现数组之间复制的方法一定要看一下，下面就用到了arraycopy()方法实现数组自己复制自己
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * 删除该列表中指定位置的元素。 将任何后续元素移动到左侧（从其索引中减去一个元素）。 
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
      //从列表中删除的元素 
        return oldValue;
    }

    /**
     * 从列表中删除指定元素的第一个出现（如果存在）。 如果列表不包含该元素，则它不会更改。
     *返回true，如果此列表包含指定的元素
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    /**
     * 从列表中删除所有元素。 
     */
    public void clear() {
        modCount++;

        // 把数组中所有的元素的值设为null
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * 按指定集合的Iterator返回的顺序将指定集合中的所有元素追加到此列表的末尾。
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 将指定集合中的所有元素插入到此列表中，从指定的位置开始。
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 从此列表中删除所有索引为fromIndex （含）和toIndex之间的元素。
     *将任何后续元素移动到左侧（减少其索引）。
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * 检查给定的索引是否在范围内。
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * add和addAll使用的rangeCheck的一个版本
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 返回IndexOutOfBoundsException细节信息
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    /**
     * 从此列表中删除指定集合中包含的所有元素。 
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        //如果此列表被修改则返回true
        return batchRemove(c, false);
    }

    /**
     * 仅保留此列表中包含在指定集合中的元素。
     *换句话说，从此列表中删除其中不包含在指定集合中的所有元素。 
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }


    /**
     * 从列表中的指定位置开始，返回列表中的元素（按正确顺序）的列表迭代器。
     *指定的索引表示初始调用将返回的第一个元素为next 。 初始调用previous将返回指定索引减1的元素。 
     *返回的列表迭代器是fail-fast 。 
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     *返回列表中的列表迭代器（按适当的顺序）。 
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     *以正确的顺序返回该列表中的元素的迭代器。 
     *返回的迭代器是fail-fast 。 
     */
    public Iterator<E> iterator() {
        return new Itr();
    }
}
```

## 源码分析

### ensureCapacity()方法

顾名思义这个方法就是检测数组的容量是否够用，在每次进行add()元素的时候都会调用改方法，并且该方法还是一个public方法，意味着程序员也能够进行调用。

```java
public void ensureCapacity(int minCapacity) {
    // DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {} 默认空数组
    // DEFAULT_CAPACITY 长度为10的数组。    
    // 首先判断当前的数组的长度，是为空还是为长度为10的默认数组，
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)? 0: DEFAULT_CAPACITY;
	
    // 当需要扩容的长度minCapacity 是大于 10或则是0 的时候才会被扩容， 主要是在默认时候需要进行的检测。
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```

因为默认的数组长度为10，所以如果当我们操作大量数据的时候可以调用该方法先进行扩容，通过传递minCapacity参数，进行预先扩容，这样就可以避免重复的扩容可以增加执行效率，因为List中的扩容是通过复制进行的，比较耗时。

### System.arraycopy()和Arrays.copyOf()方法

通过上面源码我们发现这两个实现数组复制的方法被广泛使用而且很多地方都特别巧妙。比如下面add(int index, E element)方法就很巧妙的用到了arraycopy()方法让数组自己复制自己实现让index开始之后的所有成员后移一个位置:

```java
/**
 * 在此列表中的指定位置插入指定的元素。 
 *先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
 *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
*/

public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //arraycopy()方法实现数组自己复制自己
    //elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的起始位置； size - index：要复制的数组元素的数量；
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
    size++;
}
```

又如toArray()方法中用到了copyOf()方法

```java
/**
 *以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。 
 *返回的数组将是“安全的”，因为该列表不保留对它的引用。 （换句话说，这个方法必须分配一个新的数组）。
 *因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。
 */
public Object[] toArray() {
    //elementData：要复制的数组；size：要复制的长度
    return Arrays.copyOf(elementData, size);
}
```

### 两者联系与区别

**联系：** 看两者源代码可以发现`copyOf()`内部调用了`System.arraycopy()`方法 **区别：**

1. arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置
2. copyOf()是系统自动在内部新建一个数组，并返回该数组。

### 扩容机制

```java
//下面是ArrayList的扩容机制
//ArrayList的扩容机制提高了性能，如果每次只扩充一个，
// 那么频繁的插入会导致频繁的拷贝，降低性能，而ArrayList的扩容机制避免了这种情况。
/**
 * 如有必要，增加此ArrayList实例的容量，以确保它至少能容纳元素的数量
 * @param   minCapacity   所需的最小容量
*/
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // any size if not default element table
        ? 0
        // larger than default for default empty table. It's already
        // supposed to be at default size.
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}

//得到最小扩容量
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 获取默认的容量和传入参数的较大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

//判断是否需要扩容,上面两个方法都要调用
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 如果说minCapacity也就是所需的最小容量大于保存ArrayList数据的数组的长度的话，就需要调用grow(minCapacity)方法扩容。
    //这个minCapacity到底为多少呢？举个例子在添加元素(add)方法中这个minCapacity的大小就为现在数组的长度加1
    if (minCapacity - elementData.length > 0)
        //调用grow方法进行扩容，调用此方法代表已经开始扩容了
        grow(minCapacity);
}
```

grow方法详解。

```java
/**
 * ArrayList扩容的核心方法。
*/
private void grow(int minCapacity) {
    //elementData为保存ArrayList数据的数组
    ///elementData.length求数组长度elementData.size是求数组中的元素个数
    // oldCapacity为旧容量，newCapacity为新容量
    int oldCapacity = elementData.length;
    //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
    //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //再检查新容量是否超出了ArrayList所定义的最大容量，
    //若超出了，则调用hugeCapacity()来比较minCapacity和 MAX_ARRAY_SIZE，
    //如果minCapacity大于MAX_ARRAY_SIZE，则新容量则为Interger.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE。
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

扩容机制代码已经做了详细的解释。另外值得注意的是大家很容易忽略的一个运算符：**移位运算符** 　

**简介**：移位运算符就是在二进制的基础上对数字进行平移。按照平移的方向和填充数字的规则分为三种:<<(左移)、>>(带符号右移)和>>>(无符号右移)。 　  　

**作用**：**对于大数据的2进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源**  

比如这里：int newCapacity = oldCapacity + (oldCapacity >> 1); 右移一位相当于除2，右移n位相当于除以 2 的 n 次方。这里 oldCapacity 明显右移了1位所以相当于oldCapacity /2。

**另外需要注意的是：**

1. java 中的**length 属性**是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 length 这个属性.
2. java 中的**length()方法**是针对字 符串String说的,如果想看这个字符串的长度则用到 length()这个方法.
3. .java 中的**size()方法**是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看!

### 内部类

ArrayList总共有三个内部类，

```java
// 进行迭代的时候进行的调用 
private class Itr implements Iterator<E> {..}
private class ListItr extends Itr implements ListIterator<E> {...}
private class SubList extends AbstractList<E> implements RandomAccess {..}
```

相当于有两个迭代器，在进行调用迭代器的时候其实都是在新建一个迭代器对象。

```java
    /**
     * Returns a list iterator over the elements in this list (in proper
     * sequence), starting at the specified position in the list.
     * The specified index indicates the first element that would be
     * returned by an initial call to {@link ListIterator#next next}.
     * An initial call to {@link ListIterator#previous previous} would  
     * return the element with the specified index minus one.
     *
     * <p>The returned list iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     * Returns a list iterator over the elements in this list (in proper
     * sequence).
     *
     * <p>The returned list iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
     *
     * @see #listIterator(int)
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

```



```java
public Iterator<E> iterator() {
    return new Itr();
}

```



# 五、LinkList

[参考链接](https://github.com/LostStarTvT/JavaGuide/blob/master/docs/java/collection/LinkedList.md)  

LinkedList是一个实现了List接口和Deque接口的双端链表。 LinkedList底层的链表结构使它支持高效的插入和删除操作，另外它实现了Deque接口，使得LinkedList类也具有队列的特性; LinkedList不是线程安全的，如果想使LinkedList变成线程安全的，可以调用静态类Collections类中的synchronizedList方法。

内部的存储结构：就是一个简单的双向链表。 

![LinkedList.jpg](https://pic.tyzhang.top/images/2020/07/24/LinkedList.jpg)

## LinkedList源码分析

### 内部节点

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### 构造方法

**空构造方法：**

```java
public LinkedList() {
}
```

**用已有的集合创建链表的构造方法：**

```java
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

### add方法

**add(E e)** 方法：将元素添加到链表尾部

```java
public boolean add(E e) {
        linkLast(e);//这里就只调用了这一个方法
        return true;
    }
   /**
     * 链接使e作为最后一个元素。
     */
    void linkLast(E e) {
        final Node<E> l = last;
        // 直接新建节点，然后将数据增加进去。
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;//新建节点
        if (l == null)
            first = newNode;
        else
            l.next = newNode;//指向后继元素也就是指向下一个元素
        size++;
        modCount++;
    }
```

**add(int index,E e)**：在指定位置添加元素

```java
public void add(int index, E element) {
    checkPositionIndex(index); //检查索引是否处于[0-size]之间

    if (index == size)//添加在链表尾部
        linkLast(element);
    else//添加在链表中间
        linkBefore(element, node(index));
}
```

linkBefore方法需要给定两个参数，一个插入节点的值，一个指定的node，所以我们又调用了Node(index)去找到index对应的node

**addAll(Collection c )：将集合插入到链表尾部**

```java
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
```

**addAll(int index, Collection c)：** 将集合从指定位置开始插入

```java
public boolean addAll(int index, Collection<? extends E> c) {
        //1:检查index范围是否在size之内
        checkPositionIndex(index);

        //2:toArray()方法把集合的数据存到对象数组中
        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        //3：得到插入位置的前驱节点和后继节点
        Node<E> pred, succ;
        //如果插入位置为尾部，前驱节点为last，后继节点为null
        if (index == size) {
            succ = null;
            pred = last;
        }
        //否则，调用node()方法得到后继节点，再得到前驱节点
        else {
            succ = node(index);
            pred = succ.prev;
        }

        // 4：遍历数据将数据插入
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            //创建新节点
            Node<E> newNode = new Node<>(pred, e, null);
            //如果插入位置在链表头部
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        //如果插入位置在尾部，重置last节点
        if (succ == null) {
            last = pred;
        }
        //否则，将插入的链表与先前链表连接起来
        else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }    
```

上面可以看出addAll方法通常包括下面四个步骤：

1. 检查index范围是否在size之内
2. toArray()方法把集合的数据存到对象数组中
3. 得到插入位置的前驱和后继节点
4. 遍历数据，将数据插入到指定位置

**addFirst(E e)：** 将元素添加到链表头部

```java
 public void addFirst(E e) {
        linkFirst(e);
    }
private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);//新建节点，以头节点为后继节点
        first = newNode;
        //如果链表为空，last节点也指向该节点
        if (f == null)
            last = newNode;
        //否则，将头节点的前驱指针指向新节点，也就是指向前一个元素
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
```

**addLast(E e)：** 将元素添加到链表尾部，与 **add(E e)** 方法一样

```java
public void addLast(E e) {
        linkLast(e);
    }
```

### 根据位置取数据的方法

**get(int index)：** 根据指定索引返回数据

```java
public E get(int index) {
        //检查index范围是否在size之内
        checkElementIndex(index);
        //调用Node(index)去找到index对应的node然后返回它的值
        return node(index).item;
    }
```

**获取头节点（index=0）数据方法:**

```java
public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
public E element() {
        return getFirst();
    }
public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }
```

**区别：** getFirst(),element(),peek(),peekFirst() 这四个获取头结点方法的区别在于对链表为空时的处理，是抛出异常还是返回null，其中**getFirst()** 和**element()** 方法将会在链表为空时，抛出异常

element()方法的内部就是使用getFirst()实现的。它们会在链表为空时，抛出NoSuchElementException
**获取尾节点（index=-1）数据方法:**

```java
 public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
 public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }
```

**两者区别：** **getLast()** 方法在链表为空时，会抛出**NoSuchElementException**，而**peekLast()** 则不会，只是会返回 **null**。

### 根据对象得到索引的方法

**int indexOf(Object o)：** 从头遍历找

```java
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        //从头遍历
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        //从头遍历
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

**int lastIndexOf(Object o)：** 从尾遍历找

```java
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        //从尾遍历
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        //从尾遍历
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```

### 检查链表是否包含某对象的方法：

**contains(Object o)：** 检查对象o是否存在于链表中

```java
 public boolean contains(Object o) {
        return indexOf(o) != -1;
    }
```

### 删除方法

**remove()** ,**removeFirst(),pop():** 删除头节点

```java
public E pop() {
        return removeFirst();
    }
public E remove() {
        return removeFirst();
    }
public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
```

**removeLast(),pollLast():** 删除尾节点

```java
public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }
```

**区别：** removeLast()在链表为空时将抛出NoSuchElementException，而pollLast()方法返回null。

**remove(Object o):** 删除指定元素

```java
public boolean remove(Object o) {
        //如果删除对象为null
        if (o == null) {
            //从头开始遍历
            for (Node<E> x = first; x != null; x = x.next) {
                //找到元素
                if (x.item == null) {
                   //从链表中移除找到的元素
                    unlink(x);
                    return true;
                }
            }
        } else {
            //从头开始遍历
            for (Node<E> x = first; x != null; x = x.next) {
                //找到元素
                if (o.equals(x.item)) {
                    //从链表中移除找到的元素
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

当删除指定对象时，只需调用remove(Object o)即可，不过该方法一次只会删除一个匹配的对象，如果删除了匹配对象，返回true，否则false。

unlink(Node x) 方法：

```java
E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;//得到后继节点
        final Node<E> prev = x.prev;//得到前驱节点

        //删除前驱指针
        if (prev == null) {
            first = next;//如果删除的节点是头节点,令头节点指向该节点的后继节点
        } else {
            prev.next = next;//将前驱节点的后继节点指向后继节点
            x.prev = null;
        }

        //删除后继指针
        if (next == null) {
            last = prev;//如果删除的节点是尾节点,令尾节点指向该节点的前驱节点
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

**remove(int index)**：删除指定位置的元素

```java
public E remove(int index) {
        //检查index范围
        checkElementIndex(index);
        //将节点删除
        return unlink(node(index));
    }
```

# 六、Queue

[java中的队列Queue的使用](https://juejin.im/post/5e1d517ef265da3df860faca)

队列就是使用链表或则是数组进行实现。 简单记录下队列的使用方法就行。

## 队列是什么 

队列是一种重要的抽象**数据结构**，可类比于生活中的排队场景Java 语言提供了队列的支持，内置了多种类型的队列供我们使用 队列的特点是**先进先出**。队列的用处很大

## 为什么要使用队列

例如：

**消息队列**是用来解决这样的问题的：将突发的大量请求转换为服务器能够处理的队列请求。eg:在一个秒杀活动中，服务器1秒可以处理100条请求。而在秒杀活动开启时1秒进来1000个请求并且持续10秒。这个时候就需要将这10000个请求放入消息队列里面，后端按照原来的能力处理，用100秒将队列中的请求处理完毕。这样就不会导致宕机

## java中如何使用队列

### 单向队列

单向队列比较简单，只能向**队尾添加元素**，从**队头删除元素**。比如最典型的**排队买票**例子，新来的人只能在队列后面，排到最前边的人才可以买票，买完了以后，离开队伍。这个过程是一个非常典型的队列。

Java 定义了队列的基本操作，接口类型为 java.util.Queue，接口定义如下所示。Queue 定义了两套队列操作方法：

- add、remove、element 操作失败抛出异常；
- offer 操作失败返回 false 或抛出异常，poll、peek 操作失败返回 null；

```java
public interface Queue<E> extends Collection<E> {
    //插入元素，成功返回true，失败抛出异常
    boolean add(E e);

    //插入元素，成功返回true，失败返回false或抛出异常 
    boolean offer(E e);

    //取出并移除头部元素，空队列抛出异常 
    E remove();

    //取出并移除头部元素，空队列返回null 
    E poll();

    //取出但不移除头部元素，空队列抛出异常 
    E element();

    //取出但不移除头部元素，空队列返回null 
    E peek();
}
```

### 双向队列

Deque : double ended queue的缩写。

如果一个队列的头和尾都支持元素入队，出队，那么这种队列就称为双向队列，英文是Deque，可以通过java.util.Deque来查看Deque的接口定义，Deque 也同样定义了两套队列操作方法，针对头部操作方法为 xxxFirst、针对尾部操作方法为 xxxLast：

- add、remove、get 操作失败抛出异常；
- offer 操作失败返回 false 或抛出异常，poll、peek 操作失败返回 null；
- Deque 另外还有 removeFirstOccurrence、removeLastOccurrence 方法用于删除指定元素，元素存在则删除，不存在则队列不变。

```java
public interface Deque<E> extends Queue<E> {
    //插入元素到队列头部，失败抛出异常 
    void addFirst(E e);

    //插入元素到队列尾部，失败抛出异常  
    void addLast(E e);

    //插入元素到队列头部，失败返回false或抛出异常 
    boolean offerFirst(E e);

    //插入元素到队列尾部，失败返回false抛出异常  
    boolean offerLast(E e);

    //取出并移除头部元素，空队列抛出异常 
    E removeFirst();

    //取出并移除尾部元素，空队列抛出异常 
    E removeLast();

    //取出并移除头部元素，空队列返回null
    E pollFirst();

    //取出并移除尾部元素，空队列返回null
    E pollLast();

    //取出但不移除头部元素，空队列抛出异常
    E getFirst();

    //取出但不移除尾部元素，空队列抛出异常
    E getLast();

    //取出但不移除头部元素，空队列返回null
    E peekFirst();

    //取出但不移除尾部元素，空队列返回null
    E peekLast();

    //移除指定头部元素，若不存在队列不变，移除成功返回true 
    boolean removeFirstOccurrence(Object o);

    //移除指定尾部元素，若不存在队列不变，移除成功返回true 
    boolean removeLastOccurrence(Object o);

    //单向队列方法，参考Queue   
    //栈方法，参考栈
    //集合方法，参考集合定义   
}
```

### 队列的具体实现

通常情况下，队列有数组和链表两种实现方式。

1. 采用链表实现的队列，没有个数限制。插入元素时直接接在链表的尾部，取出元素时直接从链表的头部取出即可。
2. 采用数组实现的队列，通常是循环数组，受限于数组的大小，存在天然的个数上限。插入和取出元素时，必须采用队列头部指针和队列尾部指针进行队列满和队列空的判断。

#### 单向队列

1. **LinkedList**基于链表的实现方式是线程不安全的
2. **PriorityQueue**无界的优先级队列，保存队列元素的顺序不是按照及加入队列的顺序，而是按照队列元素的大小进行重新排序。因此当调用peek()或pool()方法取出队列中头部的元素时，并不是取出最先进入队列的元素，而是取出队列的**最小**元素。

> 我们刚刚才说到队列的特点是先进先出，为什么这里就按照大小顺序排序了呢？我们还是先看一下它的介绍，直接翻译过来：

换句话说，使用PriorityQueue也就是可以实现大顶堆或者是小顶堆。其中默认情况下PriorityQueue是想的是小顶堆，即第一个元素为最小值。

大顶堆的实现方式如下：

```java
// 1 2 3 4 5  则小顶堆存放345即存放后半部分数，我们需要的是3  12为大顶堆，因为我们需要的是2，即将数据分为前半部分和后半部分。
// 小顶堆
static PriorityQueue<Integer> low = new PriorityQueue<>();

// 大顶对，逆序
static PriorityQueue<Integer> high = new PriorityQueue<>(new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2.compareTo(o1);
    }
});
```

基于优先级堆的无界的优先级队列。 PriorityQueue的元素根据自然排序进行排序，或者按队列构建时提供的 Comparator进行排序，具体取决于使用的构造方法。 优先队列不允许 null 元素。 通过自然排序的PriorityQueue不允许插入不可比较的对象。 该队列的头是根据指定排序的最小元素。 如果多个元素都是最小值，则头部是其中的一个元素——任意选取一个。 队列检索操作poll、remove、peek和element访问队列头部的元素。 优先队列是无界的，但有一个内部容量，用于管理用于存储队列中元素的数组的大小。 基本上它的大小至少和队列大小一样大。 当元素被添加到优先队列时，它的容量会自动增长。增长策略的细节没有指定。

> 一句话概括，PriorityQueue使用了一个高效的数据结构：堆。底层是使用数组保存数据。还会进行排序，优先将元素的最小值存到队头

PriorityQueue 本质也是一个动态数组，在这一方面与ArrayList是一致的

```java
public PriorityQueue(int initialCapacity) {
    this(initialCapacity, null);
}

public PriorityQueue(Comparator<? super E> comparator) {
    this(DEFAULT_INITIAL_CAPACITY, comparator);
}

public PriorityQueue(int initialCapacity,
                     Comparator<? super E> comparator) {
    // Note: This restriction of at least one is not actually needed,
    // but continues for 1.5 compatibility
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.queue = new Object[initialCapacity];
    this.comparator = comparator;
}
```

- PriorityQueue调用默认的构造方法时，使用默认的初始容量（DEFAULT_IITIAL_CAPACITY = 11）创建一个PriorityQueue，并根据其自然顺序来排序其元素（使用加入其中的集合元素实现的Comparable）。
- 当使用指定容量的构造方法时，使用指定的初始容量创建一个 PriorityQueue，并根据其自然顺序来排序其元素（使用加入其中的集合元素实现的Comparable）
- 当使用指定的初始容量创建一个 PriorityQueue，并根据指定的比较器comparator来排序其元素。当添加元素到集合时，会先检查数组是否还有余量，有余量则把新元素加入集合，没余量则调用  grow()方法增加容量，然后调用siftUp将新加入的元素排序插入对应位置。  除了这些，还要注意的是：  1.PriorityQueue不是线程安全的。如果多个线程中的任意线程从结构上修改了列表， 则这些线程不应同时访问 PriorityQueue 实例，这时请使用线程安全的PriorityBlockingQueue 类。  2.不允许插入 null 元素。  3.PriorityQueue实现插入方法（offer、poll、remove() 和 add 方法） 的时间复杂度是O(log(n)) ；实现 remove(Object) 和 contains(Object) 方法的时间复杂度是O(n) ；实现检索方法（peek、element 和 size）的时间复杂度是O(1)。所以在遍历时，若不需要删除元素，则以peek的方式遍历每个元素。  4.方法iterator()中提供的迭代器并不保证以有序的方式遍历PriorityQueue中的元素。

1. **ConcurrentLinkedQueue******并发队列是一个基于链表实现的线程安全的无界队列。内部元素按先进先出的顺序排序，最后插入的元素位于队列尾部。ConcurrentLinkedQueue 不允许插入 null。当有多个线程同时访问队列时，此队列是一个比较合适的选择。

Queue 接口的子接口 BlockingQueue 定义了阻塞队列。阻塞队列访问队列元素时，可能会一直阻塞直到操作完成，比如从空队列中取元素、向满队列中插入元素等，主要有延时队列、同步队列、链表阻塞队列、数组阻塞队列、优先级阻塞队列。

#### 双向队列

Deque 接口的实现相对较少，主要有 LinkedList、ArrayDeque、ConcurrentLinkedDeque 和 LinkedBlockingDeque。

# 七、Stack

java中Stacke是基于Vector进行实现的。

```java
	/**
	 * Stack类
	 * 栈：桶型或箱型数据类型，后进先出，相对堆Heap为二叉树类型，可以快速定位并操作
	 * Stack<E>，支持泛型
	 * public class Stack<E> extends Vector<E>
	 * Stack的方法调用的Vector的方法，被synchronized修饰，为线程安全(Vector也是)
	 * Stack methods：
	 * push : 把项压入堆栈顶部 ，并作为此函数的值返回该对象
	 * pop : 移除堆栈顶部的对象，并作为此函数的值返回该对象 
	 * peek : 查看堆栈顶部的对象，，并作为此函数的值返回该对象，但不从堆栈中移除它
	 * empty : 测试堆栈是否为空 
	 * search : 返回对象在堆栈中的位置，以 1 为基数 
	 * */
```

但是基于Vector的栈可能效率不是很高，因为Vector是数组的线程安全版，现在建议使用双向Denqu进行实现栈的操作。

[为什么 java.util.Stack不被官方所推荐使用！](https://www.cnblogs.com/cosmos-wong/p/11845934.html)  

> 总结：在java.util.stack中，栈的底层是使用数组来实现的，数组初始大小为10。每当元素个数超出数组容量就扩展为原来的2倍，将原数组中的元素拷贝到新数组中，随着数组元素的增多，这种开销也越大。Java并不推荐使用java.util.stack来进行栈的操作，而是推荐使用一个双端队列“deque ”来代替它。

