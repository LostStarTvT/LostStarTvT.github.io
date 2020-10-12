---
layout: post
title: Java：垃圾收集算法总结
tags: java

---

> 总结JVM垃圾回收算法。

## [什么时候才会触发Full GC？](https://www.zhihu.com/question/41922036/answer/93079526)

针对HotSpot VM的实现，它里面的GC其实准确分类只有两大种：

- Partial GC：并不收集整个GC堆的模式

- - Young GC：只收集young gen的GC
  - Old GC：只收集old gen的GC。**只有CMS的concurrent collection是这个模式**
  - Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式

- Full GC：收集整个堆，包括young gen、old gen、perm gen（如果存在的话）等所有部分的模式。

那么换句话说，其他垃圾收集器并没有单单针对于老年代的GC 算法，都是在Full GC的时候对整个堆进行垃圾回收，同时使用标记整理算法对老年代进行垃圾回收。下面说的是HotSpot中的垃圾回收算法，只被分为两种回收，young gc 和full gc。

最简单的分代式GC策略，按HotSpot VM的serial GC的实现来看，触发条件是：

- young GC：当young gen中的eden区分配满的时候触发。注意young GC中有部分存活对象会晋升到old gen，所以young GC后old gen的占用量通常会有所升高。
- full GC：当准备要触发一次young GC时，如果发现统计数据说之前young GC的平均晋升大小比目前old gen剩余的空间大，则不会触发young GC而是转为触发full GC（因为HotSpot VM的GC里，除了CMS的concurrent collection之外，其它能收集old gen的GC都会同时收集整个GC堆，包括young gen，所以不需要事先触发一次单独的young GC）；或者，如果有perm gen的话，要在perm gen分配空间但已经没有足够空间时，也要触发一次full GC；或者System.gc()、heap dump带GC，默认也是触发full GC。

- 新生代GC： Minor GC
- 老年代GC：Major GC
- 整个Java堆： Full GC



# 并行和并发

并行（Parallel）： 并行描述的是多条垃圾收集器线程之间的关系，说明同一时间有多条这样的线程在协同工作，通常默认此时用户是处于等待状态。

并发（concurrent）：并发描述的是垃圾收集器线程与用户线程之间的关系，说明同一时间垃圾收集线程与用户线程都在运行，由于用户线程并没有被冻结，所以程序仍然能够响应服务请求，大师由于垃圾收集器线程占用了一部分系统资源，此时应用程序的处理吞吐量将受到一定的影响。

# 内存分配规则

minor GC 新生代中的GC， major GC 老年代GC，其实也就是和full  GC差不多的意思。

### 对象优先在Eden区分配

大多数情况下，对象在新生代Eden区中分配，当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC

### 大对象直接进入老年代

大对象就是指需要大量连续内存空间的java对象，最典型的大对象就是那种很长的字符串，或者元素数量很庞大的数组，例如byte[]数组就是典型的大对象。

HotSpot虚拟机提供-XX：PretenureSizeThreshold参数，指定大于该设置值的对象直接在老年代分配，这样做的目的就是避免在Eden区一级两个Survivor区之间来回复制，产生大量的内存复制操作。

### 长期存活的对象进入老年代

这个就是一直记得的年龄问题，当年龄超过15岁以后，便会将对象转移到老年代。年龄阈值可通过参数 `-XX:MaxTenuringThreshold = 指定值` 来设定。

#### 对对象年龄判定的优化

JVM中，如果**Survivor区**中的**相同年龄**的**所有对象的大小**总和大于**Survivor空间**的一半，那么这些同窗们就直接进入老年代。无须等到上面的年龄阈值。

首先介绍一下 `Mnior GC` 和 `Full GC` 的区别:

> MniorGC : 发生在新生代的垃圾回收活动，由于新生代的Java对象 短命的特性，这种垃圾回收活动频繁，回收速度较快。(就像扫碎纸屑) Full GC : 发生在老年的垃圾回收活动，不过出现一次 Full GC,也会伴随着一次 MniorGC,由于老年代中的对象基本都是大对象，长命，所以Full GC的速度比Mnior GC 的速度慢10倍以上。(就像搬大石头)

