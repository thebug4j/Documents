[TOC]



# 1、镜像操作命令

![image-20221224220201036](/Users/thebug4j/workspace/github/Documents/学习/assets/image-20221224220201036.png)

```
#1、拉取镜像
docker pull redis
#2、列出镜像
docker images
#3、导出镜像
docker save -o redis.tar redis:latest
#4、删除镜像
docker rmi redis:latest
#5、采用文件方式加载回镜像
docker load -i redis.tar
```

# 2、容器操作命令

![image-20221224223041470](/Users/thebug4j/workspace/github/Documents/学习/assets/image-20221224223041470.png)

```
#1、新建并运行容器
docker run --name mynginx -p 80:80 -d nginx
#2、查看容器状态
docker ps
docker ps -a
#3、查看日志
docker logs mynginx
docker logs -f mynginx
#4、进入容器
docker exec -it mynginx /bin/bash
#5、停止容器（start、pause）
docker stop mynginx
#6、删除容器 （-v命令会删除容器依赖的卷）
docker rm -f -v mynginx
```



# 3、挂载

1. **数据卷挂载（管理卷） ： **：耦合度低，由docker来管理目录，但是目录较深，不好找
2. **目录挂载（绑定挂载卷）**：耦合度高，需要我们自己管理目录，不过目录容易寻找查看

```
1、数据卷挂载 （自行创建管理卷 html）
docker run --name mn -v html:/root/html -p 8080:80 nginx
2、宿主机目录挂载
docker run --name mn -v /Users/thebug4j/html:/root/html -p 8080:80 nginx

```



# 4、Dockerfile

Dockerfile就是一个文本文件，其中包含一个个的指令(Instruction)，用指令来说明要执行什么操作来构建镜像。每一个指令都会形成一层Layer

![image-20230104125402687](/Users/thebug4j/workspace/github/Documents/学习/assets/image-20230104125402687.png)

更新详细语法说明，请参考官网文档： https://docs.docker.com/engine/reference/builder

```
# 指定基础镜像
FROM ubuntu:16.04
# 配置环境变量，JDK的安装目录
ENV JAVA_DIR=/usr/local

# 拷贝jdk和java项目的包
COPY ./jdk8.tar.gz $JAVA_DIR/
COPY ./docker-demo.jar /tmp/app.jar

# 安装JDK
RUN cd $JAVA_DIR && tar -xf ./jdk8.tar.gz && mv ./jdk1.8.0_144 ./java8

# 配置环境变量
ENV JAVA_HOME=$JAVA_DIR/java8
ENV PATH=$PATH:$JAVA_HOME/bin

# 暴露端口
EXPOSE 8090
# 入口，java项目的启动命令
ENTRYPOINT java -jar /tmp/app.jar
```

# 5、自定义镜像

## 1、创建Dockerfile文件

```
#FROM java:8-alpine
FROM openjdk:8-jdk-alpine
COPY ./app.jar /tmp/app.jar
ENTRYPOINT java -jar /tmp/app.jar
```

## 2、构建镜像

```
docker build -t javaweb:1.0 .
```

# 6、DockerCompose

Docker Compose可以基于Compose文件帮我们快速的部署分布式应用，而无需手动一个个创建和运行容器！Compose文件是一个文本文件，通过指令定义集群中的每个容器如何运行。

```
version: "3.2"

services:
  nacos:
    image: nacos/nacos-server
    environment:
      MODE: standalone
    ports:
      - "8848:8848"
  mysql:
    image: mysql:5.7.25
    environment:
      MYSQL_ROOT_PASSWORD: 123
    volumes:
      - "$PWD/mysql/data:/var/lib/mysql"
      - "$PWD/mysql/conf:/etc/mysql/conf.d/"
  userservice:
    build: ./user-service
  orderservice:
    build: ./order-service
  gateway:
    build: ./gateway
    ports:
      - "10010:10010"
```

DockerCompose的详细语法参考官网：https://docs.docker.com/compose/compose-file/
