通常情况下，java中不用的本地变量没必要显式赋值为null，如下面的代码：
```java
    public void handle() {
        List<Map<String, Object>> mapList = IntStream.range(1, 500000)
                .mapToObj(e -> {
                    Map<String,Object> map = new CaseInsensitiveHashMap();
                    //put something into map
                    return map;
                }).collect(Collectors.toList());
        List<Object> list = mapList.stream().map(e -> Optional.ofNullable(e.get("key")).orElse("empty")).collect(Collectors.toList());
	//idea提示不需要显式指定为null
        mapList = null;
        doSomething(list);
    }
```
这里我们模拟了一个从第三方接口获取大量数据，返回了一个map的list，然后从map中取一个字段进行批量处理，mapList转换完后就不再需要，我们这里将其设为null，idea给出了提示，表示我们不需要这样做，但是假设代码最后一行的doSomething方法中又尝试分配大量内存，mapList部分情况下并不会被正确回收掉，进而导致内存溢出。以下是示例代码：
```java
    public static void main(String[] args) {
        load(500000);
    }


    public static void load(int size) {
        List<String> list = IntStream.range(0, 10)
                .mapToObj(i -> IntStream.rangeClosed(0, i)
                        .mapToObj(e -> "资产")
                        .collect(Collectors.joining()))
                .collect(Collectors.toList());
        List<CaseInsensitiveHashMap> mapList = IntStream.range(1, size)
                .mapToObj(e -> {
                    CaseInsensitiveHashMap caseInsensitiveHashMap = new CaseInsensitiveHashMap();
                    list.forEach(key -> caseInsensitiveHashMap.put(key, key));
                    return caseInsensitiveHashMap;
                }).collect(Collectors.toList());
        List<String> collect = mapList.stream()
                .map(e -> Optional.ofNullable(e.get("资产")).map(String::valueOf).orElse("default"))
                .collect(Collectors.toList());
	//mapList = null;
	//mapList.clear();
        calculateCollect(collect, size);

    }

    public static void calculateCollect(List<String> collect, int size) {
        List<String> list = IntStream.range(0, 10)
                .mapToObj(i -> IntStream.rangeClosed(0, i).mapToObj(e -> "机构").collect(Collectors.joining()))
                .collect(Collectors.toList());
        List<CaseInsensitiveHashMap> mapList = IntStream.range(1, size)
                .mapToObj(e -> {
                    CaseInsensitiveHashMap caseInsensitiveHashMap = new CaseInsensitiveHashMap();
                    list.forEach(key -> caseInsensitiveHashMap.put(key, key));
                    return caseInsensitiveHashMap;
                }).collect(Collectors.toList());
        System.out.println(mapList.size());
    }

public class CaseInsensitiveHashMap extends LinkedHashMap<String, Object> {
    private final Map<String, String> lowerCaseMap = new HashMap<String, String>();

    private static final long serialVersionUID = -2848100435296897392L;

    @Override
    public boolean containsKey(Object key) {
        Object realKey = lowerCaseMap.get(key.toString().toLowerCase(Locale.ENGLISH));
        return super.containsKey(realKey);
    }

    @Override
    public Object get(Object key) {
        Object realKey = lowerCaseMap.get(key.toString().toLowerCase(Locale.ENGLISH));
        return super.get(realKey);
    }

    @Override
    public Object put(String key, Object value) {
        Object oldKey = lowerCaseMap.put(key.toLowerCase(Locale.ENGLISH), key);
        Object oldValue = super.remove(oldKey);
        super.put(key, value);
        return oldValue;
    }

    @Override
    public void putAll(Map<? extends String, ?> m) {
        for (Map.Entry<? extends String, ?> entry : m.entrySet()) {
            String key = entry.getKey();
            Object value = entry.getValue();
            this.put(key, value);
        }
    }

    @Override
    public Object remove(Object key) {
        Object realKey = lowerCaseMap.remove(key.toString().toLowerCase(Locale.ENGLISH));
        return super.remove(realKey);
    }
}
```
上面的代码里有一个map类，这是一个忽略key大小写的map，代码是apache dbutil包中的MapListHandler默认实现，然后我们的两个方法，
1、load模拟从数据库中读取数据，提取其中某个字段进一步传给calculateCollect方法处理
2、calculateCollect方法又一次读取了大量的数据

指定jvm最大内存为700m，执行以上代码，代码会在calculateCollect方法中报堆内存不足，正常来说既然第一个方法已经执行到了结尾，说明堆容得下一个50w的maplist和另外一个string list，进入第二个方法后，同样是一个50w的maplist，由于之前的maplist已经不需要了，将其gc掉后内存应该容得下这个对象，但是事实上并没有按我们预想的那样(jdk8和jdk17均是如此),但是如果我们手动将load方法中的maplist指定为null，重新执行一次可以正常结束。或者执行mapList的clear方法也不会造成内存溢出，难道说gc不够智能吗？

重新修改代码，main函数中先用小数据量预热load方法，再执行同样的代码：
```java
public static void main(String[] args){
        for (int i = 0; i < 3000; i++) {
            test(2);
        }
        test(500000);
}
```
我们会发现，不需要显式赋值null，也不需要clear，同样的数据这次可以正常执行完成了，这是因为我们通过多次调用预热触发了jit，jit生成的优化后的代码足够智能，知道mapList可以被回收，所以代码正常执行完成。触发jit的方法调用次数取决于jvm，而且由于jit的阶梯式优化，一共有4个优化等级，调用次数越多越可能被优化为更高级别的代码，所以循环较低的时候，即使也已经触发了jit但是由于优化级别不够还是会导致内存溢出。

还有一种方法可以避免这个问题：将mapList单独放在一个代码块里:
```java
    public static void test(int size) {
        List<String> list = IntStream.range(0, 10)
                .mapToObj(i -> IntStream.rangeClosed(0, i)
                        .mapToObj(e -> "资产")
                        .collect(Collectors.joining()))
                .collect(Collectors.toList());
        List<String> collect;
        {
            List<CaseInsensitiveHashMap> mapList = IntStream.range(1, size)
                    .mapToObj(e -> {
                        CaseInsensitiveHashMap caseInsensitiveHashMap = new CaseInsensitiveHashMap();
                        list.forEach(key -> caseInsensitiveHashMap.put(key, key));
                        return caseInsensitiveHashMap;
                    }).collect(Collectors.toList());
             collect = mapList.stream()
                    .map(e -> Optional.ofNullable(e.get("资产")).map(String::valueOf).orElse("default"))
                    .collect(Collectors.toList());
        }
        calculateCollect(collect, size);

    }
```
mapList退出其作用域后也会很快被标记为可回收。

结论：
通常情况下本地变量不需要显式赋值null，它会随着离开作用域自然而然地被垃圾回收感知到，但是如果在不需要但是未出作用域之前调用了一个耗时很久或者占用内存很高的方法，手动清理本地变量可以避免极端情况下的内存溢出。



### JitWatch工具
jit的编译情况可以使用JitWatch工具，具体可以到github下载。使用时config项将代码目录配置到src/java,class目录配置到target/classes即可，不需要指定单个文件
