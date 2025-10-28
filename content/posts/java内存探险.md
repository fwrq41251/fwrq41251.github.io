---
title: java内存探险(待续)
date: 2015-10-13
categories:
  - Programming
tags:
  - JVM
---

之前写的b站直播订阅通知的程序内存占用始终不能满意,
想到java是跑在jvm上的,
是否jvm本身就需要占用那么多的内存呢,
决定打开jconsole来看一看.
java内存分为堆和非堆,来看定义:

>##### Heap memory
>
The heap memory is the runtime data area from which the Java VM allocates memory for all class instances and arrays. The heap may be of a fixed or variable size. The garbage collector is an automatic memory management system that reclaims heap memory for objects.
>
* Eden Space: The pool from which memory is initially allocated for most objects.
>
* Survivor Space: The pool containing objects that have survived the garbage collection of the Eden space.
>
* Tenured Generation: The pool containing objects that have existed for some time in the survivor space.

>##### Non-heap memory
>
Non-heap memory includes a method area shared among all threads and memory required for the internal processing or optimization for the Java VM. It stores per-class structures such as a runtime constant pool, field and method data, and the code for methods and constructors. The method area is logically part of the heap but, depending on the implementation, a Java VM may not garbage collect or compact it. Like the heap memory, the method area may be of a fixed or variable size. The memory for the method area does not need to be contiguous.
>
* Permanent Generation: The pool containing all the reflective data of the virtual machine itself, such as class and method objects. With Java VMs that use class data sharing, this generation is divided into read-only and read-write areas.
>
* Code Cache: The HotSpot Java VM also includes a code cache, containing memory that is used for compilation and storage of native code.

我的程序的堆内存占用:
![](/img/561cad5c33358.png)
这个程序每五分钟会去爬一次直播间的直播状态,
在这个点上堆内存占用升高,
GC之后又回落下来,
整体上在6.5M左右徘徊,
问题不大.
再来看非堆:
![](/img/561cae38326d2.png)
整体30M左右,基本上不变化.
***
实际上JAVA程序在启动时可以指定内存占用的大小,

```
-Xms128m -Xmx256m
```

上面这行代码就指定了堆的初始化内存为128MB,最大为256MB.

```
-XX:PermSize256m XX:MaxPermSize512m
```

上面这行代码用来指定非堆内存的永久代(**PermGen**)大小.
然而我试过将MaxPermSize设为20m,
非堆内存占用并没有什么变化.
***
手头上还有一个舰娘的辅助程序,从github上下载的,
来看一下它的内存占用.
堆内存:
![](/img/561cb1413c8a5.png)
非堆:
![](/img/561cb18c012a9.png)
非堆几乎是一样的.

这下可以安心了,自己的程序内存占用是正常的.
参考:
[Configuring JVM Options](https://docs.oracle.com/cd/E22289_01/html/821-1274/configuring-the-default-jvm-and-java-arguments.html)
[How to set the maximum memory usage for JVM](http://stackoverflow.com/questions/1493913/how-to-set-the-maximum-memory-usage-for-jvm)
