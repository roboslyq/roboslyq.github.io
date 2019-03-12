---
layout: post
title:  "Java基础(1)-ThreadLocal原理"
categories: Java基础
tags:  Java基础 ThreadLocal
author: roboslyq
---

* content
{:toc}
# 1.ThreadLocal简介

`ThreadLocal`是从JDK1.2就开始有的一个类。在源码注释中有下面一段话：

```
this class provides thread-local variables.  These variables differ from
their normal counterparts in that each thread that accesses one (via its
{@code get} or {@code set} method) has its own, independently initialized
copy of the variable.  {@code ThreadLocal} instances are typically private
static fields in classes that wish to associate state with a thread (e.g.,
a user ID or Transaction ID).
```

简单翻译过来有以下几点：

- 这个类提供了线程局部(变量)，线程局部变量可以理解为**线程** +**局部** 这两部分。首先是线程，这个变量有有效范围是线程，即不同线程间可以共享。其次是局部，即相对线程来说是局部的，即不同线程间变量是安全。
- 在同一线程中，任何地方的方法调用可以方便的获取当前线程保存在`ThreadLocal`中的值。而不用通过函数传参。
- 每个线程都保持对其线程局部变量副本的隐式引用，只要线程是活动的并且 `ThreadLocal` 实例是可访问的；在线程消失之后，其线程局部实例的所有副本都会被垃圾回收（除非存在对这些副本的其他引用）。

# 2. 代码演示

```java
package com.roboslyq.jdk.lang;
public class ThreadLocalTest {
   private final static ThreadLocal<String> THREAD_LOCAL_1 = new ThreadLocal<String>();
   private final static ThreadLocal<String> THREAD_LOCAL_2 = new ThreadLocal<String>();
    public static void main(String[] args) throws InterruptedException {
        //模拟不同的线程赋值和取值
        for(int i=0;i<2;i++){
            new Thread(
                    new Runnable() {
                        public void run() {
                            ThreadLocalTest.getThreadLocal1().set("currentThead -- 								THREAD_LOCAL_1 -- " + Thread.currentThread().getId());
                            ThreadLocalTest.getThreadLocal2().set("currentThead -- 								THREAD_LOCAL_2 -- " + Thread.currentThread().getName());
                            Print print = new Print();
                            print.print();
                        }
                    }
            ).start();
        }
        //等待1秒，让线程在控制台完成打印
        Thread.sleep(1000);
    }
    public static ThreadLocal<String> getThreadLocal1() {
        return THREAD_LOCAL_1;
    }
    public static ThreadLocal<String> getThreadLocal2() {
        return THREAD_LOCAL_2;
    }
}
//单独一个类，模拟取值
class Print{
    //从ThreadLocal载体(ThreadLocalTest)中，获取对应的ThreadLocal保持的值
    public void print(){
        System.out.println(ThreadLocalTest.getThreadLocal1().get());
        System.out.println(ThreadLocalTest.getThreadLocal2().get());
    }
}
```

**打印日志**

```
currentThead -- THREAD_LOCAL_1 -- 11
currentThead -- THREAD_LOCAL_2 -- Thread-0
currentThead -- THREAD_LOCAL_1 -- 12
currentThead -- THREAD_LOCAL_2 -- Thread-1
Process finished with exit code 0
```

# 3. Set过程

![set](https://roboslyq.github.io/images/java-core/thread-material/set.jpg)

 	1. 调用ThreadLocal.set(T value)方法，在方法内部通过Thread.currentThread获取到当前线程的引用。
 	2. 通过当前线程(Current Thread)来获取线程中保存的ThreadLocalMaps引用，然后从ThreadLocalMaps中以ThreadLocal为Key获取具体的ThreadLocalMap。如果是首次调，则获取到的ThreadLocalMap为Null,此时调用createMap(t, value)创建一个新的对象。如果不是第一次，则ThreadLocalMap不为空，直接保存即可。

# 4. get过程

![get](https://roboslyq.github.io/images/java-core/thread-material/get.jpg)

1. 调用ThreadLocal.set(T value)方法，在方法内部通过Thread.currentThread获取到当前线程的引用。
2. 通过当前线程(Current Thread)来获取线程中保存的ThreadLocalMaps引用，然后从ThreadLocalMaps中以ThreadLocal为Key获取具体的ThreadLocalMap。如果是首次调，则获取到的ThreadLocalMap为Null。如果已经正常赋值，则ThreadLocalMap不为空，直接获取返回值即可。

# 5.数据结构

![data_struct](https://roboslyq.github.io/images/java-core/thread-material/data_struct.jpg)



1、ThreadLocal定义为某个类(载体类)类变量，所有线程共享一个ThreadLocal（上图中的ThreadLocal1和ThreadLocal2）。

2、ThreadLocal中定义了一个内部类ThreadLocalMap,此Map没有实现Map接口，是内部自定义的Map结构。

3、线程Thread有一个类变量叫ThreadLocalMap。即每个线程有自己独享的ThreadLocalMap。而ThreadLocalMap的key为ThreadLocal对象。

简单理解原理就是：每一个线程都有自己独立的ThreadLocal.ThreadLocalMap。然后在线程中的任何地方可以通过Thread.CurrentThread获得当前线程，从而获取当前线程的ThreadLocalMap。进而通过ThreadLocalMap的Key(ThreadLocal)获得对应的值。

并且由于Thread是持有ThreadLocalMap引用，所以一个Thread可以有多个ThreadLocal。

总结：每个Thread中持有ThreadLocalMap引用。当调用`ThreadLocal.set(T value)`等价于调用`Thread.currentThread.ThreadLocalMap.set(this,T value)`。而调用`ThreadLocal.get()`方法时,等价于调用`Thread.currentThread.ThreadLocalMap.get(this)`。

# 6. 内存溢出

**TODO**



# 参考资料

[ThreadLocal-面试必问深度解析](https://www.jianshu.com/p/98b68c97df9b)