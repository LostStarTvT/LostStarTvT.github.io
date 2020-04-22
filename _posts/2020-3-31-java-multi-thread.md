---
layout: post
title: Java：多线程
tags: java  
---


> 看java多线程学习笔记记录，总结多线程知识，但是视频好像找不到了..

##  目录
* 目录
{:toc}
### 0.说明

学习java多线程笔记总结，留以备用。

### 1.两种线程启动方式

1，实现Runable接口方法。  

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

2,继承实现Thread类。  

```java
public class Mythread extends Thread{
	public void run (){
    	//...
    }
}

public static void main(String [] args){
	Mythread mth = new mythread();
    mth.start();
}
```

3,匿名类实现方式  

```java
new Thread(new Runnable() {
    @Override
    public void run() {

    }
})
```

以上三种实现的方式主要是要借助于Thread类来进行承载，实现接口的方式也是为了进行接口回调，因为核心还是要用Thread对象来进行承载。  

### 2.Synchronized关键字

synchronized关键字为同步，即为加锁，为什么要加锁？因为多个引用进行操作一个对线时，需要等待才能保证数据的正确性。或者说对一个对象进行一个操作时，需要保持其原子单一性，即某一时刻只能有一个引用进行操作。那么多线程是怎么实现更快的呢？ 除掉等待必须原子性操作的时间，其他时间可以并发的执行，这样就实现了并发，即合作干事。  

