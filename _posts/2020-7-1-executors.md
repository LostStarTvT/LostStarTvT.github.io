---
layout: post
title: Java：自定义线程池
tags: java  
---


> 总结线程池的作用，并且实现一个简单的自定义线程池。

#  目录
* 目录
{:toc}
# 前言

线程池主要解决两个问题：

1.  当执行大量异步任务时线程池能够提供较好的性能。 
2. 线程池提供了一种资源限制和管理手段，比如可以限制线程的个数，动态的增加线程等等。

对于一，类似于通过雇佣几个人来执行一起干工作，通过初始化几个线程然后轮流的进行处理数据，处理任务等。

线程池的状态：

1. `RUNNING`:接受新任务并且处理阻塞队列里的任务。
2. `SHUTDOWN`:拒绝新任务但是处理阻塞队列里的任务。
3. `STOP`:拒绝新任务并且抛弃阻塞队列里的任务，同时会中断正在处理的任务。
4. `TIDYING`:所有的任务都执行完了（包括阻塞队列里面的任务）后当期那线程池活动线程数为0，将要调用terminated方法。
5. `TERMINAYTED`:终止状态，terminated方法调用完成以后的状态。

线程池状态转换：

- `RUNNING ->SHUTDOWN`:显示的调用shutdown方法，或者隐式的调用了finalize()方法里面的shutdown()方法。
- `RUNNING`或`SHUTDOWN->STOP`:显示的调用了shutdownNow()方法。
- `SHUTDOWN->TIDYING`:当线程池和任务队列都为空时。
- `STOP->TIDYING`:当线程池为空时。
- `TIDYING->TERMINAYTED`:当terminated() hook方法执行完成时。

Executors提供的线程池类型：以下主要是是通过配置Executors参数实现不同的线程池类型。

1. `newFixedThreadPool(int nThread)` 创建有n个Thread的线程池。
2. `newSingleThreadExecutor() `创建具有一个线程的线程池。
3. `newCachedThreadPool() `创建一个按需的线程池。

# 线程池参数

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 || maximumPoolSize <= 0 || maximumPoolSize < corePoolSize || keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ? null : AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

## corePoolSize

线程池核心线程数量，核心线程不会被回收，即使没有任务执行，也会保持空闲状态。如果线程池中的线程少于此数目，则在执行任务时创建。

## maximumPoolSize

池允许最大的线程数，当线程数量达到corePoolSize，且workQueue队列塞满任务了之后，继续创建线程。

## keepAliveTime

超过corePoolSize之后的“临时线程”的存活时间。

## unit

keepAliveTime的单位。

## workQueue

当前线程数超过corePoolSize时，新的任务会处在等待状态，并存在workQueue中，BlockingQueue是一个先进先出的阻塞式队列实现，底层实现会涉及Java并发的AQS机制，有关于AQS的相关知识，我会单独写一篇，敬请期待。

## threadFactory

创建线程的工厂类，通常我们会自定一个threadFactory设置线程的名称，这样我们就可以知道线程是由哪个工厂类创建的，可以快速定位。

## handler

线程池执行拒绝策略，当线数量达到maximumPoolSize大小，并且workQueue也已经塞满了任务的情况下，线程池会调用handler拒绝策略来处理请求。

系统默认的拒绝策略有以下几种：

1. AbortPolicy：为线程池默认的拒绝策略，该策略直接抛异常处理。
2. DiscardPolicy：直接抛弃不处理。
3. DiscardOldestPolicy：丢弃队列中最老的任务。
4. CallerRunsPolicy：将任务分配给当前执行execute方法线程来处理。

我们还可以自定义拒绝策略，只需要实现RejectedExecutionHandler接口即可，友好的拒绝策略实现有如下：

1. 将数据保存到数据，待系统空闲时再进行处理
2. 将数据用日志进行记录，后由人工处理

# 自定义线程池

在进行自定义线程池的时候需要先明白两件事，就是需要定义一个阻塞队列，因为线程池是解决大量任务，所以需要一个“缓冲区”即阻塞队列来进行接客，即排队等待区，另外还需要定义几个技师来进行处理客人的请求，其中请求也就是用户传进来的实现了Runable接口的类。

## 任务缓冲区

所以首先定义自定义等待队列：使用使用一个队列加锁实现，而不是使用JUC中的阻塞队列。

