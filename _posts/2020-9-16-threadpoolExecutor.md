---
layout: post
title: Java：线程池源码解析
tags: java
---

> 学习线程池的操作逻辑。

#  目录
* 目录
{:toc}
![ThreadPoolExecutor.png](https://pic.tyzhang.top/images/2020/09/18/ThreadPoolExecutor.png)

# 线程池参数

线程池中的参数：

- 核心线程数：线程池中常驻的线程数，核心线程在启动以后就会一直运行，即使没有任务

- 最大线程数：当核心线程都被占用，阻塞队列也已经满的时候，就会新建额外的线程进行处理任务，但是新建的线程数要小于最大线程数

- 线程存活时间：当额外的线程没有任务时，线程保留的时间

- 存活时间单位：存活时间的单位

- 任务队列：当任务来不及处理的时候，就会被放进任务队列。

- 线程工厂： 创建线程的工厂

- 拒绝策略： 当线程已经达到了最大线程数且任务队列也已经满了，那么就会执行拒绝策略，默认是抛出异常。

其实也就是ThreadPoolExecutor的构造函数的参数。

# 继承关系

![ThreadPoolExtent.png](https://pic.tyzhang.top/images/2020/09/18/ThreadPoolExtent.png)

# Exexutors

一般来说都是使用ExecutorService接口进行操作线程池，但是具体的线程池代码是在ThreadPoolExecutor上进行执行。其中Executor源码定义了线程池的最主要的接口，如下：

```java
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    // 一般提交不带有返回值的都是 使用这个接口。
    void execute(Runnable command);
}
```

不管是提交Runable接口还是提交Callable接口，都会使用这个接口进行完成任务。

# ExecutorService

在线程池中，不管是定义的Callable还是Runable，Runable是最主要的。对于ExecutorService接口来说如下：它定义了线程池应该有的功能和接口和扩展了带有返回值的submit方法。

```java
public interface ExecutorService extends Executor {
    // 关闭操作、
    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long var1, TimeUnit var3) throws InterruptedException;
	
    // 提交一个带有返回值的数据
    <T> Future<T> submit(Callable<T> var1);

    <T> Future<T> submit(Runnable var1, T var2);

    Future<?> submit(Runnable var1);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> var1) throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> var1, long var2, TimeUnit var4) throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> var1) throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> var1, long var2, TimeUnit var4) throws InterruptedException, ExecutionException, TimeoutException;
}
```

对于AbstractExecutorService来说，主要是实现了一个submit方法，可以发现都是使用new FutureTask( )这个类进行实现的。

```java
public Future<?> submit(Runnable var1) {
    if (var1 == null) {
        throw new NullPointerException();
    } else {
        RunnableFuture var2 = this.newTaskFor(var1, (Object)null);
        this.execute(var2); // 可以发现，对于Future来说，也是使用execute()方法进行执行。。
        return var2;
    }
}

public <T> Future<T> submit(Runnable var1, T var2) {
    if (var1 == null) {
        throw new NullPointerException();
    } else {
        RunnableFuture var3 = this.newTaskFor(var1, var2);
        this.execute(var3);
        return var3;
    }
}

protected <T> RunnableFuture<T> newTaskFor(Runnable var1, T var2) {
    return new FutureTask(var1, var2); //FutureTask 是一个Future的具体实现类，那么是怎么
}

protected <T> RunnableFuture<T> newTaskFor(Callable<T> var1) {
    return new FutureTask(var1);
}
```

从上面可以发现，对于future任务来说，也是被execute进行执行，他被封装成了一个RunableFuture类型传递给execute方法中，

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```

这样就只需要封装一个execute方法就行，那么为什么可以通过一个execute() 方法执行Callable()接口的方法呢？ 主要是因为Callable接口的方法还是依赖于Run()接口的运行，当提交一个Callable接口的任务后，首先他会被封装成一个FutureTask()任务类，在FutureTask中，使用run方法调用call方法，并且FutureTask中会使用一个内部属性保存call方法的返回值，那么就可以使用Future中的get方法获取到这个属性的值，也符合我们的get set方法，只不过set方法是在FutureTask内部进行自动的设置，另外为什么调用Future会被阻塞？ 因为在获取的时候会判断当前的状态值，如果没有达到完成状态，那么就会调用cas将自己添加到等待队列，这样其实也就是解释了当初socket为什么会等待队列，以及是怎么实现的，直接定义一个Thread类型就可以获取一个Thread，很神奇。

```java
static final class WaitNode {
    volatile Thread thread;
    volatile WaitNode next;
    WaitNode() { thread = Thread.currentThread(); }
}
```

线程也好，其他也好都只是方法的调用，因为Thread类已经封装好了，也是一个具体的执行者与代码之间的关系，所以有时候理解起来比较抽象，但是具体底层也都是run方法的重载和互相调用。然后又会被封装成一个RunableFuture任务类，这里面有run()方法，

![CallableAndRunable.png](https://pic.tyzhang.top/images/2020/09/18/CallableAndRunable.png)

# ThreadPoolExecutor

## 一、属性知识

### 核心属性

```java
// AtomicInteger是原子类  ctlOf()返回值为RUNNING；
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
// 高3位表示线程状态
private static final int COUNT_BITS = Integer.SIZE - 3;  // = 29
// 低29位表示workerCount容量
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
// 能接收任务且能处理阻塞队列中的任务
private static final int RUNNING    = -1 << COUNT_BITS;
1110 0000 0000 0000 0000 0000 0000 0000  // 最改为的三位都是 111 表示为 RUNNING

// 不能接收新任务，但可以处理队列中的任务。
private static final int SHUTDOWN   =  0 << COUNT_BITS;
000
    
// 不接收新任务，不处理队列任务。
private static final int STOP       =  1 << COUNT_BITS;
001

// 所有任务都终止
private static final int TIDYING    =  2 << COUNT_BITS;
010

// 什么都不做
private static final int TERMINATED =  3 << COUNT_BITS;
011
    
// 存放任务的阻塞队列
private final BlockingQueue<Runnable> workQueue;

private final HashSet<Worker> workers = new HashSet<Worker>(); // worker集合。

private final ReentrantLock mainLock = new ReentrantLock(); // 为了进行同步使用hashset。 还有进行一些其他更改属性的操作需要。因为是使用cas所以不会阻塞进程。

private final Condition termination = mainLock.newCondition();
```

线程池状态变换：

| `RUNNING`    | `111-00000000000000000000000000000` | -536870912 | 运行中状态，可以接收新的任务和执行任务队列中的任务           |
| ------------ | ----------------------------------- | ---------- | ------------------------------------------------------------ |
| `SHUTDOWN`   | `000-00000000000000000000000000000` | 0          | shutdown状态，不再接收新的任务，但是会执行任务队列中的任务   |
| `STOP`       | `001-00000000000000000000000000000` | 536870912  | 停止状态，不再接收新的任务，也不会执行任务队列中的任务，中断所有执行中的任务 |
| `TIDYING`    | `010-00000000000000000000000000000` | 1073741824 | 整理中状态，所有任务已经终结，工作线程数为0，过渡到此状态的工作线程会调用钩子方法`terminated()` |
| `TERMINATED` | `011-00000000000000000000000000000` | 1610612736 | 终结状态，钩子方法`terminated()`执行完毕                     |

### 线程池状态变换图

![PoolStateConvert.png](https://pic.tyzhang.top/images/2020/09/18/PoolStateConvert.png)

### 构造函数

```java
public ThreadPoolExecutor(int corePoolSize,//线程池初始启动时线程的数量
                          int maximumPoolSize,//最大线程数量
                          long keepAliveTime,//空闲线程多久关闭?
                          TimeUnit unit,// 计时单位
                          BlockingQueue<Runnable> workQueue,//放任务的阻塞队列
                          ThreadFactory threadFactory,//线程工厂
                          RejectedExecutionHandler handler// 拒绝策略) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

线程池中的参数：

- 核心线程数：线程池中常驻的线程数，核心线程在启动以后就会一直运行，即使没有任务
- 最大线程数：当核心线程都被占用，阻塞队列也已经满的时候，就会新建额外的线程进行处理任务，但是新建的线程数要小于最大线程数
- 线程存活时间：当额外的线程没有任务时，线程保留的时间
- 存活时间单位：存活时间的单位
- 任务队列：当任务来不及处理的时候，就会被放进任务队列。
- 线程工厂： 创建线程的工厂
- 拒绝策略： 当线程已经达到了最大线程数且任务队列也已经满了，那么就会执行拒绝策略，默认是抛出异常。
  - `AbortPolicy`：直接拒绝策略，也就是不会执行任务，直接抛出`RejectedExecutionException`，这是**默认的拒绝策略**。
  - `DiscardPolicy`：抛弃策略，也就是直接忽略提交的任务（通俗来说就是空实现）。
  - `DiscardOldestPolicy`：抛弃最老任务策略，也就是通过`poll()`方法取出任务队列队头的任务抛弃，然后执行当前提交的任务。
  - `CallerRunsPolicy`：调用者执行策略，也就是当前调用`Executor#execute()`的线程直接调用任务`Runnable#run()`，**一般不希望任务丢失会选用这种策略，但从实际角度来看，原来的异步调用意图会退化为同步调用**。

构建`ThreadPoolExecutor`实例的时候，需要定义`maximumPoolSize`（线程池最大线程数）和`corePoolSize`（核心线程数）。当任务队列是有界的阻塞队列，核心线程满负载，任务队列已经满的情况下，会尝试创建额外的`maximumPoolSize - corePoolSize`个线程去执行新提交的任务。当`ThreadPoolExecutor`这里实现的两个主要附加功能是：

- 一定条件下会创建非核心线程去执行任务，非核心线程的回收周期（线程生命周期终结时刻）是`keepAliveTime`，线程生命周期终结的条件是：下一次通过任务队列获取任务的时候并且存活时间超过`keepAliveTime`。
- 提供拒绝策略，也就是在核心线程满负载、任务队列已满、非核心线程满负载的条件下会触发拒绝策略。

## 二、正常运行

### Execute()

execute()方法为整个线程池的核心方法，所有的Runable和Callable任务的都是由这个接口进行运行和实现。

```java
// 执行命令，其中命令（下面称任务）对象是Runnable的实例
public void execute(Runnable command) {
    // 判断命令（任务）对象非空
    if (command == null)
        throw new NullPointerException();
    // 获取ctl的值， 也就是状态码和记录工作线程个数的值。
    int c = ctl.get();
    // 判断如果当前工作线程数小于核心线程数，则创建新的核心线程并且执行传入的任务 workerCountOf(c) 
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            // 如果创建新的核心线程成功则直接返回
            return;
        // 这里说明创建核心线程失败，需要更新ctl的临时变量c
        c = ctl.get();
    }
    // 走到这里说明创建新的核心线程失败，也就是当前工作线程数大于等于corePoolSize
    // 判断线程池是否处于运行中状态，同时尝试用非阻塞方法向任务队列放入任务（放入任务失败返回false）
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 这里是向任务队列投放任务成功，对线程池的运行中状态做二次检查
        // 如果线程池二次检查状态是非运行中状态，则从任务队列移除当前的任务调用拒绝策略处理之（也就是移除前面成功入队的任务实例）
        if (! isRunning(recheck) && remove(command))
            // 调用拒绝策略处理任务 - 返回
            reject(command);
        // 走到下面的else if分支，说明有以下的前提：
        // 0、待执行的任务已经成功加入任务队列
        // 1、线程池可能是RUNNING状态
        // 2、传入的任务可能从任务队列中移除失败（移除失败的唯一可能就是任务已经被执行了）
        // 如果当前工作线程数量为0，则创建一个非核心线程并且传入的任务对象为null - 返回
        // 也就是创建的非核心线程不会马上运行，而是等待获取任务队列的任务去执行 
        // 如果前工作线程数量不为0，原来应该是最后的else分支，但是可以什么也不做，因为任务已经成功入队列，总会有合适的时机分配其他空闲线程去执行它
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 走到这里说明有以下的前提：
    // 0、线程池中的工作线程总数已经大于等于corePoolSize（简单来说就是核心线程已经全部懒创建完毕）
    // 1、线程池可能不是RUNNING状态
    // 2、线程池可能是RUNNING状态同时任务队列已经满了
    // 如果向任务队列投放任务失败，则会尝试创建非核心线程传入任务执行
    // 创建非核心线程失败，此时需要拒绝执行任务
    else if (!addWorker(command, false))
        // 调用拒绝策略处理任务 - 返回
        reject(command);
}
```

这里简单分析一下整个流程：

1. 如果当前工作线程总数小于`corePoolSize`，则直接创建核心线程执行任务（任务实例会传入直接用于构造工作线程实例）。
2. 如果当前工作线程总数大于等于`corePoolSize`，判断线程池是否处于运行中状态，同时尝试用非阻塞方法向任务队列放入任务，这里会二次检查线程池运行状态，如果当前工作线程数量为0，则创建一个非核心线程并且传入的任务对象为null。
3. 如果向任务队列投放任务失败（任务队列已经满了），则会尝试创建非核心线程传入任务实例执行。
4. 如果创建非核心线程失败，此时需要拒绝执行任务，调用拒绝策略处理任务。

**这里是一个疑惑点**：为什么需要二次检查线程池的运行状态，当前工作线程数量为0，尝试创建一个非核心线程并且传入的任务对象为null？这个可以看API注释：

> 如果一个任务成功加入任务队列，我们依然需要二次检查是否需要添加一个工作线程（因为所有存活的工作线程有可能在最后一次检查之后已经终结）或者执行当前方法的时候线程池是否已经shutdown了。所以我们需要二次检查线程池的状态，必须时把任务从任务队列中移除或者在没有可用的工作线程的前提下新建一个工作线程。

任务提交流程从调用者的角度来看如下：

![ExecuteFlow.png](https://pic.tyzhang.top/images/2020/09/18/ExecuteFlow.png)

### addWorker()

当核心线程不够的时候，需要进行添加工人线程。

```java
// 添加工作线程，如果返回false说明没有新创建工作线程，如果返回true说明创建和启动工作线程成功
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:  
    // 注意这是一个死循环 - 最外层循环
    for (int c = ctl.get();;) {
        // 这个是十分复杂的条件，这里先拆分多个与（&&）条件：
        // 1. 线程池状态至少为SHUTDOWN状态，也就是rs >= SHUTDOWN(0)
        // 2. 线程池状态至少为STOP状态，也就是rs >= STOP(1)，或者传入的任务实例firstTask不为null，或者任务队列为空
        // 其实这个判断的边界是线程池状态为shutdown状态下，不会再接受新的任务，在此前提下如果状态已经到了STOP、或者传入任务不为空、或者任务队列为空（已经没有积压任务）都不需要添加新的线程
        if (runStateAtLeast(c, SHUTDOWN)
            && (runStateAtLeast(c, STOP)
                || firstTask != null
                || workQueue.isEmpty()))
            return false;
        // 注意这也是一个死循环 - 二层循环
        for (;;) {
            // 这里每一轮循环都会重新获取工作线程数wc
            // 1. 如果传入的core为true，表示将要创建核心线程，通过wc和corePoolSize判断，如果wc >= corePoolSize，则返回false表示创建核心线程失败
            // 1. 如果传入的core为false，表示将要创非建核心线程，通过wc和maximumPoolSize判断，如果wc >= maximumPoolSize，则返回false表示创建非核心线程失败
            if (workerCountOf(c)
                >= ((core ? corePoolSize : maximumPoolSize) & COUNT_MASK))
                return false;
            // 成功通过CAS更新工作线程数wc，则break到最外层的循环
            if (compareAndIncrementWorkerCount(c))
                break retry;
            // 走到这里说明了通过CAS更新工作线程数wc失败，这个时候需要重新判断线程池的状态是否由RUNNING已经变为SHUTDOWN
            c = ctl.get();  // Re-read ctl
            // 如果线程池状态已经由RUNNING已经变为SHUTDOWN，则重新跳出到外层循环继续执行
            if (runStateAtLeast(c, SHUTDOWN))
                continue retry;
            // 如果线程池状态依然是RUNNING，CAS更新工作线程数wc失败说明有可能是并发更新导致的失败，则在内层循环重试即可 
            // else CAS failed due to workerCount change; retry inner loop 
        }
    }
    // 标记工作线程是否启动成功
    boolean workerStarted = false;
    // 标记工作线程是否创建成功
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 传入任务实例firstTask创建Worker实例，Worker构造里面会通过线程工厂创建新的Thread对象，所以下面可以直接操作Thread t = w.thread
        // 这一步Worker实例已经创建，但是没有加入工作线程集合或者启动它持有的线程Thread实例
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            // 这里需要全局加锁，因为会改变一些指标值和非线程安全的集合
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int c = ctl.get();
                // 这里主要在加锁的前提下判断ThreadFactory创建的线程是否存活或者判断获取锁成功之后线程池状态是否已经更变为SHUTDOWN
                // 1. 如果线程池状态依然为RUNNING，则只需要判断线程实例是否存活，需要添加到工作线程集合和启动新的Worker
                // 2. 如果线程池状态小于STOP，也就是RUNNING或者SHUTDOWN状态下，同时传入的任务实例firstTask为null，则需要添加到工作线程集合和启动新的Worker
                // 对于2，换言之，如果线程池处于SHUTDOWN状态下，同时传入的任务实例firstTask不为null，则不会添加到工作线程集合和启动新的Worker
                // 这一步其实有可能创建了新的Worker实例但是并不启动（临时对象，没有任何强引用），这种Worker有可能成功下一轮GC被收集的垃圾对象
                if (isRunning(c) ||
                    (runStateLessThan(c, STOP) && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    // 把创建的工作线程实例添加到工作线程集合
                    workers.add(w);
                    int s = workers.size();
                    // 尝试更新历史峰值工作线程数，也就是线程池峰值容量
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    // 这里更新工作线程是否启动成功标识为true，后面才会调用Thread#start()方法启动真实的线程实例
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            // 如果成功添加工作线程，则调用Worker内部的线程实例t的Thread#start()方法启动真实的线程实例
            if (workerAdded) {
                // 启动线程。
                t.start();
                // 标记线程启动成功
                workerStarted = true;
            }
        }
    } finally {
        // 线程启动失败，需要从工作线程集合移除对应的Worker
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}

// 添加Worker失败
private void addWorkerFailed(Worker w) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 从工作线程集合移除之
        if (w != null)
            workers.remove(w);
        // wc数量减1    
        decrementWorkerCount();
        // 基于状态判断尝试终结线程池
        tryTerminate();
    } finally {
        mainLock.unlock();
    }
}
```

笔者发现了`Doug Lea`大神十分喜欢复杂的条件判断，而且单行复杂判断不喜欢加花括号，像下面这种代码在他编写的很多类库中都比较常见：

```java
if (runStateAtLeast(c, SHUTDOWN)
    && (runStateAtLeast(c, STOP)
        || firstTask != null
        || workQueue.isEmpty()))
    return false;
// ....
//  代码拆分一下如下 
boolean atLeastShutdown = runStateAtLeast(c, SHUTDOWN);     # rs >= SHUTDOWN(0)
boolean atLeastStop = runStateAtLeast(c, STOP) || firstTask != null || workQueue.isEmpty();     
if (atLeastShutdown && atLeastStop){
   return false;
}
```

上面的分析逻辑中需要注意一点，`Worker`实例创建的同时，在其构造函数中会通过`ThreadFactory`创建一个Java线程`Thread`实例，后面会加锁后二次检查是否需要把`Worker`实例添加到工作线程集合`workers`中和是否需要启动`Worker`中持有的`Thread`实例，只有启动了`Thread`实例实例，`Worker`才真正开始运作，否则只是一个无用的临时对象。`Worker`本身也实现了`Runnable`接口，它可以看成是一个`Runnable`的适配器。

### Worker内部类

线程池中的每一个具体的工作线程被包装为内部类`Worker`实例，`Worker`继承于`AbstractQueuedSynchronizer(AQS)`，实现了`Runnable`接口：

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
    /**
        * This class will never be serialized, but we provide a
        * serialVersionUID to suppress a javac warning.
        */
    private static final long serialVersionUID = 6138294804551838833L;

    // 保存ThreadFactory创建的线程实例，如果ThreadFactory创建线程失败则为null
    final Thread thread;
    // 保存传入的Runnable任务实例
    Runnable firstTask;
    // 记录每个线程完成的任务总数
    volatile long completedTasks;
    
    // 唯一的构造函数，传入任务实例firstTask，注意可以为null
    Worker(Runnable firstTask) {
        // 禁止线程中断，直到runWorker()方法执行
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        // 通过ThreadFactory创建线程实例，注意一下Worker实例自身作为Runnable用于创建新的线程实例
        this.thread = getThreadFactory().newThread(this);
    }

    // 委托到外部的runWorker()方法，注意runWorker()方法是线程池的方法，而不是Worker的方法
    public void run() {
        runWorker(this);
    }

    // Lock methods
    //
    // The value 0 represents the unlocked state.
    // The value 1 represents the locked state.
    //  是否持有独占锁，state值为1的时候表示持有锁，state值为0的时候表示已经释放锁
    protected boolean isHeldExclusively() {
        return getState() != 0;
    }
	
    // 加锁为 1  解锁设置为0 
    
    // 独占模式下尝试获取资源，这里没有判断传入的变量，直接CAS判断0更新为1是否成功，成功则设置独占线程为当前线程
    protected boolean tryAcquire(int unused) {
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }
    
    // 独占模式下尝试是否资源，这里没有判断传入的变量，直接把state设置为0
    protected boolean tryRelease(int unused) {
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }
    
    // 加锁
    public void lock()        { acquire(1); } // 调用的也是tryacquire()

    // 尝试加锁
    public boolean tryLock()  { return tryAcquire(1); }

    // 解锁
    public void unlock()      { release(1); }

    // 是否锁定
    public boolean isLocked() { return isHeldExclusively(); }
    
    // 启动后进行线程中断，注意这里会判断线程实例的中断标志位是否为false，只有中断标志位为false才会中断
    void interruptIfStarted() {
        Thread t;
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}
```

`Worker`的构造函数里面的逻辑十分重要，通过`ThreadFactory`创建的`Thread`实例同时传入`Worker`实例，因为`Worker`本身实现了`Runnable`，所以可以作为任务提交到线程中执行。只要`Worker`持有的线程实例`w`调用`Thread#start()`方法就能在合适时机执行`Worker#run()`。简化一下逻辑如下：

```java
// addWorker()方法中构造
Worker worker = createWorker();
// 通过线程池构造时候传入
ThreadFactory threadFactory = getThreadFactory();
// Worker构造函数中
Thread thread = threadFactory.newThread(worker);
// addWorker()方法中启动
thread.start();
```

`Worker`继承自`AQS`，这里使用了`AQS`的独占模式，有个技巧是构造`Worker`的时候，把`AQS`的资源（状态）通过`setState(-1)`设置为-1，这是因为`Worker`实例刚创建时`AQS`中`state`的默认值为0，此时线程尚未启动，不能在这个时候进行线程中断，见`Worker#interruptIfStarted()`方法。`Worker`中两个覆盖`AQS`的方法`tryAcquire()`和`tryRelease()`都没有判断外部传入的变量，前者直接`CAS(0,1)`，后者直接`setState(0)`。

### RunWorker()

接着看核心方法`ThreadPoolExecutor#runWorker()`：

```java
final void runWorker(Worker w) {
    // 获取当前线程，实际上和Worker持有的线程实例是相同的
    Thread wt = Thread.currentThread();
    // 获取Worker中持有的初始化时传入的任务对象，这里注意存放在临时变量task中
    Runnable task = w.firstTask;
    // 设置Worker中持有的初始化时传入的任务对象为null
    w.firstTask = null;
    // 由于Worker初始化时AQS中state设置为-1，这里要先做一次解锁把state更新为0，允许线程中断
    w.unlock(); // allow interrupts
    // 记录线程是否因为用户异常终结，默认是true
    boolean completedAbruptly = true;
    try {
        // 初始化任务对象不为null，或者从任务队列获取任务不为空（从任务队列获取到的任务会更新到临时变量task中）
        // getTask()由于使用了阻塞队列，这个while循环如果命中后半段会处于阻塞或者超时阻塞状态，getTask()返回为null会导致线程跳出死循环使线程终结
        while (task != null || (task = getTask()) != null) {
            // Worker加锁，本质是AQS获取资源并且尝试CAS更新state由0更变为1
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            // 如果线程池正在停止（也就是由RUNNING或者SHUTDOWN状态向STOP状态变更），那么要确保当前工作线程是中断状态
            // 否则，要保证当前线程不是中断状态
            if ((runStateAtLeast(ctl.get(), STOP) ||
                    (Thread.interrupted() &&
                    runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                // 钩子方法，任务执行前
                beforeExecute(wt, task);
                try {
                    task.run();
                    // 钩子方法，任务执行后 - 正常情况
                    afterExecute(task, null);
                } catch (Throwable ex) {
                    // 钩子方法，任务执行后 - 异常情况
                    afterExecute(task, ex);
                    throw ex;
                }
            } finally {
                // 清空task临时变量，这个很重要，否则while会死循环执行同一个task
                task = null;
                // 累加Worker完成的任务数
                w.completedTasks++;
                // Worker解锁，本质是AQS释放资源，设置state为0
                w.unlock();
            }
        }
        // 走到这里说明某一次getTask()返回为null，线程正常退出
        completedAbruptly = false;
    } finally {
        // 处理线程退出，completedAbruptly为true说明由于用户异常导致线程非正常退出
        processWorkerExit(w, completedAbruptly);
    }
}
```

这里重点拆解分析一下判断当前工作线程中断状态的代码：

```java
if ((runStateAtLeast(ctl.get(), STOP) ||
        (Thread.interrupted() &&
        runStateAtLeast(ctl.get(), STOP))) &&
    !wt.isInterrupted())
    wt.interrupt();
// 先简化一下判断逻辑，如下
// 判断线程池状态是否至少为STOP，rs >= STOP(1)
boolean atLeastStop = runStateAtLeast(ctl.get(), STOP);
// 判断线程池状态是否至少为STOP，同时判断当前线程的中断状态并且清空当前线程的中断状态
boolean interruptedAndAtLeastStop = Thread.interrupted() && runStateAtLeast(ctl.get(), STOP);
if (atLeastStop || interruptedAndAtLeastStop && !wt.isInterrupted()){
    wt.interrupt();
}
```

`Thread.interrupted()`方法获取线程的中断状态同时会清空该中断状态，这里之所以会调用这个方法是因为在执行上面这个`if`逻辑同时外部有可能调用`shutdownNow()`方法，`shutdownNow()`方法中也存在中断所有`Worker`线程的逻辑，但是由于`shutdownNow()`方法中会遍历所有`Worker`做线程中断，有可能无法及时在任务提交到`Worker`执行之前进行中断，所以这个中断逻辑会在`Worker`内部执行，就是`if`代码块的逻辑。这里还要注意的是：`STOP`状态下会拒绝所有新提交的任务，不会再执行任务队列中的任务，同时会中断所有`Worker`线程。也就是，**即使任务Runnable已经`runWorker()`中前半段逻辑取出，只要还没走到调用其Runnable#run()，都有可能被中断**。假设刚好发生了进入`if`代码块的逻辑同时外部调用了`shutdownNow()`方法，那么`if`逻辑内会判断线程中断状态并且重置，那么`shutdownNow()`方法中调用的`interruptWorkers()`就不会因为中断状态判断出现问题导致二次中断线程（会导致异常）。

小结一下上面`runWorker()`方法的核心流程：

1. `Worker`先执行一次解锁操作，用于解除不可中断状态。
2. 通过`while`循环调用`getTask()`方法从任务队列中获取任务（当然，首轮循环也有可能是外部传入的firstTask任务实例）。
3. 如果线程池更变为`STOP`状态，则需要确保工作线程是中断状态并且进行中断处理，否则要保证工作线程必须不是中断状态。
4. 执行任务实例`Runnale#run()`方法，任务实例执行之前和之后（包括正常执行完毕和异常执行情况）分别会调用钩子方法`beforeExecute()`和`afterExecute()`。
5. `while`循环跳出意味着`runWorker()`方法结束和工作线程生命周期结束（`Worker#run()`生命周期完结），会调用`processWorkerExit()`处理工作线程退出的后续工作。

![RunWokerFlow.png](https://pic.tyzhang.top/images/2020/09/18/RunWokerFlow.png)

接下来分析一下从任务队列中获取任务的`getTask()`方法和处理线程退出的后续工作的`processWorkerExit()`方法。

### getTask()

`getTask()`方法是工作线程在`while`死循环中获取任务队列中的任务对象的方法：

```java
private Runnable getTask() {
    // 记录上一次从队列中拉取的时候是否超时
    boolean timedOut = false; // Did the last poll() time out?
    // 注意这是死循环
    for (;;) {
        int c = ctl.get();

        // Check if queue empty only if necessary.
        // 第一个if：如果线程池状态至少为SHUTDOWN，也就是rs >= SHUTDOWN(0)，则需要判断两种情况（或逻辑）：
        // 1. 线程池状态至少为STOP(1)，也就是线程池正在停止，一般是调用了shutdownNow()方法
        // 2. 任务队列为空
        // 如果在线程池至少为SHUTDOWN状态并且满足上面两个条件之一，则工作线程数wc减去1，然后直接返回null
        if (runStateAtLeast(c, SHUTDOWN)
            && (runStateAtLeast(c, STOP) || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }
        // 跑到这里说明线程池还处于RUNNING状态，重新获取一次工作线程数
        int wc = workerCountOf(c);

        // Are workers subject to culling?
        // timed临时变量勇于线程超时控制，决定是否需要通过poll()此带超时的非阻塞方法进行任务队列的任务拉取
        // 1.allowCoreThreadTimeOut默认值为false，如果设置为true，则允许核心线程也能通过poll()方法从任务队列中拉取任务
        // 2.工作线程数大于核心线程数的时候，说明线程池中创建了额外的非核心线程，这些非核心线程一定是通过poll()方法从任务队列中拉取任务
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
        // 第二个if：
        // 1.wc > maximumPoolSize说明当前的工作线程总数大于maximumPoolSize，说明了通过setMaximumPoolSize()方法减少了线程池容量
        // 或者 2.timed && timedOut说明了线程命中了超时控制并且上一轮循环通过poll()方法从任务队列中拉取任务为null
        // 并且 3. 工作线程总数大于1或者任务队列为空，则通过CAS把线程数减去1，同时返回null，
        // CAS把线程数减去1失败会进入下一轮循环做重试
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            // 如果timed为true，通过poll()方法做超时拉取，keepAliveTime时间内没有等待到有效的任务，则返回null
            // 如果timed为false，通过take()做阻塞拉取，会阻塞到有下一个有效的任务时候再返回（一般不会是null）
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            // 这里很重要，只有非null时候才返回，null的情况下会进入下一轮循环
            if (r != null)
                return r;
            // 跑到这里说明上一次从任务队列中获取到的任务为null，一般是workQueue.poll()方法超时返回null
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

这个方法中，有两处十分庞大的`if`逻辑，对于第一处`if`可能导致工作线程数减去1直接返回`null`的场景有：

1. 线程池状态为`SHUTDOWN`，一般是调用了`shutdown()`方法，并且任务队列为空。
2. 线程池状态为`STOP`。

对于第二处`if`，逻辑有点复杂，先拆解一下：

```java
// 工作线程总数大于maximumPoolSize，说明了通过setMaximumPoolSize()方法减少了线程池容量
boolean b1 = wc > maximumPoolSize;
// 允许线程超时同时上一轮通过poll()方法从任务队列中拉取任务为null
boolean b2 = timed && timedOut;
// 工作线程总数大于1
boolean b3 = wc > 1;
// 任务队列为空
boolean b4 = workQueue.isEmpty();
boolean r = (b1 || b2) && (b3 || b4);
if (r) {
    if (compareAndDecrementWorkerCount(c)){
        return null;
    }else{
        continue;
    }
}
```

这段逻辑大多数情况下是针对非核心线程。在`execute()`方法中，当线程池总数已经超过了`corePoolSize`并且还小于`maximumPoolSize`时，当任务队列已经满了的时候，会通过`addWorker(task,false)`添加非核心线程。而这里的逻辑恰好类似于`addWorker(task,false)`的反向操作，用于减少非核心线程，使得工作线程总数趋向于`corePoolSize`。如果对于非核心线程，上一轮循环获取任务对象为`null`，这一轮循环很容易满足`timed && timedOut`为true，这个时候`getTask()`返回null会导致`Worker#runWorker()`方法跳出死循环，之后执行`processWorkerExit()`方法处理后续工作，而该非核心线程对应的`Worker`则变成“游离对象”，等待被JVM回收。当`allowCoreThreadTimeOut`设置为true的时候，这里分析的非核心线程的生命周期终结逻辑同时会适用于核心线程。那么可以总结出`keepAliveTime`的意义：

- 当允许核心线程超时，也就是`allowCoreThreadTimeOut`设置为true的时候，此时`keepAliveTime`表示空闲的工作线程的存活周期。
- 默认情况下不允许核心线程超时，此时`keepAliveTime`表示空闲的非核心线程的存活周期。

在一些特定的场景下，配置合理的`keepAliveTime`能够更好地利用线程池的工作线程资源。

## 三、线程池关闭

### shutdown()

线程池关闭操作有几个相关的变体方法，先看`shutdown()`：

```java
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 权限校验，安全策略相关判断
        checkShutdownAccess();
        // 设置SHUTDOWN状态
        advanceRunState(SHUTDOWN);
        // 中断所有的空闲的工作线程
        interruptIdleWorkers();
        // 钩子方法
        onShutdown(); // hook for ScheduledThreadPoolExecutor
    } finally {
        mainLock.unlock();
    }
    // 调用上面分析果敢的尝试terminate方法，使状态更变为TIDYING，执行钩子方法terminated()后，最终状态更新为TERMINATED
    tryTerminate();
}

// 升提状态
private void advanceRunState(int targetState) {
    // assert targetState == SHUTDOWN || targetState == STOP;
    for (;;) {
        int c = ctl.get();
        // 线程池状态至少为targetState或者CAS设置状态为targetState则跳出循环
        if (runStateAtLeast(c, targetState) ||
            ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))
            break;
    }
}

// 中断所有的空闲的工作线程
private void interruptIdleWorkers() {
    interruptIdleWorkers(false);
}
```

shutdown只是将当前空闲的线程和还没有进入开始执行任务的线程记性中断。但是如果线程任务已经在执行，即已经调用了task.run()方法，那么不会将其停止。具体的逻辑可以看runWorker中在进行run执行进了一些状态的判断。

```java
if ((runStateAtLeast(ctl.get(), STOP) ||
        (Thread.interrupted() &&
        runStateAtLeast(ctl.get(), STOP))) &&
    !wt.isInterrupted()
   )
    wt.interrupt();  // Thread wt = Thread.currentThread();
```

### shutdownNow()

接着看`shutdownNow()`方法：

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 权限校验，安全策略相关判断
        checkShutdownAccess();
        // 设置STOP状态
        advanceRunState(STOP);
        // 中断所有的工作线程
        interruptWorkers();
        // 清空工作队列并且取出所有的未执行的任务
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
     // 调用上面分析果敢的尝试terminate方法，使状态更变为TIDYING，执行钩子方法terminated()后，最终状态更新为TERMINATED
    tryTerminate();
    return tasks;
}

// 遍历所有的工作线程，如果state > 0（启动状态）则进行中断
private void interruptWorkers() {
    // assert mainLock.isHeldByCurrentThread();
    for (Worker w : workers)
        w.interruptIfStarted();
}

// 调用worker中的中断方法， 相当于直接调用Thread的中断方法，会将线程停止下来。
void interruptIfStarted() {
    Thread t;
    if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
        try {
            t.interrupt();
        } catch (SecurityException ignore) {
        }
    }
}
```

`shutdownNow()`方法会把线程池状态先更变为`STOP`，中断所有的工作线程（`AbstractQueuedSynchronizer`的`state`值大于0的`Worker`实例，也就是包括正在执行任务的`Worker`和空闲的`Worker`），然后遍历任务队列，取出（移除）所有任务存放在一个列表中返回。

最后看`awaitTermination()`方法：

```java
public boolean awaitTermination(long timeout, TimeUnit unit)
    throws InterruptedException {
    // 转换timeout的单位为纳秒
    long nanos = unit.toNanos(timeout);
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 循环等待直到线程池状态更变为TERMINATED，每轮循环等待nanos纳秒
        while (runStateLessThan(ctl.get(), TERMINATED)) {
            if (nanos <= 0L)
                return false;
            nanos = termination.awaitNanos(nanos);
        }
        return true;
    } finally {
        mainLock.unlock();
    }
}
```

`awaitTermination()`虽然不是`shutdown()`方法体系，但是它的处理逻辑就是确保调用此方法的线程会阻塞到`tryTerminate()`方法成功把线程池状态更新为`TERMINATED`后再返回，可以使用在某些需要感知线程池终结时刻的场景。

有一点值得关注的是：`shutdown()`方法**只会中断空闲的工作线程**，如果工作线程正在执行任务对象`Runnable#run()`，这种情况下的工作线程不会中断，而是等待下一轮执行`getTask()`方法的时候通过线程池状态判断正常终结该工作线程。

### processWorkerExit()

`processWorkerExit()`方法是为将要终结的`Worker`做一次清理和数据记录工作（因为`processWorkerExit()`方法也包裹在`runWorker()`方法`finally`代码块中，其实工作线程在执行完`processWorkerExit()`方法才算真正的终结）。

```java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    // 因为抛出用户异常导致线程终结，直接使工作线程数减1即可
    // 如果没有任何异常抛出的情况下是通过getTask()返回null引导线程正常跳出runWorker()方法的while死循环从而正常终结，这种情况下，在getTask()中已经把线程数减1
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        decrementWorkerCount();

    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 全局的已完成任务记录数加上此将要终结的Worker中的已完成任务数
        completedTaskCount += w.completedTasks;
        // 工作线程集合中移除此将要终结的Worker
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }
     
    // 见下一小节分析，用于根据当前线程池的状态判断是否需要进行线程池terminate处理
    tryTerminate();

    int c = ctl.get();
    // 如果线程池的状态小于STOP，也就是处于RUNNING或者SHUTDOWN状态的前提下：
    // 1.如果线程不是由于抛出用户异常终结，如果允许核心线程超时，则保持线程池中至少存在一个工作线程
    // 2.如果线程由于抛出用户异常终结，或者当前工作线程数，那么直接添加一个新的非核心线程
    if (runStateLessThan(c, STOP)) {
        if (!completedAbruptly) {
            // 如果允许核心线程超时，最小值为0，否则为corePoolSize
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            // 如果最小值为0，同时任务队列不空，则更新最小值为1
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;
            // 工作线程数大于等于最小值，直接返回不新增非核心线程
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        addWorker(null, false);
    }
}
```

代码的后面部分区域，会判断线程池的状态，如果线程池是`RUNNING`或者`SHUTDOWN`状态的前提下，如果当前的工作线程由于抛出用户异常被终结，那么会新创建一个非核心线程。如果当前的工作线程并不是抛出用户异常被终结（正常情况下的终结），那么会这样处理：

- `allowCoreThreadTimeOut`为true，也就是允许核心线程超时的前提下，如果任务队列空，则会通过创建一个非核心线程保持线程池中至少有一个工作线程。
- `allowCoreThreadTimeOut`为false，如果工作线程总数大于`corePoolSize`则直接返回，否则创建一个非核心线程，也就是会趋向于保持线程池中的工作线程数量趋向于`corePoolSize`。

`processWorkerExit()`执行完毕之后，意味着该工作线程的生命周期已经完结。

### tryTerminate()

每个工作线程终结的时候都会调用`tryTerminate()`方法：

```java
final void tryTerminate() {
    for (;;) {
        int c = ctl.get();
        // 判断线程池的状态，如果是下面三种情况下的任意一种则直接返回：
        // 1.线程池处于RUNNING状态
        // 2.线程池至少为TIDYING状态，也就是TIDYING或者TERMINATED状态，意味着已经走到了下面的步骤，线程池即将终结
        // 3.线程池至少为STOP状态并且任务队列不为空
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateLessThan(c, STOP) && ! workQueue.isEmpty()))
            return;
        // 工作线程数不为0，则中断工作线程集合中的第一个空闲的工作线程
        if (workerCountOf(c) != 0) { // Eligible to terminate
            interruptIdleWorkers(ONLY_ONE);
            return;
        }

        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            // CAS设置线程池状态为TIDYING，如果设置成功则执行钩子方法terminated()
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated();
                } finally {
                    // 最后更新线程池状态为TERMINATED
                    ctl.set(ctlOf(TERMINATED, 0));
                    // 唤醒阻塞在termination条件的所有线程，这个变量的await()方法在awaitTermination()中调用
                    termination.signalAll();
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
    }
}

// 中断空闲的工作线程，onlyOne为true的时候，只会中断工作线程集合中的某一个线程
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers) {
            Thread t = w.thread;
            // 这里判断线程不是中断状态并且尝试获取锁成功的时候才进行线程中断
            if (!t.isInterrupted() && w.tryLock()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();
                }
            }
            // 这里跳出循环，也就是只中断集合中第一个工作线程
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock();
    }
}
```

这里有疑惑的地方是`tryTerminate()`方法的第二个`if`代码逻辑：工作线程数不为0，则中断工作线程集合中的第一个空闲的工作线程。方法API注释中有这样一段话：

> If otherwise eligible to terminate but workerCount is nonzero, interrupts an idle worker to ensure that shutdown signals propagate.
> 当满足终结线程池的条件但是工作线程数不为0，这个时候需要中断一个空闲的工作线程去确保线程池关闭的信号得以传播。

下面将会分析的`shutdown()`方法中会通过`interruptIdleWorkers()`中断所有的空闲线程，这个时候有可能有非空闲的线程在执行某个任务，执行任务完毕之后，如果它刚好是核心线程，就会在下一轮循环阻塞在任务队列的`take()`方法，如果不做额外的干预，它甚至会在线程池关闭之后永久阻塞在任务队列的`take()`方法中。为了避免这种情况，每个工作线程退出的时候都会尝试中断工作线程集合中的某一个空闲的线程，确保所有空闲的线程都能够正常退出。

`interruptIdleWorkers()`方法中会对每一个工作线程先进行`tryLock()`判断，只有返回`true`才有可能进行线程中断。我们知道`runWorker()`方法中，工作线程在每次从任务队列中获取到非null的任务之后，会先进行加锁`Worker#lock()`操作，这样就能避免线程在执行任务的过程中被中断，保证被中断的一定是空闲的工作线程。

## reject()

`reject(Runnable command)`方法很简单：

```java
final void reject(Runnable command) {
    handler.rejectedExecution(command, this);
}
```

调用线程池持有的成员`RejectedExecutionHandler`实例回调任务实例和当前线程池实例。

## 钩子方法分析

到JDK11为止，`ThreadPoolExecutor`提供的钩子方法没有增加，有以下几个：

- `beforeExecute(Thread t, Runnable r)`：任务对象`Runnable#run()`执行之前触发回调。
- `afterExecute(Runnable r, Throwable t)`：任务对象`Runnable#run()`执行之后（包括异常完成情况和正常完成情况）触发回调。
- `terminated()`：线程池关闭的时候，状态更变为`TIDYING`成功之后会回调此方法，执行此方法完毕后，线程池状态会更新为`TERMINATED`。
- `onShutdown()`：`shutdown()`方法执行时候会回调此方法，API注释中提到此方法主要提供给`ScheduledThreadPoolExecutor`使用。

其中`onShutdown()`的方法修饰符为`default`，其他三个方法的修饰符为`protected`，必要时候可以自行扩展这些方法，可以实现监控、基于特定时机触发具体操作等等。

## 其他方法

线程池本身提供了大量数据统计相关的方法、扩容方法、预创建方法等等，这些方法的源码并不复杂，这里不做展开分析。

**核心线程相关：**

- `getCorePoolSize()`：获取核心线程数。
- `setCorePoolSize()`：重新设置线程池的核心线程数。
- `prestartCoreThread()`：预启动一个核心线程，当且仅当工作线程数量小于核心线程数量。
- `prestartAllCoreThreads()`：预启动所有核心线程。

**线程池容量相关：**

- `getMaximumPoolSize()`：获取线程池容量。
- `setMaximumPoolSize()`：重新设置线程池的最大容量。

**线程存活周期相关：**

- `setKeepAliveTime()`：设置空闲工作线程的存活周期。
- `getKeepAliveTime()`：获取空闲工作线程的存活周期。

**其他监控统计相关方法：**

- `getTaskCount()`：获取所有已经被执行的任务总数的近似值。
- `getCompletedTaskCount()`：获取所有已经执行完成的任务总数的近似值。
- `getLargestPoolSize()`：获取线程池的峰值线程数（最大池容量）。
- `getActiveCount()`：获取所有活跃线程总数（正在执行任务的工作线程）的近似值。
- `getPoolSize()`：获取工作线程集合的容量（当前线程池中的总工作线程数）。

**任务队列操作相关方法：**

- `purge()`：移除任务队列中所有是`Future`类型并且已经处于`Cancelled`状态的任务。
- `remove()`：从任务队列中移除指定的任务。
- `BlockingQueue<Runnable> getQueue()`：获取任务队列的引用。

## 小结

总的来说线程池就是以上的运行流程，其中所有的worker 都被存储在HashSet中，因为HashSet不是线程安全的，当需要添加worker或者是删除worker的时候，都需要进行申请ThreadPoolExecutor中的内部类锁`mainLock`，这个是一个ReentrantLock类型的锁，调用的场景也就限定在更新内部属性的时候，防止多线程竞争，需要知道的是，只有操作线程池对象的线程才能修改线程池的属性，比如调用shutdown()方法，还有就是当主线程去调用ExecutorService.execute()方法时候发现核心线程没有被创建的时候，向worker中添加线程。

为什么在进行runworker之前会进行unlock()方法？ 主要是因为线程池在进行创建线程的时候不能被中断，此时他的state被设置为-1，所有在运行线程的时候，需要将其state更改为0，表示此时线程是正常的。而且线程在进行质心任务的时候，还会对线程加锁，其实也不是加锁，只是设置一下state的状态，让其他线程知道当前线程在尽心执行任务。

参考：[JUC线程池ThreadPoolExecutor源码分析](https://www.throwable.club/2019/07/15/java-concurrency-thread-pool-executor/)