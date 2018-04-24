---
layout: post
title:  Spring boot项目部署到Linux
date:   2018-04-23 11:15:10
categories: 工具
tags: Linuxkeywords: Linuxd
escription: ---
----------------------------------

## 1.在Linux上安装好所需要的环境（比如Java环境、Mysql）

 整个项目的部署，只需要用maven将项目打包，上传至linux服务器，运行java -jar xx.jar 则可。 
 无需任何配置，因为Spring Boot自动继承Tomcat插件，所以Linux上Tomcat都无需安装。


* (1)在官网上下载好Linux对应版本的的下载包 jdk-8u161-linux-x64.tar.gz
* (2)用rz -be指令将windows上下载好的下载包传输到Linux 的/usr/java文件夹下
* (3)用指令解压下载包
 
```
tar jdk-8u161-linux-x64.tar.gz
```
Java的文件夹就会在 /usr/java/jdk1.8.0_161 目录下
**（4）配置环境**

```
vim /etc/profile
```
在后面添加环境
```
#set java environment
export JAVA_HOME=/usr/java/jdk1.8.0_161

export JRE_HOME=$JAVA_HOME/jre

export CLASSPATH=$JAVA_HOME/lib:$CLASSPATH

export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```
修改完之后要source一下让配置文件生效。
```
source /etc/profile
```

**（5）java -version查看是否安装成功**
输出以下代表成功
![enter image description here](http://p7lixluhf.bkt.clouddn.com/20180404152842262.png)
 


----------
## 2.将Spring boot工程下的Java项目打包成Jar包

使用Maven打包JAR包。
**（1）右上角点击Edit Configurations...**
**（2）点绿色的+号添加Maven设置，输入Name；在command line 输入clean package -DskipTests=true**
![enter image description here](http://p7lixluhf.bkt.clouddn.com/2018040415362586.png)

（3）发现右上角可选package，运行image
![enter image description here](http://p7lixluhf.bkt.clouddn.com/20180404154128279.png)
（4）输出building success表示成功， 在target目录下就可以找到jar包
![enter image description here](http://p7lixluhf.bkt.clouddn.com/20180404154253628.png)
 


----------
## 3.将Jar包传输到Linux

用rz -be指令将Jar传输到Linux上，用指令运行

```
java -jar xxxxxxx.jar(jar包名字)
```
弹出，运行成功！
![enter image description here](http://p7lixluhf.bkt.clouddn.com/20180404154704605.png)
注意：Jar所使用的Java大版本要和Linux的版本一样，比如都是java 1.8 不然就会报错。


----------
## 4.持续执行
=
用命令启动spring boot 项目，一旦终端命令窗口关闭，项目也就关闭了，所以我们采用脚本的方式来运行jar。这样就算关了终端窗口，程序也在持续执行。

脚本启动，vim 创建 start.sh
```shell
#!/bin/sh
rm -f tpid
nohup java -jar xxx.jar(jar的相对路径或者绝对路径) --spring.profiles.active=stg > /dev/null 2>&1 &
echo $! > tpid

```
脚本关闭

```shell
tpid=`cat tpid | awk '{print $1}'`
tpid=`ps -aef | grep $tpid | awk '{print $2}' |grep $tpid`
if [ ${tpid} ]; then 
        kill -9 $tpid
fi
```

