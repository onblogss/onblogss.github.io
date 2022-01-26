---
layout: post
title: 并发编程之详解InheritableThreadLocal类原理
date: 2019-08-16 15:14:00
author: 薛师兄
tags: Java
---
在Java并发编程中，InheritableThreadLocal 与 ThreadLocal 都可以用于线程间通信，不同的是 InheritableThreadLocal 继承了 ThreadLocal，并且扩展了 ThreadLocal。使用类 InheritableThreadLocal 可使子线程继承父线程的值。相反，类 ThreadLocal 不能实现值继承。

使用示例：

```java
public class LocalThread extends Thread {
    private static InheritableThreadLocal local = new InheritableThreadLocal();

    @Override
    public void run() {
        System.out.println("thread线程："+ local.get());
    }

    public static void main(String[] args) throws InterruptedException {
        local.set("main的值");
        LocalThread t = new LocalThread();
        t.start();
        System.out.println("main线程："+ local.get());
    }

}
```


分析下 InheritableThreadLocal 类源码：

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    /**
     * Computes the child's initial value for this inheritable thread-local
     * variable as a function of the parent's value at the time the child
     * thread is created.  This method is called from within the parent
     * thread before the child is started.
     * <p>
     * This method merely returns its input argument, and should be overridden
     * if a different behavior is desired.
     *
     * @param parentValue the parent thread's value
     * @return the child thread's initial value
     */
    protected T childValue(T parentValue) {
        return parentValue;
    }
 
    /**
     * Get the map associated with a ThreadLocal.
     *
     * @param t the current thread
     */
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }
 
    /**
     * Create the map associated with a ThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the table.
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

可以看到，getMap() 方法和 creatMap() 方法都是重写的 ThreadLocal 类方法，区别在于把 ThreadLocal 中的 threadLocals 换成了 inheritableThreadLocals，这两个变量都是ThreadLocalMap类型，并且都是Thread类的属性，源码如下：

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;

/*
 * InheritableThreadLocal values pertaining to this thread. This map is
 * maintained by the InheritableThreadLocal class.
 */
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

inheritableThreadLocal 如何实现值继承的呢？继续看下面的代码：

``` java
/**
 * Construct a new map including all Inheritable ThreadLocals
 * from given parent map. Called only by createInheritedMap.
 *
 * @param parentMap the map associated with parent thread.
 */
private ThreadLocalMap(ThreadLocalMap parentMap) {
    Entry[] parentTable = parentMap.table;
    int len = parentTable.length;
    setThreshold(len);
    table = new Entry[len];

    for (int j = 0; j < len; j++) {
        Entry e = parentTable[j];
        if (e != null) {
            @SuppressWarnings("unchecked")
            ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
            if (key != null) {
                Object value = key.childValue(e.value);
                Entry c = new Entry(key, value);
                int h = key.threadLocalHashCode & (len - 1);
                while (table[h] != null)
                    h = nextIndex(h, len);
                table[h] = c;
                size++;
            }
        }
    }
}
```

在构造方法的完整源代码算法中可以发现，子线程将父线程中的 table 对象以复制的方式赋值给子线程的 table 数组，这个过程是在创建 Thread 类对象时发生的，也就说明当子线程对象创建完毕后，子线程中的数据就是主线程中旧的数据，主线程使用新的数据时，子线程还是使用旧的数据，因为主子线程使用两个 Entry[] 对象数组各自存储自己的值。

这部分涉及到 Java 的值传递。对于对象来说，值的内容其实是对象的引用。当在父线程中修改对象的某一属性，子线程由于引用着相同对象，所以可以感知到，本质上是在操作同一块内存地址。

对于基本数据类型（int、long）来说，由于传递的是值，在父线程改变了数据后，子线程依旧使用的是旧的数据。这里尤其要提 String 字符串，String 虽然不是基本数据类型，但是由于内部字符数组被 final 修饰带来的不可变型，当父线程修改其 String 类型数据时，等于替换掉该 String 对象，而并不是修改原 String 对象的值，所以子线程依旧不会发生变化。

另外，重写类 InheritableThreadLocal 的 childValue() 方法可以对继承的值进行加工，比如通过调用clone() 方法返回 parentValue 的浅拷贝，以达到子线程无法影响父线程的目的。

代码如下：

```java
public class Local extends InheritableThreadLocal {

    @Override
    protected Object initialValue() {
        return new Date();
    }

    @Override
    protected Object childValue(Object parentValue) {
        return parentValue+"[子线程增强版]";  // parentValue.clone();
    }
}
```