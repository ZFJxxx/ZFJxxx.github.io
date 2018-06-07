---
layout: post
title:   Docker In Action
date:   2018-05-06 19:15:10
categories:  工具
tags: Docker
keywords: 
description: 
---

## Docker安装
Ubuntu
```
$ sudo apt-get install -y docker.io 
```
CentOS6
```
$ sudo yum install -y docker-io
```
CentOS7
```
$　sudo yum install -y docker
```

## Docker三大核心概念
* 镜像（image）
* 容器（Container）
* 仓库（Repository）

### 1.镜像
Docker运行容器之间需要本地存在对应镜像，如果不存在，会去默认的仓库（docker hub）下载，用户也可以通过配置使用自定义的镜像仓库。

```
//从Docker hub的Ubuntu仓库下载一个最新的Ubuntu操作系统的镜像
$ sudo docker pull ubuntu

//下载特定版本的Ubuntu镜像
$ sudo docker pull ubuntu：14.04
```
也可以从其它指定仓库下载，比如DockerPool社区的镜像源（dl.dockerpool.com）下载.
```
$ sudo docker pull dl.dockerpool.com:5000/ubuntu
```
下载好之后，就可以使用该镜像了，利用该镜像创建一个容器，在其中运行bash应用
```
$ sudo docker run -t -i ubuntu /bin/bash
```

**查看镜像信息**
```
$ sudo docker images
REPOSITORY   TAG       IMAGE ID      CREATED        VIRTUAL SIZE
ubuntu      14.04     5506de2b643b  1 weeks ago        197.8MB
```
TAG信息用来标记来自同一仓库的不同镜像。为了方便在后续工作中使用这个镜像，还可以使用docker tag命令为本地镜像添加新的标签。tag类似别名，你可以给同一个镜像不同的起tag，但他们实际指向同一个镜像。

**搜索镜像**

如下就可以搜索docker hub中mysql相关镜像
```
$ sudo docker search mysql
```

**删除镜像**
```
//可用标签或者ID标识
$ sudo docker rmi 14.04
```

**创建镜像**

可用已有镜像的容器创建，也可以基于Dockerfile创建,先介绍基于已有镜像容器创建

假设我们在已有ubuntu镜像上新建个文件
```
$ sudo docker run -ti ubuntu:14.04 /bin/bash
root@a925cb0b3f0: /# touch test 
root@a925cb0b3f0: /# exit
```
这个容器的ID为a925cb0b3f0，原先的ubuntu镜像已经发生改变，我们用docker commit命令来提交一个新的镜像，提交时可用ID或者名称来指定容器
```
$ sudo docker commit -m "new file" -a "Docker Newbee" a a925cb0b3f0 test
```
顺利的话会返回新创建的镜像ID信息，再看看镜像信息
```
$ sudo docker images
REPOSITORY   TAG       IMAGE ID      CREATED        VIRTUAL SIZE
test      latest     9e9c814023bc   4 seconds ago      225.4MB
```

**上传镜像**

把自己的镜像上传到Docker Hub仓库(类似github，需要登录)
```
$ sudo docker push test:latest
```

### 2.容器
Docker容器类似一个简易的Linux系统环境，以及运行在其中的应用程序打包成的一个盒子，Docker利用容器来运行和隔离应用。

容器就是从镜像创建的应用运行实例，可以启动、开始、停止、删除，而这些容器之间是相互隔离、互不可见的。


**创建新容器**

可以用creat，这样创建出来的容器是停止状态，可用docker start命令启动
```
$ sudo docker creat -it ubuntu:latest
```
也可以用run，这样就创建出来的就是启动状态的
```
//启动一个容器并输出hello world
$ sudo docker run ubuntu /bin/echo 'hello world'
hello world
```
启动一个bash终端，允许用户进行交互,用户就可以进入容器输入指令
```
$ sudo docker run -t -i ubuntu:14.04 /bin/bash
root@af8bae53bdd3:/#
//退出容器，推出后容器处于终止状态
root@af8bae53bdd3:/# exit
```
也可以处于后台状态运行，加-d
```
$ sudo docker run -d ubuntu:14.04 /bin/bash
```
容器处于后台运行的时候，也可以用exec命令进入该容器
```
$ sudo docker run -idt ubuntu
243c32535da7d1421432143254234324

$ sudo docker ps
CONTAINER ID    IMAGE         COMMAND      CREATED        STATUS 
243c32535da7   ubuntu:latest  "/bin/bash"  18 seconds ago   up
$ sudo docker exec -ti 243c32535da7 /bin/bash
root@243c32535da7:/#
```

**删除容器**
```
$ sudo docker rm 243c32535da7
```


**容器互联实现容器通信**

linking会在源和接受容器之间创建一个隧道，接受容器可以看到源容器指定的信息。

比如web容器连到db容器
```
$ sudo docker -d -P --name web --link db:db
```

### 3.仓库

类似于github的一个远端库，用于存储镜像。
