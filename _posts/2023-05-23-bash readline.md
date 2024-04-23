---
layout: post
title: "bash readline"
tags: ["bash","linux","readline"]
---

## bash中的readline
readline负责读取并展示用户终端的输入，终端的各种快捷键其实也是由它来处理的，emacs和vi模式
也是将readline提供的方法绑定到了快捷键上。

### readline中的方法
readline提供了很多现有的方法，这些方法可以通过bind -l查看
```shell
$ bind -l 
abort
accept-line
alias-expand-line
...
```
bind -p则用于列出当前和快捷键绑定的方法，
```shell
$ bind -p
"\C-g": abort
"\C-x\C-g": abort
"\e\C-g": abort
"\C-j": accept-line
"\C-m": accept-line
# alias-expand-line (not bound)
# arrow-key-prefix (not bound)
# backward-byte (not bound)
"\C-b": backward-char
...
```
没有和按键绑定的有提示 not bound,对于快捷键Ctrl+a，即默认的回到开头，可以通过如下方式查看其绑定的
方法
```shell
$ bind -p | grep -F '\C-a'
"\C-a": beginning-of-line
```
可以看到实际上这个快捷键绑定到了beginning-of-line这个方法，所以能够实现回到行开头的功能。


### readline 宏
与方法不同，宏是一组按键，把宏绑定到快捷键后，按下快捷键，宏定义的按键会自动输出到命令行，当前已定义的宏
可以通过bind -s命令查看
```shell
$ bind -s
"\C-i": "pwd"
```
可以看到，pwd这个宏绑定到了Ctrl+i快捷键，按下后，pwd会自动出现在命令行。

> 忽略快捷键语法，可以发现，宏和方法绑定的不同之处在于，宏需要用双引号括住，而方法不需要，记住这个区别，后续
> 我们自定义绑定需要用到

### 自定义
了解了以上内容后，我们就可以按照自己的喜好定制快捷键了，bind命令可以动态修改当前shell的快捷键绑定，语法如下

> bind '***"快捷键"***: ***方法名*** | ***"宏内容"*** '

快捷键包含多个按键，需要用双引号括住，接一个冒号，后续跟一个方法名或一个宏，如上面所说，方法名不需要双引号，宏
则需要用双引号括住，同时由于快捷键经常会有特殊字符，我们将整个bind参数用但因号括住，从而无需自行转义

```shell
$ bind '"\C-i":beginning-of-line'
$ bind '"\C-t":"ttttt"'
```
上面第一个命令将Ctrl+i绑定到了beginning-of-line方法，此时按下Ctrl+i和按下Ctrl+a一样都能回到行首  
第二个命令则是将ttttt这个宏绑定到了Ctrl+t，按下Ctrl+t，会自动输入ttttt

> bind只影响当前的shell，如果想要所有shell都生效，可以将以上绑定写入用户目录的.inputrc文件内，如下
> ```shell
> $ cat .inputrc
> "\C-i":beginning-of-line
> "\C-t":"ttttt"
>```
> 绑定语法与bind命令相同
> 当然也可将bind写入.bashrc文件，效果相同  
 
> 最好使用bind写入bashrc的写法，因为.inputrc会影响所有readline的行为
> 部分情况可能有意想不到的结果

这个例子只列出了使用Ctrl的快捷键，用\C-表示，其他按键也有对应的表示：  

| 表示  | 按键                  |
|-----|---------------------|
| \C- | Ctrl                |
| \M- | Meta 通常被映射为Alt      |
| \e  | Esc  有的系统Alt也会视为Esc |
| \\\ | 反斜线                 |
| \\" | 双引号                 |
| \\' | 单因号                 |

反斜线，单双引号使用时需要转义(这里需要看页面展示，因为markdown自身对反斜线的处理，
原始文本中的反斜线会多一个)

> 当需要输入特殊按键（比如Ctrl+F1的组合），可以在命令行中先按下Ctrl+v,再按下想要的
> 组合键，此时命令行会展示其编码，将这部分复制到上面的快捷键即可

[gnu链接](https://www.gnu.org/software/bash/manual/html_node/Readline-Init-File-Syntax.html)


### stty命令
stty命令可以用来查看当前终端的一些设置
```shell
$ stty -a

speed 38400 baud; rows 37; columns 87; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>;
eol2 = <undef>; swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R;
werase = ^W; lnext = ^V; discard = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
-ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff
-iuclc -ixany -imaxbel iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt
echoctl echoke -flusho -extproc
```
这些输出其实是termios系统库（位于man page的第三类，不是直接的系统调用，其底层系统调用为ioctl）当前的一些配置，termios系统调用用于设置终端的一些属性，比如ECHO属性，用于设置是否在用户输入后回显，一些输入密码的场景，终端不回显用户的输入，就是这个来控制的。具体可以看man page，也可以参考[这个博客](https://blog.nelhage.com/2009/12/a-brief-introduction-to-termios-termios3-and-stty/)



### 终端的一些快捷键
Ctrl-s 暂停输出，通常用于cat了某个特别大的文件，长时间无法结束需要强制结束
Ctrl-q 配套Ctrl-s，恢复输出
