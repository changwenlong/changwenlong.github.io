---
layout: post
title:  "排序算法的稳定性"
date:   2016-06-22
author:  
categories: 算法编程
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 8中排序算法的稳定性

算法 | 稳定性 | 算法 | 稳定性
-----| ------ | -----|------ 
冒泡 | 是|选择|否
插入 | 是|快排|否
归并 | 是|希尔|否
基数 | 是|堆排序|否

## java.util.Arrays的排序实现（jdk1.7）

- 对于引用类型数组，采用TimSort。TimSort is a hybrid stable sorting algorithm , derived from merge sort and insert sort.
- 对于primitive数组，采用Dual-Pivot QuickSort。Dual-Pivot QuickSort是QuickSort的变形，选择两个比较值，将元素放在他们之间。

### Why java Arrays use two differt sort algorithm for different types?

1. A stable merge sort for Reference (TimSort);
2. For Primitive , can use quick sort which is not stable (Dual-Pivot QuickSort)

Reason: For primitive, there is no way to distinguish two same value;
