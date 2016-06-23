---
layout: post
title:  "数据库连接池"
date:   2016-05-28
author:  
categories: java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 应用程序直接获取数据库连接的缺点

用户每次请求都需要向数据库获得链接，而数据库创建连接通常需要消耗相对较大的资源，创建时间也较长。假设网站一天10万访问量，数据库服务器就需要创建10万次连接，极大的浪费数据库的资源，并且极易造成数据库服务器内存溢出、拓机。

## 使用数据库连接池优化程序性能

### 数据库连接池的基本概念

数据库连接是一种关键的有限的昂贵的资源,这一点在多用户的网页应用程序中体现的尤为突出.对数据库连接的管理能显著影响到整个应用程序的伸缩性和健壮性,影响到程序的性能指标.数据库连接池正式针对这个问题提出来的.==数据库连接池负责分配,管理和释放数据库连接,它允许应用程序重复使用一个现有的数据库连接,而不是重新建立一个==。

数据库连接池在初始化时将创建一定数量的数据库连接放到连接池中, 这些数据库连接的数量是由最小数据库连接数来设定的.无论这些数据库连接是否被使用,连接池都将一直保证至少拥有这么多的连接数量.连接池的最大数据库连接数量限定了这个连接池能占有的最大连接数,当应用程序向连接池请求的连接数超过最大连接数量时,这些请求将被加入到等待队列中.

数据库连接池的最小连接数和最大连接数的设置要考虑到以下几个因素:

- 最小连接数:是连接池一直保持的数据库连接,所以如果应用程序对数据库连接的使用量不大,将会有大量的数据库连接资源被浪费.
-  最大连接数:是连接池能申请的最大连接数,如果数据库连接请求超过次数,后面的数据库连接请求将被加入到等待队列中,这会影响以后的数据库操作
- 如果最小连接数与最大连接数相差很大:那么最先连接请求将会获利,之后超过最小连接数量的连接请求等价于建立一个新的数据库连接.不过,这些大于最小连接数的数据库连接在使用完不会马上被释放,他将被放到连接池中等待重复使用或是空间超时后被释放.

### 编写数据库连接池

编写连接池需实现java.sql.DataSource接口==。DataSource有两个重载的getConnection方法。

    Connection getConnection();
    Connection getConnection(String username, String password);
    
实现DataSource接口，并实现连接池功能的步骤：

1. 在DataSource构造函数中批量创建与数据库的连接，并把创建的连接加入连接池对象中。
2. 实现getConnection方法，让getConnection方法每次调用时，从连接池中取一个Connection返回给用户。
3. 当用户使用完Connection，调用Connection.close()方法时，Collection对象应保证将自己返回到连接池中,而不要把conn还给数据库。==Collection保证将自己返回到连接池中是此处编程的难点。

>以上所说的链接池就是一个存放connection的容器，可使用阻塞队列实现，这样在连接池为空时取连接会被阻塞。

在getConnection时，使用动态代理创建Connection对象的代理对象，并将其返回。动态代理就可实现改变原来对象中的某些行为。


    Connection proxyConn = (Connection) Proxy.newProxyInstance(this
    		.getClass().getClassLoader(), conn.getClass().getInterfaces(),
    		new InvocationHandler() {
    			// 此处为内部类，当close方法被调用时将conn还回池中,其它方法直接执行
    			@Override
    			public Object invoke(Object proxy, Method method,
    					Object[] args) throws Throwable {
    				if (method.getName().equals("close")) {
    					pool.addLast(conn);
    					return null;
    				}
    				return method.invoke(conn, args);
    			}
    		});
    		
## 开源数据库连接池

现在很多WEB服务器(Weblogic,WebSphere,Tomcat)都提供了DataSoruce的实现，即连接池的实现。通常我们把DataSource的实现，按其英文含义称之为数据源，数据源中都包含了数据库连接池的实现。

也有一些开源组织提供了数据源的独立实现：

- DBCP 数据库连接池
- C3P0 数据库连接池

在使用了数据库连接池之后，在项目的实际开发中就不需要编写连接数据库的代码了，直接从数据源获得数据库的连接。

### DBCP数据源

DBCP 是 Apache 软件基金组织下的开源连接池实现，要使用DBCP数据源，需要应用程序应在系统中增加如下两个 jar 文件：

- Commons-dbcp.jar：连接池的实现
- Commons-pool.jar：连接池实现的依赖库

Tomcat 的连接池正是采用该连接池来实现的。该数据库连接池既可以与应用服务器整合使用，也可由应用程序独立使用。

### C3P0数据源

C3P0是一个开源的JDBC连接池，它实现了数据源和JNDI绑定，支持JDBC3规范和JDBC2的标准扩展。目前使用它的开源项目有Hibernate，Spring等。C3P0数据源在项目开发中使用得比较多。

c3p0与dbcp区别：

- dbcp没有自动回收空闲连接的功能
- c3p0有自动回收空闲连接功能

## 参考

[javaweb学习总结(三十九)——数据库连接池](http://www.cnblogs.com/xdp-gacl/p/4002804.html)