```java
package multiRun.Pool;

import java.util.ArrayDeque;
import java.util.Deque;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class BlockingQueue<T> {
    // 任务队列  双向队列，具有栈的队列的性质。
    private Deque<T> queue = new ArrayDeque<>();
    // 锁
    private ReentrantLock lock = new ReentrantLock();
    // 生产者条件变量，添加线程，满的时候等待
    private Condition fullWaitSet = lock.newCondition();
    // 消费者条件变量，执行线程，空的时候等待
    private Condition emptyWaitSet = lock.newCondition();
    // 容量
    private int capcity;

    public BlockingQueue(int capcity) {
        this.capcity = capcity;
    }
    // 阻塞获取
    public T task() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                try {
                    emptyWaitSet.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeFirst();
            fullWaitSet.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }
    // 带超时的阻塞获取
    public T poll(long timeout, TimeUnit unit) {
        lock.lock();
        try {
            long nanos = unit.toNanos(timeout);
            while (queue.isEmpty()) {
                try {
                    if (nanos <= 0) {
                        return null;
                    }
                    nanos = emptyWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeFirst();
            return t;
        } finally {
            lock.unlock();
        }
    }
    // 阻塞添加
    public void put(T task) {
        lock.lock();
        try {
            while (queue.size() == capcity) {
                try {
                    fullWaitSet.await();
                    System.out.println("等待任务加入队列{}" + task);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("加入任务队列{}" + task);
            queue.addLast(task);
            emptyWaitSet.signal();
        } finally {
            lock.unlock();
        }
    }
    // 带超时的阻塞添加
    public boolean offer(T task, long timeout, TimeUnit timeUnit) {
        lock.lock();
        try {
            long nanos = timeUnit.toNanos(timeout);
            while (queue.size() == capcity) {
                try {
                    if (nanos <= 0) {
                        return false;
                    }

                    System.out.println("等待加入队列{}" + task);
                    nanos = fullWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            System.out.println("加入任务队列{}" + task);
            queue.addLast(task);
            emptyWaitSet.signal();
            return true;
        } finally {
            lock.unlock();
        }
    }
    // 返回队列长度
    public int size() {
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }

    // 带拒绝策略的添加
    public void tryPut(RejectPolicy<T> rejectPolicy, T task) {
        lock.lock();
        try {
            if (queue.size() == capcity) {
                rejectPolicy.reject(this, task);
            } else {
                System.out.println("加入任务队列{}" + task);
                queue.addLast(task);
                emptyWaitSet.signal();
            }
        } finally {
            lock.unlock();
        }
    }
}
```

上面的缓冲区主要提供了增加和取出的接口。添加接口如下：

- put(T task): 阻塞添加
- offer(T task, long timeout, TimeUnit timeUnit) ：带有超时的添加

取出接口：

-  public T task()：阻塞获取
- public T poll(long timeout, TimeUnit unit) ：带有超时的获取

通过线程池的实现对象向缓冲区添加待完成的任务，即主线程添加， 然后n个工作线程去获取获取缓冲区的任务。

## 拒绝策略

```java
@FunctionalInterface
public interface RejectPolicy<T> {
    void reject(BlockingQueue<T> queue, T task);
}
```

## 线程池

线程池类主要就是提供一个execute(Runnable task)接口进行接受任务。需要传递一个实现Runable接口的对象，然后让然后线程池进行运行，相当于不用定义一个Thread对象了。

```java
package multiRun.Pool;

import java.util.HashSet;
import java.util.concurrent.TimeUnit;

public class ThreadPool {
    // 任务队列  自定义的任务队列， 实现加锁。
    private BlockingQueue<Runnable> taskQueue;
    // 线程集合  HashSet 保证集合里面的数据都不不同的，即没有相同的数据，是一个数组类型的，通过Hash值保证值的不相同。
    private HashSet<Worker> workers = new HashSet<>();
    // 核心线程数
    private int coreSize;
    // 获取任务时的超时时间
    private long timeout;
    private TimeUnit timeUnit;
    private RejectPolicy<Runnable> rejectPolicy;

    public ThreadPool(int coreSize, long timeout, TimeUnit timeUnit, RejectPolicy<Runnable> rejectPolicy, int queueCapcity) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.rejectPolicy = rejectPolicy;
        this.taskQueue = new BlockingQueue<>(queueCapcity);
    }

    // 执行任务
    public void execute(Runnable task) {
        synchronized (workers) {
            // 进行创建worker线程。
            if (workers.size() < coreSize) {

                // 将传递过来的任务传递给工人。
                Worker worker = new Worker(task);

                System.out.println("新增worker{}"+ worker);
                workers.add(worker);

                // worker 是一个继承Thread的类，所有可以调用start方法。
                worker.start();
            } else {
                // 如果worker都被用了，那么就尝试放进等待队列。 如果队列满了还可能去阻塞这些队列。 使用主线程去添加任务到队列
                taskQueue.tryPut(rejectPolicy, task);
            }
        }
    }


    //自己定义的worker内部类。  一个worker其实就是一个线程，然后通过调用传递过来的task.run方法进行执行对应的方法。
    class Worker extends Thread {
        private Runnable task;
        public Worker(Runnable task) {
            this.task = task;
        }
        @Override
        public void run() {
            // 当task不为空，执行任务；当task执行完毕，从任务队列中获取任务并执行
            // 因为可能多个work去队列中申请执行任务，所有是要设计成为阻塞队列。
            // 如果当前有任务并且taskQueue中还有没有执行的任务，则就去申请没有执行完的任务去执行，指导所有任务执行完整。
            while (task != null || (task = taskQueue.poll(timeout, timeUnit)) != null) {
                try {
                    System.out.println("正在执行{}" + task);
                    // 为什么会调用接口的方法？ 其实相当于重写，因为这个方法线程肯定会执行，用户在外面虽然不能直接的待用这个方法，但是内部的程序是可以调用
                    //这个方法进行实现功能的。。
                    task.run();

                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    task = null;
                }
            }

            // 所有的任务执行完以后便被remove掉。
            synchronized (workers) {
                System.out.println("worker被移除{}" + this);
                workers.remove(this);
            }
        }
    }
}
```

