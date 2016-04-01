---
layout: post
title:  "类加载器"
date:   2016-03-21
author:  
categories: java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 介绍

虚拟机设计团队把类加载阶段中的“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。实现这一动作的代码模块称为“类加载器”。

类加载器在类层次划分、OSGI、热部署、代码加密等领域大放异彩，成为了Java技术体系中一块重要的基石。

## 类与类加载器

类加载器虽然只用于实现类的加载动作,但它在Java程序中起到的作用却远远不限于类加载阶段。对于任意一个类,都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性,每一个类加载器,都拥有一个独立的类名称空间。这句话可以表达得更通俗一些 :比较两个类是否“相等”,只有在这两个类是由同一个类加载器加载的前提下才有意义 ,否则,即使这两个类来源于同一个Class文件 ,被同一个虚拟机加载,只要加载它们的类加载器不同,那这两个类就必定不相等。


    /
     * 简单的类加载器
     * 
     */
    public class ClassLoaderTest {
    
       public static void main(String[] args) throws Exception {
    
           ClassLoader myLoader = new SimpleClassLoader();
    
           Object obj = myLoader.loadClass("edu.zju.chwl.jvm.ClassLoaderTest").newInstance();
    
           System.out.println(obj.getClass());
           System.out.println(obj instanceof edu.zju.chwl.jvm.ClassLoaderTest);
           System.out.println(obj.getClass().getClassLoader());
           System.out.println(edu.zju.chwl.jvm.ClassLoaderTest.class.getClassLoader());
       }
    }
    
    class SimpleClassLoader extends ClassLoader {
        @Override
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            try {
                String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                InputStream is = getClass().getResourceAsStream(fileName);
                if (is == null) {
                    return super.loadClass(name);
                }
                byte[] b = new byte[is.available()];
                is.read(b);
                return defineClass(name, b, 0, b.length);
            } catch (IOException e) {
                throw new ClassNotFoundException(name);
            }
        }
    };

运行结果:

    class edu.zju.chwl.jvm.ClassLoaderTest
    false
    edu.zju.chwl.jvm.SimpleClassLoader@565dd915
    sun.misc.Launcher$AppClassLoader@7692ed85

我们构造了一个简单的类加载器SimpleClassLoader，它可以加载与自己在同一路径下的Class文件。我们使用这个类加载器去加载了一个名为“edu.zju.chwl.jvm.ClassLoaderTest”的类 ,并实例化了这个类的对象。由输出可见，系统类加载器和自定义加载器都对ClassLoaderTest类进行了加载，虚拟机中存在两个ClassLoaderTest，他们相互独立，互不“相等”。

## 双亲委派模型

从Java虚拟机的角度来讲,只存在两种不同的类加载器:一种是启动类加载器 ( Bootstrap ClassLoader ) ,这个类加载器使用C++语言实现,是虚拟机自身的一部分;另—种就是所有其他的类加载器,这些类加载器都由Java语言实现,独立于虚拟机外部,并且全都继承自抽象类java.lang.ClassLoader。

从Java开发人员的角度来看,类加载器还可以划分得更细致一些 ,绝大部分Java程序都会使用到以下3种系统提供的类加载器。

1. 启动类加载器(Bootstrap ClassLoader) : 前面已经介绍过,这个类将器负责将存放在<JAVA_HOME>\lib目录中的,或者被-Xbootclasspath参数所指定的路径中的,并且是虚拟机识别的(仅按照文件名识别,如rt.jar,名字不符合的类库即使放在lib目录中也不会被加载)类库加载到虚拟机内存中。启动类加载器无法被Java程序直接引用,用户在编写自定义类加载器时,如果需要把加载请求委派给引导类加载器,那直接使用null代替即可,如java.lang.ClassLoader.getClassLoader()方法的代码片段所示。

        public ClassLoader getClassLoader() {
            ClassLoader cl = getClassLoader0();
            if (cl == null)
                return null;
            SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                ClassLoader.checkClassLoaderPermission(cl, Reflection.getCallerClass());
            }
            return cl;
        }

