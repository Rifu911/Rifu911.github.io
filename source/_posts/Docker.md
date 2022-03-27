---
layout: posts
date: 2019-09-11 09:18:57
title: Docker 
categories: 学习笔记
tags: [Docker, 容器]
---



# Docker

### 一、简介

Docker是一种容器，它里面可以装载多种软件镜像，并将其放进容器，启动该容器就相当于启动了该软件。

eg:在Docker里面装载了Mysql的镜像，Docker就会为其建立一个容器，而启动该容器就相当于启动了Mysql

docker加速：registry.docker-cn.com/library/

<!-- more -->

### 二、安装Docker(Centos7)

```shell
yum install docker
```



### 三、常用操作指令

1）、启动Docker :

```shell
service docker start
```

2）、设置开机自启动：

```shell
systemctl enable docker
```

3）、查看Docker的公共Repository可以安装的镜像

```shell
docker search mysql
```

4）、下载某个镜像(tag指定要安装的版本号)

```shell
docker pull mysql:tag
```

5）、查看docker已安装的镜像​	

```shell
docker images
```

6）、删除镜像

```shell
docker rmi image-id
```

7）、运行镜像

```shell
docker run --name mytomcat -d tomcat:tag
```

		(当出现一串字符，表示运行成功)

8）、查看当前那些镜像在运行

```shell
docker ps
```

9）、停止运行中的容器

```shell
docker stop containerId(或者自己起的容器名字)
```

10）、查看所有容器（镜像安装后称为容器）

```shell
docker ps  -a 
```

11）、启动容器

```shell
docker start containerId
```

12）、删除指定的容器(该容器需要在停止状态)

```shell
docker rm containerId
```

13）、带端口映射的启动容器(-d     代表后台运行)​	

```shell
docker run -d -p  docker.port:container.port   tomcat:tag

eg:docker run -d -p  8888:8080   tomcat:tag    (将主机的8888端口映射到容器的8080端口)
```

14）、查看容器的日志

```shell
docker logs container-name/container-id
```



====注意：一个镜像可以启动为多个端口，在启动的时候指定不同的名字即可====





### 四、docker安装mysql示例

```shell
1.docker pull mysql

2.docker run --name mysql-01 -e  MYSQL_ROOT_PASSWORD=528428 -d mysql:tag
```



> 错误示例：没有映射端口，无法正常使用mysql
>
> 正确的启动方式：
>
> ```shell
> docker run -p 3306:3306 --name mysql-01 -e  MYSQL_ROOT_PASSWORD=528428 -d mysql:tag
> ```

> 高级一点的启动方式：
>
> ```shell
> docker run --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
> 
> 将主机的/my/custom文件挂载到/etc/mysql/conf.d所在的文件夹里面
> (两个配置文件是合并的关系)
> ```