从上面可以看出，对应于上面线程池中的shutdown，当所有的任务完成以后，便会将所有的线程进行删除，即开除服务员，然后结束线程池，从这个角度来说，线程池也就是Thread的封装工具类，帮你批量打开Thread，然后还能循环的使用。循环的调用的方法就是在worker中进行观察。

```java
while (task != null || (task = taskQueue.poll(timeout, timeUnit)) != null) {
    try {
        System.out.println("正在执行{}" + task);
        // 为什么会调用接口的方法？ 其实相当于重写，因为这个方法线程肯定会执行，用户在外面虽然不能直接的待用这个方法，但是内部的程序是可以调用
        //这个方法进行实现功能的。。
        task.run();

    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        task = null;
    }
}
```

其中在run中调用task.run() 相当于硬核的重载，即嵌套调用实现动态的绑定任务。其中死循环的条件就是当前任务还有或者是缓冲区中还有任务则worker还有继续工作。继续调用下一个task.run()方法。通过以上的方法就实现了多个线程的复用。

还有个问题就是，为什么需要将队列设置为阻塞队列？

因为有多个worker会去竞争获取队列中的尚未完成的任务。而在放入任务到队列中则是由主线程去完成的。即

` taskQueue.tryPut(rejectPolicy, task);`去实现，这个是在execute()方法进行调用的时候会进行，此时如果队列，满的话主线程也会被阻塞。。

## 测试使用

其实从一个角度来说，线程池也就是对以下的方法进行了封装。 

```java
public class MyRunable implements Runable{
	@Override
    public void run(){
    //...
    }
}

public static void main(String [] args){
	MyRunable instance = new MyRunable();
    Thread  th = new Thread(instand);
    th.start();
}
```

线程池测试使用。

```java
package multiRun.Pool;

import java.util.concurrent.TimeUnit;

// 实现自定义线程池进行测试。
public class Demo {
    public static void main(String[] args) {
		
        //初始化自定义的线程池。
        ThreadPool threadPool = new ThreadPool(1,
                1000, TimeUnit.MILLISECONDS, (queue, task) -> {
            // 1. 死等
            // queue.put(task);
            // 2) 带超时等待
            queue.offer(task, 1500, TimeUnit.MILLISECONDS);
            // 3) 让调用者放弃任务执行
            // log.debug("放弃{}", task);
            // 4) 让调用者抛出异常
            // throw new RuntimeException("任务执行失败 " + task);
            // 5) 让调用者自己执行任务
            // task.run();
        }, 1);
        
        // 添加进去4个任务，然后执行。
        for (int i = 0; i < 4; i++) {
            int j = i;
            // 这时候执行的任务就是进行睡眠。
            threadPool.execute(() -> {
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("{}" + j);
            });
        }
    }
}

```

运行结果：

```shell
新增worker{}Thread[Thread-0,5,main]
加入任务队列{}multiRun.Pool.Demo$$Lambda$2/558638686@7699a589
等待加入队列{}multiRun.Pool.Demo$$Lambda$2/558638686@58372a00
正在执行{}multiRun.Pool.Demo$$Lambda$2/558638686@2ebdbbab
{}0
正在执行{}multiRun.Pool.Demo$$Lambda$2/558638686@7699a589
加入任务队列{}multiRun.Pool.Demo$$Lambda$2/558638686@58372a00
等待加入队列{}multiRun.Pool.Demo$$Lambda$2/558638686@6d03e736
{}1
正在执行{}multiRun.Pool.Demo$$Lambda$2/558638686@58372a00
加入任务队列{}multiRun.Pool.Demo$$Lambda$2/558638686@6d03e736
{}2
正在执行{}multiRun.Pool.Demo$$Lambda$2/558638686@6d03e736
{}3
worker被移除{}Thread[Thread-0,5,main]
```