2. 扩展类加载器（Extension ClassLoader）：这个加载器由sun.misc.Launcher$ExtClassLoader实现,它负责加载<JAVE_HOME>\lib\ext对目录中的,或者被java.ext.dirs系统变量所指定的路径中的所有类库,开发者可以直接使用扩展类加载器。

3. 应用程序类加载器( Application ClassLoader ) : 这个类加载器由sun.misc.Launcher$AppClassLoader实现。由于这个类加载器是ClassLoader中的getSystemClassLoader()方法的返回值 ,所以一般也称它为系统类加载器。它负责加载用户类路径(ClassPath)上所指定的类库,开发者可以直接使用这个类加载器,如果应用程序中没有自定义过自己的类加载器,一 般情况下这个就是程序中默认的类加载器。

![类加载器双亲委派模型]({{"/static/imgs/classloader.png"}})

上图中展示的类加载器之间的这种层次关系,称为类加载器的双亲委派模型(Parents Delegation Model ) 。 双亲委派模型要求除了顶层的启动类加载器外,其余的类加载器都应当有自己的父类加载器。这里类加载器之间的父子关系一般不会以继承(Inheritance )的关系来实现,而是都使用组合(Composition)关系来复用父加载器的代码。

类加载器的双亲委派模型在JDK 1.2期间被引入并被广泛应用于之后几乎所有的Java程序中 ,但它并不是一个强制性的约束模型,而是Java设计者推荐给开发者的一种类加载器实现方式。

双亲委派模型的工作过程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。

### 使用双亲委派模型的好处

使用双亲委派模型来组织类加载器之间的关系,有一个显而易见的好处就是Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如类java.lang.Object,它存放在rt.jar之中,无论哪一个类加载器要加载这个类,最终都是委派给处于模型最顶端的启动类加载器进行加载,因此Object类在程序的各种类加载器环境中都是同一个类。相反 ,如果没有使用双亲委派模型,由各个类加载器自行去加载的话,如果用户自己编写了一个称为 java.lang.Object的类 ,并放在程序的ClassPath中 ,那系统中将会出现多个不同的Object类 ,Java类型体系中最基础的行为也就无法保证,应用程序也将会变得一片混乱。如果读者有兴趣的话,可以尝试去编写一个与rt.jar类库中已有类重名的Java类 ,将会发现可以正常编译 ,但永远无法被加载运行。

### ClassLoader如何实现双亲委派？

双亲委派模型对于保证Java程序的稳定运作很重要,但它的实现却非常简单,实现双亲委派的代码都集中在java.lang.ClassLoader的loadClass()方法之中,如以下代码片段所示：

    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

逻辑清晰易懂:先检查是否已经被加载过,若没有加载则调用父加载器的loadClass()方法 ,若父加载器为空则默认使用启动类加载器作为父加载器。如果父类加载失败,拋出 ClassNotFoundException异常后,再调用自己的findClass()方法进行加载。

这里只限于HotSpot , 像MRP、Maxife等虚拟机,整个虚拟机本身都是由Java编写的,自然Bootstrap ClassLoader也是由Java语言而不是C++实现的。退一步讲,除了HotSpot以外的其他两个高能虚拟机JRockit和J9都有一个代表Bootstrap ClassLoader的Java类存在,但是关键方法的实现仍然是使用JNI回调到C ( 注意不是C++ ) 的实现上,这个Bootstrap ClassLoader的实例也无法被用户获取到。

即使自定义了自己的类加载器,强行用defineClass()方法去加载以“java.lang”开头的类也不会成功。如果尝试这样做的话,将会收到一个由虚拟机自己拋出的“java.lang.SecurityException : Prohibited package name :java.lang”异常。


### 破坏双亲委派模型

上文提到过双亲委派模型并不是一个强制性的约束模型,而是Java设计者推荐给开发者的类加载器实现方式。在Java的世界中大部分的类加载器都遵循这个模型,但也有例外,到目前为止,双亲委派模型主要出现过3较大规模的“被破坏”情况。

