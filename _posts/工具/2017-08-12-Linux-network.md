---
layout: post
title:   Linux 网络指令
date:   2017-08-12 19:15:10
categories:  工具
tags:  Linux
keywords: Linux
description: 
---

## netstat

利用netstat指令可让你得知整个Linux系统的网络情况。

-a (all)显示所有选项，默认不显示LISTEN相关
-t (tcp)仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化成数字。
-l 仅列出有在 Listen (监听) 的服務状态
-p 显示建立相关链接的程序名
-r 显示路由信息，路由表
-e 显示扩展信息，例如uid等
-s 按各个协议进行统计
-c 每隔一个固定时间，执行该netstat命令。


```
 //列出所有 tcp 端口 netstate -at
 $ netstat -at
 Active Internet connections (servers and established)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State
 tcp        0      0 localhost:30037         *:*                     LISTEN
 tcp        0      0 localhost:ipp           *:*                     LISTEN
 tcp        0      0 *:smtp                  *:*                     LISTEN
 tcp6       0      0 localhost:ipp           [::]:*                  LISTEN


 // 列出所有 udp 端口 netstat -au
 $ netstat -au
 Active Internet connections (servers and established)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State
 udp        0      0 *:bootpc                *:*
 udp        0      0 *:49119                 *:*
 udp        0      0 *:mdns                  *:*
```

netstat -p 可以与其它开关一起使用，就可以添加 “PID/进程名称” 到 netstat 输出中，这样 debugging 的时候可以很方便的发现特定端口运行的程序。
```
//列出 tcp 端口，处于监听状态，且不显示别名，输出中列出进程名称
$ netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      5861/httpd.bin      
tcp        0      0 127.0.0.1:8005          0.0.0.0:*               LISTEN      17292/java          
tcp        0      0 0.0.0.0:8009            0.0.0.0:*               LISTEN      17292/java         
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      17292/java          
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5861/httpd.bin      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      28705/sshd         
tcp6       0      0 :::3306                 :::*                    LISTEN      13572/mysqld        
```
这样就能结合grep找到对应端口运行的程序，并kill掉它，比如
```
$ netstat -tnlp |grep 8080
$ kill -9 17292
```

