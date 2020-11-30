# 何为Docker

准确的说，我是昨天才刚知道Docker这一个单词的。Docker，直译过来就是码头工人的意思，而Docker的logo则是一个装满了集装箱的大货轮。从命名方式和logo就可以看出来Docker的作者想让其承担的角色：做一个像集装箱那样的容器。

那么问题来了，集装箱解决了什么问题呢？答案是运输的标准化。集装箱的出现，为货物的运输制定了一个标准。正是有了标准的制定，我们不需要知道我们运的是什么货物。在运输时，我们的车只要符合标准所规定的运输条件，就能自由的运送各种各样的货物。而且，各个集装箱在货轮里面码放时变得井然有序，在大大地减少了站用货轮空间的同时，又互不干扰。Docker正是因为这样的思想而诞生的。

# Docker解决了什么问题？

我们所有服务和web应用程序都是部署并运行在服务器上面的，在实际应用中，一台服务器上大概率会运行着不止一个应用程序，而这些不同的应用程序往往依赖着不同的运行环境，就像php开发的网站和java开发的网站所依赖的软件就不一样。所以，我们在分别安装这些程序运行所必备的依赖环境时就很麻烦，要调试很久也就算了，说不定还会遇到各种各样奇葩的问题，比如端口冲突啥的。按照传统方式，我们可以在服务器上创建不同的虚拟机，然后分别在不同的虚拟机里面分别部署并运行不同的应用程序。这样确实可以解决眼前的问题，但是，这样做的弊端就是虚拟机的开销往往很高。一台服务器本身的算力就是有限的，再把这些有限的算力给浪费在虚拟机的开销上，着实划不来。Docker可以实现创建虚拟机实现的效果，而且开销远远低于虚拟机。

在开发的过程中，我们的开发环境和部署环境极有可能是不相同的。比如我现在的开发环境是Windows的系统，Java14，Tomcat9，数据库为MySQL5.6。但是当我把系统开发完成后部署在服务器上时，服务器大概率是跑着Linux系统，Java版本和Tomcat版本也有可能不同，有可能我现在用的数据库版本 在对应的Linux系统上不支持。所以，当我们部署时，这些各种各样的非技术性问题，会让我们十分头疼，令本就稀疏的头发变得越来越少。Docker可以把整个开发环境直接封装起来部署，十分省心。

# Docker常用指令

```shell
#查看正在运行着的容器
[root@VM_0_13_centos ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
4fc27f062088        mysql:5.6           "docker-entrypoint.s…"   23 hours ago        Up 23 hours         0.0.0.0:3306->3306/tcp   mysql
#查看所有的容器（包括已经停止的容器）
[root@VM_0_13_centos ~]# docker container ls -a
#查看所有本地镜像
[root@VM_0_13_centos ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               5.6                 a599e29874aa        6 days ago          302MB
#进入容器 4fc27f062088为容器ID
[root@VM_0_13_centos ~]# docker exec -it 4fc27f062088 /bin/bash
#已经进入容器，可以直接使用MySQL的相关命令
root@4fc27f062088:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 5.6.49 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
#查询所有数据库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)
#一些常用的命令

#查询名字为xx的可用镜像
[root@VM_0_13_centos ~]# docker search xx 
#下载名字为xx，版本号为x.x的镜像到本地（版本号不指定默认pull latest）
[root@VM_0_13_centos ~]# docker pull xx:x.x
#停止容器
[root@VM_0_13_centos ~]# docker stop 4fc27f062088 
4fc27f062088
#可以看到容器已退出
[root@VM_0_13_centos ~]# docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
4fc27f062088        mysql:5.6           "docker-entrypoint.s…"   23 hours ago        Exited (0) 22 seconds ago                       mysql
#查看正在运行的容器列表中已经没有了刚刚停止的容器
[root@VM_0_13_centos ~]# docker container ls
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS   PORTS   NAMES
#启动容器
[root@VM_0_13_centos ~]# docker start 4fc27f062088 
4fc27f062088
[root@VM_0_13_centos ~]# docker container ls
CONTAINER ID   IMAGE   COMMAND  CREATED  STATUS   PORTS      NAMES
4fc27f062088  mysql:5.6  "docker-entrypoint.s…"   23 hours ago        Up 8 seconds        0.0.0.0:3306->3306/tcp   mysql
#删除容器
[root@VM_0_13_centos ~]# docker container rm container_id
#删除镜像
[root@VM_0_13_centos ~]# docker image rm image_id
```

