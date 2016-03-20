---
layout: post
title:  "虚拟机类加载机制"
date:   2016-03-20
author:  
categories: java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 介绍

虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形
成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

与那些编译时需要进行连接工作的语言不同，在Java中，类型的加载、连接和初始化过程都是在程序运行期间完成，Java里天生可扩展的语言特性就是依赖运行期动态加载和动态连接这个特点实现的。例如，如果编写一个面向接口的应用程序，可以等到运行时再指定其实际的实现类。

## 类加载过程

类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、验证（Verification）、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)和卸载(Unloading)7个阶段。其中准备、验证、解析3个部分统称为连接（Linking）。如图所示。

![runtime data area]({{"/static/imgs/jvm-classloader.png"}})

加载、验证、准备、初始化和卸载这5个阶段的顺序是确定的，类的加载过程必须按照这种顺序按部就班地开始，而解析阶段则不一定：它在某些情况下可以在初始化阶段之后再开始，这是为了支持Java语言的运行时绑定（也称为动态绑定或晚期绑定）。以下陈述的内容都已HotSpot为基准。

### 加载

在加载阶段（可以参考java.lang.ClassLoader的loadClass()方法），虚拟机需要完成以下3件事情：


1. 通过一个类的全限定名来获取定义此类的二进制字节流（并没有指明要从一个Class文件中获取，可以从其他渠道，譬如：网络、动态生成、数据库等）；
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构；
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口；

对于数组类而言，情况有所不同，数组类本身不是通过类加载器创建，它是由Java虚拟机直接创建的。但数组类的元素类型（Element Type，指的是去掉维度的类型）最终还是要靠类加载器去创建。

加载阶段和连接阶段（Linking）的部分内容（如一部分字节码文件格式验证动作）是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始，但这些夹在加载阶段之中进行的动作，仍然属于连接阶段的内容，这两个阶段的开始时间仍然保持着固定的先后顺序。

### 验证

验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。
验证阶段大致会完成4个阶段的检验动作：

1. 文件格式验证：验证字节流是否符合Class文件格式的规范；例如：是否以魔术0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
2. 元数据验证：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了java.lang.Object之外。
3. 字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
符号引用验证：确保解析动作能正确执行。

验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用-Xverifynone参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。

### 准备

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。这时候进行内存分配的仅包括类变量（被static修饰的变量），而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在堆中。其次，这里所说的初始值“通常情况”下是数据类型的零值，假设一个类变量的定义为：

    public static int value=123;

那变量value在准备阶段过后的初始值为0而不是123.因为这时候尚未开始执行任何java方法，而把value赋值为123的putstatic指令是程序被编译后，存放于类构造器()方法之中，所以把value赋值为123的动作将在初始化阶段才会执行。

至于“特殊情况”是指：public static final int value=123，即当类字段的字段属性是ConstantValue时，会在准备阶段初始化为指定的值，所以标注为final之后，value的值在准备阶段初始化为123而非0.

### 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。

### 初始化

类初始化阶段是类加载过程的最后一步，到了初始化阶段，才真正开始执行类中定义的java程序代码。在准备极端，变量已经付过一次系统要求的初始值，而在初始化阶段，则根据程序猿通过程序制定的主观计划去初始化类变量和其他资源，或者说：初始化阶段是执行类构造器&lt;clinit&gt;()方法的过程.

&lt;clinit&gt;()方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块static{}中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问。如下：

    public class Test
    {
        static
        {
            i=0;
            System.out.println(i);//这句编译器会报错：Cannot reference a field before it is defined（非法向前应用）
        }
        static int i=1;
    }

&lt;clinit&gt;()方法与实例构造器&lt;init&gt;()方法不同，它不需要显示地调用父类构造器，虚拟机会保证在子类&lt;cinit&gt;()方法执行之前，父类的&lt;clinit&gt;()方法方法已经执行完毕.因此在虚拟机中第一个被执行&lt;clinit&gt;()方法的类肯定是`java.lang.Object`

由于父类的&lt;clinit&gt;()方法先执行，也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作。

&lt;clinit&gt;()方法对于类或者接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生产&lt;clinit&gt;()方法。

接口中不能使用静态语句块，但仍然有变量初始化的赋值操作，因此接口与类一样都会生成&lt;clinit&gt;()方法。但接口与类不同的是，执行接口的&lt;clinit&gt;()方法不需要先执行父接口的&lt;clinit&gt;()方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的&lt;clinit&gt;()方法。

虚拟机会保证一个类的&lt;clinit&gt;()方法在多线程环境中被正确的加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的&lt;clinit&gt;()方法，其他线程都需要阻塞等待，直到活动线程执行&lt;clinit&gt;()方法完毕。如果在一个类的&lt;clinit&gt;()方法中有好事很长的操作，就可能造成多个线程阻塞，在实际应用中这种阻塞往往是隐藏的。

    static class DeadLoopClass
    {
        static
        {
            if(true)
            {
                System.out.println(Thread.currentThread()+"init DeadLoopClass");
                while(true)
                {
                }
            }
        }
    }
    
    public static void main(String[] args)
    {
        Runnable script = new Runnable(){
            public void run()
            {
                System.out.println(Thread.currentThread()+" start");
                DeadLoopClass dlc = new DeadLoopClass();
                System.out.println(Thread.currentThread()+" run over");
            }
        };
    
        Thread thread1 = new Thread(script);
        Thread thread2 = new Thread(script);
        thread1.start();
        thread2.start();
    }

