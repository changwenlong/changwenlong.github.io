---
layout: post
title:  "java中的线程池"
date:   2016-05-21
author:  
categories: java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 合理使用线程池的好处
1. **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
2. **提高响应速度**。当任务到达时，任务不需要等待线程创建就能立即执行。
3. **提高线程的可管理性**。使用线程池可以对线程进行统一的分配、调优和监控。


## 线程池的实现原理

当向线程池提交一个任务之后，线程池是如何处理这个任务的呢？

### 线程池的处理流程


```
graph TD
A[提交任务]-->B{核心线程池是否已满}
B-->|否|D[创建线程执行任务]
B-->|是|C{队列是否已满}
C-->|是|E{线程池是否已满}
C-->|否|F[将任务放入队列]
E-->|是|G[按照策略处理无法执行的任务]
E-->|否|H[创建线程执行任务]
```

ThreadPoolExecutor执行execute方法分四种情况，就是图中的四个叶节点。

### 工作线程

线程池在创建线程时，会将线程封装成工作线程Worker,Worker在执行完任务后，还会循环获取工作队列中的任务来执行。我们可以从Worker类的run()方法里看到。

线程池执行任务分两种：
1. 在execute()方法中创建一个线程是，会让这个线程执行当前任务。
2. 线程执行完当前任务后，会反复从工作队列(BlockingQueue)获取任务来执行。


## 线程池的使用

### 线程池的创建

    new ThreadPoolExecutor(corePoolSize,maximumPoolSize,keepAliveTime,milliseconds,runnableTaskQueue,handler);
    
创建一个线程池时需要以下几个参数。

1. keepAliveTime:线程池的工作线程空闲后，保持存活的时间。
2. RejectedExecutionHandler(饱和策略)：当队列和线程池都满了，说明线程处于饱和状态，那么必须采用一种策略处理提交的新任务。默认为AbortPolicy。JDK 1.5中Java线程池框架提供以下4种策略。
    - AbortPolicy:直接抛出异常。
    - CallerRunsPolicy:使用调用者所在线程来运行任务。
    - DiscardOldestPolicy:丢弃队列里最近的一个任务，并执行当前任务。
    - DiscardPolicy:不处理，丢弃掉。
3. runnableTaskQueue(任务队列)：用来保存等待执行的任务的阻塞队列。有以下几种选择。
    - ArrayBlockingQueue：一个基于数组的有界阻塞队列，按FIFO原则对元素进行排序。
    - LinkedBlockingQueue：一个基于链表的阻塞队列，按FIFO排序元素。吞吐量优于- ----   - 
    - ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用这个队列。
    - SynchronousQueue：一个不存储元素的队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量优于LinkedBlockingQueue。静态工厂方法Executors.newCachedThreadPool使用这个队列。
    - PriorityBlockingQueue：一个具有优先级的无界阻塞队列。

### 关闭线程池

可通过调用线程池的shutdown或shutdownNow方法来关闭线程池。他们的原理是遍历线程池中的工作线程，然后逐个调用interrupt方法来中断线程，`所以无法响应中断的任务可能永远无法终止`。

两者区别：

1. 调用shutdownNow首先将线程池设置为STOP状态，然后interrupt所有工作线程，并返回等待返回等待执行任务的列表。
2. 调用shutdown将线程池设置为SHUTDOWN状态，然后interrupt所有没有正在执行任务的工作线程。

只要调用流入两个关闭方法中的任意一个，isShutdown方法就返回true。当所有的工作线程都已关闭后，才表示线程池成功关闭，这时调用isTerminal方法才返回true。

