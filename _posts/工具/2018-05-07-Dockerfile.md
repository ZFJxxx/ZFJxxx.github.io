---
layout: post
title:   Dockerfile
date:   2018-05-07 19:15:10
categories:  工具
tags: Docker
keywords: 
description: 
---

## Dockerfile结构
Dockerfile由一行行命令语句组成，可以快速的创建自定义镜像。
Dockerfile一般分为4个部分：基础镜像信息、维护中信息、镜像操作指令和容器、启动时执行的命令。
```
#第一行必须指令基于的基础镜像
From ubuntu

#维护者信息
MAINTAINER docker_user  docker_user@mail.com

#镜像的操作指令
apt/sourcelist.list

RUN apt-get update && apt-get install -y ngnix 
RUN echo "\ndaemon off;">>/etc/ngnix/nignix.conf

#容器启动时执行指令
CMD /usr/sbin/ngnix
```

比如以ubuntu为父镜像的基础上安装nginx、apache2、openssh-server等软件。
```
#Nginx
#
#VERSION 0.0.1

From ubuntu
MAINTAINER Victor victor@mail.com

RUN apt-get update && apt-get install -y ngnix apache2 openssh-server 

```

编写完之后，通过docker build命令来创建镜像,下面希望指定生成镜像处于/tmp/docker_builder/ ，并希望生成标签first_image
```
$ sudo docker build -t first_image /tmp/docker_builder/ 
```
