---
layout: post
title:  "【转】java日期处理"
date:   2016-01-14
author:  
categories: java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 介绍

java8之前的日期操作经常接触的几个类，`java.util.Date`，`java.util.Calendar`，`java.text.SimpleDateFormat`，以下将记录使用方法。

## 日期的获取与设置

### Calendar对象的获取

    //获取当前时刻
    Calendar now = Calendar.getInstance();
    //指定时区和地区，也可以只输入其中一个参数
    Calendar now = Calendar.getInstance(timeZone, locale); 
    //获取子类的实例
    Calendar now = new GregorianCalendar();

### 通过Calendar实例获取各种时间数据

    int year = now.get(Calendar.YEAR); //2016，当前年份
    int month = now.get(Calendar.MONTH) + 1; //1，当前月，注意加 1
    int day = now.get(Calendar.DATE); //14，当前日
    Date date = now.getTime(); //直接取得一个 Date 类型的日期
    //获取本月的天数
    int endDayOfMonth = calendar.getActualMaximum(Calendar.DAY_OF_MONTH);

要取得其他类型的时间数据仅需修改 now.get() 内的参数，除了以上三种参数，其他常用参数如下：

- Calendar.DAY_OF_MONTH：日期，和 Calendar.DATE 相同
- Calendar.HOUR：12 小时制的小时数
- Calendar.HOUR_OF_DAY：24小时制的小时数
- Calendar.MINUTE：分钟
- Calendar.SECOND：秒
- Calendar.DAY_OF_WEEK：周几

### 设置日期对象

Calendar是可变的，提供`set`方法来设置日期对象：

    //只设定某个字段的值 
    // public final void set(int field, int value)
    now.set(Calendar.YEAR, 2015);  
    //设定年月日或者年月日时分或年月日时分秒 
    // public final void set(int year, int month, int date[, int hourOfDay, int minute, int second])
    now.set(2016, 1, 1); 
    now.set(2016, 1, 1, 11, 1, 1); 
    //直接传入一个 Date 类型的日期 
    // public final void setTime(Date date)
    now.setTime(date);

注意：

- 当设置了时间参数后，其他相关的数值都会重新计算，例如当你把日期设为 11 号后，周几就会作对应变化。
- 获得的月份加 1 才是实际月份。
- 在 Calendar 类中，周日是 1，周一是 2，以此类推。

## 日期转化

借助`SimpleDateFormat`，实现`Date`与`String`之间的转化：

    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    try { // 日期转字符串
        Calendar calendar = Calendar.getInstance();
        Date date = calendar.getTime();
        String dateStringParse = sdf.format(date);
        // 字符串转日期
        String dateString = "2016-01-01 11:11:11";
        Date dateParse = sdf.parse(dateString);
    } catch (ParseException e) {
        e.printStackTrace();
    }


注意：

- 创建 SimpleDateFormat 对象时必须指定转换格式。
- 转换格式区分大小写，yyyy 代表年份，MM 代表月份，dd 代表日期，HH 代表 24 进制的小时，hh 代表 12 进制的小时，mm 代表分钟，ss 代表秒。

## 日期加减

    //根据现在时间计算
    Calendar now = Calendar.getInstance();  
    now.add(Calendar.YEAR, 1); //现在时间的1年后
    now.add(Calendar.YEAR, -1); //现在时间的1年前 
    //根据某个特定的时间 date (Date 型) 计算
    Calendar specialDate = Calendar.getInstance(); 
    specialDate.setTime(date); //注意在此处将 specialDate 的值改为特定日期
    specialDate.add(Calendar.YEAR, 1); //特定时间的1年后
    specialDate.add(Calendar.YEAR, -1); //特定时间的1年前

> 注意使用了`Calendar`对象的`add`方法，可以更改`Calendar.YEAR`为任意时间单位字段，完成各种时间单位下的日期计算。

### 计算时间差

    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String dateString = "2016-01-01 11:11:11";
    Calendar calendar = Calendar.getInstance();
    long nowDate = calendar.getTime().getTime(); // Date.getTime() 获得毫秒型日期
    try {
        long specialDate = sdf.parse(dateString).getTime();
        long betweenDate = (specialDate - nowDate) / (1000 * 60 * 60 * 24); // 计算间隔多少天，则除以毫秒到天的转换公式
        System.out.print(betweenDate);
    } catch (ParseException e) {
        e.printStackTrace();
    }

## 日期比较

有两种方法实现日期比较：


1. 使用`Date`的`after`，`before`方法
2. 使用`Date`的`compareTo`方法

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateString_01 = "2016-01-01 11:11:11";
        String dateString_02 = "2016-01-02 11:11:11";
        try {
            Date date_01 = sdf.parse(dateString_01);
            Date date_02 = sdf.parse(dateString_02);
            System.out.println(date_01.before(date_02)); // true
            System.out.println(date_02.after(date_01)); // true
            System.out.println(date_01.compareTo(date_02)); // -1
            System.out.println(date_02.compareTo(date_01)); // 1
            System.out.println(date_02.compareTo(date_02)); // 0
        } catch (ParseException e) {
            e.printStackTrace();
        }

## 工具库推荐

Joda-Time，已纳入java8。
强烈建议使用。

[Joda-Time 简介](http://h819.iteye.com/blog/611099 "Joda-Time 简介")

## 转自

[聊聊 Java 中日期的几种常见操作 —— 取值、转换、加减、比较](http://www.kuqin.com/shuoit/20151231/349758.html?url_type=39&object_type=webpage&pos=1 "聊聊 Java 中日期的几种常见操作 —— 取值、转换、加减、比较")
