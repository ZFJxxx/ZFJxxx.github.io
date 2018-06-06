---
layout: post
title:   Linux 重要指令
date:   2018-03-14 19:15:10
categories:  工具
tags:  Linux
keywords: Linux
description: 
---

## ps

ps命令用于显示当前进程 (process) 的状态。

```
ps [options] [--help]
```

 -A 和 -e 列出所有的进程

ps -ef 是用标准的格式显示进程
```
UID        PID  PPID  C STIME TTY          TIME      CMD
root		  1       0   0  06:50  ?            00:00:02  /sbin/init
root            2      0   0  06:50  ?             00:00:00 [kthreadd]
```

```
UID    //用户ID、但输出的是用户名
PID    //进程的ID
PPID   //父进程ID
C      //进程占用CPU的百分比
STIME  //进程启动到现在的时间
TTY    //该进程在那个终端上运行，若与终端无关，则显示? 若为pts/0等，则表示由网络连接主机进程。
TIME   //此进程运行的总时间
CMD    //命令的名称和参数
```

ps -aux 是用BSD的格式来显示

```
USER       PID %CPU %MEM   VSZ  RSS  TTY                   STAT     START   TIME     COMMAND
root         1  0.0  0.1 19224 1540  ?                     Ss       06:50   0:03     /sbin/init
root         2  0.0  0.0     0    0  ?                     S        06:50   0:00     [kthreadd]
root         3  0.0  0.0     0    0  ?                     S        06:50   0:00     [migration/1]
root         4  0.0  0.0     0    0  ?                     S        06:50   0:00     [ksoftirqd/0]
```
```
USER      //用户名
%CPU      //进程占用的CPU百分比
%MEM      //占用内存的百分比
VSZ      //该进程使用的虚拟內存量（KB）
RSS      //该进程占用的固定內存量（KB）（驻留中页的数量）
STAT      //进程的状态
START    //该进程被触发启动时间
TIME      //该进程实际使用CPU运行的时间

其中STAT状态位常见的状态字符有
D      //无法中断的休眠状态（通常 IO 的进程）；
R      //正在运行可中在队列中可过行的；
S      //处于休眠状态；
T      //停止或被追踪；
W      //进入内存交换 （从内核2.6开始无效）；
X      //死掉的进程 （基本很少见）；
Z      //僵尸进程；
<      //优先级高的进程
N      //优先级较低的进程
L      //有些页被锁进内存；
s      //进程的领导者（在它之下有子进程）；
l      //多线程，克隆线程（使用 CLONE_THREAD, 类似 NPTL pthreads）；
+      //位于后台的进程组；
```
## kill
```
kill -9 [进程ID]    
```
强制终止进程，可用ps -ef 列出进程，然后杀掉想终止的进程
 

## ls

ls命令(list)用于显示指定工作目录下之内容。

```
 ls [-alrtAFR] [name...]
```

ls常用命令

* -a 显示当前目录下的所有文件及文件夹包括隐藏的.和..等
* -al 显示当前目录下的所有文件及文件夹包括隐藏的.和..等并显示详细信息，详细信息包括大小，属组，创建时间
* -l 除文件名称外，亦将文件型态、权限、拥有者、文件大小等资讯详细列出(和-al比不显示隐藏文件)
* -r 将文件以相反次序显示(原定依英文字母次序)
* -t 将文件依建立时间之先后次序列出
* -A 同 -a ，但不列出 "." (目前目录) 及 ".." (父目录)
* -F 在列出的文件名称后加一符号；例如可执行档则加 "*", 目录则加 "/"
* -R 若目录下有文件，则以下之文件亦皆依序列出


例子.详细列出/home下的文件
```
ls -l /home
```
ll是 ls -l的缩写，作用相同，推荐使用
```
ll /home
```

## yum (CentOS)

yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令.

```
yum [options] [command] [package ...]
```

* options：可选，选项包括-h（帮助），-y（当安装过程提示选择全部为"yes"），-q（不显示安装的过程）等等。
* command：要进行的操作。
* package操作的对象。

