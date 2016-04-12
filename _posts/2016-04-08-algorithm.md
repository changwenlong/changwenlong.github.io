---
layout: post
title:  "网易算法题"
date:   2016-04-08
author:  
categories: 算法编程
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 介绍

以下三题是网易2016实习研发工程师编程题，可在[牛客网](http://www.nowcoder.com/test/1429468/summary "网易2016实习研发工程师编程题")上练习。

## 题目1：比较重量

小明陪小红去看钻石，他们从一堆钻石中随机抽取两颗并比较她们的重量。这些钻石的重量各不相同。在他们们比较了一段时间后，它们看中了两颗钻石g1和g2。现在请你根据之前比较的信息判断这两颗钻石的哪颗更重。
给定两颗钻石的编号g1,g2，编号从1开始，同时给定关系数组vector,其中元素为一些二元组，第一个元素为一次比较中较重的钻石的编号，第二个元素为较轻的钻石的编号。最后给定之前的比较次数n。请返回这两颗钻石的关系，若g1更重返回1，g2更重返回-1，无法判断返回0。输入数据保证合法，不会有矛盾情况出现。

测试样例：

>输入：2,3,[[1,2],[2,4],[1,3],[4,3]],4
>
>输出：1

### 思路

此题可以用图论的知识求解。把每颗砖石当成是图中的一个节点，钻石1比钻石2重可以表示为从钻石1指向钻石2的一条边，这样我们就可以根据已知构造出图，图由链接链表表示。

得到图的链接链表后，在遍历需要比较大小两个节点（记为g1,g2）子节点,若g2是g1的子节点，则g1>g2,返回1；若g1是g2的子节点，则g1<g2,返回-1；否则无法判断大小，返回0。

### 实现

    public int cmp(int g1, int g2, int[][] records, int n) {
        if (n == 0) {
            return 0;
        }
        //图的邻接链表，构造图
        Map<Integer, Set<Integer>> lists = new HashMap<Integer, Set<Integer>>();
        for (int i = 0; i < n; i++) {
            if(records[i][0]==records[i][1]){
                continue;
            }
            Set<Integer> list = lists.get(records[i][0]);
            if (list == null) {
                list = new HashSet<Integer>();
            }
            list.add(records[i][1]);
            lists.put(records[i][0], list);
        }
        LinkedList<Integer> q = new LinkedList<Integer>();
        q.add(g1);
        if (searchChild(g2, lists, q)) {
            return 1;
        }
        q.clear();
        q.add(g2);
        if (searchChild(g1, lists, q)) {
            return -1;
        }
        return 0;
    }

    //遍历子节点
    public boolean searchChild(int g1, Map<Integer, Set<Integer>> lists,
            LinkedList<Integer> q) {
        while (!q.isEmpty()) {
            Integer value = q.poll();
            Set<Integer> list = lists.get(value);
            if (list != null) {
                for (Integer item : list) {
                    if (item == g1) {
                        return true;
                    }
                    q.add(item);
                }
            }

        }
        return false;
    }

### 相关题目

室友之前面百度，提到问过一个算法，即给定一组数的大小关系，要求将这组数排好序。

思考了一下，也可以把每个数当做图中的一个节点，大小关系则用来构造图中的边，所构造出的有向无环图（DAG）的拓扑排序即为我们要的排序。

求图的拓扑排序：可以在构造图的同时，用一个Map保存各节点的入度。遍历Map，得到入度为0的节点，放入队列中。

1. 若队列非空，将队列中的队首节点（node）出队。
2. 将node节点的子节点的入度减1，再将入度为0的节点入队；
3. 重复1；




## 题目2：二叉树

有一棵二叉树，树上每个点标有权值，权值各不相同，请设计一个算法算出权值最大的叶节点到权值最小的叶节点的距离。二叉树每条边的距离为1，一个节点经过多少条边到达另一个节点为这两个节点之间的距离。
给定二叉树的根节点root，请返回所求距离。

### 思路

### 实现

### 相关题目


## 题目3：寻找第K大

有一个整数数组，请你根据快速排序的思路，找出数组中第K大的数。
给定一个整数数组a,同时给定它的大小n和要找的K(K在1到n之间)，请返回第K大的数，保证答案存在。

测试样例：

>输入：[1,3,5,2,2],5,3
>
>输出：2

### 思路

### 实现

### 相关题目
