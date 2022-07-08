# InvokeDynamic and MethodHandle

## InvokeDynamic

InvokeDynamic从java7引入，是实现java8 lambda和接口默认方法的核心。因为是jvm的指令，所以同样可以用于运行于jvm上的其他语言，比如jruby和nashorn。

InvokeDynamic是java1.0以来第一个加入的新的字节码指令，它和已有的invokeSpecial、invokeVIrtual、invokeStatic、invokeInterface，共同构成了jvm调用方法的指令集。

```
invokeVirtual
	分派实例方法
invokeInterface
	分派通过接口调用的方法
invokeStatic
	分派静态方法
InvokeSpecial
	调用确切的方法（比如说InvokeVirtual会根据多态调用子类实现，invokeSpecial允许绕过这个限制直接调用确切的方法）

	
invokeVirtual和invokeInterface区别：
例如：
	List<String> list = new ArrayList();
	list.add("1"); //1
	ArrayList<String> alist = new ArrayList();
	alist.add("2");//2
两者虽然都是arraylist的实例，但是1处调用invokeInterface，因为List是接口，而2处这调用invokeVirtual，调用ArrayList的实例方法。

invokeSpecial：
	用于调用不需要在运行时动态分派的方法，例如private方法和super调用，因为他们不可以被重写，所以在编译时就可以确定具体要调用的对象。
```



可以发现原有的四种调用已经涵盖了所有的java方法调用方式，那么为什么要引入invokeDynamic指令呢。

invokeDynamic指令的主要目的是引入了一种新的调用分派方式：在应用层面控制应该调用哪个方法，即在jvm执行到invokeDynamic指令之前，无需知道具体应该调用哪个方法。为java提供了更强的动态性。

引入该指令的主要意图是用户可以通过method handles相关api来决定在运行时的方法分派，同时又无需担心反射造成的性能和安全问题。并且声明中希望invokeDynamic能拥有和invokeVirtual相近的性能。

虽然invokeDynamic在java7时引入jvm，但是最初它只是为jvm上运行的其他动态语言提供的，javac并不会生成带有invokeDynamic的字节码。这一状况在java8得到了改变，invokeDynamic用于实现lambda和接口默认方法。

虽然java8lambda和接口默认方法用到了invokeDynamic指令，但是java并没有提供专门的关键字或者库用于直接编译成invokeDynamic的调用，所以对java开发者来说，这个概念依然是模糊的。接下来我们来看如何用代码改变这一状况。

## MethodHandle

为了让invokeDynamic正确运行，一个核心的概念就是method handle。method handle可以表示应该通过invokeDynamic调用的方法。

总的想法是每一条invokeDynamic指令都关联一个特殊的方法（bootstrap method，or BSM），当解释器执行到invokeDynamic指令时，BSM被调用，BSM返回一个对象（包含一个method handle）表面那个方法应该被调用。

听起来有点像反射，但是反射有一些限制导致其不能被用于invokeDynamic。相反的，java.lang.invoke.MethodHandle和其子类在java7时被加入API，用于表示可以被invokeDynamic调用的目标方法，MethodHandle类会被jvm特殊对待。

可以把method handle看做是一种更安全的、更现代的、用于最大可能类型安全的反射。invokeDynamic需要它，并且它也可以被单独使用。



## Method Type

一个java方法可以看做由以下四个部分构成

```
方法名
方法签名（包含返回值）
方法所属的类
方法的字节码
```

这意味着如果想要引用一个Method，我们需要找到一种高效的表示方法签名的方式，而不是直接使用Class<?>[]这种使用反射的方式。

换句话说，实现method handle的第一个模块就是找到一种表示方法签名的方法。在method handle API中，由method type担任这一角色。method type使用一个不可变的实例来表示方法签名。

如下用于构造方法签名，第一个参数为方法的返回值，后续参数为方法的参数，都用对应的class对象表示。

```java
// Signature of toString()
MethodType mtToString = MethodType.methodType(String.class);

// Signature of a setter method
MethodType mtSetter = MethodType.methodType(void.class, Object.class);

// Signature of compare() from Comparator<String>
MethodType mtStringComparator = MethodType.methodType(int.class, String.class, String.class);
```

现在有了表示方法签名的method type对象，结合方法名、类名，现在可以查找对应的method handle了。首先我们调用MethodHandlers.lookup()静态方法，该方法返回一个lookup上下文，而该lookup的访问权限基于当前调用lookup方法的类。不同于反射的setAccessible方法，lookup没有提供破坏访问权限的途径。

```java
final MethodType methodType = MethodType.methodType(void.class,String.class);
final MethodHandles.Lookup lookup =
                MethodHandles.lookup();
final MethodHandle test = lookup.findSpecial(TypeLearn.class, "test", methodType,TypeLearn.class);
```

LookUp对象包含多个findXXX方法，用于根据类、方法签名、方法名来查找对应的method handle，LookUp的实例拥有的权限基于调用放，所以无法访问非本类的private方法。

拿到MethodHandle对象后，可以通过invoke()或者invokeExact()来调用对应的方法，两者区别在于，invokeExact使用精确地方法签名执行调用，而invoke更灵活，可以通过如下一些规则对参数进行匹配：

```
基本类型包装
包装类解包
基本类型扩展为更大范围的类型
返回值为void可以被转为0（如果要求的返回值是基本类型）或者null（如果要求的返回类型为引用）
不管静态类型如何，null都被视为正确并被传递
```



Method Handle 和 invokeDynamic

invokeDynamic通过bootstrap method来使用method handle，不同于invokeVirtual，invokeDynamic不需要接收对象，相反，它更加类似于invokeStatic，通过BSM返回一个CallSite类型的对象，这个对象包含method handle。

当拥有invokeDynamic指令的类被加载时，call site处于一个“未绑定”的状态，当对应的BSM首次被调用并返回之后，这个call site视为“已绑定”。