yum常用命令

```
 1.列出所有可更新的软件清单命令：yum check-update
    2.更新所有软件命令：yum update
    3.仅安装指定的软件命令：yum install <package_name>
    4.仅更新指定的软件命令：yum update <package_name>
    5.列出所有可安裝的软件清单命令：yum list
    6.删除软件包命令：yum remove <package_name>
    7.查找软件包 命令：yum search <keyword>
    8.清除缓存命令:
        yum clean packages: 清除缓存目录下的软件包
        yum clean headers: 清除缓存目录下的 headers
        yum clean oldheaders: 清除缓存目录下旧的 headers
        yum clean, yum clean all (= yum clean packages; yum clean oldheaders) :清除缓存目录下的软件包及旧的headers
```


## apt-get (Debian ubuntu) 

Debian和ubuntu版本的apt-get。 用法和yum类似。


## grep

grep（global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。 

```
grep [option] pattern file
```

* -v   显示不包含匹配文本的所有行。 
* -i   忽略字符大小写的差别。   
* -n   在显示符合样式的那一行之前，标示出该行的列数编号。
* -c   计算符合样式的列数。


```
//1.查找后缀有“test”的文件包含“test”字符串的文件  
$ grep test test* 
testfile1:This a Linux testfile!       #列出testfile1 文件中包含test字符的行  
testfile_2:This is a linux testfile!   #列出testfile_2 文件中包含test字符的行  
testfile_2:Linux test                  #列出testfile_2 文件中包含test字符的行 

//2.查找进程中包含bash的进程
 ps | grep bash
xxxx pts/0    00:00:00 bash
xxxx pts/0    00:00:00 bash

//3.以递归的方式查找符合条件的文件。例如，查找指定目录/etc/acpi 及其子目录（如果存在子目录的话）下所有文件中包含字符串"update"的文件，并打印出该字符串所在行的内容.
grep -r update /etc/acpi 
#下包含“update”的文件  
/etc/acpi/ac.d/85-anacron.sh:# (Things like the slocate updatedb cause a lot of IO.)  
Rather than  
/etc/acpi/resume.d/85-anacron.sh:# (Things like the slocate updatedb cause a lot of  
IO.) Rather than  
/etc/acpi/events/thinkpad-cmos:action=/usr/sbin/thinkpad-keys--update 

//4.反向查找。前面各个例子是查找并打印出符合条件的行，通过"-v"参数可以打印出不符合条件行的内容。
查找文件名中包含 test 的文件中不包含test 的行.
grep -v test *test*
```


## su

su命令用于变更为其他使用者的身份，除 root 外，需要键入该使用者的密码。

```
切换用户
hnlinux@w3cschool.cc:~$ whoami //显示当前用户
hnlinux
hnlinux@w3cschool.cc:~$ pwd //显示当前目录
/home/hnlinux
hnlinux@w3cschool.cc:~$ su - root //切换到root用户
密码： 
root@w3cschool.cc:/home/hnlinux# whoami 
root
root@w3cschool.cc:/home/hnlinux# pwd
/home/hnlinux

切换用户，改变环境变量
hnlinux@w3cschool.cc:~$ whoami //显示当前用户
hnlinux
hnlinux@w3cschool.cc:~$ pwd //显示当前目录
/home/hnlinux
hnlinux@w3cschool.cc:~$ su - root //切换到root用户
密码： 
root@w3cschool.cc:/home/hnlinux# whoami 
root
root@w3cschool.cc:/home/hnlinux# pwd //显示当前目录
/root
```


## echo
linux的echo命令，在shell编程中极为常用,echo命令的功能是在显示器上显示一段文字，一般起到一个提示的作用。 


## wget

```
wget [option] URL
```

* -a<日志文件> 在指定的日志文件中记录资料的执行过程；
* -A<后缀名> 指定要下载文件的后缀名，多个后缀名之间使用逗号进行分隔；
* -b 进行后台的方式运行wget；
* -O   指定下载目录和文件名
* -r 下载整个网站、目录

