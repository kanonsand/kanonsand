---
layout: post
title: "Linux下perf工具"
tags: ["linux","perf"]
---

Linux下可以使用perf命令来监控cpu状态进而排查性能问题。

## 安装
deepin,Ubuntu,Debian下用apt安装
```shell
$ apt install linux-tools-$(uname -r) linux-tools-generic
```
确认下安转是否完成
```shell
$ perf --version
perf version 5.15.5
```

## perf选项
默认情况下perf需要以root权限使用。

perf命令选项比较多，可以通过-h查看
```shell
$ perf -h

 usage: perf [--version] [--help] [OPTIONS] COMMAND [ARGS]

 The most commonly used perf commands are:
   annotate        Read perf.data (created by perf record) and display annotated code
   archive         Create archive with object files with build-ids found in perf.data file
   bench           General framework for benchmark suites
   buildid-cache   Manage build-id cache.
   buildid-list    List the buildids in a perf.data file
   c2c             Shared Data C2C/HITM Analyzer.
   config          Get and set variables in a configuration file.
   daemon          Run record sessions on background
   data            Data file related processing
   diff            Read perf.data files and display the differential profile
   evlist          List the event names in a perf.data file
   ftrace          simple wrapper for kernel's ftrace functionality
   inject          Filter to augment the events stream with additional information
   iostat          Show I/O performance metrics
   kallsyms        Searches running kernel for symbols
   kmem            Tool to trace/measure kernel memory properties
   kvm             Tool to trace/measure kvm guest os
   list            List all symbolic event types
   lock            Analyze lock events
   mem             Profile memory accesses
   record          Run a command and record its profile into perf.data
   report          Read perf.data (created by perf record) and display the profile
   sched           Tool to trace/measure scheduler properties (latencies)
   script          Read perf.data (created by perf record) and display trace output
   stat            Run a command and gather performance counter statistics
   test            Runs sanity tests.
   timechart       Tool to visualize total system behavior during a workload
   top             System profiling tool.
   version         display the version of perf binary
   probe           Define new dynamic tracepoints
   trace           strace inspired tool

 See 'perf help COMMAND' for more information on a specific command.
```
根据提示，每个选项可以通过 ***perf help 选项*** 来查看详细帮助
```shell
$ perf help top
#或者用下面的命令
$ perf top -h
```

## 一般情况下的perf使用流程
perf选项非常多，其中大部分实际用到的比较少，这里介绍下比较通用的使用流程

### list子命令
list子命令用于列出当前可以观测的事件，这些事件可以用于其他perf子命令的 ***-e*** 参数中。
```shell
$ perf list
List of pre-defined events (to be used in -e):

  branch-instructions OR branches                    [Hardware event]
  branch-misses                                      [Hardware event]
  bus-cycles                                         [Hardware event]
  cache-misses                                       [Hardware event]
  cache-references                                   [Hardware event]
  cpu-cycles OR cycles                               [Hardware event]
  instructions                                       [Hardware event]
  ref-cycles                                         [Hardware event]

  alignment-faults                                   [Software event]
  bpf-output                                         [Software event]
  cgroup-switches                                    [Software event]
  context-switches OR cs                             [Software event]
  ...
```
输出比较多，包括硬件、软件、缓存等多种类型的事件

