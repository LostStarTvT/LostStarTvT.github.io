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

以下的规定为**重写equals时需要满足的条件，**默认也是满足的，因为只有当且仅当`System.out.println(student.equals(student));` 即自己和自己比较时才会返回True，对应到HashMap中就是“我找我自己，我就与我自己对比”。而默认情况下内容相同的对象返回False，所以要重写。

1. 如果两个对象相等，则 hashcode 一定也是相同的
2. 两个对象相等,对两个对象分别调用 equals 方法都返回 true
3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

[Java hashCode() 和 equals()的若干问题解答](https://www.cnblogs.com/skywang12345/p/3324958.html)  

那么为什么**在重写equals方法的同时，必须重写hashCode方法？** 主要是因为java提供的API是满足上面所说的条件，但当重写了equal后，如果不重写HashCode方法，可能会出现两个对象相同但是HashCode不同，违反了java设计原则。此时在使用HashMap等API便会出错。

# HashMap

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

主要需要记住的就是初始容量为16，即只有16个桶可以放数据，并且负载因子为0.75，所以最多只能存储threadhold = 16\*0.75= 12 个元素，当HashMap中存储的元素大于12时，就需要进行扩容，并且默认扩容是按照扩大二倍进行，即16\*2。

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
方法一：
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
方法二： 根据hash值找到在table中的位置
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

```
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

上面1.7版本的比较复杂，一般都是使用1.7的来进行说明，实现方法，首先构建一个新的桶，然后将旧桶中的数据遍历，重新计算索引并且使用头插法将所有数据转移到新桶中。

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

结束。。