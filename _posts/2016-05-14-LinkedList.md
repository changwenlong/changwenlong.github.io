---
layout: post
title:  "LinkedList用作Stack，Queue"
date:   2016-05-15
author:  
categories: 算法编程
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 介绍

LinkedList实现了Stack，Queue的功能，可用作Stack，Queue

## 用作Stack

    LinkedList<TreeNode> s = new LinkedList<TreeNode>();
    q.isEmpty;//判空
    q.push(1);//添加到第一个元素之前
    Integer val=q.peek();//获取最后一个元素
    Integer val=q.pop();//获取最后一个元素,并删除


## 用作Queue

    LinkedList<TreeNode> q = new LinkedList<TreeNode>();
    q.isEmpty;//判空
    q.offer(1);//添加到末尾
    Integer val=q.peek();//获取最后一个元素
    Integer val=q.poll();//获取最后一个元素,并删除


