---
layout: post
title: 深入理解［Master-Worker模式］原理与技术
date: 2018-09-24 14:11:00
author: 薛师兄
tags: 设计模式
---
Master-Worker模式是常用的并行模式之一。它的核心思想是，系统由两类进程协作工作：Master进程和Worker进程。Master进程负责接收和分配任务，Worker进程负责处理子任务。当各个Worker进程将子任务处理完成后，将结果返回给Master进程，由Master进程做归纳和汇总，从而得到系统的最终结果，其处理过程如图1所示。

![](./20180924深入理解MasterWorker模式原理与技术/1136672-20180924141058825-2091628000.png)


Master-Worker模式的好处，它能够将一个大任务分解成若干个小任务，并且执行，从而提高系统的吞吐量。而对于系统请求者Client来说，任务一旦提交，Master进程会分配任务并立即返回，并不会等待系统全部处理完成后再返回，其处理过程是异步的。因此Client不会出现等待现象。

## 1.Master-Worker的模式结构

Master-Worker模式是一种使用多线程进行数据结构处理的结构。

Master进程为主要进程，它维护了一个Worker进程队列、子任务队列和子结果集。Worker进程队列中的Worker进程，不停地从任务队列中提取要处理的子任务，并将子任务的处理结果写入结果集。

## 2.Master-Worker的代码实现

基于以上的思路实现一个简易的master-worker框架。其中Master部分的代码如下：

```java
public class Master {
    //任务队列
    protected Queue<Object> workQuery = new ConcurrentLinkedQueue<Object>();
    //worker进程队列
    protected Map<String, Thread> threadMap = new HashMap<>();
    //子任务处理结果集
    protected Map<String, Object> resultMap = new ConcurrentHashMap<>();

    //是否所有的子任务都结束了
    public boolean isComplete() {
        for (Map.Entry<String, Thread> entry : threadMap.entrySet()) {
            if (entry.getValue().getState()!=Thread.State.TERMINATED){
                return false;
            }
        }
        return true;
    }

    //Master 的构造，需要一个Worker 进程逻辑，和需要的Worker进程数量
    public Master(Worker worker,int countWorker){
        worker.setWorkQueue(workQuery);
        worker.setResultMap(resultMap);
        for (int i = 0; i < countWorker; i++) {
            threadMap.put(Integer.toString(i),new Thread(worker));
        }
    }

    //提交一个任务
    public void submit(Object job){
        workQuery.add(job);
    }

    //返回子任务结果集
    public Map<String,Object> getResultMap(){
        return resultMap;
    }

    //开始运行所有的worker进程，进行处理
    public void  execute(){
        for (Map.Entry<String,Thread> entry : threadMap.entrySet()){
            entry.getValue().start();
        }
    }

}
```

对应的Worker进程的代码实现：

```java
public class Worker implements Runnable {
    //任务队列，用于取得子任务
    protected Queue<Object> workQueue;
    //子任务处理结果集
    protected Map<String, Object> resultMap;

    public void setWorkQueue(Queue<Object> workQueue) {
        this.workQueue = workQueue;
    }

    public void setResultMap(Map<String, Object> resultMap) {
        this.resultMap = resultMap;
    }

    //子任务处理的逻辑，在子类中实现具体逻辑
    public Object handle(Object input) {
        return input;
    }

    @Override
    public void run() {
        while (true) {
            //获取子任务，poll()方法取出（并删除）队首的对象
            Object input = workQueue.poll();
            if (input == null) {
                break;
            }
            //处理子任务
            Object re = handle(input);
            //将处理结果写入结果集
            resultMap.put(Integer.toString(input.hashCode()), re);
        }
    }
}
```

以上两段代码已经展示了Master-Worker框架的全貌。应用程序通过重载 Worker.handle() 方法实现应用层逻辑。

例如，要实现计算1+2+..+100的结果，代码如下：

```java
public class PlusWorker extends Worker {

    @Override
    public Object handle(Object input) {
        Integer i = (Integer) input;
        return i+1;
    }

    public static void main(String[] args) {
        Master master = new Master(new PlusWorker(), 5);
        for (int i = 0; i < 100; i++) {
            master.submit(i); //提交一百个子任务
        }
        master.execute(); //开始计算
        int re = 0;
        Map<String, Object> resultMap = master.getResultMap();
        while (resultMap.size() > 0 || !master.isComplete()) {
            Set<String> keys = resultMap.keySet();
            String key = null;
            for (String k : keys) {
                key = k;
                break;
            }
            Integer i = null;
            if (key != null) {
                i = (Integer) resultMap.get(key);   //从结果集中获取结果
            }
            if (i != null) {
                re += i;        //最终结果
            }
            if (key != null) {
                resultMap.remove(key);      //移除已经被计算过的项
            }
        }
        System.out.println("result: " + re);
    }

}
```

运行结果：

```java
result: 5050
```

在应用层代码中，创建了5个Worker工作进程和Worker工作实例PlusWorker。在提交了100个子任务后，便开始子任务的计算。这些子任务中，由生成的5个Worker进程共同完成。Master并不等待所有的Worker执行完毕，就开始访问子结果集进行最终结果的计算，直到子结果集中所有的数据都被处理，并且5个活跃的Worker进程全部终止，才给出最终计算结果。

> Master-Worker模式是一种将串行任务并行化的方法，被分解的子任务在系统中可以被并行处理。同时，如果有需要，Master进程不需要等待所有子任务都完成计算，就可以根据已有的部分结果集计算最终结果。



## 3.Amino框架提供的Master-Worker模式

在Amino框架中为Master-Worker模式提供了较为完善的实现和便捷的操作接口。Amino实现了两套Master-Worker实现：一种是静态的Master-Worker实现，另一种是动态实现。

