---
layout: post
title: "String中的intern"
tags: ["java"]
---

不建议使用String中的intern方法的几个理由。

## 背景
String提供了intern方法，允许我们手动将字符串对象放入常量池中，很多文章都说使用intern可以提高性能，但是实际上，至少有以下
三个理由阻止我们使用intern方法

## native方法带来的损耗
字符串常量池由jvm维护，底层是一个C++的hashtable，因此intern是一个native方法。调用native方法不可避免带来性能损耗。

## 常量池自身实现的限制
上面说到，常量池是一个C++的hashtable，想Java的hashmap一样，并发操作、数据量大小都会影响其性能。Java的并法包下面提供了
优化并发操作的ConcurrentHashMap，而且Java的HashMap会自动扩容。但是C++的hashtable并没有与时俱进的提供类似的功能。更致命
的是这个hashtable不会自动扩容，只能在jvm启动时指定大小，意味着随着放入的数据增加，hash冲突会逐渐增多。

## 影响GC
常量池通常是GC ROOT的一部分，而GC Root的大小会显著影响GC标记的性能。通常情况下常量池中的字符串主要来自加载的class文件，
而加载的类很少被卸载，所以大部分GC实现都不会主动检测和压缩常量池的大小，因此手动intern会让情况更加糟糕。

## 结论
综上，intern方法几乎任何情况下都不是一个好的选择，不如Java中手动维护一个类似的map，起码可以控制其并发、扩容等操作，甚至性能
反而会更好。