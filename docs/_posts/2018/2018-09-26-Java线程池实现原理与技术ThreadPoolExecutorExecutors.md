---
layout: post
title: Java线程池实现原理与技术(ThreadPoolExecutor、Executors)
date: 2018-09-26 12:24:00
author: 薛勤
tags:
  - Java
  - 多线程
---
> 本文将通过实现一个简易的线程池理解线程池的原理，以及介绍JDK中自带的线程池ThreadPoolExecutor和Executor框架。

## 1.无限制线程的缺陷

多线程的软件设计方法确实可以最大限度地发挥多核处理器的计算能力，提高生产系统的吞吐量和性能。但是，若不加控制和管理的随意使用线程，对系统的性能反而会产生不利的影响。

一种最为简单的线程创建和回收的方法类似如下：

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        //do sth
    }
}).start();
```

以上代码创建了一条线程，并在run()方法结束后，自动回收该线程。在简单的应用系统中，这段代码并没有太多问题。但是在真实的生产环境中，系统由于真实环境的需要，可能会开启很多线程来支撑其应用。而当线程数量过大时，反而会耗尽CPU和内存资源。

首先，虽然与进程相比，线程是一种轻量级的工具，但其创建和关闭依然需要花费时间，如果为每一个小的任务都创建一个线程，很有可能出现创建和销毁线程所占用的时间大于该线程真实工作所消耗的时间，反而会得不偿失。

其次，线程本身也是要占用内存空间的，大量的线程会抢占宝贵的内部资源。

因此，在实际生产环境中，线程的数量必须得到控制。盲目地大量创建线程对系统性能是有伤害的。

## 2.简单的线程池实现

下面给出一个最简单的线程池，该线程池不是一个完善的线程池，但已经实现了一个基本线程池的核心功能，有助于快速理解线程池的实现。

1.线程池的实现

```java
public class ThreadPool {
    private static ThreadPool instance = null;

    //空闲的线程队列
    private List<PThread> idleThreads;
    //已有的线程总数
    private int threadCounter;
    private boolean isShutDown = false;

    private ThreadPool() {
        this.idleThreads = new Vector<>(5);
        threadCounter = 0;
    }

    public int getCreatedThreadCounter() {
        return threadCounter;
    }

    //取得线程池的实例
    public synchronized static ThreadPool getInstance() {
        if (instance == null) {
            instance = new ThreadPool();
        }
        return instance;
    }

    //将线程池放入池中
    protected synchronized void repool(PThread repoolingThread) {
        if (!isShutDown) {
            idleThreads.add(repoolingThread);
        } else {
            repoolingThread.shutDown();
        }
    }

    //停止池中所有线程
    public synchronized void shutDown() {
        isShutDown = true;
        for (int threadIndex = 0; threadIndex < idleThreads.size(); threadIndex++) {
            PThread pThread = idleThreads.get(threadIndex);
            pThread.shutDown();
        }
    }

    //执行任务
    public synchronized void start(Runnable target) {
        PThread thread = null;
        //如果有空闲线程，则直接使用
        if (idleThreads.size() > 0) {
            int lastIndex = idleThreads.size() - 1;
            thread = idleThreads.get(lastIndex);
            idleThreads.remove(thread);
            //立即执行这个任务
            thread.setTarget(target);
        }//没有空闲线程，则创建线程
        else {
            threadCounter++;
            //创建新线程
            thread = new PThread(target, "PThread #" + threadCounter, this);
            //启动这个线程
            thread.start();
        }
    }

}
```

2.要实现上面的线程池，就需要一个永不退出的线程与之配合。PThread就是一个这样的线程。它的主体部分是一个无限循环，该线程在手动关闭前永不结束，并一直等待新的任务到达。

```java
public class PThread extends Thread {
    //线程池
    private ThreadPool pool;
    //任务
    private Runnable target;
    private boolean isShutDown = false;
    private boolean isIdle = false; //是否闲置
    //构造函数
    public PThread(Runnable target,String name, ThreadPool pool){
        super(name);
        this.pool = pool;
        this.target = target;
    }

    public Runnable getTarget(){
        return target;
    }

