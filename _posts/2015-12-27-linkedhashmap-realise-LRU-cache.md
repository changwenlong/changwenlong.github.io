---
layout: post
title:  "10行Java代码实现LRU缓存"
date:   2015-12-27
author: Changwl
categories: java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}


##介绍

阅读了[LinkedHashMap](/2015/12/26/linkedhashmap "LinkedHashMap")源码之后，发现LinkedHashMap隐藏实现了LRU算法，本文就是用LinkedHashMap来实现LRU缓存。

LRU-Least Recently Used 近期最少使用算法。

##实现

直接上实现。

    public class LRUCache<K,V> extends LinkedHashMap<K,V>{

        private int cacheSize;
    
        //设置accessOrder = true，实现LRU访问
        LRUCache(int initialCapacity,float loadFactor,int cacheSize){
            super(initialCapacity,loadFactor,true);
            this.cacheSize = cacheSize;
        }
    
        //限制缓存的大小
        @override
        protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
            return size > cacheSize;
        }
    
    }



##参考
[10行Java代码实现最近被使用（LRU）缓存](http://www.importnew.com/16264.html "Java 10行Java代码实现最近被使用（LRU）缓存")

[LRU](https://en.wikipedia.org/wiki/LRU "LRU")