运行结果：（即一条线程在死循环以模拟长时间操作，另一条线程在阻塞等待）

    Thread[Thread-0,5,main] start
    Thread[Thread-1,5,main] start
    Thread[Thread-0,5,main]init DeadLoopClass

## 类初始化的时机

先看几个例子：

- 通过子类引用父类的静态字段，不会导致子类的初始化

        public class SuperClass 
        {
            static
            {
                System.out.println("SuperClass init!");
            }
         
            public static int value = 123;
        }
    
        public class SubClass extends SuperClass
        {
            static
            {
                System.out.println("SubClass init");
            }
         
        }

        public class NotInitialization
        {
            public static void main(String[] args)
            {
                System.out.println(SubClass.value);
            }
        }

运行结果：

    SuperClass init!

- 通过数组定义来引用类，不会触发类的初始化

        public class NotInitialization
        {
            public static void main(String[] args)
            {
                SuperClass[] sca = new SuperClass[10];
            }
        }

无输出

- 常量在编译阶段会存入调用类的常量池，本质上并没有引用到定义常量的类，因此不会触发定义常量的类的初始化

        public class ConstClass
        {
            static
            {
                System.out.println("ConstClass init!");
            }
            public static  final String HELLOWORLD = "hello world";
        }
        public class NotInitialization
        {
            public static void main(String[] args)
            {
                System.out.println(ConstClass.HELLOWORLD);
            }
        }

运行结果：

    hello world

-----

什么情况下需要开始类加载过程的第一阶段：加载？Java虚拟机规范中并没有进行强制约束，这点由虚拟机的具体实现决定。

但对于初始化阶段，虚拟机规范严格规定了有且只有5中情况（jdk1.7）必须对类进行“初始化”（而加载、验证、准备自然需要在此之前开始）：

1. 遇到new,getstatic,putstatic,invokestatic这失调字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译器把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
2. 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
3. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
5. 当使用jdk1.7动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getstatic,REF_putstatic,REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行初始化，则需要先出触发其初始化。

对于这五种会触发类进行初始化的场景，虚拟机规范中使用了一个很强烈的限定语：“有且只有”，这五种场景中的行为称为对一个类进行主动引用。除此之外，所有引用类的方式都不会触发类的初始化，称为被动引用。上面的3个例子就是典型的被动引用。

## 案例分析

    package jvm.classload;
     
    public class StaticTest
    {
        public static void main(String[] args)
        {
            staticFunction();
        }
     
        static StaticTest st = new StaticTest();
     
        static
        {
            System.out.println("1");
        }
     
        {
            System.out.println("2");
        }
     
        StaticTest()
        {
            System.out.println("3");
            System.out.println("a="+a+",b="+b);
        }
     
        public static void staticFunction(){
            System.out.println("4");
        }
     
        int a=110;
        static int b =112;
    }

问输出是什么？

    2
    3
    a=110,b=0
    1
    4

类的生命周期是：加载-&gt;验证-&gt;准备-&gt;解析-&gt;初始化-&gt;使用-&gt;卸载，只有在准备阶段和初始化阶段才会涉及类变量的初始化和赋值，因此只针对这两个阶段进行分析；

类的准备阶段需要做是为类变量分配内存并设置默认值，因此类变量st为null、b为0；（需要注意的是如果类变量是final，编译时javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将变量设置为指定的值，如果这里这么定义：static final int b=112,那么在准备阶段b的值就是112，而不再是0了。）

类的初始化阶段需要做是执行类构造器（类构造器是编译器收集所有静态语句块和类变量的赋值语句按语句在源码中的顺序合并生成类构造器，对象的构造方法是&lt;init&gt;()，类的构造方法是&lt;clinit&gt;()，可以在堆栈信息中看到），因此先执行第一条静态变量的赋值语句即st = new StaticTest ()，此时会进行对象的初始化，对象的初始化是先初始化成员变量再执行构造方法，因此打印2-&gt;设置a为110-&gt;执行构造方法(打印3,此时a已经赋值为110，但是b只是设置了默认值0，并未完成赋值动作)，等对象的初始化完成后继续执行之前的类构造器的语句，接下来就不详细说了，按照语句在源码中的顺序执行即可。

这里面还牵涉到一个冷知识，就是在嵌套初始化时有一个特别的逻辑。特别是内嵌的这个变量恰好是个静态成员，而且是本类的实例。
这会导致一个有趣的现象：“实例初始化竟然出现在静态初始化之前”。



## 参考

《深入理解Java虚拟机》

[Java虚拟机类加载机制](http://www.importnew.com/18548.html "Java虚拟机类加载机制")


[Java虚拟机类加载机制](http://www.importnew.com/18566.html "Java虚拟机类加载机制")