**通过以上的线程复用解决了批量任务时频繁开启线程的开销，通过复用能够大大的提升程序的执行效率，另外当所有的任务执行完成时候线程worker也会被摧毁，重新调用的时候也会重新创建新的线程worker，**之前我还以为是新建的时候直接创建，然后等待任务的分配，其实这样也是一个懒加载的实现。即用的时候给你创建，但是以后再用就不用创建了。然后都用完了还自动的删除掉。达到节约资源的目的。

# 线程池配置相关

线程池大小的设置:  

这其实是一个面试的考点，很多面试官会问你线程池coreSize 的大小来考察你对于线程池的理解。  
首先针对于这个问题，我们必须要明确我们的需求是计算密集型还是IO密集型，只有了解了这一点，我们才能更好的去设置线程池的数量进行限制。  

1、计算密集型：  
顾名思义就是应用需要非常多的CPU计算资源，在多核CPU时代，我们要让每一个CPU核心都参与计算，将CPU的性能充分利用起来，这样才算是没有浪费服务器配置，如果在非常好的服务器配置上还运行着单线程程序那将是多么重大的浪费。对于计算密集型的应用，完全是靠CPU的核数来工作，所以为了让它的优势完全发挥出来，避免过多的线程上下文切换，比较理想方案是：  

线程数 = CPU核数+1，也可以设置成CPU核数*2，但还要看JDK的版本以及CPU配置(服务器的CPU有超线程)。   

一般设置CPU * 2即可。  

2、IO密集型  
我们现在做的开发 大部分都是WEB应用，涉及到大量的网络传输，不仅如此，与数据库，与缓存间的交互也涉及到IO，一旦发生IO，线程就会处于等待状态，当IO结束，数据准备好后，线程才会继续执行。因此从这里可以发现，对于IO密集型的应用，我们可以多设置一些线程池中线程的数量，这样就能让在等待IO的这段时间内，线程可以去做其它事，提高并发处理效率。那么这个线程池的数据量是不是可以随便设置呢？当然不是的，请一定要记得，线程上下文切换是有代价的。目前总结了一套公式，对于IO密集型应用：
线程数 = CPU核心数/(1-阻塞系数) 这个阻塞系数一般为0.8~0.9之间，也可以取0.8或者0.9。  
套用公式，对于双核CPU来说，它比较理想的线程数就是20，当然这都不是绝对的，需要根据实际情况以及实际业务来调整：final int poolSize = (int)(cpuCore/(1-0.9))    

# 钩子函数

线程池提供了几个钩子函数

```java
protected void beforeExecute(Thread t, Runnable r) { } // 任务执行前
protected void afterExecute(Runnable r, Throwable t) { } // 任务执行后
protected void terminated() { } // 线程池执行结束后
```

# Callable返回值

定义带有返回值的类。以下是实现接受多个线程的返回值的情况。

```java

public class cable implements Callable<String> {//定义一个类实现Callable<V>接口
    private  int a,b;
    public cable(int a,int b) {
        this.a=a;
        this.b=b;
    }

    @Override
    public String call() throws Exception {
        System.out.println(a+b+" "+Thread.currentThread().getName());
        Thread.sleep(1000);
        return "没错，我执行完了上面的代码，还告诉你我完事了。";
    }
}
```

```java

public class App {

    public App() throws InterruptedException, ExecutionException {
        //创建ExecutorService线程池
        ExecutorService threadPool = Executors.newSingleThreadExecutor();
        //创建存储Future对象的集合，用来存放ExecutorService的执行结果
         ArrayList<Future<String>> future = new ArrayList<Future<String>>();
        //举例子：开3个线程，将返回的Future对象放入集合中
        future.add(threadPool.submit(new cable(1,2)));
        future.add(threadPool.submit(new cable(4,5)));
        future.add(threadPool.submit(new cable(7,8)));
        
        for (Future<String> fs : future) {
            //判断线程是否执行结束，如果执行结束就将结果打印
            if (fs.isDone()) {
                System.out.println("22222"+fs.get());
            } else {
                System.out.println("44444"+fs.toString());
            }
        }
        //关闭线程池，不再接收新的线程，未执行完的线程不会被关闭
        threadPool.shutdown();
        System.out.println("main方法执行结束");
    }
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        new App();
    }
}
```