### top子命令
top子命令和linux自身的top命令类似，可以实时展示cpu的状态
```shell
$ perf top
Samples: 150K of event 'cycles', 4000 Hz, Event count (approx.): 38938083757 lost: 0/0 drop: 0/0                                                                                 
Overhead  Shared Object                          Symbol     
   0.29%  [kernel]                               [k] switch_mm_irqs_off                                                                                                         ◆
   0.29%  [kernel]                               [k] __update_load_avg_cfs_rq
   0.28%  Xorg                                   [.] miSpriteTrace
   0.28%  [kernel]                               [k] select_task_rq_fair
   0.28%  [kernel]                               [k] native_irq_return_iret       
   0.27%  [kernel]                               [k] __list_del_entry_valid
   0.27%  [kernel]                               [k] update_sd_lb_stats.constprop.145  
   0.27%  [kernel]                               [k] copy_user_enhanced_fast_string    
   0.27%  libQt5Gui.so.5.15.3                    [.] 0x00000000000ff189
   0.26%  [kernel]                               [k] sock_poll
   0.26%  [kernel]                               [k] rb_erase
   0.26%  libQt5Gui.so.5.15.3                    [.] 0x00000000000ff199           
   0.24%  [kernel]                               [k] read_tsc
```
注意第一行输出，默认情况下top子命令展示的是cycles，即上面list子命令中的
```shell
cpu-cycles OR cycles                               [Hardware event]
```
这一行，相当于执行了命令
```shell
$ perf top -e cycles
```
因为监控的是cpu时钟，输出结果第一列为cpu占用情况，第二列是调用函数的应用或者lib，第三列为调用的具体函数，
注意第三列有个中括号，[k]表示kernel，即内核空间，[.]则代表用户空间。
#### top命令查看其他事件
同样我们可以通过-e参数指定list子命令中列出的其他事件
```shell
$ perf top -e cache-misses -d 5
Samples: 14K of event 'cache-misses', 4000 Hz, Event count (approx.): 190132790 lost: 0/0 drop: 0/0                                                                              
Overhead  Shared Object                       Symbol                                                                                                                             
  36.28%  libc-2.28.so                        [.] 0x000000000015c86f
   2.48%  libQt5Gui.so.5.15.3                 [.] 0x00000000000ff185
   2.42%  sunloginclient                      [.] 0x00000000009c40e2
   2.33%  sunloginclient                      [.] 0x00000000009cc8c9
   1.74%  libQt5Gui.so.5.15.3                 [.] 0x00000000000ff191
```
这次监控cache-misses事件，-d参数控制每次刷新间隔，这里5表示5秒刷新一次，默认1秒刷新一次。


## stat子命令
top子命令可以监控某一个事件的实时状态，stat命令则可以统计一段时间内的硬件事件和软件事件。

```shell
$ perf stat -a
^C
 Performance counter stats for 'system wide':

         20,003.07 msec cpu-clock                 #    4.000 CPUs utilized          
            45,869      context-switches          #    2.293 K/sec                  
             4,396      cpu-migrations            #  219.766 /sec                   
             4,457      page-faults               #  222.816 /sec                   
     2,493,047,432      cycles                    #    0.125 GHz                    
     1,253,153,416      instructions              #    0.50  insn per cycle         
       266,531,196      branches                  #   13.325 M/sec                  
        12,135,477      branch-misses             #    4.55% of all branches        

       5.000422076 seconds time elapsed
```
-a参数表示监控所有cpu，执行后需要手动ctrl+c结束执行，然后会打印执行信息。-a参数可以不用传，是默认的参数，
如果想指定某个Cpu，通过 -C 指定
```shell
$ perf stat -C 1
^C
 Performance counter stats for 'CPU(s) 1':

          2,848.94 msec cpu-clock                 #    1.000 CPUs utilized          
             7,113      context-switches          #    2.497 K/sec                  
               539      cpu-migrations            #  189.193 /sec                   
               929      page-faults               #  326.086 /sec                   
       497,256,145      cycles                    #    0.175 GHz                    
       328,008,568      instructions              #    0.66  insn per cycle         
        70,037,831      branches                  #   24.584 M/sec                  
         2,410,521      branch-misses             #    3.44% of all branches        

       2.848895686 seconds time elapsed
```
查看输出第一行，这里显示是第一个cpu的状态，而上个指令则是system wide，即系统层级。