双亲委派模型的第一次“被破坏”其实发生在双亲委派模型出现之前——即JDK1.2发布之前。由于双亲委派模型在JDK 1.2之后才被引入,而类加载器和抽象类java.lang.ClassLoader则在JDK 1.0时代就已经存在,面对已经存在的用户自定义类加载器的实现代码,Java设计者引入双亲委派模型时不得不做出一些妥协。为了向前兼容, JDK 1.2之后的java.lang.ClassLoader添加了一个新的protected方法findClass()（用户自定义加载类逻辑）, 在此之前,用户去继承java.lang.ClassLoader的唯一目的就是为了重写loadClass()方法（实现双亲委派模型） ,因为虚拟机在进行类加载的时候会调用加载器的私有方法loadClassInternal() 这个方法的唯一逻辑就是去调用自己的loadClass()。

上一节我们已经看过loadClass()方法的代码,双亲委派的具体逻辑就实现在这个方法之中, JDK 1.2之后已不提倡用户再去覆盖loadClass()方法,而应当把自己的类加载逻辑写到findClass()方法中,在loadClass()方法的逻辑里如果父类加载失败,则会调用自己的findClass()方法来完成加载,这样就可以保证新写出来的类加载器是符合双亲委派规则的。

双亲委派模型的第二次“被破坏”是由这个模型自身的缺陷所导致的,双亲委派很好地解决了各个类加载器的基础类的统一问题(越基础的类由越上层的加载器进行加载),基础类之所以称为“基础” ,是因为它们总是作为被用户代码调用的A PI,但世事往往没有绝对的完美 ,如果基础类又要调用回用户的代码,那该怎么办?

这并非是不可能的事情,一个典型的例子便是JNDI服务 ,JNDI现在已经是Java的标准服务,它的代码由启动类加载器去加载(在JDK 1.3时放进去的rt.jar) ,但JNDI的目的就是对资源进行集中管理和查找,它需要调用由独立厂商实现并部署在应用程序的ClassPath下的JNDI接口提供者(SH,Service Provider Interface)的代码,但启动类加载器不可能“认识”这些 代码啊!那该怎么办?

为了解决这个问题,Java设计团队只好引入了一个不太优雅的设计:线程上下文类加载器(Thread Context ClassLoader)。这个类加载器可以通过java.lang.Thread类的setContextClassLoaser()方法进行设置,如果创建线程时还未设置,它将会从父线程中继承一个 ,如果在应用程序的全局范围内都没有设置过的话,那这个类加载器默认就是应用程序类加载器。

有了线程上下文类加载器,就可以做一些“舞弊” 的事情了,JNDI服务使用这个线程上下文类加载器去加载所需要的SPI代码 ,也就是父类加载器请求子类加载器去完成类加载的动作 ,这种行为实际上就是打通了双亲委派模型的层次结构来逆向使用类加载器,实际上已经违背了双亲委派模型的一般性原则,但这也是无可奈何的事情。Java中所有涉及SPI的加载动作基本上都采用这种方式,例如JNDI、JDBC、JCE、JAXB和JBI等。

双亲委派模型的第三次“被破坏”是由于用户对程序动态性的追求而导致的,这里所说的“动态性”指的是当前一些非常“热门”的名词:代码热替换(HotSwap) 、模块热部署(Hot Deployment)等 ,说白了就是希望应用程序能像我们的计算机外设那样,接上鼠标、U盘 ,不用重启就能立即使用，鼠标有问题或要升级就换个鼠标,不用停机也不用重启。对于个人计算机来说,重启一次其实没有什么大不了的,但对于一些生产系统来说,关机重启一次可能就要被列为生产事故,这种情况下热部署就对软件开发者,尤其是企业级软件开发者具有很大的吸引力。

