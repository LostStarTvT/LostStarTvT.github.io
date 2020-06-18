---
layout: post
title: Java: 并发编程之美
tags: java  
---


> 读《java并发之美》的一些记录，捋一捋之间的关系。

#  目录
* 目录
{:toc}
# 导读

对于java并发来说，使用高并发可以更好的利用计算机资源，充分发挥计算机的性能，但是“人多事杂“，线程也是一样，多线程就需要控制线程之间的运行关系与资源的竞争，主要包括两个问题：

1. 高并发下如何控制资源的同步与共享？
2. 高并发下如何实现线程运行顺序控制？

对于问题1，因为高并发下会出现一个资源多个线程去访问，所有就出现竞争问题，此问题是使用锁进行处理。  

对于问题2，有些任务需要多个线程去配合完成，配合便有了依赖，此时就需要使用信号量去进行线程之间的同步。整个高并发的设计都是围绕这两个问题来进行阐述。

## 锁的概念

对于锁来说，可以大致分为独享锁和共享锁，其中独享锁思想就是悲观锁，共享锁的思想就是乐观锁。

- 悲观锁：在我干事情的时候别人都不能进行干涉，即在进行操作共享资源的时候全程都要加锁。（一刀切）
- 乐观锁：认为只有在更改共享资源的时候才需要加锁，而进行读取资源的时候不需要加锁，相当于更加的细化。（因地制宜）

对于乐观锁来说，在进行加锁的时候时候是和悲观锁使用的思想是一致的，但是实现的手段是不一样的。

加锁方式：

- 悲观锁：在自己进行操作的时候，直接将其他线程进行阻塞。
- 乐观锁：在自己进行操作的时候使用循环来一直判断有没有更改成功，使用一个类似于版本控制的方式，在更改时判断更改的结果与自己预期的结果是否一致，如果一致则更改成功，否则使用死循环再次尝试，直到更改成功，也就说使用cpu资源代替效率。

# 锁的实现

java中主要有两种方式实现锁，一种是使用jvm提供的**synchronize关键字**，一种就是使用**JUC**(java.util.concurrent，java并发工具包)中的ReentrantLock进行加锁。

其中synchronize实现的就是悲观锁，即将一些非原子性操作加锁编程原子性操作，另外通过对象中的wait()和notify() notifyAll()关键字可以实现线程之间的同步。主要的原理就是利用jvm中的内存屏障来实现。另外，jvm还提供一个volatile关键字作为轻量级的锁，主要作为资源的同步。 

对于JUC来说，其底层使用的是AQS(abstractQueuedSynchronizer)进行的实现，AQS主要就是调用UnSafe工具类进行对线程的操作，其中UnSafe是java提供的一个基于c++实现线程操作的库，通过park()和unpark()方法实现线程的阻塞和挂起，从而实现独享锁。另外，Unsafe还提供了CAS(compareAndSet，比较并设置)实操作，即cas(expect, update)，其中expect是期望update以后成为的值，update为更新需要操作的值，例如，操作1+x，如果update = 2，则expect = 1+2 = 3。即AQS通过使用循环进行CAS操作实现共享锁。

基于AQS实现的工具类：

- 锁的实现
  - ReentrantLock: 独享锁
  - ReentantReadWriteLock: 对于写进行独享锁，对于读进行共享锁
  - StampedLock: 提供独享写锁，独享读锁和共享读锁
- 并发队列
  - ConcurrentLinkedQueue ：基于单链表实现的非阻塞(基于CAS实现的乐观锁)队列。共享锁
  - LinkedBlockingQueue：基于单链表实现的阻塞(使用Reentrantlock锁实现)队列。独享锁
  - ArrayBlockingQueue：基于有界数组实现的阻塞(使用ReentrantLock锁实现)队列。独享锁
  - PriorityBlockingQueue：基于数组实现的阻塞(使用ReentrantLock锁实现)队列，使用堆算法平衡二叉树进行实现优先级。独享锁和共享锁
  - DelayQueue：无界阻塞(使用ReentrantLock锁实现)队列，使用PriorityQueue存放数据。队列中每个元素都有个过期时间，每次只有过期元素才能出队。
- 线程池
  - ThreadPoolExecutor ：
  - ScheduledThreadPoolExexutor
- 信号量控制
  - CountDownLatch ：状态量是递减的
  - CyclicBarrier： 状态量是可以重复利用的，回环的形式
  - Semaphore： 状态量是递增的

# AQS介绍

## 锁的实现

JUC下的并发控制都是基于AQS进行实现，这个也就是java通过C++库(UnSafe,本地方法库对应jvm中的本地方法栈)进行封装实现的线程操作，仅仅从代码的角度去实现，而不是类似synchronize和volatile关键字从虚拟机的角度去实现锁和并发的控制。在AQS中所有的线程都被封装成node节点。

**AQS中实现线程阻塞的方式：**   

