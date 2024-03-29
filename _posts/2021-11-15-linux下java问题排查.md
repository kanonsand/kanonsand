---
layout: post
title: "linux下jvm问题排查"
tags: ["linux", "jvm", "java"]
---

Linux下获取java进程状态的一些命令和步骤

### 找到java进程中cpu占用最高的线程

可以直接使用arthas的thread命令，不过要把采样时间稍微拉长。这里介绍下直接使用linux命令查看的步骤。

+ top查看cpu占用最高的java进程pid，或者jps、ps都可以，获取到进程pid即可，记为$pid
+ top -H -p *$pid* 查看pid对应java进程的线程cpu占用情况
+ 选择top -H命令返回的PID，这个是线程PID，记为$PID使用printf '%x' *$PID* 打印线程PID对应的
16进制数
+ jstack 获取java进程的栈状态，grep 上面的16进制PID查看对应线程的栈

```shell
[root@localhost]# jps | grep MyServer
1109 MyServer

[root@localhost]# top -H -p 1109
   PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                      
255436 root      20   0 11.918g 1.817g  29964 S  0.3  2.9   0:02.55 java                                                                                                         
259166 root      20   0 11.918g 1.817g  29964 S  0.3  2.9   0:01.67 java                                                                                                         
260187 root      20   0 11.918g 1.817g  29964 S  0.3  2.9   0:00.60 java                                                                                                         
260251 root      20   0 11.918g 1.817g  29964 S  0.3  2.9   0:00.02 java                                                                                                         
255389 root      20   0 11.918g 1.817g  29964 S  0.0  2.9   0:00.00 java 

[root@localhost]# printf '%x\n' 255436
3e5cc

[root@localhost]# jstack 1109 > result.txt
[root@localhost]# less result.txt | grep 3e5cc
```

### 指定用户和分组执行命令
通常情况下，使用su命令切换用户然后执行命令即可。但是当一个用户属于多个组，我们想同时指定用户和组执行命令，需要
用runuser命令。语法如下
```shell
    runuser -l $username -g $group -c '$command'
```
用sudo命令也可以，语法更加简单，-u指定用户名后后续输入都被解释为命令
```shell
    sudo -u $username $command
```

顺便查看用户所属的分组使用groups命令
```shell
[root@localhost ~]# groups root
root : root
```
如果执行单条命令，runuser可以替代su命令。

***arthas如果连接失败可以检查下用户和组***

### arthas无法attach进程
+ 查看是否以[运行进程的用户](linux下jvm问题排查.md:38)执行arthas
+ 查看目标进程的java.policy文件，比如ES权限控制比较严格，需要将arthas的路径加入java.policy文件授权

### arthas获取dubbo的线程池对象（2.7.8版本）

```shell
ognl '@org.apache.dubbo.common.extension.ExtensionLoader@getExtensionLoader(@org.apache.dubbo.common.threadpool.manager.ExecutorRepository@class).getDefaultExtension().data'
```

这是一个以端口号为key，线程池对象为value的map，根据需要取对应端口号即可。