    public boolean isIdle() {
        return isIdle;
    }

    @Override
    public void run() {
        //只要没有关闭，则一直不结束该线程
        while (!isShutDown){
            isIdle =  false;
            if (target != null){
                //运行任务
                target.run();
            }
            try {
                //任务结束了,到闲置状态
                isIdle = true;
                pool.repool(this);
                synchronized (this){
                    //线程空闲，等待新的任务到来
                    wait();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            isIdle = false;
        }
    }

    public synchronized void setTarget(Runnable newTarget){
        target = newTarget;
        //设置了任务之后，通知run方法，开始执行这个任务
        notifyAll();
    }

    //关闭线程
    public synchronized void shutDown(){
        isShutDown = true;
        notifyAll();
    }

}
```

3.测试Main方法

```java
public static void main(String[] args) throws InterruptedException {
    for (int i = 0; i < 1000; i++) {
        ThreadPool.getInstance().start(new Runnable() {
            @Override
            public void run() {
                try {
                    //休眠100ms
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

## 3.ThreadPoolExecutor

为了能够更好地控制多线程，JDK提供了一套Executor框架，帮助开发人员有效地进行线程控制。Executor框架无论是newFixedThreadPool()方法、newSingleThreadExecutor()方法、newScheduledThreadPool()方法、还是newCachedThreadPool()方法，其内部实现均使用了 ThreadPoolExecutor：

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

由以上线程池的实现代码可以知道，它们只是对 ThreadPoolExecutor 类的封装。为何 ThreadPoolExecutor 类有如此强大的功能？来看一下 ThreadPoolExecutor 最重要的构造方法。

### 3.1 构造方法

ThreadPoolExecutor最重要的构造方法如下：

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
```

方法参数如下：

| 参数            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| corePoolSize    | 指定了线程池中的线程数量                                     |
| maximumPoolSize | 指定了线程池中最大的线程数量                                 |
| keepAliveTime   | 当线程池线程数量超过corePoolSize时，多余的空闲线程的存活时间。<br />即，超过corePoolSize的空闲线程，在多长时间内会被销毁 |
| unit            | keepAliveTime 的单位，如：`TimeUnit.SECONDS`                 |
| workQueue       | 任务队列，被提交但尚未被执行的任务。                         |
| threadFactory   | 线程工厂，用于创建线程，一般用默认的即可。                   |
| handler         | 拒绝策略。当任务太多来不及处理，如何拒绝任务。               |

ThreadPoolExecutor的使用示例，通过execute()方法提交任务。

```java
public static void main(String[] args) {
    ThreadPoolExecutor executor = new ThreadPoolExecutor(4, 5, 0, TimeUnit.SECONDS, new LinkedBlockingQueue<>());
    for (int i = 0; i < 10; i++) {
        executor.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        });
    }
    executor.shutdown();
}
```

或者通过submit()方法提交任务

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ThreadPoolExecutor executor = new ThreadPoolExecutor(4, 5, 0, TimeUnit.SECONDS, new LinkedBlockingQueue<>());
    List<Future> futureList = new Vector<>();
    //在其它线程中执行100次下列方法
    for (int i = 0; i < 100; i++) {
        futureList.add(executor.submit(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return Thread.currentThread().getName();
            }
        }));
    }
    for (int i = 0;i<futureList.size();i++){
        Object o = futureList.get(i).get();
        System.out.println(o.toString());
    }
    executor.shutdown();
}
```

运行结果：

```java
...
pool-1-thread-4
pool-1-thread-3
pool-1-thread-2
```

下面主要讲解ThreadPoolExecutor的构造方法中workQueue和RejectedExecutionHandler参数，其它参数都很简单。

### 3.2 workQueue任务队列

用于保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列。

- ArrayBlockingQueue: 是一个基于数组结构的有界阻塞队列，按FIFO原则进行排序

- LinkedBlockingQueue: 一个基于链表结构的阻塞队列，吞吐量高于ArrayBlockingQueue。静态工厂方法Excutors.newFixedThreadPool()使用了这个队列

- SynchronousQueue:  一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量高于LinkedBlockingQueue，静态工厂方法Excutors.newCachedThreadPool()使用了这个队列

- PriorityBlockingQueue: 一个具有优先级的无限阻塞队列。

### 3.3 RejectedExecutionHandler饱和策略

当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略还处理新提交的任务。它可以有如下四个选项：

- AbortPolicy : 直接抛出异常，默认情况下采用这种策略
- CallerRunsPolicy : 只用调用者所在线程来运行任务
- DiscardOldestPolicy : 丢弃队列里最近的一个任务，并执行当前任务
- DiscardPolicy : 不处理，丢弃掉

更多的时候，我们应该通过实现RejectedExecutionHandler 接口来自定义策略，比如记录日志或持久化存储等。

### 3.4 submit()与execute()

可以使用execute和submit两个方法向线程池提交任务。

1. execute方法用于提交不需要返回值的任务，利用这种方式提交的任务无法得知是否正常执行
2. submit方法用于提交一个任务并带有返回值，这个方法将返回一个Future类型对象。可以通过这个返回对象判断任务是否执行成功，并且可以通过future.get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成。

### 3.5 shutdown()与shutdownNow()

可以通过调用 `shutdown()` 或 `shutdownNow()` 方法来关闭线程池。它们的原理是遍历线程池中的工作线程，然后逐个调用线程的 `interrupt` 方法来中断线程，所以无法响应中断的任务可能永远无法停止。

这俩方法的区别是，shutdownNow() 首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表，而 shutdown() 只是将线程池的状态设置成 SHUTDOWN 状态，然后中断所有没有正在执行任务的线程。

只要调用了这两个关闭方法的任意一个，isShutdown 方法就会返回 true。当所有的任务都已关闭了，才表示线程池关闭成功，这时调用 isTerminaced 方法会返回 true。

通常调用 shutdown() 方法来关闭线程池，如果任务不一定要执行完，则可以调用 shutdownNow() 方法。

### 3.6 合理配置线程池

要想合理地配置线程池，首先要分析任务特性

- 任务的性质：CPU密集型任务、IO密集型任务和混合型任务。
- 任务的优先级：高、中和低。
- 任务的执行时间：长、中和短。
- 任务的依赖性：是否依赖其他系统资源，如数据库连接。

性质不同的任务可以用不同规模的线程池分开处理。

CPU密集型任务应该配置尽可能少的线程，如配置N+1个线程，N位CPU的个数。

而IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如2*N。

混合型任务，如果可以拆分，将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐量将高于串行执行的吞吐量。如果这两个任务执行的时间相差很大，则没有必要进行分解。可以通过`Runtime.getRuntime().availableProcessors()`方法获得当前设备的CPU个数。

优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先执行。

### 3.7 线程池的监控

由于大量的使用线程池，所以很有必要对其进行监控。可以通过继承线程池来自定义线程池，重写线程池的beforeExecute、afterExecute 和 terminated 方法，也可以在任务执行前，执行后和线程池关闭前执行一些代码来进行监控。在监控线程池的时候可以使用一下属性：

(1) taskCount：线程池需要执行的任务数量

(2) completedTaskCount：线程池在运行过程中已完成的任务数量，小于或等于taskCount

(3) largestPoolSize： 线程池里曾经创建过最大的线程数量。通过这个数据可以知道线程池是否曾经满过。如该数值等于线程池最大大小，则表示线程池曾经满过。

(4) getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。

(5) getActiveCount：获取活动的线程数



## 4.Executor多线程框架

ThreadPoolExecutor 表示一个线程池，Executors 类则扮演着线程池工厂的角色，通过 Executors 可以取得一个特定功能的线程池。

使用 Executors 框架实现上节中的例子，其代码如下：

```java
public static void main(String[] args) {
    //新建一个线程池
    ExecutorService executor = Executors.newCachedThreadPool();
    //在其它线程中执行100次下列方法
    for (int i = 0; i < 100; i++) {
        executor.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        });
    }
    //执行完关闭
    executor.shutdown();
}
```

### 4.1 Executors框架的结构

1. 任务

   包括被执行任务需要实现的接口：Runnable 接口或 Callable 接口。

2. 任务的执行

   包括任务执行机制的核心接口 Executor，以及继承自 Executor 的ExecutorService 接口。Executor框架有两个关键类实现了 ExecutorService 接口（ThreadPoolExecutor 和 ScheduledThreadPoolExecutor）。

3. 异步计算的结果

   包括接口 Future 和实现Future接口的FutureTask类。

### 4.2 Executors工厂方法

Executors工厂类的主要方法：

```java
public static ExecutorService newFixedThreadPool(int nThreads)
```

- 该方法返回一个固定线程数量的线程池，该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。

```java
public static ExecutorService newSingleThreadExecutor()
```

- 该方法返回一个只有一个线程的线程池。若多余一个任务被提交到线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。

```java
public static ExecutorService newCachedThreadPool()
```

- 该方法返回一个可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。但所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。

```java
public static ScheduledExecutorService newSingleThreadScheduledExecutor() 
```

- 该方法返回一个ScheduledExecutorService对象，线程池大小为1。ScheduledExecutorService接口在ExecutorService接口之上扩展了在给定时间执行某任务的功能，如在某个固定的延时之后执行，或者周期性执行某个任务。

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
```

- 该方法也返回一个 ScheduledExecutorService 对象，但该线程池可以指定线程数量。

### 4.3 ThreadPoolExecutor与ScheduledThreadPoolExecutor

在前面提到了Executors 类扮演着线程池工厂的角色，通过 Executors 可以取得一个特定功能的线程池。Executors 工厂类的主要方法可以创建 ThreadPoolExecutor 和 ScheduledThreadPoolExecutor 线程池。

关于ThreadPoolExecutor ，前面第3节已经详细叙述。ScheduledThreadPoolExecutor 也是ExecutorService接口的实现类，可以在给定的延迟后运行命令，或者定期执行命令。ScheduledThreadPoolExecutor 比 Timer 更灵活，功能更强大。

### 4.4 Future与FutureTask

上面的示例中使用 execute() 方法提交任务，用于提交不需要返回值的任务。如果我们需要获取执行任务之后的返回值，可以使用submit()方法。

示例代码：

```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
     //新建一个线程池
     ExecutorService executor = Executors.newCachedThreadPool();
     List<Future> futureList = new Vector<>();
     //在其它线程中执行100次下列方法
     for (int i = 0; i < 100; i++) {
         futureList.add(executor.submit(new Callable<String>() {
             @Override
             public String call() throws Exception {
                 return Thread.currentThread().getName()+" "+System.currentTimeMillis()+" ";
             }
         }));
     }
     for (int i = 0;i<futureList.size();i++){
         Object o = futureList.get(i).get();
         System.out.println(o.toString()+i);
     }
     executor.shutdown();
 }
```

运行结果：

```java
...
pool-1-thread-11 1537872778612 96
pool-1-thread-11 1537872778613 97
pool-1-thread-10 1537872778613 98
pool-1-thread-10 1537872778613 99
```

到这里，就不得不提Future接口与FutureTask实现类，它们代表异步计算的结果。

```java
Future<T> submit(Callable<T> task)
Future<?> submit(Runnable task);
Future<T> submit(Runnable task, T result);
```

当我们submit()提交后，会返回一个Future对象，到JDK1.8，返回的实际是FutureTask实现类。submit() 方法支持 Runnable 或 Callable 类型的参数。Runnable 接口 和Callable 接口的区别就是 Runnable 不会返回结果，Callable 会返回结果。

主线程可以执行 futureTask.get() 方法来阻塞当前线程直到任务执行完成，任务完成后返回任务执行的结果。

futureTask.get(long timeout, TimeUnit unit) 方法则会阻塞当前线程一段时间立即返回，这时候有可能任务没有执行完。

主线程也可以执行 futureTask.cancel(boolean mayInterruptIfRunning) 来取消此任务的执行。

futureTask.isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

futureTask.isDone方法表示任务是否已经完成，若任务完成，则返回true。

> 如果没有什么特殊要求，可以直接使用JDK中的内置线程池，来改善系统的性能。

## 参考

*《Java程序性能优化》葛一鸣著*

*《Java并发编程的艺术》方、魏、程著*