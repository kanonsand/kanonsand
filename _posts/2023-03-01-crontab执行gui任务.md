---
layout: post
title: "crontab 执行gui任务"
tags: ["linux","crontab"]
---

crontab用于定时执行一些任务，但是当这些任务需要打开窗口时，会遇到类似如下的错误日志
```
Error: Can't open display: (null)
Failed creating new xdo instance
```
第一步可以尝试通过设置DISPLAY变量，如下：
```shell
export DISPLAY=:0
```
具体的值可以在桌面shell中执行
```shell
$ echo $DISPLAY
:0
```
获得，如果还是不行可以尝试如下：
首先在正常的桌面环境下打开shell，执行如下命令
```shell
xauth extract /tmp/xauth-foo $DISPLAY
```
/tmp/xauth-foo是随便的一个文件，后续会用到。
再执行
```shell
$ echo $DISOLAY
:0
```
在要执行的脚本开头，添加
```shell
xauth merge /tmp/xauth-foo
export DISPLAY=:0
```
即把上面生成的文件和环境变量设置到脚本中，查看是否可以正常运行了。
