---
layout: post
title: "java泛型的协变、逆变与不变"
tags: ["语法", "泛型", "java"]
---

泛型的协变、逆变、不变.

## 协变、逆变、不变 （covariance，contra variance，invariance）

基本概念如下：

_假定Base为基类，Derived为衍生类_

### 协变（covariance）

协变允许使用更加具体的类来代替原来的类，例如c# 中

```c# 
IEnumerable<Derived> d = new List<Derived>();
IEnumerable<Base> b = d;
```

### 逆变（contra variance）

逆变允许使用更基础的类来替换原来的类，例如c# 中

```c# 
Action<Base> b = (target) => { Console.WriteLine(target.GetType().Name); };
Action<Derived> d = b;
d(new Derived());
```

### 不变（invariance）

不变意味着只能使用声明时确定的类型

简单来说，协变接受子类，逆变接受父类。java中协变逆变和c# 中较为不同，接下来详细讨论。

### Array

java中的数组是协变的。

首先，声明父类的数组元素可以包含子类

```java
/**
	这里声明了一个Number类型的数字，其子类Integer和Double都可以放入该数组
*/
Number[] numArr = new Number[2];
numArr[0] = Integer.parseInt("1");
numArr[1] = Double.parseDouble("1.0");
```

其次，子类型的数组可以赋值给父类型的数组

```java
/**
	Integer是Number的子类，Integer[]可以被赋值给Number[]
*/
Integer[] intArr = new Integer[5];
Number[] numArr = intArr;
/**
	这里编译n正常但是会在运行时报错，因为numArr的实际类型是Integer[]
*/
numArr[0] = Double.parseDouble("1.0");
```

但是注意，java运行时保存了数组的真实类型，所以上面将double放入数组时尽管编译可以通过，运行时会抛出ArrayStoreException，因为在运行时jvm知道数组的实际类型，尝试放入非法的类型导致抛出异常。

### 泛型

与数组不同的是，泛型信息在运行时会被擦除（详细内容可以见 java泛型擦除.md,并且相对的，数组不支持泛型，编译时要知道数组的具体类型，所以数组必须以明确的类型声明。为了提供运行时动态根据类型创建数组的能力，sun在reflect包下的Array类中提供了newInstance方法，通过反射动态创建对应类型的数组），jvm无法知道真实类型，如下

```java
List<Integer> intList = new ArrayList<>(); //1
List<Number> numList = intList; //2
numList.add(Double.parseDouble("1.0")); //3
```

上面的代码，将Integer的List赋值给Number的List，之后给该List放入一个Double。如果jvm知道numList的实际类型是List<Integer>，那么可以在运行时抛出异常，但是java泛型信息在运行时已被擦除，jvm并不知道numList的实际类型不允许放入double，所以无法发现这个问题，导致数据被污染（heap pollution）。为了解决这个问题，java将该检查放在了编译期间，第2步会编译错误，禁止将其编译成class文件，以预防第3步类似的错误。

所以说，对简单泛型来说，java的泛型是不变的（invariance），为了支持协变和逆变，java提供了统配符（wildcard）

### 通配符（wildcard）

java中可以如下声明：

```java
List<?> list = new ArrayList<Integer>();
List<? super Integer> list = new ArrayList<Integer>();
List<? extends Integer> list = new ArrayList<Integer>();
```

这里的？代表未知的类型，可以通过super声明该类型的下限，表示？必须为该类型或者该类型的父类；另外也可以通过extends声明该类型的上限，表示？必须为该类型或者该类型的子类。

协变和逆变产生了不同的结果，协变是read-only，逆变是write-only。

再次声明，协变是可以接收本类和子类，即<？ extends Integer>

```java
List<Long> longList = Arrays.asList(1L,2L);
List<? extends Number> numList = longList;
numList.add(Integer.valueOf(0));//3
```

在上面的代码中，一个Long的list可以赋值给List<? extends Number>，从这个List中读取数据时，我们知道它可以安全地被转换为Number，但是如果尝试写入数据，因为无法确定其具体类型，所以第3行代码，add操作被禁止。即协变是read-only。

```java
List<? super Thread> list = new ArrayList<Runnable>();
list.add(new Thread());
list.add(new Thread(){
    @Override
    public void run(){
        System.out.println(1);
    }
});
```

上面的代码，Thread实现了Runnable，所以List<Runnable>可以赋值给List<? super Thread>，代码2、3行中我们可以安全的将Thread或者Thread的子类放入list（第3行创建了一个匿名内部类，其继承自Thread，所以是Thread的子类，具体可以看 嵌套类.md）。但是从list读取时，我们无法将其安全的声明为Thread（虽然可以强制转换，但是可能会运行时抛出ClassCastException），因为我们并不知道具体类型是否是Thread或者Thread的父类，所以逆变是write-only。



### 总结

java是一门l静态类型语言，java通过compile和runtime结合检查的方式来保证类型安全。对于数组，runtime时保存了数组的具体类型信息，所以不会在编译期检查，非法操作将在runtime抛出异常；而对于java泛型，因为运行时泛型信息被擦除（也有语言在运行时记录了泛型信息，但泛型是java后期引入的，为了兼容之前版本的class，java采取了泛型擦除的方式来实现），导致运行时无法检测错误，因此java设计者将对泛型的检查放在了compile，在compile时阻止非法的操作，保证runtime时的类型安全。

java的普通泛型是invariance的，但是通过通配符？可以实现逆变和协变。协变是read-only，逆变是write-only，这个特性可以用在对函数入参的限制上，如果我们只需要从参数中读取，则使用协变，如果需要往参数中写入，则使用逆变。