```
//1.使用wget下载单个文件
wget http://www.linuxde.net/testfile.zip

//2.下载并以不同的文件名保存
wget -O wordpress.zip http://www.linuxde.net/testfile.zip

//3. wget限速下载
wget --limit-rate=300k http://www.linuxde.net/testfile.zip
当你执行wget的时候，它默认会占用全部可能的宽带下载。但是当你准备下载一个大文件，而你还需要下载其它文件时就有必要限速了。 

//4.使用wget断点续传
wget -c http://www.linuxde.net/testfile.zip
使用wget -c重新启动下载中断的文件，对于我们下载大文件时突然由于网络等原因中断非常有帮助，我们可以继续接着下载而不是重新下载一个文件。需要继续中断的下载时可以使用-c参数。

//5.使用wget后台下载
wget -b http://www.linuxde.net/testfile.zip

Continuing in background, pid 1840.
Output will be written to `wget-log'.
对于下载非常大的文件的时候，我们可以使用参数-b进行后台下载，你可以使用以下命令来察看下载进度：  

//6. 下载指定格式文件
wget -r -A.zip http://www.linuxde.net/
```


## find
find 和 grep 是 linux中 最 常 用 的 两 个 搜 索 函 数
凡是搜索文件名，就用find，凡是搜索文件内容，就要grep。

我们使用find来查找想要的文件，但是我们一般查找出来的额并不仅仅是看看而已，还会有进一步的操作，这个时候exec的作用就显现出来了。
```
-exec {} \;
```
一对{}，一个空格和一个\，最后是一个分号。

{} 花括号代表前面find查找出来的文件名。
```
find 命令匹配到了当前目录下的所有普通文件，并在 -exec 选项中使用 ls -l 命令将它们列出。
# find ./ -type f -exec ls -l {} \; 

查找前目录中文件属主具有读、写权限，并且文件所属组的用户和其他用户具有读权限的文件：
# find . -type f -perm 644 -exec ls -l {} \;

为了查找系统中所有文件长度为0的普通文件，并列出它们的完整路径：
# find / -type f -size 0 -exec ls -l {} \;

在目录中查找更改时间在14日以前的文件并删除它们
# find ./ -mtime +14 -type f -exec rm {} \; 

查找文件并移动到指定目录， （ .. 是路径名）
# find  ./  -name  "*.log"  -exec  mv {} .. \;

用exec选项执行 cp 命令，（test3 是个目录，不然cp不进去。 ）
# find  ./  -name  "*.log"  -exec  cp {}  test3  \;　　　

-exec 中使用 grep 命令
# find /etc/  -name "passwd"  -exec  grep  "root" {} \;
```


## scp

scp是 secure copy的缩写, scp是linux系统下基于ssh登陆进行安全的远程文件拷贝命令。

```
scp [可选参数] file_source file_target 
```

* -1： 强制scp命令使用协议ssh1
* -2： 强制scp命令使用协议ssh2
* -4： 强制scp命令只使用IPv4寻址
* -6： 强制scp命令只使用IPv6寻址
* -B： 使用批处理模式（传输过程中不询问传输口令或短语）
* -C： 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
* -p：保留原文件的修改时间，访问时间和访问权限。
* -q： 不显示传输进度条。
* -r： 递归复制整个目录。
* -v：详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。

```
//1、从本地复制到远程 
scp /home/space/music/1.mp3 root@zfj:/home/root/music 

//2、从远程复制到本地
从远程复制到本地，只要将从本地复制到远程的命令的后2个参数调换顺序即可，如下实例
scp root@zfj:/home/root/music /home/music/1.mp3 
```


## rz

rz：运行该命令会弹出一个文件选择窗口，从本地选择文件上传到服务器（receive）
　
注意：单独用rz会有两个问题：上传中断、上传文件变化（md5不同），解决办法是上传是用rz -be

## sz
sz 可以将服务器的文件上传到本地。
```
//rz sz指令安装
yum install lrzsz
sz filename
```
