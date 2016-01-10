---
layout: post
title:  "通过示例学习StackOverflowError和OutOfMemoryError"
date:   2016-01-10
author: Changwl
categories: java jvm
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}


##介绍

`StackOverflow`和`OutOfMemoryError`均继承自`VirtualMachineError`，是虚拟机层面的error，java应用运行时，[JVM运行时数据区](/2016/01/05/jvm-runtime-data-area "JVM运行时数据区")不同区域可能出现这两类error，下面通过示例来观察。


##Java虚拟机栈溢出

若在方法调用的过程中，请求的栈深度大于最大可用的栈深度，则抛出`StackOverflowError`；如果 Java 栈可以动态扩展，而在扩展栈的过程中没有足够的内存空间来支持栈的发展，则抛出 `OutOfMemeoryError`。可以使用`-Xss` 参数来设置栈的大小，栈的大小直接决定了函数调用的可达深度。

> 递归调用显示栈的最大深度

    public class JvmStack {
        public static void main(String[] args) {
            try {
                recursion();
            } catch (Throwable e) {
                System.out.println("deep of stack is "+count);
                e.printStackTrace();
            }
        }
        
        private static int count=0;
        
        public void recursion(){
            count++;
            recursion();
        }
    }

> 输出：
    
    deep of stack is 12571
    java.lang.StackOverflowError
        at edu.zju.chwl.jvm.JvmStack.recursion(JvmStack.java:17)
        at edu.zju.chwl.jvm.JvmStack.recursion(JvmStack.java:18)
        at edu.zju.chwl.jvm.JvmStack.recursion(JvmStack.java:18)
        at edu.zju.chwl.jvm.JvmStack.recursion(JvmStack.java:18)
        at edu.zju.chwl.jvm.JvmStack.recursion(JvmStack.java:18)


##Java堆溢出

在Java堆中创建变量时，若Java堆空间不够，则会抛出`OutOfMemoryError`。

> 创建超过堆大小的对象

    public class JavaHeap {
        /**
         * jvm参数：-Xms20M -Xmx20M
         * -Xms：初始堆大小
         * -Xmx：最大堆大小
         */
        public static void main(String[] args) {        
            try {
                //在堆中创建大小为21M的byte数组
                byte[] b1 = new byte[1024*1024*21];
            } catch (Throwable e) {
                e.printStackTrace();
            }                        
        }
    }

> jvm参数：-Xms20M -Xmx20M，输出:

    java.lang.OutOfMemoryError: Java heap space
        at edu.zju.chwl.jvm.JavaHeap.main(JavaHeap.java:7)



##运行时常量池溢出

运行时常量池是方法区的一部分，受到方法区内存的限制，当常量池无法再申请到内存时会抛出`OutOfMemoryError`异常。

> 将Integer范围内的字符串写入运行时常量池

    public class RuntimeConstantPool {
        /**
         * jvm参数：-XX:PermSize=10M -XX:MaxPermSize=10M  -Xms10M -Xmx10M
         * -XX:PermSize：初始方法区大小
         * -XX:MaxPermSize：最大方法区大小
         */
        public static void main(String[] args) {
            try {
                // 使用List保持着常量池引用，避免GC回收常量池行为
                List<String> list = new ArrayList<String>();
                // 10MB的PermSize在integer范围内足够产生OOM了
                int i = 0;
                while (true) {
                    list.add(String.valueOf(i++).intern());
                }
            } catch (Throwable e) {
                e.printStackTrace();
            }
        }
    }


1. jvm参数：-XX:PermSize=10M -XX:MaxPermSize=10M，此时无任何输出，说明设置方法区大小对输出结果没什么影响。

    笔者电脑的java环境：
    
        C:\Users\chwl>java -version
        java version "1.7.0_51"
        Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
        Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
   
    [jdk7-relnotes](http://www.oracle.com/technetwork/java/javase/jdk7-relnotes-418459.html "jdk7-relnotes") 中可找到相应解释(运行时常量池从jdk7开始从方法区迁移到堆内)：
    
    > Area: HotSpot
    > 
    > Synopsis: In JDK 7, interned strings are no longer allocated in the permanent generation of the Java heap, but are instead allocated in the main part of the Java heap (known as the young and old generations), along with the other objects created by the application. This change will result in more data residing in the main Java heap, and less data in the permanent generation, and thus may require heap sizes to be adjusted. Most applications will see only relatively small differences in heap usage due to this change, but larger applications that load many classes or make heavy use of the String.intern() method will see more significant differences.
    > 
    > RFE: 6962931
       

2. jvm参数：-Xms10M -Xmx10M，输出：

        Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
           at edu.zju.chwl.jvm.RuntimeConstantPool.main(RuntimeConstantPool.java:19)

##直接内存溢出

DirectMemorySize可以通过设置 -XX:MaxDirectMemorySize参数指定容量大小，如果不指定的话，那么就跟堆的最大值一致。

    public class DirectMemoryOOM {
    
        private static final int _1MB = 1024 * 1024;
        /**
         * jvm参数：-XX:MaxDirectMemorySize=10M
         */
        public static void main(String[] args) throws IllegalArgumentException,
                IllegalAccessException {
            Field unsafeField = Unsafe.class.getDeclaredFields()[0];
            unsafeField.setAccessible(true);
            Unsafe unsafe = (Unsafe) unsafeField.get(null);
            while (true) {
                unsafe.allocateMemory(_1MB);
            }
        }
    }

> jvm参数：-XX:MaxDirectMemorySize=10M，输出：

    Exception in thread "main" java.lang.OutOfMemoryError
        at sun.misc.Unsafe.allocateMemory(Native Method)
        at edu.zju.chwl.jvm.DirectMemoryOOM.main(DirectMemoryOOM.java:20)


##总结

通过以上示例的学习，对jvm运行时常量池有了更进一步的认识，示例中使用了一些jvm参数配置来限制各区域的大小，便于尽快出现溢出error。[通过示例观察jvm垃圾回收及jvm参数讲解](/2016/01/11/jvm-gc-arguments "jvm arguments") 将具体介绍java虚拟机参数。


##参考
[从内存溢出看Java 环境中的内存结构](http://www.cnblogs.com/fantiantian/p/3658489.html "从内存溢出看Java 环境中的内存结构")

[jdk7-relnotes](http://www.oracle.com/technetwork/java/javase/jdk7-relnotes-418459.html "jdk7-relnotes")

[《JVM笔记》之一：Java内存区域与内存溢出异常](http://yidao620c.iteye.com/blog/1938886 "《JVM笔记》之一：Java内存区域与内存溢出异常")

