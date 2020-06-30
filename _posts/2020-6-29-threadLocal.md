---
layout: post
title: Java：理解ThreadLocal
tags: java  
---


> 对于ThreadLocal还是不是很理解，在此做个总结

#  目录
* 目录
{:toc}
# 参考

[聊聊spring中的线程安全性](https://sylvanassun.github.io/2017/11/06/2017-11-06-spring_and_thread-safe/)  [面试再问ThreadLocal，别说你不会](https://juejin.im/post/5d427f306fb9a06b122f1b94)

# 概述

ThreadLocal是线程同步中的一个工具类，之前一直没有仔细的思考过这个类，首先，他解决的是在多线程中的资源共享问题，所以不能使用单线程的思想去思考这个东西。**线程同步机制是多个线程共享同一个变量，而ThreadLocal是为每个线程创建一个单独的变量副本，每个线程都可以改变自己的变量副本而不影响其它线程所对应的副本**。

ThreadLocal与像synchronized这样的锁机制是不同的。首先，它们的应用场景与实现思路就不一样，**锁更强调的是如何同步多个线程去正确地共享一个变量，ThreadLocal则是为了解决同一个变量如何不被多个线程共享**。从性能开销的角度上来讲，如果锁机制是用时间换空间的话，那么ThreadLocal就是用空间换时间。  

其应用场景在多线程中，比如说，在一个多线程的类中进行统计每个线程的调用次数，此时如果只是在线程中定义一个变量，是完成不了相应的功能，因为所有的线程都会去竞争去操作这个变量，然后去自加。这时候就需要使用ThreadLocal，为每一个线程都复制一份需要的变量去进行统计。

# 实例

```java
public class SeqCount {

    //虽然是个外部变量，但是每个线程在使用的时候都会将该值放在自己的局部变量中，各自更改互不干扰。
    private static ThreadLocal<Integer> seqCount = new ThreadLocal<Integer>() {
        // 进行初始化数据。
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };


    public int nextSeq() {
        // 调用get方法进行增加值
        seqCount.set(seqCount.get() +1);
        // 调用get方法获取自己的值。
        return seqCount.get();
    }


    public static class SeqThread extends Thread {

        // 获取到定义的ThreadLocal变量。
        private SeqCount seqCount;

        public SeqThread(SeqCount seqCount) {
            this.seqCount = seqCount;
        }
        @Override
        public void run() {
            for (int i=0; i<3; i++) {
                System.out.println(Thread.currentThread().getName()+" seqCount:"+seqCount.nextSeq());
            }
        }
    }


    public static void main(String [] args) {
        SeqCount seqCount = new SeqCount();

        // 开启四个线程，然后进行统计，但是调用的是外部类的变量。
        SeqThread seqThread1 = new SeqThread(seqCount);
        SeqThread seqThread2 = new SeqThread(seqCount);
        SeqThread seqThread3 = new SeqThread(seqCount);
        SeqThread seqThread4 = new SeqThread(seqCount);

        seqThread1.start();
        seqThread2.start();
        seqThread3.start();
        seqThread4.start();
    }
}

```

运行结果：

```shell
Thread-0 seqCount:1
Thread-3 seqCount:1
Thread-2 seqCount:1
Thread-1 seqCount:1
Thread-2 seqCount:2
Thread-3 seqCount:2
Thread-3 seqCount:3
Thread-0 seqCount:2
Thread-2 seqCount:3
Thread-1 seqCount:2
Thread-0 seqCount:3
Thread-1 seqCount:3
```

通过以上的运行结果可以看出，虽然只是定义了一个变量，但是每个变量进行操作对象的时候都不会互相干扰。达到了预期的目标。

从上面可看出，对于并发来说，为什么会需要锁？ **主要是因为需要对共享变量进行操作，而不是单纯的读取**。而我之前写的代码都是单单的读取，并没有涉及到多线程之间的操作竞争，所以没有考虑到这种情形，思路还是要转变一下。回归到对现场解决的问题，线程的同步的资源的竞争。其一就是控制线程的执行顺序，其二就是解决对于资源的操作（同步），如果是单单的读取的话，就不会涉及到操作，所以final关键字就是一个不会被**操作**只能读取的变量。

# 源码分析

## 常用api：

ThreadLocal定义了四个方法:

- get():返回此线程局部变量当前副本中的值
- set(T value):将线程局部变量当前副本中的值设置为指定值
- initialValue():返回此线程局部变量当前副本中的初始值
- remove():移除此线程局部变量当前副本中的值
- ThreadLocal还有一个特别重要的**静态内部类ThreadLocalMap**，该类才是实现线程隔离机制的关键。get()、set()、remove()都是基于该内部类进行操作，ThreadLocalMap用键值对方式存储每个线程变量的副本，key为当前的ThreadLocal对象，value为对应线程的变量副本。
   试想，每个线程都有自己的ThreadLocal对象，也就是都有自己的ThreadLocalMap，对自己的ThreadLocalMap操作，当然是互不影响的了，这就不存在线程安全问题了，所以ThreadLocal是以空间来交换安全性的解决思路。

主要就是以上的api，然后解决问题。

## get()

要获得当前线程私有的变量副本需要调用get()函数。首先，它会调用getMap()函数去获得当前线程的ThreadLocalMap，这个函数需要接收当前线程的实例作为参数。如果得到的ThreadLocalMap为null，那么就去调用setInitialValue()函数来进行初始化，如果不为null，就通过map来获得变量副本并返回。

setInitialValue()函数会去先调用initialValue()函数来生成初始值，该函数默认返回null，我们可以通过重写这个函数来返回我们想要在ThreadLocal中维护的变量。之后，去调用getMap()函数获得ThreadLocalMap，如果该map已经存在，那么就用新获得value去覆盖旧值，否则就调用createMap()函数来创建新的map。

```java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
	
/**
 * Variant of set() to establish initialValue. Used instead
 * of set() in case user has overridden the set() method.
 *
 * @return the initial value
 */
private T setInitialValue() {
    T value = initialValue();  // 调用重写方法
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value); // 当第一次进行调用的时候，创建的就是自己重写的传进来的值。
    return value;
}

// 也就是自己重写的值，
protected T initialValue() {
    return null;
}
```

## set(T value)

ThreadLocal的set()与remove()函数要比get()的实现还要简单，都只是通过getMap()来获得ThreadLocalMap然后对其进行操作。

```java
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread(); //获取当前线程。
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
/**
 * Removes the current thread's value for this thread-local
 * variable.  If this thread-local variable is subsequently
 * {@linkplain #get read} by the current thread, its value will be
 * reinitialized by invoking its {@link #initialValue} method,
 * unless its value is {@linkplain #set set} by the current thread
 * in the interim.  This may result in multiple invocations of the
 * {@code initialValue} method in the current thread.
 *
 * @since 1.5
 */
 public void remove() {
     ThreadLocalMap m = getMap(Thread.currentThread());
     if (m != null)
         m.remove(this);
 }   rehash();
}
```

## ThreadLocalMap内部类

ThreadLocal内部的数据存储是使用ThreadLocalMap内部类进行实现，并没有调用内部已经实现的HashMap。

ThreadLocalMap内部是利用Entry来进行key-value的存储的。

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

上面源码中key就是ThreadLocal，value就是值，Entry继承WeakReference，所以Entry对应key的引用（ThreadLocal实例）是一个弱引用。

### set(ThreadLocal key, Object value)

```java
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    //根据ThreadLocal的散列值，查找对应元素在数组中的位置
    int i = key.threadLocalHashCode & (len-1);
    //采用线性探测法寻找合适位置
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        //key存在，直接覆盖
        if (k == key) {
            e.value = value;
            return;
        }
        // key == null，但是存在值（因为此处的e != null），说明之前的ThreadLocal对象已经被回收了
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    //ThreadLocal对应的key实例不存在，new一个
    tab[i] = new Entry(key, value);
    int sz = ++size;
    //清楚陈旧的Entry(key == null的)
    // 如果没有清理陈旧的 Entry 并且数组中的元素大于了阈值，则进行 rehash
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}

```

这个set操作和集合Map解决散列冲突的方法不同，集合Map采用的是链地址法，这里采用的是开放定址法（线性探测）。set()方法中的replaceStaleEntry()和cleanSomeSlots()，这两个方法可以清除掉key ==null的实例，防止内存泄漏。

### getEntry()

```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
```

由于采用了开放定址法，当前keu的散列值和元素在数组中的索引并不是一一对应的，首先取一个猜测数（key的散列值），如果所对应的key是我们要找的元素，那么直接返回，否则调用getEntryAfterMiss

```java
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

这里一直在探测寻找下一个元素，知道找的元素的key是我们要找的。这里当key==null时，调用expungeStaleEntry有利于GC的回收，用于防止内存泄漏。

## ThreadLocal为什么会内存泄漏

ThreadLocalMap的key为ThreadLocal实例，他是一个弱引用，我们知道弱引用有利于GC的回收，当key == null时，GC就会回收这部分空间，但value不一定能被回收，因为他和Current Thread之间还存在一个强引用的关系。由于这个强引用的关系，会导致value无法回收，如果线程对象不消除这个强引用的关系，就可能会出现OOM。有些时候，我们调用ThreadLocalMap的remove()方法进行显式处理。

## 总结

- ThreadLocal不是用来解决共享变量的问题，也不是协调线程同步，他是为了方便各线程管理自己的状态而引用的一个机制。
- 每个ThreadLocal内部都有一个ThreadLocalMap,他保存的key是ThreadLocal的实例，他的值是当前线程的局部变量的副本的值。