对于AQS来说主要就是使用一个队列来进行线程的控制，即将被park()函数阻塞的线程放入阻塞队列AQS阻塞队列，然后等待被unpark()函数激活，主要使用的是rt.jar包中的LockSupport工具类进行实现。

**AQS中公平锁与非公平锁：**    

另外还实现了公平锁与非公平锁，其中公平锁来说，就是严格按照队列中的线程依次进行唤醒执行，而对于非公平锁来说，即先尝试获取锁的线程并不一定比后尝试获取到锁的线程优先获取到锁。 一个例子：假设线程A先尝试获取锁，然后失败被阻塞到AQS阻塞队列中，然后又有一个线程B来尝试获取锁，结果刚好锁被释放，然后线程B就获取到了锁，而A仍然在等待，此时的锁就是非公平的。 在AQS中实现公平锁的做法，当线程B尝试获取锁的时候，会检查AQS阻塞队列中是否有被阻塞的线程，如果有则把锁给队头的线程，新来的线程则被阻塞。

**AQS中如何判断锁已经被获取？**  

在AQS中维护了一个单一状态信息state，可以通过getState、setState、compareAndSetState函数来修改值。每个线程通过读取这个值来判断此时锁是否已经被获取。比如state=0，表示锁可以被获取，如果state=1表示已经被占用，对于可冲入锁则是简单的state = state +1。在释放锁的时候在进行减1。

**AQS独享锁与共享锁的实现方式？ ** 

在获取锁的时候主要有六个方式

- 独享锁
  - void acquire(int arg)
  - void acquireInterruptibly(int arg)
  - boolean release(int arg)
- 共享锁
  - void acquireShared(int arg)
  - void acquireSharedInterruptibly(int arg)
  - boolean releaseShared(int arg)

从上面可以看出，加上shared表示为共享锁，而不加的表示为独享锁。以独享锁的acquire()方法为例，当调用此方法的时候，会调用tryAcquire()方法，其他5个也是类似，在AQS框架中，tryXXX()方法是不提供的，即需要子类去提供对应的方法实现，以上六个方法所进行的操作为在调用的时候会尝试调用子类的tryXXX()方法，如果tryXXX失败，则会将自己添加到AQS阻塞队列，否则获取成功。其中阻塞队列中会标记阻塞的线程为node.EXCLUSIVE（独享）还是node.SHARED（共享）的。

其中的共享锁和独享锁的实现思想需要在子类的tryXXX方法中进行体现。

就以ReentrantLock独享锁的tryAcquire为例，这种只会尝试获取一次，如果失败则被阻塞，等待被release方法唤醒。而对于ReentrantReadWriteLock()中的读锁来说，是基于共享锁实现的，写锁是独享锁实现，那么读锁在进行获取时，会进行自旋获取，即在获取的时候检测有没有独享锁已经被获取，如果有则阻塞自己，没有的话就进行自旋获取，即使用死循环一致尝试获取，并且在死循环的过程中还有检测有没有写锁干扰，有的话也会阻塞自己。读锁使用自旋获取的原因就是读操作一般都是很快的，所以可以多尝试几次便能获取到。  

**锁是如何进行锁住线程的呢？**

以下面的代码为例，他是如何识别本线程而被阻塞的呢？

```java
ReentrantLock lock = new ReentrantLock();

lock.lock();
//自己的代码
lock.unlock();
```

对于lock.lock()方法来说，底层调用的就是tryAcquire方法，即尝试获取锁，那么锁是如何与线程绑定的呢？对于Thread来说，有个方法`Thread current = Thread.currentThread`，此时便能够获取到调用锁的线程是哪个线程，然后调用Unsafe方法便能够实现对象的唤醒与操作。

## 线程同步的实现

以上只是解决了资源共享的问题，对于第二个问题线程执行顺序来说，就需要使用ConditionObject来进行实现。

AQS中维护一个ConditionObject对象来维护线程之间的同步问题如下图所示：其中对于一个lock来说，可以有n个Condition，

![AQS.png](https://pic.tyzhang.top/images/2020/06/18/AQS.png)

```java
ReentrantLock lock = new ReentrantLock();
Condition condition = lock.newConditon()

lock.lock();
    //自己的代码
	condition.await();
lock.unlock();

lock.lock();
    //自己的代码
	condition.singal();
lock.unlock();
```

以上就是实现了一个线程同步的控制，其实也类似于synchronize中的object.wait()  object.singal()方法，所不同的是当condition的调用必须在lock、unlock之间进行调用，另外调用await的线程，会调用park函数将自己阻塞并且将自己添加到上图中的条件队列中，当有线程调用singal后，此时会将该线程移动到AQS阻塞队列中等待获取锁。如果调用signalAll则会一次性把所有的条件队列中的线程转移到AQS阻塞队列中等待获取锁。锁也就是state的值。