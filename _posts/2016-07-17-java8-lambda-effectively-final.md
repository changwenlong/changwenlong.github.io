---
layout: post
title:  "java8 lambda & effectively final"
date:   2016-07-17
author:  
categories: java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 需求

将已知list中的元素加10，构造一个新的addList

## java8 lambda表达式实现

    @Test
    public void testLambda(String[] args) {
        ArrayList<Integer> list = new ArrayList<Integer>(Arrays.asList(new Integer[]{10,2,3,4,5}));
        ArrayList<Integer> addList = new ArrayList<Integer>();
        list.forEach(i->{addList.add(i+10);});//lambda表达式实现
        System.out.println(addList);
    }
    
## java8 匿名内部类实现

java8的lambda表达式其实就是对只有一个函数的接口的简写

    @Test
    public void testLambda(String[] args) {
        ArrayList<Integer> list = new ArrayList<Integer>(Arrays.asList(new Integer[]{10,2,3,4,5}));
        ArrayList<Integer> addList = new ArrayList<Integer>();
        list.forEach(new Consumer<Integer>() {
            @Override
            public void accept(Integer i) {
                addList.add(i+10);//java8中，匿名内部类中访问局部变量addList不需要加final        
            }
        });
        System.out.println(addList);
    }
    
java8中接口Consumer的定义：

    package java.util.function;
    
    import java.util.Objects;

    @FunctionalInterface
    public interface Consumer<T> {

        void accept(T t);
    
        default Consumer<T> andThen(Consumer<? super T> after) {
            Objects.requireNonNull(after);
            return (T t) -> { accept(t); after.accept(t); };
        }
    }
    
从上面的代码可以看到，匿名内部类中访问了非final类型的addList局部变量，这在java8之前是不允许的。

java8之前，匿名内部类访问局部变量时，局部变量一定要定义为final。

## java7 匿名内部类实现

    //ArrayList<Integer> addList = new ArrayList<Integer>();
    
    @Test
    public void test() {
        ArrayList<Integer> list = new ArrayList<Integer>(Arrays.asList(new Integer[]{10,2,3,4,5}));
        final ArrayList<Integer> addList = new ArrayList<Integer>();
        forEach(list,new Consumer<Integer>() {
            @Override
            public void accept(Integer i) {
                /*
                 * java7中，匿名内部类中访问局部变量addList需要加final;
                 * 但若addList定义为外围类的变量，则无需加final，因为内部类本身都会含有一个外围了的引用（外围类.this）                
                 */
                addList.add(i+10);//
            }
        });
        System.out.println(addList);
    }

    private static void forEach(ArrayList<Integer> list, Consumer<Integer> consumer) {
        for(int i:list){
            consumer.accept(i);
        }        
    }
    
`java7中，匿名内部类中访问局部变量addList需要加final;但若addList定义为外围类的变量，则无需加final，因为内部类本身都会含有一个外围了的引用（外围类.this）。`

## java8之前匿名内部类的参数引用final?

内部类通常都含有回调，引用那个匿名内部类的函数执行完了就没了，所以内部类中引用外面的局部变量需要是final的，这样在回调的时候才能找到那个变量，而如果是外围类的成员变量就不需要是final的，因为内部类本身都会含有一个外围了的引用（外围类.this），所以回调的时候一定可以访问到。

内部类通常都含有回调，引用那个匿名内部类的函数执行完了就没了，所以内部类中引用外面的局部变量需要是final的，这样在回调的时候才能找到那个变量，而如果是外围类的成员变量就不需要是final的，因为内部类本身都会含有一个外围了的引用（外围类.this），所以回调的时候一定可以访问到。


[java为什么匿名内部类的参数引用时final？](https://www.zhihu.com/question/21395848/answer/29665218)

## java8匿名内部类的参数引用不需要final？

Local variable addList defined in an enclosing scope must be final or effectively final.

A variable or parameter whose value is never changed after it is initialized is effectively final.

    @Test
    public void testLambda(String[] args) {
        ArrayList<Integer> list = new ArrayList<Integer>(Arrays.asList(new Integer[]{10,2,3,4,5}));
        ArrayList<Integer> addList = new ArrayList<Integer>();
        list.forEach(i->{
            addList.add(i+10);
            addList=null;//compile error.Local variable addList defined in an enclosing scope must be final or effectively final
            });
        System.out.println(addList);
    }

[Difference between final and effectively final](http://stackoverflow.com/questions/20938095/difference-between-final-and-effectively-final)


