---
layout: post
title:  "java线程中断"
date:   2016-06-03
author:  
categories: java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 线程的interrupt方法可以终止线程吗？

先看下面一段程序：

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new Runnable(){
            @Override
            public void run() {    
                int i=0;
                while(i<100){
                    System.out.println("Thread is running"+i++);
                }
            }
            
        });
        thread.start();
        Thread.sleep(2);
        System.out.println("Before interrupt");
        thread.interrupt();
        System.out.println("thread is interupted:"+thread.isInterrupted());
        System.out.println("After interrupt");
    }


> 输出结果为：

    Thread is running0
    ...
    Thread is running42
    Thread is running43
    Thread is running44
    Before interrupt
    Thread is running45
    thread is interupted:true
    After interrupt
    Thread is running46
    Thread is running47
    ...
    Thread is running99
    
应用程序并不会退出，启动的线程没有因为调用interrupt而终止，可是从调用isInterrupted方法返回的结果可以清楚地知道该线程已经中断了。那位什么会出现这种情况呢？到底是interrupt方法出问题了还是isInterrupted方法出问题了？在Thread类中还有一个测试中断状态的方法（静态的）interrupted，换用这个方法测试，得到的结果是一样的。

## 线程中断的情况下，怎样终止线程？

文章 [Java Thread.interrupt 害人！中断JAVA线程](http://blog.csdn.net/z69183787/article/details/25076119) 详细的说明了应该如何使用interrupt来中断一个线程的执行。 

实际上，在JAVA API文档中对该方法进行了详细的说明。该方法实际上只是设置了一个中断状态，当该线程由于下列原因而受阻时，这个中断状态就起作用了：

1. 如果线程在调用 Object 类的 wait()、wait(long) 或 wait(long, int) 方法，或者该类的 join()、join(long)、join(long, int)、sleep(long) 或 sleep(long, int) 方法过程中受阻，则其中断状态将被清除，它还将收到一个InterruptedException异常。这个时候，我们可以通过捕获InterruptedException异常来终止线程的执行，具体可以通过return等退出或改变共享变量的值使其退出。


        public static void main(String[] args) throws InterruptedException {
            Thread thread = new Thread(new Runnable(){
                @Override
                public void run() {    
                    int i=0;
                    while(i<100){
                        try {
                            Thread.sleep(1);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                            return;//推出循环，终止线程
                        }
                        System.out.println("Thread is running"+i++);
                    }
                }
                
            });
            thread.start();
            Thread.sleep(20);//可以自己调整，主要用来观察输出
            System.out.println("Before interrupt");
            thread.interrupt();
            System.out.println("thread is interupted:"+thread.isInterrupted());
            System.out.println("After interrupt");
        }

    > 运行结果（Thread.sleep(1)响应中断）
    
        ...
        Thread is running17
        Thread is running18
        Thread is running19
        Before interrupt
        thread is interupted:false
        After interrupt
        java.lang.InterruptedException: sleep interrupted
            at java.lang.Thread.sleep(Native Method)
            at edu.zju.chwl.thread.InterruptedDemo$1.run(InterruptedDemo.java:17)
            at java.lang.Thread.run(Thread.java:744)

2. 如果该线程在可中断的通道上的 I/O 操作中受阻，则该通道将被关闭，该线程的中断状态将被设置并且该线程将收到一个 ClosedByInterruptException。这时候处理方法一样，只是捕获的异常不一样而已。

## 通用的终止线程的方法

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new Runnable(){
            @Override
            public void run() {    
                int i=0;
                while(true){
                    System.out.println("Thread is running"+i++);
                    if(Thread.currentThread().isInterrupted()){//判断中断状态，终止线程执行
                        return;
                    }
                }
            }
            
        });
        thread.start();
        Thread.sleep(20);
        System.out.println("Before interrupt");
        thread.interrupt();
        System.out.println("thread is interupted:"+thread.isInterrupted());
        System.out.println("After interrupt");
    }
    
## Thread的interrupt、interrupted与isInterrupted方法

1. `interrupt` interrupt方法用于中断线程。调用该方法的线程的状态为将被置为"中断"状态。
2. `isInterrupted` 判断当前线程是否被中断。
3. `interrupted` interrupted是static方法，用来判断调用者线程是否被中断，并清空中断状态。

        //源码
        public static boolean interrupted() {
            return currentThread().isInterrupted(true);
        }
        public boolean isInterrupted() {
            return isInterrupted(false);
        }
        //使用
        thread.isInterrupted();//判断thread线程
        Thread.interrupted();//判断调用者线程

## 参考
[多线程的使用——中断线程详解(Interrupt)](http://polaris.blog.51cto.com/1146394/372146/)
