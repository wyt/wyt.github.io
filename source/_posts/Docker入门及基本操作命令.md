---
title: Docker入门及基本操作命令
date: 2018-7-14 21:56:00
updated: 2018-7-14 21:56:00
comments: true
categories:
- docker
tags:
- docker
---

### 安装

参考 [Docker install](/2018/10/25/Docker%20install/)

<!--more-->

### 镜像

#### 搜索/下载/查看/删除

``` bash
# 搜索镜像
$ docker search redis
```

``` bash
# 下载镜像
$ docker pull redis
```

``` bash
# 查看本地镜像列表
$ docker images
```

``` bash
# 删除指定镜像
$ docker rmi image-id
```

### 容器

#### 实例化容器

``` bash
# 创建一个新的容器并启动
$ docker run --name container-name -d image-name

# 例如，创建一个redis容器，并启动
$ docker run --name redis-ctnr -d redis
```

#### 端口映射

``` bash
# docker容器中程序使用的端口，在容器外无法直接访问，需要将其映射出来
# 将redis默认端口，映射出来，使其外部可访问
$ docker run -d -p 6379:6379 --name redis-ctnr2 redis
```

#### 容器列表

``` bash
# 查看运行的容器列表
$ docker ps

# 查看运行和停止的容器列表
$ docker ps -a
```

#### 启动和停止容器

``` bash
# 停止一个容器
$ docker stop container-name/container-id

# 例如，停止redis
$ docker stop redis-ctnr

# 启动一个容器
$ docker start container-name/container-id

# 启动 redis
$ docker start redis-ctnr
```

#### 容器删除

``` bash
# 删除指定容器
$ docker rm container-id

# 删除
$ docker rm redis-ctnr

# 删除所有容器
$ docker rm $(docker ps -a -q)
```

#### 容器日志

``` bash
# 查看当前容器日志
$ docker logs container-name/container-id

# 查看redis日志
$ docker logs redis-ctnr2
```

#### 登录容器

运行中的容器其实就是一个linux系统，我们可以登录进容器

``` bash
# 登录容器
$ docker exec -it container-name/container-id bash

$ docker exec -it redis-ctnr2 bash

# 使用exit退出登录
```

#### 示例
``` bash
# 实例化一个RabbitMQ容器
$ docker run -d -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

### Dockerfile

#### 写一个SpringBoot示例项目

``` java
package github.wyt;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class DockerFileApplication {

  public static void main(String[] args) {
    SpringApplication.run(DockerFileApplication.class, args);
  }

  @RequestMapping("/")
  public String home() {
    return "Hello docker!";
  }
}

```

打完包后，上传到/tmp/df目录下：

``` bash
[root@localhost df]# pwd
/tmp/df
[root@localhost df]# ls -lh
total 14M
-rw-r--r--. 1 root root 134 Jul 15 17:44 Dockerfile
-rw-r--r--. 1 root root 14M Jul 15 17:37 dockerfile-demo-1.0-SNAPSHOT.jar
```

创建一个Dockerfile文件，内容如下：

``` bash
FROM java:8 # 基镜像

MAINTAINER wangyongtao # 维护者

ADD dockerfile-demo-1.0-SNAPSHOT.jar app.jar # 添加到镜像中并且重命名

EXPOSE 8080 # 将容器的8080端口暴露出来，容器外可做端口映射

ENTRYPOINT ["java","-jar","/app.jar"] #容器启动时运行 java -jar app.jar

```

编译镜像

``` bash
[root@localhost df]# docker build -t wyt/dockerfile-demo .

Sending build context to Docker daemon 14.57 MB
Step 1 : FROM java:8
Trying to pull repository docker.io/library/java ... 
8: Pulling from docker.io/library/java

fce5728aad85: Pull complete 
76610ec20bf5: Pull complete 
60170fec2151: Pull complete 
e98f73de8f0d: Pull complete 
11f7af24ed9c: Pull complete 
49e2d6393f32: Pull complete 
bb9cdec9c7f3: Pull complete 
Digest: sha256:c1ff613e8ba25833d2e1940da0940c3824f03f802c449f3d1815a66b7f8c0e9d
 ---> d23bdf5b1b1b
Step 2 : MAINTAINER wangyongtao
 ---> Running in 12ebd4a0fd57
 ---> 046529d38bd4
Removing intermediate container 12ebd4a0fd57
Step 3 : ADD dockerfile-demo-1.0-SNAPSHOT.jar app.jar
 ---> 7aec1880832e
Removing intermediate container 8712af5da6f3
Step 4 : EXPOSE 8080
 ---> Running in 1f0127445cf1
 ---> 9129fa1cc69b
Removing intermediate container 1f0127445cf1
Step 5 : ENTRYPOINT java -jar /app.jar
 ---> Running in 19e258a38a53
 ---> 916933260f9e
Removing intermediate container 19e258a38a53
Successfully built 916933260f9e
```

编译时也可以带着Tag

``` bash
[root@localhost df]# docker build -t wyt/dockerfile-demo:0.0.1 .
```

查看编译好的镜像

``` bash
[root@localhost df]# docker images
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
wyt/dockerfile-demo                 0.0.1               916933260f9e        30 minutes ago      657.7 MB
wyt/dockerfile-demo                 latest              916933260f9e        30 minutes ago      657.7 MB
```

从镜像实例化容器并启动

``` bash
[root@localhost df]# docker run --name docker-demo2 -p 8080:8080 -d wyt/dockerfile-demo
```

最后使用docker所在系统ip和8080端口即可访问应用

``` bash
[root@localhost df]# curl http://192.168.91.146:8080
Hello docker!
[root@localhost df]# 
```


