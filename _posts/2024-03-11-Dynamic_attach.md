
jmap和jstack命令我们经常用到，但是如果jvm已经异常，比如说发生了oom，或者负载满了，就会无法响应请求，此时获取堆栈信息会报错，可以通过添加-F参数强制拉取堆栈或者获取coredump，但是这个耗时和正常情况下能够差到几个量级。尽管我们只是添加了一个-F参数，实际上底层使用完全不同的两种逻辑获取jvm信息。

## Dynamic Attach
以jmap为例，整理下其工作流程（假设目标jvm pid为1234）
1.attack之前俺，jmap先在目标进程工作目录或者/tmp目录下创建一个.attach_pid1234的文件
2.jvm向目标进程发送SIGQUIT信号（kill -3,接收到该信号jvm会自动打印堆栈信息),目标jvm接收到信号并找到.attach_pid1234文件后，会开启一个AttachListener线程
3.AttachListener线程创建一个unix socket /tmp/.java_pid1234,这个socket监听外部工具的命令
4.为了安全起见，尝试连接到jvm时，目标jvm会检查对方的euid和egid，确保和当前jvm的相同才会进行连接（即使root用户也不能突破这个限制），所以必须要以目标jvm相同的用户执行attach
5.jmap连接到unix socket，发送dumpheap命令
6.AttachListener接收并处理该命令，并将输出写回到socket，因为这个时候heapdump是在jvm内生成的，所以速度很快。需要注意的是jvm在到达safepoint时才可以执行这个操作，如果safepoint迟迟无法达到（程序hung住或者长时间GC），jmap会超时并退出。

### 优点
1.dump由jvm自己处理，速度很快
2.jmap和jstack不存在版本限制，不依赖目标jvm版本

### 缺点
1.需要以和目标进程相同的用户执行attach
2.目标jvm存活并且状态正常才可使用
3.目标jvm添加了-XX:+DisableAttachMechanism参数后会无法使用

## ServiceAbility Agent
当jmap/jstack使用-F参数，会切换为使用HotSpot ServiceAbility Agent完成，这个工具通过操作系统提供的debug能力来读取目标进程的内存和core file（在linux上使用ptrace和/proc（主要是ptrace），Solaris系统使用libproc，Windows下使用dbgeng.dll。下面会以linux为例）
1.jmap -F调用PTRACE_ATTACH，目标jvm会因为响应SIGSTOP无条件暂停
2.jmap使用PTRACE_PEEKDATA读取目标进程的内存，但是每次只能读取一个word，所以非常慢
3.jmap重构jvm内部结构（不同版本jvm内存布局有差异，必须和目标jvm的版本相同)
4.jmap生成heap dump并恢复目标进程的运行

### 优点
1.不需要目标jvm响应，可以作用于已经hang住的进程
2.ptrace使用操作系统的权限，root用户可以dump所有进程

### 缺点
1.读取慢，特别是大堆
2.工具和目标jvm版本必须匹配
3.不保证目标属于safe point，尽管jmap尽量处理所有特殊情况，但仍然可能处于不一致的状态

### 使用gdb加速jmap -F获取heapdump
如上所述，jmap -F在堆比较大时会很慢，此时我们可以借助gdb先生成heapdump（比jmap快得多，但是格式不是hprof），再使用jmap将其转换为hprof。

#### gdb获取heapdump
```shell
# gdb -pid 1234
(gdb) gcore /tmp/jvm.core
(gdb) detach
(gdb) quit
```

#### jmap转换gdb的heapdump
```shell
# jmap -dump:format=b,file=jvm.hprof /usr/bin/java /tmp/jvm.core
```

