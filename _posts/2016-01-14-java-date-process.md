---
layout: post
title:  "java 日期处理"
date:   2016-01-14
author: Changwl
categories: 备忘
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

##介绍

1.String与Date互转，SimpleDateFormat;

2.Date设置Calendar，`calendar.setTime(Date) `;

3.Calendar实例获取Date,`calendar.getTime()`;

4.Calendar实例获取年、月、日；

5.年、月、日设置Calendar，`calendar.set(year, month, day)`;

6.Calendar实例获取月份的的最后一天，`calendar.getActualMaximum(Calendar.DAY_OF_MONTH)`;

7.Date对象大小比较，`date.before(Date)`，`date.after(Date)`.


##代码

    package edu.zju.chwl.util;

    import java.text.ParseException;
    import java.text.SimpleDateFormat;
    import java.util.Calendar;
    import java.util.Date;
    import java.util.GregorianCalendar;

    import static junit.framework.Assert.*;

    import org.junit.Test;

    public class DateUtilTest {
    
        @Test(expected=ParseException.class)
        public void testSimpleDateFormatNotMatch() throws ParseException {
            new SimpleDateFormat("dd/MM/yyyy").parse("20150516");
        }
        
        @Test
        public void testSimpleDateFormat() throws ParseException {
            new SimpleDateFormat("dd/MM/yyyy").parse("16/05/2015");
        }
        
        @Test
        public void testSimpleDateFormatInvalidDay() throws ParseException {
            SimpleDateFormat dateFormat = new SimpleDateFormat("dd/MM/yyyy");
            String dateStr="44/05/2015";
            if(!dateFormat.format(dateFormat.parse(dateStr)).equals(dateStr)){
                System.out.println(dateStr+" is a illgel date as format "+"dd/MM/yyyy");
            }
        }
        
        @Test
        public void testCalendar() throws ParseException{
            SimpleDateFormat dateFormat = new SimpleDateFormat("dd/MM/yyyy");
            String dateStr = "16/05/2015";
            Date date = dateFormat.parse(dateStr);
            Calendar calendar = new GregorianCalendar();
            calendar.setTime(date);
            int year = calendar.get(Calendar.YEAR);
            int month = calendar.get(Calendar.MONTH);
            int day = calendar.get(Calendar.DAY_OF_MONTH);
            assertEquals(2015, year);
            assertEquals("Month (Index from 0)",4, month);
            assertEquals(16, day);
            calendar.set(year, month,1);
            Date startDate = calendar.getTime();
            assertEquals("Start Date of Month","01/05/2015", dateFormat.format(startDate));
            int endDayOfMonth = calendar.getActualMaximum(Calendar.DAY_OF_MONTH);
            calendar.set(year, month,endDayOfMonth);
            Date endDate=calendar.getTime();
            assertEquals("End Date of Month","01/05/2015", dateFormat.format(startDate));
            assertTrue("Start Date is before End Date",startDate.before(endDate));
            assertTrue("End Date is after Start Date",endDate.after(startDate));
        }
    
    }

##测试结果：绿灯
