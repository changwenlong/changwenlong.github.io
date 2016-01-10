---
layout: post
title:  "通过示例观察jvm垃圾回收及jvm参数讲解"
date:   2016-01-11
author: Changwl
categories: java jvm
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}


##jvm垃圾回收

###介绍

jvm垃圾回收包括Minor GC和Full GC。Minor GC用来回收年轻代中的垃圾，new对象时发现eden区空间不足，就可能触发Minor GC回收年轻代（根据不同的GC算法来定，有时候会直接在老年代new对象）；Full GC用来回收整个java堆以及方法区中的垃圾。


###GC示例

示例代码如下：

    public class MinorGCAndFullGC {
        /**
         * jvm参数：-XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=15 -Xms40M -Xmx40M -Xmn20M
         */
        public static void main(String[] args) {
            //创建对象b1,占用eden区0.5M
            byte[] b1 = new byte[1024 * 1024 / 2];
            //创建对象b2,占用eden区8M
            byte[] b2 = new byte[1024 * 1024 * 8];
            b2 = null;
            //创建对象b3,需要8M的内存空间，eden区所剩区间<=16M-0.5M-8M=7.5M,空间不足，b3对象直接创建在老年代
            byte[] b3 = new byte[1024 * 1024 * 8];
            b3 = null;
            //创建对象b3,需要16M的内存空间，此时eden区所剩
            byte[] b4 = new byte[1024 * 1024 * 16];// Minor GC + Full GC
        }
    }

> jvm参数 -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=15 -Xms40M -Xmx40M -Xmn20M，输出：

    [GC [PSYoungGen: 9688K->1096K(18432K)] 17880K->9288K(38912K), 0.0012052 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [GC [PSYoungGen: 1096K->1064K(18432K)] 9288K->9256K(38912K), 0.0008621 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [Full GC [PSYoungGen: 1064K->0K(18432K)] [ParOldGen: 8192K->1002K(20480K)] 9256K->1002K(38912K) [PSPermGen: 2511K->2510K(21504K)], 0.0080229 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
    Heap
     PSYoungGen      total 18432K, used 983K [0x00000000fec00000, 0x0000000100000000, 0x0000000100000000)
      eden space 16384K, 6% used [0x00000000fec00000,0x00000000fecf5d70,0x00000000ffc00000)
      from space 2048K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x0000000100000000)
      to   space 2048K, 0% used [0x00000000ffc00000,0x00000000ffc00000,0x00000000ffe00000)
     ParOldGen       total 20480K, used 17386K [0x00000000fd800000, 0x00000000fec00000, 0x00000000fec00000)
      object space 20480K, 84% used [0x00000000fd800000,0x00000000fe8fa928,0x00000000fec00000)
     PSPermGen       total 21504K, used 2520K [0x00000000f8600000, 0x00000000f9b00000, 0x00000000fd800000)
      object space 21504K, 11% used [0x00000000f8600000,0x00000000f8876168,0x00000000f9b00000)

###分析

>JVM参数分析：
>
1. -XX:+PrintGCDetails 用来输出jvm GC信息。
2. -XX:SurvivorRatio=8 设置Eden与survivor0的大小比例，此处设为8表示Eden:survivor0:survivor1=8:1:1=16M：2M：2M (年轻代大小为20M)。
3. -XX:MaxTenuringThreshold=15 年轻代中的对象在经过15次垃圾回收后，若还存活，则进入老年代。
4. -Xms40M -Xms40M 初始堆大小和最大堆大小均设为40M。
5. -Xmn20M 年轻代大小设为20M。

------

>代码分析：
>
1. 创建b3对象时，发现eden区间不够，但老年代够用，b3直接创建在老年代，此时并没有触发GC，这样做减少了GC发生的次数，提高了JVM的运行效率。
2. 创建b4对象时，eden区与老年代空间都不够用，先触发Minor GC，对应的GC日志是第一行的GC；回收后发现eden区还是不够用，此时在触发Full GC，对应的GC日志第二行GC加第三行Full GC。
3. GC日志后面是Java堆各区间的使用情况。

------

> *注：分析GC时没有结合jvm的垃圾GC算法进行分析，分析可能不太准确。后续笔记会对GC算法进行总结。

##java堆参数总结

Java 堆操作是主要的数据存储操作，总结的主要参数配置如下。

与 Java 应用程序堆内存相关的 JVM 参数有：

1. -Xms：设置 Java 应用程序启动时的初始堆大小；
2. -Xmx：设置 Java 应用程序能获得的最大堆大小；
3. -Xss：设置线程栈的大小；
4. -XX：MinHeapFreeRatio：设置堆空间最小空闲比例。当堆空间的空闲内存小于这个数值时，JVM 便会扩展堆空间；
5. -XX：MaxHeapFreeRatio：设置堆空间的最大空闲比例。当堆空间的空闲内存大于这个数值时，便会压缩堆空间，得到一个较小的堆；
6. -XX：NewSize：设置新生代的大小；
7. -Xmn：设置新生代大小；
7. -XX：NewRatio：设置老年代与新生代的比例，它等于老年代大小除以新生代大小；
8. -XX：SurvivorRatio：新生代中 eden 区与 survivor 区的比例；
9. -XX：MaxPermSize：设置最大的持久区大小；
10. -XX:MaxDirectMemorySize：设置直接内存大小；
11. -XX:MaxTenuringThreshold：设置新生代中对象进入老年代的年龄；
12. -XX：TargetSurvivorRatio： 设置 survivor 区的可使用率。当 survivor 区的空间使用率达到这个数值时，会将对象送入老年代。

> *注：记得后续跟进补充。

##参考

[Java Major and Minor Garbage Collections](http://stackoverflow.com/questions/16549066/java-major-and-minor-garbage-collections "Java Major and Minor Garbage Collections")

[JVM 数据存储介绍及性能优化](http://www.ibm.com/developerworks/cn/java/j-lo-JVM-Optimize/index.html "JVM 数据存储介绍及性能优化")

