---
layout: post
title:  "二分查找"
date:   2016-03-31
author:  
categories: 算法编程
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 介绍

二分查找又称折半查找，用来在有序表中进行查找，时间复杂度为O(logn)。


## 算法实现

    //查找target在排序数组nums中的位置，找不到则返回-1
    public int binarySearch(int[] nums,int target){
        if(nums==null||nums.length==0){
            return -1;
        }
        int low = 0;
        int high = nums.length-1;
        while(low<high){
            int mid=(high-low)/2+low;
            if(nums[mid]==target){
                return mid;
            }else if(nums[mid]>target){
                high = mid;
            }else{
                low=mid+1;
            }
        }
        //0<=low<nums.length
        if(nums[low]==target){
            return low;
        }
        return -1;
    }

## 变形题

### 查找有序组中某元素出现的次数

    public int getNumberOfK(int [] array , int k) {
        if(array==null||array.length==0){
            return 0;
        }
        int start = binarySearchFirst(array,k);
        int end = binarySearchLast(array,k);
        return end-start;
    }
    
    //寻找元素k的插入位置，值相等时插在前面    
    private int binarySearchFirst(int[] array,int k){
        if(array==null||array.length==0){
            return -1;
        }
        int start=0,end=array.length-1;
        while(start<end){
            int mid = (end-start)/2+start;
            if(array[mid]>=k){
                end = mid;
            }else{
                start=mid+1;
            }
        }
        if(start==array.length-1&&k>array[start]){
            start++;
        }
        return start;   
    }
    
    //寻找元素k的插入位置，值相等时插在后面
    private int binarySearchLast(int[] array,int k){
        if(array==null||array.length==0){
            return -1;
        }
        int start=0,end=array.length-1;
        while(start<end){
            int mid = (end-start)/2+start;
            if(array[mid]>k){
                end = mid;
            }else{
                start=mid+1;
            }
        }
        if(start==array.length-1&&k>=array[start]){
            start++;
        }
        return start;            
    }

### 查找有序数组中最大的小于target的值

    public int searchLast(int[] array,int k){
        if(array==null||array.length==0){
            return -1;
        }
        int start=0,end=array.length-1;
        while(start<end){
            int mid = (end-start)/2+start;
            if(array[mid]>=k){
                end = mid;
            }else{
                start=mid+1;
            }
        }
        if(start==array.length-1&&k>array[start]){
            start++;
        }
        if(start==0){
            throw new NoSuchElementException();
        }
        return array[start-1];
    }

### 查找有序数组中最小的大于target的值

    public int searchFirst(int[] array,int k){
        if(array==null||array.length==0){
            return -1;
        }
        int start=0,end=array.length-1;
        while(start<end){
            int mid = (end-start)/2+start;
            if(array[mid]>k){
                end = mid;
            }else{
                start=mid+1;
            }
        }
        if(start==array.length-1&&k>=array[start]){
            start++;
        }
        if(start==array.length){
            throw new NoSuchElementException();
        }
        return array[start];
    }



