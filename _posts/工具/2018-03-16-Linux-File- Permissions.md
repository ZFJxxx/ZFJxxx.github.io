---
layout: post
title:   Linux 文件权限
date:   2018-03-16 19:15:10
categories:  工具
tags:  Linux
keywords: Linux
description: 
---

## User, Group 及 Others

[用户与群组]的功能可是相当健全而好用的一个安全防护, 由于 Linux 是个多人多任务的系统，因此可能常常会有多人同时使用这部主机来进行工作的情况发生， 为了考虑每个人的隐私权以及每个人喜好的工作环境。

在 Linux 里面，任何一个文件都具有User, Group 及 Others三种身份的个别权限。
比如，对于一个你自己的文件，你希望能够有读写和运行三种权限，对于你group中的朋友，你希望他们能够读和运行，但是不能改。对于Others你希望他们既不能读写，也不能运行该文件。这就是用户与群组存在的意义。

## Linux 文件属性
先查看文件属性

```
# ls -al
drwx------  2 root  root     4096  3月 31  2017 .aptitude
-rw-------  1 root  root    42842  5月 16 19:55 .bash_history
-rw-r--r--  1 root  root     3106  2月 20  2014 .bashrc
drwx------ 16 root  root     4096  4月  1  2017 .cache
drwx------ 20 root  root     4096  8月 23  2017 .config
drwx------  3 root  root     4096  3月 31  2017 .dbus
drwxr-xr-x  2 root  root     4096  3月 31  2017 Desktop
```

![enter image description here](http://p7lixluhf.bkt.clouddn.com/20180327172044557.png)

![enter image description here](http://p7lixluhf.bkt.clouddn.com/linux123.png)

第一个字符代表这个文件是目录、文件或链接文件等等：
* 当为[ d ]则是目录
* 当为[ - ]则是文件

以三个为一组，且均为rwx 的三个参数的组合。其中，[ r ]代表可读(read)、[ w ]代表可写(write)、[ x ]代表可执行(execute)。分别标识user 、group、others的权限。

**改变文件拥有者**
=
chown ：改变文件拥有者,只有文件主和超级用户才可以便用该命令.

> -c或——changes：效果类似“-v”参数，但仅回报更改的部分；
-f或--quite或——silent：不显示错误信息；
-h或--no-dereference：只对符号连接的文件作修改，而不更改其他任何相关文件；
-R或——recursive：递归处理，将指定目录下的所有文件及子目录一并处理；
-v或——version：显示指令执行过程；

```
//将目录/usr/meng及其下面的所有文件、子目录的文件主改成 liu：
chown -R liu /usr/meng
```


## 改变文件权限

chmod：改变权限,  分别可以使用数字或者是符号来进行权限的变更

owner/group/others 三种身份各有自己的 read/write/execute 权限，其中
* r:4    r对应4
* w:2   w对于2
* x:1   x对于1

比如对于-rwxrwx--- 
* owner = rwx = 4+2+1 = 7
* group = rwx = 4+2+1 = 7
* others= --- = 0+0+0 = 0
该文件的权限数字就是 770

```
//将.bashrc文件权限改成777
 chmod 777 .bashrc
```

> 
-c或——changes：效果类似“-v”参数，但仅回报更改的部分；
-f或--quiet或——silent：不显示错误信息；
-R或——recursive：递归处理，将指令目录下的所有文件及子目录一并处理；
-v或——verbose：显示指令执行过程；

## 改变文件群组

chgrp：改变文件所属群组,组名可以是用户组的id，也可以是用户组的组名。果用户不是该文件的文件主或超级用户(root)，则不能改变该文件的组。 

> -c或——changes：效果类似“-v”参数，但仅回报更改的部分；
-f或--quiet或——silent：不显示错误信息；
-h或--no-dereference：只对符号连接的文件作修改，而不是该其他任何相关文件；
-R或——recursive：递归处理，将指令目录下的所有文件及子目录一并处理；
-v或——verbose：显示指令执行过程；

```
// 将/usr/meng及其子目录下的所有文件的用户组改为group2
chgrp -R group2 /usr/meng
```