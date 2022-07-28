---
layout: post
title: "使用try-with-resource处理文件stream"
tags: ["java", "arthas"]
---

java7引入了try-with-resource，免去了手动关闭流的操作。java8的Files类提供了将文件读取为
Stream<String>的方法，使得读取文件更加方便。

### 使用Files类读取文件并转为Stream<String>

传统try-with-resource读取文件

```java
    try(BufferedReader reader=Files.newBufferedReader(path,StandardCharsets.UTF_8)){
        reader.readLine();
        }
```

使用Files类将文件转为Stream<String>

```java
    Files.lines("input.txt").foreach(System.out::println)
```

非常简洁，直到发现idea自动提示没有使用try-with-resource关闭stream。重新看了下Stream接口，发现它居然实现了
AutoCloseable接口。

```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
}

public interface BaseStream<T, S extends BaseStream<T, S>>
        extends AutoCloseable {
}
```

**通常情况下，AutoCloseable接口会用于文件、数据库等需要手动关闭的地方，这是不是说我们也需要像关闭其他资源一样关闭
Stream呢？**

### 关于AutoCloseable接口

来看下AutoCloseable接口的文档

```java
/**
 API Note:
 It is possible, and in fact common, for a base class to implement AutoCloseable even though not all of 
 its subclasses or instances will hold releasable resources. For code that must operate in complete generality, 
 or when it is known that the AutoCloseable instance requires resource release, it is recommended to use 
 try-with-resources constructions. However, when using facilities such as java.util.stream.Stream that support 
 both I/O-based and non-I/O-based forms, try-with-resources blocks are in general unnecessary when using 
 non-I/O-based forms.
 /**
```

大致意思：一些情况下，尽管父类实现了AutoCloseable接口，但是其子类并没有持有需要手动释放的资源，这种情况下如果使用该子类的实例，则无需使用
try-with-resource，如果子类持有需要释放的资源，那么需要使用try-with-resource。
这里特别提出了Stream接口，这个接口同时支持基于I/O和不基于I/O的形式，因此只需要在使用基于I/O的Stream时使用try-with-resource。

即：**普通的Stream流无需使用try-with-resource，如果是基于I/O或者其他持有资源的Stream，则需要使用try-with-resource**

```java
    try(Stream<String> linesStream=Files.lines(Paths.get("file.txt"))){
        linesStream.forEach(System.out::println);
        }catch(IOException e){
        // TODO: handle IOException...
        }
```

Stream不会自动调用(AutoCloseable::onClose)方法来执行清理操作，有点违反直觉，因为我们知道Stream中已经消费过的元素无法再次被消费，而I/O流我们
还需要手动清理。

既然这是Stream的特性，我们**最好还是按照规范使用try-with-resource处理I/O相关的Stream**。

### 实际需要try-with-resource的Stream

目前没有找到官方提供的文档说明需要手动关闭的API，但是我们可以查找BaseStream接口（Stream接口的父接口）里的onClose方法调用方，该方法基本实现
在AbstractPipeLine类，查看该方法的调用，发现主要在Files类和Scanner类的如下方法：

```java
        Files.list(Path dir);
        Files.walk(Path start,int maxDepth,FileVisitOption...options);
        Files.find(Path start,int maxDepth,BiPredicate<Path, BasicFileAttributes> matcher,FileVisitOption...options);
        Files.lines(Path path);
        Files.lines(Path path,Charset cs);
        Scanner.tokens();
        Scanner.findAll(Pattern pattern);
```

调用这些方法时需要使用try-with-resource

### 关闭一个Stream

控制Stream的关闭操作非常简单，只需要重写onClose方法，传入一个Runnable，进行清理操作。需要注意，只有在try-with-resource中使用，onClose
的清理操作才会被执行。

```java
    Stream<String> stringStream=Stream.of("a","b","c","d").onClose(()->System.out.println("closed"));

        stringStream.forEach(System.out::println);//不会打印 closed

        try(stringStream){
        stringStream.forEach(System.out::println); //会打印 closed
        }
```

实际上，我们甚至可以写多个onClose方法

```java
Stream<String> stringStream=Stream.of("a","b","c","d")
        .onClose(()->System.out.println("stage1 closed"))
        .onClose(()->System.out.println("stage2 closed"))
        .onClose(()->System.out.println("stage3 closed"));
```

输出

```
stage1 closed 
stage2 closed
stage3 closed
```

可以看到，使用try-with-resource，会按照顺序执行这些方法。

如果某个onClose方法发生异常，剩余方法依然会执行

```java
Stream<String> stringStream=Stream.of("a","b","c","d")
        .onClose(()->System.out.println("stage1 closed"))
        .onClose(()->{throw new IllegalArgumentException();}) //抛出异常
        .onClose(()->System.out.println("stage3 closed"));
```

输出

```
stage1 closed 
stage3 closed
Exception in thread "main" java.lang.IllegalArgumentException
```

如果多个onClose方法发生异常

```java
Stream<String> stringStream=Stream.of("a","b","c","d")
        .onClose(()->System.out.println("stage1 closed"))
        .onClose(()->{throw new IllegalArgumentException();})
        .onClose(()->{throw new NullPointerException();});
```

输出

```
stage1 closed
Exception in thread "main" java.lang.IllegalArgumentException
    ...
	Suppressed: java.lang.NullPointerException
```

第一个抛出的异常会suppress后续抛出的异常，并且所有异常都会保留在异常栈中。

### Stream和flatMap

当Stream遇到flatMap，会发生微妙的变化。

```java
        Stream<String> stream1=Stream.of("a","b","c").onClose(()->System.out.println("stream 1 closed"));

        Stream<String> stream2=Stream.of("d","e","f").onClose(()->System.out.println("stream 2 closed"));

        Stream<Stream<String>>combineStream=Stream.of(stream1,stream2).onClose(()->System.out.println("combine stream closed"));

        long count=combineStream.flatMap(Function.identity()).count();

        System.out.println(count);
```
输出
```
stream 1 closed
stream 2 closed
6
```
可以看到我们并没有使用try-with-resource，但是stream1和stream2被自动关闭了。

所以**传给flatMap的流会被自动关闭**