state命令还可以监控其他命令执行过程中的状态，比如想查看ls命令的执行，如下
```shell
$ perf stat ls -la
 Performance counter stats for 'ls -la':

              2.26 msec task-clock                #    0.061 CPUs utilized          
                 2      context-switches          #  884.489 /sec                   
                 0      cpu-migrations            #    0.000 /sec                   
               140      page-faults               #   61.914 K/sec                  
         8,103,208      cycles                    #    3.584 GHz                    
         8,228,263      instructions              #    1.02  insn per cycle         
         1,696,831      branches                  #  750.414 M/sec                  
            37,290      branch-misses             #    2.20% of all branches        

       0.036874836 seconds time elapsed

       0.000000000 seconds user
       0.002878000 seconds sys
```
这里输出省略了ls -la命令的结果，可以看到perf stat打印出ls命令执行过程中的状态，注意stat用于监控
其他命令时，其他命令返回后会自动输出，不需要手动ctrl+c，因此如果想要实现指定时间和perf stat自动结束，
可以借助sleep命令，比如perf stat sleep 5，因为sleep命令是等待5秒后返回，而其本身基本不占用资源，
因此等效于让perf stat命令5s后结束。

stat子命令还可以监控指定进程的运行，通过-p参数指定pid即可。
```shell
perf stat -p 354559
^C
 Performance counter stats for process id '354559':

              0.28 msec task-clock                #    0.000 CPUs utilized          
                 4      context-switches          #   14.384 K/sec                  
                 2      cpu-migrations            #    7.192 K/sec                  
                 5      page-faults               #   17.980 K/sec                  
           976,377      cycles                    #    3.511 GHz                    
           210,943      instructions              #    0.22  insn per cycle         
            44,318      branches                  #  159.370 M/sec                  
             3,767      branch-misses             #    8.50% of all branches        

       2.744651863 seconds time elapsed
```

stat子命令同样支持-e参数指定需要监控的项。
```shell
perf stat -p 354559 -e cycles -e branch-loads
^C
 Performance counter stats for process id '354559':

           307,441      cycles                                                      
            12,641      branch-loads                                                

       1.342169631 seconds time elapsed

```
此时结果就会只包含我们指定的event。

## record子命令
record子命令和stat命令参数基本相同，同样用于监控一段时间内的event，不过record会将结果记录到***perf.data***文件中。
```shell
$ perf record sleep 5
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.017 MB perf.data (7 samples) ]
```
命令和perf stat sleep 5基本相同，不过命令完成后输出了记录文件的大小。

record和stat一样也可以有-e -p参数，这里不在赘述。

## report子命令
report子命令用于读取record子命令生成的文件。
```shell
$ perf report --stdio
# To display the perf.data header info, please use --header/--header-only options.
#
#
# Total Lost Samples: 0
#
# Samples: 7  of event 'cycles'
# Event count (approx.): 2263562
#
# Overhead  Command  Shared Object      Symbol                  
# ........  .......  .................  ........................
#
    93.45%  sleep    [kernel.kallsyms]  [k] strnlen_user
     6.28%  perf-ex  [kernel.kallsyms]  [k] strnlen
     0.26%  perf-ex  [kernel.kallsyms]  [k] intel_pmu_handle_irq
     0.01%  perf-ex  [kernel.kallsyms]  [k] native_write_msr


#
# (Tip: Order by the overhead of source file name and line number: perf report -s srcline)
#
```
--stdio参数用于将结果输出到标准输出，没有该参数则默认进入一个类似于vi的窗口查看结果。

## script子命令
script子命令同样用于读取perf.data文件，不过不同于report命令对event进行了整合，script命令
将每个event单独输出。
```shell
$ perf script
       perf-exec 375173 275691.191220:          1 cycles:  ffffffff95c7ca98 native_write_msr+0x8 ([kernel.kallsyms])
       perf-exec 375173 275691.191223:          1 cycles:  ffffffff95c7ca98 native_write_msr+0x8 ([kernel.kallsyms])
       perf-exec 375173 275691.191224:         10 cycles:  ffffffff95c7ca98 native_write_msr+0x8 ([kernel.kallsyms])
       perf-exec 375173 275691.191226:        234 cycles:  ffffffff95c7ca9a native_write_msr+0xa ([kernel.kallsyms])
       perf-exec 375173 275691.191227:       5843 cycles:  ffffffff95c0f8e7 intel_pmu_handle_irq+0x157 ([kernel.kallsyms])
       perf-exec 375173 275691.191229:     142195 cycles:  ffffffff961eeea5 strnlen+0x25 ([kernel.kallsyms])
           sleep 375173 275691.191270:    2115278 cycles:  ffffffff961e0025 strnlen_user+0x65 ([kernel.kallsyms])
```