静态实现不允许在任务开始时添加新的子任务，而动态的Master-Worker允许在任务执行过程中，由Master或Worker添加新的子任务。

在Amino框架中，`MasterWorkerFactory.newStatic(new Pow3(),20)`用于创建静态的Master-Worker模式，

第二个参数为Worker线程数，第一个参数为执行的任务类，该类需实现`Doable<Integer,Integer>`接口，该接口泛型的第一个类型为任务方法的参数类型，第二个类型为方法返回类型。`MasterWorkerFactory.newDynamic(new Pow3Dyn())`用于创建动态的Master-Worker模式，其中参数为实现`DynamicWorker`接口的实例。

`submit()`方法用于提交应用层任务，`execute()`方法将执行所有任务。

Amino框架需要自行下载，下载地址：[https://sourceforge.net/projects/amino-cbbs/files/cbbs/0.5.3/](https://sourceforge.net/projects/amino-cbbs/files/cbbs/0.5.3/)，找到[cbbs-java-bin-0.5.3.tar.gz](https://sourceforge.net/projects/amino-cbbs/files/cbbs/0.5.3/cbbs-java-bin-0.5.3.tar.gz/download) 下载即可。

下面用Amino框架演示1+2+..+100的完整示例。

```java
public class Pow3 implements Doable<Integer,Integer> {
    @Override
    public Integer run(Integer input) {
        //业务逻辑
        return input;
    }
}
```

```java
public class Pow3Dyn implements DynamicWorker<Integer,Integer> {
    @Override
    public Integer run(Integer integer, WorkQueue<Integer> workQueue) {
        //业务逻辑
        return integer;
    }
}
```

```java
public class AminoDemo {

    /
     * Amino 框架提供开箱即用的Master-Worker模式
     * 其它用法参考API文档
     */
    public static void main(String[] args) {
        new AminoDemo().testDynamic();
        new AminoDemo().testStatic();
    }

    /
     * 静态模式，不允许在任务开始后添加新的任务
     */
    public void testStatic(){
        MasterWorker<Integer,Integer> mw = MasterWorkerFactory.newStatic(new Pow3(),20);//静态模式，可指定线程数
        List<MasterWorker.ResultKey> keyList = new Vector<>();
        for (int i = 1; i <= 100; i++) {
            keyList.add(mw.submit(i)); //传参并调度任务，key用于取得任务结果
        }
        mw.execute();//执行所有任务
        int re = 0;
        while (keyList.size()> 0){ //不等待全部执行完成，就开始求和
            MasterWorker.ResultKey k = keyList.get(0);
            Integer i = mw.result(k); //由Key取得一个任务结果
            if (i!=null){
                re+=i;
                keyList.remove(0); //累加完成后
            }
        }
        System.out.println("result:"+re);
        mw.shutdown();//关闭master-worker，释放资源
    }

    /
     * 动态模式，可在开始执行任务后继续添加任务
     */
    public void testDynamic(){
        MasterWorker<Integer,Integer> mw = MasterWorkerFactory.newDynamic(new Pow3Dyn());//动态模式，可指定线程数
        List<MasterWorker.ResultKey> keyList = new Vector<>();
        for (int i = 1; i < 50; i++) {
            keyList.add(mw.submit(i)); //传参并调度任务，key用于取得任务结果
        }
        mw.execute();
        for (int i = 50; i <= 100; i++) {
            keyList.add(mw.submit(i)); //传参并调度任务，key用于取得任务结果
        }
        int re = 0;
        while (keyList.size()> 0){
            MasterWorker.ResultKey k = keyList.get(0);
            Integer i = mw.result(k); //由Key取得一个任务结果
            if (i!=null){
                re+=i;
                keyList.remove(0); //累加完成后
            }
        }
        System.out.println("result:"+re);
        mw.shutdown();
    }

}
```

运行结果：

```java
result:5050
result:5050
```

MasterWorker类的方法摘要，其它请自行下载API文档。[cbbs-java-apidocs-0.5.3.tar.gz](https://sourceforge.net/projects/amino-cbbs/files/cbbs/0.5.3/cbbs-java-apidocs-0.5.3.tar.gz/download)

| 方法摘要                   |                                                              |
| -------------------------- | ------------------------------------------------------------ |
| ` boolean`                 | `execute()`            Begin processing of the work items submitted. |
| ` boolean`                 | `execute(long timeout, java.util.concurrent.TimeUnit unit)`            Begin processing of the work items submitted. |
| ` void`                    | `finished()`            Indicate to the master/worker that there is not more work coming. |
| ` java.util.Collection<T>` | `getAllResults()`            Obtain all of the results from the processing work items. |
| ` boolean`                 | `isCompleted()`            Poll an executing master/worker for completion. |
| ` boolean`                 | `isStatic()`            Determine if a master/worker is static. |
| ` int`                     | `numWorkers()`            Get the number of active workers.  |
| ` T`                       | `result(MasterWorker.ResultKey k)`            Obtain the results from the processing of a work item. |
| ` void`                    | `shutdown()`            Shutdown the master/worker.          |
| ` MasterWorker.ResultKey`  | `submit(S w)`            Submit a work item for processing.  |
| ` MasterWorker.ResultKey`  | `submit(S w, long timeout, java.util.concurrent.TimeUnit unit)`            Submit a work item for processing and block until it is either submitted successfully or the specified timeout period has expired. |
| ` boolean`                 | `waitForCompletion()`            Wait until all workers have completed. |
| ` boolean`                 | `waitForCompletion(long timeout, java.util.concurrent.TimeUnit unit)`            Wait until all workers have completed or the specified timeout period expires. |

## 参考

*《Java程序性能优化》葛一鸣著*