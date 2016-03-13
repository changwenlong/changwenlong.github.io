---
layout: post
title:  "LinkedHashMap工作原理"
date:   2015-12-26
author: Changwl
categories: java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 环境
    
    C:\Users\chwl>java -version
    java version "1.7.0_51"
    Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
    Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)

## 介绍

了解了HashMap的工作原理后，理解LinkedHashMap就简单多了，HashMap底层的数据结构采用哈希表，LinkedHashMap在此基础上加上了循环双向链表的数据结构，存储节点上多加了两个指针before，after。

循环双向链表以及存储节点的定义。

    //循环双向链表
    private transient Entry<K,V> header;

    //存储节点定义 （实现了循环双向链表）
    private static class Entry<K,V> extends HashMap.Entry<K,V> {
        // 顺序遍历时需要用到的双向链表指针
        Entry<K,V> before, after;

        Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
            super(hash, key, value, next);
        }

        //删除双向链表中的当前节点
        private void remove() {
            before.after = after;
            after.before = before;
        }

        //头插法
        private void addBefore(Entry<K,V> existingEntry) {
            after  = existingEntry;
            before = existingEntry.before;
            before.after = this;
            after.before = this;
        }

        //记录访问
        void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            //若为LRU访问顺序，则删除再插入
            if (lm.accessOrder) {
                lm.modCount++;
                remove();
                addBefore(lm.header);
            }
        }

        //删除节点
        void recordRemoval(HashMap<K,V> m) {
            remove();
        }
    }   

LinkedHashMap根据变量`accessOrder`的值提供两种遍历顺序：

    private final boolean accessOrder;

1.`accessOrder = false`，按插入顺序遍历(默认情况);

2.`accessOrder = true`，按LRU算法顺序遍历。

LinkedHashMap继承自HashMap，通过覆盖HashMap的部分方法来实现顺序遍历。

接下来依旧按照增删该查等方面来看LinkedHashMap的实现。

## 签名

    public class LinkedHashMap<K,V>
        extends HashMap<K,V>
        implements Map<K,V>

## 创建LinkedHashMap

1.构造函数

调用父类HashMap的构造函数，初始化遍历顺序（插入顺序）。

    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }

    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }

    public LinkedHashMap() {
        super();
        accessOrder = false;
    }

    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super(m);
        accessOrder = false;
    }

    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }


## 新增与修改

LinkedHashMap对元素进行新增与修改时，不仅要维护哈希表`table`，还要维护循环双向链表`header`

1.源码

    //父类的put方法
    public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                //记录访问，LinkedHashMap.Entry中实现
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
    //覆盖父类addEntry方法
    void addEntry(int hash, K key, V value, int bucketIndex) {
        super.addEntry(hash, key, value, bucketIndex);

        // 限制LinkedHashMap元素的个数，对于LRU访问顺序，删除最久未访问的元素
        Entry<K,V> eldest = header.after;
        if (removeEldestEntry(eldest)) {
            removeEntryForKey(eldest.key);
        }
    }
    
    //可在LinkedHashMap的子类中覆盖，实现基于LRU淘汰算法的定长缓存
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }

    final Entry<K,V> removeEntryForKey(Object key) {
        if (size == 0) {
            return null;
        }
        int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                modCount++;
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                //双向链表元素删除
                e.recordRemoval(this);
                return e;
            }
            prev = e;
            e = next;
        }

        return e;
    }

    //覆盖自父类HashMap，为哈希表和双向链表插入元素
    void createEntry(int hash, K key, V value, int bucketIndex) {
        HashMap.Entry<K,V> old = table[bucketIndex];
        Entry<K,V> e = new Entry<>(hash, key, value, old);
        table[bucketIndex] = e;
        e.addBefore(header);
        size++;
    }
    
2.分析

元素的新增与修改都在`put`方法中实现，`put`方法先对元素的key值进行`hash`与`equals`比较，来判定是新增元素还是修改元素。

新增与修改都需要对哈希表及双向链表进行处理，具体的处理方法在上面的源码部分贴出。

还有一点要提出来，就是当entry数量大于定义的阀值时，会考虑对元素进行`resize`,resize通过`transfer`方法来实现。

