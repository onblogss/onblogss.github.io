---
layout: post
title: Java中数组和集合的foreach操作编译后究竟是啥
date: 2019-04-25 13:02:00
author: 薛师兄
tags: Java
---
今天和同事在关于foreach编译后是for循环还是迭代器有了不同意见，特做了个Demo，了解一下。

是啥自己来看吧！

```java
public class Demo {
    public static void main(String[] args) {
        int[] ints = new int[5];
        for (int s : ints) {
            System.out.println(s);
        }

        String[] intss = new String[5];
        for (String s : intss) {
            System.out.println(s);
        }

        List<String> strings = new ArrayList<>();
        for (String s : strings) {
            System.out.println(s);
        }

        Map<String,String> map = new HashMap<>();
        for (Map.Entry e: map.entrySet()) {
            System.out.println(e.getKey());
        }

    }
}
```

编译后查看的源码：

```java
public class Demo {
    public Demo() {
    }

    public static void main(String[] args) {
        int[] ints = new int[5];
        int[] var2 = ints;
        int var3 = ints.length;

        int var4;
        int s;
        for(var4 = 0; var4 < var3; ++var4) {
            s = var2[var4];
            System.out.println(s);
        }

        String[] intss = new String[5];
        String[] var8 = intss;
        var4 = intss.length;

        for(s = 0; s < var4; ++s) {
            String s = var8[s];
            System.out.println(s);
        }

        List<String> strings = new ArrayList();
        Iterator var10 = strings.iterator();

        while(var10.hasNext()) {
            String s = (String)var10.next();
            System.out.println(s);
        }

        Map<String, String> map = new HashMap();
        Iterator var13 = map.entrySet().iterator();

        while(var13.hasNext()) {
            Entry e = (Entry)var13.next();
            System.out.println(e.getKey());
        }

    }
}
```

总结：

foreach对于数组来说依旧是for循环，对List、Map集合来说是迭代器。