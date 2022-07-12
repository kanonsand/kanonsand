---
layout: post
title: "Linux OOM Killer"
tags: ["linux", "操作系统"]
---

Linux通过OOM killer机制，在系统内存不足时强制杀掉某些进程来释放内存，来确保操作系统自身的运行不受影响。

## 背景
接之前一篇 [ES索引备份和恢复](ES索引备份和恢复.md)。构造完ES数据并进行测试，发现程序写入过程中，ES和写入程序都有概率被系统直接杀掉，
导致数据导入不完整。经过查阅资料发现了操作系统有OOM killer这个机制。


### 为什么有OOM Killer

现代操作系统中，应用程序申请内存时，操作系统并不会直接给程序提供物理内存，这基于这样一个现状：大部分程序申请内存后并不会马上使用，有的甚至从不使用。
所以操作系统会先给应用程序分配一个虚拟的内存，知道用户真正访问这块内存，操作系统产生缺页中断，才会把真实内存分配给应用程序。


这种情况下，应用程序理论上可以申请超过操作系统剩余内存的内存空间，linux通过/proc/sys/vm/overcommit_memory文件来控制可以overcommit的内存
大小。默认情况下大部分linux将其设置为0，表示不受限。


通过overcommit的机制，应用程序的malloc()调用总会成功（因为这个时候内存并没有真实分配），但是操作系统需要一个机制，来保证内存真正用尽的情况下不
影响操作系统自身的正常运行。这个机制就是OOM Killer。



OOM Killer在没有内存可用的时候通过杀掉某些进程来确保操作系统的运行。OOM Killer会为每个应用计算一个得分，得分最高的应用会被杀死以释放内存。



### OOM Killer评分计算逻辑

上面提到，OOM Killer会强制杀掉评分最高的应用，这个评分按照以下逻辑计算：

+ 占用内存越高评分越高

+ 运行时间越久评分越低

### 控制OOM Killer的选择

应用评分可以通过以下文件查看

```
/proc/${pid}/oom_score
```

可以手动设置应用程序的oom调整策略，通过修改如下文件内容

```
/proc/${pid}/oom_adj
可以写入-17到15之间的任意数值，数值越高，OOM Killer会优先选择该应用杀死，数值越低优先级越低，如果设置为-17，OOM Killer永远不会杀死该应用
```



### 手动触发OOM Killer

除了操作系统自动调用外，我们可以手动触发OOM Killer运行

```
chmod 777 /proc/sysrq-trigger
echo f > /proc/sysrq-trigger
sysrq-trigger是内核提供的一个“魔法文件”，通过写入不同的值，可以无视内核的状态触发一些特定的程序。f就对应了OOM Killer
```

触发之后通过如下命令查看

```
dmesg
dmesg输出内核环形缓冲区内的内容
```

因为输出较多，可以重定向到文件后用less或者cat查看。如下格式的输出就是OOM Killer的日志

```
[73762.623298] Out of memory: Kill process 9992 (chromium-browse) score 336 or sacrifice child
[73762.623304] Killed process 9992 (chromium-browse) total-vm:764960kB, anon-rss:184248kB, file-rss:116796kB
```

可以看到进程号9992以及它的评分336

