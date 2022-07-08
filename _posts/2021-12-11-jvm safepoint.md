jvm safepoint

以下操作会阻止jvm进入safepoint：

1、初始化大对象（比如一个10G的数组）

2、array copying

3、jni处理分配（JNI handle allocation）

4、JNI critical regions

5、counted loop（带计数的循环，通常情况下jvm会尝试在一轮循环结束时检查是否进入safepoint，但是如果是明确计数的循环，有可能会优化掉这个检查，如果循环内操作耗时很久，会导致迟迟无法进入safepoint。这里的循环必须是int计数，long计数不会使用这个优化）

6、nio mapped byte buffers

可以在jvm添加如下参数-XX:+SafepointTimeout -XX:SafepointTimeoutDelay=<timeout in ms>让jvm打印出超过指定时间未进入safepoint的线程信息，可以帮助开发人员排查。

- `-XX:+PrintGCApplicationStoppedTime`

- `-XX:+PrintGCDetails`

- `-XX:+PrintSafepointStatistics`

- `-XX:PrintSafepointStatisticsCount=1`

  上面这些参数也可以帮助排查因为jvm暂停导致的问题。



reference

http://jvm-options.tech.xebia.fr/#  查询所有可用的jvm参数