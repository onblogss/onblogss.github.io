---
layout: post
title: Java中的并发工具类(CountDownLatch、CyclicBarrier、Semaphore、Exchanger)
date: 2018-09-26 17:45:00
author: 薛师兄
tags:
  - Java
  - 多线程
---
在JDK的并发包里提供了很多有意思的并发工具类。CountDownLatch、CyclicBarrier和Semaphore 工具类提供了一种并发流程控制的手段，Exchanger 工具类则提供了在线程间交换数据的一种手段。

## 1.等待多线程完成的 CountDownLatch

CountDownLatch允许一个或多个线程等待其他线程完成操作。

其实最简单的做噶是使用join()方法，join用于让当前执行线程等待join线程执行结束。其实现原理是不停检查join线程是否存活，如果join线程存活则让当前线程永远等待。其中，wait(0) 表示永远等待下去，代码片段如下：

```java
while (isAlive()) {
    wait(0);
}
```

知道线程中止后，线程的 this.notifyAll() 方法被调用，调用 notifyAll() 方法是在 JVM里实现的，所以在JDK里看不到，大家可以查看JVM源码。

在JDK1.5之后的并发包CountDownLatch也可以实现join的功能，并且功能更多，更强大。

示例代码：

```java
public static void main(String[] args) throws InterruptedException {
    CountDownLatch c = new CountDownLatch(2);
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println(1);
            c.countDown();
            System.out.println(2);
            c.countDown();//注释这行
        }
    }).start();
    c.await();
    System.out.println("3");
}
```

运行结果：

```java
1
2
3
```

CountDownLatch的构造方法接收一个int类型的参数作为计数器，如果你想等待N个点完成，这里就传入N。

当我们调用CountDownLatch的countDown()方法时，N就会减1，CountDownLatch的 await() 方法会阻塞当前线程，直到N变成零。由于countDown()方法可以用在任何地方，所以这里说的N个点，也可以是N个线程。用在多个线程时，你只需要把这个CountDownLatch的引用传递到线程里。

如果有某个线程处理的比较慢，我们不可能让主线程一直等待，所以我们可以使用另外一个带指定时间的await方法，await(long time, TimeUnit unit)， 这个方法等待特定时间后，就会不再阻塞当前线程。join也有类似的方法。

> **注意：**计数器必须大于等于0，只是等于0时候，计数器就是零，调用await方法时不会阻塞当前线程。CountDownLatch不可能重新初始化或者修改CountDownLatch对象的内部计数器的值。一个线程调用countDown方法 happen-before 另外一个线程调用await方法。

## 2.同步屏障CyclicBarrier

CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

### 2.1 构造方法

CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

示例代码：

```java
public static void main(String[] args) throws BrokenBarrierException, InterruptedException {
    CyclicBarrier c = new CyclicBarrier(2);
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                c.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("1");
        }
    }).start();
    c.await();
    System.out.println(2);
}
```

运行结果：

```java
1
2
```

或者

```java
2
1
```

如果把new CyclicBarrier(2)修改成new CyclicBarrier(3)则主线程和子线程会永远等待，因为没有第三个线程执行await方法，即没有第三个线程到达屏障，所以之前到达屏障的两个线程都不会继续执行

------

CyclicBarrier还提供一个更高级的构造函数CyclicBarrier(int parties, Runnable barrierAction)，用于在线程到达屏障时，优先执行barrierAction，方便处理更复杂的业务场景。

示例代码：

```java
public static void main(String[] args) throws BrokenBarrierException, InterruptedException {
    CyclicBarrier c = new CyclicBarrier(2, new Runnable() {
        @Override
        public void run() {
            System.out.println(3);
        }
    });
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                c.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("1");
        }
    }).start();
    c.await();
    System.out.println(2);
}
```

运行结果：

```java
3
1
2
```

### 2.2 应用场景

CyclicBarrier可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个Excel保存了用户所有银行流水，每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水。

### 2.3 CyclicBarrier和CountDownLatch的区别

CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset() 方法重置。所以CyclicBarrier能处理更为复杂的业务场景，例如，如果计算发生错误，可以重置计数器，并让线程们重新执行一次。

CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量。isBroken方法用来知道阻塞的线程是否被中断。

示例代码：

```java
public static void main(String[] args){
    CyclicBarrier c = new CyclicBarrier(2);
    Thread t = new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                c.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    });
    t.start();
    t.interrupt();
    try {
        c.await();
    } catch (InterruptedException e) {
          e.printStackTrace();
    } catch (BrokenBarrierException e) {
          e.printStackTrace();
    }finally {
        System.out.println(c.isBroken());
    }
}
```

运行结果：

```java
true
```

## 3.控制并发线程数的Semaphore

#### 3.1 作用

Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。

#### 3.2 简介

Semaphore也是一个线程同步的辅助类，可以维护当前访问自身的线程个数，并提供了同步机制。使用Semaphore可以控制同时访问资源的线程个数，例如，实现一个文件允许的并发访问数。

#### 3.3 应用场景

Semaphore可以用于做流量控制，特别公用资源有限的应用场景，比如数据库连接。假如有一个需求，要读取几万个文件的数据，因为都是IO密集型任务，我们可以启动几十个线程并发的读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接数只有10个，这时我们必须控制只有十个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连接。这个时候，我们就可以使用Semaphore来做流控，代码如下：

```java
public static void main(String[] args) {
    final int THREAD_NUM = 30;
    ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_NUM);
    Semaphore s = new Semaphore(10);//只允许10个线程并发执行
    for (int i = 0; i < THREAD_NUM; i++) {
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    s.acquire();
                    System.out.println("play");
                    s.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
    }
    threadPool.shutdown();
}
```

在代码中，虽然有30个线程在执行，但是只允许10个并发的执行。Semaphore的构造方法Semaphore(int permits) 接受一个整型的数字，表示可用的许可证数量。Semaphore(10)表示允许10个线程获取许可证，也就是最大并发数是10。Semaphore的用法也很简单，首先线程使用Semaphore的acquire()获取一个许可证，使用完之后调用release()归还许可证。还可以用tryAcquire()方法尝试获取许可证。

#### 3.4 其他方法

Semaphore还提供一些其他方法：

- int availablePermits() ：返回此信号量中当前可用的许可证数。
- int getQueueLength()：返回正在等待获取许可证的线程数。
- boolean hasQueuedThreads() ：是否有线程正在等待获取许可证。
- void reducePermits(int reduction) ：减少reduction个许可证。是个protected方法。
- Collection getQueuedThreads() ：返回所有等待获取许可证的线程集合。是个protected方法。

## 4.线程间交换数据的Exchanger

Exchanger（交换者）是一个用于线程间协作的工具类。Exchanger用于进行线程间的数据交换。它提供一个同步点，在这个同步点两个线程可以交换彼此的数据。这两个线程通过exchange方法交换数据， 如果第一个线程先执行exchange方法，它会一直等待第二个线程也执行exchange，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

#### 应用场景

**1、**Exchanger可以用于遗传算法，遗传算法里需要选出两个人作为交配对象，这时候会交换两人的数据，并使用交叉规则得出2个交配结果。
**2、**Exchanger也可以用于校对工作。比如我们需要将纸制银流通过人工的方式录入成电子银行流水，为了避免错误，采用AB岗两人进行录入，录入到Excel之后，系统需要加载这两个Excel，并对这两个Excel数据进行校对，看看是否录入的一致。代码如下：

```java
public static void main(String[] args) {
    Exchanger<String> exchanger = new Exchanger<>();
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                String a = "银行流水A";
                exchanger.exchange(a);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }).start();
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                String b = "银行流水B";
                String a = exchanger.exchange(b);
                System.out.println("在B中获取到录入的A是："+a);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }).start();
}
```

运行结果：

```java
在B中获取到录入的A是：银行流水A
```

如果两个线程有一个没有到达exchange方法，则会一直等待,如果担心有特殊情况发生，避免一直等待，可以使用exchange(V data, long time, TimeUnit unit)设置最大等待时长。

## 参考

*《Java并发编程的艺术》方、魏、程著*