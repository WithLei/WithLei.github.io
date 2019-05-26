---
layout: post  
title:  "ArrayList、Vector、HashMap、HashTable如何扩容"  
date: 2019-04-10  
description: "ArrayList、Vector、HashMap、HashTable如何扩容"
tag: Java
---

> 在复习Java基础容器扩容相关时，发现许多博客写的十分混乱，整理一下源码和结论

## ArrayList
默认初始10个大小，每次扩容是原容量的1.5倍，具体代码如下
```java
public ArrayList() {
	this(10);
} 
int newCapacity = (oldCapacity * 3)/2 + 1;

public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```
如 ArrayList 的容量为10，一次扩容后是容量为15
## Vector
默认初始10个大小，若容量增量系数(capacityIncrement) > 0，则将容器大小增加到capacityIncrement，否则将容量增加一倍，具体代码如下：
```java
public Vector() {
	this(10);
} 
int newCapacity = (capacityIncrement > 0) ? 
	(oldCapacity + capacityIncrement) : (oldCapacity * 2);
```
如 Vector 的容量为10，一次扩容后是容量为20
## HashMap
默认初始16个大小（必须是2的次方），当hashmap中的元素个数超过数组大小loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，也就是说，默认情况下，数组大小为16，那么当hashmap中元素个数超过16 * 0.75=12的时候，就把数组的大小扩展为2*16=32，即扩大一倍，然后重新计算每个元素在数组中的位置，具体代码如下：
```java
static final int DEFAULT_INITIAL_CAPACITY = 16; 
static final float DEFAULT_LOAD_FACTOR = 0.75f;
public HashMap() {
	this.loadFactor = DEFAULT_LOAD_FACTOR;
	threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
	table = new Entry[DEFAULT_INITIAL_CAPACITY];
	init();
}

resize(2 * table.length);
```
如 HashMap 的容量为16，一次扩容后是容量为32
## HashTable
默认初始11个大小，当元素个数超过容量长度的0.75倍时进行扩容，扩容增量为2*原数组长度+1，具体代码如下：
```java
public Hashtable() {
	this(11, 0.75f);
}
int newCapacity = oldCapacity * 2 + 1;
```
如 HashTable 的容量为11，一次扩容后是容量为23