Sun公司所提出的JSR-294、JSR-277规范在与JCP组织的模块化规范之争中落败给JSR-291(即OSGi R4.2),虽然Sun不甘失去Java模块化的主导权,独立在发展Jigsaw项目,但目前OSGi已经成为了业界“事实上”的Java模块化标准,而OSGi实现模块化热部署的关键则是它自定义的类加载器机制的实现。每一个程序模块(OSGi中称为Bundle)都有一个自己的类加载器,当需要更换一个Bundle时,就把Bundle连同类加载器一起换掉以实现代码的热替换。

在OSGi环境下,类加载器不再是双亲委派模型中的树状结构,而是进一步发展为更加复杂的网状结构,当收到类加载请求时,OSGi将按照下面的顺序进行类搜索:

1. 将以java.*开头的类委派给父类加载器加载。
2. 否则,将委派列表名单内的类委派给父类加载器加载。
3. 否则,将Import列表中的类委派给Export这个类的Bundle的类加载器加载。
4. 否则,查找当前Bundle的ClassPath, 使用自己的类加载器加载。
5. 否则,查找类是否在自己的Fragment Bundle中,如果在,则委派给Fragment Bundle的类加载器加载。
6. 否则,查找Dynamic Import列表的Bundle, 委派给对应Bundle的类加载器加载。
7. 否则,类查找失败。

上面的查找顺序中只有开头两点仍然符合双亲委派规则,其余的类查找都是在平级的类加载器中进行的。

笔者虽然使用了“被破坏”这个词来形容上述不符合双亲委派模型原则的行为,但这里“被破坏”并不带有贬义的感情色彩。只要有足够意义和理由,突破已有的原则就可认为是一种创新。正如OSGi中的类加载器并不符合传统的双亲委派的类加载器,并且业界对其为了实现热部署而带来的额外的高复杂度还存在不少争议,但在Java程序员中基本有一个共识 :OSGi中对类加载器的使用是很值得学习的,弄懂了OSGi的实现,就可以算是掌握了类加载器的精髓。

## 自定义类加载器

首先在了解一下ClassLoader这个类：

    package java.lang;
    
    public abstract class ClassLoader {
    
        protected final Class<?> defineClass(String name, byte[] b, int off, int len);

        protected Class<?> findClass(String name);

        public Class<?> loadClass(String name);
        
        protected final void resolveClass(Class<?> c) 
    }

defineClass用来将byte字节流解析成JVM能够识别的Class对象；

loaderClass包含双亲委派的具体逻辑，对于无法通过父类加载器加载的类，调用findClass来完成；

findClass由集成ClassLoader的自定义加载其实现；

resolveClass用来对Class进行连接解析。

### 自定义类加载器实例

以下代码片段实现一个用来加载远程Class文件的类加载器：

    package edu.zju.chwl.jvm;
    
    public class NetClassLoader extends ClassLoader {
    
        @Override
        protected Class<?> findClass(String url) throws ClassNotFoundException {
            //父类加载器加载不了时，由次加载器加载
            byte[] classData = getDataFormNet(url);
            if(classData==null){
                throw new ClassNotFoundException();
            }else{
                return defineClass(url,classData,0,classData.length);
            }
        }
    
        private byte[] getDataFormNet(String url) {
            //根据url从远程网络获取Class文件的字节数组
            return null;
        }
    
        
    }

### 如何保证自定义类加载器符合双亲委派模型？

上一节我们已经看过loadClass()方法的代码,双亲委派的具体逻辑就实现在这个方法之中, JDK 1.2之后已不提倡用户再去覆盖loadClass()方法,而应当把自己的类加载逻辑写到findClass()方法中,在loadClass()方法的逻辑里如果父类加载失败,则会调用自己的findClass()方法来完成加载,这样就可以保证新写出来的类加载器是符合双亲委派规则的。

## 参考

《深入理解Java虚拟机》

[深入浅出ClassLoader](http://ifeve.com/classloader/ "你真的了解ClassLoader吗？")

[深入分析Java ClassLoader原理](http://blog.csdn.net/xyang81/article/details/7292380 "深入分析Java ClassLoader原理")