也可以通过--header参数展示额外的信息。
```shell
$ perf script --header
# ========
# captured on    : Mon Nov  7 19:22:59 2022
# header version : 1
# data offset    : 280
# data size      : 17336
# feat offset    : 17616
# hostname : user-PC
# os release : 5.15.45-amd64-desktop
# perf version : 5.15.5
# arch : x86_64
# nrcpus online : 4
# nrcpus avail : 4
# cpudesc : Intel(R) Core(TM) i3-8100 CPU @ 3.60GHz
# cpuid : GenuineIntel,6,158,11
# total memory : 7906864 kB
# cmdline : /usr/bin/perf_5.15 record sleep 5 
# event : name = cycles, , id = { 1005, 1006, 1007, 1008 }, size = 128, { sample_period, sample_freq } = 4000, sample_type = IP|TID|TIME|PERIOD, read_format = ID, disabled = 1, 
# CPU_TOPOLOGY info available, use -I to display
# NUMA_TOPOLOGY info available, use -I to display
# pmu mappings: intel_pt = 8, software = 1, power = 19, uprobe = 7, uncore_imc = 10, cpu = 4, cstate_core = 17, uncore_cbox_2 = 13, breakpoint = 5, uncore_cbox_0 = 11, tracepoin
# CACHE info available, use -I to display
# time of first sample : 275691.191220
# time of last sample : 275691.191270
# sample duration :      0.050 ms
# MEM_TOPOLOGY info available, use -I to display
# bpf_prog_info 3: bpf_prog_6deef7357e7b4530 addr 0xffffffffc042c3b0 size 54
# bpf_prog_info 4: bpf_prog_6deef7357e7b4530 addr 0xffffffffc042e4e4 size 54
# bpf_prog_info 5: bpf_prog_6deef7357e7b4530 addr 0xffffffffc05f3970 size 54
# bpf_prog_info 6: bpf_prog_6deef7357e7b4530 addr 0xffffffffc05f5dc4 size 54
# bpf_prog_info 7: bpf_prog_6deef7357e7b4530 addr 0xffffffffc0f38984 size 54
# bpf_prog_info 8: bpf_prog_6deef7357e7b4530 addr 0xffffffffc0f3a598 size 54
# bpf_prog_info 17: bpf_prog_6deef7357e7b4530 addr 0xffffffffc10b613c size 54
# bpf_prog_info 18: bpf_prog_6deef7357e7b4530 addr 0xffffffffc10b89e8 size 54
# cpu pmu capabilities: branches=32, max_precise=3, pmu_name=skylake
# missing features: TRACING_DATA BRANCH_STACK GROUP_DESC AUXTRACE STAT CLOCKID DIR_FORMAT COMPRESSED CLOCK_DATA HYBRID_TOPOLOGY HYBRID_CPU_PMU_CAPS 
# ========
#
       perf-exec 375173 275691.191220:          1 cycles:  ffffffff95c7ca98 native_write_msr+0x8 ([kernel.kallsyms])
       perf-exec 375173 275691.191223:          1 cycles:  ffffffff95c7ca98 native_write_msr+0x8 ([kernel.kallsyms])
       perf-exec 375173 275691.191224:         10 cycles:  ffffffff95c7ca98 native_write_msr+0x8 ([kernel.kallsyms])
       perf-exec 375173 275691.191226:        234 cycles:  ffffffff95c7ca9a native_write_msr+0xa ([kernel.kallsyms])
       perf-exec 375173 275691.191227:       5843 cycles:  ffffffff95c0f8e7 intel_pmu_handle_irq+0x157 ([kernel.kallsyms])
       perf-exec 375173 275691.191229:     142195 cycles:  ffffffff961eeea5 strnlen+0x25 ([kernel.kallsyms])
```
可以看到多了一些header信息。