新生代的垃圾回收算法采用的是 [复制算法](https://www.yuque.com/henudada/kir653/hl39eo#vxPl5) ,当进行一次 `Mnior GC` 时，会将新生代的活动区域( `Eden区` 和Survivor中的 `From区` )中的存活对象复制到 Survivor中的 `to区` ,如果 to 区的内存不足以放下这些对象，那么这时就需要老年代出马，进行分配担保机制,将放不下的对象放到老年代。所以，在进行 `Monior GC` 前，JVM会做以下流程的检查，以确认老年代是否能够放得下那些对象，来选择进行 `Mnior GC` 还是 `Full GC` 。

![spaceGC.png](https://pic.tyzhang.top/images/2020/09/24/spaceGC.png)

在发生Minor GC之前，虚拟机必须检查老年代的最大可用的连续空间是否大于新生所有对象总空间，如果这个条件成立，那么这一次的Minor GC是可以确保安全的，如果不成立，则虚拟机会先查看 -XX:HandlerPromotionFailure 参数的设置值是否允许担保失败，如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升老年代对象的平均大小，如果大于，将尝试进行一次Minor GC，尽管这次Minor GC是有风险的，如果小于，或者-XX:HandlePromotionFailure设置不允许冒险，那这时候高=要改为进行一次Full GC。

为什么会冒险？因为最极端的情况就是一个Minor GC后，对象都存活，所以这时候就需要使用老年代进行担保，把survivor区无法容纳的对象直接放到老年代。因为minor GC是将  e区的对象和s1区的对象标记复制到s0区，如果s0区放不下，那么就直接放在老年代。

参考：

- [JVM内存分配机制与回收策略选择-JVM学习笔记](https://juejin.im/post/6844903873841201165)

# 并发可达性分析理论

如果是STW状态下，进行可达性分析时比较简单，只需遍历就好，但是在并发可达性标记的时候就会出现一些问题，如标记过程中引用更改等问题。

三色标记：

- 白色：表示对象尚未被垃圾收集器访问过。分析结束白色表示不可达。
- 黑色：表示对象已经被垃圾收集器访问过。并且这个对象的所有引用都已经被扫描过。
- 灰色：表示对象已经被垃圾收集器访问过，但这个对象上至少存在一个引用还没有被扫描过。

对象消失问题条件：

- 赋值器插入了一条或多条从黑色对象到白色对象的新引用
- 赋值器删除了全部从灰色对象到该白色对象的直接或者间接引用。

类比于死锁的条件，解决对象消失问题，打破其中一个条件就行，于是出现了两种方案：

- 增量更新：破坏第一个条件，**当黑色对象插入指向白色对象的引用关系时**，就将这个要删除的引用记录下来，在扫描完成后，在将这些记录过的引用关系中的灰色对象为根，重新扫描，重新扫描需要STW，也就是CMS的做法
  - 简化理解： 黑色对象一段新插入了执向白色对象的引用之后，它就变回灰色对象了
- 原始快照：破坏第二个条件。**当灰色对象要删除指向白色对象的引用关系时**，就将这个要删除的引用记录下来，在并发扫描结束后，将这些记录过的引用关系中的灰色对象为根，在扫描一下。G1的做法。
  - 讲话理解：无论引用关系删除与否，都会按照刚刚开始扫描的那一刻的对象图快照进行搜索。

其实也就是必须扫描一个不变的东西，通过日志记录更新。 

# 垃圾回收器总览

![gcSum.png](https://pic.tyzhang.top/images/2020/09/29/gcSum.png)

# 垃圾回收器

## Serial回收器：串行回收

- Serial收集器是Hotspot中Client模式下的默认新生代垃圾收集器。
- 采用复制算法、串行回收和Stop-the-world机制的方式执行内存的回收。
- 处理老年代以外，Serial收集器还提供了用于执行老年代垃圾收集的Serial Old 收集器。Serial Old 收集器同样也采用了创兴回收和STW机制，只不过内存回收算法使用的是标记整理算法。
  - Serial Old 是运行在Client模式下默认的老年代垃圾回收器
  - Serial Old 是在server模式下主要的两个用途：
    - 与新生代的Parallel Scavenge配合使用。
    - 作为老年代CMS收集器的后备垃圾收集方案。

![Serial_SerialOldGC.png](https://pic.tyzhang.top/images/2020/09/23/Serial_SerialOldGC.png)

从上图可以看出，Serial回收器是一个串行的回收器，他的单线程的意义并不仅仅说明它只会使用一个CPU或一条收集线程去完成垃圾收集工作，更重要的是它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集完成。

**优点：**

1. 简单而高效（与其他的收集器的单线程相比），对于限定在单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。
   1. 运行在Client模式下的虚拟机是个不错的选择。
2. 在用户的桌面应用场景中，可用内存一般不大(几十MB至一百MB)，可以在较短时间内完成垃圾收集（几十ms至100多ms），只要不频繁的发生，使用串行回收器是可以接受的。
3. 在HotSpot虚拟机中，使用 -XX:UseSerialGC参数可以指定年轻代和老年代都使用串行收集器。即新生代使用Serial GC 且老年代使用Serial Old GC

## ParNew回收器：并行回收

- 其中Par是parallel并行的意思，new 表示在新生代回收。可以简单的认为ParNew是Serial的多线程版本。
- ParNew收集器处理采用不能够细啊回收的方式执行内存回收外，Serial 与ParNew二者几乎没有任何的区别。
- ParNew是很多JVM运行在Server模式下新生代的默认垃圾收集器。

![ParNew_SerialOldGC.png](https://pic.tyzhang.top/images/2020/09/23/ParNew_SerialOldGC.png)

对应上图而言：

- 对于新生代，回收次数频繁，使用并行的方法比较高效。
- 对于老年代，回收的次数少，使用串行的方式节省资源（CPU并行需要切换线程，串行可以省去切换线程的资源）

但是ParNew只能在多CPU的环境下比Serial快，因为可以利用多CPU的特点。但是在单CPU的环境下，Serial更快。

另外，**目前除了Serial只有ParNew GC能够与CMS配合工作**。

## Parallel Scavenge回收器：吞吐量优先

Scavenge 打扫的意思。PS收集器。

吞吐量=程序运行时间/（程序运行时间+GC时间）

HotSpot的年轻代处理拥有ParNew收集器是基于并行回收以外，Parallel Scavenge收集器同样也是采用复制算法，并行回收和STW机制。

那么Parallel收集器的出现是否就是多此一举呢？

- 和ParNew不同，Parallel Scavenge收集器的目标是达到一个可控制的吞吐量(Throughput),他也被称之为吞吐量优先的GC
- 自适应调节策略也是Parallel Scavenge与ParNew的一个重要区别。
- **他不能与CMS配合使用。**

## Parallel Old回收器

parallel old 收集器采用标记-压缩算法，但同样也是基于并行回收和STW机制。他是在JDk1.6时提供了用于执行老年代的垃圾收集器，用来代替Serial Old收集器。它能够提供高吞吐量，而高吞吐量可以高效的利用CPU时间，尽快完成程序的运行任务，主要是和在后台运算而不需要太多交互的任务，因此，常见在服务器环境中使用，例如执行批量处理、订单处理、工资支付、科学计算的应用程序。所以常用在服务器上。

![ParallelScavenge_ParallelOldGC.png](https://pic.tyzhang.top/images/2020/09/23/ParallelScavenge_ParallelOldGC.png)

上图是双并行的垃圾收集器，适合在程序吞吐量优先的应用场景，在Server模式下的内存回收性能很不错。

## CMS回收器：低延迟

这个也是一个并行的垃圾收集器。但是主打的是低延迟，即尽量所短STW的时间，实现并行的垃圾回收。

在JDK 1.5时期，HotSpot推出了一款在强交互应用中几乎可以认为有跨时代的垃圾收集器，CMS(Concurrent Mark Sweep)收集器，这款收集器是HotSpot虚拟机中第一款真正意义上的并发收集器，它是第一次实现了让垃圾收集线程与用户线程同时工作。

CMS收集器其的关注点是尽可能的缩短垃圾收集时用户线程的停顿时间，停顿时间越短（低延迟）就越适合与用户交互的程序中，良好的相应速度能提升用户体验。

- 目前很大一部分的Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤其重视服务的相应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS收集器就非常符合这类应用的需求，

CMS的垃圾收集算法采用的是标记-清除算法，sweep 就是清除的意思，mark是标记。并且有会出现STW。

CMS主要有四个部分

1. 初始标记(CMS initial Mark) **STW**
2. 并发标记(CMS concurrent mark)
3. 重新标记(CMS remark) **STW**
4. 并发清除(CMS concurrent sweep)

![CMS.png](https://pic.tyzhang.top/images/2020/09/23/CMS.png)

### 工作原理：

**初始标记阶段：**：在这个阶段，程序中所有的工作线程都将会因为STW机制而出现短暂的暂停，这个阶段的主要任务仅仅只是标记处GC Root能直接关联到的对象，即第一层直接相连的对象。一旦标记完成之后就会恢复到之前被暂停的所有应用线程，由于关联的对象比较小，所以这里的速度非常的快。

**并发标记阶段**：从GC Roots的直接关联对象开始遍历整个对象图的过程，这个耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行。

**重新标记阶段**： 由于在并发标记阶段中，程序的工作线程会和垃圾收集线程同时运行或者交叉运行，因此为了修正并发标记期间，因为用户程序继续运作而导致标记产生变动那一部分对象的标记记录，则个阶段的停顿时间通常会比初始标记阶段稍微长一些，但也远比并发标记阶段的时间短。

**并发清除阶段**：此阶段清楚删除掉标记阶段判断的已经死亡的对象，释放内存空间，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的。

### 总结:

尽管CMS收集器采用的是并发回收，非独占式，但是在其初始化标记和再次标记这两个阶段仍然需要执行STW机制暂停程序中的工作线程，不过暂停时间不会太长，因此可以说明目前所有的垃圾收集器都做不到完全不需要STW，只是尽可能的所短暂停时间。

由于最耗费时间的并发标记与并发清楚阶段都不需要暂停工作，所以整体的回收是低停顿的。另外由于在垃圾收集阶段用户线程没有中断，所有在CMS回收过程中，还应该确保应用程序用户线程有足够的内存可用。因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，而是当堆内存使用率达到某一阈值时，便开始进行回收，以确保应用程序在CMS工作过程中依然有足够的控件支持应用程序运行，**要是CMS运行期间预留的内存无法满足程序需要，就会出现一次Concurrent Mode Failure 失败，这时虚拟机将启动后备预案，临时启动Serial Old收集器来重新进行老年代的垃圾收集，这时候停顿时间就很长了。**其实也就是启动Serial Old 进行Full  GC，这也就是一个不足。

CMS收集器的垃圾收集算法采用的是标记-清除算法，这意味着每次执行完内存回收后，由于被执行内存回收的无用对象所占用的内存空间极有可能是不连续的一些内存块，不可避免的将会产生一些内存碎片，那么CMS在为新对象分配内存空间时，将无法使用指针碰撞技术（标记-整理算法），而只能选择空闲列表（标记清除算法）执行内存分配。

### 为什么不把CMS的标记清除算法Mark Sweep 换成标记压缩 Mark Compact算法？

因为并发清除的时候，用Compact整理内存的话，原来的用户线程使用的内存还怎么用？要保证用户线程能继续执行，前提它运行的资源不能受影响，所以只能直接的将内存清除。

### CMS的优点

- 并发收集
- 低延迟

### CMS的缺点

- 会产生内存碎片。导致并发清除后，用户线程可用的空间不足，在无法分配大对象的情况下，不得不提前触发Full GC
- CMS收集器对CPU资源非常敏感，在并发阶段，它虽然不会导致用户停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。
- CMS收集器无法处理浮动垃圾。可能出现Concurrent Mode Failure 失败而导致另一次Full GC的产生，在并发标记阶段由于程序的工作线程和垃圾收集线程是同时运行或者交叉运行的，那么在并发标记阶段如果产生新的垃圾对象，CMS无法对这些垃圾对象进行标记，最终会导致这些新产生的垃圾对象没有被及时回收，从而只能在下次执行GC的时候释放这些之前未被回收的内存空间。

 参考链接：

- [CMS垃圾收集器](https://juejin.im/post/6844903782107578382)
- [CMS收集器](https://www.cnblogs.com/webor2006/p/11055468.html)
- 

# 垃圾回收器总结比较

|        收集器         | 串行、并行or并发 | 新生代/老年代 |        算法        |     目标     |                 适用场景                  |
| :-------------------: | :--------------: | :-----------: | :----------------: | :----------: | :---------------------------------------: |
|      **Serial**       |       串行       |    新生代     |      复制算法      | 响应速度优先 |          单CPU环境下的Client模式          |
|    **Serial Old**     |       串行       |    老年代     |     标记-整理      | 响应速度优先 |  单CPU环境下的Client模式、CMS的后备预案   |
|      **ParNew**       |       并行       |    新生代     |      复制算法      | 响应速度优先 |    多CPU环境时在Server模式下与CMS配合     |
| **Parallel Scavenge** |       并行       |    新生代     |      复制算法      |  吞吐量优先  |     在后台运算而不需要太多交互的任务      |
|   **Parallel Old**    |       并行       |    老年代     |     标记-整理      |  吞吐量优先  |     在后台运算而不需要太多交互的任务      |
|        **CMS**        |       并发       |    老年代     |     标记-清除      | 响应速度优先 | 集中在互联网站或B/S系统服务端上的Java应用 |
|        **G1**         |       并发       |     both      | 标记-整理+复制算法 | 响应速度优先 |        面向服务端应用，将来替换CMS        |

总结：

如果想要最小化使用内存和并行开销，请选择Serial GC；

如果你想要最大化应用程序吞吐量，请选择Parallel GC；

如果想要最小化GC中断或停顿时间，请选择CMS GC；