LinkedHashMap覆盖了父类`transfer`的实现
    
    //不再使用HashMap中遍历table来实现resize，而是通过遍历双向链表header实现
    @Override
    void transfer(HashMap.Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e = header.after; e != header; e = e.after) {
            if (rehash)
                e.hash = (e.key == null) ? 0 : hash(e.key);
            int index = indexFor(e.hash, newCapacity);
            e.next = newTable[index];
            newTable[index] = e;
        }
    }


## 删除

元素删除采用父类的逻辑，双向链表的删除通过`e.recordRemoval(this)`来实现，`recordRemoval`在HashMap.Entry中是空方法,在LinkedHashMap.Entry中提供了具体的实现。


    public V remove(Object key) {
        Entry<K,V> e = removeEntryForKey(key);
        return (e == null ? null : e.value);
    }

    final Entry<K,V> removeEntryForKey(Object key) {
        if (size == 0) {
            return null;
        }
        int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                modCount++;
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                    //双向链表中节点删除
                    e.recordRemoval(this);
                return e;
            }
            prev = e;
            e = next;
        }

        return e;
    }



## 查找

使用哈希表实现查找，效率高。

若定义的访问顺序是LRU访问顺序，则查找元素的同时，需对双向链表进行LRU调整。

    public V get(Object key) {
        Entry<K,V> e = (Entry<K,V>)getEntry(key);
        if (e == null)
            return null;
        //查找的同时，维护双向链表，实现LRU
        e.recordAccess(this);
        return e.value;
    }

## LinkedHashIterator

LinkedHashMap提供的顺序访问就是通过LinkedHashIterator实现。

HashIterator是fast-fail的，若一线程在使用它遍历集合的同时，另一线程对集合中的元素进行了增减，则会抛出`ConcurrentModificationException`异常。

    private abstract class LinkedHashIterator<T> implements Iterator<T> {
        Entry<K,V> nextEntry    = header.after;
        Entry<K,V> lastReturned = null;

        int expectedModCount = modCount;

        public boolean hasNext() {
            return nextEntry != header;
        }

        public void remove() {
            if (lastReturned == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();

            LinkedHashMap.this.remove(lastReturned.key);
            lastReturned = null;
            expectedModCount = modCount;
        }

        Entry<K,V> nextEntry() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (nextEntry == header)
                throw new NoSuchElementException();

            Entry<K,V> e = lastReturned = nextEntry;
            nextEntry = e.after;
            return e;
        }
    }

LinkedHashMap提供三种集合视角，LinkedHashIterator。

    private class KeyIterator extends LinkedHashIterator<K> {
        public K next() { return nextEntry().getKey(); }
    }

    private class ValueIterator extends LinkedHashIterator<V> {
        public V next() { return nextEntry().value; }
    }

    private class EntryIterator extends LinkedHashIterator<Map.Entry<K,V>> {
        public Map.Entry<K,V> next() { return nextEntry(); }
    }

## `containsValue`

LinkedHashMap覆盖了父类containsValue的实现，通过遍历双向链表来实现跟高效的访问。
    
    public boolean containsValue(Object value) {
        // Overridden to take advantage of faster iterator
        if (value==null) {
            for (Entry e = header.after; e != header; e = e.after)
                if (e.value==null)
                    return true;
        } else {
            for (Entry e = header.after; e != header; e = e.after)
                if (value.equals(e.value))
                    return true;
        }
        return false;
    }

## `removeEldestEntry`

`removeEldestEntry`定义什么时候删除最老的元素，在`addEntry`中被调用

    //可在LinkedHashMap的子类中覆盖，实现基于LRU淘汰算法的定长缓存
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }

## 扩展

[10行Java代码实现LRU缓存](/2015/12/27/linkedhashmap-realise-LRU-cache/ "10行Java代码实现最近被使用（LRU）缓存")

## 总结

学习LinkedHashMap源码发现，一些逻辑已在父类HashMap留了钩子，在HashMap中不提供实现，而是到子类LinkedHashMap中才具体实现，像`put`方法中的`e.recordAccess(this)`，这也相当于是一种策略模式。

学习源码的过程主要是为了一探究竟、加深理解，源码的设计融入了一些常用的设计模式，也是编码的一大启发，希望能吸收并应用到自己的编码中。


## 参考
[Java LinkedHashMap源码解析](http://liujiacai.net/blog/2015/09/12/java-linkedhashmap/ "Java LinkedHashMap源码解析")