![bingfa.png](https://pic.tyzhang.top/images/2020/03/31/bingfa.png)

这个线程图也就是表示，其实线程并不是会同时进行执行。而是顺序等待进行执行， 主要是因为其中的黄线部分是原子性操作，所以必须进行等待，但是即使是如此也是效率更加的高。 如果是单线程的话，就是一个等一个，直接累加。  

类比于一个现实的问题，就是服务器并发访问，相当于一个web对象需要多个线程同时引用，为了数据的一致性，就需要进行加锁，而加锁的目标是实例化的对象。  

一个简单的多线程累减demo。  

```java
package offerDay2;

//开5个线程去进行减10  这样就是一个一对多的例子， 正常情况下是 10 9 8 7 6  但是可能出现 7 7 7 6 9
//出现的原因是 因为在输出的时候，读取的是一个公共的值，那么如果特别慢的话，就会出现 在进行打印的时候，其他线程已经更改值，致使
//值不准确。解决的方法就是加锁。synchronized
public class T implements Runnable {

    private int count = 10;
    @Override
    public /*synchronized */ void run() {
        count --;
        System.out.println(Thread.currentThread().getName() + "count=  " + count);
    }

    public static void main(String[] args){
        T t = new T();
        for (int i = 0; i < 5; i++) {
            new Thread(t,"Thread + " + i).start();
        }
    }
}

```

### 3.volitile关键字

volatile  /ˈvɒlətaɪl/ /  含义：挥发性的不稳定的。  
这个关键字可以将线程中的变量共享，因为线程上的值是复制到cpu缓存中，如果不加的话，在另开一个线程进行更改这个变量值的时候，cpu缓存中的值是没有改的，而加了volatile之后，其他线程在更改了此变量的值时，会通知所有引用此东西的线程，说内存中的值已经更改，需要重新读取。  

涉及到cpu与内存的缓存之间的数据同步问题。 这个就是确定cpu一定会刷新缓存的值，其他情况也可能会读取。比如sleep ，此种做法也就是让cpu缓存常回家“内存”看看。  

volatile 会强制所有线程去堆内存中去读取数据，而不是单纯的读取cpu缓存的值。  

参考文档：  

[全面理解java内存模型](https://blog.csdn.net/suifeng3051/article/details/52611310)  

[volatile和synchronized的区别](https://blog.csdn.net/suifeng3051/article/details/52611233)  

### 4.死锁是怎么产生的？

死锁定义：“死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。”换一个更加规范的定义：“集合中的每一个进程都在等待只能由本集合中的其他进程才能引发的事件，那么该组进程是死锁的。”

#### 举个例子

如果此时有个线程A，按照先锁a在获得锁b的顺序获得锁，而在此时又有另外一个线程B，按照先锁b在锁a的顺序获得锁，如下图所示：  

![lock.png](https://pic.tyzhang.top/images/2020/03/31/lock.png)

代码描述：产生死锁。  

```java
public static void main(String[] args) {
    final Object a = new Object();
    final Object b = new Object();
    Thread threadA = new Thread(new Runnable() {
        public void run() {
            synchronized (a) {
                try {
                    System.out.println("now i in threadA-locka");
                    Thread.sleep(1000l);
                    synchronized (b) {
                        System.out.println("now i in threadA-lockb");
                    }
                } catch (Exception e) {
                    // ignore
                }
            }
        }
    });

    Thread threadB = new Thread(new Runnable() {
        public void run() {
            synchronized (b) {
                try {
                    System.out.println("now i in threadB-lockb");
                    Thread.sleep(1000l);
                    synchronized (a) {
                        System.out.println("now i in threadB-locka");
                    }
                } catch (Exception e) {
                    // ignore
                }
            }
        }
    });

    threadA.start();
    threadB.start();
}

```

### 5.加锁的理解

锁，是针对于对象来说，对于对象锁，必须是一个对象多个变量引用其时才会出现，这个对象可以随意，可以用对象本身，也可以新建一个object对象。  

**ps: wait()会释放锁。wait方法99%都是和while 一起使用。**  

下面的是一个测试，两个线程协作，操作一个数组对象。当线程1增加5个数据的时候，线程2获得通知，并将线程2停止。

```java
package offerDay2;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

//java 多线程的测试
//主要实现的功能， 线程一进行添加5个数据，然后线程二进行监听线程1的数据变化，当线程1增加5个元素的时候，需要将线程2停止。
public class MyContainer {
    volatile List list  =  new ArrayList();

    public void  add(Object o){
        list.add(o);
    }

    public int size(){
        return list.size();
    }

    public static void main(String[] args) {
		
        //需要等待的需要先执行。
        MyContainer myContainer = new MyContainer();
        final Object lock = new Object();  // 这个对象就作为一个锁，因为他们两个线程对于这个对象是竞争关系，通过这种竞争关系，达到锁的效果
        //故其类的类型无关，只需要对一个对象形成竞争关系即可。
        new Thread(()->{
            synchronized (lock){ //lock作为竞争关系。
                System.out.println("t2 启动");
                if (myContainer.size()!=5){  //等待notifyAll去进行解锁，
                    try {
                        lock.wait();  //wait 会释放锁。
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("t2 完成");
                lock.notifyAll(); //自己执行完以后，必须释放锁，否则线程1会一直等待下去。
            }
        },"t2").start();


        new Thread(()->{
            synchronized (lock){ //也有lock。
                for (int i = 0; i < 10; i++) {
                    myContainer.add(new Object());
                    System.out.println("add" + i);
                    if (myContainer.size() == 5){
                        lock.notifyAll();
                        try {
                            lock.wait();  //当加到5个数据以后，会释放锁，并且进行等待， 等待的消除标志 就是线程2释放锁。
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }

        },"t1 启动").start();
    }
}


//然后还有一种简单的实现，countLatch 门闩的机制。

//java 多线程的测试
//主要实现的功能， 线程一进行添加5个数据，然后线程二进行监听线程1的数据变化，当线程1增加5个元素的时候，需要将线程2停止。
//使用门闩进行线程之间的锁同步？
public class MyContainer1 {
    volatile List list  =  new ArrayList();

    public void  add(Object o){
        list.add(o);
    }

    public int size(){
        return list.size();
    }

    public static void main(String[] args) {
        MyContainer1 myContainer = new MyContainer1();

        CountDownLatch latch = new CountDownLatch(1); //当1变成0的时候，表示开锁。  新建一个门闩。

        new Thread(()->{
            System.out.println("t2 启动");
            if (myContainer.size()!=5){
                try {
                    latch.await(); //门闩等待
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("t2 完成");
        },"t2").start();


        new Thread(()->{
                for (int i = 0; i < 10; i++) {
                    myContainer.add(new Object());
                    System.out.println("add" + i);
                    if (myContainer.size() == 5){
                       latch.countDown(); //调用打开门闩，然后程序接着执行。这样就不需要那么多的notifyAll
                    }
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } 
                }

        },"t1 启动").start();
    }
}

```

以上的例子有点像线程之间的通信，但是高并还有一种就是多个引用调用同一个对象的情况。代码如下：

```java
public class SynchronizedExample {

    public void func1() {
        synchronized (this) { //this的话，就是绑定对应的实例化对象。 也就是使用自己这个对象进行绑定
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}

//使用两个线程同时操作同一个对象，这种加锁，也是执行顺序。而上面的是直接利用一个对象，实现两个线程之间的逻辑先后关系，等待关系。 可能有点绕，但是有点区别的。
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e1.func1());
}
```

### 6.ReentrantLock 类

其实现的功能与synchronized 差不多，就是更加的灵活一点，通过手动的加锁和解锁，能够做到的更加的细致。  

**二者之间的差别：**  

synchronized是一个关键字，而ReentantLock是一个类，可以进行很多的操作。具有更多的灵活性。另外 ReentantLock可以设置公平锁。  

使用说明：  

```java
//1. 最基础的加锁机制
Lock lock = new ReentractLock();
lock.lock();

//coding 

lock.unlock; //一定要加解锁。

//2. try lock 

try{
boolean locked = lock.tryLock(5,TimeUnit.SECONDS) //尝试等待5秒钟去申请锁，locked 返回一个申请成功的布尔值。 
catch(){}
finally(){
	if(locked){
     lock.unlock()
    }
   }  //需要判断一下有没有申请成功，然后在进行解锁与不解锁。

//3. 可打断的锁  即如果一个线程锁死，另一个线程在申请锁的时候如果发现，则可以自动的跳出不进行死等。
    
lock.interruptibly()  //在申请锁的时候可以被打断。 对比 lock.lock()；死等申请锁。

//4.申请公平锁
Lock lock = new ReentractLock(true); //构造函数 带个true 则表示为公平锁。   
```

### 7.生产者消费者的同步问题

一个商店，有两个生产者线程去生产。10个消费者线程去消费，实现对应的操作。

```java
//使用线程实现生产者消费者的问题
// 也就是多个线程竞争一个对象。
public class Shop {
    final List<Integer> lists = new LinkedList<>();
    int Max = 10;
    int count = 0;
    private void  get()  { //获取
        synchronized (this){   //上锁的只是一个get的操作，而操作的对象是Shop，即里面的bean，即List数组，为共享对象实例。  加锁的目的是，在进行执行这快代码的时候，只有一个线程，
            //但是同时进行等待的时候，所有线程会同时检测到，但是只有一个线程能够进行操作，也就是锁的含义。
            while (lists.size() == 0){ //为什么用while？ 因为为空的时候，10个线程都在等待增加货物，这时候放进去一个的时候，10个都在竞争，所以需要时刻检测不为0的情况，
                //否则会出现只有一个数据 而10个都获取到了的情况。即数据异常。
                try {
                    System.out.println("不足；");
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            lists.remove(0);
            System.out.println("拿出一个");
            count --;
            this.notifyAll(); //发送拿出一个是因为可能 此时已满，生产者线程正在等待。
        }
    }

    private void put(){ //放置货物。
        synchronized (this){
            while (lists.size() ==Max){ //满了的话就不能放了  
                try {
                    System.out.println("已满；");
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            lists.add(1);
            System.out.println("放进一个");
            count ++ ;
            this.notifyAll();//通知来消费
        }
    }

    public static void main(String[] args) {
        Shop s = new Shop();

        //生产者
        for (int i = 0; i < 2; i++) {
            new Thread(()->{
                for (int j = 0; j < 10; j++) {
                    s.put();
                }
            }).start();
        }

        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //消费者
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                for (int j = 0; j < 2; j++) {
                    s.get();
                }
            }).start();
        }
    }
}
```

### 8.ThreadLocal

当多个线程同时操作同一个对象的时候，使用ThreadLocal可以让操作只是在本地线程中有效。

### 9.线程安全的单列(singleton)

ArrayList 与vector都是数组，实现的功能是一致的，但是vector是线程安全的，而ArrayList 不具备线程安全，即vector在操作的时候系统会自动的加锁。经典的卖票问题。  

```java
package Day4;

import java.util.Vector;
import java.util.concurrent.TimeUnit;

//卖票问题
public class TickerSellers {
    static Vector<String> tickets = new Vector<>();

    static {
        for (int i = 0; i < 1000; i++) {
            tickets.add("票编号" + i);
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
			
               synchronized (tickets) {//加锁就不会出现不同的情况。即变成一个原子操作。  但是效率会低。

                    while (tickets.size() > 0) {  //虽然vector是线程安全的， size 和remove 是分开的，所有说也会出现 异常的情况，即取多。

                        //对size 和remove 来说 在操作的时候各自都是线程安全的，但是一块操作的时候，可能出现线程不安全，即中间可能出现延迟的情况。
                        try {
                            TimeUnit.MILLISECONDS.sleep(10);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                        System.out.println("销售了" + tickets.remove(0));
                    }
                }


            }).start();
        }
    }
}

```

上面的实现效率不是很高，可以通过判断其值来进行操作，即使用一个安全的单例操作，可以实现线程安全并且效率更高。

```java
public class TicketSeller1 {

    static Queue<String> tickets = new ConcurrentLinkedDeque<>(); //是线程安全的。

    static {
        for (int i = 0; i < 1000; i++) {
            tickets.add("票号 " + i);
        }
    }

    public static void main(String[] args) {

        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                while (true){
                    String s = tickets.poll(); //通过判断出去的值以后，然后对值进行判断是否为null，这样的话就只操作了一次，不存在两次相加
                    //够不成原子性的情况~~
                    if (s == null)break;
                    System.out.println(s);
                }

            }).start();
        }
    }
}
```

### 10.高并发容器

首先java的容器框架中分为四大类：List,Set,Queue,Map。这些在单线程中运用没有太大的问题，高并发容器就是针对高并发的特定对四大容器进行的优化。使其能后满足在高并发场景下的运用。  

高并发容器也就是将容器中线程不安全的操作，转成线程安全的操作。并发容器是专门针对多线程并发设计的，使用了锁分段技术，只对操作的位置进行同步操作，但是其他没有操作的位置其他线程任然可以访问，挺高程序的吞吐量。  

其中使用Collection.synchronizedList可以对基础的容器变成线程安全的容器。  

```java
List<String> str = new ArrayList<>();
List<String> strSync = Collection.synchronizedList(str)  //返回一个加锁的数组
```

七大并发容器：

1. **ConcurrentHashMap**：对应的非并发容器：HashMap
2. **CopyOnWriteArrayList**：对应的非并发容器：ArrayList
3. **CopyOnWriteArraySet**：对应的非并发容器：HashSet
4. **ConcurrentSkipListMap**：对应的非并发容器：TreeMap
5. **ConcurrentSkipListSet**：对应的非并发容器：TreeSet
6. **ConcurrentLinkedQueue**：对应的非并发容器：Queue
7. **LinkedBlockingQueue、ArrayBlockingQueue、PriorityBlockingQueue**：对应的非并发容器：BlockingQueue

### 11.线程池

线程池 类比于单线程，单线程是来一个进行执行，然后依次执行， 但是线程池的话，我一次开了5个线程工作，就是意思可以一次有五个人来帮我干活，然后循环干，效率会更高，而且线程在完成任务以后，线程不会消失，只会等待下一个任务的到来，然后再干。多雇人的意思。线程的启动和关闭耗费比较大。  

对比一个程序来说，就是 当有个计算任务进来时，将其切分为5个子任务去执行。

虽然使用for 进行new出来5个线程也可以，但是这样的新建与销毁很容易出现溢出，还有就是新建和关闭的耗费都很大，所以出现了线程池的做派。

Future<Interger> f = new Thread（new Callable） future 就可以获得对应的返回值。

线程池进行并行计算，等会打出来。

```java
//使用线程池实现并行计算。 进行计算素数 ，带返回的线程。

public class ParallelCompute {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        List<Integer> results = getPrime(1,200000);
        long end = System.currentTimeMillis();
        System.out.println(end -start);
        //约 3995
        final  int CPUNumber = 4;
        ExecutorService service = Executors.newFixedThreadPool(CPUNumber); //创造一个线程池，总共有四个线程，

        MyTask t1 = new MyTask(1,80000);
        MyTask t2 = new MyTask(80001,130000);
        MyTask t3 = new MyTask(130001,170000);
        MyTask t4 = new MyTask(170001,200000);

        Future<List<Integer>> f1 = service.submit(t1);
        Future<List<Integer>> f2 = service.submit(t2);
        Future<List<Integer>> f3 = service.submit(t3);
        Future<List<Integer>> f4 = service.submit(t4);  //其实四个人干的都是同一件事，然后都交给线程池去处理，这样也就更快了，但是高并发的意思也就是，同一件事，上千万个人来进行请求，所以会出现高并发。
        //那么对应我也应该有上千万个人来服务，也就是线程， 所以需要线程池去做，如果能做那么都做，否则就是阻塞进行等待这样。

        start = System.currentTimeMillis();
        f1.get(); //得到其计算的结果。 也是线程池。 这个是阻塞的。即任务完成以后 会进行阻塞等待。
        f2.get();
        f3.get();
        f4.get();
        end = System.currentTimeMillis();
        System.out.println(end -start);
        //约1362   差不多3倍。
    }

    static class  MyTask implements Callable<List<Integer>> {
        int starPos,endPos;
        MyTask(int s, int e){
            this.starPos = s;
            this.endPos = e;
        }

        //带返回值的callable ，类似于RunAble 功能一样。
        @Override
        public List<Integer> call() throws Exception {
            return getPrime(starPos,endPos);
        }
    }

    private static boolean isPrime(int number){
        for (int i = 2; i < number/2 ; i++) {
            if (number % i == 0) return false;
        }
        return  true;
    }

    //进行计算素数
    private static List<Integer> getPrime(int start, int end){
        List<Integer> results = new ArrayList<>();
        for (int i = start; i <= end; i++) {
            if(isPrime(i))results.add(i);
        }
        return results;
    }
}
```

6个线程池：

1. newFixedThreadPool 这个是开固定的几个线程，

2. newCachedThreadPool 这个是弹性的ThreadPool  如果没有则新开一个线程，知道系统不能开， 如果一个线程超过60秒等待没有用，则进行销毁。

3. SingleThreadPool  单例线程池。只有一个线程。

4. ScheduledPool  延迟计算的线程池。

5. ForkJoinPool  分叉合并线程。

6. WorkStealingPool  精灵线程，运行在后台。

举例说明：  

```java
//分和线程任务，实现加1000000数据，用线程进行自动的分进行计算。
public class ForkJoinTask {
    static int [] nums = new  int[1000000];
    static final int MAX_NUM = 5000;

    static Random r = new Random();

    static {
        for (int i = 0; i < nums.length ; i++) {
            nums[i] = r.nextInt(100);
        }
//        System.out.println(Arrays.stream(nums).sum());
    }


    static class AddTask extends RecursiveTask<Long> {

        int start,end;
        AddTask(int s,int e){
            this.start = s;
            this.end =e;
        }

        @Override
        protected Long compute() {
            if(end -start <= MAX_NUM){
                long  sum = 0L;
                for (int i = start;i<end;i++) sum += nums[i];
                return sum;
            }
            int middle = start + (end -start)/2;
            AddTask suTask1 = new AddTask(start,middle);
            AddTask suTask2 = new AddTask(middle,end);
            suTask1.fork();  //将程序进行fork 也就是分叉
            suTask2.fork();

            return suTask1.join() + suTask2.join(); //将结果合并， join  这样就是程序帮你进行递归分类。
        }
    }

    public static void main(String[] args) throws IOException {
        ForkJoinPool fjp =new ForkJoinPool();
        AddTask task = new  AddTask(0,nums.length);
        fjp.execute(task);
        long result = task.join();
        System.out.println(result);
    }
}

```

其实背后都是调用ThreadPoolExecutor。以下使用parallelStream()并行计算，对比之前的可以效率多加一倍。

```java
public class ParallelStreamAPI {
    public static void main(String[] args) {
        List<Integer> nums = new ArrayList<>();

        Random random = new Random();
        for (int i = 0; i < 10000; i++) {
            nums.add(1000000 + random.nextInt(1000000));
        }


        long start = System.currentTimeMillis();
        nums.forEach(v->isPrime(v));  //挨个轮着计算是否是质数
        long end = System.currentTimeMillis();

        System.out.println(end -start); //90

        start = System.currentTimeMillis();
        nums.parallelStream().forEach(ParallelStreamAPI::isPrime); //带有并行计算的流进行计算。 多线程进行计算。
        end = System.currentTimeMillis();
        System.out.println(end -start);//20 效率4倍多线程。
    }

    private static boolean isPrime(int num){
        for (int i = 2; i <  num/2; i++) {
            if (num % 2 ==0 ) return false;
        }
        return  true;
    }
}
```

多用多线程的思维去解决问题。。